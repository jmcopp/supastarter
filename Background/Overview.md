Overview
Learn how to create background tasks & cron jobs in your supastarter application.

With every larger application, you'll get to a point where you need to run tasks in the background or want to queue up or schedule tasks to be executed. This could be sending emails, processing images, or any other task that doesn't need to be done immediately, take too long for an API call, or need to be run at a specific time.

There are multiple approaches to implement tasks in your supastarter application.

Serverless and containerized environments
If you deploy your FastAPI application in a serverless or containerized environment, you want to use a third-party service to schedule and trigger your tasks like trigger.dev or upstash:

trigger.dev
Upstash QStash
Self-hosted environments
If you have your FastAPI backend deployed on a long-running server (like with Docker containers on Fly.io or similar), you can also use Python libraries like Celery with Redis/RabbitMQ, or FastAPI-compatible task queues to create and schedule background tasks.

For containerized FastAPI deployments, consider:
- **Celery**: Distributed task queue for Python with Redis/RabbitMQ
- **RQ (Redis Queue)**: Simple job queue for Python
- **FastAPI-Queue**: Lightweight background tasks for FastAPI
- **APScheduler**: Advanced Python Scheduler for cron-like jobs
