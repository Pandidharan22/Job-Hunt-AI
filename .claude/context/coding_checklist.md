# Coding Checklist — JobHunt AI

> A comprehensive checklist for code review and Definition of Done verification. Use this before
> marking a task complete.

---

## Pre-Review: Self-Check (Developer)

Before requesting review, developer must verify:

### Code Quality

- [ ] **No placeholder code.** Every function/class is real and complete.
  - Grep for: `pass`, `return None`, `...`, `NotImplementedError`, etc.
  - Command: `grep -r "pass\|NotImplementedError" backend/app/` — must be empty.
- [ ] **No `TODO` / `FIXME` comments.** Tasks tracked in sprint board, not code.
  - Command: `grep -r "TODO\|FIXME" backend/app/` — must be empty.
- [ ] **No hardcoded secrets, URLs, or credentials.**
  - Grep for: `api_key`, `password`, `secret`, real email addresses, real API keys.
  - Command: `grep -r "api_key\|password\|secret" backend/app/` — only in config defaults.
- [ ] **All functions have docstrings.** Google-style, stating *what* and *why*.
  - Check: Every public function/class/module has a docstring.
- [ ] **No `print()`** in committed code. Loguru only.
  - Command: `grep -r "print(" backend/app/` — must be empty.

### Validation & Types

- [ ] **Format check passes.** `ruff format --check .`
  - Command: `cd backend && uv run ruff format --check .`
- [ ] **Lint passes.** `ruff check .`
  - Command: `cd backend && uv run ruff check .`
- [ ] **Type check clean.** `mypy app/` with strict mode.
  - Command: `cd backend && uv run mypy app/` — zero errors.
- [ ] **No `# type: ignore` without reason.** Each use must have a trailing comment.

### Tests

- [ ] **Tests written for all new code.** Unit + integration as appropriate.
  - In same commit/PR as code.
- [ ] **All tests green.** `pytest tests/ -x` (stop on first fail).
  - Command: `cd backend && uv run pytest tests/ -x`
- [ ] **Coverage meets phase target.** ≥ 70% Phase 1, ≥ 90% Phase 6.
  - Command: `cd backend && uv run pytest tests/ --cov=app --cov-report=term-missing`
  - Check: Coverage % shown in output.
- [ ] **External systems mocked in unit tests.** (Ollama, SMTP/IMAP, Qdrant, MinIO, Crawl4AI.)
- [ ] **Integration tests use test DB** and real wiring (except external calls).
- [ ] **E2E tests for apply flows** use a local mock ATS page.
- [ ] **No skipped tests.** If a test fails, fix it; don't skip it.

### Documentation

- [ ] **All affected docs updated.** Check: `sprint_tracking.md`, `architecture.md`, `coding_standards.md`, API docs, `.env.example`.
- [ ] **Docstrings are current.** Match the actual code behavior.
- [ ] **Pydantic field descriptions are clear.** (For API request/response schemas.)
- [ ] **ADR written if decision warrants it.** (See [`../commands/create-adr.md`](../commands/create-adr.md).)

### Architecture & Layering

- [ ] **Clean Architecture respected.** Inner rings don't import outer rings.
  - Check: `app/models/` has no business logic.
  - Check: `app/schemas/` has no ORM or DB access.
  - Check: `app/agents/` doesn't know about HTTP or Celery internals.
  - Check: `app/services/` contains zero business logic; only integrations.
  - Check: `app/api/` routes don't contain agent logic.
  - Check: `app/tasks/` are thin wrappers; call agents/services only.
- [ ] **Async end-to-end.** No blocking I/O on async paths.
  - Check: Uses `httpx` not `requests`, `sqlalchemy[asyncio]` not blocking DB, `aioimaplib` not `smtplib`, etc.
- [ ] **Async sessions not shared** across tasks/requests. One per unit of work.
- [ ] **Custom exceptions used.** Not bare `except:` or `except Exception`.
  - Check: Exceptions come from `app/core/exceptions.py` and are mapped to HTTP at the API boundary.
