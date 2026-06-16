# Sprint S1 — Project Bootstrap · Execution Plan

> Detailed, dependency-ordered execution plan for Sprint S1. This is a **planning artifact** — it
> defines the tasks, ordering, risks, and checkpoints for the bootstrap sprint. It does not contain
> implementation. Source scope: [`docs/sprint_tracking.md`](sprint_tracking.md) and
> [`PLAN.md` §5–§7, §10–§12](../PLAN.md).

- **Phase:** 1 — Core Foundation & Job Discovery
- **Sprint:** S1 — Project Bootstrap
- **Status entering:** Not started
- **Authoring roles:** Principal Architect / Technical Program Manager / Senior Staff Engineer
- **Date:** 2026-06-14
- **Revised:** 2026-06-15 — **T02 split into T02A (uv project bootstrap) + T02B (quality toolchain)**;
  downstream dependencies repointed to T02B. Task count 21 → 22.

---

## 0. Sprint Framing

**Sprint goal:** A reproducible monorepo where `docker-compose up` brings up the full local stack,
the complete PostgreSQL schema is applied via Alembic, and a user can register/log in over JWT — with
`ruff`/`mypy`/`pytest` green in CI.

**S1 exit criteria (Definition of Done):**

- `docker-compose up` starts api, postgres, redis, celery-worker, celery-beat, flower, qdrant, minio,
  nginx — all healthy.
- `alembic upgrade head` applies the full [§7](../PLAN.md) schema; `downgrade base` is clean;
  autogenerate diff is empty.
- Register → login → access a protected route works end-to-end.
- CI runs format-check + lint + type-check + tests on every PR and is green.
- `.env.example` documents every variable from [`PLAN.md` §12.2](../PLAN.md).

**Explicitly OUT of scope for S1 (deferred):** scrapers & `BaseAgent`/Job Scout (S2), Qdrant/MinIO
*application* integration (S2), Ollama (S3), Playwright (S4), profile/preferences CRUD + resume
upload (S2), React shell (S2), Ntfy/WebSocket (S6), Prometheus/Grafana (S11). The Qdrant and MinIO
**containers** boot in S1 but carry no app-level wiring yet.

**Assumptions:** Python 3.12, Docker Desktop, Node deferred to S2. The dependency set is scoped to the
FastAPI/SQLAlchemy/Celery/auth stack only — `langchain`, `langgraph`, `playwright`, `crawl4ai`,
`sentence-transformers`, `qdrant-client` are deferred to keep the S1 image fast (candidate ADR-0019).

---

## 1. Task Summary

| ID | Task | Deps | Cplx | Risk | Context |
|----|------|------|:---:|:---:|:---:|
| **T01** | Repo hygiene & monorepo skeleton | — | 2 | 1 | Low |
| **T02A** | Backend uv project bootstrap (runtime deps + lockfile) | T01 | 2 | 3 | Low–Med |
| **T02B** | Quality toolchain config (ruff `select=ALL` / mypy strict / pytest) | T02A | 3 | 3 | Low–Med |
| **T03** | Core config (`pydantic-settings`) + `.env.example` | T02B | 3 | 3 | Med |
| **T04** | Core primitives: logging (Loguru) + exceptions | T02B,T03 | 2 | 2 | Low |
| **T05** | Async DB engine + session factory | T02B,T03 | 4 | 4 | Med |
| **T06** | Security utils (password hashing + JWT) | T02B,T03 | 4 | 5 | Med |
| **T07** | ORM base: declarative Base, naming convention, mixins | T02B | 4 | 5 | Med |
| **T08** | Core domain models (User, UserPreferences, Company, Job) | T07 | 4 | 3 | Med |
| **T09** | Application-domain models (Resume, ResumeVersion, Application, StatusHistory) | T08 | 5 | 4 | Med |
| **T10** | Outreach/conversation/notification models | T09 | 5 | 3 | Med–High |
| **T11** | Alembic async env + initial migration (+ §7.2 indexes) | T05,T10 | 6 | 6 | Med |
| **T12** | FastAPI app factory + middleware + `/health` + router | T03,T04,T05 | 4 | 3 | Med |
| **T13** | Test harness (conftest, async DB + client fixtures) | T05,T10,T12 | 5 | 5 | Med |
| **T14** | Auth via FastAPI-Users (schemas, manager, JWT backend, routes) | T06,T08,T12,T13 | 7 | 7 | High |
| **T15** | Celery app factory + Beat skeleton + health task | T03,T04 | 4 | 4 | Med |
| **T16** | Backend Dockerfile | T02B,T12 | 4 | 4 | Med |
| **T17** | docker-compose: data plane (postgres/redis/qdrant/minio) | T01,T03 | 4 | 4 | Med |
| **T18** | docker-compose: app/edge plane (api/worker/beat/flower/nginx) + root compose | T11,T15,T16,T17 | 6 | 6 | High |
| **T19** | GitHub Actions CI | T02B,T11,T14 | 5 | 5 | Med |
| **T20** | Developer experience (Makefile/justfile + minimal seed) | T02B,T12 | 2 | 2 | Low |
| **T21** | Sprint S1 closeout (tracking/README/retro) | T18,T19 | 2 | 2 | Low–Med |

