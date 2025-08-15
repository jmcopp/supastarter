# Use Database Client

Learn how to use SQLModel and Supabase in supastarter's multi-container architecture.

## Overview

Supastarter uses **SQLModel** (built on SQLAlchemy) for type-safe database operations with:
- **Automatic type generation** from model definitions
- **Row Level Security (RLS)** integration with Supabase
- **Context-isolated sessions** for multi-user security
- **Async/await support** for high performance
- **Direct Supabase client** for real-time features

## Architecture

```
┌────────────────────────────────────────────────────────────┐
│                    Database Access Layer                    │
├────────────────────────────────────────────────────────────┤
│                                                              │
│  FastAPI Routes    ←──→   SQLModel     ←──→   Supabase       │
│  ┌─────────────┐         ┌────────┐         ┌──────────┐ │
│  │ get_current_  │         │ Models │         │ PostgreSQL │ │
│  │ user()        │         │ + RLS  │         │ + RLS      │ │
│  │             │         │        │         │ Policies   │ │
│  │ Session     │         │ Async  │         │            │ │
│  │ Management  │         │ Queries│         │ Real-time  │ │
│  └─────────────┘         └────────┘         └──────────┘ │
└────────────────────────────────────────────────────────────┘
```

**Best Practice**: Keep database operations in dedicated service/repository layers (`api-main/app/services/`) rather than directly in route handlers.

## SQLModel Database Operations

### Database Session Management

**File**: `api-main/app/core/database.py`

```python
from sqlmodel import SQLModel, create_engine, Session
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from app.core.config import settings
from typing import AsyncGenerator

# Create engine
engine = create_engine(
    settings.DATABASE_URL,
    pool_pre_ping=True,
    pool_recycle=300,
    echo=settings.DEBUG  # SQL logging in development
)

# For async operations (optional)
async_engine = create_async_engine(
    settings.DATABASE_URL.replace("postgresql://", "postgresql+asyncpg://"),
    pool_pre_ping=True,
    pool_recycle=300,
    echo=settings.DEBUG
)

# Dependency for route handlers
def get_db() -> Session:
    """Get database session for sync operations"""
    with Session(engine) as session:
        yield session

async def get_async_db() -> AsyncGenerator[AsyncSession, None]:
    """Get async database session"""
    async with AsyncSession(async_engine) as session:
        yield session

# Create tables (run once during startup)
def create_db_and_tables():
    SQLModel.metadata.create_all(engine)
```

### Querying Records

#### Basic Queries with RLS

```python
from sqlmodel import Session, select
from app.models.posts import Post
from app.core.auth import get_current_user
from fastapi import Depends

@router.get("/posts")
async def get_posts(
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Get all posts for current user - RLS applied automatically"""
    statement = select(Post).where(Post.owner_id == current_user)
    posts = db.exec(statement).all()
    return posts

# With pagination and filtering
@router.get("/posts/search")
async def search_posts(
    query: str = None,
    skip: int = 0,
    limit: int = 10,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    statement = select(Post).where(Post.owner_id == current_user)
    
    if query:
        statement = statement.where(Post.title.contains(query))
    
    statement = statement.offset(skip).limit(limit).order_by(Post.created_at.desc())
    
    posts = db.exec(statement).all()
    return posts
```

#### Complex Queries with Relationships

```python
# Query with joins and relationships
from sqlmodel import select
from app.models.users import User
from app.models.posts import Post

@router.get("/users/{user_id}/posts")
async def get_user_posts(
    user_id: str,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Get posts by specific user (admin only or own posts)"""
    
    # Admin can see any user's posts
    if current_user != user_id:
        # Check if current user is admin
        current_user_obj = db.get(User, current_user)
        if not current_user_obj or current_user_obj.role != "admin":
            raise HTTPException(403, "Access denied")
    
    statement = (
        select(Post, User)
        .join(User, Post.owner_id == User.id)
        .where(Post.owner_id == user_id)
        .order_by(Post.created_at.desc())
    )
    
    results = db.exec(statement).all()
    
    return [
        {
            "post": post,
            "author": user
        }
        for post, user in results
    ]
```
### Creating Records

#### Basic Record Creation

```python
from app.models.posts import Post, PostCreate
from datetime import datetime
import uuid

@router.post("/posts", response_model=Post)
async def create_post(
    post_data: PostCreate,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Create a new post with automatic owner assignment"""
    
    # Create post instance
    post = Post(
        id=uuid.uuid4(),
        title=post_data.title,
        content=post_data.content,
        published=post_data.published,
        owner_id=current_user,  # Automatic from JWT
        created_at=datetime.utcnow(),
        updated_at=datetime.utcnow()
    )
    
    # Save to database
    db.add(post)
    db.commit()
    db.refresh(post)  # Get updated object with DB-generated fields
    
    return post
```

#### Batch Creation

