# Development Workflow — JobHunt AI

> This is the **operating procedure** for all work on JobHunt AI. It applies to every contributor —
> human or AI agent — and to every change, from a one-line fix to a whole agent. Read
> [`coding_standards.md`](coding_standards.md) for *how to write the code* and this document for
> *the process around every change*.

---

## 1. The Golden Rule

> **Every implementation task follows the eight steps below, in order. No step is ever skipped.**

If a step cannot be completed (e.g. tests fail), you **stop and report it honestly** — you do not
fake success, comment out the failing check, or move on. A red build is reported as red.

## 2. The Eight-Step Task Workflow

```
1. Analyze requirements   →  2. Implement        →  3. Run validation
        ↓                                                    ↓
8. Verify commit success  ←  7. Create git commit ← 6. Update dev_journal.md
        ↑                                                    ↓
        └──────────────  5. Update documentation  ←  4. Run tests
```

### Step 1 — Analyze Requirements

- Re-read the relevant section(s) of [`PLAN.md`](../PLAN.md) and [`project_context.md`](project_context.md).
- Identify scope, affected layers, inputs/outputs, and the Definition of Done for the task.
- Confirm which sprint/phase the task belongs to in [`sprint_tracking.md`](sprint_tracking.md).
- Decide whether the work warrants an ADR (see §6).
- **Exit criteria:** you can state, in one or two sentences, exactly what you will build and how you
  will know it works.

### Step 2 — Implement

