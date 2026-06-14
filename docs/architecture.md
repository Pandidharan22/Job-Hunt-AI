# Architecture — JobHunt AI

> This document describes the system's structure, its layering rules, and how the pieces communicate.
> It distills and operationalizes [`PLAN.md` §4–§9](../PLAN.md). For the authoritative, exhaustive
> specification (full DB schema, every endpoint, every agent flow), read `PLAN.md` directly.

---

## 1. Architectural Style

JobHunt AI combines three complementary styles:

- **Clean Architecture** — dependencies point inward. Business rules (agents, models) do not depend
  on delivery mechanisms (API, browser, scrapers). External systems sit behind `services/` adapters.
- **Agent-First** — each major capability is an autonomous LangGraph state machine, not a procedural
  script. Agents own decisions; tasks and the API only *trigger* and *collect results*.
- **Event-Driven** — work is dispatched as asynchronous Celery tasks over a Redis broker. Recurring
  work is scheduled by Celery Beat. Real-time updates fan out over Redis pub/sub to WebSockets.

## 2. High-Level Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
│  React Web UI  │  Tauri Desktop App  │  Expo Mobile App     │
│                │  Ntfy Notifications │                      │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP / WebSocket
┌──────────────────────────▼──────────────────────────────────┐
│                  API GATEWAY LAYER                           │
│  FastAPI REST API  │  WebSocket Gateway  │  JWT Auth        │
└──────────────────────────┬──────────────────────────────────┘
                           │ Python function calls
┌──────────────────────────▼──────────────────────────────────┐
│              AGENT ORCHESTRATION LAYER                       │
│  Job Scout │ Resume Tailor │ Apply │ Email Parser │ ...      │
└──────────────────────────┬──────────────────────────────────┘
                           │ Task dispatch
┌──────────────────────────▼──────────────────────────────────┐
│                TASK / MESSAGE QUEUE                          │
│  Celery Workers  │  Redis Broker  │  Celery Beat (cron)     │
└──────────────────────────┬──────────────────────────────────┘
                           │ DB / AI calls
┌──────────────────────────▼──────────────────────────────────┐
│               DATA + AI LAYER                                │
│  PostgreSQL 16 │ Qdrant │ Redis │ MinIO │ Ollama │ Vault    │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP / browser
┌──────────────────────────▼──────────────────────────────────┐
│          SCRAPING / BROWSER AUTOMATION LAYER                 │
│  Playwright │ Crawl4AI │ Proxy Rotator │ IMAP Watcher       │
└─────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

| Layer | Owns | Must NOT |
|-------|------|----------|
| **Presentation** | Rendering, user interaction, optimistic UI, WebSocket consumption. | Contain business rules. |
| **API Gateway** | Request validation (Pydantic), auth (JWT), routing, rate limiting, serialization, WebSocket fan-out. | Implement agent logic or talk to external sites directly. |
| **Agent Orchestration** | Decision loops, LangGraph state, prompt construction, tool selection, result shaping. | Know about HTTP, ORM sessions' lifecycle, or queue mechanics beyond its `run(context)` contract. |
| **Task / Queue** | Async dispatch, scheduling (Beat), retries, backoff, concurrency. | Hold business logic — tasks are thin wrappers that call agents/services. |
| **Data + AI** | Persistence, vector search, caching, object storage, LLM inference, secrets. | Reach back up into agents or the API. |
| **Scraping / Browser** | Fetching external pages, filling forms, watching mailboxes. | Contain orchestration decisions — they are tools invoked by agents via `services/`. |

## 3. Clean Architecture Mapping

The backend folder layout (see [`PLAN.md` §6](../PLAN.md)) maps to Clean Architecture rings. The
**dependency rule**: inner rings never import outer rings.

| Ring (inner → outer) | Backend module | Role |
|----------------------|----------------|------|
| **Entities** | `app/models/` (SQLAlchemy), `app/schemas/` (Pydantic) | Core data shapes and persistence models. |
| **Use cases** | `app/agents/` | Application-specific business rules; the decision loops. |
| **Interface adapters** | `app/api/`, `app/tasks/` | Translate the outside world (HTTP, queue) into use-case calls. |
| **Frameworks & drivers** | `app/services/`, `app/core/` | Concrete integrations: Ollama, Qdrant, MinIO, Playwright, IMAP/SMTP, DB engine, Celery, config, security. |

**Allowed import directions**

```
api/  ──►  agents/  ──►  services/  ──►  (external systems)
  │           │             │
  └──►  schemas/      models/      core/   (shared, leaf-level)
tasks/ ──► agents/, services/      core/
```

- `services/` contain **no business logic** — only external-integration mechanics (per `PLAN.md` §6).
- `agents/` depend on `services/` through narrow tool functions, never on `api/` or `tasks/`.
- `core/` (config, database, security, celery, websocket, exceptions, middleware) is leaf-level
  infrastructure importable from anywhere, but imports nothing from `agents/`, `api/`, or `tasks/`.

## 4. Agent Orchestration Pattern

Every agent extends a single `BaseAgent` contract (full code in [`PLAN.md` §8.1](../PLAN.md)):

```python
@dataclass
class AgentResult:
    status: str                 # "success" | "failed" | "pending_approval" | "skipped"
    data: dict[str, Any]
    next_action: str | None = None
    error: str | None = None

class BaseAgent(ABC):
    @abstractmethod
    async def run(self, context: dict[str, Any]) -> AgentResult: ...
```

