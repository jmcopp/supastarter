# Configuration

Learn how to configure your supastarter multi-container architecture.

## Overview

Supastarter's configuration system supports the multi-container deployment with environment-specific settings for:

- **Frontend Container**: Next.js configuration and environment variables
- **API Container**: FastAPI settings and Supabase integration
- **Development**: Local development with hot reloading
- **Production**: Optimized deployment on Fly.io

## Configuration Architecture

### Environment-Based Configuration
```
┌─────────────────────────────────────────────────────────────┐
│                     Configuration Layers                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Frontend Config          API Config           Shared        │
│  ┌────────────────┐   ┌────────────────┐   ┌──────────────┐ │
│  │ .env.local       │   │ api-main/.env   │   │ Fly.io       │ │
│  │ next.config.js   │   │ settings.py     │   │ secrets      │ │
│  │ config/index.ts  │   │ main.py         │   │              │ │
│  └────────────────┘   └────────────────┘   └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Frontend Configuration

### Main Config File

**File**: `frontend/config/index.ts`

```typescript
export const config = {
  // Application info
  app: {
    name: "Supastarter",
    description: "Full-stack application template",
    url: process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000",
    apiUrl: process.env.NEXT_PUBLIC_API_URL || "http://localhost:8000",
  },

  // Multi-container architecture
  architecture: {
    frontend: {
      port: 3000,
      minInstances: 1,  // Always keep 1 instance running
      autoSuspend: true,
    },
    api: {
      port: 8000,
      minInstances: 0,  // Scale to zero
      autoSuspend: true,
      healthCheckPath: "/health",
    },
  },

  // Supabase integration
  supabase: {
    url: process.env.NEXT_PUBLIC_SUPABASE_URL!,
    anonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    // Auth configuration
    auth: {
      autoRefreshToken: true,
      persistSession: true,
      detectSessionInUrl: true,
      flowType: 'pkce', // Recommended for web apps
    },
  },

  // Authentication settings
  auth: {
    enableSignup: true,
    enableMagicLink: true,
    enableSocialLogin: true,
    enablePasswordLogin: true,
    redirectAfterSignIn: "/app",
    redirectAfterLogout: "/",
    
    // Social providers (configured in Supabase)
    socialProviders: [
      { name: 'google', enabled: true },
      { name: 'github', enabled: true },
      { name: 'discord', enabled: false },
    ],
  },

  // Internationalization
  i18n: {
    enabled: true,
    locales: {
      en: {
        currency: "USD",
        label: "English",
        flag: "🇺🇸"
      },
      de: {
        currency: "EUR",
        label: "Deutsch",
        flag: "🇩🇪"
      },
      es: {
        currency: "EUR",
        label: "Español",
        flag: "🇪🇸"
      }
    },
    defaultLocale: "en",
    defaultCurrency: "USD",
    localeCookieName: "NEXT_LOCALE",
  },

  // Organizations
  organizations: {
    enabled: true,
    enableBilling: true,
    hideOrganization: false,
    enableUsersToCreateOrganizations: true,
    requireOrganization: false,
    maxMembersPerOrganization: 50,
    avatarColors: ["#4e6df5", "#e5a158", "#9dbee5", "#ced3d9"],
    forbiddenSlugs: [
      "api", "admin", "www", "mail", "ftp", "localhost", "staging",
      "new-organization", "settings", "billing", "support"
    ],
  },

  // User management
  users: {
    enableBilling: false, // Use organization billing instead
    enableOnboarding: true,
    profilePictureUpload: true,
    allowAccountDeletion: true,
    roles: {
      admin: { permissions: ['*'] },
      user: { permissions: ['read:own', 'write:own'] },
      premium: { permissions: ['read:own', 'write:own', 'premium:features'] }
    }
  },

  // UI & UX
  ui: {
    enabledThemes: ["light", "dark", "system"],
    defaultTheme: "system",
    saas: {
      enabled: true,
      useSidebarLayout: true,
      enableSearch: true,
      enableNotifications: true,
    },
    marketing: {
      enabled: true,
      showPricing: true,
      showTestimonials: true,
      showFAQ: true,
    },
  },

  // File storage
  storage: {
    provider: "supabase", // or "s3", "cloudinary"
    buckets: {
      avatars: "avatars",
      documents: "documents",
      uploads: "uploads"
    },
    maxFileSize: 10 * 1024 * 1024, // 10MB
    allowedFileTypes: {
      images: ['jpg', 'jpeg', 'png', 'gif', 'webp'],
      documents: ['pdf', 'doc', 'docx', 'txt'],
    }
  },

  // Email configuration
  email: {
    from: "hello@yourdomain.com",
    replyTo: "support@yourdomain.com",
    provider: "resend", // or "postmark", "sendgrid"
    templates: {
      welcome: "welcome-template",
      passwordReset: "password-reset-template",
      invitation: "invitation-template"
    }
  },

  // Contact form
  contactForm: {
    enabled: true,
    to: "hello@yourdomain.com",
    subject: "New contact form submission",
    enableFileAttachments: false,
  },

  // Payment configuration
  payments: {
    enabled: true,
    provider: "stripe", // or "lemonsqueezy", "paddle"
    mode: process.env.NODE_ENV === "production" ? "live" : "test",
    currency: "USD",
    
    plans: {
      starter: {
        name: "Starter",
        price: 0,
        interval: "month",
        features: ["Up to 3 projects", "Basic support"]
      },
      pro: {
        name: "Pro",
        price: 29,
        interval: "month",
        features: ["Unlimited projects", "Priority support", "Advanced features"]
      },
      enterprise: {
        name: "Enterprise",
        price: 99,
        interval: "month",
        features: ["Everything in Pro", "Custom integrations", "Dedicated support"]
      }
    }
  },

  // Feature flags
  features: {
    enableAnalytics: true,
    enableChat: false,
    enableAI: false,
    enableNotifications: true,
    enableSearch: true,
    enableDarkMode: true,
  },

  // Security
  security: {
    enableCSRF: true,
    enableRateLimiting: true,
    maxRequestsPerMinute: 100,
    sessionTimeout: 60 * 60 * 24 * 7, // 7 days
  },
};

