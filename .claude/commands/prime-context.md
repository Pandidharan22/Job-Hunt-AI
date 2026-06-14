# PRIME-CONTEXT Command

> **The session-start ritual.** Run this first in any new Claude Code session before doing task work.
> It orients a cold session to the project's current state, verifies the working tree, and identifies
> the next dependency-ready task — so you never start work from a wrong or stale assumption.

---

## Why this exists

Each Claude Code session starts with **no memory of previous sessions**. Without a deliberate
orientation step, a session can resume the wrong task, duplicate finished work, or act on stale
status. `/prime-context` is the antidote: it rebuilds an accurate picture of "where we are" from the
durable sources (docs, journal, git) rather than from assumption.

```
User: "Let's continue."
You: "I'll run /prime-context first to orient to the current state."
```

---

## Workflow

### 1. Read the orientation layer (in order)

1. [`CLAUDE.md`](../../CLAUDE.md) — the operating brief (invariants, current sprint, next task).
2. [`context/sprint_status.md`](../context/sprint_status.md) — live sprint board, critical path, checkpoints.
3. [`context/project_rules.md`](../context/project_rules.md) — invariants and architecture constraints.
4. [`../dev_journal.md`](../../dev_journal.md) — read the **last 1–2 entries** to see what was just done
   and the "Next Recommended Task".

### 2. Verify the actual repository state (don't trust docs alone)

```bash
git branch --show-current        # Which branch?
git status --short               # Clean tree? Uncommitted work?
git log --oneline -5             # What landed recently?
```

Reconcile reality with the docs:

- Does `git log` match the journal's latest entry? If the last journal entry says `Commit Hash:
  _pending_`, a hash backfill is owed.
- Is the working tree clean? If not, there is **in-progress work** — inspect it before starting
  anything new (look for a handoff note via `/session-handoff`).
- Does `sprint_status.md` reflect what git shows? If not, flag the drift.

### 3. Identify the next ready task

From [`context/sprint_status.md`](../context/sprint_status.md) / [`docs/sprint_s1_execution_plan.md`](../../docs/sprint_s1_execution_plan.md):

- Find the lowest-numbered task whose **dependencies are all ☑ done** and which is **not yet done**.
- That is the **next ready task**. If several are ready (parallelizable), prefer the critical-path one.

### 4. Report the orientation (output)

```markdown
## Session Orientation

- **Phase / Sprint:** Phase 1 — Core Foundation / S1 — Project Bootstrap
- **Branch:** <branch> · **Working tree:** clean / <N> uncommitted files
- **Last completed:** <task / commit> (journal entry <date>)
- **Pending hash backfill?** No / Yes (<entry>)
- **In-progress work?** No / Yes — <summary + handoff note if present>
- **Next ready task:** S1-TNN — <name>  (deps: all ☑)
- **Blocked tasks:** <IDs whose deps are incomplete>
- **Drift detected:** None / <docs vs git mismatch to fix>

**Recommended next action:** `/start-task S1-TNN` — or address drift / backfill / handoff first.
```

### 5. Hand off to the right command

- Clean state, dependencies ready → **`/start-task`**.
- In-progress work from a prior session → review the handoff note, then continue or `/close-task`.
- Drift (docs ≠ git) → fix the docs first (small `docs:` commit), then proceed.
- Owed hash backfill → do it before new work.

---

## What this command does NOT do

- It does **not** start coding. It only orients and recommends.
- It does **not** modify files (except, if you choose, fixing detected drift — which is its own change).

## See Also

- [`../../CLAUDE.md`](../../CLAUDE.md) — Auto-loaded operating brief.
- [`start-task.md`](start-task.md) — The next step once oriented.
- [`session-handoff.md`](session-handoff.md) — Its counterpart at session end.
- [`../context/sprint_status.md`](../context/sprint_status.md) — Live status this command reads.
