# Setup Guide

Learn how to set up supastarter's multi-container architecture for development.

## Overview

This guide covers the complete setup process for supastarter's modern architecture:
- **Multi-container development** with hot reloading
- **Supabase integration** for database, auth, and storage
- **FastAPI + Next.js** full-stack setup
- **Production-ready configuration** from day one

## Prerequisites

Before getting started, ensure you have:

- **Node.js** (v20 or higher)
- **Python** (3.11 or higher) 
- **Git**
- **pnpm** for Node.js dependency management
- **Docker** (for production deployment)
- **Supabase account** (free tier available)
- **VSCode** (recommended) or any code editor
## Project Setup

### Step 1: Create Supabase Project

Supastarter uses **Supabase** as the primary backend infrastructure:

1. **Create account**: Go to [supabase.io](https://supabase.io) and sign up
2. **Create new project**: Click "New project" and configure:
   - Project name: `my-supastarter-app`
   - Database password: Generate a strong password
   - Region: Choose closest to your users
3. **Wait for setup**: Takes ~2 minutes to initialize

### Step 2: Get Supabase Credentials

In your Supabase dashboard:

1. **API Settings** (`Settings > API`):
   - Copy `Project URL`
   - Copy `anon public` key
   - Copy `service_role` key (keep private)
   - Copy `JWT Secret`

2. **Database Settings** (`Settings > Database`):
   - Copy connection string under "Connection pooler"
   - Copy direct connection string

**Example connection strings**:
```bash
# Pooled connection (for app)
DATABASE_URL="postgresql://postgres:[password]@db.[project-ref].supabase.co:6543/postgres?pgbouncer=true"

# Direct connection (for migrations)
DIRECT_URL="postgresql://postgres:[password]@db.[project-ref].supabase.co:5432/postgres"
```
### Step 3: Clone and Initialize Project

```bash
# Clone the repository
git clone https://github.com/supastarter/supastarter-nextjs.git my-supastarter-app
cd my-supastarter-app

# Install dependencies
npm install  # or pnpm install

# Install Python dependencies (for API)
cd api-main
pip install -r requirements.txt
cd ..
```

### Step 4: Environment Configuration

Create environment files:

#### Frontend Environment (`.env.local`)
```bash
# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL="https://[project-ref].supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
NEXT_PUBLIC_API_URL="http://localhost:8000"

# Development
NODE_ENV="development"
```

#### API Environment (`api-main/.env`)
```bash
# Database
DATABASE_URL="postgresql://postgres:[password]@db.[project-ref].supabase.co:6543/postgres?pgbouncer=true"
DIRECT_URL="postgresql://postgres:[password]@db.[project-ref].supabase.co:5432/postgres"

# Supabase
SUPABASE_URL="https://[project-ref].supabase.co"
SUPABASE_SERVICE_KEY="eyJ..."  # service_role key
SUPABASE_JWT_SECRET="your-jwt-secret"

# Configuration
USE_SUPABASE_AUTH="true"
ENVIRONMENT="development"
```

### Step 5: Database Schema Setup

Initialize your database with SQLModel schemas:

```bash
# Navigate to API directory
cd api-main

# Run database migrations
alembic upgrade head

# Or create tables programmatically
python -c "from app.models import *; from app.core.database import engine; SQLModel.metadata.create_all(engine)"
```

### Step 6: Enable Row Level Security (RLS)

In Supabase SQL Editor, run:

```sql
-- Enable RLS on all tables
ALTER TABLE public.items ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.organizations ENABLE ROW LEVEL SECURITY;

-- Create policies for user data isolation
CREATE POLICY "Users access own items" ON public.items
  FOR ALL USING (auth.uid() = owner_id);

CREATE POLICY "Users access own organizations" ON public.organizations
  FOR ALL USING (auth.uid() = owner_id);

-- Performance indexes
CREATE INDEX idx_items_owner_id ON public.items(owner_id);
CREATE INDEX idx_organizations_owner_id ON public.organizations(owner_id);
```

### Step 7: Storage Setup (Optional)

For file uploads (avatars, documents):

1. **Create Storage Bucket** in Supabase:
   - Go to `Storage` in dashboard
   - Click "Create bucket"
   - Name: `avatars` (keep private)

2. **Get S3 Credentials**:
   - Go to `Settings > Storage`
   - Create new S3 access key
   - Copy Access Key ID and Secret

3. **Add to environment**:
```bash
# Add to api-main/.env
S3_ACCESS_KEY_ID="your-access-key"
S3_SECRET_ACCESS_KEY="your-secret-key"
S3_ENDPOINT="https://[project-ref].supabase.co/storage/v1/s3"
S3_BUCKET_NAME="avatars"
```

### Step 8: Email Provider Setup

For transactional emails (auth, notifications):

1. **Choose provider**: Recommended options:
   - **Resend** (easiest setup)
   - **Postmark** (high deliverability)
   - **SendGrid** (scalable)

2. **Configure in API** (`api-main/app/core/mail.py`):
```python
# Email configuration
from app.core.config import settings

EMAIL_CONFIG = {
    "provider": "resend",  # or "postmark", "sendgrid"
    "api_key": settings.RESEND_API_KEY,
    "from_email": "hello@yourdomain.com",
}
```

3. **Environment variables**:
```bash
# Add to api-main/.env
RESEND_API_KEY="re_..."
# or POSTMARK_SERVER_TOKEN="..."
# or SENDGRID_API_KEY="..."
```

### Step 9: Payment Provider Setup

For subscriptions and payments:

1. **Choose provider**:
   - **Stripe** (most popular)
   - **LemonSqueezy** (indie-friendly)
   - **Paddle** (global taxes)

2. **Configure Stripe** (`api-main/app/core/payments.py`):
```python
import stripe
from app.core.config import settings

stripe.api_key = settings.STRIPE_SECRET_KEY

# Webhook endpoint
@router.post("/webhooks/stripe")
async def stripe_webhook(request: Request):
    # Handle Stripe webhooks
    pass
```

3. **Environment variables**:
```bash
# Add to .env.local and api-main/.env
STRIPE_PUBLISHABLE_KEY="pk_test_..."
STRIPE_SECRET_KEY="sk_test_..."
STRIPE_WEBHOOK_SECRET="whsec_..."
```
### Step 10: Verify Database Setup

1. **Check tables** in Supabase dashboard:
   - Go to `Table Editor`
   - Verify your tables were created
   - Check RLS policies are enabled

2. **Test database connection**:
```bash
# Test from API container
cd api-main
python -c "from app.core.database import engine; print('Database connected:', engine.url)"
```

3. **Create test user** in Supabase:
   - Go to `Authentication > Users`
   - Click "Add user"
   - Add email and password for testing
### Step 11: Initialize Git Repository

```bash
# Initialize new Git repository
rm -rf .git
git init
git add .
git commit -m "Initial supastarter setup"

# Add upstream for updates
git remote add upstream https://github.com/supastarter/supastarter-nextjs.git
```

## Development Server

### Start Both Services

Open **two terminals**:

#### Terminal 1: FastAPI Backend
```bash
cd api-main

# Start FastAPI with hot reload
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# API will be available at:
# http://localhost:8000 (API endpoints)
# http://localhost:8000/docs (Swagger UI)
# http://localhost:8000/redoc (ReDoc)
```

#### Terminal 2: Next.js Frontend
```bash
# Start Next.js with hot reload
npm run dev
# or: pnpm dev

# Frontend will be available at:
# http://localhost:3000
```

### Verify Setup

1. **API Health**: Visit `http://localhost:8000/health`
2. **API Docs**: Visit `http://localhost:8000/docs`
3. **Frontend**: Visit `http://localhost:3000`
4. **Authentication**: Try signing up/logging in

### Create Admin User

Use Supabase Auth dashboard or create programmatically:

```python
# api-main/scripts/create_admin.py
from app.core.auth import create_admin_user

async def create_admin():
    user = await create_admin_user(
        email="admin@yourdomain.com",
        password="secure-password",
        role="admin"
    )
    print(f"Admin user created: {user.email}")

# Run: python scripts/create_admin.py
```

## Next Steps

1. **[Configure your application](Configuration.md)** - Customize branding, features
2. **[Set up authentication](Authentication/Overview.md)** - OAuth providers, policies
3. **[Deploy to production](Deployment/Fly_io.md)** - Multi-container Fly.io deployment
4. **[Add custom features](Customization/Overview.md)** - Extend functionality

## Troubleshooting

### Common Issues

**Database Connection Errors**
- Verify Supabase project is active
- Check connection strings are correct
- Ensure RLS policies don't block access

**Authentication Issues**
- Check JWT secret matches between Supabase and API
- Verify CORS settings allow localhost
- Test with Supabase Auth directly first

**API Not Starting**
- Check Python dependencies are installed
- Verify environment variables are set
- Look for import errors in logs

**Frontend Build Errors**
- Clear node_modules and reinstall
- Check environment variables are prefixed with NEXT_PUBLIC_
- Verify API_URL is accessible

Your supastarter application is now ready for development! 🚀

Previous

Tech Stack

Next

Configuration

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




