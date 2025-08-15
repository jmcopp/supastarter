# Super Admin & Admin UI

Learn how to create and manage admin users in supastarter's multi-container architecture.

## Creating Admin Users

### Method 1: Using Supabase Dashboard

1. Go to your Supabase dashboard
2. Navigate to **Authentication** → **Users**
3. Click **Add user** → **Create new user**
4. Fill in user details
5. After creation, update user metadata:
   - Click on the user
   - Edit **User metadata**
   - Add: `{"role": "admin"}`

### Method 2: Using SQL in Supabase

Run in Supabase SQL Editor:

```sql
-- Create user in auth.users
INSERT INTO auth.users (
  id,
  email,
  encrypted_password,
  email_confirmed_at,
  raw_user_meta_data
) VALUES (
  gen_random_uuid(),
  'admin@example.com',
  crypt('SecurePassword123!', gen_salt('bf')),
  NOW(),
  '{"role": "admin"}'::jsonb
);
```

### Method 3: CLI Script for FastAPI

**File**: `api-main/scripts/create_admin.py`

```python
#!/usr/bin/env python
import asyncio
from supabase import create_client
from app.core.config import settings
import getpass

async def create_admin_user():
    """Create an admin user via Supabase Admin API."""
    
    # Initialize Supabase admin client
    supabase = create_client(
        settings.SUPABASE_URL,
        settings.SUPABASE_SERVICE_KEY
    )
    
    # Get user input
    email = input("Admin email: ")
    password = getpass.getpass("Admin password: ")
    name = input("Admin name: ")
    
    try:
        # Create user with admin role
        response = supabase.auth.admin.create_user({
            "email": email,
            "password": password,
            "email_confirm": True,
            "user_metadata": {
                "name": name,
                "role": "admin"
            }
        })
        
        if response.user:
            print(f"✅ Admin user created successfully!")
            print(f"   Email: {email}")
            print(f"   ID: {response.user.id}")
            print(f"   Role: admin")
        else:
            print("❌ Failed to create admin user")
            
    except Exception as e:
        print(f"❌ Error: {str(e)}")

if __name__ == "__main__":
    asyncio.run(create_admin_user())
```

Run the script:

```bash
cd api-main
python scripts/create_admin.py
```

## Admin API Endpoints

### Protected Admin Routes

**File**: `api-main/app/routes/admin.py`

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlmodel import Session, select
from app.core.auth import get_current_user, require_admin
from app.core.database import get_db
from app.models.user import User
from typing import List

router = APIRouter(
    prefix="/admin",
    tags=["admin"],
    dependencies=[Depends(require_admin)]  # Require admin for all routes
)

@router.get("/users", response_model=List[User])
async def get_all_users(
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db)
):
    """Get all users (admin only)."""
    statement = select(User).offset(skip).limit(limit)
    users = db.exec(statement).all()
    return users

@router.put("/users/{user_id}/role")
async def update_user_role(
    user_id: str,
    role: str,
    db: Session = Depends(get_db)
):
    """Update user role (admin only)."""
    user = db.get(User, user_id)
    
    if not user:
        raise HTTPException(404, "User not found")
    
    # Update role in Supabase
    supabase = create_client(
        settings.SUPABASE_URL,
        settings.SUPABASE_SERVICE_KEY
    )
    
    supabase.auth.admin.update_user_by_id(
        user_id,
        {"user_metadata": {"role": role}}
    )
    
    return {"message": f"User role updated to {role}"}

@router.delete("/users/{user_id}")
async def delete_user(
    user_id: str,
    db: Session = Depends(get_db)
):
    """Delete a user (admin only)."""
    # Delete from Supabase
    supabase = create_client(
        settings.SUPABASE_URL,
        settings.SUPABASE_SERVICE_KEY
    )
    
    supabase.auth.admin.delete_user(user_id)
    
    return {"message": "User deleted"}
```

### Admin Authentication Middleware

**File**: `api-main/app/core/auth.py`

```python
from fastapi import Depends, HTTPException

async def require_admin(
    user_context: Dict[str, Any] = Depends(get_user_context)
) -> Dict[str, Any]:
    """Require admin role for endpoint access."""
    user_role = user_context.get("role")
    
    if user_role != "admin":
        raise HTTPException(
            status_code=403,
            detail="Admin access required"
        )
    
    return user_context
```

## Admin UI Components

### Admin Dashboard Page

**File**: `frontend/app/admin/page.tsx`

```typescript
'use client'

import { useEffect, useState } from 'react'
import { useAuth } from '@/lib/hooks/useAuth'
import { redirect } from 'next/navigation'

