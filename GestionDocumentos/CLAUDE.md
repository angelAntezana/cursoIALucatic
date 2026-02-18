# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GestionDocumentos** is a local-first web application for managing personal documents (IDs, bills, contracts, permits, insurance). It helps families and freelancers organize critical documents with automatic classification, expiration date extraction, and expiration alerts.

**Current Status**: Planning phase. Implementation pending.

**Target Architecture**:
- Backend: Java 21 + Spring Boot 3 (Maven)
- Frontend: Vue 3 + Vite (TypeScript recommended)
- Database: PostgreSQL 14+
- Authentication: JWT
- Storage: Local filesystem (`./_storage` directory)

## Core Development Principles

This project follows a constitution-based development approach. All code MUST comply with these principles defined in `.specify/memory/constitution.md`:

1. **API-First Architecture**: Backend exposes REST API (`/api/v1/`), frontend consumes only documented endpoints
2. **Security by Design**: BCrypt passwords, JWT auth, input validation, MIME type verification, file size limits (≤10MB)
3. **Test-Driven Quality**: JUnit 5 (backend) and Vitest (frontend) tests required before merge
4. **Data Integrity & Migrations**: Flyway migrations for all schema changes, foreign key constraints enforced
5. **Local-First Development**: Everything runs locally (no cloud dependencies in MVP)
6. **Clear Separation of Concerns**: Controllers → Services → Repositories (backend); Components → Stores → Views (frontend)
7. **Pragmatic Simplicity**: YAGNI - implement only features in REQUISITOS.md, avoid over-engineering

## Key Documentation Files

**Read these files to understand the project**:
- `REQUISITOS.md` - MVP requirements and functional scope (Spanish)
- `Braninstorming.md` - Detailed technical specification with API contracts, data models, and curl examples
- `AGENTS.md` - Development guidelines for code structure, testing, commits, and security
- `.specify/memory/constitution.md` - Immutable project principles that supersede all other guidelines

## Development Commands

### Backend (once created in `backend/`)
```bash
# Build and run tests
./mvnw clean install

# Run backend (default: http://localhost:8080)
./mvnw spring-boot:run

# Run tests only
./mvnw test

# Format code (if Spotless configured)
./mvnw spotless:apply
```

### Frontend (once created in `frontend/`)
```bash
# Install dependencies
npm install

# Run dev server (default: http://localhost:5173)
npm run dev

# Lint and format
npm run lint

# Run tests
npm run test:unit
```

### Database Setup
```bash
# Option 1: Docker Compose (PostgreSQL only)
docker compose up -d

# Option 2: Local PostgreSQL
# Create database and user as specified in backend/src/main/resources/application.yml:
# Database: docsguard
# User: docsguard
# Password: (configured in application-local.yml)
```

## API Documentation

Once backend is running, access interactive API docs at:
- Swagger UI: `http://localhost:8080/swagger-ui`
- Health check: `http://localhost:8080/actuator/health`

## Project Structure (Future)

```
GestionDocumentos/
├── backend/                    # Spring Boot application
│   ├── src/main/java/com/example/docsguard/
│   │   ├── controller/        # REST endpoints (thin)
│   │   ├── service/           # Business logic
│   │   ├── domain/            # JPA entities
│   │   ├── repository/        # Data access
│   │   ├── dto/               # Data transfer objects
│   │   ├── config/            # Spring configuration
│   │   └── security/          # JWT filters, SecurityConfig
│   ├── src/main/resources/
│   │   ├── application.yml    # Main config
│   │   └── db/migration/      # Flyway SQL migrations
│   └── src/test/java/         # JUnit tests (mirror package structure)
├── frontend/                   # Vue 3 application
│   ├── src/
│   │   ├── components/        # Reusable Vue components
│   │   ├── views/             # Page-level components
│   │   ├── stores/            # Pinia state management
│   │   ├── api/               # Axios API clients
│   │   └── router/            # Vue Router configuration
│   └── .env.local             # Local config (VITE_API_BASE)
├── _storage/                   # Local file storage (gitignored)
├── docs/                       # Additional documentation
└── REQUISITOS.md               # Project requirements
```

## Coding Standards

### Java (Backend)
- 4-space indentation
- `camelCase` for methods/variables
- `PascalCase` for classes
- `snake_case` for database columns
- DTOs prefixed with purpose: `DocumentMetadataDTO`, `UploadRequestDTO`
- REST paths lowercase with hyphens: `/documents/expiry`

### Vue/TypeScript (Frontend)
- 2-space indentation
- `PascalCase` for component filenames: `UploadDialog.vue`
- `kebab-case` for props and events
- Scoped CSS per component

