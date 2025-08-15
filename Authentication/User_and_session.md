# User and Session Management

Learn how to access user and session data in supastarter's hybrid authentication system.

## Frontend Session Management

### Supabase Auth Hook

Supastarter uses Supabase Auth for session management with custom hooks:

**File**: `frontend/lib/hooks/useAuth.ts`

```typescript
import { useEffect, useState } from 'react'
import { User, Session } from '@supabase/supabase-js'
import { supabase } from '@/lib/supabase'

interface AuthState {
  user: User | null
  session: Session | null
  loading: boolean
}

export function useAuth(): AuthState {
  const [user, setUser] = useState<User | null>(null)
  const [session, setSession] = useState<Session | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    // Get initial session
    const getInitialSession = async () => {
      const { data: { session }, error } = await supabase.auth.getSession()
      
      if (error) {
        console.error('Error getting session:', error)
      } else {
        setSession(session)
        setUser(session?.user || null)
      }
      
      setLoading(false)
    }

    getInitialSession()

    // Listen for auth changes
    const {
      data: { subscription },
    } = supabase.auth.onAuthStateChange(async (event, session) => {
      setSession(session)
      setUser(session?.user || null)
      setLoading(false)
    })

    return () => subscription.unsubscribe()
  }, [])

  return { user, session, loading }
}
```

### Using Authentication in Components

```typescript
import { useAuth } from '@/lib/hooks/useAuth'

export function UserProfile() {
  const { user, session, loading } = useAuth()

  if (loading) {
    return <div>Loading...</div>
  }

  if (!user) {
    return <div>Please sign in</div>
  }

  return (
    <div>
      <h1>Welcome, {user.email}!</h1>
      <p>User ID: {user.id}</p>
      <p>Email verified: {user.email_confirmed_at ? 'Yes' : 'No'}</p>
      <p>Last sign in: {user.last_sign_in_at}</p>
    </div>
  )
}
```

## User and Session Types

### Supabase User Type

```typescript
// Extended from Supabase User type
interface User {
  id: string                    // UUID from auth.users
  email?: string
  email_confirmed_at?: string
  phone?: string
  phone_confirmed_at?: string
  last_sign_in_at?: string
  created_at: string
  updated_at: string
  
  // Custom user metadata
  user_metadata: {
    name?: string
    avatar_url?: string
    role?: 'admin' | 'user' | 'premium'
    onboarding_complete?: boolean
  }
  
  // App-specific metadata
  app_metadata: {
    provider?: string
    providers?: string[]
  }
}

interface Session {
  access_token: string          // JWT token for API calls
  refresh_token: string         // For token refresh
  expires_in: number           // Token expiry time
  expires_at?: number          // Unix timestamp
  token_type: 'bearer'
  user: User
}
```
## Loading States and Protection

### Wait for Session Loading

```typescript
export function ProtectedPage() {
  const { user, session, loading } = useAuth()

  // Show loading state
  if (loading) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600" />
        <span className="ml-2">Loading...</span>
      </div>
    )
  }

  // Redirect to login if not authenticated
  if (!user) {
    redirect('/login')
    return null
  }

  // Render protected content
  return (
    <div>
      <h1>Protected Content</h1>
      <p>Welcome, {user.email}!</p>
    </div>
  )
}
```

### Route Protection with Middleware

**File**: `frontend/middleware.ts`

```typescript
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export async function middleware(req: NextRequest) {
  const res = NextResponse.next()
  const supabase = createMiddlewareClient({ req, res })
  
  const {
    data: { session },
  } = await supabase.auth.getSession()
  
  // Protect /app routes
  if (req.nextUrl.pathname.startsWith('/app')) {
    if (!session) {
      const redirectUrl = new URL('/login', req.url)
      redirectUrl.searchParams.set('redirect', req.nextUrl.pathname)
      return NextResponse.redirect(redirectUrl)
    }
  }
  
  // Redirect authenticated users away from auth pages
  if (session && req.nextUrl.pathname.startsWith('/login')) {
    return NextResponse.redirect(new URL('/app', req.url))
  }
  
  return res
}

export const config = {
  matcher: ['/app/:path*', '/login', '/signup']
}
```
## Session Management Actions

### Refresh Session

```typescript
// Enhanced auth hook with actions
export function useAuth() {
  // ... previous state code ...

  const refreshSession = async () => {
    setLoading(true)
    const { data: { session }, error } = await supabase.auth.refreshSession()
    
    if (error) {
      console.error('Error refreshing session:', error)
    } else {
      setSession(session)
      setUser(session?.user || null)
    }
    
    setLoading(false)
  }

  const signOut = async () => {
    setLoading(true)
    await supabase.auth.signOut()
    setSession(null)
    setUser(null)
    setLoading(false)
  }

  const updateProfile = async (updates: {
    email?: string
    password?: string
    data?: object
  }) => {
    const { data, error } = await supabase.auth.updateUser(updates)
    
    if (error) {
      throw error
    }
    
    // Refresh session to get updated user data
    await refreshSession()
    
    return data
  }

  return {
    user,
    session,
    loading,
    refreshSession,
    signOut,
    updateProfile
  }
}
```

### Update User Profile

