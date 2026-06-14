# SESSION-HANDOFF Command

> **The session-end ritual** for work that isn't finished. When a session must stop mid-task, this
> command captures enough state that the **next** cold session can resume without loss — the
> counterpart to [`/prime-context`](prime-context.md).

---

## Why this exists

The eight-step workflow assumes a task runs to completion (commit + journal). Reality intrudes:
context runs out, the user steps away, a task is larger than one session. Without a handoff, the next
session re-derives state from scratch and risks duplicating or contradicting the in-progress work.
`/session-handoff` writes a durable note so continuity survives the session boundary.

```
User: "Let's stop here for today."
You: "There's unfinished work on T11. I'll run /session-handoff to record where we are."
```

---

## When to Use

- A task is **partially implemented** and will not be committed this session.
- You are blocked and need to record the blocker for whoever resumes.
- Context is running low mid-task.

If the task **is** complete, use [`/close-task`](close-task.md) instead — a finished task needs no
handoff, only its journal entry and commit.

---

## Workflow

### 1. Capture the state honestly

Gather, from reality:

```bash
git branch --show-current
git status --short          # what's changed but uncommitted
git diff --stat             # scope of in-progress changes
```

### 2. Decide what to do with uncommitted work

Pick one and state it explicitly in the note:

- **Commit a WIP checkpoint** on the task branch with a clear `chore(wip):` or `feat(...): [WIP]`
  message (only if it is coherent enough to commit; never commit broken code to `main`).
- **Leave it uncommitted** in the working tree (note exactly which files and why).
- **Stash it** (`git stash push -m "S1-TNN wip"`) and record the stash reference.

> Default: prefer a clearly-labelled WIP commit on the task branch, so the work is durable and
> recoverable. Never leave the repo in a state where the next session can't tell what's in flight.

### 3. Write the handoff note

Use [`templates/session_handoff.md`](../templates/session_handoff.md). Save it as
`.claude/HANDOFF.md` (a single, overwritten file — the *current* open handoff). Include:

- Task ID + name and which workflow steps are done vs remaining.
- Exactly what was changed (files), and where uncommitted work lives (branch/stash/working tree).
- The next concrete action to take.
- Any blocker, decision pending, or `_pending_` journal hash owed.

### 4. Point the journal (do NOT fake completion)

- Do **not** write a final journal entry for an unfinished task (the journal records *completed*
  tasks). Instead, reference the open `HANDOFF.md` so the next session finds it.
- If a WIP checkpoint was committed, note its hash in the handoff note (not in the journal).

### 5. Report (output)

```markdown
## Session Handoff Recorded

- **Task:** S1-TNN — <name> (steps done: 1–X of 8)
- **In-progress work:** <committed WIP `<hash>` / stashed `<ref>` / uncommitted files>
- **Next action:** <the single next concrete step>
- **Blockers:** <none / description>
- **Handoff note:** `.claude/HANDOFF.md`

Next session: run `/prime-context`, then open `.claude/HANDOFF.md` to resume.
```

---

## Resuming (next session)

The next session's [`/prime-context`](prime-context.md) checks for `.claude/HANDOFF.md`. When found,
read it, reconcile with `git status`, finish the task through `/close-task`, then **delete
`HANDOFF.md`** (it represents the *open* handoff; a closed task has none).

## Rules

- **Honesty over optics.** Record what is actually done and what is broken — not an idealized state.
- **No fake completion.** Never write a "done" journal entry for unfinished work.
- **One open handoff at a time.** `HANDOFF.md` is overwritten, then deleted on resume.

## See Also

- [`prime-context.md`](prime-context.md) — Reads the handoff on the next session.
- [`templates/session_handoff.md`](../templates/session_handoff.md) — The handoff note template.
- [`close-task.md`](close-task.md) — Use this instead when the task is actually finished.