*Context bands: Low ≈ <15k tokens · Med ≈ 15–40k · High ≈ 40–70k, per fresh per-task Claude Code
context (read dependencies + generate code + tests + docs + journal).*

---

## 2. Dependency Order & Critical Path

**The spine (critical path):**

```
T01 → T02A → T02B → T07 → T08 → T09 → T10 → T11 → T12 → T13 → T14 → T16 → T18 → T19 → T21
```

The **data-model chain (T07→T11)** is the longest dependency run and the highest-stakes — it gates
the app factory, tests, auth, and the container that runs migrations. Treat it as the pacing item.

**Side branches feeding the spine:** `T03` (config) and `T05` (engine) feed T11/T12; `T06` (security)
feeds T14; `T15` (celery) and `T17` (data-plane compose) feed T18.

```
T01 ─ T02A ─ T02B ─┬─ T03 ─┬─ T04 ──────────────┐
                   │       ├─ T05 ─┐             │
                   │       └─ T06 ─┼──────────┐  │
                   ├─ T07 ─ T08 ─ T09 ─ T10 ─┴─ T11 ─ T12 ─ T13 ─ T14   (T14 also needs T06)
                   ├─ T15 ──────────────────────────────┐
                   └─ T17 ──────────────────────────────┤
                                                  T16 ──┤   (T16 also needs T12)
                                                   └─ T18 ─ T19 ─ T21   (T19 also needs T14)
```

## 3. Parallelization Plan (suggested waves)

- **Wave 0:** T01 → T02A → T02B (serial; everything roots here).
- **Wave 1 (parallel):** T03, T07. *(config track + ORM-base track diverge here)*
- **Wave 2 (parallel):** T04, T05, T06 (after T03) ‖ T08 (after T07) ‖ T15 (after T03/T04) ‖
  T17 (after T03) ‖ T20 (after T02B).
- **Wave 3:** T09 → T10 (model chain) ‖ T12 (after T03/T04/T05).
- **Wave 4:** T11 (needs T05+T10) → T13 (needs T11+T12) → T14 (needs T06+T13) ‖ T16 (after T12).
- **Wave 5:** T18 (integration) → T19 (CI) → T21 (closeout).

A solo developer should still run the model chain serially; the genuine parallelism is
**infra/tooling (T15/T16/T17/T20) alongside the model chain**.

---

## 4. Detailed Task Cards

> Each card: Objective · Dependencies · Files Expected · Validation Criteria · Definition of Done ·
> Recommended Commit Message · Complexity/Risk/Context.

### S1-T01 — Repository hygiene & monorepo skeleton

- **Objective:** Create the top-level directory skeleton from [`PLAN.md` §6](../PLAN.md) and enforce
  cross-platform hygiene (the establishment commit showed LF→CRLF warnings on Windows).
- **Dependencies:** none.
- **Files Expected:** `.gitignore`, `.gitattributes` (force `* text=auto eol=lf`), `.editorconfig`;
  placeholder `.gitkeep` in `apps/`, `backend/`, `infra/`, `scripts/`, `.github/workflows/`.
- **Validation Criteria:** `git check-attr -a` shows `eol=lf`; re-staging existing files yields **no**
  CRLF warnings; tree matches §6 layout.
