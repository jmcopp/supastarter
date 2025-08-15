# Protect API Endpoints

Learn how to protect FastAPI endpoints using supastarter's hybrid authentication system.

## Authentication System Overview

Supastarter uses a **hybrid authentication system** with:
- **Supabase Auth** for JWT token management
- **FastAPI Dependencies** for endpoint protection
- **Context isolation** with per-request user sessions
- **Row Level Security** for automatic data filtering

## Authentication Dependency

The `get_current_user` dependency handles all authentication:

**File**: `api-main/app/core/auth.py`

```python
from fastapi import Depends, HTTPException, Header
from typing import Dict, Any, Optional
import jwt
import contextvars
import uuid

# Context variables for request isolation
current_user_context: contextvars.ContextVar[Optional[Dict[str, Any]]] = \
    contextvars.ContextVar('current_user_context', default=None)

request_id_context: contextvars.ContextVar[Optional[str]] = \
    contextvars.ContextVar('request_id_context', default=None)

class HybridAuthBackend:
    def __init__(self):
        self.supabase_url = settings.SUPABASE_URL
        self.jwt_secret = settings.SUPABASE_JWT_SECRET
        
    async def verify_jwt_token(self, authorization: str = Header(None)) -> Dict[str, Any]:
        """Extract and verify JWT token with complete isolation"""
        if not authorization or not authorization.startswith("Bearer "):
            raise HTTPException(401, "Missing or invalid authorization header")
        
        token = authorization[7:]
        
        try:
            # Decode JWT
            payload = jwt.decode(
                token, 
                self.jwt_secret, 
                algorithms=["HS256"],
                audience="authenticated"
            )
            
            if "sub" not in payload:
                raise HTTPException(401, "Invalid token: missing user ID")
            
            # Generate request ID for tracing
            request_id = str(uuid.uuid4())
            request_id_context.set(request_id)
            
            # Set user context
            user_context = {
                "user_id": payload["sub"],
                "email": payload.get("email"),
                "role": payload.get("role", "authenticated"),
                "token": token,
                "request_id": request_id
            }
            
            current_user_context.set(user_context)
            return user_context
            
        except jwt.ExpiredSignatureError:
            raise HTTPException(401, "Token expired")
        except jwt.InvalidTokenError as e:
            raise HTTPException(401, f"Invalid token: {str(e)}")

# Global auth backend instance
auth_backend = HybridAuthBackend()

async def get_current_user(
    user_data: Dict[str, Any] = Depends(auth_backend.verify_jwt_token)
) -> str:
    """Get current user ID with isolation"""
    return user_data["user_id"]

async def get_user_context(
    user_data: Dict[str, Any] = Depends(auth_backend.verify_jwt_token)
) -> Dict[str, Any]:
    """Get full user context for request"""
    return user_data
```

## Protect Endpoints

### Basic Authentication

Use the `get_current_user` dependency to require authentication:

```python
from fastapi import APIRouter, Depends
from app.core.auth import get_current_user

router = APIRouter(prefix="/posts", tags=["Posts"])

@router.get("/")
async def get_posts(
    current_user: str = Depends(get_current_user),  # Requires auth
    db: Session = Depends(get_db)
):
    """Get all posts - requires authentication"""
    # current_user contains the user ID from JWT
    posts = await get_user_posts(current_user, db)
    return posts
```

### Access User Information

Get detailed user context in your endpoints:

```python
from app.core.auth import get_user_context

@router.post("/")
async def create_post(
    post_data: PostCreate,
    user_context: Dict[str, Any] = Depends(get_user_context),
    db: Session = Depends(get_db)
):
    """Create a post with full user context"""
    # Access user information
    user_id = user_context["user_id"]
    user_email = user_context["email"]
    user_role = user_context["role"]
    request_id = user_context["request_id"]
    
    # Create post with automatic ownership
    post = Post(
        title=post_data.title,
        content=post_data.content,
        owner_id=user_id
    )
    
    db.add(post)
    db.commit()
    
    return post
```
## Role-Based Access Control

### Admin-Only Endpoints

Create role-based dependencies for admin access:

```python
from app.core.auth import get_user_context

async def require_admin(
    user_context: Dict[str, Any] = Depends(get_user_context)
) -> Dict[str, Any]:
    """Require admin role for endpoint access"""
    user_role = user_context.get("role")
    
    if user_role != "admin":
        raise HTTPException(
            status_code=403,
            detail="Admin access required"
        )
    
    return user_context

# Admin router
admin_router = APIRouter(
    prefix="/admin",
    tags=["Admin"],
    dependencies=[Depends(require_admin)]  # Apply to all endpoints
)

@admin_router.get("/users")
async def get_all_users(
    db: Session = Depends(get_db)
):
    """Admin-only: Get all users"""
    users = db.exec(select(User)).all()
    return users

@admin_router.delete("/users/{user_id}")
async def delete_user(
    user_id: uuid.UUID,
    db: Session = Depends(get_db)
):
    """Admin-only: Delete any user"""
    user = db.get(User, user_id)
    if not user:
        raise HTTPException(404, "User not found")
    
    db.delete(user)
    db.commit()
    
    return {"message": "User deleted"}
```

