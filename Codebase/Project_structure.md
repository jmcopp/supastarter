# Project Structure

Learn how supastarter's multi-container architecture is organized for production deployment.

## Architecture Overview

supastarter uses a **container-optimized monorepo structure** that enables:
- Independent deployment and scaling of each service
- Shared code and types between containers
- Efficient development workflow with hot reloading
- Production-ready security isolation

## Multi-Container Repository Structure

```
merged-project/
├── frontend/                    # Next.js Container
│   ├── Dockerfile
│   ├── fly.toml
│   ├── package.json
│   ├── .env.production
│   └── app/
│       ├── (auth)/             # Auth pages
│       ├── (dashboard)/        # Protected pages
│       └── api/                # API routes (if needed)
├── api-main/                   # Primary FastAPI Container
│   ├── Dockerfile
│   ├── fly.toml
│   ├── pyproject.toml
│   ├── .env.production
│   └── app/
│       ├── core/
│       │   ├── auth.py         # Hybrid auth system
│       │   ├── security.py     # Security isolation
│       │   └── config.py
│       ├── models/
│       ├── routes/
│       └── main.py
├── api-gpu/                    # GPU-enabled Container (optional)
│   ├── Dockerfile
│   ├── fly.toml
│   ├── requirements.txt
│   └── app/
├── shared/                     # Shared Code
│   ├── types/                 # TypeScript types
│   ├── schemas/               # Pydantic schemas
│   └── utils/
└── scripts/
    ├── deploy.sh
    ├── setup-fly.sh
    └── validate-security.py
```

## Container Responsibilities

### Frontend Container (`frontend/`)
- **Purpose**: Next.js application with SSR/SSG capabilities
- **Deployment**: Public-facing with 1 minimum running instance
- **Authentication**: JWT token management and session handling
- **Scaling**: Auto-start/suspend based on traffic

### API Main Container (`api-main/`)
- **Purpose**: Primary business logic and database operations
- **Deployment**: Private internal networking only
- **Authentication**: Hybrid auth system supporting Supabase + custom JWT
- **Scaling**: Scale-to-zero with auto-start on requests

### API GPU Container (`api-gpu/`) - Optional
- **Purpose**: GPU-intensive operations (ML, AI, image processing)
- **Deployment**: Private with specialized GPU hardware
- **Scaling**: Scale-to-zero to optimize GPU costs

### Shared Libraries (`shared/`)
- **Types**: TypeScript definitions shared across containers
- **Schemas**: Pydantic models for API validation
- **Utils**: Common utilities and helpers

## Development Structure

During development, the monorepo maintains compatibility with the original structure while preparing for container deployment:

```
apps/                          # Development Apps
└── web/                       # → Becomes frontend/ container
    ├── app/                   # Next.js app directory
    ├── components/            # UI components
    └── lib/                   # Client-side utilities

packages/                      # Shared Packages
├── auth/                      # → Integrated into api-main/
├── api/                       # → Becomes api-main/ container
├── database/                  # → SQLModel integration
├── shared-types/              # → shared/types/
└── ui/                        # → Part of frontend/

config/                        # Build Configuration
├── docker/                    # Docker configurations
├── fly/                       # Fly.io deployment configs
└── env/                       # Environment templates
```

## Security Isolation

Each container operates with strict security boundaries:

- **Network Isolation**: Private internal networking between API containers
- **Authentication Context**: Per-request user isolation with context variables
- **Database Security**: Row Level Security (RLS) policies on all tables
- **Rate Limiting**: Per-user request limiting across all containers

## Configuration Management

Environment-specific configurations:

```
.env.local                     # Development environment
.env.production               # Production secrets
frontend/.env.production      # Frontend-specific config
api-main/.env.production      # Backend-specific config
```

## Deployment Artifacts

- **Multi-stage Dockerfiles** optimized for each container type
- **Fly.toml configurations** with container-specific scaling rules
- **Health check endpoints** for monitoring and auto-recovery
- **Security validation scripts** for deployment verification
