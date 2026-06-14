# Session Handoff Template

> Template for [`/session-handoff`](../commands/session-handoff.md). When a task can't finish this
> session, copy the block below into `.claude/HANDOFF.md` (a single, overwritten file representing the
> one *open* handoff). The next session's [`/prime-context`](../commands/prime-context.md) reads it,
> resumes, and **deletes it on completion**. Delete this header when producing the real note.

---

```markdown
# Session Handoff — OPEN

**Written:** YYYY-MM-DDTHH:MMZ
**Task:** S1-TNN — <name>
**Branch:** <type>/s1-tNN-<slug>

## Progress (eight-step workflow)

- [x] 1. Analyze
- [x] 2. Implement — **partial** (<what's done>)
- [ ] 3. Validate
- [ ] 4. Test
- [ ] 5. Document
- [ ] 6. Journal
- [ ] 7. Commit
- [ ] 8. Verify

## State of the work

- **What's done:** <summary of completed implementation>
- **What's left:** <the remaining work, concretely>
- **Where the code is:**
  - WIP commit: `<hash>` on `<branch>` (message: "<...>")  — or
  - Stashed: `git stash list` → `<ref>`  — or
  - Uncommitted in working tree: `path/one`, `path/two`
- **Files touched so far:**
  - `path/one` — <state>
  - `path/two` — <state>

## Next concrete action

1. <the single next step to take on resume>
2. <then ...>

## Blockers / decisions pending

- <blocker, question for the user, or "none">
- **ADR needed?** <ADR-00NN — title / no>

## Watch-outs

- **Owed hash backfill?** <yes — entry / no>
- **Invariants in play:** <HITL / no-fabrication / type-safety / none>
- <any gotcha the next session must know>

## Resume protocol

Next session: run `/prime-context`, read this file, reconcile with `git status`, finish via
`/close-task`, then **delete `.claude/HANDOFF.md`**.
```

---

## Rules

- **One open handoff.** `HANDOFF.md` is overwritten each time and deleted when the task closes.
- **Never fake completion.** This note is for *unfinished* work; a finished task gets a journal entry
  and commit instead.
- **Point to durable artifacts.** Reference commit hashes / stash refs / file paths — not memory.

## See Also

- [`../commands/session-handoff.md`](../commands/session-handoff.md) — Produces this note.
- [`../commands/prime-context.md`](../commands/prime-context.md) — Consumes it next session.