- **Definition of Done:** Skeleton committed; no ignored-but-tracked files; line-ending policy active.
- **Commit:** `chore(repo): add monorepo skeleton, gitignore and line-ending policy`
- **Complexity 2 · Risk 1 · Context Low**

### S1-T02A — Backend uv project bootstrap (runtime deps + lockfile)

- **Objective:** Stand up the `backend/` Python project with `uv` and declare + lock the **S1 runtime
  dependency surface**; make the `app` package importable. This is where the dependency-scoping
  decision ([ADR-0019](decision_log.md)) and the FastAPI-Users ⇄ SQLAlchemy compatibility matrix
  ([TR2](#5-risk-register)) are settled.
- **Dependencies:** T01.
- **Files Expected:** `backend/pyproject.toml` (`[project]` metadata + runtime deps: `fastapi`,
  `uvicorn[standard]`, `sqlalchemy[asyncio]`, `alembic`, `pydantic`, `pydantic-settings`,
  `celery[redis]`, `redis`, `fastapi-users[sqlalchemy]`, `python-jose[cryptography]`,
  `passlib[bcrypt]`, `httpx`, `loguru`), `backend/uv.lock`, `backend/.python-version`,
  `backend/app/__init__.py`. (Removes `backend/.gitkeep`.)
- **Validation Criteria:** `uv sync` resolves the full S1 runtime set with no conflicts;
  `uv run python -c "import app"` succeeds; `langchain`/`langgraph`/`playwright`/`crawl4ai`/
  `sentence-transformers`/`qdrant-client` are **not** present (deferred per ADR-0019).
- **Definition of Done:** Reproducible runtime environment from a committed lockfile; app package imports.
- **Commit:** `chore(backend): bootstrap uv project and lock S1 runtime dependencies`
- **Complexity 2 · Risk 3 (dependency resolution + fastapi-users/sqlalchemy compat) · Context Low–Med**

### S1-T02B — Quality toolchain configuration (ruff / mypy / pytest)

- **Objective:** Add and configure the quality gates per
  [`coding_standards.md` §2](coding_standards.md) and [ADR-0011](decision_log.md): ruff (lint +
  format), mypy strict, and pytest. Isolates the opinionated, iteration-prone `select=ALL`
  ignore-tuning into its own reviewable diff.
- **Dependencies:** T02A.
- **Files Expected:** `backend/pyproject.toml` (dev group: `ruff`, `mypy`, `pytest`,
  `pytest-asyncio`, `pytest-cov`; plus `[tool.ruff]` line-length 100 + `select=ALL` with **documented
  ignores**, `[tool.mypy]` strict, `[tool.pytest.ini_options]` + coverage config); updated
  `backend/uv.lock`.
- **Validation Criteria:** `uv run ruff format --check .` and `uv run ruff check .` pass on the empty
  tree; `uv run mypy app/` clean (strict); `uv run pytest` collects 0 tests without error.
- **Definition of Done:** All four quality commands green and reproducible from the lockfile; every
  ruff ignore has a one-line justification.
- **Commit:** `chore(backend): configure ruff, mypy strict and pytest toolchain`
- **Complexity 3 · Risk 3 (select=ALL ignore tuning) · Context Low–Med**

### S1-T03 — Core configuration + environment contract

- **Objective:** Implement `Settings` (pydantic-settings) covering **every** variable in
  [`PLAN.md` §12.2](../PLAN.md); produce `.env.example` files.
- **Dependencies:** T02B.
- **Files Expected:** `backend/app/core/__init__.py`, `backend/app/core/config.py`,
  `backend/.env.example`, root `.env.example`; `backend/tests/unit/test_config.py`.
- **Validation Criteria:** Settings loads from env; missing required → clear validation error;
  `.env.example` ⇄ `Settings` fields are 1:1; ruff/mypy clean; unit test passes.
- **Definition of Done:** Single typed source of config; no hardcoded secrets; env contract documented.
- **Commit:** `feat(core): add typed application settings and env templates`
- **Complexity 3 · Risk 3 (env contract is consumed everywhere) · Context Med**

### S1-T04 — Logging & exceptions primitives

- **Objective:** Loguru-based `get_logger` and the custom exception hierarchy (`core/exceptions.py`)
  used across layers.
- **Dependencies:** T02B, T03 (log level from settings).
- **Files Expected:** `backend/app/core/logging.py`, `backend/app/core/exceptions.py`,
  `backend/tests/unit/test_logging.py`.
- **Validation Criteria:** Logger emits structured output at configured level; base + key subclasses
  defined with HTTP-mapping metadata; ruff/mypy clean; tests pass.
- **Definition of Done:** No `print`; one logger factory; exceptions ready for the API handler (T12).
- **Commit:** `feat(core): add structured logging and exception hierarchy`
- **Complexity 2 · Risk 2 · Context Low**

### S1-T05 — Async DB engine + session factory

- **Objective:** Async SQLAlchemy 2.0 engine, `async_sessionmaker`, and a `get_session` dependency
  (unit-of-work contract referenced in [`coding_standards.md` §2.5](coding_standards.md)).
- **Dependencies:** T02B, T03.
- **Files Expected:** `backend/app/core/database.py`, `backend/tests/integration/test_database.py`.
- **Validation Criteria:** `SELECT 1` over an async session against a live PG; session
  closes/rolls back correctly; ruff/mypy clean.
- **Definition of Done:** Canonical session dependency exists; no blocking calls; documented lifecycle.
- **Commit:** `feat(core): add async SQLAlchemy engine and session factory`
- **Complexity 4 · Risk 4 (async correctness) · Context Med**

### S1-T06 — Security utilities (hashing + JWT)

- **Objective:** `passlib[bcrypt]` hashing and `python-jose` access/refresh token create/verify per
  candidate [ADR-0016](#8-adrs-likely-required-during-s1).
- **Dependencies:** T02B, T03.
- **Files Expected:** `backend/app/core/security.py`, `backend/tests/unit/test_security.py`.
- **Validation Criteria:** hash↔verify round-trip; token encode/decode honors expiry & algorithm;
  tampered/expired tokens rejected; ruff/mypy clean.
- **Definition of Done:** Reusable, tested crypto primitives; no secrets logged.
- **Commit:** `feat(core): add password hashing and JWT token utilities`
- **Complexity 4 · Risk 5 (security-sensitive) · Context Med**

### S1-T07 — ORM base, naming conventions & mixins

- **Objective:** Declarative `Base` with an explicit SQLAlchemy **naming convention** (deterministic
  constraint/index names → clean autogenerate) and `UUIDPKMixin` + `TimestampMixin` matching §7
  (`gen_random_uuid()`, `server_default=now()`). **Architectural — locks patterns for all models.**
- **Dependencies:** T02B.
- **Files Expected:** `backend/app/models/__init__.py`, `backend/app/models/base.py`,
  `backend/tests/unit/test_models_base.py`.
- **Validation Criteria:** Mixins produce expected columns/server defaults; naming convention applied
  to a probe model; ruff/mypy clean.
- **Definition of Done:** One canonical Base+mixin set; **ADR-0013 written** before T08.
- **Commit:** `feat(models): add declarative base, naming convention and mixins`
- **Complexity 4 · Risk 5 (propagates to every model/migration) · Context Med**

### S1-T08 — Core domain models

- **Objective:** `User`, `UserPreferences`, `Company`, `Job` exactly per [`PLAN.md` §7](../PLAN.md)
  (types, arrays, JSONB, defaults, `__repr__`).
- **Dependencies:** T07.
- **Files Expected:** `backend/app/models/user.py`, `backend/app/models/job.py`,
  `backend/app/models/__init__.py` (exports); `backend/tests/unit/test_models_core.py`.
- **Validation Criteria:** `Base.metadata.create_all` builds these tables on a test DB;
  columns/constraints match §7; ruff/mypy clean.
- **Definition of Done:** Four models complete and FK-consistent (prefs→users, job→company).
- **Commit:** `feat(models): add user, preferences, company and job models`
- **Complexity 4 · Risk 3 · Context Med**

### S1-T09 — Application-domain models

- **Objective:** `Resume`, `ResumeVersion`, `Application`, `StatusHistory`. Note §7 models
  `resume_versions.application_id` as a **soft reference** (no hard FK) to avoid the
  resume↔application cycle — preserve that.
- **Dependencies:** T08.
- **Files Expected:** `backend/app/models/resume.py`, `backend/app/models/application.py`,
  `__init__.py` exports; `backend/tests/unit/test_models_application.py`.
- **Validation Criteria:** `create_all` succeeds with prior groups loaded; FK targets resolve
  (resume_versions→resumes/jobs; applications→users/jobs/resume_versions; status_history→applications);
  ruff/mypy clean.
- **Definition of Done:** Models complete; cyclic-FK handling documented in code + ADR-0013.
- **Commit:** `feat(models): add resume, application and status-history models`
- **Complexity 5 · Risk 4 (soft-FK/cycle) · Context Med**

### S1-T10 — Outreach, conversation & notification models

- **Objective:** `Contact`, `Referral`, `ColdEmail`, `Conversation`, `Message`, `Notification`,
  `NotificationSettings` per §7.
- **Dependencies:** T09.
- **Files Expected:** `backend/app/models/outreach.py`, `backend/app/models/conversation.py`,
  `backend/app/models/notification.py`, `__init__.py` exports;
  `backend/tests/unit/test_models_outreach.py`.
- **Validation Criteria:** Full `create_all` of the entire schema succeeds; all cross-aggregate FKs
  resolve; enum-like status strings match §7 comments; ruff/mypy clean.
- **Definition of Done:** **All §7 tables modeled**; metadata is complete for migration generation.
- **Commit:** `feat(models): add outreach, conversation and notification models`
- **Complexity 5 · Risk 3 · Context Med–High (large §7 surface)**

### S1-T11 — Alembic async env + initial migration

- **Objective:** Wire Alembic to the async engine + `Base.metadata`; autogenerate
  `0001_initial_schema` with all tables and **all §7.2 indexes**.
- **Dependencies:** T05, T10.
- **Files Expected:** `backend/alembic.ini`, `backend/alembic/env.py`,
  `backend/alembic/script.py.mako`, `backend/alembic/versions/0001_initial_schema.py`;
  `backend/tests/integration/test_migrations.py`.
- **Validation Criteria:** `alembic upgrade head` on a fresh DB; `downgrade base` clean; **second
  autogenerate yields an empty diff** (model⇄migration parity); §7.2 indexes present; ruff/mypy clean.
- **Definition of Done:** Reproducible schema via migration; parity proven; **ADR-0014 written**.
- **Commit:** `feat(db): add alembic async env and initial schema migration`
- **Complexity 6 · Risk 6 (async Alembic + autogenerate quirks) · Context Med**

### S1-T12 — FastAPI app factory + middleware + health

- **Objective:** `create_app()` with CORS/logging/timing middleware, exception handlers (from T04),
  OpenAPI metadata, `/api/v1` router aggregator, and `/health` (liveness + DB readiness).
- **Dependencies:** T03, T04, T05.
- **Files Expected:** `backend/main.py`, `backend/app/core/middleware.py`,
  `backend/app/api/__init__.py`, `backend/app/api/v1/__init__.py`, `backend/app/api/v1/router.py`;
  `backend/tests/integration/test_health.py`.
- **Validation Criteria:** `TestClient` `GET /health` → 200 with DB-ok; OpenAPI served at `/docs`;
  exception handler maps a custom exception correctly; ruff/mypy clean.
- **Definition of Done:** App boots; health green; router ready for feature routers.
- **Commit:** `feat(api): add FastAPI app factory, middleware and health endpoint`
- **Complexity 4 · Risk 3 · Context Med**

### S1-T13 — Test harness (conftest & fixtures)

- **Objective:** Async test DB provisioning, transactional rollback isolation, and an HTTPX/ASGI
  client fixture — the project-wide test contract.
- **Dependencies:** T05, T10, T12.
- **Files Expected:** `backend/tests/conftest.py`, `backend/tests/unit/__init__.py`,
  `backend/tests/integration/__init__.py`.
- **Validation Criteria:** Per-test isolation verified (a write in one test is invisible to the next);
  client fixture reaches `/health`; coverage reporting wired; ruff/mypy clean.
- **Definition of Done:** Reusable fixtures; deterministic, isolated tests; **ADR-0015 written**.
- **Commit:** `test(backend): add async test harness and database fixtures`
- **Complexity 5 · Risk 5 (async fixture isolation) · Context Med**

### S1-T14 — Authentication (FastAPI-Users)

- **Objective:** JWT auth — `schemas/user.py` (Read/Create/Update), user manager, JWT auth backend
  (access+refresh per ADR-0016), wire `auth.py` routes (`register`, `login`, `refresh`, `logout`) into
  `/api/v1`.
- **Dependencies:** T06, T08, T12, T13.
- **Files Expected:** `backend/app/schemas/__init__.py`, `backend/app/schemas/user.py`,
  `backend/app/api/v1/auth.py`, auth wiring in `app/core/security.py`/router;
  `backend/tests/integration/test_auth.py`.
- **Validation Criteria:** register → login → call protected route → refresh → logout all pass; bad
  creds/expired tokens rejected; password never returned/logged; ruff/mypy clean; auth coverage ≥
  target.
- **Definition of Done:** Working JWT auth (a core S1 DoD item); security-reviewed at CP-C.
- **Commit:** `feat(auth): add JWT authentication with fastapi-users`
- **Complexity 7 · Risk 7 (version compat + security) · Context High**

### S1-T15 — Celery app factory + Beat skeleton + health task

- **Objective:** Celery app (Redis broker/result backend), empty Beat schedule placeholder, and one
  **real** `ping`/health task (not a mock) proving worker execution.
- **Dependencies:** T03, T04.
- **Files Expected:** `backend/app/core/celery.py`, `backend/app/tasks/__init__.py`,
  `backend/app/tasks/health.py`; `backend/tests/unit/test_celery.py` (eager mode).
- **Validation Criteria:** Worker imports/boots; ping task returns expected value in eager mode; Beat
  config loads; ruff/mypy clean.
- **Definition of Done:** Task plumbing ready for agent tasks (S2+); worker/beat runnable.
- **Commit:** `feat(core): add celery app factory and health task`
- **Complexity 4 · Risk 4 · Context Med**

### S1-T16 — Backend Dockerfile

- **Objective:** Multi-stage image (uv install → slim runtime), non-root user, healthcheck, uvicorn
  entrypoint.
- **Dependencies:** T02B, T12.
- **Files Expected:** `backend/Dockerfile`, `backend/.dockerignore`, optional
  `backend/docker-entrypoint.sh`.
- **Validation Criteria:** `docker build` succeeds; container serves `/health`; image excludes
  tests/caches; non-root verified.
- **Definition of Done:** Reproducible api image consumable by compose; **ADR-0017 written**.
- **Commit:** `feat(infra): add backend Dockerfile`
- **Complexity 4 · Risk 4 · Context Med**

### S1-T17 — docker-compose: data plane

- **Objective:** `postgres`, `redis`, `qdrant`, `minio` with healthchecks, named volumes, and an
  internal bridge network.
- **Dependencies:** T01, T03.
- **Files Expected:** `infra/docker-compose.yml` (data services first), `infra/.env` wiring via
  `env_file`.
- **Validation Criteria:** `docker compose up postgres redis qdrant minio` → all healthy; volumes
  persist across restart; ports per [`PLAN.md` §12.1](../PLAN.md).
- **Definition of Done:** Stateful backing services run and persist.
- **Commit:** `feat(infra): add data-plane services to docker-compose`
- **Complexity 4 · Risk 4 (healthcheck tuning) · Context Med**

### S1-T18 — docker-compose: app/edge plane + root compose

- **Objective:** Add `api`, `celery-worker`, `celery-beat`, `flower`, `nginx`;
  `depends_on: condition: service_healthy`; migrations run on api startup; nginx reverse proxy
  (`/`→web placeholder, `/api`→api, Flower). Decide root-compose strategy (avoid the Windows symlink —
  use a canonical `infra/` file + documented `-f`, or a thin root file).
- **Dependencies:** T11, T15, T16, T17.
- **Files Expected:** `infra/docker-compose.yml` (app+edge), `infra/nginx/nginx.conf`, root
  `docker-compose.yml` (or documented compose path).
- **Validation Criteria:** **`docker-compose up` brings the full stack healthy** (S1 DoD); migrations
  auto-apply; `/health` reachable via nginx; register/login works through the proxy.
- **Definition of Done:** One-command full local stack; ordering correct; **ADR-0018 optional**.
- **Commit:** `feat(infra): add application and edge services to docker-compose`
- **Complexity 6 · Risk 6 (integration + startup ordering) · Context High**

### S1-T19 — GitHub Actions CI

- **Objective:** PR pipeline: `uv sync` → ruff format-check → ruff check → mypy → pytest+coverage,
  with postgres/redis **service containers**.
- **Dependencies:** T02B, T11, T14.
- **Files Expected:** `.github/workflows/ci.yml`.
- **Validation Criteria:** Workflow valid (`actionlint`/dry parse); mirrors
  [`development_workflow.md` §5](development_workflow.md) commands; runs migrations + full suite
  against the service DB; coverage gate enforced.
- **Definition of Done:** CI green on a PR; required-check candidate for branch protection.
- **Commit:** `ci: add lint, type-check and test pipeline`
- **Complexity 5 · Risk 5 (service containers/secrets) · Context Med**

### S1-T20 — Developer experience (Make/just + minimal seed)

- **Objective:** Encode the workflow commands (`up`, `down`, `migrate`, `test`, `lint`, `format`,
  `typecheck`) and a **minimal real** seed (one demo user) — not a mock.
- **Dependencies:** T02B, T12 (seed needs models/session; keep to one user, jobs deferred to S2).
- **Files Expected:** `Makefile` *or* `justfile`, `scripts/seed_data.py`.
- **Validation Criteria:** Each target runs the documented command; seed inserts and is idempotent.
- **Definition of Done:** One-line access to common tasks; seed runnable.
- **Commit:** `chore(dx): add task runner and minimal seed script`
- **Complexity 2 · Risk 2 · Context Low**

### S1-T21 — Sprint S1 closeout

- **Objective:** Flip S1 to ☑ in [`sprint_tracking.md`](sprint_tracking.md), update the README status
  banner (foundation → Phase 1 in progress) and quick-start (now real), and append the S1
  retrospective to [`dev_journal.md`](../dev_journal.md).
- **Dependencies:** T18, T19.
- **Files Expected:** `docs/sprint_tracking.md`, `README.md`, `dev_journal.md`.
- **Validation Criteria:** Tracking reflects reality; README quick-start verified against the running
  stack; all S1 DoD boxes ticked.
- **Definition of Done:** Sprint state truthful; history recorded; ready for S2.
- **Commit:** `docs: close out sprint S1 and record retrospective`
- **Complexity 2 · Risk 2 · Context Low–Med**

---

## 5. Risk Register

### Technical Risks

| # | Risk | L×I | Mitigation |
|---|------|:---:|-----------|
| TR1 | Async Alembic `env.py` + autogenerate quirks (greenlet, async run) | H·H | Use the official async template; gate T11 on an **empty second autogenerate diff**. |
| TR2 | FastAPI-Users ⇄ SQLAlchemy 2.0 async version drift | M·H | Pin a known-compatible matrix in T02A; integration test (T14) as the proof. |
| TR3 | Cyclic FK (resume_versions ↔ applications) | M·M | Keep `application_id` a **soft ref** per §7 (no hard FK); document in ADR-0013. |
| TR4 | uv-in-Docker layer caching / lockfile mismatch | M·M | Pin uv version; copy lock before source; CI uses the same lock. |
| TR5 | Windows host friction (CRLF, mounts, Docker perf) | M·M | `.gitattributes` (T01); develop against containers; CI on Linux is source of truth. |
| TR6 | Compose start-order (api before DB ready) | M·M | `depends_on: condition: service_healthy` + robust healthchecks (T17/T18). |
| TR7 | bcrypt/native build deps in slim image | L·M | Install build deps in the builder stage only; verify at T16. |

### Architectural Risks

| # | Risk | L×I | Mitigation |
|---|------|:---:|-----------|
| AR1 | ORM naming convention/mixins wrong → permanent migration churn | M·H | **ADR-0013 before T08**; lock convention; enforce empty-diff check. |
| AR2 | Env/config contract churn (consumed everywhere) | M·H | Freeze names from §12.2 in T03; treat renames as ADRs. |
| AR3 | Session/unit-of-work contract leaks blocking calls | M·H | Establish `get_session` contract in T05; codify in coding_standards. |
| AR4 | Token/auth shape changes ripple into S2 frontend + all routes | M·M | **ADR-0016** fixes access/refresh shape before T14. |
| AR5 | Test-DB strategy hurts speed/reliability project-wide | M·M | **ADR-0015** in T13 (transactional rollback baseline). |

---

## 6. Branch Strategy

- **Trunk-based, short-lived per-task branches** off `main`, named `<type>/s1-tNN-<slug>`
  (e.g. `feat/s1-t08-core-domain-models`, `chore/s1-t01-repo-hygiene`) — matching
  [`development_workflow.md` §3](development_workflow.md).
- **One PR per task** (or per tightly-coupled pair, e.g. T09+T10); **squash-merge** to keep `main`
  one-commit-per-task and green.
- **Branch protection** on `main`: require the T19 CI check (enable once T19 lands).
- The model chain (T07–T11) merges strictly in order; infra tasks (T15/T16/T17/T20) merge
  independently.
- **Tag `v0.1.0-alpha`** at T21 to mark the bootstrap milestone.
- Each merged task produces one [`dev_journal.md`](../dev_journal.md) entry (per-task discipline).

## 7. Review Checkpoints

| CP | After | Focus | Action |
|----|-------|-------|--------|
| **CP-A** | T03 | Foundation: layout, toolchain, **env/config contract** | Architecture sign-off before model work. |
| **CP-B** | T11 | **Schema fidelity to §7**, naming, indexes, up/down, autogenerate parity | Highest-stakes review; verify PII columns. |
| **CP-C** | T14 | Auth & API security | Run **`/security-review`**; verify JWT/hashing/protected routes, no secret leakage. |
| **CP-D** | T18 | Full-stack integration | Run **`/run`** / `/verify`: `docker-compose up`, hit `/health` via nginx, register/login. |
| **CP-E** | T21 | Sprint DoD sign-off | Confirm every S1 exit criterion; baseline coverage; retro logged. |

## 8. ADRs Likely Required During S1

| ADR | Title | Triggered by | Priority |
|-----|-------|--------------|:---:|
| **ADR-0013** | ORM conventions: UUID PKs, timestamp mixin, constraint/index naming | T07 (before T08) | Must |
| **ADR-0014** | Alembic migration strategy (async env, single baseline, autogenerate policy) | T11 | Must |
| **ADR-0015** | Test database & fixture isolation strategy | T13 | Must |
| **ADR-0016** | Authentication & token model (FastAPI-Users, access+refresh, expiry, transport) | T06/T14 | Must |
| **ADR-0017** | Container runtime & build (base image, uv-in-Docker, multi-stage, non-root) | T16 | Should |
| **ADR-0018** | Ingress & port topology (nginx single entrypoint; root-compose vs `-f`) | T18 | Optional |
| **ADR-0019** | S1 dependency scoping (defer langchain/langgraph/playwright/crawl4ai to S2–S4) | T02A | Should |

These continue the sequence after ADR-0001–0012 in [`decision_log.md`](decision_log.md).

---

## 9. Recommended Execution Order (linear, for a solo build)

```
T01 → T02A → T02B → T03 → T04 → T05 → T06 → T07 →(ADR-0013)→ T08 → T09 → T10 →
T11 →(ADR-0014)→ T12 → T13 →(ADR-0015)→ T14 →(ADR-0016, CP-C)→ T15 →
T16 →(ADR-0017)→ T17 → T18 →(CP-D)→ T19 → T20 → T21 →(CP-E)
```

Interleave T15/T17/T20 earlier if parallel capacity exists. **CP-A** lands after T03; **CP-B** after
T11.

---

## 10. See Also

- [`sprint_tracking.md`](sprint_tracking.md) — the live sprint board (S1 detail).
- [`development_workflow.md`](development_workflow.md) — the eight-step per-task workflow + Git rules.
- [`decision_log.md`](decision_log.md) — ADRs (0001–0012 today; 0013+ during S1).
- [`architecture.md`](architecture.md) — layering the S1 modules must respect.
- [`../PLAN.md`](../PLAN.md) — master specification (§5 stack, §6 structure, §7 schema, §12 deploy).
