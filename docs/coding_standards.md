# Coding Standards — JobHunt AI

> These standards are binding for all code in this repository, written by humans or AI agents. They
> exist to keep the codebase maintainable, type-safe, and consistent. Pair this with
> [`development_workflow.md`](development_workflow.md) (the per-task process) and
> [`architecture.md`](architecture.md) (the layering rules).

---

## 1. Quality Rules (Absolute)

These are the project's bedrock quality rules. They are not negotiable and apply to every change.

1. **No placeholder code.** Every line committed must be real, working code. No stub functions that
   `pass`, no `return None  # implement later`, no scaffolding left half-built.
2. **No `TODO` / `FIXME` comments.** If something needs doing, either do it now or track it in
   [`sprint_tracking.md`](sprint_tracking.md) / an issue / an ADR. The code itself stays clean.
3. **No mock implementations unless explicitly requested.** Do not fake a function's behavior. Mocks
   belong in tests, or in code only when the user explicitly asks for a mock/stub.
4. **Keep files maintainable.** Prefer small, focused modules. If a file grows past readability,
   split it along responsibility lines.
5. **Follow Clean Architecture.** Respect the dependency rules in [`architecture.md`](architecture.md):
   inner rings never import outer rings; `services/` hold no business logic.
6. **Prefer simplicity over abstraction.** Do not add layers, generics, or indirection for
   hypothetical future needs. Write the simplest thing that correctly solves the present problem.
7. **Explain every major decision.** Significant choices get an ADR in
   [`decision_log.md`](decision_log.md) and a note in the [`dev_journal.md`](../dev_journal.md) entry.

## 2. Python (Backend)

### 2.1 Toolchain

| Tool | Role | Configuration |
|------|------|---------------|
| **Python 3.12** | Runtime. | — |
| **`uv`** | Package & venv management (replaces pip/poetry). | `pyproject.toml` |
| **`ruff`** | Linter **and** formatter (replaces flake8, black, isort). | line-length = 100, `select = ALL` (with documented, justified ignores) |
| **`mypy`** | Static type checking in **strict** mode. | `pyproject.toml`, strict = true |
| **`pytest`** + `pytest-asyncio` + `pytest-cov` | Testing. | `pyproject.toml` |

All tooling is configured in `backend/pyproject.toml` (lands in Sprint S1).

### 2.2 Typing

- **Full type annotations** on every function signature, parameter, and return value.
- mypy strict must pass with **zero errors**. Do not use `# type: ignore` without a trailing reason
  comment and, where it recurs, an ADR.
- Use modern syntax: `str | None`, `list[str]`, `dict[str, Any]` (no `Optional`, `List`, `Dict`).
- **Pydantic v2** for all request/response schemas, settings, and structured LLM I/O. Use
  `model_config`, `Field`, and validators idiomatically.

### 2.3 Style & Naming

- `snake_case` for functions, variables, modules; `PascalCase` for classes; `UPPER_SNAKE_CASE` for
  constants.
- Module names match their `PLAN.md` §6 layout exactly (e.g. `job_scout.py`, `vector_store.py`).
- Line length 100. Let `ruff format` decide formatting — never hand-format against it.
- Imports: standard lib → third-party → first-party, each group sorted (ruff/isort enforces this).

### 2.4 Docstrings

- Google-style docstrings on every public module, class, and function.
- A docstring states **what** and **why**, not a restatement of the signature. Document raised
  exceptions and non-obvious side effects.

```python
async def search_similar(self, vector: list[float], top_k: int = 10) -> list[JobMatch]:
    """Return the top_k jobs most semantically similar to a query vector.

    Args:
        vector: The query embedding, produced by the nomic-embed-text model.
        top_k: Maximum number of matches to return.

    Returns:
        Job matches ordered by descending cosine similarity.

    Raises:
        VectorStoreError: If the Qdrant collection is unavailable.
    """
```

### 2.5 Async

- The backend is async end-to-end. Use `async`/`await` consistently.
- Never call blocking I/O inside an async path. Use `sqlalchemy[asyncio]`, `httpx`, `aioimaplib`,
  `aiosmtplib`. Offload unavoidable blocking work to a thread executor.
- One `AsyncSession` per unit of work; do not share sessions across tasks.

### 2.6 Error Handling

- Define and raise custom exceptions from `app/core/exceptions.py`; map them to HTTP responses at the
  API boundary.
- **No bare `except:`** and no `except Exception` that silently swallows. Catch the narrowest type,
  log with context, and re-raise or translate.
- External calls (scrapers, LLM, SMTP/IMAP, HTTP) use **retry with exponential backoff** and respect
  rate limits.

### 2.7 Logging

