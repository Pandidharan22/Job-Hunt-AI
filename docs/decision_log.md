# Decision Log — JobHunt AI

> This is the project's **Architecture Decision Record (ADR)** log. Each ADR captures one significant
> technical decision: its context, the decision itself, and its consequences. ADRs are **append-only
> and immutable** — to change a decision, add a new ADR that supersedes the old one (and mark the old
> one `Superseded by ADR-XXXX`). Never rewrite history.
>
> See [`development_workflow.md` §6](development_workflow.md) for *when* to write an ADR.

---

## ADR Format

Every ADR uses this template:

```markdown
## ADR-XXXX — <Short Title>

- **Status:** Proposed | Accepted | Deprecated | Superseded by ADR-YYYY
- **Date:** YYYY-MM-DD
- **Deciders:** <who>
- **Context:** <the forces at play — the problem, constraints, and requirements>
- **Decision:** <what we decided to do>
- **Consequences:** <the results — positive, negative, and follow-ups>
- **Alternatives considered:** <what else we evaluated and why we rejected it>
- **References:** <links to PLAN.md sections, docs, external sources>
```

**Status definitions**

| Status | Meaning |
|--------|---------|
| **Proposed** | Under discussion; not yet in force. |
| **Accepted** | In force; code must comply. |
| **Deprecated** | Discouraged but not yet removed. |
| **Superseded** | Replaced by a later ADR (named). |

---

## Index

