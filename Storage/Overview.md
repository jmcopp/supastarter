Overview
Learn how to use storage in supastarter to store files in the cloud.

With supastarter it's really easy to upload and store files like images or documents to the cloud. supastarter out-of-the-box uses the storage functionality to enable the upload of user avatars and organization logos, but you can use it for any other kind of files as well.

We currently support all S3 compatible storage providers like AWS S3, DigitalOcean Spaces, MinIO, etc. and Supabase Storage.

Uploading files in a scalable architecture
supastarter uses FastAPI for the backend API with multi-container deployment support. This architecture provides excellent performance for file uploads while supporting both containerized and serverless deployments.

For optimal performance and security, supastarter utilizes the presigned URLs feature provided by S3 compatible storage providers. Instead of routing files through the FastAPI backend, the client requests a presigned URL from the API and then uploads the file directly to the storage provider. This approach:

- Eliminates bandwidth costs on your API server
- Provides better upload performance for large files
- Maintains secure authentication and authorization
- Works seamlessly with both containerized and serverless deployments
- Enables better scalability as upload traffic doesn't affect your API performance

Connect to S3 storage
Upload files
Access stored files
Previous

Add a new onboarding step

Next

Connect to S3 storage

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




