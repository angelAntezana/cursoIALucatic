# Repository Guidelines

## Project Structure & Module Organization
- `backend/` (Spring Boot) hosts REST controllers, services, and `src/main/resources/application-local.yml`; keep domain logic separated under `service` and `domain` packages and reference the `storage/` folder for downloaded files.
- `frontend/` (Vue 3) holds `src/` modules, `components/`, and `views/`; pair each view with a scoped CSS file and store shared assets under `frontend/assets/`.
- `docs/` is reserved for planning notes like `REQUISITOS.md` and future runbooks; keep supplementary research in `docs/brainstorming/` to avoid cluttering the root.
- `scripts/` or `ops/` (create as needed) should contain helpers such as `init-db.sh` or `load-fixtures.sql` so scripts are discoverable.

## Build, Test, and Development Commands
- `./mvnw clean install` runs Java formatting, compiles the backend, and packages artifacts.
- `./mvnw spring-boot:run` boots the backend locally (uses the local PostgreSQL config in `application-local.yml`).
- `npm install` then `npm run dev` inside `frontend/` starts the hot-reload Vue app against `http://localhost:8080`.
- `npm run lint` (from `frontend/`) enforces ESLint/Prettier rules before commits.
- `npm run test:unit` runs Vue Test Utils suites; `./mvnw test` executes backend JUnit coverage.

## Coding Style & Naming Conventions
- Java uses 4-space indentation; keep Spring components in `camelCase` with PascalCase class names and `snake_case` database columns.
- Vue components use 2-space indentation, `PascalCase` filenames, and kebab-case props/events.
- Prefer `application` or `document` prefixes when naming DTOs (e.g., `DocumentMetadataDTO`), and keep REST paths lowercase with hyphens (`/documents/expiry`).
- Run `./mvnw spotless:apply` (if configured) and `npm run lint` before pushing.

## Testing Guidelines
- Backend tests rely on Spring Boot + JUnit 5; keep tests under `backend/src/test/java` mirroring package structure.
- Frontend uses Vitest/Vue Test Utils; place specs next to components (`MyCard.spec.ts`).
- Name tests to describe behavior (`shouldExtractExpiryDateWhenTextPresent`).
- Seed PostgreSQL fixtures via scripts in `scripts/` and run `./mvnw test`/`npm run test:unit` in CI.

## Commit & Pull Request Guidelines
- Follow Conventional Commits: `feat(backend): add document classification`, `fix(frontend): handle empty uploads`.
- Open PRs only after passing both test suites; include a short description, linked issue (if any), and mention manual steps (e.g., `Requires local PostgreSQL with schema X`).
- Attach screen captures if UI changes affect the dashboard/alerts so reviewers can verify styling.
- Tag reviewers explicitly and add `Ready for review` once backend and frontend builds succeed.

## Security & Configuration Tips
- Keep secrets out of Git: store credentials in `backend/src/main/resources/application-local.yml` (ignored) and `frontend/.env.local`; do not commit these files.
- Document Postgres initialization steps in `docs/README.md` so other contributors can mirror the local setup.
