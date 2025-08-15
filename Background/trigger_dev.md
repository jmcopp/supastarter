trigger.dev
Learn how to set up background jobs, cron jobs and queues with trigger.dev in your supastarter application.

Before we get started, make sure to create a new trigger.dev and project.

You can find a full example of this recipe in the feat/trigger.dev branch of the supastarter-nextjs repository.

1. Create a new tasks module in your FastAPI application
Create a new directory for tasks in your FastAPI project and add the following files:

api/tasks/requirements.txt

```
trigger.dev==3.3.17
sqlmodel>=0.0.8
pydantic>=2.0.0
```

api/tasks/pyproject.toml

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "supastarter-tasks"
version = "0.1.0"
dependencies = [
    "trigger.dev==3.3.17",
    "sqlmodel>=0.0.8",
    "pydantic>=2.0.0",
]

[project.scripts]
dev = "trigger-cli dev --env-file ../.env"
deploy = "trigger-cli deploy"
```
api/tasks/trigger.config.py

```python
from trigger import configure

configure(
    project="your_project_id",
    runtime="python",
    log_level="info",
    max_duration=300,
    dirs=["./trigger_tasks"],
    # Database connection will use your existing FastAPI database connection
)
```
2. Create your first task
Now to create your first task, create a new file test_task.py in the api/tasks/trigger_tasks directory and add the following code:

api/tasks/trigger_tasks/test_task.py

```python
from trigger import task
import logging

logger = logging.getLogger(__name__)

@task(id="test-task")
async def test_task():
    """A simple test task to verify trigger.dev integration."""
    logger.info("Test task executed successfully")
    return {"status": "completed", "message": "Test task ran"}
```
Read the trigger.dev documentation for more information on how to create tasks or cron jobs.

3. Test your task
You can easily test your task locally by navigating to the tasks directory and running:

```bash
cd api/tasks
python -m trigger dev
```

This will deploy your task to trigger.dev in the development environment, so you can trigger it from there. When you run the task in the development environment, it will be executed on your local machine.

4. Deploy your task
To deploy your task to trigger.dev, run:

```bash
cd api/tasks
python -m trigger deploy
```

You can also add this command as an automated deployment step in your CI/CD pipeline, by creating a new github action.

You need to add the TRIGGER_ACCESS_TOKEN secret to your repositories secrets, which you can create in the trigger.dev dashboard.

.github/workflows/trigger-deploy.yml

name: Deploy to trigger.dev (prod)
 
on:
  push:
    branches:
      - main
 
jobs:
  deploy:
    runs-on: ubuntu-latest
 
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: lts/*
      - uses: pnpm/action-setup@v4
      - name: Install dependencies
        run: pnpm install
      - name: Deploy trigger tasks
        env:
          TRIGGER_ACCESS_TOKEN: ${{ secrets.TRIGGER_ACCESS_TOKEN }}
        run: |
          cd api/tasks
          python -m trigger deploy

5. Programmatically trigger a task
If you want to trigger one of your tasks from your FastAPI application, for example as an action from an API call, you can import and trigger the task.

Create a trigger client in your FastAPI app:

api/app/services/tasks.py

```python
from trigger import TriggerClient
import os

# Initialize trigger client
trigger_client = TriggerClient(
    api_key=os.getenv("TRIGGER_ACCESS_TOKEN"),
    endpoint="https://api.trigger.dev"
)

async def trigger_test_task():
    """Trigger the test task."""
    result = await trigger_client.send_event(
        event_name="test-task",
        payload={}
    )
    return result
```

Use in your FastAPI endpoints:

api/app/routers/newsletter.py

```python
from fastapi import APIRouter, BackgroundTasks
from app.services.tasks import trigger_test_task

router = APIRouter(prefix="/newsletter", tags=["newsletter"])

@router.post("/signup")
async def newsletter_signup(
    email: str,
    background_tasks: BackgroundTasks
):
    # Handle newsletter signup logic...
    
    # Trigger the background task
    background_tasks.add_task(trigger_test_task)
    
    return {"message": "Subscription successful"}
```
