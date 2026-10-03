# Architecture

## 1. Backend — layered architecture

Standard Spring Boot layered architecture, one vertical slice per module
(`society`, `resident`, `staff`, `visitor`, `notice`, `complaint`, `billing`, `amenity`, `chat`,
`notification`, `dashboard`), each following the same `controller → service → repository` shape:

```
┌───────────────────────────────────────────────────────────────────────────┐
│                              HTTP (JSON)                                  │
└───────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ Controller layer   (@RestController, @PreAuthorize, request/response DTOs)│
│   - validates input (jakarta.validation)                                  │
│   - enforces role checks declaratively                                    │
│   - maps HTTP <-> service calls, no business logic                        │
└───────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ Service layer      (@Service, @RequiredArgsConstructor, @Transactional)    │
│   - business rules, orchestration across repositories                     │
│   - society-scoped data isolation (derives societyId from principal)      │
│   - calls AiService for complaint categorization / visitor risk / chat    │
│   - calls NotificationService / EmailService / AuditLogService            │
└───────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ Repository layer   (Spring Data JPA, interfaces extending JpaRepository)   │
│   - entity <-> table mapping                                              │
│   - explicit @Query for aggregate/report queries                          │
└───────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ PostgreSQL 15  (Flyway-managed schema: V1__init_schema.sql, V2__seed_...) │
└───────────────────────────────────────────────────────────────────────────┘
```

Cross-cutting concerns live outside the vertical slices:

- `config/` — `SecurityConfig` (JWT filter chain, CORS), `OpenApiConfig` (Swagger), `WebConfig`,
  `JacksonConfig`.
- `security/` — `JwtTokenProvider`, `JwtAuthenticationFilter`, `CustomUserDetailsService`.
- `common/exception` — `GlobalExceptionHandler` (`@ControllerAdvice`) turning exceptions into the
  standard `{ timestamp, status, error, message, path, validationErrors? }` error shape.
- `common/dto` — `PageResponse<T>` (list endpoints), `ApiResponse<T>`.
- `common/audit` — `AuditLogService` + `@Auditable` aspect, writing to `audit_logs`.

## 2. Frontend — layered architecture

```
┌───────────────────────────────────────────────────────────────────────────┐
│ app/ (Next.js App Router)                                                  │
│   (auth)/login, register, forgot-password      — public routes            │
│   (dashboard)/{admin,resident,guard,committee}/* — role-scoped routes     │
│   layout.tsx → role-aware sidebar/topbar shell, dark mode (next-themes)   │
└───────────────────────────────────────────────────────────────────────────┘
                                   │  uses
                                   ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ hooks/ (React Query hooks: use-auth, use-residents, use-visitors, ...)     │
│   - one hook file per module, wraps queries/mutations                     │
│   - owns cache keys + invalidation                                        │
└───────────────────────────────────────────────────────────────────────────┘
                                   │  calls
                                   ▼
┌───────────────────────────────────────────────────────────────────────────┐
│ lib/api-client.ts                                                          │
│   - fetch/axios wrapper, attaches JWT access token                        │
│   - 401 → silent refresh via /auth/refresh, retries original request      │
│   - base URL = NEXT_PUBLIC_API_URL                                        │
└───────────────────────────────────────────────────────────────────────────┘
                                   │  HTTP
                                   ▼
                        Backend API (/api/v1/**)
```

Supporting layers: `components/ui` (ShadCN primitives), `components/shared` (DataTable, StatCard,
PageHeader, ProtectedRoute, RoleGate, ChartCard), `store/auth-store.ts` (Zustand, persisted —
current user + role, used by `RoleGate`/`middleware.ts`), `types/` (DTO mirrors), `middleware.ts`
(redirects unauthenticated/unauthorized requests before they hit a protected route).

## 3. AI abstraction layer

```
                     ┌───────────────────────────┐
                     │   AiService (interface)    │
                     │  categorizeComplaint(...)  │
                     │  assessVisitorRisk(...)    │
                     │  generateAnalyticsSummary()│
                     │  chat(...)                 │
                     └─────────────┬─────────────┘
                     ┌─────────────┴─────────────┐
                     ▼                           ▼
        ┌─────────────────────────┐  ┌───────────────────────────┐
        │  RuleBasedAiService      │  │  OpenAiService             │
        │  (default, always on)   │  │  (opt-in)                  │
        │  - keyword/heuristic     │  │  - OpenAI Chat Completions │
        │    classification        │  │    via RestClient/WebClient│
        │  - deterministic scores  │  │  - needs OPENAI_API_KEY    │
        │  - zero external calls   │  │                             │
        └─────────────────────────┘  └───────────────────────────┘
```

