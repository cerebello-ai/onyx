# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Onyx is an open-source enterprise search and Gen-AI platform that connects to company documents, apps, and people. It features a chat interface with pluggable LLM support and maintains synchronized knowledge across 40+ connectors.

## Key Commands

### Backend Development
```bash
# Setup Python virtual environment (Python 3.11 required)
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r backend/requirements/default.txt
pip install -r backend/requirements/dev.txt

# Run database migrations
alembic -c backend/alembic.ini upgrade head

# Start backend API server
python backend/onyx/main.py

# Run tests
pytest backend/tests/unit/
pytest backend/tests/integration/
pytest backend/tests/daily/  # Connector tests

# Type checking
mypy backend/

# Code formatting (pre-commit hooks)
black backend/
reorder-python-imports backend/**/*.py
```

### Frontend Development
```bash
# Install dependencies
cd web
npm install

# Development server with hot reload
npm run dev

# Build for production
npm run build

# Run linting
npm run lint

# Type checking
npm run type-check
```

### Docker Development
```bash
# Start all services
docker compose -f docker-compose.dev.yml up -d

# View logs
docker compose -f docker-compose.dev.yml logs -f api_server
docker compose -f docker-compose.dev.yml logs -f web_server

# Rebuild after changes
docker compose -f docker-compose.dev.yml up -d --build
```

## Architecture Overview

### Backend Structure
- **API Server**: FastAPI application at `backend/onyx/main.py` handling REST endpoints
- **Background Jobs**: Celery workers in `backend/onyx/background/` for async processing
- **Document Pipeline**: `backend/onyx/indexing/` handles document ingestion and embedding
- **Search Engine**: `backend/onyx/context/search/` implements hybrid vector + keyword search
- **Chat System**: `backend/onyx/chat/` manages LLM interactions with streaming support
- **Agent System**: `backend/onyx/agents/` provides advanced research capabilities using LangGraph
- **Connectors**: `backend/onyx/connectors/` contains 40+ data source integrations

### Frontend Structure
- **Next.js App**: Located in `web/` with TypeScript and React
- **API Client**: `web/src/lib/` contains API communication logic
- **Components**: `web/src/components/` has reusable UI components
- **Chat Interface**: `web/src/app/chat/` implements the main chat UI
- **Admin Dashboard**: `web/src/app/admin/` provides system administration

### Database Architecture
- **PostgreSQL**: Primary database for metadata, users, and configuration
- **Vespa**: Vector database for document storage and similarity search
- **Redis**: Caching layer and Celery message broker
- **MinIO**: S3-compatible object storage for files

### Key Integration Points
- **LLM Providers**: Configurable via LiteLLM in `backend/onyx/llm/`
- **Authentication**: FastAPI-Users with multiple backends in `backend/onyx/auth/`
- **Connector Framework**: Standard interfaces in `backend/onyx/connectors/interfaces.py`
- **Search Pipeline**: Combines vector embeddings with keyword matching for hybrid search
- **Background Task Coordination**: Celery beat scheduler manages periodic indexing

## Critical Workflows

### Document Indexing Flow
1. Connector fetches documents from source
2. Documents are chunked and processed in `backend/onyx/indexing/chunker.py`
3. Embeddings generated via model server
4. Chunks stored in Vespa with metadata
5. Access permissions synchronized

### Chat Request Flow
1. User message received at API endpoint
2. Context retrieved via hybrid search
3. LLM generates response with citations
4. Response streamed back to frontend
5. Conversation persisted to database

### Connector Development
- Implement interfaces in `backend/onyx/connectors/interfaces.py`
- Add connector config in `backend/onyx/connectors/factory.py`
- Create tests in `backend/tests/daily/connectors/`
- Update frontend form in `web/src/lib/connectors/`

## Testing Approach

- Unit tests focus on individual components
- Integration tests verify cross-service functionality
- Daily tests ensure connector reliability
- Use pytest fixtures for test data
- Mock external services in tests

## Important Considerations

- Always check existing connector implementations before creating new ones
- Follow SQLAlchemy patterns for database operations
- Use type hints throughout Python code
- Maintain backward compatibility for API changes
- Test with both PostgreSQL and Vespa running
- Consider access control implications for new features
- Use streaming for large responses to improve UX