# ADR Template

> Template for a new Architecture Decision Record, used by [`/create-adr`](../commands/create-adr.md).
> It matches the format already in [`docs/decision_log.md`](../../docs/decision_log.md). Two things get
> written: (1) an **index row** in the table at the top of the decision log, and (2) the **full ADR
> body** appended in numbered sequence. Delete this header when producing the real ADR.

---

## 1. Index Row (add to the table at the top of `docs/decision_log.md`)

```markdown
| [ADR-00NN](#adr-00nn--short-title) | <Short Title> | Accepted | YYYY-MM-DD |
```

> The anchor follows GitHub's slug rules: lowercase, spaces → `-`, em-dash dropped (so
> `ADR-0013 — ORM conventions` → `#adr-0013--orm-conventions`, note the double hyphen).

## 2. ADR Body (append below the last ADR in `docs/decision_log.md`)

```markdown
## ADR-00NN — <Short Title>

- **Status:** Proposed | Accepted | Deprecated | Superseded by ADR-YYYY
- **Date:** YYYY-MM-DD
- **Deciders:** <name(s) / role(s)>
- **Context:** <The forces at play — the problem, constraints, and requirements that make a decision
  necessary. State what is being pressured and why now.>
- **Decision:** <What was decided, stated plainly and unambiguously.>
- **Consequences:** <The results. Lead with the positive, then the negative ("(−) ..."), then any
  follow-up work this creates.>
- **Alternatives considered:** <Each option evaluated and the concrete reason it was rejected.>
- **References:** <Links to PLAN.md sections, docs/ files, ADRs this relates to, external sources.>
```

---

## Status Lifecycle

| Status | Meaning |
|--------|---------|
| **Proposed** | Under discussion; not yet in force. |
| **Accepted** | In force; code must comply. |
| **Deprecated** | Discouraged but not yet removed. |
| **Superseded** | Replaced by a later ADR (name it: `Superseded by ADR-YYYY`). |

## Rules

- **Append-only & immutable.** Never rewrite a historical ADR. To change a decision, write a new ADR
  that supersedes the old one and update the old one's status line only.
- **Sequential numbering, zero-padded.** Next free ID after ADR-0012 is **ADR-0013**. Never reuse a number.
- **Explain the trade-off.** A good ADR makes the *cost* of the decision explicit, not just the choice.

## Worked Example (shape only)

```markdown
## ADR-0013 — ORM conventions: UUID PKs, timestamp mixin, constraint/index naming

- **Status:** Accepted
- **Date:** 2026-06-20
- **Deciders:** Pandidharan G R
- **Context:** Every model in §7 needs UUID primary keys and created/updated timestamps, and Alembic
  autogenerate produces noisy diffs unless constraint/index names are deterministic...
- **Decision:** Define a single declarative `Base` with an explicit SQLAlchemy naming convention plus
  `UUIDPKMixin` and `TimestampMixin`; all models inherit them...
- **Consequences:** (+) Clean, repeatable migrations; consistent schema. (−) All models must use the
  mixins; a later convention change forces a migration...
- **Alternatives considered:** Per-model ad-hoc columns — rejected (inconsistency, migration churn)...
- **References:** PLAN.md §7, §7.2; docs/architecture.md §3; S1-T07.
```

## See Also

- [`../commands/create-adr.md`](../commands/create-adr.md) — The workflow that uses this template.
- [`../../docs/decision_log.md`](../../docs/decision_log.md) — Format spec + ADR-0001 … ADR-0012.