- Write real, complete code that satisfies the [Quality Rules](coding_standards.md#1-quality-rules-absolute).
- Stay within the layering rules in [`architecture.md`](architecture.md).
- Keep the change focused — one logical concern per branch/commit.
- **Exit criteria:** the feature is fully implemented with no placeholders, `TODO`s, or stubs.

### Step 3 — Run Validation (lint + types)

Backend:

```bash
cd backend
uv run ruff format --check .
uv run ruff check .
uv run mypy app/
```

Frontend:

```bash
cd apps/web
npm run lint
npm run typecheck
```

- **Exit criteria:** linter, formatter, and type checker all pass with zero errors. Fix issues; do
  not suppress them without a documented reason.

### Step 4 — Run Tests

```bash
cd backend
uv run pytest tests/ --cov=app --cov-report=term-missing
```

- New code ships with tests in the **same** change (unit + integration as appropriate; E2E for
  apply flows).
- Coverage must not regress and must meet the phase target (≥ 70% Phase 1 → ≥ 90% Phase 6).
- **Exit criteria:** the full suite is green and coverage meets target. If a test fails, fix the code
  or the test — never delete or skip a test to go green.

### Step 5 — Update Documentation

- Update any doc the change affects: [`sprint_tracking.md`](sprint_tracking.md) (tick deliverables),
  [`architecture.md`](architecture.md), [`coding_standards.md`](coding_standards.md), API docs,
  `.env.example`, and the MkDocs `docs/` pages (Phase 6+).
- Keep docstrings and Pydantic field descriptions current.
- **Exit criteria:** no documentation now contradicts the code.

### Step 6 — Update `dev_journal.md`

- Append a new entry following the mandatory template in
  [`../dev_journal.md`](../dev_journal.md). Every field is required.
- Leave the **Commit Hash** field as `_pending_` for now (it is filled in Step 8).
- **Exit criteria:** a complete journal entry exists for this task.

### Step 7 — Create Git Commit

- Stage the change and commit using **Conventional Commits** (§4).
- Commit message body explains the *why*, references the sprint/ADR where relevant, and ends with the
  required co-author trailer.
- **Exit criteria:** `git commit` succeeds and a hook-clean commit is created on a feature branch.

### Step 8 — Verify Commit Success

```bash
git log -1 --stat
git status
```

- Confirm the commit exists, contains the intended files, and the working tree is clean.
- Copy the new commit hash into the `Commit Hash` field of the journal entry from Step 6, then make a
  small follow-up `docs:` commit to record it (the hash of a commit cannot be known until after it is
  created — see the [dev journal conventions](../dev_journal.md)).
- **Exit criteria:** the commit is confirmed present, the journal records its hash, and the working
  tree is clean.

## 3. Branching Strategy

- `main` is always releasable and green. Never commit experimental or broken work to `main`.
- Branch per task from up-to-date `main`. Branch names use the Conventional Commit type as a prefix:

| Prefix | Use for | Example |
|--------|---------|---------|
| `feat/` | New feature | `feat/job-scout-agent` |
| `fix/` | Bug fix | `fix/imap-reconnect` |
| `refactor/` | Restructuring, no behavior change | `refactor/agent-base-contract` |
| `docs/` | Documentation only | `docs/architecture-update` |
| `test/` | Tests only | `test/resume-tailor-coverage` |
| `chore/` | Tooling, deps, CI | `chore/bump-ruff` |

## 4. Conventional Commits (Mandatory)

Format:

```
<type>(<optional scope>): <short imperative summary>

<optional body — the why, wrapped at ~72 cols>

<optional footer — BREAKING CHANGE:, refs, co-author trailer>
```

**Allowed types**

| Type | Meaning |
|------|---------|
| `feat` | A new feature. |
| `fix` | A bug fix. |
| `refactor` | A code change that neither fixes a bug nor adds a feature. |
| `docs` | Documentation only. |
| `test` | Adding or correcting tests. |
| `chore` | Build process, tooling, dependencies, CI. |

**Rules**

- Summary in the **imperative mood** ("add", not "added"/"adds"), ≤ 72 characters, no trailing period.
- Scope is optional but encouraged; use the area touched (`agents`, `api`, `web`, `infra`, `docs`).
- Breaking changes: add a `BREAKING CHANGE:` footer describing the impact and migration.
- Every commit message ends with the co-author trailer:

  ```
  Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>
  ```

**Examples**

```
feat(agents): add cold email agent with company research pipeline
fix(api): reconnect IMAP watcher after idle timeout
refactor(agents): extract shared LangGraph node helpers into base_agent
docs(workflow): document the eight-step task workflow
test(agents): cover resume tailor ATS-score iteration loop
chore(ci): add mypy strict check to the pull-request pipeline
```

## 5. Validation Command Reference

| Goal | Command |
|------|---------|
| Format check (backend) | `uv run ruff format --check .` |
| Lint (backend) | `uv run ruff check .` |
| Types (backend) | `uv run mypy app/` |
| Tests + coverage (backend) | `uv run pytest tests/ --cov=app --cov-report=term-missing` |
| Single test, fail fast | `uv run pytest tests/path::test_name -x -q` |
| Lint (frontend) | `npm run lint` |
| Types (frontend) | `npm run typecheck` |

> These commands assume the S1 scaffolding (`pyproject.toml`, `package.json`) is in place. Until then
> they are the **target** commands the bootstrap sprint must wire up.

## 6. When to Write an ADR

Create an [Architecture Decision Record](decision_log.md) when a change:

- Introduces or replaces a framework, library, datastore, or external service.
- Establishes a cross-cutting pattern (auth, error handling, agent contract, eventing).
- Makes a trade-off future contributors would otherwise have to reverse-engineer.
- Deviates from `PLAN.md` (the ADR must explain why and `PLAN.md` should be updated).

Routine feature work that follows existing patterns does **not** need an ADR — the journal entry is
sufficient.

## 7. Definition of Done (Per Task)

A task is done only when **all** of these hold:

- [ ] Requirements satisfied and within scope (Step 1).
- [ ] Code complete, no placeholders/`TODO`s/stubs (Step 2).
- [ ] `ruff` + `mypy` (and frontend lint/typecheck) pass clean (Step 3).
- [ ] Tests written and green; coverage meets the phase target (Step 4).
- [ ] All affected documentation updated (Step 5).
- [ ] `dev_journal.md` entry appended (Step 6).
- [ ] Conventional Commit created (Step 7).
- [ ] Commit verified and its hash recorded in the journal (Step 8).

## 8. Handling Failures Honestly

- If validation or tests fail and cannot be fixed within the task, **report the failure with the
  actual output**. Do not commit broken code to `main`, do not skip tests, do not weaken assertions
  or HITL gates to force a pass.
- If a step is intentionally skipped (e.g. no doc change was needed), say so explicitly in the
  journal entry rather than silently omitting it.

## 9. See Also

- [`coding_standards.md`](coding_standards.md) — code quality rules.
- [`sprint_tracking.md`](sprint_tracking.md) — what to work on now.
- [`decision_log.md`](decision_log.md) — recording decisions.
- [`../dev_journal.md`](../dev_journal.md) — the engineering history and entry template.
