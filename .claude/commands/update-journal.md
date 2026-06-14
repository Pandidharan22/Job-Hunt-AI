# UPDATE-JOURNAL Command

> Used to append a complete, structured entry to [`dev_journal.md`](../../dev_journal.md) after a task
> is implemented and committed. This is **Step 6** of the eight-step workflow, with the commit-hash
> field finalized in **Step 8**. The journal is the project's long-term engineering memory.

---

## When to Use

Once a task's code, tests, and docs are done — right before (and after) committing:

```
You: "Code and docs are ready. I'll use /update-journal to record the entry before committing."
```

---

## The 13 Required Fields

Every entry **must** contain all of these, in order. A missing field means the entry is incomplete.

| # | Field | What to write |
|---|-------|---------------|
| 1 | **Timestamp** | ISO 8601 UTC (`YYYY-MM-DDTHH:MMZ`); local time optional in parentheses. |
| 2 | **Sprint** | Sprint ID + name (e.g. `S1 — Project Bootstrap`). |
| 3 | **Phase** | Phase from the roadmap (e.g. `Phase 1 — Core Foundation`). |
| 4 | **Task Name** | Specific name of the completed task (e.g. `S1-T05 — Async DB engine`). |
| 5 | **Objective** | What the task set out to achieve, and why. |
| 6 | **Files Modified** | Every file created/changed/deleted (bullet list). |
| 7 | **Technical Decisions** | The concrete technical choices made. |
| 8 | **Reasoning Behind Decisions** | Why those choices over the alternatives. |
| 9 | **Problems Encountered** | Obstacles, bugs, surprises (or "None"). |
| 10 | **Solutions Applied** | How each problem was resolved (or "N/A"). |
| 11 | **Validation Performed** | Exact commands run + their outcomes. |
| 12 | **Commit Hash** | The Git hash of the commit containing the work. |
| 13 | **Next Recommended Task** | The logical next piece of work. |

Use [`templates/journal_entry.md`](../templates/journal_entry.md) as the starting point.

---

## Workflow

### 1. Append, Never Overwrite

- Add the new entry at the **bottom** of the `## Entries` section (chronological, newest last).
- **Never edit a past entry** except to backfill its `Commit Hash`.

### 2. Fill Every Field Honestly

- **Validation Performed** must reflect what actually ran. If a check was skipped or a test failed,
  say so explicitly — the journal records reality, not an idealized version.
- **Problems/Solutions** are valuable history; don't write "None" if something non-trivial happened.

### 3. Handle the Commit Hash (Backfill Convention)

A commit cannot contain its own hash, so:

1. Write the entry with `Commit Hash: _pending_`.
2. Create the task commit (workflow Step 7).
3. Run `git rev-parse HEAD`, replace `_pending_` with the real hash.
4. Record it via a small follow-up commit, e.g.
   `docs: backfill commit hash for S1-TNN journal entry`.

The recorded hash always points to the commit containing the **substantive work**.

---

## Example Entry (Shape)

```markdown
### 2026-06-20T14:05Z — S1-T05 — Async DB engine + session factory

- **Timestamp:** 2026-06-20T14:05Z (19:35 IST)
- **Sprint:** S1 — Project Bootstrap
- **Phase:** Phase 1 — Core Foundation & Job Discovery
- **Task Name:** S1-T05 — Async DB engine + session factory
- **Objective:** Provide the canonical async SQLAlchemy engine and session dependency...
- **Files Modified:**
  - `backend/app/core/database.py`
  - `backend/tests/integration/test_database.py`
- **Technical Decisions:** Used `async_sessionmaker` with `expire_on_commit=False`...
- **Reasoning Behind Decisions:** ...
- **Problems Encountered:** ...
- **Solutions Applied:** ...
- **Validation Performed:** `uv run ruff check .` ✓, `uv run mypy app/` ✓, `uv run pytest` ✓ (N tests)...
- **Commit Hash:** _pending_
- **Next Recommended Task:** S1-T06 — Security utilities (hashing + JWT).
```

---

## Refusals / Red Flags

- [ ] **Do NOT skip the entry** "to save time." It is a mandatory step.
- [ ] **Do NOT leave fields blank.** Use "None"/"N/A" where genuinely empty, never omit a field.
- [ ] **Do NOT leave `Commit Hash: _pending_`** permanently — always backfill (Gate 6 of `/close-task`).
- [ ] **Do NOT edit historical entries** except for the hash backfill.

## See Also

- [`../../dev_journal.md`](../../dev_journal.md) — The journal itself + required-fields spec.
- [`templates/journal_entry.md`](../templates/journal_entry.md) — Copyable entry template.
- [`close-task.md`](close-task.md) — Where this command runs (Gate 4 + Gate 6).
- [`../../docs/development_workflow.md`](../../docs/development_workflow.md) — Steps 6 and 8.