```typescript
export function ProfileForm() {
  const { user, updateProfile } = useAuth()
  const [loading, setLoading] = useState(false)

  const handleSubmit = async (formData: FormData) => {
    setLoading(true)
    
    try {
      await updateProfile({
        data: {
          name: formData.get('name'),
          avatar_url: formData.get('avatar_url')
        }
      })
      
      toast.success('Profile updated successfully')
    } catch (error) {
      toast.error('Failed to update profile')
    } finally {
      setLoading(false)
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input 
        name="name" 
        defaultValue={user?.user_metadata?.name || ''}
        placeholder="Full name"
      />
      <input 
        name="avatar_url" 
        defaultValue={user?.user_metadata?.avatar_url || ''}
        placeholder="Avatar URL"
      />
      <button type="submit" disabled={loading}>
        {loading ? 'Updating...' : 'Update Profile'}
      </button>
    </form>
  )
}
```
## Server-Side Session Access

### Next.js Server Components

Access session data in Server Components:

**File**: `frontend/lib/auth/server.ts`

```typescript
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { redirect } from 'next/navigation'

export async function getSession() {
  const supabase = createServerComponentClient({ cookies })
  
  const {
    data: { session },
    error
  } = await supabase.auth.getSession()
  
  if (error) {
    console.error('Error getting session:', error)
    return null
  }
  
  return session
}

export async function requireAuth() {
  const session = await getSession()
  
  if (!session) {
    redirect('/login')
  }
  
  return session
}

export async function requireRole(allowedRoles: string[]) {
  const session = await requireAuth()
  const userRole = session.user.user_metadata?.role || 'user'
  
  if (!allowedRoles.includes(userRole)) {
    redirect('/unauthorized')
  }
  
  return session
}
```

### Using in Server Components

```typescript
import { getSession, requireAuth } from '@/lib/auth/server'

// Optional authentication
export default async function HomePage() {
  const session = await getSession()
  
  if (session) {
    return <div>Welcome back, {session.user.email}!</div>
  }
  
  return <div>Welcome! Please sign in.</div>
}

// Required authentication
export default async function DashboardPage() {
  const session = await requireAuth()
  
  return (
    <div>
      <h1>Dashboard</h1>
      <p>User ID: {session.user.id}</p>
      <p>Email: {session.user.email}</p>
    </div>
  )
}

// Role-based access
export default async function AdminPage() {
  const session = await requireRole(['admin'])
  
  return (
    <div>
      <h1>Admin Dashboard</h1>
      <p>Welcome, admin {session.user.email}</p>
    </div>
  )
}
```

### API Route Protection

```typescript
// frontend/app/api/profile/route.ts
import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const supabase = createRouteHandlerClient({ cookies })
  
  const {
    data: { session },
    error
  } = await supabase.auth.getSession()
  
  if (error || !session) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    )
  }
  
  // Fetch user profile data
  const { data: profile } = await supabase
    .from('profiles')
    .select('*')
    .eq('id', session.user.id)
    .single()
  
  return NextResponse.json({ 
    user: session.user,
    profile 
  })
}
```

## Backend Session Validation

### FastAPI Integration

The FastAPI backend validates JWT tokens from the frontend:

```python
# api-main/app/core/auth.py (referenced from Protect API endpoints)
async def get_current_user(
    authorization: str = Header(None)
) -> str:
    """Validate Supabase JWT and return user ID"""
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(401, "Missing authorization header")
    
    token = authorization[7:]
    
    try:
        # Verify JWT with Supabase secret
        payload = jwt.decode(
            token,
            settings.SUPABASE_JWT_SECRET,
            algorithms=["HS256"],
            audience="authenticated"
        )
        
        return payload["sub"]  # User ID
        
    except jwt.InvalidTokenError:
        raise HTTPException(401, "Invalid token")
```

## User Metadata Management

### Custom User Fields

Store additional user data in `user_metadata`:

```typescript
// Update user metadata
const updateUserMetadata = async (metadata: object) => {
  const { data, error } = await supabase.auth.updateUser({
    data: {
      ...metadata,
      updated_at: new Date().toISOString()
    }
  })
  
  if (error) throw error
  return data
}

// Usage
await updateUserMetadata({
  name: 'John Doe',
  role: 'premium',
  onboarding_complete: true,
  preferences: {
    theme: 'dark',
    notifications: true
  }
})
```

## Session Security

### Token Management

- **JWT Tokens**: Supabase handles secure JWT generation and validation
- **Automatic Refresh**: Tokens are automatically refreshed before expiry
- **Secure Storage**: Tokens stored in httpOnly cookies (SSR) or secure storage
- **Context Isolation**: Backend ensures per-request user context isolation

### Best Practices

1. **Always check loading state** before rendering user data
2. **Use middleware for route protection** instead of component-level checks
3. **Validate sessions server-side** for sensitive operations
4. **Refresh sessions** after profile updates
5. **Handle auth state changes** gracefully
6. **Store minimal data in user_metadata** (use database for complex data)

The session management system provides seamless integration between frontend Supabase Auth and backend FastAPI authentication with full type safety and security.
Previous

Overview

Next

Permissions and access control

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