Internally each agent is a **LangGraph state machine**: nodes call tools or make LLM decisions, edges
route conditionally, and a final node persists results. The graph is the single place where an
agent's control flow lives — no business branching leaks into tasks or the API.

## 5. Communication Pattern

```
FastAPI Route
     │  (validates, authorizes)
     ▼
Celery Task (async, retriable)
     │  (thin wrapper)
     ▼
Agent.run(context: dict)
     │
     ├── LangGraph State Machine
     │        ├── Node: call_tool(name, args)     # via services/
     │        ├── Node: llm_decision(prompt, state)
     │        ├── Edge: conditional routing
     │        └── Node: persist_result(db)
     │
     └── Returns AgentResult(status, data, next_action)
```

Real-time results propagate outward via **Redis pub/sub → WebSocket ConnectionManager → clients**,
and via **Ntfy** for mobile/desktop push.

## 6. End-to-End Data Flow (Condensed)

The canonical 21-step flow is in [`PLAN.md` §4.2](../PLAN.md). In brief:

1. **Discover** — Beat triggers Job Scout → Crawl4AI scrapes boards → LLM structures jobs.
2. **Match** — JDs embedded (`nomic-embed-text`) → scored against profile in Qdrant → top matches
   persisted → WebSocket "new jobs" push.
3. **Tailor** — Resume Tailor rewrites resume to JD, runs the ATS-score loop, generates a cover
   letter, exports PDF/DOCX to MinIO.
4. **Apply** — Apply Agent drives Playwright, detects ATS type, fills the form, sends a preview for
   **HITL approval**, submits on approval, screenshots confirmation to MinIO.
5. **Outreach** — Referral Agent finds and scores contacts, queues approval-gated message sequences.
6. **Track** — Email Parser watches IMAP, classifies replies, updates status, writes `status_history`.
7. **Notify** — WebSocket + Ntfy push on every state change.
8. **Close** — Deal Closer classifies intent on inbound replies, drafts approval-gated responses,
   coaches negotiation until the deal is Closed Won/Lost.

## 7. Data Stores and Their Roles

| Store | Role |
|-------|------|
| **PostgreSQL 16** | Primary relational store: users, jobs, applications, status history, outreach, conversations, notifications. Full schema in [`PLAN.md` §7](../PLAN.md). |
| **Qdrant** | Vector database for semantic job ↔ profile matching. |
| **Redis 7** | Celery broker, cache, rate-limit counters, and pub/sub for real-time events. |
| **MinIO** | S3-compatible object storage for resume PDFs/DOCX and confirmation screenshots. |
| **Ollama** | Local LLM inference server (`llama3.1:8b`, `llama3.1:70b`, `nomic-embed-text`). |
| **Vault / OS keychain** | Encrypted secret storage for user credentials. |

## 8. Cross-Cutting Concerns

| Concern | Approach |
|---------|----------|
| **Auth** | JWT via `fastapi-users`; access + refresh tokens; WebSocket auth via query-param token. |
| **Config** | `pydantic-settings` reads all environment variables (see [`PLAN.md` §12.2](../PLAN.md)). No hardcoded secrets. |
| **Logging** | Structured logging via Loguru; one logger per agent/service; never log secrets. |
| **Observability** | Prometheus metrics on agents and endpoints; Grafana dashboards; Flower for Celery. |
| **Security** | Encrypted credential vault, input validation everywhere, per-domain rate limits, HITL gates. |
| **Error handling** | Custom exceptions in `core/exceptions.py`; retries with exponential backoff on external calls. |

## 9. Key Architectural Constraints

These are **invariants**. Violating them is a design bug, not a style choice.

1. **Human-in-the-Loop on all outbound actions.** No application is submitted and no email/DM is sent
   without an explicit approval gate (configurable, on by default). See
   [ADR-0009](decision_log.md).
2. **Privacy-first.** Default inference is local (Ollama). No user data leaves the machine unless the
   user opts into a hosted LLM. See [ADR-0005](decision_log.md).
3. **No fabrication in resumes.** The Resume Tailor only rephrases/reorders existing content; the
   no-fabrication system prompt is mandatory and verbatim (see [`PLAN.md` §8.3](../PLAN.md)).
4. **Type-safe end to end.** mypy strict on the backend, TypeScript strict on the frontend,
   Pydantic v2 at every boundary.
5. **Services hold no business logic.** Decisions live in agents; integrations live in services.

## 10. Technology Stack (Summary)

Full versions and rationale in [`PLAN.md` §5](../PLAN.md).

- **Backend:** Python 3.12, FastAPI, SQLAlchemy 2.0 (async), Alembic, Pydantic v2, Celery, Redis.
- **AI:** LangChain, LangGraph, Ollama, sentence-transformers, Qdrant, tiktoken.
- **Automation:** Playwright, Crawl4AI, aioimaplib, aiosmtplib.
- **Frontend:** React 19, Vite, TypeScript, Tailwind, shadcn/ui, TanStack Query/Table, Zustand.
- **Desktop/Mobile:** Tauri 2, Expo (React Native), Ntfy.
- **DevOps:** Docker Compose, GitHub Actions, Prometheus, Grafana, Nginx, `uv`.

## 11. See Also

- [`project_context.md`](project_context.md) — why the project exists.
- [`coding_standards.md`](coding_standards.md) — how layers are implemented in code.
- [`decision_log.md`](decision_log.md) — why each architectural choice was made.
