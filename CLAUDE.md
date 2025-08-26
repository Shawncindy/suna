# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Suna (Kortix) is an open-source platform for building, managing, and training autonomous AI agents. The codebase consists of a full-stack application with React/Next.js frontend, Python/FastAPI backend, and containerized agent execution environments.

## Common Development Commands

### Quick Start
```bash
# Initial setup with wizard
python setup.py

# Start entire platform
python start.py

# Stop platform
python start.py  # (same command toggles start/stop)
```

### Frontend Development
```bash
cd frontend
npm run dev          # Start development server with Turbopack
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
npm run format       # Format code with Prettier
npm run format:check # Check formatting
```

### Backend Development
```bash
cd backend
uv run api.py                    # Start FastAPI server
uv run dramatiq run_agent_background  # Start worker process

# Testing
pytest                          # Run tests
pytest-asyncio                  # Run async tests
```

### Docker Operations
```bash
docker compose up -d            # Start all services
docker compose down             # Stop all services
docker compose logs -f          # View logs
docker compose up redis -d      # Start just Redis
```

### Database Operations
```bash
# Supabase CLI operations
supabase start                  # Start local Supabase
supabase db push               # Push migrations
supabase link --project-ref <ref>  # Link to project
```

## Architecture Overview

### Core Components
- **Frontend**: Next.js 15+ with App Router, TypeScript, Tailwind CSS, Radix UI
- **Backend**: Python 3.11+ with FastAPI, Supabase integration, Redis caching
- **Agent Runtime**: Docker containers with browser automation, code interpreter
- **Database**: PostgreSQL via Supabase with Row Level Security (RLS)
- **Authentication**: Supabase Auth with JWT validation

### Key Directory Structure
```
├── frontend/                   # Next.js React application
│   ├── src/app/               # App Router pages and API routes
│   ├── src/components/        # Reusable React components
│   ├── src/hooks/             # Custom React hooks with React Query
│   ├── src/lib/               # Utilities, API clients, stores
│   └── src/providers/         # Context providers and wrappers
├── backend/                   # Python FastAPI backend
│   ├── agent/                 # Core AI agent implementation
│   ├── tools/                 # Agent tool implementations
│   ├── services/              # Business logic services
│   ├── models/                # Data models and database schemas
│   ├── supabase/migrations/   # Database migration files
│   └── api.py                 # Main FastAPI application entry
├── sdk/                       # Python SDK for agent development
└── docs/                      # Documentation files
```

### Agent System Architecture
- **Agent Builder**: Visual tools for configuring custom agents
- **Versioning System**: Multiple agent versions stored in `agent_versions` table
- **Tool Integration**: Dual-schema decorators (OpenAPI + XML) for tool definitions
- **Workflow Engine**: Step-by-step execution with conditional logic
- **Trigger System**: Scheduled and event-based automation
- **MCP Integration**: Model Context Protocol for extending functionality

### Technology Stack Specifics
- **State Management**: Zustand stores, React Query for server state
- **UI Framework**: Radix UI components with Tailwind CSS
- **Database ORM**: Direct Supabase client, no traditional ORM
- **Task Queue**: Dramatiq for background job processing
- **LLM Integration**: LiteLLM for multi-provider support (Anthropic, OpenAI, etc.)
- **Real-time**: Supabase subscriptions for live updates

## Development Patterns

### Error Handling
- Frontend: React Error Boundaries, structured error responses
- Backend: FastAPI exception handlers, structured logging with structlog
- Always use proper HTTP status codes and consistent error formats

### Authentication Flow
- Supabase Auth JWT tokens validated on backend
- Row Level Security (RLS) policies enforce database access
- No signature verification - trusts Supabase token validation

### Database Patterns
- All migrations are idempotent SQL with proper error handling
- Use proper indexing on foreign keys and frequently queried columns
- Encrypted storage for sensitive data (API keys, credentials)

### Tool Development
- Tools use dual decorators: `@tool_openapi_schema` and `@tool_xml_schema`
- All tools return `ToolResult` objects with structured data
- Sandbox isolation for all agent code execution

### Testing Strategy
- Frontend: Component testing with Jest/Testing Library
- Backend: pytest for unit tests, pytest-asyncio for async code
- Integration tests for critical API endpoints
- E2E tests for complete user workflows

## Environment Configuration

### Required Environment Variables
- **Database**: `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`
- **LLM Providers**: `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `OPENROUTER_API_KEY`
- **Agent Execution**: `DAYTONA_API_KEY`, `DAYTONA_SERVER_URL`
- **Search/Scraping**: `TAVILY_API_KEY`, `FIRECRAWL_API_KEY`
- **Caching**: Redis connection details
- **Security**: `MCP_CREDENTIAL_ENCRYPTION_KEY` for credential encryption

### Setup Process
1. Run `python setup.py` - comprehensive wizard with progress saving
2. Links Supabase project and pushes migrations
3. Configures all required API keys and services
4. Sets up Docker environments and dependencies
5. Can resume from interruption using saved progress

## Key Services & Integrations

### Agent Execution (Daytona)
- Secure sandbox environments for agent code execution
- Pre-configured with browser automation, file system access
- Snapshot: `kortix/suna:0.1.3.11` with supervisord entrypoint

### LLM Integration (LiteLLM)
- Multi-provider support with unified interface
- Model pricing and capability management
- Structured output and function calling support

### Background Jobs (Dramatiq)
- Redis-backed task queue for agent execution
- Worker processes handle long-running agent tasks
- Monitoring and error handling for job failures

## Performance Considerations

### Frontend Optimization
- Next.js 15+ with Turbopack for fast development builds
- Code splitting and lazy loading for large components
- React Query for efficient server state management and caching

### Backend Performance
- FastAPI with async/await patterns throughout
- Redis caching for frequently accessed data
- Connection pooling for database operations
- Proper indexing and query optimization

### Agent Execution
- Docker container resource limits and timeouts
- Cleanup procedures for stopped containers
- Monitoring and health checks for execution environments

## Security Best Practices

### Data Protection
- All sensitive credentials encrypted at rest
- Row Level Security (RLS) policies on all database tables
- Input validation using Pydantic models
- No secrets in logs or error messages

### Agent Sandboxing
- Isolated Docker environments for each agent execution
- Limited network access and filesystem permissions
- Automatic cleanup of temporary files and containers