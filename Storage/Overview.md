Overview
Learn how to use storage in supastarter to store files in the cloud.

With supastarter it's really easy to upload and store files like images or documents to the cloud. supastarter out-of-the-box uses the storage functionality to enable the upload of user avatars and organiaztion logos, but you can use it for any other kind of files as well.

We currently support all S3 compatible storage providers like AWS S3, DigitalOcean Spaces, MinIO, etc. and Supabase Storage.

Uploading files in a serverless architecture
supastarter uses Next.js route handlers for providing the API, which have some limitations when being deployed on serverless platforms like Vercel or Netlify. For example there is a maximum execution time and a maximum payload size. These limitations are not a problem for most use cases, but if you need to upload very large files or need to process the files in a way that takes a long time, you might need to use a different approach.

That's why supastarter utilizes the presigned URLs feature provided by S3 compatible storage providers. That means instead of sending the files to the serverless function, the client requests a presigned URL from the serverless function and then uploads the file directly to the storage provider. This way you can make sure to not expose any credentials, handle authentication and authorization and also don't need to worry about the limitations of the serverless platforms.

Connect to S3 storage
Upload files
Access stored files
Previous

Add a new onboarding step

Next

Connect to S3 storage

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




