Fly.io
Learn how to deploy your supastarter application to Fly.io.

This guide will show you how to deploy your supastarter SaaS to fly.io in minutes. And the best part? You can actually start for free thanks to the generous free tier of fly.io.

Why host your SaaS on fly.io?
Self-hosting your SaaS app as a docker image allows you to have complete control over your server environment. It ensures better privacy, contributes to cost savings if managed correctly, and offers you the flexibility to customize your server setup to suit your specific needs. It can also give your application a performance boost compared to hosting on a serverless platform like Vercel, since it removes cold starts.

## Prerequisites

1. **Fly.io Account**: Sign up at [fly.io](https://fly.io)
2. **Fly CLI**: Install the [Fly CLI](https://fly.io/docs/hands-on/install-flyctl/)
3. **Supabase Project**: Set up your Supabase project and get credentials
4. **Docker**: Ensure Docker is running locally

## Step 1: Project Structure Setup

Ensure your project follows the multi-container structure:

```bash
# From project root
mkdir -p frontend api-main shared scripts

# Move existing files
mv apps/web/* frontend/
mv packages/api/* api-main/
mv packages/database/models shared/schemas/
```

## Step 2: Configure Container Applications

### Frontend Container (`frontend/fly.toml`)

```toml
app = "myapp-frontend"
primary_region = "ord"

[build]
  dockerfile = "Dockerfile"

[http_service]
  internal_port = 3000
  force_https = true
  auto_stop_machines = "suspend"
  auto_start_machines = true
  min_machines_running = 1  # Keep responsive

  [http_service.concurrency]
    type = "requests"
    soft_limit = 25
    hard_limit = 50

[[http_service.checks]]
  interval = "10s"
  method = "GET"
  path = "/api/health"
  timeout = "2s"

[env]
  NODE_ENV = "production"
  NEXT_PUBLIC_API_URL = "https://myapp-api.fly.dev"
  NEXT_PUBLIC_SUPABASE_URL = "${SUPABASE_URL}"
  NEXT_PUBLIC_SUPABASE_ANON_KEY = "${SUPABASE_ANON_KEY}"
```

### API Container (`api-main/fly.toml`)

```toml
app = "myapp-api"
primary_region = "ord"

[build]
  dockerfile = "Dockerfile"

# NO [[services]] section = private by default

[http_service]
  internal_port = 8000
  auto_stop_machines = "suspend"
  auto_start_machines = true
  min_machines_running = 0  # Scale to zero

  [http_service.concurrency]
    type = "requests"
    soft_limit = 20
    hard_limit = 40

[[http_service.checks]]
  interval = "15s"
  method = "GET"
  path = "/health"
  timeout = "5s"

[env]
  ENVIRONMENT = "production"
  USE_SUPABASE_AUTH = "true"

[[vm]]
  cpus = 2
  memory = "2gb"
```

## Step 3: Create Dockerfiles

### Frontend Dockerfile (`frontend/Dockerfile`)

```dockerfile
FROM node:18-alpine AS base

FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci --only=production

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build with proper env vars
ARG NEXT_PUBLIC_SUPABASE_URL
ARG NEXT_PUBLIC_SUPABASE_ANON_KEY
ARG NEXT_PUBLIC_API_URL

RUN npm run build

FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

### API Dockerfile (`api-main/Dockerfile`)

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY pyproject.toml ./
RUN pip install --no-cache-dir uv && \
    uv pip install --system --no-cache .

# Copy application
COPY ./app ./app

# Create non-root user
RUN useradd --create-home --shell /bin/bash app
USER app

EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Step 4: Deploy with Automated Script

Create `scripts/deploy.sh`:

```bash
#!/bin/bash
# scripts/deploy.sh

set -e

echo "🚀 Starting multi-container deployment..."

# Set secrets
fly secrets set -a myapp-api \
  SUPABASE_URL="$SUPABASE_URL" \
  SUPABASE_KEY="$SUPABASE_KEY" \
  SUPABASE_JWT_SECRET="$SUPABASE_JWT_SECRET" \
  DATABASE_URL="$DATABASE_URL"

fly secrets set -a myapp-frontend \
  NEXT_PUBLIC_SUPABASE_URL="$SUPABASE_URL" \
  NEXT_PUBLIC_SUPABASE_ANON_KEY="$SUPABASE_ANON_KEY"

# Deploy backend first
echo "📦 Deploying API..."
cd api-main
fly deploy --remote-only

# Deploy frontend
echo "📦 Deploying Frontend..."
cd ../frontend
fly deploy --remote-only

echo "✅ Deployment complete!"
```

## Step 5: Manual Deployment Steps

If you prefer manual deployment:

### Deploy API Container
```bash
cd api-main
fly launch --dockerfile Dockerfile
# Follow prompts, set app name to "myapp-api"
# Configure internal port to 8000
fly deploy
```

### Deploy Frontend Container
```bash
cd frontend
fly launch --dockerfile Dockerfile
# Follow prompts, set app name to "myapp-frontend"
# Configure port to 3000
fly deploy
```

### Set Environment Variables
```bash
# API secrets
fly secrets set -a myapp-api \
  SUPABASE_URL="https://your-project.supabase.co" \
  SUPABASE_JWT_SECRET="your-jwt-secret" \
  DATABASE_URL="postgresql://..."

# Frontend secrets
fly secrets set -a myapp-frontend \
  NEXT_PUBLIC_SUPABASE_URL="https://your-project.supabase.co" \
  NEXT_PUBLIC_SUPABASE_ANON_KEY="eyJ..."
```

## Step 6: Verify Deployment

### Check App Status
```bash
fly status -a myapp-frontend
fly status -a myapp-api
```

### Monitor Logs
```bash
# Frontend logs
fly logs -a myapp-frontend

# API logs
fly logs -a myapp-api
```

### Test Endpoints
```bash
# Frontend
curl https://myapp-frontend.fly.dev/

# API health check (internal)
fly ssh console -a myapp-frontend
curl http://myapp-api.internal:8000/health
```

## Production Optimization

### Cost Optimization
- **Frontend**: 1 minimum instance for responsiveness
- **API**: Scale-to-zero to minimize costs
- **Regional deployment**: Choose region closest to users

### Performance Tuning
- **Concurrency limits**: Adjust based on traffic patterns
- **Health checks**: Optimize intervals for your use case
- **Machine sizing**: Start small and scale based on metrics

### Security Hardening
- **Private networking**: API containers are internal-only
- **Secret management**: All sensitive data in Fly secrets
- **SSL/TLS**: Automatic HTTPS with Let's Encrypt certificates



## Monitoring and Observability

### Health Checks
Both containers include comprehensive health checks:

```python
# api-main/app/routes/health.py
@router.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat(),
        "service": "api-main",
        "version": "1.0.0"
    }
```

### Logging
```bash
# Stream live logs
fly logs -a myapp-frontend --region ord
fly logs -a myapp-api --region ord

# Search logs
fly logs -a myapp-api | grep ERROR
```

### Metrics
```bash
# View app metrics
fly dashboard -a myapp-frontend
fly dashboard -a myapp-api
```

## Troubleshooting

### Common Issues

**SSL Errors (ERR_SSL_PACKET_LENGTH_TOO_LONG)**
- Ensure your API URLs use HTTPS in production
- Check internal networking configuration

**Container Won't Start**
```bash
# Check container logs
fly logs -a myapp-api

# SSH into container
fly ssh console -a myapp-api

# Check process status
fly status -a myapp-api
```

**Database Connection Issues**
- Verify Supabase credentials in Fly secrets
- Check DATABASE_URL format
- Ensure Supabase project allows connections from Fly.io IPs

**Authentication Failures**
- Verify JWT secret matches between Supabase and API
- Check CORS configuration for cross-origin requests
- Ensure RLS policies are properly configured

### Getting Help
- Check the [troubleshooting guide](../Troubleshooting.md)
- Review Fly.io documentation for platform-specific issues
- Use `fly doctor` for environment diagnostics

## Cost Management

Estimated monthly costs (USD):
- **Frontend**: ~$5-15 (1 shared-cpu-1x, 256MB RAM)
- **API**: ~$0-10 (scales to zero, usage-based)
- **Total**: ~$5-25 depending on traffic

Optimization tips:
- Use `fly scale count 0` for API during development
- Monitor usage with `fly dashboard`
- Consider upgrading to performance tiers only when needed

## Next Steps

After successful deployment:

1. **[Configure DNS](https://fly.io/docs/networking/custom-domain/)** for your custom domain
2. **[Set up monitoring](../Monitoring/Overview.md)** with Sentry or similar
3. **[Configure CI/CD](https://fly.io/docs/app-guides/continuous-deployment-with-github-actions/)** for automated deployments
4. **[Scale based on metrics](https://fly.io/docs/scaling/)** as your app grows

Your supastarter application is now running with enterprise-grade security, auto-scaling, and cost optimization on Fly.io's global infrastructure.

---

**Previous**: [Docker Deployment](Docker.md)  
**Next**: [Monitoring Setup](../Monitoring/Overview.md)

© 2025 supastarter. All rights reserved.