// Type-safe config access
export type Config = typeof config;
```
## Backend Configuration

### FastAPI Settings

**File**: `api-main/app/core/config.py`

```python
from pydantic_settings import BaseSettings
from typing import List, Optional
import os

class Settings(BaseSettings):
    # Application
    app_name: str = "Supastarter API"
    app_version: str = "1.0.0"
    debug: bool = False
    environment: str = "development"
    
    # Database
    database_url: str
    direct_url: Optional[str] = None
    
    # Supabase
    supabase_url: str
    supabase_service_key: str
    supabase_jwt_secret: str
    use_supabase_auth: bool = True
    
    # Security
    secret_key: str = "your-secret-key"
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 1440  # 24 hours
    
    # CORS
    allowed_origins: List[str] = [
        "http://localhost:3000",
        "http://localhost:3001",
        "https://yourdomain.com"
    ]
    
    # Rate limiting
    rate_limit_requests: int = 100
    rate_limit_window: int = 60  # seconds
    
    # Email
    email_provider: str = "resend"
    resend_api_key: Optional[str] = None
    postmark_server_token: Optional[str] = None
    sendgrid_api_key: Optional[str] = None
    email_from: str = "hello@yourdomain.com"
    
    # Storage
    s3_access_key_id: Optional[str] = None
    s3_secret_access_key: Optional[str] = None
    s3_endpoint: Optional[str] = None
    s3_bucket_name: str = "avatars"
    
    # Payments
    stripe_secret_key: Optional[str] = None
    stripe_publishable_key: Optional[str] = None
    stripe_webhook_secret: Optional[str] = None
    
    # Monitoring
    sentry_dsn: Optional[str] = None
    enable_logging: bool = True
    log_level: str = "INFO"
    
    class Config:
        env_file = ".env"
        case_sensitive = False

# Global settings instance
settings = Settings()

# Environment-specific configurations
class DevelopmentSettings(Settings):
    debug: bool = True
    log_level: str = "DEBUG"
    environment: str = "development"

class ProductionSettings(Settings):
    debug: bool = False
    log_level: str = "WARNING"
    environment: str = "production"

class TestSettings(Settings):
    debug: bool = True
    environment: str = "test"
    database_url: str = "sqlite:///./test.db"

# Factory function
def get_settings() -> Settings:
    env = os.getenv("ENVIRONMENT", "development")
    
    if env == "production":
        return ProductionSettings()
    elif env == "test":
        return TestSettings()
    else:
        return DevelopmentSettings()

settings = get_settings()
```

## Environment Variables

### Development Environment

**File**: `.env.local` (Frontend)
```bash
# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:8000

# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...

# Features
NEXT_PUBLIC_SAAS_ENABLED=true
NEXT_PUBLIC_MARKETING_ENABLED=true
NEXT_PUBLIC_ANALYTICS_ENABLED=false

# Payments
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
```

**File**: `api-main/.env` (Backend)
```bash
# Environment
ENVIRONMENT=development
DEBUG=true

# Database
DATABASE_URL=postgresql://postgres:[password]@db.[project].supabase.co:6543/postgres?pgbouncer=true
DIRECT_URL=postgresql://postgres:[password]@db.[project].supabase.co:5432/postgres

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_KEY=eyJ...
SUPABASE_JWT_SECRET=your-jwt-secret
USE_SUPABASE_AUTH=true

# Email
EMAIL_PROVIDER=resend
RESEND_API_KEY=re_...
EMAIL_FROM=hello@yourdomain.com

# Storage
S3_ACCESS_KEY_ID=your-access-key
S3_SECRET_ACCESS_KEY=your-secret
S3_ENDPOINT=https://[project].supabase.co/storage/v1/s3
S3_BUCKET_NAME=avatars

