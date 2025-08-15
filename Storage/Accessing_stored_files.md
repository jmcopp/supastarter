Accessing stored files
Learn how to access and download files stored in your supastarter application.

In the previous step you learned how to upload files to your storage provider. At this point, these uploaded files are not accessible to anyone including the user that uploaded them.

This behavior is by design as we want control the access to the files in the API layer of the application.

So in this step you will learn how to make these files accessible to the user(s) that should be allowed to access them.

Reminder: The uploaded files from the previous step are stored in the documents bucket prefixed with the user's id.

Accessing the files
To access the files, you need to create a signed URL for the file.

This can either be done on demand or if you are listing your file entries from the database which should have the file path attached to them, you can already create a signed URL for each file entry.

All you need to do is to call the getSignedUrl function from the storage package.


import { getSignedUrl } from '@supabase/storage-js';
 
const signedUrl = await getSignedUrl(filePath, {
    bucket: config.storage.bucketNames.documents,
    expiresIn: 60 * 60, // set the expiration to what makes sense for your use case
});
In the above example, we are creating a signed URL for the file based on the filePath which would be the value from the database entry and set the expiration to 1 hour.

The returned URL can then be used to access the file from the browser and will be valid for the duration you set.

Note: The signed URL will work for everyone that has access to the URL, so be careful with the expiration time and who you output the signed URL to.

With this you could for example create a download button in your UI that calls an API route that checks the permission of the user and creates a signed URL for the file.

packages/api/src/routes/documents.ts

export const documentsRouter = new Hono().basePath("/documents").get(
	"/download",
    // using the authMiddleware will make sure the user is authenticated
	authMiddleware,
	validator(
		"query",
		z.object({
			fileId: z.string().min(1),
		}),
	),
 // ...
	async (c) => {
		const { fileId } = c.req.valid("query");
        const user = c.get("user");
 
  // ... get the file path from the database by the provided fileId
 
        // ... check with the `user` object if the user has access to the file
 
		const signedUrl = await getSignedUrl(filePath, {
            bucket: config.storage.bucketNames.documents,
            expiresIn: 60, // expire in 1 minute to avoid leaking the file path
        });
 
        return c.json({ signedUrl });
	},
);
Then you can fetch the signed URL form the frontend and open it to download the file.


const DownloadFileButton = ({ fileId }: { fileId: string }) => {
    const downloadFile = async () => {
        const response = await apiClient.documents.download.$get({
            query: {
                fileId,
            },
        });
 
        if (!response.ok) {
            throw new Error('Failed to download file');
        }
 
        const { signedUrl } = await response.json();
 
        window.open(signedUrl, '_blank');
    };
 
    return <button onClick={downloadFile}>Download</button>;
};
Alternative: Create a file proxy
If you to avoid creating signed URLs each time you can create a proxy endpoint that will automatically create a signed URL for the requested file. This proxy endpoint should also include additional logic like checking if the user has access to the file or adding a cache layer.

apps/web/app/document-proxy/route.ts

import { config } from "@repo/config";
import { getSignedUrl } from "@repo/storage";
import { NextResponse } from "next/server";
 
export const GET = async (
	req: Request,
	{ params }: { params: Promise<{ path: string[] }> },
) => {
	const { path } = await params;
 
	const [bucket, ...rest] = path;
 
	if (!bucket || !rest.length) {
		return new Response("Invalid path", { status: 400 });
	}
 
    // only make this available for the documents
	if (bucket === config.storage.bucketNames.documents) {
 
        // ... check if the user has access to the file
 
		const signedUrl = await getSignedUrl(rest.join("/"), {
			bucket,
			expiresIn: 60 * 60,
		});
 
		return NextResponse.redirect(signedUrl, {
            // optionally add a cache duration to the response so to save compute cost on this endpoint
			headers: { "Cache-Control": "max-age=3600" },
		});
	}
 
	return new Response("Not found", {
		status: 404,
	});
};
Previous

Uploading files

Next

Overview

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




