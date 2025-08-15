Uploading files
Learn how to upload files in your supastarter application.

Before you start uploading files, make sure to setup your storage.

You also want to make sure that you have the bucket created that you want to upload files to. For this example we are going to upload a PDF file to a bucket called documents, that we assume you have already created in your storage provider.

Make sure to disable public access to all your buckets as we will care about access control in the API layer of the application.

Add the bucket name to the config
For easy reusability, we recommend adding the bucket name to the FastAPI configuration.

This also allows you to use a dynamic value by using an environment variable instead of a static value.

api/app/core/config.py

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # Storage configuration
    s3_access_key_id: str
    s3_secret_access_key: str
    s3_endpoint: str
    s3_region: str = "auto"
    
    # Bucket names
    documents_bucket_name: str = "documents"
    avatars_bucket_name: str = "avatars"
    public_files_bucket_name: str = "public-files"
    
    class Config:
        env_file = ".env"

settings = Settings()
```
Prepare upload endpoint
As explained on the overview page, supastarter uses the presigned URLs to upload files to your storage provider. So what we need to do first is to create the signed upload URL endpoint to upload files to the documents bucket.

api/app/routers/uploads.py

```python
from fastapi import APIRouter, HTTPException, Depends, Query
from app.core.auth import get_current_user
from app.core.config import settings
from app.services.storage import get_signed_upload_url
from app.models.user import User

router = APIRouter(prefix="/uploads", tags=["uploads"])

@router.post("/signed-upload-url")
async def create_signed_upload_url(
    bucket: str = Query(..., min_length=1),
    path: str = Query(..., min_length=1),
    current_user: User = Depends(get_current_user)
):
    """
    Create a signed URL for uploading files to storage.
    Requires authentication to ensure only logged-in users can upload.
    """
    
    # Only allow uploads to the documents bucket
    if bucket == settings.documents_bucket_name:
        signed_url = await get_signed_upload_url(path, bucket=bucket)
        return {"signed_url": signed_url}
    
    raise HTTPException(status_code=403, detail="Access to this bucket is forbidden")
```
This route will now allow authenticated users to get a signed upload URL for the documents bucket. Be careful though, as this will allow anyone to write any path to the documents bucket.

If you want to only allow uploading to specific paths or check for specific file types, you can add validation logic:

```python
@router.post("/signed-upload-url")
async def create_signed_upload_url(
    bucket: str = Query(..., min_length=1),
    path: str = Query(..., min_length=1),
    current_user: User = Depends(get_current_user)
):
    if bucket == settings.documents_bucket_name:
        # Only allow PDF files
        if not path.endswith(".pdf"):
            raise HTTPException(status_code=400, detail="Only PDF files are allowed")
        
        # Only allow files in the root directory
        if len(path.split("/")) > 1:
            raise HTTPException(status_code=400, detail="Files must be in root directory")
        
        # Only allow paths that include the user id
        if not path.startswith(f"{current_user.id}/"):
            raise HTTPException(status_code=400, detail="File path must start with user ID")
        
        signed_url = await get_signed_upload_url(path, bucket=bucket)
        return {"signed_url": signed_url}
```
Allow public uploads
In general, we don't recommend allowing public uploads as this can lead to security issues or spam to your storage provider.

However, if you want to allow public uploads, you can create a new route that doesn't require authentication.

```python
@router.post("/public-signed-upload-url")
async def create_public_signed_upload_url(
    bucket: str = Query(..., min_length=1),
    path: str = Query(..., min_length=1)
):
    """
    Create a signed URL for public file uploads (no authentication required).
    Use with caution - consider rate limiting and file type restrictions.
    """
    
    if bucket == settings.public_files_bucket_name:
        signed_url = await get_signed_upload_url(path, bucket=bucket)
        return {"signed_url": signed_url}
    
    raise HTTPException(status_code=403, detail="Access to this bucket is forbidden")
```
Upload files from the UI
In order to upload files from the UI, use your preferred method to select a file (like a file input or a dropzone component). Then you need to execute the following steps:

Get a signed upload URL for the file you want to upload.
Upload the file to the signed URL.
Store the file url to your database to be able to use the file later
Here is an example of how you can upload a file from the UI:


export function DocumentUpload() {
	const [uploading, setUploading] = useState(false);
	const { user } = useSession();
	const getSignedUploadUrlMutation = useSignedUploadUrlMutation();
 
	const { getRootProps, getInputProps } = useDropzone({
		onDrop: async (acceptedFiles) => {
			if (!user || !acceptedFiles.length) return;
 
			setUploading(true);
 
			try {
                // we commend to use a unique name for the uploaded file here and store the file name in the database to avoid conflicts
    // we are also prefixing the file path with the user id to enable easier filtering of files later
                const path = `${user.id}/${uuid()}.pdf`; 
				const response = await fetch(`/api/uploads/signed-upload-url?bucket=documents&path=${path}`, {
					method: 'POST',
					headers: {
						'Authorization': `Bearer ${token}` // Use your auth token here
					}
				});
				
				if (!response.ok) {
					throw new Error('Failed to get signed URL');
				}
				
				const { signed_url: signedUrl } = await response.json();
 
				const uploadResponse = await fetch(signedUrl, {
					method: "PUT",
					body: acceptedFiles[0],
					headers: {
						"Content-Type": acceptedFiles[0].type,
					},
				});
 
				if (!uploadResponse.ok) {
					throw new Error("Failed to upload document");
				}
 
    // TODO: store the file path to the database
			} catch (e) {
    // TODO: handle error
			} finally {
				setUploading(false);
			}
		},
		accept: {
			"application/pdf": [".pdf"],
		},
		multiple: false,
	});
 
	return (
		<div {...getRootProps()}>
			<input {...getInputProps()} />
 
			{uploading ? (
                <Spinner />
			) : (
				<Button>Upload document</Button>
			)}
		</div>
	);
}
Now your users can use the upload component to upload files to the documents bucket. In the next step you can learn how to access the uploaded files.

Previous

Connect to S3 storage

Next

Accessing stored files

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