```python
@router.post("/posts/batch")
async def create_posts_batch(
    posts_data: List[PostCreate],
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Create multiple posts in a single transaction"""
    
    posts = []
    for post_data in posts_data:
        post = Post(
            id=uuid.uuid4(),
            **post_data.dict(),
            owner_id=current_user,
            created_at=datetime.utcnow(),
            updated_at=datetime.utcnow()
        )
        posts.append(post)
        db.add(post)
    
    db.commit()
    
    # Refresh all posts
    for post in posts:
        db.refresh(post)
    
    return posts
```

#### Transaction Management

```python
from sqlmodel import Session
from contextlib import contextmanager

@contextmanager
def db_transaction(db: Session):
    """Context manager for database transactions"""
    try:
        yield db
        db.commit()
    except Exception as e:
        db.rollback()
        raise e

@router.post("/posts/with-category")
async def create_post_with_category(
    post_data: PostCreate,
    category_name: str,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Create post and category in a single transaction"""
    
    with db_transaction(db):
        # Create category if not exists
        category = db.exec(
            select(Category).where(Category.name == category_name)
        ).first()
        
        if not category:
            category = Category(
                id=uuid.uuid4(),
                name=category_name,
                owner_id=current_user
            )
            db.add(category)
            db.flush()  # Get the ID without committing
        
        # Create post
        post = Post(
            **post_data.dict(),
            owner_id=current_user,
            category_id=category.id
        )
        db.add(post)
    
    db.refresh(post)
    return post
```
### Updating Records

#### Basic Updates with RLS

```python
@router.put("/posts/{post_id}", response_model=Post)
async def update_post(
    post_id: uuid.UUID,
    post_update: PostUpdate,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Update post with ownership verification"""
    
    # Find existing post with ownership check
    statement = select(Post).where(
        Post.id == post_id,
        Post.owner_id == current_user  # RLS enforcement
    )
    post = db.exec(statement).first()
    
    if not post:
        raise HTTPException(404, "Post not found or access denied")
    
    # Update only provided fields (partial update)
    update_data = post_update.dict(exclude_unset=True)
    
    for field, value in update_data.items():
        if hasattr(post, field):
            setattr(post, field, value)
    
    # Always update timestamp
    post.updated_at = datetime.utcnow()
    
    db.add(post)
    db.commit()
    db.refresh(post)
    
    return post
```

#### Conditional Updates

```python
@router.patch("/posts/{post_id}/publish")
async def publish_post(
    post_id: uuid.UUID,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Publish a post if it's ready"""
    
    statement = select(Post).where(
        Post.id == post_id,
        Post.owner_id == current_user
    )
    post = db.exec(statement).first()
    
    if not post:
        raise HTTPException(404, "Post not found")
    
    # Conditional logic
    if not post.title or not post.content:
        raise HTTPException(400, "Post must have title and content to publish")
    
    if post.published:
        raise HTTPException(400, "Post is already published")
    
    # Update
    post.published = True
    post.published_at = datetime.utcnow()
    post.updated_at = datetime.utcnow()
    
    db.add(post)
    db.commit()
    db.refresh(post)
    
    return {"message": "Post published successfully", "post": post}
```

#### Bulk Updates

```python
from sqlalchemy import update

@router.patch("/posts/bulk-publish")
async def bulk_publish_posts(
    post_ids: List[uuid.UUID],
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Publish multiple posts at once"""
    
    # Verify ownership first
    existing_posts = db.exec(
        select(Post).where(
            Post.id.in_(post_ids),
            Post.owner_id == current_user
        )
    ).all()
    
    if len(existing_posts) != len(post_ids):
        raise HTTPException(400, "Some posts not found or access denied")
    
    # Bulk update using SQLAlchemy core
    stmt = (
        update(Post)
        .where(
            Post.id.in_(post_ids),
            Post.owner_id == current_user
        )
        .values(
            published=True,
            published_at=datetime.utcnow(),
            updated_at=datetime.utcnow()
        )
    )
    
    result = db.exec(stmt)
    db.commit()
    
    return {
        "message": f"Published {result.rowcount} posts",
        "updated_count": result.rowcount
    }
```
### Deleting Records

#### Safe Deletion with RLS

```python
from fastapi import status

@router.delete("/posts/{post_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_post(
    post_id: uuid.UUID,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Delete post with ownership verification"""
    
    # Find and verify ownership
    statement = select(Post).where(
        Post.id == post_id,
        Post.owner_id == current_user
    )
    post = db.exec(statement).first()
    
    if not post:
        raise HTTPException(404, "Post not found or access denied")
    
    # Delete the post
    db.delete(post)
    db.commit()
    
    return None  # 204 No Content
```

#### Soft Deletion

