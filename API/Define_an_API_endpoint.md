# Define an API Endpoint

Learn how to define FastAPI endpoints in your supastarter multi-container architecture.

## Overview

Supastarter uses **FastAPI** for all API endpoints with:
- **Automatic OpenAPI documentation** generation
- **Type-safe request/response models** with Pydantic
- **SQLModel integration** for database operations
- **Supabase Auth middleware** for user authentication
- **Row Level Security** automatically enforced

This guide assumes you have SQLModel models defined for your feature. See the [Database Documentation](../Database/Overview.md) for model creation.

## Create a New Router

In FastAPI, routers organize related endpoints together. Create routers in the `api-main/app/routes/` directory.

### Directory Structure
For complex features with multiple endpoints:
```
api-main/app/routes/
├── posts/
│   ├── __init__.py
│   ├── router.py
│   └── models.py
└── users/
    ├── __init__.py
    └── router.py
```

For simple features:
```
api-main/app/routes/
├── posts.py
└── users.py
```

### Create Posts Router

**File**: `api-main/app/routes/posts/router.py`

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlmodel import Session, select
from typing import List
from app.core.database import get_db
from app.core.auth import get_current_user
from app.models.posts import Post, PostCreate, PostUpdate

# Create router with prefix
router = APIRouter(
    prefix="/posts",
    tags=["Posts"],
    responses={404: {"description": "Not found"}}
)
```

The `prefix="/posts"` automatically prefixes all endpoints with `/posts`, making them available at `/api/posts/`.

## Define Endpoints

### Get All Posts Endpoint

```python
@router.get("/", response_model=List[Post])
async def get_posts(
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
) -> List[Post]:
    """
    Get all posts for the current user.
    
    - **Returns**: List of posts owned by the current user
    - **Security**: Requires authentication
    - **RLS**: Automatically filters posts by owner_id
    """
    statement = select(Post).where(Post.owner_id == current_user)
    posts = db.exec(statement).all()
    return posts
```

### Key Features:
- **Automatic OpenAPI docs**: The docstring and type hints generate Swagger documentation
- **Type safety**: `response_model=List[Post]` ensures response validation
- **Authentication**: `get_current_user` dependency provides user context
- **RLS Integration**: Database queries automatically filter by user ownership

## Mount Router in Main App

Add the router to your main FastAPI application:

**File**: `api-main/app/main.py`

```python
from fastapi import FastAPI
from app.routes.posts.router import router as posts_router
from app.routes.users.router import router as users_router

app = FastAPI(
    title="Supastarter API",
    description="Multi-container FastAPI backend",
    version="1.0.0",
    docs_url="/docs",  # Swagger UI
    redoc_url="/redoc"  # ReDoc
)

# Include routers
app.include_router(posts_router, prefix="/api")
app.include_router(users_router, prefix="/api")
```

Now your endpoints are available at:
- `GET /api/posts/` - Get all posts
- `POST /api/posts/` - Create post
- `GET /api/posts/{id}` - Get specific post
- Documentation at `/docs` and `/redoc`

## CRUD Endpoints

### Create a Post

```python
@router.post("/", response_model=Post)
async def create_post(
    post_data: PostCreate,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
) -> Post:
    """
    Create a new post.
    
    - **post_data**: Post creation data (title, content)
    - **Returns**: The created post
    - **Security**: Requires authentication, auto-assigns owner
    """
    # Create post with automatic owner assignment
    post = Post(
        title=post_data.title,
        content=post_data.content,
        owner_id=current_user  # Automatically set from JWT
    )
    
    db.add(post)
    db.commit()
    db.refresh(post)
    
    return post
```

### Pydantic Models

**File**: `api-main/app/models/posts.py`

```python
from sqlmodel import SQLModel, Field
from typing import Optional
from datetime import datetime
import uuid

# Base model for shared fields
class PostBase(SQLModel):
    title: str = Field(max_length=255)
    content: str
    published: bool = Field(default=False)

