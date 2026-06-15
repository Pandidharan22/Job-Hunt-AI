# Sprint Status — JobHunt AI

> Live snapshot of the current sprint, upcoming work, and critical path. Last updated: 2026-06-14.

---

## Current Position

- **Phase:** Phase 1 — Core Foundation & Job Discovery
- **Sprint:** **S1 — Project Bootstrap**
- **Status:** ◐ In progress (T01 done; T02 next)
- **Entry point:** [`docs/sprint_s1_execution_plan.md`](../../docs/sprint_s1_execution_plan.md)

---

## S1 Definition of Done (DoD)

All of these must be true for S1 to be complete:

- [ ] `docker-compose up` brings all services healthy (api, postgres, redis, celery, beat, qdrant, minio, flower, nginx).
- [ ] `alembic upgrade head` applies full schema; `downgrade base` is clean; second autogenerate yields empty diff.
- [ ] User registration → login → JWT → protected route access works end-to-end.
- [ ] CI runs format-check + lint + type-check + tests on every PR; green build required.
- [ ] `.env.example` documents every variable from [`PLAN.md` §12.2](../../PLAN.md).
- [ ] Backend code coverage ≥ 70% (target for Phase 1).
- [ ] All 21 tasks (T01–T21) complete + merged to `main`.
- [ ] Journal entries for all tasks + one retrospective entry at sprint close.

---

## S1 Tasks (21 Total) — Critical Path

**The spine (longest dependency run):**

```
T01 → T02 → T07 → T08 → T09 → T10 → T11 → T12 → T13 → T14 → T16 → T18 → T19 → T21
```

**Side tasks (can be parallelized):** T03, T04, T05, T06, T15, T17, T20.

### Critical Path Tasks (In Execution Order)

| # | Task | Status | Deps | What it does |
|---|------|--------|------|-------------|
| **T01** | Repo hygiene & skeleton | ☑ | — | T01A ☑ `.gitattributes`/`.editorconfig`/`.gitignore`; T01B ☑ top-level skeleton (`apps`/`backend`/`infra`/`scripts`/`.github/workflows` + `.gitkeep`) |
| **T02** | Backend project init | ☐ | T01 | `uv`, `ruff`, `mypy`, `pytest`, `pyproject.toml` |
| **T07** | ORM base, naming convention, mixins | ☐ | T02 | Declarative Base, UUID/timestamp mixins, SQL naming |
| **T08** | Core domain models | ☐ | T07 | `User`, `UserPreferences`, `Company`, `Job` |
| **T09** | Application models | ☐ | T08 | `Resume`, `ResumeVersion`, `Application`, `StatusHistory` |
| **T10** | Outreach/conversation models | ☐ | T09 | `Contact`, `Referral`, `ColdEmail`, `Conversation`, `Message`, `Notification` |
| **T11** | Alembic + migrations | ☐ | T05,T10 | `env.py`, `0001_initial_schema.py`, all §7.2 indexes |
| **T12** | FastAPI app factory | ☐ | T03,T04,T05 | `create_app()`, middleware, `/health` endpoint |
| **T13** | Test harness | ☐ | T05,T10,T12 | `conftest.py`, async DB fixtures, client fixture |
| **T14** | Authentication (JWT) | ☐ | T06,T08,T12,T13 | Register, login, refresh, logout, protected routes |
| **T16** | Backend Dockerfile | ☐ | T02,T12 | Multi-stage image, non-root, healthcheck |
| **T18** | docker-compose (full stack) | ☐ | T11,T15,T16,T17 | App + edge plane, migrations on startup, nginx reverse proxy |
| **T19** | GitHub Actions CI | ☐ | T02,T11,T14 | Format-check, lint, type-check, tests, service containers |
| **T21** | Sprint closeout | ☐ | T18,T19 | Update tracking, README, journal retrospective |

### Side Tasks (Can Parallelize with Critical Path)

| # | Task | Status | Deps | What it does |
|---|------|--------|------|-------------|
| **T03** | Core config + .env.example | ☐ | T02 | `pydantic-settings`, all env vars from §12.2 |
| **T04** | Logging + exceptions | ☐ | T02,T03 | Loguru, `get_logger()`, custom exception classes |
| **T05** | Async DB engine | ☐ | T02,T03 | SQLAlchemy async, `async_sessionmaker`, `get_session()` |
| **T06** | Security (hash + JWT) | ☐ | T02,T03 | `passlib[bcrypt]`, `python-jose` tokens |
| **T15** | Celery app factory | ☐ | T03,T04 | Celery app, Beat skeleton, ping health task |
| **T17** | docker-compose data plane | ☐ | T01,T03 | postgres, redis, qdrant, minio, healthchecks, volumes |
| **T20** | Developer experience (Make/scripts) | ☐ | T02,T12 | Makefile/justfile, seed script (one demo user) |

---

## Execution Strategy

### Wave 0 (Serial Foundation)
- T01 → T02 (repo + project init)

### Wave 1 (Parallel Divergence)
- **Config track:** T03 → (feeds T04, T05, T06, T12, T15, T17)
- **ORM track:** T07 → T08 → T09 → T10 (feeds T11, T13, T14)

### Wave 2 (Side Jobs, Parallel with Model Chain)
- T04, T05, T06 (after T03)
- T15 (after T03, T04)
- T17 (after T01, T03)
- T20 (after T02)

