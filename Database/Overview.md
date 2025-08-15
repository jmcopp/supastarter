Overview
Learn how to use the database in supastarter.

With supastarter you can choose between Prisma and Drizzle as your database ORM.

You can find the schema and configuration in the packages/database directory either in the prisma or drizzle folder.

## Database Components

### [Schema Management](Update_schema_&_migrate_changes.md)
Learn how to update database schemas using SQLModel and Alembic migrations with Supabase integration.

### [Database Client Usage](Use_database_client.md)
How to use the SQLModel database client with user context isolation and RLS policies.

### [Database Studio](Use_database_studio.md)
Access the Supabase Studio for visual database management, query building, and real-time data viewing.

### [Supabase Setup](Supabase_setup.md)
Complete setup guide for integrating Supabase with supastarter's multi-container architecture.

## Security Model

### User Context Isolation
```python
# api-main/app/models/base.py
from sqlmodel import Field, SQLModel
from datetime import datetime
import uuid

class TimestampMixin(SQLModel):
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

class UserOwnedMixin(SQLModel):
    owner_id: uuid.UUID = Field(
        foreign_key="auth.users.id", 
        nullable=False, 
        index=True
    )
```

### Automatic RLS Enforcement
All database operations automatically respect RLS policies:
```python
# User context is automatically applied
async def get_user_items(user_id: str, db: AsyncSession):
    # RLS ensures only user's items are returned
    items = await db.exec(select(Item)).all()
    return items
```

### Performance Optimization
```sql
-- Optimized indexes for RLS queries
CREATE INDEX idx_items_owner_created ON public.items(owner_id, created_at DESC);
CREATE INDEX idx_items_owner_title ON public.items(owner_id, title);
```

## Multi-Container Database Access

### API Main Container
- **Primary database operations** for business logic
- **User-scoped queries** with automatic RLS
- **Transaction management** with context isolation

### Frontend Container
- **Supabase client** for real-time subscriptions
- **Direct database access** for read-only operations (optional)
- **Authentication state** synchronized with backend

## Database Configuration

### Connection Setup
```python
# api-main/app/core/database.py
from sqlmodel import create_engine
from supabase import create_client

# SQLModel engine for API operations
engine = create_engine(
    settings.DATABASE_URL,
    pool_pre_ping=True,
    pool_recycle=300,
)

# Supabase client for real-time features
supabase = create_client(
    settings.SUPABASE_URL,
    settings.SUPABASE_ANON_KEY
)
```

### Environment Variables
```bash
# Production configuration
DATABASE_URL=postgresql://...
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_KEY=eyJ...  # Server-side only
SUPABASE_JWT_SECRET=your-jwt-secret
```

## Development vs Production

### Development
- **Local Supabase** instance or development project
- **Database migrations** applied automatically
- **Schema validation** on startup

### Production
- **Hosted Supabase** with connection pooling
- **Automated backups** and point-in-time recovery
- **Read replicas** for performance (if needed)
- **Connection monitoring** and alerting

## Migration from Other ORMs

If migrating from Prisma or Drizzle:

1. **Schema Conversion**: Convert existing models to SQLModel format
2. **RLS Setup**: Enable and configure Row Level Security
3. **Data Migration**: Preserve existing data during transition
4. **Client Updates**: Update all database access to use new patterns

Detailed migration guides are available in each component section.
Previous

Local Development

Next

Use database client

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




