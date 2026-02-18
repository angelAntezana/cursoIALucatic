<!--
  SYNC IMPACT REPORT
  ==================
  Version: 0.0.0 → 1.0.0 (MAJOR - Initial ratification)
  Date: 2026-02-17

  Modified Principles:
  - NEW: I. API-First Architecture
  - NEW: II. Security by Design
  - NEW: III. Test-Driven Quality
  - NEW: IV. Data Integrity & Migrations
  - NEW: V. Local-First Development
  - NEW: VI. Clear Separation of Concerns
  - NEW: VII. Pragmatic Simplicity

  Added Sections:
  - Technology Standards
  - Development Workflow

  Templates Status:
  ✅ plan-template.md - Constitution Check section aligned with principles
  ✅ spec-template.md - Requirements section aligned with security/quality principles
  ✅ tasks-template.md - Task categorization reflects test-first and separation principles
  ⚠️  AGENTS.md - Already contains detailed guidelines; constitution references it

  Follow-up TODOs:
  - None. All critical placeholders filled based on REQUISITOS.md and project context.
-->

# GestionDocumentos Constitution

## Core Principles

### I. API-First Architecture

The backend MUST expose all functionality through a well-defined REST API. Frontend applications MUST consume only documented API endpoints.

**Rules**:
- Backend defines clear API contracts with request/response schemas
- All endpoints documented (OpenAPI/Swagger) at `/swagger-ui`
- Frontend never bypasses API to access database or file system directly
- API versioning via URL path (`/api/v1/`) for future compatibility

**Rationale**: Clear API boundaries enable independent frontend/backend development, testing, and future mobile app additions without backend changes.

### II. Security by Design

Security MUST be built into every layer, not added afterward. All authentication, authorization, input validation, and file handling MUST follow secure practices.

**Rules**:
- Passwords MUST use BCrypt hashing (never plain text)
- Authentication via JWT with configurable expiration (15-30 minutes)
- Authorization MUST filter all queries by `user_id` to prevent data leakage
- Input validation using Bean Validation on all endpoints
- File uploads MUST validate MIME type, extension (pdf|png|jpg|jpeg), and size (≤10MB)
- File storage paths MUST be outside web root with UUIDs (never user-supplied names)
- Secrets MUST NOT be committed (use `application-local.yml` in `.gitignore`)
- CORS restricted to configured origins only

**Rationale**: Security vulnerabilities are costly to fix post-deployment. Building security in from the start protects user data and reduces risk.

### III. Test-Driven Quality

Tests MUST be written for all critical paths. Backend and frontend test suites MUST pass before any merge.

**Rules**:
- Backend: JUnit 5 tests in `backend/src/test/java` mirroring package structure
- Frontend: Vitest/Vue Test Utils specs next to components (`ComponentName.spec.ts`)
- Test naming describes behavior: `shouldExtractExpiryDateWhenTextPresent`
- Contract tests MUST verify API endpoint behavior matches documented contracts
- Integration tests MUST cover critical user journeys (upload → classify → alert)
- CI/CD pipeline MUST run `./mvnw test` and `npm run test:unit` before merge

**Rationale**: Tests catch regressions early, document expected behavior, and enable confident refactoring. Test failures signal broken assumptions.

### IV. Data Integrity & Migrations

Database schema changes MUST be versioned and applied via migrations. Referential integrity MUST be enforced at the database level.

**Rules**:
- All schema changes via Flyway migrations in `backend/src/main/resources/db/migration/`
- Foreign keys MUST use proper constraints with cascades where appropriate
- Transactions MUST be used for multi-table operations
- `snake_case` for database columns, `camelCase` for Java fields
- Database initialization steps MUST be documented in `docs/README.md`
- Never use `hibernate.ddl-auto: create` or `update` in production-like environments

**Rationale**: Versioned migrations enable reproducible database state across environments. Database constraints prevent orphaned records and data corruption.

### V. Local-First Development

All MVP features MUST run entirely on the developer's local machine. No external services or cloud dependencies allowed in MVP.

**Rules**:
- Backend, frontend, and PostgreSQL MUST run locally
- File storage MUST use local filesystem (e.g., `./_storage` folder)
- No external APIs for OCR, email, SMS, or cloud storage in MVP
- Configuration for local development in `application-local.yml` and `frontend/.env.local`
- Docker Compose MAY be used for local PostgreSQL but is optional
- Document all local setup steps in project README

**Rationale**: Local development maximizes developer productivity, eliminates network dependencies, and keeps the MVP scope focused and achievable.

### VI. Clear Separation of Concerns

Code MUST be organized by responsibility with clear boundaries between layers and modules.

**Rules**:
- **Backend**: Controllers (thin) → Services (business logic) → Repositories (data access)
- **Backend packages**: `controller`, `service`, `domain`, `repository`, `dto`, `config`, `security`
- **Frontend**: Components (presentation) → Services/Stores (state/API) → Views (composition)
- **Frontend structure**: `src/components/`, `src/views/`, `src/stores/`, `src/api/`, `src/router/`
- Domain logic MUST NOT leak into controllers or components
- Cross-cutting concerns (error handling, logging) via Spring `@ControllerAdvice` and global Vue error handlers
- Shared utilities in `backend/src/main/java/.../util` and `frontend/src/utils`