### Wave 3 (Integration)
- T11 (needs T05 + T10)
- T12 (needs T03 + T04 + T05)
- T13 (needs T05 + T10 + T12)
- T14 (needs T06 + T08 + T12 + T13)
- T16 (needs T02 + T12)

### Wave 4 (Full Stack + Validation)
- T18 (integration: needs T11 + T15 + T16 + T17)
- T19 (CI: needs T02 + T11 + T14)
- T21 (closeout: needs T18 + T19)

---

## Review Checkpoints

| CP | After | Focus |
|----|-------|-------|
| **CP-A** | T03 | Config contract, env/settings layout; no hardcoded secrets |
| **CP-B** | T11 | Schema fidelity to §7, indexes, up/down parity, PII columns |
| **CP-C** | T14 | JWT/hashing, no secret leaks, protected routes, auth coverage |
| **CP-D** | T18 | Full-stack: `docker-compose up`, `/health`, register/login through nginx |
| **CP-E** | T21 | All S1 DoD boxes checked; coverage ≥ 70%; journal complete |

---

## ADRs Expected During S1

| ADR | Title | When | Who decides |
|-----|-------|------|-------------|
| ADR-0013 | ORM conventions (UUID, timestamps, constraint naming) | Before T08 | Architect |
| ADR-0014 | Alembic async strategy | During T11 | Tech lead |
| ADR-0015 | Test DB fixture isolation | During T13 | QA engineer |
| ADR-0016 | Auth token model (access/refresh/expiry) | Before T14 | Security lead |
| ADR-0017 | Container runtime (base image, multi-stage, non-root) | During T16 | DevOps |
| ADR-0018 | Ingress & port topology (nginx, root compose) | During T18 | Architect |
| ADR-0019 | S1 dependency scoping (defer langchain/playwright/etc) | During T02 | Tech lead |

All ADRs follow the format in [`docs/decision_log.md`](../../docs/decision_log.md).

---

## Risk Summary

| Risk | L×I | Mitigation | Owner |
|------|:---:|-----------|-------|
| **TR1** | Async Alembic quirks | Gate T11 on empty second autogenerate diff | DB engineer |
| **TR2** | FastAPI-Users ↔ SQLAlchemy version drift | Lock known-compatible matrix in T02; T14 tests | Architect |
| **TR3** | Cyclic FK (resume_versions ↔ applications) | Keep soft ref; document in ADR-0013 | Data modeler |
| **TR4** | uv-in-Docker lockfile mismatch | Pin uv version; use same lock in CI | DevOps |
| **TR5** | Windows CRLF, Docker perf, mounts | `.gitattributes` (T01); develop against containers | Platform |
| **TR6** | Compose start-order (api before DB ready) | `depends_on: condition: service_healthy` | DevOps |
| **AR1** | ORM naming convention churn → permanent migrations | **ADR-0013 before T08**; lock + test | Architect |
| **AR2** | Config contract consumed everywhere | Freeze names in T03; treat renames as ADRs | Architect |
| **AR3** | Session contract leaks blocking calls | Establish in T05; codify in standards | Backend lead |

---

## Success Criteria for the Sprint

✓ **Infrastructure:** `docker-compose up` works; services healthy; ports right.
✓ **Database:** Full schema applied; migrations reversible; no manual steps.
✓ **Auth:** Users can register, log in, access protected routes via JWT.
✓ **Quality:** CI green; linter + types clean; tests ≥ 70%; no hardcoded secrets.
✓ **Documentation:** `.env.example` complete; all task journals filled; no contradictions.
✓ **Git:** All work on `main`, history clean, tagged `v0.1.0-alpha`.

---

## Next Sprint (S2)

Once S1 is done, **S2 — Job Scout Agent + React Shell** begins:

- **Goal:** Crawl4AI scrapers, Qdrant semantic matching, Celery Beat scheduling, React job feed UI.
- **Entry point:** [`docs/sprint_tracking.md`](../../docs/sprint_tracking.md) (S2 section).
- **Approximate tasks:** 15–20 (search, match, embed, store, UI).
- **Critical path:** Similar shape — model/agent work in parallel with frontend.

---

## Blocked Dependencies / Pre-reqs for S1

- **Python 3.12** — required for modern typing (`str | None`, etc.).
- **Docker Desktop** — required for local compose stack.
- **uv** — package manager; `pip`/`poetry` not supported.
- **GitHub repo** — remote already set up; no forking needed for core team.

**None of these should be blockers.**

---

## Key Contacts / Escalation

- **Architecture questions:** Refer to [`docs/architecture.md`](../../docs/architecture.md), [`project_rules.md`](project_rules.md), or open an ADR.
- **Coding standards violations:** See [`coding_checklist.md`](coding_checklist.md).
- **Test failures:** Don't skip/weaken; fix the code or the test.
- **Blocked by external dependency:** Escalate to tech lead; document in issue.

---

## See Also

- [`../../docs/sprint_s1_execution_plan.md`](../../docs/sprint_s1_execution_plan.md) — Full task breakdown with all 21 cards.
- [`../../docs/sprint_tracking.md`](../../docs/sprint_tracking.md) — Master sprint board for S0–S12.
- [`project_rules.md`](project_rules.md) — Architectural constraints.
- [`../../docs/development_workflow.md`](../../docs/development_workflow.md) — Eight-step task workflow.