| ID | Title | Status | Date |
|----|-------|--------|------|
| [ADR-0001](#adr-0001--adopt-a-documentation-driven-development-operating-system) | Adopt a documentation-driven development operating system | Accepted | 2026-06-14 |
| [ADR-0002](#adr-0002--use-a-monorepo) | Use a monorepo | Accepted | 2026-06-14 |
| [ADR-0003](#adr-0003--python-312--fastapi--async-sqlalchemy-20-backend) | Python 3.12 + FastAPI + async SQLAlchemy 2.0 backend | Accepted | 2026-06-14 |
| [ADR-0004](#adr-0004--langgraph-agents-on-a-shared-baseagent-contract) | LangGraph agents on a shared BaseAgent contract | Accepted | 2026-06-14 |
| [ADR-0005](#adr-0005--local-first-llm-inference-via-ollama) | Local-first LLM inference via Ollama | Accepted | 2026-06-14 |
| [ADR-0006](#adr-0006--celery--redis-for-event-driven-task-execution) | Celery + Redis for event-driven task execution | Accepted | 2026-06-14 |
| [ADR-0007](#adr-0007--qdrant-for-semantic-job-matching) | Qdrant for semantic job matching | Accepted | 2026-06-14 |
| [ADR-0008](#adr-0008--postgresql-16-primary-store-minio-for-objects) | PostgreSQL 16 primary store, MinIO for objects | Accepted | 2026-06-14 |
| [ADR-0009](#adr-0009--human-in-the-loop-approval-gates-on-all-outbound-actions) | Human-in-the-loop approval gates on all outbound actions | Accepted | 2026-06-14 |
| [ADR-0010](#adr-0010--conventional-commits-mandatory-dev-journal-and-adrs) | Conventional Commits, mandatory dev journal, and ADRs | Accepted | 2026-06-14 |
| [ADR-0011](#adr-0011--uv--ruff--mypy-strict--pytest-toolchain) | uv + ruff + mypy strict + pytest toolchain | Accepted | 2026-06-14 |
| [ADR-0012](#adr-0012--clean-architecture-layering) | Clean Architecture layering | Accepted | 2026-06-14 |

---

## ADR-0001 — Adopt a documentation-driven development operating system

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** JobHunt AI is a large, multi-phase, multi-agent system that will be built incrementally
  over twelve sprints, partly with AI coding assistants that start each session without memory of
  prior work. Without an explicit operating system — context, standards, workflow, decision history —
  knowledge is lost between sessions and quality drifts.
- **Decision:** Before writing any application code, establish a fixed set of governance artifacts:
  `README.md`, `CONTRIBUTING.md`, `docs/project_context.md`, `docs/architecture.md`,
  `docs/coding_standards.md`, `docs/development_workflow.md`, `docs/sprint_tracking.md`,
  `docs/decision_log.md`, and `dev_journal.md`. Every task thereafter follows the eight-step workflow
  and appends a journal entry.
- **Consequences:** (+) Any contributor — human or AI — can onboard from the docs alone; the journal
  becomes a complete engineering history; decisions are traceable. (−) Per-task overhead of updating
  docs and the journal; discipline required to never skip steps.
- **Alternatives considered:** *Code-first, document-later* — rejected because context loss across
  sessions and contributors would compound quickly. *Wiki/external tracker only* — rejected to keep
  the operating system version-controlled alongside the code it governs.
- **References:** [`PLAN.md`](../PLAN.md); [`development_workflow.md`](development_workflow.md);
  [`dev_journal.md`](../dev_journal.md).

## ADR-0002 — Use a monorepo

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** The product spans a Python backend, a React web app, a Tauri desktop app, an Expo
  mobile app, shared infrastructure, and docs that must stay in lockstep across releases.
- **Decision:** Keep all components in a single repository with the structure defined in
  [`PLAN.md` §6](../PLAN.md) (`apps/`, `backend/`, `infra/`, `docs/`, `scripts/`, `.github/`).
- **Consequences:** (+) Atomic cross-cutting changes, one CI pipeline, shared docs and tooling,
  simpler self-hosting. (−) Larger checkout; CI must scope jobs by path to stay fast.
- **Alternatives considered:** *Polyrepo* — rejected for the coordination cost of versioning the API
  contract across many repos for a solo/early project.
- **References:** [`PLAN.md` §6](../PLAN.md); [`architecture.md`](architecture.md).

## ADR-0003 — Python 3.12 + FastAPI + async SQLAlchemy 2.0 backend

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** The backend is I/O-bound (scraping, LLM calls, email, DB) and must expose a typed REST
  API plus a WebSocket gateway, with first-class async support and OpenAPI docs.
- **Decision:** Build on Python 3.12, FastAPI (async, Pydantic-native), SQLAlchemy 2.0 async ORM with
  Alembic migrations, and Pydantic v2 at every boundary.
- **Consequences:** (+) High concurrency for I/O work, auto-generated OpenAPI, strong typing, mature
  ecosystem for AI/automation. (−) Async-everywhere discipline required; no blocking calls on async
  paths (enforced in [`coding_standards.md`](coding_standards.md)).
- **Alternatives considered:** *Node/NestJS* — rejected because the AI/agent/automation ecosystem is
  strongest in Python. *Django* — rejected as heavier and less async-native for this workload.
- **References:** [`PLAN.md` §5.1](../PLAN.md).

## ADR-0004 — LangGraph agents on a shared BaseAgent contract

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** Each capability needs multi-step, stateful, branching control flow with tool use and
  LLM decisions — not linear scripts. We need a uniform way to invoke, observe, and test agents.
- **Decision:** Every agent is a LangGraph state machine and extends a single `BaseAgent` ABC exposing
  `async def run(context) -> AgentResult` ([`PLAN.md` §8.1](../PLAN.md)). Control flow lives in the
  graph; tasks and the API only trigger and collect results.
- **Consequences:** (+) Consistent contract, testable nodes, observable state, clear separation of
  decision logic from delivery. (−) LangGraph learning curve; graphs must be kept readable.
- **Alternatives considered:** *Plain function pipelines* — rejected for poor handling of branching,
  retries, and human-in-the-loop pauses. *Raw LangChain chains* — rejected as less suited to stateful,
  cyclic flows (e.g. the ATS-score iteration loop).
- **References:** [`PLAN.md` §8](../PLAN.md); [`architecture.md` §4](architecture.md).

## ADR-0005 — Local-first LLM inference via Ollama

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** The product handles a user's resume, inbox, and private outreach. Privacy is a core
  promise, and self-hosting must work offline. LLM cost must not gate usage.
- **Decision:** Default all inference to local Ollama models — `llama3.1:8b` for speed-critical tasks,
  `llama3.1:70b` for quality-critical tasks, `nomic-embed-text` for embeddings. Provide a drop-in
  hosted fallback (Groq) selectable via environment variable, with **no code changes**.
- **Consequences:** (+) Privacy by default, zero per-token cost when self-hosted, offline capability.
  (−) Large models are slow/heavy on CPU-only machines — mitigated by the 8B/70B split and the Groq
  fallback ([`PLAN.md` §14](../PLAN.md)).
- **Alternatives considered:** *Cloud-LLM-only* (OpenAI/Anthropic) — rejected as incompatible with the
  privacy-first and free-self-host pillars, though supported as an optional hosted tier.
- **References:** [`PLAN.md` §5.2, §12.5, §14](../PLAN.md).

## ADR-0006 — Celery + Redis for event-driven task execution

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** Agent work is long-running and must run off the request path, on schedules (every 6h,
  every 15m, weekly), with retries and horizontal scaling.
- **Decision:** Use Celery workers with a Redis broker for async tasks, Celery Beat for scheduling,
  and Flower for monitoring. Redis also serves caching, rate-limit counters, and pub/sub.
- **Consequences:** (+) Mature, scalable, observable async execution; one dependency (Redis) covers
  broker + cache + pub/sub. (−) Operational surface of a worker + beat + broker; tasks must stay thin
  (logic lives in agents).
- **Alternatives considered:** *FastAPI BackgroundTasks* — rejected: no scheduling, retries, or
  scaling. *RabbitMQ* — rejected to avoid a second infrastructure dependency when Redis suffices.
- **References:** [`PLAN.md` §3, §5.4](../PLAN.md).

## ADR-0007 — Qdrant for semantic job matching

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** Matching jobs to a user profile requires semantic similarity over embeddings, scored
  and filtered at scale, in a self-hostable component.
- **Decision:** Use Qdrant as the vector database; embed JDs and the user profile with
  `nomic-embed-text` and score with cosine similarity.
- **Consequences:** (+) Purpose-built vector search, self-hostable via Docker, good Python client.
  (−) Another stateful service to run and back up.
- **Alternatives considered:** *pgvector* — viable and one-less-service, but Qdrant offers richer
  vector features and operational clarity for this workload. *Pinecone* — rejected: hosted-only,
  conflicts with self-host/privacy pillars.
- **References:** [`PLAN.md` §5.2, §5.4, §8.2](../PLAN.md).

## ADR-0008 — PostgreSQL 16 primary store, MinIO for objects

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** The domain is highly relational (users, jobs, applications, status history, outreach,
  conversations) and also needs binary artifact storage (resume PDFs/DOCX, screenshots).
- **Decision:** PostgreSQL 16 is the primary relational store ([`PLAN.md` §7](../PLAN.md)); MinIO
  (S3-compatible, self-hosted) stores binary artifacts. Files are referenced by path from Postgres rows.
- **Consequences:** (+) Strong relational integrity + JSONB flexibility; S3 semantics without a cloud
  dependency. (−) Two storage systems to operate and back up.
- **Alternatives considered:** *Files in Postgres (bytea/large objects)* — rejected: bloats the DB and
  complicates backups. *Cloud S3* — rejected for the self-host/privacy pillars.
- **References:** [`PLAN.md` §5.4, §7](../PLAN.md).

## ADR-0009 — Human-in-the-loop approval gates on all outbound actions

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** The system can submit applications and send emails/DMs on the user's behalf. Mistakes
  here are high-impact and hard to reverse, and there are legal/ethical limits (CAN-SPAM, GDPR, ToS).
- **Decision:** Every outbound action (application submit, referral message, cold email, deal reply)
  passes through an explicit, configurable approval gate that is **on by default**. The agent prepares
  a preview; the human approves before anything is sent. Auto modes are opt-in and capped.
- **Consequences:** (+) Safety, user trust, legal/ethical compliance, protection from LLM mistakes.
  (−) Less "fully autonomous" by default — an intentional trade-off. Approval flows add UI/eventing
  work (WebSocket `approval_required`, 24h timeout).
- **Alternatives considered:** *Full autonomy by default* — rejected as unsafe and non-compliant.
- **References:** [`PLAN.md` §2.3, §8.4, §14](../PLAN.md); [`architecture.md` §9](architecture.md).

## ADR-0010 — Conventional Commits, mandatory dev journal, and ADRs

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** A long-lived, multi-contributor project needs a legible history and traceable
  rationale, including for AI-assisted sessions that lack persistent memory.
- **Decision:** All commits use Conventional Commits ([`development_workflow.md` §4](development_workflow.md));
  every completed task appends a [`dev_journal.md`](../dev_journal.md) entry; every significant
  decision is recorded as an ADR here.
- **Consequences:** (+) Readable history, automatable changelogs/releases, full traceability.
  (−) Per-commit and per-task discipline; minor overhead.
- **Alternatives considered:** *Freeform commit messages* — rejected for poor traceability and no
  changelog automation.
- **References:** [`development_workflow.md`](development_workflow.md); [`dev_journal.md`](../dev_journal.md).

## ADR-0011 — uv + ruff + mypy strict + pytest toolchain

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** Code quality must be enforced mechanically and fast, with minimal tool sprawl.
- **Decision:** Use `uv` for packaging/venv, `ruff` for linting **and** formatting (line-length 100,
  `select = ALL` with justified ignores), `mypy` in strict mode, and `pytest`/`pytest-asyncio`/
  `pytest-cov` for tests. All configured in `backend/pyproject.toml`; enforced in CI.
- **Consequences:** (+) One fast linter/formatter replaces flake8+black+isort; strict typing catches
  whole classes of bugs; reproducible installs. (−) `select = ALL` is strict — some rules need
  documented, justified ignores.
- **Alternatives considered:** *pip/poetry + flake8 + black + isort* — rejected as slower and more
  tools to maintain than the `uv` + `ruff` combination.
- **References:** [`PLAN.md` §5.1, §5.7](../PLAN.md); [`coding_standards.md`](coding_standards.md).

## ADR-0012 — Clean Architecture layering

- **Status:** Accepted
- **Date:** 2026-06-14
- **Deciders:** Pandidharan G R
- **Context:** With eight agents and many external integrations, business logic must not entangle with
  delivery mechanisms or third-party SDKs, or the system becomes untestable and brittle.
- **Decision:** Enforce Clean Architecture: `models`/`schemas` (entities) ← `agents` (use cases) ←
  `api`/`tasks` (adapters) ← `services`/`core` (frameworks & drivers). Dependencies point inward;
  `services/` hold no business logic. Mapping in [`architecture.md` §3](architecture.md).
- **Consequences:** (+) Testable core, swappable integrations, clear ownership of logic. (−) Some
  boilerplate at boundaries; contributors must learn the import rules.
- **Alternatives considered:** *Pragmatic/flat layering* — rejected: at this system's size, decision
  logic would leak into routes and SDK calls, harming testability.
- **References:** [`PLAN.md` §3, §6](../PLAN.md); [`architecture.md`](architecture.md);
  [`coding_standards.md` §2.9](coding_standards.md).