```python
# Add to your model
class Post(PostBase, table=True):
    # ... other fields
    deleted_at: Optional[datetime] = Field(default=None)
    deleted_by: Optional[str] = Field(default=None)

@router.delete("/posts/{post_id}/soft")
async def soft_delete_post(
    post_id: uuid.UUID,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Soft delete - mark as deleted without removing from database"""
    
    statement = select(Post).where(
        Post.id == post_id,
        Post.owner_id == current_user,
        Post.deleted_at.is_(None)  # Not already deleted
    )
    post = db.exec(statement).first()
    
    if not post:
        raise HTTPException(404, "Post not found or already deleted")
    
    # Mark as deleted
    post.deleted_at = datetime.utcnow()
    post.deleted_by = current_user
    
    db.add(post)
    db.commit()
    
    return {"message": "Post deleted successfully"}

# Update queries to exclude soft-deleted records
@router.get("/posts")
async def get_posts(
    include_deleted: bool = False,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    statement = select(Post).where(Post.owner_id == current_user)
    
    if not include_deleted:
        statement = statement.where(Post.deleted_at.is_(None))
    
    posts = db.exec(statement).all()
    return posts
```

#### Cascade Deletion

```python
@router.delete("/categories/{category_id}")
async def delete_category(
    category_id: uuid.UUID,
    force: bool = False,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Delete category and handle related posts"""
    
    # Check if category exists and user owns it
    category = db.exec(
        select(Category).where(
            Category.id == category_id,
            Category.owner_id == current_user
        )
    ).first()
    
    if not category:
        raise HTTPException(404, "Category not found")
    
    # Check for related posts
    related_posts = db.exec(
        select(Post).where(Post.category_id == category_id)
    ).all()
    
    if related_posts and not force:
        raise HTTPException(
            400, 
            f"Category has {len(related_posts)} related posts. Use force=true to delete anyway."
        )
    
    # Handle related posts
    if force:
        # Option 1: Delete all related posts
        for post in related_posts:
            db.delete(post)
        
        # Option 2: Move to "Uncategorized"
        # uncategorized = get_or_create_uncategorized_category(db, current_user)
        # for post in related_posts:
        #     post.category_id = uncategorized.id
        #     db.add(post)
    
    # Delete the category
    db.delete(category)
    db.commit()
    
    return {
        "message": "Category deleted",
        "deleted_posts": len(related_posts) if force else 0
    }
```

## Supabase Client Integration

### Real-time Subscriptions

For real-time features, use the Supabase client alongside SQLModel:

```python
from supabase import create_client
from app.core.auth import get_user_supabase_client

@router.get("/posts/realtime")
async def get_posts_realtime(
    supabase: Client = Depends(get_user_supabase_client)
):
    """Get posts with real-time subscription setup"""
    
    # Query with RLS automatically applied
    result = supabase.table('posts').select('*').execute()
    
    # Real-time subscription (handled on frontend)
    # supabase.channel('posts').on('postgres_changes', ...)
    
    return result.data
```

### Direct SQL Queries

For complex queries, use raw SQL with parameter binding:

```python
from sqlalchemy import text

@router.get("/posts/analytics")
async def get_post_analytics(
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    """Complex analytics query with raw SQL"""
    
    query = text("""
        SELECT 
            DATE_TRUNC('month', created_at) as month,
            COUNT(*) as post_count,
            COUNT(CASE WHEN published = true THEN 1 END) as published_count
        FROM posts 
        WHERE owner_id = :user_id 
        GROUP BY DATE_TRUNC('month', created_at)
        ORDER BY month DESC
        LIMIT 12
    """)
    
    result = db.exec(query, {"user_id": current_user})
    
    return [
        {
            "month": row.month.isoformat(),
            "total_posts": row.post_count,
            "published_posts": row.published_count
        }
        for row in result
    ]
```

## Error Handling and Best Practices

### Database Error Handling

```python
from sqlalchemy.exc import IntegrityError, DataError
from fastapi import HTTPException
import logging

logger = logging.getLogger(__name__)

@router.post("/posts")
async def create_post_with_error_handling(
    post_data: PostCreate,
    db: Session = Depends(get_db),
    current_user: str = Depends(get_current_user)
):
    try:
        post = Post(**post_data.dict(), owner_id=current_user)
        db.add(post)
        db.commit()
        db.refresh(post)
        return post
        
    except IntegrityError as e:
        db.rollback()
        logger.error(f"Database integrity error: {e}")
        
        if "unique constraint" in str(e).lower():
            raise HTTPException(400, "A post with this title already exists")
        else:
            raise HTTPException(400, "Database constraint violation")
            
    except DataError as e:
        db.rollback()
        logger.error(f"Database data error: {e}")
        raise HTTPException(400, "Invalid data format")
        
    except Exception as e:
        db.rollback()
        logger.error(f"Unexpected database error: {e}")
        raise HTTPException(500, "Internal server error")
```

### Connection Pool Management

```python
# api-main/app/core/database.py
from sqlmodel import create_engine

engine = create_engine(
    settings.DATABASE_URL,
    # Connection pool settings
    pool_size=20,          # Number of connections to maintain
    max_overflow=0,        # Additional connections when pool is full
    pool_pre_ping=True,    # Validate connections before use
    pool_recycle=3600,     # Recreate connections every hour
    echo=False,            # Set to True for SQL logging
    connect_args={
        "connect_timeout": 10,
        "server_settings": {
            "application_name": "supastarter-api",
        },
    },
)
```

This SQLModel approach provides type safety, automatic RLS enforcement, and seamless integration with Supabase while maintaining high performance through proper session management and error handling.
