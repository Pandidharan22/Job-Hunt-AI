# START-TASK Command

> Used to begin work on a task. This command reads the task specification, identifies dependencies,
> checks blockers, and produces an implementation plan.

---

## When to Use

At the very start of an implementation task (Step 1 of the eight-step workflow):

```
User: "Implement T05 (Async DB engine + session factory)"
You: "I'll use /start-task to create an implementation plan."
```

---

## Workflow

### 1. Read Context (Silent)

Before responding to the user, read all of these in order:

1. **Task source:** [`docs/sprint_s1_execution_plan.md`](../../docs/sprint_s1_execution_plan.md) (find the task card).
2. **Architecture:** [`context/architecture_summary.md`](../context/architecture_summary.md) (system overview).
3. **Project rules:** [`context/project_rules.md`](../context/project_rules.md) (constraints + pillars).
4. **Coding standards:** [`docs/coding_standards.md`](../../docs/coding_standards.md) (language + testing rules).
5. **Current sprint:** [`context/sprint_status.md`](../context/sprint_status.md) (dependencies, blockers, checkpoints).
6. **Related tasks:** If the task has dependencies (marked "Deps" in the sprint table), skim their task cards.

### 2. Validate Task Readiness

Check all of these before committing:

- [ ] **Task dependencies are complete.** If the task lists dependencies (e.g., "T05 needs T02, T03"), verify they are ☑ done in [`docs/sprint_tracking.md`](../../docs/sprint_tracking.md).
- [ ] **Task is in scope for the current sprint.** (Don't start S2 work during S1.)
- [ ] **No external blockers.** (If a task needs Python 3.12 or Docker, confirm they're available.)
- [ ] **Task objective is clear.** (Can you state in one sentence what the task builds and why?)

**If any check fails:** Report it and refuse the task. Don't start work.

Example refusal:
> "T05 depends on T02 and T03. T02 is done but T03 (Core config) is not yet complete. You must finish T03
> before starting T05. Ready to help with T03?"

### 3. Create Implementation Plan

Output a structured plan using the template: [`templates/task_plan.md`](../templates/task_plan.md).

The plan must include:

#### Task Summary
- **ID & name** (e.g., S1-T05)
- **One-sentence objective** (e.g., "Create async SQLAlchemy engine and session factory for the project.")
- **Why it matters** (reference to architecture/phase goal)

#### Requirements (From Task Card)
- **What must be implemented.** (Verbatim from the task card's "Files Expected" section.)
- **Validation criteria.** (Verbatim from the task card.)

#### Dependency Check
- **Upstream dependencies** (tasks that must be done first).
- **Confirmed?** (Yes/No — did you verify they're complete?)

#### Critical Path & Parallelization
- **Is this task on the critical path?** (Yes/No — affects downstream tasks?)
- **Can other tasks run in parallel?** (List them.)

#### Architecture Alignment
- **Which layers does this touch?** (e.g., `app/core/`, `tests/`)
- **Clean Architecture respected?** (Yes/No — explain why.)
- **Any HITL gates needed?** (Yes/No.)

#### Technical Approach
- **Proposed implementation strategy.** (2–3 paragraphs explaining *how*.)
- **Key decisions.**  (Why this approach over alternatives?)
- **Risks or unknowns.** (Edge cases, version compat, etc.)

#### Deliverables Checklist
- [ ] `file/path/one.py` — description
- [ ] `file/path/two.py` — description
- [ ] `tests/unit/test_one.py` — description
- ... (all files from "Files Expected")

#### Validation Plan
- **Format/lint:** `uv run ruff format --check . && uv run ruff check .`
- **Type check:** `uv run mypy app/`
- **Tests:** `uv run pytest tests/ --cov=app --cov-report=term-missing` (target ≥ 70%)
- **Other:** (Any custom validation steps?)

#### Time Estimate
- **Complexity:** 1–10 (from task card).
- **Risk:** 1–10 (from task card).
- **Estimated duration:** (e.g., "2–3 hours for an experienced Python developer.")

#### Definition of Done
- [ ] Code complete (no placeholders, `TODO`s, stubs).
- [ ] Validation passes (ruff + mypy clean).
- [ ] Tests written & green (coverage ≥ target).
- [ ] Docstrings present (Google-style).
- [ ] Docs updated (sprint board, ADRs if needed).
- [ ] Journal entry appended (`dev_journal.md`, commit hash = `_pending_`).
- [ ] Conventional Commit created.
- [ ] Commit verified on `main`.

#### Next Steps
1. **Confirm plan with user.** "Does this plan look good? Any questions before I start?"
2. **Execute the implementation.** Follow the eight-step workflow in [`docs/development_workflow.md`](../../docs/development_workflow.md).
3. **After implementation:** Use `/review-task` to verify pre-commit checks.

---

## Example Output

**START-TASK: S1-T05 — Async DB Engine + Session Factory**

**Objective:** Create the async SQLAlchemy engine and `async_sessionmaker` factory, exposing a
`get_session()` dependency for use throughout the backend.

**Why it matters:** The database engine is foundational — every ORM model, migration, and integration
test depends on it. This task unblocks T11 (migrations), T13 (test harness), and T12 (FastAPI app).

---

**Dependencies**
- Upstream: T02 (project init) ☑, T03 (config) ☐
- **Blocker:** T03 is not complete. T05 needs the Settings object to get DATABASE_URL. 
  **Status:** Cannot start T05 until T03 is done.

---

(Continuing if dependencies were ready...)

---

## Pre-Conditions for Use

1. **You have read all CCOS context files.** (Listed in "Read Context" above.)
2. **You understand the eight-step workflow.** (Reference [`docs/development_workflow.md`](../../docs/development_workflow.md).)
3. **You have not already started coding.** (This command is Step 1; you're about to move to Step 2.)

## Command Refusals

Refuse the task and report it if:

- [ ] **Dependencies are incomplete.** (Example: T05 needs T03 but T03 isn't done.)
- [ ] **Task is out of scope.** (Example: Starting S2 work during S1.)
- [ ] **Unclear requirements.** (Example: Task card is vague; ask for clarification before proceeding.)
- [ ] **External blockers.** (Example: Task requires a tool that isn't installed; suggest resolution.)

**Never guess.** Report blockers clearly and ask the user to resolve them.

## See Also

- [`close-task.md`](close-task.md) — Called at the end of the task (Steps 3–8).
- [`review-task.md`](review-task.md) — Called before marking the task done.
- [`templates/task_plan.md`](../templates/task_plan.md) — Plan template.
- [`../context/sprint_status.md`](../context/sprint_status.md) — Current sprint + dependencies.
