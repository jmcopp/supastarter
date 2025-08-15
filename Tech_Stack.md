# Tech Stack

Learn about the comprehensive tech stack powering supastarter's multi-container architecture.

## Architecture Philosophy

supastarter combines modern frontend development with enterprise-grade backend services, deployed using production-ready containerization. Our tech stack is designed for:

- **Security-first approach** with isolated containers and RLS
- **Scalability** with auto-scaling containers and optimized resource usage
- **Type safety** from database to frontend with full-stack type generation
- **Developer experience** with hot reloading, auto-documentation, and comprehensive tooling

## Core Technologies

### Frontend Layer

**Turborepo**  
TurboRepo manages our monorepo structure, enabling efficient code sharing between frontend and backend services while maintaining clear container boundaries. The monorepo structure supports:
- Multi-container deployments with shared types and utilities
- Independent scaling of frontend and API services
- Consistent tooling across all containers

**Next.js**  
Next.js powers our frontend container with server-side rendering, static generation, and optimized performance. In our multi-container architecture:
- Deployed as a separate container from API services
- Communicates with FastAPI backend via typed clients
- Handles authentication state and JWT token management
- Maintains 1 minimum running instance for responsiveness

### Backend Layer

**FastAPI**  
FastAPI serves as our primary API framework across multiple containers:
- **API Main**: General business logic with scale-to-zero capability
- **API GPU**: Optional GPU-intensive operations
- Built-in OpenAPI documentation and type validation
- Hybrid authentication supporting both Supabase Auth and custom JWT
- Context-isolated request handling for security

**SQLModel**  
SQLModel (built on SQLAlchemy) provides our database layer with:
- Full type safety from database to API responses
- Automatic Pydantic model generation
- Support for Supabase PostgreSQL with RLS policies
- Migration management with Alembic

### Database & Authentication

**Supabase**  
Supabase provides our database and authentication infrastructure:
- PostgreSQL with Row Level Security (RLS) policies
- JWT-based authentication with auth.users table integration
- Real-time subscriptions and secure API access
- File storage with S3-compatible interface

**Hybrid Authentication System**  
Custom authentication layer supporting:
- Supabase Auth as primary provider with JWT tokens
- Context-isolated user sessions with request tracing
- Rate limiting and security middleware
- Automatic RLS policy enforcement

### Frontend Technologies

**Tanstack Query**  
Server state management with optimized caching:
- Type-safe API client generation
- Automatic background refetching
- Optimistic updates and error handling

**Tailwind CSS**  
Utility-first CSS framework providing:
- Consistent design system across containers
- Optimized production builds
- Component-based styling with shadcn/ui integration

**Radix UI / shadcn/ui**  
Accessible component primitives:
- Headless UI components for maximum customization
- Pre-built components via shadcn/ui integration
- Full accessibility compliance (WCAG 2.1)

### Deployment & Infrastructure

**Fly.io Multi-Container Deployment**  
Production deployment with:
- Container isolation with private internal networking
- Auto-scaling with scale-to-zero for cost optimization
- Health checks and automatic recovery
- Regional deployment for performance

**Docker Containerization**  
Multi-stage Docker builds optimized for:
- Minimal production image sizes
- Security hardening with non-root users
- Efficient layer caching and build times

**Content Collections**  
Markdown-based content management:
- Type-safe content with automatic validation
- MDX support for interactive documentation
- Git-based versioning and collaboration

## Security & Monitoring

- **Row Level Security (RLS)** on all database tables
- **Context isolation** with per-request user sessions
- **Rate limiting** and security headers
- **Health checks** and observability
- **Audit logging** for compliance requirements

## Development Experience

- **Full-stack type safety** with automatic code generation
- **Hot reloading** across all containers in development
- **Auto-generated API documentation** with OpenAPI/Swagger
- **Database studio** integration with Supabase
- **Comprehensive testing** with E2E and unit test support
