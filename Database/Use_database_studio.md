# Use Database Studio

Learn how to use Supabase Studio for visual database management in supastarter.

## Accessing Supabase Studio

Supabase provides a powerful web-based database studio for managing your PostgreSQL database visually.

### Access Methods

1. **Via Supabase Dashboard**
   - Log in to [supabase.com](https://supabase.com)
   - Select your project
   - Click on "Table Editor" in the sidebar

2. **Direct URL**
```
https://app.supabase.com/project/[PROJECT_REF]/editor
```

## Features

### Table Editor
- **Visual table management** - Create, modify, and delete tables
- **Column management** - Add, edit, remove columns with type safety
- **Data editing** - Insert, update, delete records directly
- **Relationships** - Visual foreign key management
- **RLS Policies** - Create and manage Row Level Security

### SQL Editor
- **Query execution** - Run SQL queries directly
- **Query history** - Access previously run queries
- **Saved queries** - Store frequently used queries
- **Schema visualization** - View database schema diagram

### Real-time Monitoring
- **Live data updates** - See changes as they happen
- **Connection monitoring** - Track active connections
- **Query performance** - Analyze slow queries

## Common Operations

### Viewing Tables

1. Navigate to "Table Editor"
2. Select a table from the sidebar
3. View and filter data
4. Export data as CSV or JSON

### Creating Tables

1. Click "New Table"
2. Define table structure:
```sql
-- Generated SQL preview
CREATE TABLE public.posts (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT,
  author_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```
3. Enable RLS if needed
4. Click "Save"

### Managing RLS Policies

1. Select a table
2. Click "RLS Policies" tab
3. Click "New Policy"
4. Choose policy template or write custom:
```sql
-- Example: Users can only see their own data
CREATE POLICY "Users view own data" ON public.posts
FOR SELECT USING (auth.uid() = author_id);
```

### Running SQL Queries

1. Navigate to "SQL Editor"
2. Write your query:
```sql
-- Example: Get user statistics
SELECT 
  u.email,
  COUNT(p.id) as post_count,
  MAX(p.created_at) as last_post
FROM auth.users u
LEFT JOIN public.posts p ON u.id = p.author_id
GROUP BY u.id, u.email
ORDER BY post_count DESC;
```
3. Click "Run" or press Cmd/Ctrl + Enter

## Database Backups

### Viewing Backups

1. Go to "Settings" → "Database"
2. Click "Backups" tab
3. View available backups (last 7 days on free tier)

### Restoring Data

1. Select a backup
2. Choose restore point
3. Click "Restore"

## Performance Optimization

### Indexes
Create indexes for frequently queried columns:
```sql
-- In SQL Editor
CREATE INDEX idx_posts_author_created 
ON public.posts(author_id, created_at DESC);
```

### Query Analysis
Use EXPLAIN ANALYZE to understand query performance:
```sql
EXPLAIN ANALYZE
SELECT * FROM posts 
WHERE author_id = 'user-uuid' 
ORDER BY created_at DESC 
LIMIT 10;
```

## Integration with SQLModel

### Syncing Schema Changes

After making changes in Supabase Studio:

1. Generate SQLModel models from existing tables:
```python
# api-main/scripts/generate_models.py
from sqlalchemy import create_engine, MetaData
from sqlmodel import SQLModel

engine = create_engine(settings.DATABASE_URL)
metadata = MetaData()
metadata.reflect(bind=engine)

# Generate model code from reflected tables
for table_name, table in metadata.tables.items():
    print(f"# Model for {table_name}")
    # Generate SQLModel class...
```

2. Create Alembic migration to track changes:
```bash
alembic revision --autogenerate -m "Sync with Supabase Studio changes"
```

## Security Considerations

### Access Control

- Studio access requires Supabase dashboard authentication
- Changes are logged in audit trail
- Use read-only credentials for viewing-only access

### Production Best Practices

- Avoid direct data manipulation in production
- Use migrations for schema changes
- Test RLS policies thoroughly before deployment
- Monitor query performance regularly
- Set up alerts for slow queries

## Troubleshooting

### Common Issues

**Cannot see tables**
- Check if RLS is enabled without policies
- Verify user permissions
- Ensure tables are in public schema

**Slow queries**
- Add appropriate indexes
- Optimize query structure
- Check for missing statistics

**RLS policy conflicts**
- Review policy conditions
- Check for overlapping policies
- Test with different user roles
