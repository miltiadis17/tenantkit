# Contributing to TenantKit

Thanks for your interest! This project aims to provide a minimal, production-minded starter for tenant-aware RBAC on Spring Boot 3 / Java 17.

## Quick Start (Dev)
- Requirements: JDK 17+, Maven 3.9+, Docker (MySQL/Testcontainers), Node 20+ (optional for tiny admin)
- DB: `docker compose up -d` (MySQL 8.4)
- Backend: `mvn spring-boot:run` (or `mvn -q verify` to run tests)
- Frontend (optional): `pnpm i && pnpm dev` or `npm i && npm run dev`

## Issues
- Use the template (Context / Scope / Acceptance criteria / Testing / DoD)
- Keep tasks focused; one feature per issue.
- Tag appropriately: `security`, `backend`, `db`, `tests`, `docs`, `good first issue`.

## Branching & Commits
- Default branch: `main` (protected)
- Create topic branches: `feature/<short-name>-#<issue>`, `fix/<short-name>-#<issue>`
- Prefer Conventional Commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`
- Link the issue in your PR description: **Closes #<issue-number>**

## Code Style & Quality
- Java 17, Spring Boot 3, JPA/Hibernate
- Keep controllers thin; push rules to services
- Add tests for security/tenant isolation
- Keep secrets out of code; use env vars
- If adding a new public API, update OpenAPI docs

## Tests
- Unit tests: JUnit 5
- Integration tests: Spring Boot Test + Testcontainers (MySQL 8.4)
- Recommended command: `mvn -q -DskipITs=false verify`

## Pull Requests
- Keep PRs small & cohesive
- Include: motivation, changes list, testing notes, screenshots for UI
- CI must pass
- PR template is applied automatically; fill the checklist

## License
By contributing, you agree that your contributions are licensed under the project’s MIT license.
