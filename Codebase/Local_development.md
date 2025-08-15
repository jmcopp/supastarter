# Local Development

Learn how to set up your local development environment for Supastarter

This guide will help you set up your local development environment for Supastarter, including the necessary services like PostgreSQL and MinIO S3 storage.

## Prerequisites

To run the application locally, you need to have the following:

- Docker and Docker Compose
- Node.js (v18 or later)
- pnpm

## Setting Up Local Services
Create a docker-compose.yml file in your project root with the following configuration:

`docker-compose.yml`

```yaml
version: '3.8'
 
services:
  postgres:
    image: postgres:15-alpine
    container_name: supastarter-postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: supastarter
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
 
  minio:
    image: minio/minio
    container_name: supastarter-minio
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
 
volumes:
  postgres_data:
  minio_data:
Starting the Services
Start the services using Docker Compose:

docker-compose up -d
Verify that the services are running:

docker-compose ps
Environment Configuration
Create or update your .env file with the following configuration:


# Database
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/supastarter"

# MinIO S3
S3_ACCESS_KEY_ID="minioadmin"
S3_SECRET_ACCESS_KEY="minioadmin"
S3_BUCKET="supastarter"
S3_REGION="us-east-1"
S3_ENDPOINT="http://localhost:9000"
Accessing the Services
PostgreSQL:

Host: localhost
Port: 5432
Username: postgres
Password: postgres
Database: supastarter
MinIO Console:

URL: http://localhost:9001
Username: minioadmin
Password: minioadmin
Creating the S3 Bucket
Access the MinIO Console at http://localhost:9001
Log in with the credentials above
Click "Create Bucket"
Name it "avatars"
Click "Create Bucket"
Start supastarter development server
Start the development server:


pnpm dev
Your Supastarter application should now be running with the local PostgreSQL database and MinIO S3 storage.

Troubleshooting
Database Connection Issues
If you're having trouble connecting to PostgreSQL:

Verify the database is running:

docker-compose ps postgres
Check the logs:

docker-compose logs postgres
S3 Storage Issues
If you're having trouble with S3 storage:

Verify MinIO is running:

docker-compose ps minio
Check the logs:

docker-compose logs minio
Ensure the bucket exists in the MinIO Console
Stopping the Services
To stop all services:


docker-compose down
To stop and remove all data (including volumes):


docker-compose down -v
Additional Resources
Docker Compose Documentation
PostgreSQL Documentation
MinIO Documentation
