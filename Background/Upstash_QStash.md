Upstash QStash
Learn how to set up background jobs, cron jobs and queues with Upstash in your supastarter application.

Upstash QStash is a reliable message queue and job scheduler that's perfect for serverless environments. It allows you to create background jobs, schedule tasks, and handle webhook processing with ease.

Prerequisites
Before getting started, you'll need:

An Upstash account
A QStash service created in your Upstash dashboard
Your QStash credentials (URL and token)
1. Install Dependencies
First, install the QStash SDK for Python:

```bash
pip install qstash-py
# or add to your requirements.txt
echo "qstash-py>=1.0.0" >> api/requirements.txt
```
2. Environment Variables
Add the following environment variables to your .env file:

```bash
QSTASH_URL=https://qstash.upstash.io
QSTASH_TOKEN=your_qstash_token_here
QSTASH_CURRENT_SIGNING_KEY=your_current_signing_key_here
QSTASH_NEXT_SIGNING_KEY=your_next_signing_key_here
```
You can find these values in your Upstash dashboard under the QStash service.

3. Create the QStash Client
Create a utility file to initialize the QStash client:

api/app/services/qstash.py

```python
from qstash import Client
import os
from app.core.config import settings

# Initialize QStash client
qstash_client = Client(
    token=settings.qstash_token,
    url=settings.qstash_url
)

def get_qstash_client() -> Client:
    """Get the QStash client instance."""
    return qstash_client
```
4. Create a task router
Create a new router to handle your tasks:

api/app/routers/tasks.py

```python
from fastapi import APIRouter, HTTPException, Request, Depends
from qstash import Receiver
import json
import logging
from app.core.config import settings

router = APIRouter(prefix="/tasks", tags=["tasks"])
logger = logging.getLogger(__name__)

def verify_qstash_signature(request: Request) -> bool:
    """Verify QStash signature for webhook security."""
    try:
        signature = request.headers.get("upstash-signature")
        if not signature:
            raise HTTPException(status_code=401, detail="Missing signature")
        
        # Note: You'll need to implement signature verification based on QStash docs
        # This is a placeholder for the actual verification logic
        receiver = Receiver(
            current_signing_key=settings.qstash_current_signing_key,
            next_signing_key=settings.qstash_next_signing_key
        )
        
        body = request.body()
        is_valid = receiver.verify(body=body, signature=signature)
        
        if not is_valid:
            raise HTTPException(status_code=401, detail="Invalid signature")
        
        return True
        
    except Exception as e:
        logger.error(f"QStash signature verification failed: {e}")
        raise HTTPException(status_code=401, detail="Invalid signature")

@router.post("/test")
async def test_task(
    request: Request,
    verify: bool = Depends(verify_qstash_signature)
):
    """Handle test task from QStash."""
    try:
        body = await request.json()
        message = body.get("message", "")
        
        logger.info(f"Received task message: {message}")
        
        # Process your task here
        # Example: send email, process data, etc.
        
        return {"message": "Task received and processed"}
        
    except Exception as e:
        logger.error(f"Error processing task: {e}")
        raise HTTPException(status_code=500, detail="Task processing failed")
```
5. Trigger a task
To trigger your task, you can use the qstash client to send a request to the task router:

```python
from app.services.qstash import get_qstash_client
from app.core.config import settings

async def trigger_test_task(message: str):
    """Trigger a test task via QStash."""
    client = get_qstash_client()
    
    result = await client.publish_json(
        url=f"{settings.api_base_url}/tasks/test",
        body={"message": message}
    )
    
    return result
```

Usage in your FastAPI endpoints:

```python
from fastapi import APIRouter
from app.services.qstash import trigger_test_task

@router.post("/trigger-task")
async def trigger_task():
    result = await trigger_test_task("Hello from FastAPI!")
    return {"status": "Task triggered", "result": result}
```
6. Usage Examples

**Cron jobs**
To schedule a task to run at a specific time, you can use the qstash client to create a cron job:

```python
from app.services.qstash import get_qstash_client
from app.core.config import settings

async def create_cron_job():
    """Create a cron job that runs every minute."""
    client = get_qstash_client()
    
    schedule = await client.schedules.create(
        destination=f"{settings.api_base_url}/tasks/test",
        cron="*/1 * * * *"  # Every minute
    )
    
    return schedule
```

**Queues**
To create a queue, you can use the qstash client to create a queue and sequence of tasks:

```python
from app.services.qstash import get_qstash_client
from app.core.config import settings

async def create_task_queue():
    """Create a queue with sequential tasks."""
    client = get_qstash_client()
    
    # Create a queue
    queue = await client.queues.create(name="my-queue")
    
    # Add tasks to the queue
    await queue.enqueue_json(
        url=f"{settings.api_base_url}/tasks/step-1",
        body={"message": "Step 1: Hello, world!"}
    )
    
    await queue.enqueue_json(
        url=f"{settings.api_base_url}/tasks/step-2", 
        body={"message": "Step 2: Processing complete!"}
    )
    
    return queue
```
Previous

trigger.dev

Next

Going to production

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




