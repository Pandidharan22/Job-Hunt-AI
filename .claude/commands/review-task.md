# REVIEW-TASK Command

> Used to review code against architecture, standards, tests, docs, journal, and Git quality before
> marking a task complete.

---

## When to Use

Before marking a task done (after Step 7, before Step 8 of the workflow):

```
You: "Code is written. I'll use /review-task to verify all checks pass."
```

This is the penultimate check — once `/review-task` passes, `/close-task` finishes the task.

---

## Workflow

### 1. Run All Validation & Tests (Automated)

Execute these commands and report results:

**Backend:**

```bash
cd backend

# Format check (no automatic fix)
uv run ruff format --check .

# Lint
uv run ruff check .

# Type check (mypy strict)
uv run mypy app/

# Tests + coverage
uv run pytest tests/ --cov=app --cov-report=term-missing -v
```

**Frontend** (when applicable):

```bash
cd apps/web
npm run lint
npm run typecheck
```

**Stop if any fails.** Don't proceed to the manual review checklist.

### 2. Architecture & Standards Review (Manual Checklist)

Use [`context/coding_checklist.md`](../context/coding_checklist.md) as the master checklist.

Check all of these:

#### Code Quality
- [ ] No placeholder code (`pass`, `...`, `return None`, etc.)
- [ ] No `TODO` / `FIXME` comments
- [ ] No hardcoded secrets, URLs, or credentials
- [ ] All functions have docstrings (Google-style)
- [ ] No `print()` in committed code

#### Architecture & Layering (Clean Architecture)
- [ ] `app/models/` contains ORM only; no business logic
- [ ] `app/schemas/` is Pydantic only; no DB access
- [ ] `app/agents/` makes decisions; doesn't know HTTP/Celery internals
- [ ] `app/services/` integrates external systems; **zero business logic**
- [ ] `app/api/` validates and routes; no agent logic
- [ ] `app/tasks/` are thin wrappers; call agents/services only
- [ ] Dependencies point inward (no upward references)

#### Async Correctness
- [ ] No blocking I/O on async paths (`httpx` not `requests`, etc.)
- [ ] Sessions not shared across tasks/requests
- [ ] All DB/HTTP/email calls are async

#### Security
- [ ] No secrets logged
- [ ] Input validated at boundaries (Pydantic)
- [ ] SQL parameterized (SQLAlchemy ORM)
- [ ] Passwords hashed (`passlib[bcrypt]`)
- [ ] JWT tokens have expiry
- [ ] HITL gates present if outbound action

#### Testing
- [ ] Tests written (unit + integration)
- [ ] All tests green
- [ ] Coverage ≥ 70% Phase 1 (target from task)
- [ ] No skipped/xfailed tests
- [ ] External systems mocked in unit tests
- [ ] Integration tests use real test DB

#### Documentation
- [ ] `sprint_tracking.md` updated (deliverable checked off)
- [ ] `.env.example` updated (if config changed)
- [ ] API docs / OpenAPI descriptions updated
- [ ] Architecture doc updated (if design changed)
- [ ] Coding standards doc updated (if new conventions)
- [ ] ADR written (if major decision) — see [`create-adr.md`](create-adr.md)

#### Git & Commits
- [ ] Commit message is Conventional Commit
  - Format: `<type>(<scope>): <summary>\n\n<body>\n\nCo-Authored-By: ...`
- [ ] One logical change per commit
- [ ] No large binaries or secrets in history

### 3. Security Review (If Task Touches Auth/Secrets)

If the task involves authentication, authorization, secrets, environment variables, or outbound
actions, also use [`security-review.md`](security-review.md).

### 4. Generate Review Report

Output a structured report using the template: [`templates/review_report.md`](../templates/review_report.md).

The report must include:

```markdown
## REVIEW-TASK Report: [Task ID]

**Date:** YYYY-MM-DD
**Reviewer:** Claude Code
**Task:** [Task name]

### Validation Results

**Format check:** ✓ PASS / ✗ FAIL
**Lint:** ✓ PASS / ✗ FAIL
**Type check:** ✓ PASS / ✗ FAIL
**Tests:** ✓ PASS (X tests, X% coverage) / ✗ FAIL

### Checklist Results

**Code Quality:** [N/M boxes checked]
- [x] No placeholder code
- [x] No TODOs
- ...

**Architecture:** [N/M boxes checked]
- [x] Clean Architecture respected
- ...

**Testing:** [N/M boxes checked]
...

**Security:** [N/M boxes checked] (or N/A if not applicable)
...

### Issues Found

1. [Issue]: [Description] → [Fix]
2. [Issue]: [Description] → [Fix]

(Or "None — all checks pass.")

### Final Verdict

**PASS:** Ready for /close-task.
**FAIL:** Fix issues and re-run /review-task.

### Next Step

→ `/close-task` to finish the task (Steps 3–8).
```

### 5. Report Result to User

- **If PASS:** "All checks pass. Ready to proceed with `/close-task`."
- **If FAIL:** "Issues found. Fix these before re-running `/review-task`: [list]."

---

## Common Issues Found & How to Fix

| Issue | Fix |
|-------|-----|
| `ruff format --check` fails | Run `uv run ruff format .` to auto-fix, then re-check. |
| `ruff check` fails | Review the violations; fix them manually. Re-run. |
| `mypy app/` has errors | Fix type annotations. Add `# type: ignore` only with reason comments. |
| Tests fail | Fix the code or the test. Don't skip tests. |
| Coverage < 70% (Phase 1) | Add tests for uncovered lines. Re-run coverage. |
| `TODO` or `FIXME` found | Remove or complete the work. Don't commit `TODO`s. |
| Hardcoded secret | Move to `.env.example` or use `pydantic-settings`. Re-check. |
| Docstring missing | Add Google-style docstring to every public function/class. |
| `print()` found | Replace with Loguru. Remove `print` calls. |

---

## What NOT to Do

- [ ] **Do NOT skip validation steps.** If `mypy` fails, don't comment out the error.
- [ ] **Do NOT weaken tests.** If a test fails, fix the code; don't skip the test.
- [ ] **Do NOT suppress errors without reason.** If you use `# type: ignore`, explain why.
- [ ] **Do NOT merge without passing review.** The checklist is non-negotiable.

---

## Pre-Conditions for Use

1. **Code is fully written.** (Steps 1–2 complete.)
2. **All validation commands can run.** (Backend tools installed, venv active.)
3. **Tests are written.** (Tests weren't skipped.)
4. **Docs are updated.** (Docstrings, sprint board, ADRs if needed.)

## See Also

- [`security-review.md`](security-review.md) — Deep security review (if task touches auth/secrets).
- [`context/coding_checklist.md`](../context/coding_checklist.md) — Checklist reference.
- [`close-task.md`](close-task.md) — Finalize the task (after review passes).
- [`templates/review_report.md`](../templates/review_report.md) — Report template.
