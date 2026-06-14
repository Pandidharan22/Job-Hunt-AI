# CLOSE-TASK Command

> Used to finish a task by enforcing every pre-commit gate. This command is the final discipline of
> the eight-step workflow — it runs **Steps 3–8** (Validate → Test → Document → Journal → Commit →
> Verify) and refuses to complete a task if any gate fails.

---

## When to Use

After the code is written and `/review-task` has passed:

```
You: "Implementation done and review passed. I'll use /close-task to finalize."
```

`/start-task` opens a task; `/review-task` inspects it; **`/close-task` lands it**.

---

## The Six Enforcement Gates

Each gate is **mandatory**. If a gate fails, **stop, report honestly, and do not proceed** — never
fake a pass, skip a step, weaken a check, or comment out a failing assertion.

### Gate 1 — Validation (Workflow Step 3)

```bash
cd backend
uv run ruff format --check .
uv run ruff check .
uv run mypy app/
```

- **Pass criteria:** formatter, linter, and type checker all report zero errors.
- **If it fails:** fix the code (`uv run ruff format .` to auto-format, then re-check). Do not add
  `# type: ignore` without a trailing reason comment.

### Gate 2 — Tests (Workflow Step 4)

```bash
cd backend
uv run pytest tests/ --cov=app --cov-report=term-missing
```

- **Pass criteria:** full suite green; coverage meets the phase target (≥ 70% Phase 1, ≥ 90% Phase 6);
  no skipped/xfailed tests; new code has tests in this same change.
- **If it fails:** fix the code or the test. Never delete or skip a test to go green.

### Gate 3 — Documentation (Workflow Step 5)

Confirm every affected doc is updated:

- [ ] [`docs/sprint_tracking.md`](../../docs/sprint_tracking.md) — deliverable ticked / task status updated.
- [ ] [`context/sprint_status.md`](../context/sprint_status.md) — task status row updated (☐ → ☑).
- [ ] `backend/.env.example` — updated if config changed.
- [ ] [`docs/architecture.md`](../../docs/architecture.md) / [`docs/coding_standards.md`](../../docs/coding_standards.md) — updated if a design or convention changed.
- [ ] API/OpenAPI descriptions and docstrings current.
- [ ] ADR written if a significant decision was made — see [`create-adr.md`](create-adr.md).

### Gate 4 — Journal (Workflow Step 6)

- Append a complete entry to [`dev_journal.md`](../../dev_journal.md) using
  [`update-journal.md`](update-journal.md) and [`templates/journal_entry.md`](../templates/journal_entry.md).
- All 13 required fields present; **Commit Hash** left as `_pending_` (backfilled in Gate 6).

### Gate 5 — Commit Creation (Workflow Step 7)

```bash
git checkout -b <type>/s1-tNN-<slug>     # if not already on a task branch
git add <files>
git commit -F <message-file>             # Conventional Commit + co-author trailer
```

- **Pass criteria:** a Conventional Commit (`<type>(<scope>): <summary>`) on a task branch, body
  explaining *why*, ending with the `Co-Authored-By:` trailer. One logical change per commit.

### Gate 6 — Commit Verification (Workflow Step 8)

```bash
git log -1 --stat
git status
git rev-parse HEAD
```

- **Pass criteria:** commit exists with the intended files; working tree clean.
- **Backfill the hash:** replace `_pending_` in the journal entry with the real hash, then make a
  small follow-up `docs:` commit (e.g. `docs: backfill commit hash for S1-TNN journal entry`) — a
  commit cannot contain its own hash. See the [dev journal conventions](../../dev_journal.md).

---

## Closeout Summary (Output)

After all six gates pass, report:

```markdown
## CLOSE-TASK Summary: S1-TNN — <Task Name>

| Gate | Result |
|------|--------|
| 1. Validation (ruff + mypy) | ✓ PASS |
| 2. Tests (X passed, Y% coverage) | ✓ PASS |
| 3. Documentation updated | ✓ PASS |
| 4. Journal entry appended | ✓ PASS |
| 5. Commit created | ✓ PASS (`<hash>`) |
| 6. Commit verified + hash backfilled | ✓ PASS |

**Branch:** <type>/s1-tNN-<slug>
**Next:** merge (squash) to `main`, then start <next task> with /start-task.
```

---

## Refusals (Do NOT Close)

Refuse to close the task and report the reason if:

- [ ] Any validation command reports an error.
- [ ] Any test fails, is skipped, or coverage is below target.
- [ ] A required doc is stale or contradicts the code.
- [ ] The journal entry is missing or incomplete.
- [ ] The commit is not a valid Conventional Commit or the trailer is missing.
- [ ] The working tree is not clean after committing.

**Honesty rule:** if a gate cannot pass, say so with the actual command output. A red result is
reported as red.

---

## Pre-Conditions for Use

1. Implementation is complete (Steps 1–2 done).
2. `/review-task` has passed.
3. All backend tooling is installed and runnable.

## See Also

- [`start-task.md`](start-task.md) — Opens the task (Step 1).
- [`review-task.md`](review-task.md) — Inspects the task before closeout.
- [`update-journal.md`](update-journal.md) — How to write the journal entry (Gate 4).
- [`security-review.md`](security-review.md) — Run before Gate 5 if the task touches auth/secrets/outbound.
- [`../context/coding_checklist.md`](../context/coding_checklist.md) — The master Definition of Done.
- [`../../docs/development_workflow.md`](../../docs/development_workflow.md) — The eight-step workflow.
