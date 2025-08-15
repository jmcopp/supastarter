Uploading files
Learn how to upload files in your supastarter application.

Before you start uploading files, make sure to setup your storage.

You also want to make sure that you have the bucket created that you want to upload files to. For this example we are going to upload a PDF file to a bucket called documents, that we assume you have already created in your storage provider.

Make sure to disable public access to all your buckets as we will care about access control in the API layer of the application.

Add the bucket name to the config
For easy reusability, we recommend adding the bucket name to the config.ts file.

This also allows you to use a dynamic value by using an environment variable instead of a static value.

apps/web/config.ts

export const config = {
	storage: {
		bucketNames: {
            // ...
			documents: "documents", // or process.env.DOCUMENTS_BUCKET_NAME if you want to a dynamic value
		},
	},
};
Prepare upload endpoint
As explained on the overview page, supastarter uses the presigned URLs to upload files to your storage provider. So what we need to do first is to extend the api/signed-upload-url API route to be able to upload files to the documents bucket.

packages/api/src/routes/uploads.ts

export const uploadsRouter = new Hono().basePath("/uploads").post(
	"/signed-upload-url",
    // using the authMiddleware will make sure the user is authenticated
	authMiddleware,
	validator(
		"query",
		z.object({
			bucket: z.string().min(1),
			path: z.string().min(1),
		}),
	),
 // ...
	async (c) => {
		const { bucket, path } = c.req.valid("query");
 
  // ...
 
        // only allow uploads to the documents bucket
		if (bucket === config.storage.bucketNames.documents) {
			const signedUrl = await getSignedUploadUrl(path, { bucket });
			return c.json({ signedUrl });
		}
 
		throw new HTTPException(403);
	},
);
This route will now allow authenticated users to get a signed upload URL for the documents bucket. Be careful though, as this will allow anyone to write any path to the documents bucket.

If you want to only allow uploading to specific paths or check for specifc file types, you can add a check for the path or file type:

packages/api/src/routes/uploads.ts

// ...
if (bucket === config.storage.bucketNames.documents) {
    // only allow pdf files
    if (!path.endsWith(".pdf")) {
        throw new HTTPException(400);
    }
    
    // only allow files in the root directory
    if (path.split("/").length > 1) {
        throw new HTTPException(400);
    }
 
    // only allow paths that include the user id
    if (!path.startsWith(`${c.get("user").id}/`)) {
        throw new HTTPException(400);
    }
}
// ...
Allow public uploads
In general, we don't recommend allowing public uploads as this can lead to security issues or spam to your storage provider.

However, if you want to allow public uploads, you can create a new route that doesn't require authentication.

packages/api/src/routes/uploads.ts

export const uploadsRouter = new Hono().basePath("/uploads").post(
	"/public-signed-upload-url",
	async (c) => {
		const { bucket, path } = c.req.valid("query");
 
		if (bucket === config.storage.bucketNames.publicFiles) {
			const signedUrl = await getSignedUploadUrl(path, { bucket });
			return c.json({ signedUrl });
		}
 
		throw new HTTPException(403);
	}
);
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
				const { signedUrl } = await getSignedUploadUrlMutation.mutateAsync({
					path,
					bucket: config.storage.bucketNames.documents,
				});
 
				const response = await fetch(signedUrl, {
					method: "PUT",
					body: acceptedFiles[0],
					headers: {
						"Content-Type": acceptedFiles[0].type,
					},
				});
 
				if (!response.ok) {
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



Blog
Documentation
Demo
Tools
SaaS ideas generator
Best SaaS ideas 2025
Boilerplates and Stacks
Showcase
Changelog
Roadmap
Become an affiliate
Privacy policy
Terms of service
Acceptable use
Disclaimer
License
Next.js SaaS starter kit
Next.js SaaS boilerplate
Nuxt SaaS starter kit
Next.js SaaS boilerplate
Nuxt SaaS boilerplate
SvelteKit SaaS starter kit
Next.js starter template
Nuxt starter template
SvelteKit starter template

