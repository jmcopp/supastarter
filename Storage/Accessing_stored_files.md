Accessing stored files
Learn how to access and download files stored in your supastarter application.

In the previous step you learned how to upload files to your storage provider. At this point, these uploaded files are not accessible to anyone including the user that uploaded them.

This behavior is by design as we want control the access to the files in the API layer of the application.

So in this step you will learn how to make these files accessible to the user(s) that should be allowed to access them.

Reminder: The uploaded files from the previous step are stored in the documents bucket prefixed with the user's id.

Accessing the files
To access the files, you need to create a signed URL for the file.

This can either be done on demand or if you are listing your file entries from the database which should have the file path attached to them, you can already create a signed URL for each file entry.

All you need to do is to call the get_signed_url function from your storage service.

```python
from app.services.storage import get_signed_url
from app.core.config import settings
 
signed_url = await get_signed_url(
    file_path, 
    bucket=settings.documents_bucket_name,
    expires_in=3600  # set the expiration to what makes sense for your use case (in seconds)
)
```
In the above example, we are creating a signed URL for the file based on the filePath which would be the value from the database entry and set the expiration to 1 hour.

The returned URL can then be used to access the file from the browser and will be valid for the duration you set.

Note: The signed URL will work for everyone that has access to the URL, so be careful with the expiration time and who you output the signed URL to.

With this you could for example create a download button in your UI that calls an API route that checks the permission of the user and creates a signed URL for the file.

api/app/routers/documents.py

```python
from fastapi import APIRouter, HTTPException, Depends, Query
from app.core.auth import get_current_user
from app.core.config import settings
from app.services.storage import get_signed_url
from app.models.user import User
from app.services.documents import get_document_by_id

router = APIRouter(prefix="/documents", tags=["documents"])

@router.get("/download")
async def download_document(
    file_id: str = Query(..., min_length=1),
    current_user: User = Depends(get_current_user)
):
    """
    Create a signed URL for downloading a document.
    Requires authentication and ownership verification.
    """
    
    # Get the file path from the database by the provided file_id
    document = await get_document_by_id(file_id)
    
    if not document:
        raise HTTPException(status_code=404, detail="Document not found")
    
    # Check if the user has access to the file
    if document.user_id != current_user.id:
        raise HTTPException(status_code=403, detail="Access denied")
    
    signed_url = await get_signed_url(
        document.file_path,
        bucket=settings.documents_bucket_name,
        expires_in=60  # expire in 1 minute to avoid leaking the file path
    )
    
    return {"signed_url": signed_url}
```
Then you can fetch the signed URL form the frontend and open it to download the file.


const DownloadFileButton = ({ fileId }: { fileId: string }) => {
    const downloadFile = async () => {
        const response = await fetch(`/api/documents/download?file_id=${fileId}`, {
            method: 'GET',
            headers: {
                'Authorization': `Bearer ${token}` // Use your auth token here
            }
        });
 
        if (!response.ok) {
            throw new Error('Failed to download file');
        }
 
        const { signed_url: signedUrl } = await response.json();
 
        window.open(signedUrl, '_blank');
    };
 
    return <button onClick={downloadFile}>Download</button>;
};
Alternative: Create a file proxy
If you want to avoid creating signed URLs each time you can create a proxy endpoint that will automatically create a signed URL for the requested file. This proxy endpoint should also include additional logic like checking if the user has access to the file or adding a cache layer.

api/app/routers/file_proxy.py

```python
from fastapi import APIRouter, HTTPException, Depends, Path
from fastapi.responses import RedirectResponse
from app.core.auth import get_current_user
from app.core.config import settings
from app.services.storage import get_signed_url
from app.models.user import User

router = APIRouter(prefix="/file-proxy", tags=["file-proxy"])

@router.get("/{bucket}/{file_path:path}")
async def proxy_file(
    bucket: str = Path(...),
    file_path: str = Path(...),
    current_user: User = Depends(get_current_user)
):
    """
    Create a file proxy that redirects to a signed URL.
    Includes authentication and access control.
    """
    
    if not bucket or not file_path:
        raise HTTPException(status_code=400, detail="Invalid path")
    
    # Only make this available for the documents bucket
    if bucket == settings.documents_bucket_name:
        
        # Check if the user has access to the file
        # You should implement your access control logic here
        # For example, check if file_path starts with user.id
        if not file_path.startswith(f"{current_user.id}/"):
            raise HTTPException(status_code=403, detail="Access denied")
        
        signed_url = await get_signed_url(
            file_path,
            bucket=bucket,
            expires_in=3600
        )
        
        return RedirectResponse(
            url=signed_url,
            # Optionally add cache duration to save compute cost
            headers={"Cache-Control": "max-age=3600"}
        )
    
    raise HTTPException(status_code=404, detail="Not found")
```
