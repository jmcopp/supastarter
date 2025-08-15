# Introduction

Welcome to **supastarter** - a comprehensive fullstack application template that combines the power of Next.js frontend, FastAPI backend services, and Supabase infrastructure in a production-ready multi-container architecture.

## What is supastarter?

supastarter is a cutting-edge fullstack starter kit designed for building scalable SaaS applications with enterprise-grade security, performance, and deployment capabilities. It embodies modern best practices through:

- **Multi-container architecture** with strict security isolation
- **Hybrid authentication system** supporting Supabase Auth with JWT tokens
- **Production-ready Fly.io deployment** with auto-scaling and cost optimization
- **Full-stack type safety** from database to frontend
- **Enterprise security** with Row Level Security (RLS) and context isolation

## Architecture Overview

supastarter implements a **multi-container separation strategy** deployed on Fly.io:

```
┌─────────────────────────────────────────────────────────────┐
│                         Fly.io Edge                          │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐     ┌──────────────┐    ┌──────────────┐ │
│  │   Frontend   │     │   API Main   │    │   API GPU    │ │
│  │   (Next.js)  │     │  (FastAPI)   │    │  (FastAPI)   │ │
│  │   Public     │     │   Private    │    │   Private    │ │
│  │ Min: 1 Run   │     │ Scale to 0   │    │ Scale to 0   │ │
│  └──────────────┘     └──────────────┘    └──────────────┘ │
│         ↓                    ↓                    ↓         │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              Supabase (External Service)               │ │
│  │         - Authentication (Auth.users table)            │ │
│  │         - Row Level Security (RLS)                     │ │
│  │         - Real-time subscriptions                      │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Configuration

supastarter is highly configurable with environment-based settings that support both development and production deployments. The architecture supports complete customization of:

- **Security policies** and authentication flows
- **Container scaling** and resource allocation  
- **Database schemas** with automatic RLS policy generation
- **API endpoints** with auto-generated OpenAPI documentation
- **Frontend styling** with full design system control

Learn more in the [Configuration](Configuration.md) section.

Features
This is just a quick overview of the features and pages included in supastarter. You can learn all about the included features in the features section. To find out about how to use them, head to the according documentation pages.

Marketing site
Home


Features


Pricing




Contact


Footer


SaaS application
Authentication
Login


Signup


Forgot password


User
User dashboard


Account Settings
General


Security


Billing


Organizations
Organization dashboard


Organization settings
General


Organization members


Administration
Users


Organizations


Next

Tech Stack

© 2025 supastarter. All rights reserved.

Featured on Startup Fame