### Custom Role Dependencies

Create flexible role-based access:

```python
from typing import List

def require_roles(allowed_roles: List[str]):
    """Create a dependency that requires specific roles"""
    async def role_checker(
        user_context: Dict[str, Any] = Depends(get_user_context)
    ) -> Dict[str, Any]:
        user_role = user_context.get("role")
        
        if user_role not in allowed_roles:
            raise HTTPException(
                status_code=403,
                detail=f"Access denied. Required roles: {allowed_roles}"
            )
        
        return user_context
    
    return role_checker

# Usage examples
@router.get("/premium-content")
async def get_premium_content(
    user_context: Dict[str, Any] = Depends(require_roles(["premium", "admin"]))
):
    """Requires premium or admin role"""
    return {"content": "Premium content here"}

@router.post("/moderate")
async def moderate_content(
    user_context: Dict[str, Any] = Depends(require_roles(["moderator", "admin"]))
):
    """Requires moderator or admin role"""
    return {"message": "Content moderated"}
```

## Row Level Security Integration

### Automatic Data Filtering

RLS policies automatically filter data based on user context:

```python
@router.get("/")
async def get_posts(
    current_user: str = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Get posts - RLS automatically filters by owner_id"""
    # This query automatically filters by owner_id due to RLS
    statement = select(Post).where(Post.owner_id == current_user)
    posts = db.exec(statement).all()
    return posts
```

### Supabase Client with RLS

For direct Supabase client usage:

```python
from app.core.auth import get_user_supabase_client

@router.get("/realtime-posts")
async def get_realtime_posts(
    supabase: Client = Depends(get_user_supabase_client)
):
    """Get posts using Supabase client with RLS context"""
    result = supabase.table('posts').select('*').execute()
    return result.data
```

## Security Middleware

### Rate Limiting

```python
from collections import defaultdict
import time

class RateLimitMiddleware:
    def __init__(self, requests_per_minute: int = 100):
        self.rate_limiter = defaultdict(list)
        self.requests_per_minute = requests_per_minute
    
    async def check_rate_limit(self, user_id: str) -> bool:
        now = time.time()
        minute_ago = now - 60
        
        # Clean old requests
        self.rate_limiter[user_id] = [
            req_time for req_time in self.rate_limiter[user_id]
            if req_time > minute_ago
        ]
        
        # Check limit
        if len(self.rate_limiter[user_id]) >= self.requests_per_minute:
            return False
        
        # Add current request
        self.rate_limiter[user_id].append(now)
        return True

# Rate limiting dependency
rate_limiter = RateLimitMiddleware()

async def check_rate_limit(
    current_user: str = Depends(get_current_user)
):
    """Check rate limit for current user"""
    if not await rate_limiter.check_rate_limit(current_user):
        raise HTTPException(
            status_code=429,
            detail="Rate limit exceeded. Try again later."
        )
    return current_user

# Apply to endpoints
@router.post("/", dependencies=[Depends(check_rate_limit)])
async def create_post(...):
    """Rate-limited endpoint"""
    pass
```

## Security Headers

### Automatic Security Headers

```python
# api-main/app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from starlette.middleware.base import BaseHTTPMiddleware

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        
        # Security headers
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Strict-Transport-Security"] = "max-age=31536000"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        
        return response

app = FastAPI()
app.add_middleware(SecurityHeadersMiddleware)
```

## Testing Protected Endpoints

### Authentication in Tests

```python
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_protected_endpoint():
    # Test without authentication
    response = client.get("/api/posts/")
    assert response.status_code == 401
    
    # Test with valid JWT
    headers = {"Authorization": "Bearer valid-jwt-token"}
    response = client.get("/api/posts/", headers=headers)
    assert response.status_code == 200

def test_admin_endpoint():
    # Test with user token
    user_headers = {"Authorization": "Bearer user-jwt-token"}
    response = client.get("/api/admin/users", headers=user_headers)
    assert response.status_code == 403
    
    # Test with admin token
    admin_headers = {"Authorization": "Bearer admin-jwt-token"}
    response = client.get("/api/admin/users", headers=admin_headers)
    assert response.status_code == 200
```

## Key Security Features

- **JWT Token Validation**: Every request validates Supabase JWT tokens
- **Context Isolation**: Each request gets isolated user context
- **Request Tracing**: Unique request IDs for debugging
- **Rate Limiting**: Per-user request limiting
- **Role-Based Access**: Flexible role and permission system
- **Row Level Security**: Automatic data filtering
- **Security Headers**: Comprehensive security headers
- **CORS Protection**: Configurable cross-origin policies
