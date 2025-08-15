Streaming responses
Learn how to stream responses from your API.

As supastarter integrates FastAPI as the API framework, you can easily stream responses from the API which is useful if you are working with AI providers.

Streaming with the API
With AI SDK and Streaming
When working with AI providers, you can use FastAPI's streaming response capability:

```python
from fastapi import APIRouter
from fastapi.responses import StreamingResponse
import json
import asyncio

router = APIRouter(prefix="/ai", tags=["ai"])

@router.post("/chat-response")
async def chat_response(messages: list, current_user: User = Depends(get_current_user)):
    """Stream AI chat responses."""
    
    async def generate_response():
        # Your AI provider streaming logic here
        for chunk in ai_provider.stream_text(messages):
            yield f"data: {json.dumps(chunk)}\n\n"
            await asyncio.sleep(0.01)  # Small delay for smooth streaming
    
    return StreamingResponse(
        generate_response(),
        media_type="text/plain",
        headers={"Cache-Control": "no-cache"}
    )
```

With FastAPI Streaming
If you want to stream any data from your API, you can use FastAPI's StreamingResponse:

```python
from fastapi.responses import StreamingResponse
import asyncio

@router.post("/stream")
async def stream_data():
    """Stream simple text data."""
    
    async def generate():
        yield "Hello"
        await asyncio.sleep(1)
        yield " FastAPI!"
    
    return StreamingResponse(generate(), media_type="text/plain")
```

To learn more about streaming with FastAPI, you can read the FastAPI documentation.

Previous

Use locale in API endpoints

Next

OpenAPI documentation

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




