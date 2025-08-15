# API Overview

Learn about supastarter's enterprise-grade FastAPI backend architecture.

## FastAPI Multi-Container Architecture

Supastarter's API layer is built with **FastAPI** and deployed across multiple containers for optimal security, performance, and scalability:

- **API Main Container**: Core business logic with user authentication and database operations
- **API GPU Container**: Optional GPU-intensive operations (ML, AI, image processing)
- **Hybrid Authentication**: Seamless integration with Supabase Auth and custom JWT handling
- **Auto-generated Documentation**: OpenAPI/Swagger documentation with full type safety

## Why FastAPI?

FastAPI is a modern, high-performance web framework for building APIs with Python 3.7+:

- **Automatic OpenAPI documentation** generation with Swagger UI
- **High performance** comparable to NodeJS and Go
- **Type safety** with Pydantic models and automatic validation
- **Async support** for high concurrency applications
- **Easy deployment** across multiple container architectures
- **Industry standard** for enterprise Python APIs

## Container Architecture

### API Main Container (`api-main/`)
- **Location**: `api-main/app/` directory
- **Purpose**: Primary business logic, authentication, database operations
- **Scaling**: Scale-to-zero for cost optimization
- **Network**: Private internal networking only

### API GPU Container (`api-gpu/`) - Optional
- **Location**: `api-gpu/app/` directory  
- **Purpose**: GPU-intensive operations (ML inference, image processing)
- **Scaling**: Scale-to-zero to optimize GPU costs
- **Hardware**: GPU-enabled machines when needed

## Security & Authentication

- **Hybrid auth system** supporting Supabase Auth + custom JWT
- **Context isolation** with per-request user sessions
- **Row Level Security** automatically enforced via Supabase
- **Rate limiting** and security middleware
- **Request tracing** for debugging and monitoring

## API Components

### [Define an API Endpoint](Define_an_API_endpoint.md)
Learn how to create new FastAPI endpoints with automatic OpenAPI documentation and type validation.

### [Protect API Endpoints](Protect_API_endpoints.md)
Implement authentication and authorization using the hybrid auth system with Supabase integration.

### [Use API in Frontend](Use_API_in_application.md)
Connect your Next.js frontend to FastAPI backend with type-safe client generation.

### [Streaming Responses](Streaming_responses.md)
Implement real-time features with FastAPI streaming responses and WebSocket support.

### [Localization Support](Use_locale_in_API_endpoints.md)
Add multi-language support to your API endpoints with FastAPI i18n integration.

## Development Features

### Auto-Generated Documentation
Access comprehensive API documentation at:
- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`
- **OpenAPI JSON**: `http://localhost:8000/openapi.json`

### Type Safety
```python
# Automatic request/response validation
from pydantic import BaseModel

class ItemCreate(BaseModel):
    title: str
    description: str | None = None

@router.post("/items/", response_model=Item)
async def create_item(item: ItemCreate, user_id: str = Depends(get_current_user)):
    # Fully type-safe with automatic validation
    return await create_user_item(item, user_id)
```

### Database Integration
```python
# SQLModel with Supabase + RLS
from sqlmodel import select

async def get_user_items(user_id: str, db: AsyncSession):
    # RLS automatically filters by user context
    statement = select(Item).where(Item.owner_id == user_id)
    return await db.exec(statement).all()
```
