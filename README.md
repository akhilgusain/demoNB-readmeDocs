# Society Management Platform

A production-grade proof-of-concept residential society management platform, in the spirit of
NoBrokerHood — gated community administration, resident self-service, visitor/security management,
complaints, billing, amenities, and an AI assistant, built as a greenfield monorepo.

> This is a POC. See [Known limitations](#known-limitations) before using it for anything real.

## Screenshots

### Login

|                                                        |
|--------------------------------------------------------|
| ![Login](docs/screenshots/00-login.png) |

### Society Admin

| | |
|---|---|
| ![Admin overview](docs/screenshots/01-admin-overview.png) **Overview** — stat cards, AI insights summary, collections trend and complaints-by-status charts. | ![Admin residents](docs/screenshots/02-admin-residents.png) **Residents** — flat-mapped resident directory. |
| ![Admin visitors](docs/screenshots/03-admin-visitors.png) **Visitors** — all visitor entries with AI risk level and status. | ![Admin complaints](docs/screenshots/04-admin-complaints.png) **Complaints** — AI category/priority, status, staff assignment. |
| ![Admin notices](docs/screenshots/05-admin-notices.png) **Notices** — announcements, events, circulars. | ![Admin billing](docs/screenshots/06-admin-billing.png) **Billing** — invoices, payment status, bulk generation. |
| ![Admin amenities](docs/screenshots/07-admin-amenities.png) **Amenities** — clubhouse/gym/hall configuration. | ![Admin staff](docs/screenshots/08-admin-staff.png) **Staff** — domestic/maintenance staff directory. |
| ![Admin reports](docs/screenshots/09-admin-reports.png) **Reports** — complaint, visitor, and financial breakdowns. | |

### Resident

| | |
|---|---|
| ![Resident overview](docs/screenshots/10-resident-overview.png) **Overview** — personal dashboard: complaints, dues, bookings, notices. | ![Resident visitors](docs/screenshots/11-resident-visitors.png) **Visitors** — pre-approve a guest, see status & QR pass. |
| ![Resident complaints](docs/screenshots/12-resident-complaints.png) **Complaints** — raise and track, with AI category/severity badges. | ![Resident billing](docs/screenshots/13-resident-billing.png) **Billing** — view and pay maintenance invoices. |
| ![Resident amenities](docs/screenshots/14-resident-amenities.png) **Amenities** — check slot availability and book. | ![Resident notices](docs/screenshots/15-resident-notices.png) **Notices** — society announcements/events/circulars. |
| ![Resident chat](docs/screenshots/16-resident-chat.png) **AI assistant chat** — ask questions, get instant answers. | |

### Security Guard

| | |
|---|---|
| ![Guard visitor entries](docs/screenshots/17-guard-visitor-entries.png) **Visitor entries** — approve/reject, check-in/check-out. | ![Guard pre-approved](docs/screenshots/18-guard-pre-approved.png) **Pre-approved** — visitors residents already cleared. |
| ![Guard staff attendance](docs/screenshots/19-guard-staff-attendance.png) **Staff attendance** — mark daily attendance for domestic staff. | |

## How to use this app

### 1. Log in
Go to the frontend URL and sign in with any seeded credential from the
[Default seed credentials](#default-seed-credentials) table below (e.g. `admin@greenvalley.com` /
`Password123!`). You're routed straight to the dashboard for your role — there's no separate "pick a
role" step, the account itself determines what you see.

### 2. As a Society Admin
- **Overview** is your home screen: headline stats, an AI-generated one-line summary of how the
  society is doing, and charts for collections and open complaints.
- **Residents** → add a resident, map them to a flat, and record family members.
- **Staff** → register domestic/maintenance staff and review attendance.
- **Visitors** → see every visitor system-wide; override a guard's decision if needed.
- **Notices** → publish an announcement, event, or circular (optionally with an attachment).
- **Complaints** → triage the queue: every complaint already has an AI-suggested category and
  priority; assign it to a staff member from the dropdown and watch its status move through the
  workflow (Open → Assigned → In Progress → Resolved/Closed).
- **Billing** → bulk-generate maintenance invoices for a billing period across all flats, track who
  has paid, and record offline payments.
- **Amenities** → define bookable amenities (clubhouse, gym, hall...), their hours and slot length.
- **Reports** → pull complaint/visitor/financial breakdowns for the whole society.

### 3. As a Resident
- **Overview** shows what needs your attention: open complaints, dues, upcoming bookings, recent
  notices.
- **Visitors** → pre-approve a guest before they arrive (name, phone, purpose); they get a QR pass
  the guard scans/checks against at the gate. You can also approve/reject a guard-flagged visitor.
- **Complaints** → raise one with a title and description; the AI assistant immediately suggests a
  category and severity so it reaches the right person faster. Track status and comment on it.
- **Billing** → see every invoice for your flat and pay the open ones.
- **Amenities** → pick an amenity, see which slots are already booked, and reserve one.
- **Notices** → read what the society has published.
- **AI chat** → ask anything (e.g. "how do I pay my maintenance bill?") via the floating chat button.

### 4. As a Security Guard
- **Visitor entries** → the live queue of everyone wanting in: approve/reject a walk-in, then
  check them in and, later, check them out.
- **Pre-approved** → visitors a resident already cleared — just check their QR pass and check them in.
- **Staff attendance** → mark domestic/maintenance staff present/absent for the day.

### 5. As a Committee Member
Same visibility as the admin into complaints, visitors, notices, and reports, without the
operational controls (no billing generation, no staff management) — for oversight rather than
day-to-day admin.

## Features / Modules

1. **Auth** — JWT access + refresh tokens, BCrypt password hashing, forgot/reset password.
2. **Societies** — society master data (SUPER_ADMIN managed).
3. **Blocks & Flats** — society structure (blocks, floors, flats).
4. **Residents** — resident profiles, family members, self-service profile management.
5. **Staff** — domestic/maintenance staff records and daily attendance.
6. **Visitors** — pre-approval, QR codes, security guard check-in/check-out, AI-based risk scoring.
7. **Notice Board** — announcements, events, and circulars.
8. **Complaints** — resident-raised complaints with AI auto-categorization and severity scoring,
   staff assignment, status workflow, and comments.
9. **Billing** — maintenance invoices (bulk generation per flat), online/offline payments, reports.
10. **Amenities** — clubhouse/gym/pool etc. with availability calendars and bookings.
11. **Chat** — AI assistant chat per user (rule-based by default, pluggable to OpenAI).
12. **Notifications** — in-app/email notifications generated by other modules.
13. **Dashboards & Reports** — role-specific dashboards (admin/resident/guard) and aggregate reports
    with an AI-generated analytics summary.

## What you get

A gated-community platform that replaces phone calls, WhatsApp groups, and paper registers with one
app, tailored to each person who uses it:

- **Residents** pre-approve visitors before they arrive, track and pay maintenance bills online,
  book the clubhouse/gym/hall in a few taps, raise a complaint and watch it get auto-categorized and
  assigned instead of going unanswered, catch up on notices, and get instant answers from an AI
  assistant instead of waiting on the admin.
- **Society admins** manage residents, flats, staff, and billing from one dashboard instead of
  spreadsheets; invoices generate in bulk, complaints route themselves by AI-assessed category and
  severity, and an AI-written summary of the society's health shows up on login instead of having to
  dig through reports.
- **Security guards** see pre-approved visitor passes with QR codes, approve/reject/check-in/check-out
  from a single screen, and get an AI risk flag on unfamiliar visitors — no logbook, no guesswork.
- **Committee members** get the same read-only visibility into complaints, visitor activity, and
  finances as the admin, without needing day-to-day operational access.
- **Maintenance staff** see exactly which complaints are assigned to them and their attendance
  history, instead of relying on word of mouth.

## Clone and run it yourself

```bash
git clone https://github.com/akhilgusain/demoNB.git
cd demoNB
git checkout claude/society-management-poc-n2yoge   # this branch, until it's merged to main
```

Then jump to **Quick start** below (Docker, fastest) or **Running locally** (hot-reload dev loop).
Either way you get the full app, seeded with a realistic demo society, logins and all — nothing else
to configure.

## Tech stack

| Layer     | Stack |
|-----------|-------|
| Backend   | Java 21, Spring Boot 3, Spring Security, Spring Data JPA, Flyway, PostgreSQL 15, JJWT, springdoc-openapi |
| Frontend  | Next.js 14+ (App Router), TypeScript, Tailwind CSS, ShadCN UI, TanStack React Query, TanStack Table, Zustand, Recharts, next-themes |
| AI layer  | Pluggable `AiService` interface — `RuleBasedAiService` (default, offline, deterministic) or `OpenAiService` (OpenAI Chat Completions, opt-in) |
| Testing   | JUnit 5 + Mockito + Testcontainers/H2 (backend), Playwright (E2E, in `e2e/`) |
| DevOps    | Docker, Docker Compose, Flyway migrations |

## Quick start (Docker Compose)

Prerequisites: Docker + Docker Compose.

```bash
cp .env.example .env     # optional - sensible dev defaults are already baked into docker-compose.yml
./scripts/start.sh        # docker compose up --build -d, then prints URLs + credentials
```

Or directly:

```bash
docker compose up --build
```

To stop:

```bash
./scripts/stop.sh          # docker compose down (keeps the pgdata volume)
```

See [docs/SETUP_GUIDE.md](docs/SETUP_GUIDE.md) for full troubleshooting.

## Running locally (no Docker for app code)

Use this path for hot-reload during development. Only Postgres runs in Docker; the backend and
frontend run directly on your machine.

**Prerequisites:** Docker (for Postgres), Java 21 (JDK), Node.js 20+ and npm 10+. Maven is not
required separately — the bundled `./mvnw` wrapper is used.

1. Start Postgres only:
   ```bash
   docker compose up postgres -d
   ```
2. Start the backend (new terminal):
   ```bash
   ./scripts/dev-backend.sh
   # equivalent to: cd backend && ./mvnw spring-boot:run
   ```
   This runs Flyway migrations (`V1__init_schema.sql` + `V2__seed_data.sql`) against the dockerized
   Postgres automatically on boot. Backend comes up at http://localhost:8080
   (Swagger UI: http://localhost:8080/swagger-ui.html).
3. Start the frontend (another terminal):
   ```bash
   ./scripts/dev-frontend.sh
   # equivalent to: cd frontend && npm install && npm run dev
   ```
   Frontend comes up at http://localhost:3000, calling the backend via `NEXT_PUBLIC_API_URL`
   (defaults to `http://localhost:8080/api/v1`).
4. Log in with any seeded credential, e.g. `admin@greenvalley.com` / `Password123!` (full table
   below).
5. Run backend tests: `cd backend && ./mvnw test`.
6. Run the Playwright E2E suite (needs the full stack already running):
   ```bash
   cd e2e && npm install && npx playwright install --with-deps chromium && npm test
   ```

See [docs/SETUP_GUIDE.md](docs/SETUP_GUIDE.md) for the Docker-only path in more detail and
troubleshooting (port conflicts, DB auth errors, Flyway checksum mismatches, etc.).

## Deployment

The supported deployment path is **Docker Compose on a single VM** — that's what `docker-compose.yml`
is built for. There are no Kubernetes manifests or Terraform/cloud templates in this POC (see
[Known limitations](#known-limitations)).

### 1. Provision a host
Any Linux VM with Docker Engine + the Compose plugin works (a $5-10/mo VPS is plenty for a POC):
EC2/Lightsail, DigitalOcean/Linode/Vultr droplet, Azure/GCP VM, or your own server.

```bash
curl -fsSL https://get.docker.com | sh          # installs Docker + the compose plugin
```

### 2. Get the code onto the host and configure real secrets

```bash
git clone https://github.com/akhilgusain/demoNB.git
cd demoNB
cp .env.example .env
```

Edit `.env` and set **real, unique values** before going anywhere near production traffic:

| Variable | Why it matters |
|---|---|
| `JWT_SECRET` | Signs every access/refresh token — the default in `.env.example` is a dev placeholder. |
| `POSTGRES_PASSWORD` | Default is `society_pass`; change it. |
| `CORS_ALLOWED_ORIGINS` | Set to your real frontend origin (e.g. `https://society.example.com`), not `localhost`. |
| `NEXT_PUBLIC_API_URL` | The **public** URL the browser will call, e.g. `https://api.society.example.com/api/v1`. This is baked into the frontend at build time (see step 4), not read at container start. |
| `AI_PROVIDER` / `OPENAI_API_KEY` | Leave `AI_PROVIDER=rule-based` (default, no key needed) or switch to `openai` and supply a key. |
| `MAIL_ENABLED` / `MAIL_*` | Set `MAIL_ENABLED=true` with real SMTP creds to send real emails instead of just logging them. |

### 3. Put a reverse proxy with TLS in front
The containers serve plain HTTP on :3000/:8080. Terminate TLS in front of them — the simplest option
is [Caddy](https://caddyserver.com/) (automatic HTTPS via Let's Encrypt):

```caddyfile
# /etc/caddy/Caddyfile
society.example.com {
    reverse_proxy localhost:3000
}
api.society.example.com {
    reverse_proxy localhost:8080
}
```

(Nginx + certbot works just as well if that's your standard.)

### 4. Build and start

```bash
docker compose up --build -d
```

Because `NEXT_PUBLIC_API_URL` is inlined into the frontend's JS bundle at build time, make sure it's
set to the **public** API URL in `.env` *before* this build — if you change it later you must rebuild
the frontend image (`docker compose up --build -d frontend`), not just restart it.

This starts Postgres, runs Flyway migrations + seed data automatically on backend boot, then starts
the frontend. Check health with `docker compose ps` and `docker compose logs -f backend`.

### 5. Ongoing operations

- **Updating to a new commit**: `git pull && docker compose up --build -d`.
- **Backups**: the Postgres data lives in the named volume `pgdata`; back it up with
  `docker compose exec postgres pg_dump -U <user> <db> > backup.sql` on a schedule.
- **Logs**: `docker compose logs -f backend` / `frontend` / `postgres`.
- **Scaling beyond one VM**: build and push the `backend`/`frontend` images to a registry
  (`docker build -t myregistry/society-backend ./backend && docker push ...`), point a managed
  Postgres instance (RDS/Cloud SQL/etc.) at `DB_URL`, and run the images on whatever you use to
  orchestrate containers (ECS, Cloud Run, a Swarm/k8s cluster) — this POC doesn't ship manifests for
  that, but the images themselves are plain, portable Docker images with no POC-specific assumptions.

## Default URLs

| Service       | URL |
|---------------|-----|
| Frontend      | http://localhost:3000 |
| Backend API   | http://localhost:8080/api/v1 |
| Swagger UI    | http://localhost:8080/swagger-ui.html |
| OpenAPI spec  | http://localhost:8080/v3/api-docs |

## Default seed credentials

All seed users share the password **`Password123!`**.

| Email | Role |
|-------|------|
| superadmin@society.com | SUPER_ADMIN |
| admin@greenvalley.com | SOCIETY_ADMIN |
| committee@greenvalley.com | COMMITTEE_MEMBER |
| guard@greenvalley.com | SECURITY_GUARD |
| resident1@greenvalley.com ... resident8@greenvalley.com | RESIDENT |
| maintenance@greenvalley.com | MAINTENANCE_STAFF |

Seed data (society "Green Valley Society", Bengaluru) includes 2 blocks, 12 flats, 8 residents with
family members, 5 staff with attendance, 10 visitors, 10 notices, 12 complaints, 12 invoices with
payments, 3 amenities with bookings, notifications, and audit logs. See
`backend/src/main/resources/db/migration/V2__seed_data.sql`.

## Folder structure

```
demoNB/
├── backend/                     Spring Boot 3 API (Java 21, Maven)
│   └── src/main/resources/db/migration/
│       ├── V1__init_schema.sql  Ground-truth schema (do not edit)
│       └── V2__seed_data.sql    Deterministic demo/seed data
├── frontend/                    Next.js 14+ App Router frontend (TypeScript, Tailwind, ShadCN)
├── e2e/                         Playwright end-to-end tests (independent of frontend/)
├── docs/                        Architecture, database, API, and setup documentation
├── scripts/                     start.sh, stop.sh, dev-backend.sh, dev-frontend.sh
├── docker-compose.yml           postgres + backend + frontend services
└── .env.example                 All environment variables used across services
```

## Documentation

- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — layered architecture, AI abstraction, RBAC, data
  isolation, deployment topology.
- [docs/DATABASE.md](docs/DATABASE.md) — full ER diagram (Mermaid) and table descriptions.
- [docs/API_GUIDE.md](docs/API_GUIDE.md) — human-readable endpoint summary per module.
- [docs/SETUP_GUIDE.md](docs/SETUP_GUIDE.md) — prerequisites, step-by-step setup (Docker and local),
  troubleshooting.

## Known limitations

This is a proof-of-concept, not a production deployment. In particular:

- **AI features are rule-based by default** (`AI_PROVIDER=rule-based`): deterministic
  keyword/heuristic classification for complaints and visitor risk, and templated analytics
  summaries/chat replies. No API key is provisioned in this environment. The `OpenAiService`
  implementation exists and can be enabled via `AI_PROVIDER=openai` + `OPENAI_API_KEY`, but has
  not been exercised against a live OpenAI account here.
- **No real payment gateway** — "paying" an invoice records a `Payment` row and marks the invoice
  `PAID`; no actual card/UPI processor is integrated.
- **Local file storage only** — avatar/attachment/photo URLs are placeholders; there is no S3/GCS
  integration. File uploads (where implemented) land on local disk inside the backend container,
  which is not persisted across container rebuilds.
- **Email is a console/log stub** — `EmailService` abstraction exists but does not send real email
  by default (no SMTP credentials provisioned).
- **Single-tenant-per-login demo data** — seed data models exactly one society ("Green Valley
  Society"); multi-society SUPER_ADMIN flows are supported by the schema/API but only lightly
  exercised by seed data and E2E tests.
- **JWT secret / DB password defaults are for local dev only** — never use the defaults in
  `.env.example` / `docker-compose.yml` outside a throwaway local environment.