- Use **Loguru** exclusively. No `print()` in committed code.
- One logger per agent/service: `self.logger = get_logger(self.__class__.__name__)`.
- Structured, contextual messages. **Never log secrets, tokens, passwords, or full email bodies.**

### 2.8 Configuration & Secrets

- All configuration comes from environment variables via `pydantic-settings` (`app/core/config.py`).
  The full variable set is in [`PLAN.md` §12.2](../PLAN.md) and `backend/.env.example`.
- **No hardcoded secrets, URLs, or credentials.** No real secret is ever committed; only
  `.env.example` with placeholder values.

### 2.9 Layering Rules (Clean Architecture)

| Module | Contains | Never contains |
|--------|----------|----------------|
| `app/models/` | SQLAlchemy ORM models, `__tablename__`, `__repr__`, timestamps. | Business logic, HTTP concerns. |
| `app/schemas/` | Pydantic request/response models. | ORM or DB access. |
| `app/agents/` | LangGraph decision logic, prompts, tool orchestration. | Direct HTTP/queue mechanics; ORM engine setup. |
| `app/services/` | External-integration adapters (LLM, Qdrant, MinIO, browser, email, PDF). | **Business logic.** |
| `app/api/` | FastAPI routers, validation, auth, serialization. | Agent logic, scraping. |
| `app/tasks/` | Thin Celery wrappers that call agents/services. | Business branching. |
| `app/core/` | config, database, celery, security, exceptions, middleware, websocket. | Imports from agents/api/tasks. |

## 3. Database & Migrations

- Schema changes go through **Alembic migrations** only — never edit the DB by hand or auto-create
  tables in production paths.
- Models mirror [`PLAN.md` §7](../PLAN.md): UUID primary keys, `created_at`/`updated_at` with
  `server_default`, and all indexes from §7.2.
- One migration per logical schema change, with a descriptive message.

## 4. Testing

- Framework: `pytest` + `pytest-asyncio`. Coverage via `pytest-cov`.
- **Coverage targets:** ≥ 70% by end of Phase 1, ≥ 90% by Phase 6 (see [`PLAN.md` §15](../PLAN.md)).
- Structure mirrors `backend/tests/{unit,integration,e2e}` per [`PLAN.md` §6](../PLAN.md).
- **Arrange–Act–Assert.** One behavior per test; descriptive names (`test_<unit>_<condition>_<result>`).
- **Mock external systems** (Crawl4AI, Ollama, SMTP/IMAP, Qdrant, MinIO) in unit tests; use the test
  DB and real wiring in integration tests; Playwright against a local mock page for E2E.
- Every new agent, service, and endpoint ships with tests **in the same change** (see
  [`development_workflow.md`](development_workflow.md), step 4).

## 5. Frontend (React / TypeScript)

- **TypeScript strict** mode; no `any` (use `unknown` + narrowing when a type is genuinely dynamic).
- Function components + hooks only. No class components.
- **Server state** via TanStack Query; **client/UI state** via Zustand. Do not duplicate server state
  into Zustand.
- Styling via **Tailwind + shadcn/ui** only — no bespoke CSS files unless unavoidable, and never
  inline style hacks.
- File/component naming: `PascalCase` components (`KanbanBoard.tsx`), `camelCase` hooks
  (`useApplications.ts`), matching the [`PLAN.md` §6](../PLAN.md) layout.
- API access through the shared Axios instance (`lib/api.ts`) with interceptors; WebSocket through
  `lib/ws.ts` / `useWebSocket.ts`.
- Components are responsive and mobile-first; provide empty and loading states.

## 6. Security

- Validate and sanitize all external input at the boundary (Pydantic on backend, schema validation on
  forms).
- Passwords hashed with `passlib[bcrypt]`; JWTs via `python-jose`.
- Respect per-domain rate limits and configurable daily caps (`MAX_APPLIES_PER_DAY`,
  `MAX_COLD_EMAILS_PER_DAY`).
- Cold/outreach emails must include an unsubscribe footer and honor volume caps (CAN-SPAM/GDPR — see
  [`PLAN.md` §14](../PLAN.md)).
- Never weaken the HITL approval gates to "make a test pass."

## 7. Git & Commits

Conventional Commits are mandatory. The full specification, branch naming, and the per-task workflow
are in [`development_workflow.md`](development_workflow.md).

## 8. Documentation

- Public APIs are self-documenting via FastAPI/OpenAPI; keep route docstrings and Pydantic field
  descriptions accurate.
- Every significant decision → an ADR ([`decision_log.md`](decision_log.md)).
- Every completed task → a [`dev_journal.md`](../dev_journal.md) entry.
- User-facing docs live under `docs/` and are published via MkDocs (Phase 6).