- [ ] **Agent logic in agents, integrations in services.** No hybrid modules.
- [ ] **HITL gates present if outbound action.** Apply, email, DM, etc. must have approval.

### Secrets & Security

- [ ] **No real credentials in commits.** Only `.env.example` with placeholders.
- [ ] **Secrets read from environment.** Via `pydantic-settings`, no hardcoding.
- [ ] **Never log secrets, tokens, passwords, or full email bodies.**
  - Check: Loguru calls don't include sensitive data.
- [ ] **Input validation at boundaries.** Pydantic on routes, DB queries parameterized.
- [ ] **No SQL injection.** SQLAlchemy ORM used; no string concatenation in queries.
- [ ] **No prompt injection.** If LLM prompt templates are used, escaped/parameterized.
- [ ] **Password hashing.** `passlib[bcrypt]`, not plain text.
- [ ] **JWT tokens** have expiry, use `HS256` or similar.

### Version Control

- [ ] **Commit is a single logical change.** One concern per commit.
- [ ] **Commit message is Conventional Commit.** Type(scope): summary + body + trailer.
  - Format: `<type>(<scope>): <summary>\n\n<body>\n\nCo-Authored-By: ...`
- [ ] **No large binary files or secrets in history.** Use `.gitignore`.
- [ ] **Branch name follows convention.** `<type>/s1-tNN-<slug>`.

---

## Reviewer Checklist

When a PR lands for review, use this checklist:

### Architecture & Design

- [ ] Code respects Clean Architecture. Dependencies point inward only.
- [ ] Agents make decisions; tasks dispatch; routes validate only.
- [ ] Services contain zero business logic; only integration adapters.
- [ ] HITL gates present if the task involves outbound actions.
- [ ] No resume fabrication (Resume Tailor: only rephrases, never adds).

### Code Quality

- [ ] No placeholder code. Everything is real and complete.
- [ ] No `TODO` / `FIXME`. Tasks tracked elsewhere.
- [ ] Docstrings are present and clear.
- [ ] No `print()`. Loguru only.
- [ ] No hardcoded secrets or credentials.

### Testing

- [ ] New code has tests (unit + integration).
- [ ] All tests green. No skipped or xfailed tests.
- [ ] Coverage meets phase target (≥ 70% Phase 1).
- [ ] External systems mocked in unit tests.
- [ ] Integration tests use real test DB.

### Validation & Types

- [ ] `ruff format --check` passes.
- [ ] `ruff check` passes.
- [ ] `mypy app/` is zero errors.
- [ ] No `# type: ignore` without reason.

### Async Correctness

- [ ] No blocking I/O on async paths.
- [ ] Sessions not shared across tasks.
- [ ] All DB/HTTP/email calls are async.

### Security

- [ ] No secrets logged.
- [ ] Input validated at boundaries (Pydantic).
- [ ] SQL parameterized (SQLAlchemy ORM).
- [ ] Passwords hashed (`passlib[bcrypt]`).
- [ ] Tokens have expiry (JWT).
- [ ] HITL gates on outbound actions.

### Git & Commits

- [ ] Commit message is Conventional Commit.
- [ ] One logical change per commit.
- [ ] No large binaries or secrets in history.

---

## Definition of Done (Task Complete)

A task is done only when **all** of these are true:

- [ ] **Requirements satisfied.** Feature works as specified in the task card.
- [ ] **Code complete.** No placeholders, `TODO`s, or stubs.
- [ ] **Validation passes.** `ruff format --check`, `ruff check`, `mypy app/` all zero errors.
- [ ] **Tests written & green.** Coverage ≥ phase target (70% Phase 1, 90% Phase 6). No skipped tests.
- [ ] **Docs updated.** All affected docs (sprint board, architecture, standards, `.env.example`, API docs, ADRs) current.
- [ ] **Journal entry appended.** `dev_journal.md` entry with all 13 required fields; **Commit Hash** left as `_pending_`.
- [ ] **Conventional Commit created.** Feature branch, proper message format, co-author trailer.
- [ ] **Commit verified.** Hash recorded in journal. Working tree clean. Commit on `main` (via squash-merge).

