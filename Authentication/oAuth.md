# OAuth Authentication

Learn how to set up OAuth providers with Supabase Auth in supastarter.

## Overview

Supastarter uses **Supabase Auth** for OAuth authentication, providing seamless integration with multiple providers while maintaining Row Level Security.

## Setting Up OAuth Providers

### Step 1: Enable Provider in Supabase

1. Go to your Supabase dashboard
2. Navigate to **Authentication** → **Providers**
3. Select the provider you want to enable (e.g., Google, GitHub, Facebook)
4. Toggle to enable the provider

### Step 2: Configure Provider Credentials

Each provider requires specific credentials:

#### Google OAuth

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project or select existing
3. Enable Google+ API
4. Create OAuth 2.0 credentials:
   - **Authorized redirect URIs**: `https://[PROJECT_REF].supabase.co/auth/v1/callback`
5. Copy Client ID and Client Secret to Supabase

#### GitHub OAuth

1. Go to GitHub Settings → Developer settings → OAuth Apps
2. Create a new OAuth App:
   - **Authorization callback URL**: `https://[PROJECT_REF].supabase.co/auth/v1/callback`
3. Copy Client ID and Client Secret to Supabase

#### Facebook OAuth

1. Go to [Facebook Developers](https://developers.facebook.com)
2. Create a new app
3. Add Facebook Login product
4. Configure OAuth redirect URI
5. Copy App ID and App Secret to Supabase

### Step 3: Configure Frontend

**File**: `frontend/lib/auth.ts`

```typescript
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)

export async function signInWithOAuth(provider: 'google' | 'github' | 'facebook') {
  const { data, error } = await supabase.auth.signInWithOAuth({
    provider,
    options: {
      redirectTo: `${window.location.origin}/auth/callback`,
      scopes: provider === 'github' ? 'read:user user:email' : undefined,
    }
  })
  
  if (error) throw error
  return data
}
```

### Step 4: Create OAuth Login Component

**File**: `frontend/components/auth/OAuthButtons.tsx`

```typescript
import { signInWithOAuth } from '@/lib/auth'
import { Button } from '@/components/ui/button'
import { Github, Chrome, Facebook } from 'lucide-react'

const providers = [
  { name: 'google', icon: Chrome, label: 'Continue with Google' },
  { name: 'github', icon: Github, label: 'Continue with GitHub' },
  { name: 'facebook', icon: Facebook, label: 'Continue with Facebook' },
] as const

export function OAuthButtons() {
  const handleOAuthLogin = async (provider: typeof providers[number]['name']) => {
    try {
      await signInWithOAuth(provider)
    } catch (error) {
      console.error(`${provider} login failed:`, error)
    }
  }

  return (
    <div className="space-y-2">
      {providers.map((provider) => {
        const Icon = provider.icon
        return (
          <Button
            key={provider.name}
            variant="outline"
            className="w-full"
            onClick={() => handleOAuthLogin(provider.name)}
          >
            <Icon className="mr-2 h-4 w-4" />
            {provider.label}
          </Button>
        )
      })}
    </div>
  )
}
```

### Step 5: Handle OAuth Callback

**File**: `frontend/app/auth/callback/route.ts`

```typescript
import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  const requestUrl = new URL(request.url)
  const code = requestUrl.searchParams.get('code')

  if (code) {
    const supabase = createRouteHandlerClient({ cookies })
    await supabase.auth.exchangeCodeForSession(code)
  }

  // Redirect to dashboard after successful authentication
  return NextResponse.redirect(new URL('/dashboard', requestUrl.origin))
}
```

## Backend Integration

### Verify OAuth Users in FastAPI

**File**: `api-main/app/core/auth.py`

```python
from app.core.supabase import supabase_admin

async def get_oauth_user(token: str):
    """Verify OAuth user token with Supabase."""
    try:
        # Verify token with Supabase
        user = supabase_admin.auth.get_user(token)
        
        if not user:
            raise HTTPException(401, "Invalid OAuth token")
        
        # Check OAuth provider
        provider = user.app_metadata.get("provider")
        
        return {
            "id": user.id,
            "email": user.email,
            "provider": provider,
            "email_verified": user.email_confirmed_at is not None
        }
    except Exception as e:
        raise HTTPException(401, f"OAuth verification failed: {str(e)}")
```

## Advanced Configuration

### Custom OAuth Scopes

Configure provider-specific scopes:

```typescript
await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: {
    scopes: 'email profile https://www.googleapis.com/auth/calendar.readonly',
    queryParams: {
      access_type: 'offline',
      prompt: 'consent',
    }
  }
})
```

### Account Linking

Link OAuth accounts to existing users:

```typescript
const { data, error } = await supabase.auth.linkIdentity({
  provider: 'google'
})
```

### Provider Tokens

Access provider tokens for API calls:

```typescript
const { data: { session } } = await supabase.auth.getSession()
const providerToken = session?.provider_token // OAuth provider's access token
const providerRefreshToken = session?.provider_refresh_token
```

## Environment Variables

### Frontend (.env.local):

```bash
NEXT_PUBLIC_SUPABASE_URL=https://[PROJECT_REF].supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
```

### Backend (api-main/.env):

```bash
SUPABASE_URL=https://[PROJECT_REF].supabase.co
SUPABASE_SERVICE_KEY=eyJ...
SUPABASE_JWT_SECRET=your-jwt-secret
```

## Security Considerations

- Always use HTTPS in production
- Validate redirect URLs to prevent open redirects
- Store tokens securely (httpOnly cookies)
- Implement rate limiting on auth endpoints
- Log authentication events for security monitoring

## Troubleshooting

### Common Issues

**"Redirect URI mismatch"**
- Ensure callback URL matches exactly in provider settings
- Check for trailing slashes
- Verify protocol (http vs https)

**"User already registered"**
- Enable account linking in Supabase
- Handle duplicate email addresses

**Token expiration**
- Implement token refresh logic
- Handle expired sessions gracefully
