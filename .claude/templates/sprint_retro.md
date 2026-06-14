# Sprint Retrospective Template

> Template for the retrospective entry written at sprint close (the final task of each sprint — e.g.
> S1-T21). It is appended to [`dev_journal.md`](../../dev_journal.md) as a journal entry **and**
> summarized in [`docs/sprint_tracking.md`](../../docs/sprint_tracking.md). Copy the block below and
> fill it. Delete this header when producing the real retro.

---

```markdown
### YYYY-MM-DDTHH:MMZ — Sprint <ID> Retrospective

- **Timestamp:** YYYY-MM-DDTHH:MMZ (HH:MM <TZ>)
- **Sprint:** S<ID> — <name>
- **Phase:** Phase <N> — <name>
- **Task Name:** Sprint <ID> closeout & retrospective
- **Objective:** Verify the sprint Definition of Done, record outcomes, and capture lessons for the
  next sprint.

#### Sprint Definition of Done — verification

- [ ] <DoD item 1> — ✓ / ✗ (<evidence>)
- [ ] <DoD item 2> — ✓ / ✗
- ... (one row per S<ID> exit criterion)

#### Outcomes

- **Tasks completed:** <N>/<M> (list any deferred + why).
- **Coverage:** <X%> (target: <≥70% Phase 1 / ≥90% Phase 6>).
- **ADRs added:** <ADR-00NN … list>.
- **Commits:** <count> on `main`; tag <vX.Y.Z-...> applied? <yes/no>.

#### What went well

- <bullet>

#### What was hard / bottlenecks

- <bullet — e.g. a risk that materialized, a task that took longer>

#### Lessons learned / process changes

- <bullet — concrete change to carry into the next sprint>
- <CCOS/doc updates this implies, if any>

#### Risks carried into next sprint

- <bullet, with mitigation owner>

- **Files Modified:** `docs/sprint_tracking.md`, `README.md`, `dev_journal.md`, `CLAUDE.md` (status block).
- **Technical Decisions:** <e.g. tag release, advance phase status>.
- **Reasoning Behind Decisions:** <...>
- **Problems Encountered:** <...>
- **Solutions Applied:** <...>
- **Validation Performed:** full suite green, coverage ≥ target, CI green on `main`.
- **Commit Hash:** _pending_
- **Next Recommended Task:** S<ID+1>-T01 — <first task of the next sprint>.
```

---

## When to Use

- The last task of every sprint (S1-T21 for S1).
- Also referenced by the "Sprint Retrospective" checklist in
  [`../context/coding_checklist.md`](../context/coding_checklist.md).

## Don't Forget (sprint-close housekeeping)

- [ ] Flip the sprint to ☑ in [`docs/sprint_tracking.md`](../../docs/sprint_tracking.md) and
      [`../context/sprint_status.md`](../context/sprint_status.md).
- [ ] Update the "Where we are right now" block in [`../../CLAUDE.md`](../../CLAUDE.md).
- [ ] Update the README status banner if the phase advanced.
- [ ] Confirm every task journal entry has its real commit hash (no `_pending_` left).
- [ ] Tag the release if the sprint plan calls for it.

## See Also

- [`journal_entry.md`](journal_entry.md) — Base journal entry format this extends.
- [`../context/coding_checklist.md`](../context/coding_checklist.md) — Sprint retrospective checklist.
- [`../../docs/sprint_tracking.md`](../../docs/sprint_tracking.md) — Where the summary lands.