**Example flow:**
1. Code implemented, tested, docs updated. ← Steps 1–5
2. Journal entry written (Commit Hash = `_pending_`). ← Step 6
3. Commit created on feature branch. ← Step 7
4. Commit hash recorded in journal, journal change committed on same branch. ← Step 8
5. Feature branch merged (squash) to `main`. ← Merge
6. `main` green, working tree clean. ← Done

If any box is unchecked, the task is **not done**. Stop and fix before proceeding.

---

## Sprint Retrospective (At Sprint End)

When closing out a sprint, use this checklist:

- [ ] All tasks in the sprint are ☑ (done).
- [ ] All journal entries complete with commit hashes.
- [ ] All ADRs written and recorded in [`docs/decision_log.md`](../../docs/decision_log.md).
- [ ] Sprint board ([`docs/sprint_tracking.md`](../../docs/sprint_tracking.md)) updated (status, coverage, risks).
- [ ] No open TODOs or blockers in code.
- [ ] README status banner updated (if phase advanced).
- [ ] CI green on `main`.
- [ ] Retro entry written in journal (lessons learned, bottlenecks, successes).
- [ ] Retrospective commit created (`docs: close out sprint <ID> and record retrospective`).

---

## Common Failure Modes (Red Flags)

| Red Flag | What to do |
|----------|-----------|
| "I skipped validation to go faster." | **Stop.** Fix the issue. Don't weaken checks. |
| "Tests fail but I moved on." | **Stop.** Fix or rewrite the test. Don't skip it. |
| "I added a `TODO` to do this later." | **Stop.** Complete the work now or open an issue. Don't commit `TODO`s. |
| "I hardcoded a secret for testing." | **Stop.** Use `.env.example` and test secrets. Don't commit real secrets. |
| "The code has no docstring but it's obvious." | **Stop.** Add a Google-style docstring. |
| "I didn't write tests because the code is simple." | **Stop.** All code gets tests. Test coverage is non-negotiable. |
| "I skipped the journal entry to save time." | **Stop.** Journal entry is mandatory Step 6. Don't skip it. |
| "The architecture is a bit messy but it works." | **Stop.** Respect Clean Architecture. Refactor if needed. |

---

## Tools & Commands (Quick Reference)

### Validation

```bash
cd backend
uv run ruff format --check .     # Format check
uv run ruff check .              # Lint
uv run mypy app/                 # Type check
```

### Testing

```bash
cd backend
uv run pytest tests/ -x           # Run all, stop on fail
uv run pytest tests/ --cov=app --cov-report=term-missing
                                  # Run with coverage
uv run pytest tests/path -k test_name -x -v
                                  # Run specific test (verbose, stop on fail)
```

### Git

```bash
git status                        # Check working tree
git log -1 --stat                 # Show last commit
git diff --cached                 # Show staged changes
git commit -m "..."               # Create commit (use Conventional format)
git log --oneline -10             # Show recent commits
```

### Search (Never Commit)

```bash
grep -r "TODO\|FIXME" backend/app/
grep -r "pass$" backend/app/
grep -r "print(" backend/app/
grep -r "api_key\|password\|secret" backend/app/
```

---

## See Also

- [`project_rules.md`](project_rules.md) — Architecture constraints.
- [`../../docs/coding_standards.md`](../../docs/coding_standards.md) — Detailed standards.
- [`../../docs/development_workflow.md`](../../docs/development_workflow.md) — Eight-step workflow.
- [`../commands/review-task.md`](../commands/review-task.md) — REVIEW-TASK command.
- [`../commands/close-task.md`](../commands/close-task.md) — CLOSE-TASK command.