Selected via `@ConditionalOnProperty(name = "app.ai.provider", havingValue = "openai")` on
`OpenAiService` (with `RuleBasedAiService` as `@ConditionalOnProperty(..., havingValue =
"rule-based", matchIfMissing = true)` or a small `@Configuration` factory bean) — swapping
providers is a one-line env var change (`AI_PROVIDER`), no code change, no redeploy of a
different artifact. The POC ships with `rule-based` as the default so the product is fully
functional offline without any provisioned API key.

## 4. RBAC model

Roles (`users.role`, enforced by a `CHECK` constraint and by Spring Security):

`SUPER_ADMIN` · `SOCIETY_ADMIN` · `RESIDENT` · `SECURITY_GUARD` · `COMMITTEE_MEMBER` ·
`MAINTENANCE_STAFF`

- JWT access tokens carry the user's `role` and `societyId` as claims.
- Controllers declare required roles with `@PreAuthorize("hasRole('SOCIETY_ADMIN')")` (or
  `hasAnyRole(...)`) per endpoint, matching the "Modules & endpoints" table in
  [API_GUIDE.md](API_GUIDE.md).
- `SUPER_ADMIN` is the only role without a fixed `society_id` (platform-level); every other role's
  `users.society_id` scopes what they can see.
- Frontend mirrors this with `RoleGate` components and `middleware.ts` route guards, so unauthorized
  UI never renders even if an API call is later also guarded by `@PreAuthorize`.

## 5. Data isolation strategy (multi-society safety net)

Every tenant-owned table (`blocks`, `flats`, `users`, `staff`, `visitors`, `notices`, `complaints`,
`invoices`, `amenities`, ...) carries a `society_id` column. Rather than relying on Postgres Row
Level Security (out of scope for this POC), isolation is enforced in the service layer:

1. `CustomUserDetails` (built from the JWT principal) exposes `getSocietyId()`.
2. Every service method for a `SOCIETY_ADMIN` / `COMMITTEE_MEMBER` / `RESIDENT` / `SECURITY_GUARD` /
   `MAINTENANCE_STAFF` request derives `societyId` from the authenticated principal — **never**
   from a client-supplied request parameter — and adds it as a mandatory filter on every query
   (`findAllBySocietyId`, `@Query(... where society_id = :societyId)`, etc.).
3. `SUPER_ADMIN` endpoints (e.g. `GET /api/v1/societies`) are the only ones allowed to omit this
   filter, and are separately `@PreAuthorize("hasRole('SUPER_ADMIN')")`-gated.
4. Row ownership is double-checked for single-entity reads/writes (`GET/PUT/DELETE /{id}`): the
   service loads the entity, then verifies `entity.societyId == principal.societyId` before
   returning/mutating it, returning `404`/`403` otherwise — this prevents ID-guessing across
   societies even though IDs are UUIDs.

## 6. Deployment topology (docker-compose)

```
                         ┌───────────────────────────────┐
                         │        society_network        │
                         │           (bridge)             │
                         │                                │
  localhost:5432 ───────▶│  postgres (postgres:15-alpine) │
                         │   volume: pgdata                │
                         │   healthcheck: pg_isready        │
                         │                │ (service_healthy)
                         │                ▼                │
  localhost:8080 ───────▶│  backend (backend/Dockerfile)   │
                         │   Spring Boot, Flyway migrate    │
                         │   on startup (V1 + V2)           │
                         │                │ (depends_on)     │
                         │                ▼                │
  localhost:3000 ───────▶│  frontend (frontend/Dockerfile) │
                         │   Next.js, NEXT_PUBLIC_API_URL   │
                         │   → http://localhost:8080/api/v1 │
                         └───────────────────────────────┘
```

- Single `docker-compose.yml` at repo root; `postgres` must report healthy before `backend` starts,
  `backend` must have started before `frontend` starts.
- All configuration flows in via environment variables (see root `.env.example`); no secrets are
  baked into images.
- `pgdata` is a named Docker volume so data survives `docker compose down` (but not `down -v`).