# Database model
class Post(PostBase, table=True):
    __tablename__ = "posts"
    
    id: uuid.UUID = Field(default_factory=uuid.uuid4, primary_key=True)
    owner_id: uuid.UUID = Field(foreign_key="auth.users.id", nullable=False)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

# Request models
class PostCreate(PostBase):
    pass

class PostUpdate(SQLModel):
    title: Optional[str] = None
    content: Optional[str] = None
    published: Optional[bool] = None
```
### Get Single Post

```python
@router.get("/{post_id}", response_model=Post)
async def get_post(
    post_id: uuid.UUID,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
) -> Post:
    """
    Get a specific post by ID.
    
    - **post_id**: UUID of the post to retrieve
    - **Returns**: The requested post
    - **Security**: Only returns posts owned by current user (RLS)
    """
    statement = select(Post).where(
        Post.id == post_id,
        Post.owner_id == current_user  # RLS enforcement
    )
    post = db.exec(statement).first()
    
    if not post:
        raise HTTPException(
            status_code=404, 
            detail="Post not found or access denied"
        )
    
    return post
```
### Update a Post

```python
@router.put("/{post_id}", response_model=Post)
async def update_post(
    post_id: uuid.UUID,
    post_update: PostUpdate,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
) -> Post:
    """
    Update an existing post.
    
    - **post_id**: UUID of the post to update
    - **post_update**: Fields to update (partial update supported)
    - **Returns**: The updated post
    - **Security**: Only allows updating own posts
    """
    # Find existing post with ownership check
    statement = select(Post).where(
        Post.id == post_id,
        Post.owner_id == current_user
    )
    post = db.exec(statement).first()
    
    if not post:
        raise HTTPException(
            status_code=404,
            detail="Post not found or access denied"
        )
    
    # Update only provided fields
    update_data = post_update.dict(exclude_unset=True)
    for field, value in update_data.items():
        setattr(post, field, value)
    
    post.updated_at = datetime.utcnow()
    
    db.add(post)
    db.commit()
    db.refresh(post)
    
    return post
```
### Delete a Post

```python
from fastapi import status

@router.delete("/{post_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_post(
    post_id: uuid.UUID,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
) -> None:
    """
    Delete a post.
    
    - **post_id**: UUID of the post to delete
    - **Returns**: No content (204)
    - **Security**: Only allows deleting own posts
    """
    # Find and verify ownership
    statement = select(Post).where(
        Post.id == post_id,
        Post.owner_id == current_user
    )
    post = db.exec(statement).first()
    
    if not post:
        raise HTTPException(
            status_code=404,
            detail="Post not found or access denied"
        )
    
    # Delete the post
    db.delete(post)
    db.commit()
    
    # Return 204 No Content
    return None
```

## Complete Router Example

**File**: `api-main/app/routes/posts/router.py`

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlmodel import Session, select
from typing import List
from datetime import datetime
import uuid

from app.core.database import get_db
from app.core.auth import get_current_user
from app.models.posts import Post, PostCreate, PostUpdate

router = APIRouter(
    prefix="/posts",
    tags=["Posts"],
    responses={404: {"description": "Not found"}}
)

# All endpoints automatically include authentication and RLS
# through the get_current_user dependency

@router.get("/", response_model=List[Post])
async def get_posts(...)  # Implementation above

@router.post("/", response_model=Post)
async def create_post(...)  # Implementation above

@router.get("/{post_id}", response_model=Post)
async def get_post(...)  # Implementation above

@router.put("/{post_id}", response_model=Post)
async def update_post(...)  # Implementation above

@router.delete("/{post_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_post(...)  # Implementation above
```

## Key Advantages

- **Type Safety**: Full type checking from request to response
- **Auto Documentation**: Swagger/ReDoc generated automatically
- **Security**: Authentication and RLS built-in
- **Validation**: Pydantic models validate all data
- **Performance**: Async/await for high concurrency
- **Developer Experience**: IDE autocomplete and error detection
Previous

Overview

Next

Protect API endpoints

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




