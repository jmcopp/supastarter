# Update Schema & Migrate Changes

Learn how to update your database schema and migrate changes with SQLModel and Alembic in supastarter.

## SQLModel Schema Management

To update your database schema, modify your SQLModel models in the `api-main/app/models/` directory.

### Adding a New Model

**File**: `api-main/app/models/post.py`

```python
from sqlmodel import SQLModel, Field, Relationship
from typing import Optional, List
import uuid
from datetime import datetime

class PostBase(SQLModel):
    title: str = Field(max_length=255)
    content: str
    published: bool = Field(default=False)

class Post(PostBase, table=True):
    __tablename__ = "posts"
    
    id: uuid.UUID = Field(default_factory=uuid.uuid4, primary_key=True)
    author_id: uuid.UUID = Field(foreign_key="auth.users.id")
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
    
    # Relationships
    author: Optional["User"] = Relationship(back_populates="posts")
    comments: List["Comment"] = Relationship(back_populates="post")

class PostCreate(PostBase):
    pass

class PostUpdate(SQLModel):
    title: Optional[str] = None
    content: Optional[str] = None
    published: Optional[bool] = None
```

## Database Migrations with Alembic

### Initialize Alembic (First Time Only)

```bash
cd api-main
alembic init alembic
```

### Configure Alembic

**File**: `api-main/alembic.ini`

```ini
[alembic]
script_location = alembic
prepend_sys_path = .
sqlalchemy.url = postgresql://user:pass@localhost/dbname

[loggers]
keys = root,sqlalchemy,alembic
```

**File**: `api-main/alembic/env.py`

```python
from logging.config import fileConfig
from sqlalchemy import engine_from_config
from sqlalchemy import pool
from alembic import context
from sqlmodel import SQLModel
from app.models import *  # Import all models
from app.core.config import settings

config = context.config
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)

target_metadata = SQLModel.metadata

def run_migrations_online():
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection, 
            target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()
```

### Creating and Running Migrations

#### 1. Create a Migration

After modifying your models, generate a migration:

```bash
cd api-main
alembic revision --autogenerate -m "Add posts table"
```

This creates a new migration file in `api-main/alembic/versions/`.

#### 2. Review the Migration

Check the generated migration file:

```python
# alembic/versions/xxx_add_posts_table.py
def upgrade():
    op.create_table('posts',
        sa.Column('id', sa.UUID(), nullable=False),
        sa.Column('title', sa.String(255), nullable=False),
        sa.Column('content', sa.Text(), nullable=False),
        sa.Column('author_id', sa.UUID(), nullable=False),
        sa.Column('created_at', sa.DateTime(), nullable=False),
        sa.Column('updated_at', sa.DateTime(), nullable=False),
        sa.ForeignKeyConstraint(['author_id'], ['auth.users.id']),
        sa.PrimaryKeyConstraint('id')
    )
    op.create_index('ix_posts_author_id', 'posts', ['author_id'])

def downgrade():
    op.drop_index('ix_posts_author_id')
    op.drop_table('posts')
```

#### 3. Apply Migrations

Run the migration to update your database:

```bash
# Apply all pending migrations
alembic upgrade head

# Or apply to a specific revision
alembic upgrade +1
```

#### 4. Rollback Migrations

If needed, rollback migrations:

```bash
# Rollback one migration
alembic downgrade -1

# Rollback to a specific revision
alembic downgrade <revision_id>
```

### Enable Row Level Security

After creating tables, enable RLS in Supabase:

```sql
-- Run in Supabase SQL Editor
ALTER TABLE public.posts ENABLE ROW LEVEL SECURITY;

-- Create RLS policies
CREATE POLICY "Users can manage own posts" ON public.posts
FOR ALL USING (auth.uid() = author_id);
```

## Production Migrations

### Running Migrations in Production

Add to your deployment script:

```bash
#!/bin/bash
# scripts/deploy.sh

echo "Running database migrations..."
cd api-main
alembic upgrade head

echo "Starting application..."
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

### Docker Integration

**File**: `api-main/Dockerfile`

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

# Run migrations on startup
CMD alembic upgrade head && uvicorn app.main:app --host 0.0.0.0 --port 8000
```

## Direct Table Creation (Development)

For quick development, create tables directly:

```python
# api-main/scripts/create_tables.py
from sqlmodel import SQLModel
from app.core.database import engine
from app.models import *  # Import all models

def create_db_and_tables():
    SQLModel.metadata.create_all(engine)
    print("Tables created successfully!")

if __name__ == "__main__":
    create_db_and_tables()
```

Run with:

```bash
cd api-main
python scripts/create_tables.py
```

## Migration Best Practices

- Always review generated migrations before applying
- Test migrations in development before production
- Keep migrations small and focused
- Include both upgrade and downgrade operations
- Use descriptive migration messages
- Never edit applied migrations - create new ones instead
- Enable RLS policies after table creation
