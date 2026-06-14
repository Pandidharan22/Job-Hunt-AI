# Project Rules — JobHunt AI

> The non-negotiable principles, architectural constraints, and coding standards that define JobHunt
> AI. This is a quick-reference distillation of [`docs/architecture.md`](../../docs/architecture.md),
> [`docs/coding_standards.md`](../../docs/coding_standards.md), and
> [`docs/decision_log.md`](../../docs/decision_log.md).

---

## Design Pillars (Inviolable)

| Pillar | What it means | When it matters |
|--------|---------------|-----------------|
| **Agent-First** | Every major function is a LangGraph agent, not a script. Agents own decisions. | Designing task queuing, refactoring linear procedures. |
| **Privacy-First** | All data stays on the user's machine. Default: local Ollama (not cloud). | Choosing where inference runs; storing user data. |
| **Human-in-the-Loop (HITL)** | Agent proposes; human approves before ANY outbound action (email, submit, DM). **Non-negotiable gate.** | Designing apply, referral, cold email, deal-close agents. |
| **Open Source** | MIT licensed, community-driven, zero vendor lock-in. | Choosing dependencies; licensing decisions. |
| **Self-Hostable** | Single `docker-compose up` on any machine. Works offline with local LLMs. | Deployment, infrastructure, feature scope. |

---

## Architectural Constraints

### 1. Clean Architecture (Dependency Rule)

Code is organized in concentric rings. **Inner rings never import outer rings.**

```
api/ ──► agents/ ──► services/ ──► external systems
  │        │           │
  └──► schemas/   models/   core/ (shared, leaf-level)
tasks/ ──► agents/, services/, core/
```

- **`app/models/`** ← SQLAlchemy ORM only. No business logic, no HTTP.
- **`app/schemas/`** ← Pydantic request/response. No ORM, no DB access.
- **`app/agents/`** ← LangGraph decision logic. Calls `services/` narrowly. No HTTP/queue details.
- **`app/services/`** ← External integration adapters (Ollama, Qdrant, MinIO, Playwright). **Zero business logic.**
- **`app/api/`** ← FastAPI routes, validation, auth, serialization. No agent logic.
- **`app/tasks/`** ← Thin Celery wrappers. Call agents/services only.
- **`app/core/`** ← Infrastructure: config, database, celery, security, exceptions. Importable everywhere. Imports nothing from agents/api/tasks.

**Violation:** If an agent directly calls an API route, or a service contains business logic, it's a
design bug.

### 2. Agent Orchestration Contract

Every agent extends `BaseAgent` and implements `async def run(context: dict) -> AgentResult`.

```python
@dataclass
class AgentResult:
    status: str                 # "success" | "failed" | "pending_approval" | "skipped"
    data: dict[str, Any]
    next_action: str | None = None
    error: str | None = None
```

- Control flow lives in the **LangGraph state machine**, not in tasks or routes.
- Tasks and routes only trigger agents; they don't branch on the result (agent decides next steps).

### 3. Human-in-the-Loop on Outbound Actions

**Every** outbound action passes through an approval gate:

- Applications submitted (Apply Agent) → approval modal, user approves before submit.
- Emails/DMs sent (Referral, Cold Email, Deal Closer) → user sees draft, approves before send.
- Auto modes **opt-in only**; default is **approval required**.

**Violation:** An email sent without user approval is a critical bug, not a feature.

### 4. No Resume Fabrication

The Resume Tailor Agent **only rephrases and reorders** existing resume content. It never:

- Adds experience the user hasn't listed.
- Changes dates, company names, or job titles.
- Fabricates metrics or achievements.

The system prompt **must** include this rule verbatim. ATS scoring iterates until ≥80%, but always
within the bounds of truthfulness.

---

## Coding Standards (Quick Version)

### Python Backend

| Rule | Why | How to check |
|------|-----|-------------|
| **No placeholder code** | Every line committed is real, working code. | Search for `pass`, `return None`, `...`, `TODO`, `FIXME`. |
| **No `TODO`/`FIXME`** | Task tracking lives in sprint board + issues, not comments. | `grep -r "TODO\|FIXME" backend/app/` — must be empty. |
| **Type-safe end-to-end** | mypy strict, Pydantic v2 everywhere, no `any`. | `uv run mypy app/` must be zero errors. |
| **ruff format-clean** | `ruff` is the source of truth for formatting. | `uv run ruff format --check .` must pass. |
| **No hardcoded secrets** | Env vars only, no real credentials in repo. | `grep -r "api_key\|password\|secret" backend/app/` — only in config defaults. |
| **Async end-to-end** | No blocking I/O on async paths. | Use `httpx`, `sqlalchemy[asyncio]`, `aioimaplib`, not `requests`, blocking DB, or `smtplib`. |
| **Structured logging** | Loguru only, never `print()`. No secrets logged. | One logger per module: `self.logger = get_logger(__name__)`. |
| **Custom exceptions** | Business errors are custom classes from `core/exceptions.py`. | Mapped to HTTP responses at API boundary. |
| **Tests in the same change** | Code + tests shipped together. Coverage ≥ 70% Phase 1, ≥ 90% Phase 6. | `uv run pytest tests/ --cov=app` must meet target. |
| **Docstrings on public APIs** | Google-style, state **what** and **why**, not just signature. | Every public function/class/module has a docstring. |

