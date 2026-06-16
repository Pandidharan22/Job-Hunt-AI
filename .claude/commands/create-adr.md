# CREATE-ADR Command

> Used to decide whether a task warrants an Architecture Decision Record, and — if it does — to write
> one and register it in [`docs/decision_log.md`](../../docs/decision_log.md). ADRs capture the *why*
> behind significant technical choices so future contributors (and memoryless AI sessions) don't have
> to reverse-engineer them.

---

## When to Use

During or right after a task (workflow Step 5), whenever you make a decision that future work would
otherwise have to guess at.

```
You: "T07 sets the ORM naming convention and mixins — that's binding for every model. I'll use
/create-adr to record ADR-0013 before writing models."
```

---

## Step 1 — Decide Whether an ADR Is Needed

Write an ADR if the change does **any** of these:

- [ ] Introduces or replaces a framework, library, datastore, or external service.
- [ ] Establishes a cross-cutting pattern (auth/token model, error handling, agent contract, eventing,
      ORM conventions, test strategy, container runtime).
- [ ] Makes a trade-off future contributors would otherwise have to reverse-engineer.
- [ ] **Deviates from [`PLAN.md`](../../PLAN.md)** — the ADR must explain why, and `PLAN.md` should be
      updated to match.

**Do NOT** write an ADR for routine feature work that follows an existing pattern — the
[`dev_journal.md`](../../dev_journal.md) entry is sufficient there.

### ADRs Already Expected in S1

The sprint plan anticipates these (see [`context/sprint_status.md`](../context/sprint_status.md)):

| ADR | Title | Trigger |
|-----|-------|---------|
| ADR-0013 | ORM conventions (UUID PKs, timestamp mixin, constraint/index naming) | Before T08 |
| ADR-0014 | Alembic async migration strategy | During T11 |
| ADR-0015 | Test database & fixture isolation strategy | During T13 |
| ADR-0016 | Authentication & token model (access/refresh/expiry/transport) | Before T14 |
| ADR-0017 | Container runtime & build (base image, uv-in-Docker, multi-stage, non-root) | During T16 |
| ADR-0018 | Ingress & port topology (nginx single entrypoint; root compose) | During T18 (optional) |
| ADR-0019 | S1 dependency scoping (defer langchain/playwright/crawl4ai to S2–S4) | During T02A |

---

## Step 2 — Assign the Next Number

- ADRs are numbered sequentially and zero-padded: `ADR-XXXX`.
- The log currently holds **ADR-0001 … ADR-0012**. The next free number is **ADR-0013**.
- Check the index table in [`docs/decision_log.md`](../../docs/decision_log.md) for the highest
  existing ID and add one. **Never reuse a number.**

---

## Step 3 — Write the ADR

Use [`templates/adr_template.md`](../templates/adr_template.md) (which matches the format already in
the decision log). Required fields:

| Field | Content |
|-------|---------|
| **Status** | `Proposed` → `Accepted` (or later `Deprecated` / `Superseded by ADR-YYYY`). |
| **Date** | `YYYY-MM-DD`. |
| **Deciders** | Who made the call. |
| **Context** | The forces at play — problem, constraints, requirements. |
| **Decision** | What was decided, stated plainly. |
| **Consequences** | Results — positive, negative, and follow-ups. |
| **Alternatives considered** | What else was evaluated and why it was rejected. |
| **References** | Links to `PLAN.md` sections, docs, external sources. |

Write **Context → Decision → Consequences** so a newcomer understands not just *what* but *why* and
*what it costs*.

---

## Step 4 — Register It in the Decision Log

Append to [`docs/decision_log.md`](../../docs/decision_log.md):

1. **Add an index row** to the table at the top (ID, title, status, date) — keep it in order.
2. **Append the full ADR body** in the numbered sequence below the existing ADRs.

ADRs are **append-only and immutable**: to change a past decision, write a new ADR that supersedes it
and mark the old one `Superseded by ADR-YYYY`. Never rewrite a historical ADR.

---

## Step 5 — Cross-Link

- Reference the new ADR from the relevant code (docstring/comment) and from the
  [`dev_journal.md`](../../dev_journal.md) entry for the task.
- If it is binding, consider adding it to the "Non-Negotiable Decision Records" table in
  [`context/project_rules.md`](../context/project_rules.md).

---

## Output

```markdown
## CREATE-ADR Summary

**Decision needed?** Yes — <one-line reason>.
**ADR:** ADR-00NN — <Title>  (Status: Accepted, Date: YYYY-MM-DD)
**Registered:** index row + body appended to docs/decision_log.md.
**Cross-linked:** journal entry, code docstring, project_rules.md (if binding).
```

If no ADR is needed: *"No ADR required — this follows existing pattern <X>; recorded in the journal
entry instead."*

## See Also

- [`templates/adr_template.md`](../templates/adr_template.md) — The ADR template.
- [`../../docs/decision_log.md`](../../docs/decision_log.md) — The log + ADR format spec + ADR-0001–0012.
- [`../../docs/development_workflow.md`](../../docs/development_workflow.md) — §6 "When to Write an ADR".
- [`../context/project_rules.md`](../context/project_rules.md) — Binding decision records.