export default function AdminDashboard() {
  const { user, session } = useAuth()
  const [users, setUsers] = useState([])
  
  // Check admin role
  useEffect(() => {
    if (user && user.user_metadata?.role !== 'admin') {
      redirect('/dashboard')
    }
  }, [user])
  
  // Fetch admin data
  useEffect(() => {
    async function fetchUsers() {
      const response = await fetch('/api/admin/users', {
        headers: {
          'Authorization': `Bearer ${session?.access_token}`
        }
      })
      
      if (response.ok) {
        const data = await response.json()
        setUsers(data)
      }
    }
    
    if (session) {
      fetchUsers()
    }
  }, [session])
  
  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold mb-6">Admin Dashboard</h1>
      
      <div className="bg-white rounded-lg shadow">
        <div className="px-6 py-4 border-b">
          <h2 className="text-lg font-semibold">Users Management</h2>
        </div>
        
        <div className="p-6">
          <table className="w-full">
            <thead>
              <tr className="border-b">
                <th className="text-left pb-2">Email</th>
                <th className="text-left pb-2">Role</th>
                <th className="text-left pb-2">Created</th>
                <th className="text-left pb-2">Actions</th>
              </tr>
            </thead>
            <tbody>
              {users.map((user) => (
                <tr key={user.id} className="border-b">
                  <td className="py-2">{user.email}</td>
                  <td className="py-2">{user.role}</td>
                  <td className="py-2">{new Date(user.created_at).toLocaleDateString()}</td>
                  <td className="py-2">
                    <button className="text-blue-600 hover:underline mr-2">
                      Edit
                    </button>
                    <button className="text-red-600 hover:underline">
                      Delete
                    </button>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  )
}
```

### Admin Navigation

Add admin menu item for admin users:

**File**: `frontend/components/navigation/NavBar.tsx`

```typescript
import { useAuth } from '@/lib/hooks/useAuth'

export function NavBar() {
  const { user } = useAuth()
  const isAdmin = user?.user_metadata?.role === 'admin'
  
  return (
    <nav>
      {/* Regular menu items */}
      
      {isAdmin && (
        <Link href="/admin" className="flex items-center space-x-2">
          <Shield className="h-4 w-4" />
          <span>Admin</span>
        </Link>
      )}
    </nav>
  )
}
```

## Database Schema for Admin Features

### Admin Activity Logging

**File**: `api-main/app/models/admin_log.py`

```python
from sqlmodel import SQLModel, Field
import uuid
from datetime import datetime

class AdminLog(SQLModel, table=True):
    __tablename__ = "admin_logs"
    
    id: uuid.UUID = Field(default_factory=uuid.uuid4, primary_key=True)
    admin_id: uuid.UUID = Field(foreign_key="auth.users.id")
    action: str  # e.g., "delete_user", "update_role"
    target_id: str  # ID of affected resource
    target_type: str  # e.g., "user", "organization"
    details: dict = Field(default={}, sa_column_kwargs={"type": "jsonb"})
    ip_address: str
    created_at: datetime = Field(default_factory=datetime.utcnow)
```

Enable RLS for admin logs:

```sql
-- In Supabase SQL Editor
ALTER TABLE public.admin_logs ENABLE ROW LEVEL SECURITY;

-- Only admins can view logs
CREATE POLICY "Admins view logs" ON public.admin_logs
FOR SELECT USING (
    EXISTS (
        SELECT 1 FROM auth.users
        WHERE auth.uid() = id
        AND raw_user_meta_data->>'role' = 'admin'
    )
);
```

### Admin Statistics API

**File**: `api-main/app/routes/admin.py` (addition)

```python
@router.get("/stats")
async def get_admin_stats(
    db: Session = Depends(get_db)
):
    """Get system statistics (admin only)."""
    
    # User statistics
    total_users = db.exec(select(func.count(User.id))).one()
    active_users = db.exec(
        select(func.count(User.id))
        .where(User.last_sign_in_at > datetime.now() - timedelta(days=30))
    ).one()
    
    # Organization statistics
    total_orgs = db.exec(select(func.count(Organization.id))).one()
    
    # Subscription statistics
    active_subscriptions = db.exec(
        select(func.count(Purchase.id))
        .where(Purchase.status == "active")
        .where(Purchase.type == "subscription")
    ).one()
    
    return {
        "users": {
            "total": total_users,
            "active_30d": active_users
        },
        "organizations": {
            "total": total_orgs
        },
        "subscriptions": {
            "active": active_subscriptions
        },
        "timestamp": datetime.utcnow().isoformat()
    }
```

## Security Best Practices

- Multi-factor authentication for admin accounts
- Audit logging for all admin actions
- IP whitelisting for admin access (optional)
- Session timeout for admin sessions
- Separate admin subdomain (e.g., admin.yourdomain.com)

### Implement Admin MFA

```python
# api-main/app/core/admin_auth.py
from pyotp import TOTP

def verify_admin_mfa(user_id: str, token: str) -> bool:
    """Verify MFA token for admin users."""
    # Get user's MFA secret from database
    user = get_user(user_id)
    
    if not user.mfa_secret:
        return False
    
    totp = TOTP(user.mfa_secret)
    return totp.verify(token, valid_window=1)
```

## Monitoring Admin Actions

### Real-time Admin Activity

```typescript
// frontend/hooks/useAdminActivity.ts
import { useEffect, useState } from 'react'
import { supabase } from '@/lib/supabase'

export function useAdminActivity() {
  const [activities, setActivities] = useState([])
  
  useEffect(() => {
    // Subscribe to admin log changes
    const subscription = supabase
      .channel('admin_logs')
      .on('postgres_changes', {
        event: 'INSERT',
        schema: 'public',
        table: 'admin_logs'
      }, (payload) => {
        setActivities(prev => [payload.new, ...prev])
      })
      .subscribe()
    
    return () => {
      subscription.unsubscribe()
    }
  }, [])
  
  return activities
}
```

## Troubleshooting

### Common Issues

**Cannot access admin panel**
- Verify user has `role: "admin"` in metadata
- Check JWT token includes role claim
- Ensure admin routes are properly protected

**Admin actions not working**
- Verify SUPABASE_SERVICE_KEY is set (not anon key)
- Check RLS policies allow admin operations
- Confirm admin user ID matches auth.uid()

**Missing admin role after OAuth login**
- Set default role in Supabase Auth hooks
- Implement role assignment workflow
- Use database triggers to set initial roles