### Testing Standards
- **Backend**: Tests in `src/test/java` mirroring package structure
- **Frontend**: Specs next to components: `MyCard.spec.ts`
- Test names describe behavior: `shouldExtractExpiryDateWhenTextPresent`
- All tests MUST pass before commits

## Commit Conventions

Follow Conventional Commits format:
- `feat(backend): add document classification endpoint`
- `fix(frontend): handle empty file uploads`
- `docs: update PostgreSQL setup instructions`
- `test(backend): add integration tests for alerts`

## Security Checklist

Before committing, verify:
- [ ] No credentials committed (check `application-local.yml`, `.env.local` are in `.gitignore`)
- [ ] All user inputs validated (Bean Validation annotations)
- [ ] File uploads validate MIME type, extension, and size
- [ ] JWT secret is not hardcoded
- [ ] All queries filter by `user_id` (authorization)
- [ ] CORS configured for allowed origins only

## Speckit Workflow

This project uses the Speckit framework for specification-driven development. Available skills:

- `/speckit.specify` - Create or update feature specifications
- `/speckit.clarify` - Identify underspecified areas and ask clarifying questions
- `/speckit.plan` - Execute implementation planning workflow
- `/speckit.tasks` - Generate actionable, dependency-ordered tasks
- `/speckit.implement` - Execute the implementation plan
- `/speckit.analyze` - Cross-artifact consistency analysis
- `/speckit.constitution` - Create or update project constitution

## MVP Feature Scope

**IN SCOPE** (from REQUISITOS.md):
1. Document upload (PDF, PNG, JPG) with automatic classification
2. Date extraction (expiration, issuance) from documents
3. Expiration alerts (UI only, no emails)
4. Dashboard with document list and filters
5. Metadata viewing and editing
6. Reminder creation and viewing

**OUT OF SCOPE** (MVP):
- Cloud storage or synchronization
- External integrations (email, SMS, external APIs)
- Advanced authentication (only JWT)
- Multi-user or multi-device support
- Real OCR/AI (basic text extraction only)
- Advanced features like cofres, signatures, export

## API Contracts Overview

Base URL: `http://localhost:8080/api/v1`

**Authentication**:
- `POST /auth/register` - User registration
- `POST /auth/login` - Returns JWT access token

**Documents**:
- `POST /documents` - Upload document (multipart/form-data)
- `GET /documents` - List with filters (query, type, date range, pagination)
- `GET /documents/{id}` - Get document details
- `GET /documents/{id}/content` - Download file
- `DELETE /documents/{id}` - Delete document
- `GET /documents/{id}/metadata` - Get metadata
- `PUT /documents/{id}/metadata` - Update metadata

**Reminders**:
- `POST /documents/{id}/reminders` - Create reminder
- `GET /documents/{id}/reminders` - List reminders
- `PATCH /reminders/{id}` - Update reminder status

All endpoints require `Authorization: Bearer <token>` except auth endpoints.

## Data Model (PostgreSQL)

**Core Tables**:
- `app_user` - User accounts (id, email, password_hash)
- `document` - Document records (id, user_id, filename, mime, storage_path, checksum)
- `doc_metadata` - Document metadata (document_id, type, issuer, person, date_issue, date_expiry, amount, tags JSONB)
- `reminder` - Document reminders (id, document_id, due_date, channel, status)

Foreign keys with CASCADE deletes. Indexes on `user_id`, `type`, and `date_expiry`.

## Local Development Setup

1. **Prerequisites**: Java 21, Node.js 18+, PostgreSQL 14+, Maven
2. **Create storage directory**: `mkdir _storage`
3. **Start PostgreSQL** (Docker or local installation)
4. **Configure backend**: Copy `application.yml` to `application-local.yml` with local credentials
5. **Run backend**: `./mvnw spring-boot:run` in `backend/`
6. **Install frontend deps**: `npm install` in `frontend/`
7. **Configure frontend**: Create `frontend/.env.local` with `VITE_API_BASE=http://localhost:8080/api/v1`
8. **Run frontend**: `npm run dev` in `frontend/`
9. **Verify**: Access http://localhost:5173 and test registration/login

## When in Doubt

1. Check if the feature is in `REQUISITOS.md` - if not, it's out of MVP scope
2. Verify compliance with constitution principles in `.specify/memory/constitution.md`
3. Follow guidelines in `AGENTS.md` for code structure and testing
4. Refer to `Braninstorming.md` for detailed API contracts and examples
5. When conflicts arise, constitution principles take precedence