### Git & Commits

| Rule | Why | How to check |
|------|-----|-------------|
| **Conventional Commits** | Parsable, automatable changelog; clear intent. | `git log --oneline` reads as a list of deliberate changes. |
| **Branch per task** | One logical concern per branch; easy to review and revert. | Branch name: `<type>/s1-tNN-<slug>` (e.g., `feat/s1-t08-models`). |
| **No skipped steps** | Validation, tests, docs, journal, commit all in order. | Every commit has a paired journal entry. |
| **Squash-merge to main** | `main` is a series of feature commits, each standalone. | One commit per merged PR; CI passes on `main` always. |

---

## Non-Negotiable Decision Records

These ADRs are binding. Violating them is a design bug, not a choice.

| ADR | Decision | Binding? |
|-----|----------|----------|
| ADR-0001 | Documentation-driven dev OS (S0 + dev journal) | **Yes** |
| ADR-0005 | Local-first Ollama (privacy default) | **Yes** |
| ADR-0009 | HITL approval gates on outbound actions | **Yes** |
| ADR-0012 | Clean Architecture layering | **Yes** |

Others (ADR-0002–0004, 0006–0008, 0010–0011) are foundational but have pragmatic fallbacks (e.g.,
ADR-0005 allows Groq as a hosted fallback if local is too slow).

---

## Sprint Workflow

1. **Current sprint:** S1 — Project Bootstrap (21 tasks, dependency-ordered).
2. **Task execution:** Follow the eight-step workflow in [`docs/development_workflow.md`](../../docs/development_workflow.md):
   - Analyze → Implement → Validate → Test → Document → Journal → Commit → Verify
3. **Task tracking:** [`docs/sprint_tracking.md`](../../docs/sprint_tracking.md) + [`docs/sprint_s1_execution_plan.md`](../../docs/sprint_s1_execution_plan.md).
4. **Never skip steps.** If a step fails, report it honestly and stop.

---

## Architecture at a Glance

```
┌─────────────────────────────────────────┐
│         PRESENTATION LAYER              │
│ React · Tauri · Expo · Ntfy             │
└────────────────┬────────────────────────┘
                 │ HTTP / WebSocket
┌────────────────▼────────────────────────┐
│      API GATEWAY LAYER                  │
│ FastAPI · WebSocket · JWT               │
└────────────────┬────────────────────────┘
                 │ Python calls
┌────────────────▼────────────────────────┐
│   AGENT ORCHESTRATION LAYER             │
│ Job Scout · Resume Tailor · Apply · ... │
└────────────────┬────────────────────────┘
                 │ Task dispatch
┌────────────────▼────────────────────────┐
│   TASK / MESSAGE QUEUE                  │
│ Celery · Redis · Celery Beat            │
└────────────────┬────────────────────────┘
                 │ DB / AI
┌────────────────▼────────────────────────┐
│    DATA + AI LAYER                      │
│ PostgreSQL · Qdrant · Redis · MinIO     │
│ Ollama · Vault                          │
└────────────────┬────────────────────────┘
                 │ HTTP / browser
┌────────────────▼────────────────────────┐
│  SCRAPING / BROWSER AUTOMATION          │
│ Playwright · Crawl4AI · IMAP/SMTP       │
└─────────────────────────────────────────┘
```

- **Layers are unidirectional.** API → Agents → Services → External.
- **Agents make decisions.** Tasks dispatch; routes validate.
- **Services integrate.** No business logic in services.

---

## Key File Locations (Quick Map)

| Concern | File(s) |
|---------|---------|
| **Configuration** | `backend/app/core/config.py`, `backend/.env.example` |
| **Database schema** | `backend/app/models/`, `backend/alembic/versions/` |
| **Authentication** | `backend/app/core/security.py`, `backend/app/api/v1/auth.py` |
| **Agents** | `backend/app/agents/base_agent.py`, `backend/app/agents/*.py` |
| **API routes** | `backend/app/api/v1/*.py` |
| **Tasks** | `backend/app/tasks/*.py` |
| **Services** | `backend/app/services/*.py` |
| **Tests** | `backend/tests/{unit,integration,e2e}/` |
| **Logs** | Structured via Loguru, routed to stdout + files (configurable). |

---

## When Something Feels Wrong

**It's probably an architectural violation.** Use this checklist:

- [ ] Does a service contain business logic? → Move it to an agent.
- [ ] Does an agent know about HTTP or Celery internals? → Extract to a task wrapper.
- [ ] Does the code use blocking I/O on an async path? → Use async libraries.
- [ ] Is there a `TODO` or placeholder? → Complete or track as a separate task.
- [ ] Does an action happen without HITL approval? → Add an approval gate.
- [ ] Does the resume tailor add experience? → That's a bug; remove it.

If you're unsure, open an ADR discussion before committing.