# Payments
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

### Production Environment

**Fly.io Secrets** (Set via `fly secrets set`)
```bash
# Backend secrets
SUPABASE_SERVICE_KEY=eyJ...
SUPABASE_JWT_SECRET=your-jwt-secret
DATABASE_URL=postgresql://...
STRIPE_SECRET_KEY=sk_live_...
RESEND_API_KEY=re_...

# Frontend secrets
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...
```

## Configuration Use Cases

### 1. Marketing-Only Deployment

```typescript
// Disable SaaS features for marketing site
export const config = {
  ui: {
    saas: { enabled: false },
    marketing: { enabled: true }
  },
  auth: {
    enableSignup: false // Collect emails only
  }
}
```

### 2. SaaS-Only Deployment

```typescript
// Disable marketing for app-only deployment
export const config = {
  ui: {
    saas: { enabled: true },
    marketing: { enabled: false }
  },
  auth: {
    redirectAfterSignIn: "/dashboard",
    redirectAfterLogout: "/login"
  }
}
```

### 3. Multi-Tenant Application

```typescript
export const config = {
  organizations: {
    enabled: true,
    requireOrganization: true,
    hideOrganization: true, // Hide org switcher
    enableUsersToCreateOrganizations: false,
    maxMembersPerOrganization: 100
  },
  auth: {
    enableSignup: false, // Admin-only user creation
    redirectAfterSignIn: "/dashboard"
  },
  users: {
    enableBilling: false // Organization-level billing only
  }
}
```

### 4. Single-User Application

```typescript
export const config = {
  organizations: {
    enabled: false // Disable organizations completely
  },
  users: {
    enableBilling: true, // User-level billing
    enableOnboarding: true
  },
  ui: {
    saas: {
      useSidebarLayout: false // Use top navigation
    }
  }
}
```

### 5. Enterprise Application

```typescript
export const config = {
  auth: {
    enableSignup: false,
    enableSocialLogin: false,
    enablePasswordLogin: true,
    enableMagicLink: false // Security requirement
  },
  security: {
    sessionTimeout: 60 * 60 * 2, // 2 hours
    maxRequestsPerMinute: 1000,
    enableCSRF: true
  },
  features: {
    enableAnalytics: true,
    enableAuditLogs: true
  }
}
```

### 6. Development vs Production

```typescript
// Use environment variables for deployment-specific config
export const config = {
  ui: {
    saas: {
      enabled: process.env.NEXT_PUBLIC_SAAS_ENABLED !== "false"
    },
    marketing: {
      enabled: process.env.NEXT_PUBLIC_MARKETING_ENABLED !== "false"
    }
  },
  features: {
    enableAnalytics: process.env.NODE_ENV === "production",
    enableDebugMode: process.env.NODE_ENV === "development"
  }
}
```

## Configuration Validation

### Runtime Validation

```typescript
// frontend/lib/config-validator.ts
import { z } from 'zod'
import { config } from '../config'

const ConfigSchema = z.object({
  app: z.object({
    name: z.string().min(1),
    url: z.string().url(),
    apiUrl: z.string().url()
  }),
  supabase: z.object({
    url: z.string().url(),
    anonKey: z.string().min(1)
  }),
  // ... other validations
})

export function validateConfig() {
  try {
    ConfigSchema.parse(config)
    console.log('✅ Configuration is valid')
  } catch (error) {
    console.error('❌ Configuration validation failed:', error)
    throw new Error('Invalid configuration')
  }
}

// Call during app initialization
if (process.env.NODE_ENV !== 'production') {
  validateConfig()
}
```

## Container-Specific Configuration

### Frontend Dockerfile Build Args

```dockerfile
# Frontend Dockerfile
FROM node:18-alpine AS builder

# Build-time configuration
ARG NEXT_PUBLIC_SUPABASE_URL
ARG NEXT_PUBLIC_SUPABASE_ANON_KEY
ARG NEXT_PUBLIC_API_URL
ARG NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY

# Make available to Next.js build
ENV NEXT_PUBLIC_SUPABASE_URL=$NEXT_PUBLIC_SUPABASE_URL
ENV NEXT_PUBLIC_SUPABASE_ANON_KEY=$NEXT_PUBLIC_SUPABASE_ANON_KEY
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=$NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY

RUN npm run build
```

### API Container Configuration Loading

```python
# api-main/app/main.py
from app.core.config import settings
from app.core.logging import setup_logging

# Configure based on environment
if settings.environment == "production":
    # Production optimizations
    import uvloop
    uvloop.install()

# Setup logging
setup_logging(settings.log_level)

# Configure CORS based on environment
if settings.debug:
    allowed_origins = ["*"]
else:
    allowed_origins = settings.allowed_origins

app.add_middleware(
    CORSMiddleware,
    allow_origins=allowed_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)
```

This configuration system provides complete flexibility for different deployment scenarios while maintaining type safety and validation across the multi-container architecture.
