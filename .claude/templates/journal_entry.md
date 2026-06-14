# Journal Entry Template

> Template for one [`dev_journal.md`](../../dev_journal.md) entry, used by
> [`/update-journal`](../commands/update-journal.md). Copy the block below, fill **all 13 fields**, and
> **append it at the bottom** of the `## Entries` section (chronological, newest last). Delete this
> header when producing the real entry.

---

```markdown
### YYYY-MM-DDTHH:MMZ — S1-TNN — <Task Name>

- **Timestamp:** YYYY-MM-DDTHH:MMZ (HH:MM <TZ>)
- **Sprint:** S? — <name>
- **Phase:** Phase ? — <name>
- **Task Name:** S1-TNN — <name>
- **Objective:** <what this task achieved and why>
- **Files Modified:**
  - `path/one`
  - `path/two`
- **Technical Decisions:** <the concrete choices made>
- **Reasoning Behind Decisions:** <why those choices over the alternatives>
- **Problems Encountered:** <obstacles, bugs, surprises — or "None">
- **Solutions Applied:** <how each problem was resolved — or "N/A">
- **Validation Performed:** <exact commands + outcomes, e.g. `uv run ruff check .` ✓, `uv run mypy app/` ✓, `uv run pytest` ✓ (N passed, X% cov)>
- **Commit Hash:** _pending_
- **Next Recommended Task:** S1-T(N+1) — <name>
```

---

## The 13 Required Fields (all mandatory)

1. Timestamp · 2. Sprint · 3. Phase · 4. Task Name · 5. Objective · 6. Files Modified ·
7. Technical Decisions · 8. Reasoning Behind Decisions · 9. Problems Encountered ·
10. Solutions Applied · 11. Validation Performed · 12. Commit Hash · 13. Next Recommended Task.

## Conventions

- **Timestamps** are UTC, ISO 8601 (`YYYY-MM-DDTHH:MMZ`); local time optional in parentheses.
- **Append, never overwrite.** Newest entry goes at the bottom. Never edit a past entry except to
  backfill its `Commit Hash`.
- **Commit Hash backfill.** Write `_pending_`, create the commit, then replace it with the real hash
  (`git rev-parse HEAD`) and record it in a small follow-up `docs:` commit — a commit cannot contain
  its own hash.
- **Honesty.** If a step was skipped or a check failed, say so explicitly. The journal records what
  actually happened.

## See Also

- [`../commands/update-journal.md`](../commands/update-journal.md) — The workflow that uses this template.
- [`../../dev_journal.md`](../../dev_journal.md) — The journal + required-fields spec + entry 0001.
- [`../commands/close-task.md`](../commands/close-task.md) — Gate 4 (write) and Gate 6 (backfill).