**Rationale**: Separation of concerns makes code maintainable, testable, and allows team members to work independently on different layers.

### VII. Pragmatic Simplicity

Start simple and add complexity only when clearly justified. YAGNI (You Aren't Gonna Need It) principles MUST guide all design decisions.

**Rules**:
- Implement only features explicitly required for MVP (see `REQUISITOS.md`)
- Avoid premature abstractions (no repositories pattern unless needed, no microservices)
- Avoid over-engineering (no feature flags, no complex caching in MVP)
- No "nice to have" features that don't directly support core user scenarios
- Document complexity justifications in implementation plans
- Prefer standard Spring Boot and Vue 3 patterns over custom frameworks

**Rationale**: Over-engineering wastes time, adds bugs, and obscures intent. Simple code is easier to understand, test, and maintain.

## Technology Standards

### Required Stack (MVP)
- **Backend**: Java 21, Spring Boot 3.3+, Maven
- **Frontend**: Vue 3, Vite, TypeScript (recommended)
- **Database**: PostgreSQL 14+
- **Authentication**: JWT (JJWT library)
- **Migrations**: Flyway
- **Testing**: JUnit 5 (backend), Vitest/Vue Test Utils (frontend)
- **Documentation**: SpringDoc OpenAPI

### Coding Standards
- **Java**: 4-space indentation, `camelCase` methods, `PascalCase` classes, `snake_case` DB columns
- **Vue**: 2-space indentation, `PascalCase` component filenames, `kebab-case` props/events
- **REST paths**: lowercase with hyphens (`/documents/expiry`)
- **DTOs**: prefix with purpose (e.g., `DocumentMetadataDTO`, `UploadRequestDTO`)
- **Formatting**: `./mvnw spotless:apply` (if configured) and `npm run lint` before commits

### Build Commands
- Backend build: `./mvnw clean install`
- Backend run: `./mvnw spring-boot:run`
- Backend tests: `./mvnw test`
- Frontend install: `npm install` (in `frontend/`)
- Frontend dev: `npm run dev`
- Frontend lint: `npm run lint`
- Frontend tests: `npm run test:unit`

## Development Workflow

### Commit Guidelines
- MUST follow Conventional Commits format:
  - `feat(backend): add document classification endpoint`
  - `fix(frontend): handle empty file uploads`
  - `docs: update PostgreSQL setup instructions`
  - `test(backend): add integration tests for alerts`
- Commits MUST be atomic and focused on a single change
- MUST run tests before committing

### Pull Request Requirements
- All tests MUST pass (`./mvnw test` and `npm run test:unit`)
- Code MUST be formatted (`./mvnw spotless:check` and `npm run lint`)
- PR description MUST include:
  - Brief summary of changes
  - Linked issue/task (if any)
  - Manual testing steps (e.g., "Requires PostgreSQL with schema version V3")
- UI changes MUST include screenshots for reviewer verification
- PRs MUST be tagged as "Ready for review" only after all checks pass

### Code Review Standards
- Reviewers MUST verify:
  - Security: No credentials committed, proper validation, authorization checks
  - Tests: Critical paths covered, tests actually test something meaningful
  - Architecture: Follows separation of concerns, no domain logic in controllers
  - Complexity: Justified per Principle VII, no premature abstractions
- Reviews MUST reference specific principles from this constitution when giving feedback
- Reviewers MUST check that changes align with REQUISITOS.md scope

### Configuration Management
- Secrets and local config MUST be in `.gitignore`:
  - `backend/src/main/resources/application-local.yml`
  - `frontend/.env.local`
- Shared configuration templates MAY be committed with placeholder values
- PostgreSQL initialization steps MUST be documented in `docs/README.md`

## Governance

This constitution supersedes all other development practices and guidelines. When conflicts arise, constitution principles take precedence.

### Amendment Process
1. Amendments MUST be proposed with clear rationale
2. Changes MUST be reviewed by project stakeholders
3. Approved amendments MUST increment version per semantic versioning:
   - **MAJOR**: Principle removals, redefinitions, or backward-incompatible governance changes
   - **MINOR**: New principles added or significant expansions
   - **PATCH**: Clarifications, wording improvements, non-semantic fixes
4. Amendment MUST include migration plan if it affects existing code
5. Amendment date MUST be recorded in "Last Amended" field

### Compliance & Enforcement
- All pull requests MUST be reviewed for constitutional compliance
- Complexity violations MUST be explicitly justified in implementation plans (see `plan-template.md` Complexity Tracking section)
- Non-compliant code MUST be rejected or refactored before merge

### Related Documentation
- This constitution defines the **"what"** and **"why"** (immutable principles)
- `AGENTS.md` provides the **"how"** (practical implementation guidelines that may evolve)
- When conflicts arise between this constitution and `AGENTS.md`, constitution takes precedence
- `AGENTS.md` SHOULD be updated to align with any constitutional amendments

### Review Schedule
- Constitution MUST be reviewed after:
  - Major architectural changes
  - Technology stack upgrades
  - Post-MVP feature additions that require new principles
  - Security incidents or audit findings
- Review frequency: At minimum, every 6 months or after 10+ merged features

**Version**: 1.0.0 | **Ratified**: 2026-02-17 | **Last Amended**: 2026-02-17
