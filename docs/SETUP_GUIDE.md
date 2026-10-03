# Setup Guide

## Prerequisites

### Option A — Docker Compose only (recommended, fastest)
- Docker Engine 24+ and the Docker Compose plugin (`docker compose version`).

### Option B — Local development (no Docker for app services)
- Docker (for Postgres only, via `docker compose up postgres -d`) **or** a locally installed
  PostgreSQL 15+.
- Java 21 (JDK) + Maven (or use the bundled `./mvnw` wrapper, no separate Maven install needed).
- Node.js 20+ and npm 10+.

---

## Path 1 — Docker Compose (everything in containers)

1. Clone the repo and `cd` into it.
2. Copy the environment template (optional — the compose file already has sane dev defaults
   baked in via `${VAR:-default}` syntax, but copying lets you override anything):
   ```bash
   cp .env.example .env
   ```
3. Start everything:
   ```bash
   ./scripts/start.sh
   # equivalent to: docker compose up --build -d
   ```
   This builds the `backend` and `frontend` images, starts `postgres` first and waits for its
   healthcheck, then starts `backend` (which runs Flyway migrations `V1__init_schema.sql` and
   `V2__seed_data.sql` automatically on boot), then `frontend`.
4. Open:
   - Frontend: http://localhost:3000
   - Swagger UI: http://localhost:8080/swagger-ui.html
5. Log in with any seeded credential, e.g. `admin@greenvalley.com` / `Password123!` (see the full
   table in the root [README.md](../README.md#default-seed-credentials)).
6. View logs: `docker compose logs -f backend` (or `frontend`, `postgres`).
7. Stop everything: `./scripts/stop.sh` (data persists in the `pgdata` volume). To wipe the
   database too: `docker compose down -v`.

## Path 2 — Local development (hot reload, no Docker for app code)

1. Start only Postgres in Docker:
   ```bash
   docker compose up postgres -d
   ```
2. In one terminal, start the backend:
   ```bash
   ./scripts/dev-backend.sh
   # equivalent to: cd backend && ./mvnw spring-boot:run
   # (reads .env if present; Flyway runs V1 + V2 against the dockerized Postgres automatically)
   ```
   Backend will be available at http://localhost:8080.
3. In another terminal, start the frontend:
   ```bash
   ./scripts/dev-frontend.sh
   # equivalent to: cd frontend && npm install && npm run dev
   ```
   Frontend will be available at http://localhost:3000, calling the backend via
   `NEXT_PUBLIC_API_URL` (default `http://localhost:8080/api/v1`).
4. Both processes support hot-reload: Spring Boot DevTools (if enabled) restarts on backend code
   changes; Next.js Fast Refresh updates the browser on frontend code changes.

### Running backend tests only

```bash
cd backend
./mvnw test
```

### Running the Playwright E2E suite

The E2E suite lives in `e2e/` (independent of `frontend/`) and expects the full stack already
running (either path above) with seed data loaded:

```bash
cd e2e
npm install
npx playwright install --with-deps chromium   # first time only
npm test
```

Useful variants:
- `npm run test:headed` — see the browser while tests run.
- `npm run test:ui` — Playwright's interactive UI mode.
- `E2E_BASE_URL=http://localhost:3000 npm test` — override the base URL if the frontend runs
  somewhere else.

---

## Troubleshooting

**Port already in use (5432 / 8080 / 3000)**
Another process (or a previous `docker compose up`) is already bound to that port. Either stop
that process, or override the port in `.env` (`POSTGRES_PORT`, `BACKEND_PORT`, `FRONTEND_PORT`)
before re-running `docker compose up`.

**Backend can't connect to the database (`Connection refused` / `FATAL: password authentication failed`)**
- Docker Compose path: make sure `postgres` is healthy first — `docker compose ps` should show
  `healthy` for the `postgres` service before `backend` starts. If you see auth failures, your
  `.env` `POSTGRES_PASSWORD` likely doesn't match what the volume was initialized with; the fix
  is `docker compose down -v` (wipes the volume) then `docker compose up --build` again so Postgres
  re-initializes with the current `.env` values.
- Local dev path: confirm `docker compose up postgres -d` is actually running
  (`docker compose ps`), and that `DB_URL` in your shell/`.env` points at
  `localhost` (not `postgres` — that hostname only resolves *inside* the compose network).

**Flyway migration checksum mismatch (`FlywayValidateException` / "Migration checksum mismatch for migration version 2")**
This means `V2__seed_data.sql` (or `V1__init_schema.sql`) was edited *after* it had already been
applied to a running database — Flyway detects the file no longer matches what it recorded.
Two ways to resolve, depending on whether you need to keep existing data:
- If it's a disposable dev database: wipe and re-migrate from scratch —
  `docker compose down -v && docker compose up --build` (Docker path), or drop/recreate your local
  database (local path).
- If you must keep existing data: do **not** edit already-applied migration files going forward —
  add a new `V3__*.sql` instead. (The `flyway-maven-plugin` is not wired into `backend/pom.xml`
  in this POC, so `mvn flyway:repair` is not available out of the box; adding it, or repairing the
  `flyway_schema_history` table by hand, are the two ways to recover from a mismatch without
  dropping data — both require care, so prefer the disposable-database route above whenever
  possible.)

**`docker compose up` fails to build `backend` or `frontend` (image not found / Dockerfile missing)**
The root `docker-compose.yml` references `backend/Dockerfile` and `frontend/Dockerfile`, each
owned by the respective backend/frontend build. If either is still in progress, the build will
fail until that Dockerfile exists — check with the other workstream, or build/run that side
locally via Path 2 in the meantime.

**Swagger UI shows no endpoints / 404 on `/swagger-ui.html`**
Confirm the backend actually started successfully (`docker compose logs backend` or the Maven
console) — a Flyway migration failure or a missing required env var (e.g. `JWT_SECRET`) will
prevent the Spring context from coming up, in which case no controllers are registered.

**Playwright can't reach `http://localhost:3000`**
Make sure the frontend is actually running (Path 1 or Path 2 above) before running `npm test` in
`e2e/` — the suite intentionally does not start the frontend itself (see the comment in
`e2e/playwright.config.ts`).
