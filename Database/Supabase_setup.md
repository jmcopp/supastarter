# Supabase Setup

Learn how to integrate Supabase with supastarter's multi-container architecture.

## Overview

This guide shows you how to set up Supabase as the complete backend infrastructure for supastarter:

- **Database**: PostgreSQL with Row Level Security (RLS)
- **Authentication**: Supabase Auth with JWT tokens (replaces Supabase Auth)
- **Storage**: S3-compatible file storage
- **Real-time**: WebSocket subscriptions for live data

**Key Change**: Unlike the previous setup, supastarter now uses **Supabase Auth directly** instead of Supabase Auth, enabling seamless RLS integration and simplified architecture.

## Prerequisites

- Supabase account (create free at [supabase.io](https://supabase.io))
- Fly.io account for deployment
- Basic understanding of PostgreSQL and JWT tokens

## Step 1: Create Supabase Project & Get Credentials


### Database Connection Strings
In the Supabase dashboard:
1. Click **Connect** → **ORM** tab → **SQLModel**
2. Copy both `DATABASE_URL` and `DIRECT_URL`

### Authentication Credentials
1. Go to **Settings** → **API**
2. Copy the following:
   - `Project URL`
   - `anon public` key
   - `service_role secret` key (keep private)

### JWT Secret
1. Go to **Settings** → **API** 
2. Copy `JWT Secret` (needed for token verification)

## Step 2: Configure Supastarter Project

### Environment Variables
Create/update your environment files:

#### Development (`.env.local`)
```bash
# Database
DATABASE_URL="postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:6543/postgres?pgbouncer=true&connection_limit=1"
DIRECT_URL="postgresql://postgres:[PASSWORD]@db.[PROJECT-REF].supabase.co:5432/postgres"

# Supabase
SUPABASE_URL="https://[PROJECT-REF].supabase.co"
SUPABASE_ANON_KEY="eyJ..."
SUPABASE_SERVICE_KEY="eyJ..."  # Server-side only
SUPABASE_JWT_SECRET="your-jwt-secret"

# API Configuration
USE_SUPABASE_AUTH="true"
NEXT_PUBLIC_SUPABASE_URL="https://[PROJECT-REF].supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
NEXT_PUBLIC_API_URL="http://localhost:8000"
```

#### Production (Fly.io secrets)
```bash
# Set API container secrets
fly secrets set -a myapp-api \
  SUPABASE_URL="https://[PROJECT-REF].supabase.co" \
  SUPABASE_SERVICE_KEY="eyJ..." \
  SUPABASE_JWT_SECRET="your-jwt-secret" \
  DATABASE_URL="postgresql://..."

# Set frontend secrets
fly secrets set -a myapp-frontend \
  NEXT_PUBLIC_SUPABASE_URL="https://[PROJECT-REF].supabase.co" \
  NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
```

## Step 3: Database Schema Setup

### SQLModel Configuration
Update your database models to use SQLModel with Supabase:

```python
# api-main/app/models/base.py
from sqlmodel import SQLModel, Field, create_engine
from datetime import datetime
import uuid
from typing import Optional

# Base mixins for all models
class TimestampMixin(SQLModel):
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

class UserOwnedMixin(SQLModel):
    """Mixin for user-owned resources with RLS"""
    owner_id: uuid.UUID = Field(
        foreign_key="auth.users.id",  # References Supabase auth.users
        nullable=False,
        index=True
    )

# Example model
class Item(ItemBase, UserOwnedMixin, TimestampMixin, table=True):
    __tablename__ = "items"
    id: uuid.UUID = Field(default_factory=uuid.uuid4, primary_key=True)
    title: str = Field(max_length=255)
    description: Optional[str] = Field(None, max_length=1000)
```

### Database Engine Configuration
```python
# api-main/app/core/database.py
from sqlmodel import create_engine
from app.core.config import settings

engine = create_engine(
    settings.DATABASE_URL,
    pool_pre_ping=True,
    pool_recycle=300,
    echo=settings.DEBUG  # SQL logging in development
)
```
## Step 4: Database Migrations & RLS Setup

### Create Tables
Run migrations to create your database schema:

```bash
# Using Alembic for SQLModel migrations
cd api-main
alembic upgrade head

# Or create tables programmatically
python -c "from app.models import *; from app.core.database import engine; SQLModel.metadata.create_all(engine)"
```

### Enable Row Level Security (RLS)
In the Supabase SQL Editor, enable RLS for all tables:

```sql
-- Enable RLS on all tables
ALTER TABLE public.items ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.organizations ENABLE ROW LEVEL SECURITY;
-- ... for all your tables

-- Create RLS policies
CREATE POLICY "Users can access own items" ON public.items
  FOR ALL USING (auth.uid() = owner_id);

CREATE POLICY "Users can manage own items" ON public.items
  FOR ALL USING (auth.uid() = owner_id);

-- Performance indexes for RLS
CREATE INDEX idx_items_owner_id ON public.items(owner_id);
CREATE INDEX idx_items_owner_created ON public.items(owner_id, created_at DESC);
```

## Step 5: Storage Configuration

### Create Storage Bucket
1. Go to **Storage** in Supabase dashboard
2. Click **Create bucket**
3. Name: `avatars` (or customize in config)
4. **Keep private** (access controlled via RLS)
5. Set file size and type restrictions as needed

### Storage Credentials
1. Go to **Settings** → **Storage**
2. Under **S3 Access Keys**, click **New access key**
3. Add description and create
4. Copy the Access Key ID and Secret

### Storage Environment Variables
```bash
# Add to .env.local
S3_ACCESS_KEY_ID="your-access-key"
S3_SECRET_ACCESS_KEY="your-secret-key"
S3_ENDPOINT="https://[PROJECT-REF].supabase.co/storage/v1/s3"
S3_BUCKET_NAME="avatars"
```

### Storage RLS Policies
```sql
-- Allow authenticated users to upload their own files
CREATE POLICY "Users can upload own files" ON storage.objects
  FOR INSERT WITH CHECK (auth.uid()::text = (storage.foldername(name))[1]);

CREATE POLICY "Users can view own files" ON storage.objects
  FOR SELECT USING (auth.uid()::text = (storage.foldername(name))[1]);

CREATE POLICY "Users can delete own files" ON storage.objects
  FOR DELETE USING (auth.uid()::text = (storage.foldername(name))[1]);
```
## Step 6: Authentication Setup

### Configure Supabase Auth
1. Go to **Authentication** → **Settings**
2. Configure your site URL: `http://localhost:3000` (development)
3. Add production URL when deploying
4. Enable desired auth providers (email, OAuth, etc.)

### Auth RLS Integration
Supabase Auth automatically works with RLS policies using `auth.uid()`:

```sql
-- RLS policies automatically use current user's ID
CREATE POLICY "policy_name" ON table_name
  FOR ALL USING (auth.uid() = owner_id);
```

## Step 7: Start Development

### Backend Development
```bash
cd api-main

# Install dependencies
pip install -r requirements.txt

# Run FastAPI server
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### Frontend Development
```bash
cd frontend

# Install dependencies
npm install

# Run Next.js
npm run dev
```

### Test Integration
1. **API Documentation**: Visit `http://localhost:8000/docs`
2. **Frontend**: Visit `http://localhost:3000`
3. **Authentication**: Test login/signup flow
4. **Database**: Verify RLS policies work correctly

## Production Deployment

For production deployment on Fly.io, see the [Fly.io Deployment Guide](../Deployment/Fly_io.md) which includes:

- Multi-container setup with proper networking
- Environment variable configuration
- Security hardening
- Monitoring and observability

## Troubleshooting

### Common Issues

**RLS Policy Errors**
- Ensure all tables have RLS enabled
- Verify policy conditions match your data model
- Test policies in Supabase SQL Editor

**Authentication Issues**
- Check JWT secret matches between Supabase and API
- Verify CORS settings for your domain
- Ensure auth.users table exists

**Connection Issues**
- Verify DATABASE_URL format
- Check Supabase project is not paused
- Confirm IP restrictions allow your deployment

Your supastarter application is now fully integrated with Supabase's authentication, database, and storage services!
