# Task Plan Template

> Output template for the [`/start-task`](../commands/start-task.md) command. Copy the block below and
> fill every section. Pull the requirement/validation/estimate values **verbatim** from the task card
> in [`docs/sprint_s1_execution_plan.md`](../../docs/sprint_s1_execution_plan.md). Delete this header
> and the parenthetical hints when producing the actual plan.

---

```markdown
# Task Plan: S1-TNN — <Task Name>

## Task Summary
- **ID & Name:** S1-TNN — <name>
- **Objective (one sentence):** <what this task builds>
- **Why it matters:** <link to architecture/phase goal; what it unblocks>

## Requirements (from the task card)
- **Files Expected:**
  - `path/one` — <purpose>
  - `path/two` — <purpose>
- **Validation Criteria:** <verbatim from the card>

## Dependency Check
| Upstream task | Required? | Status (☑/☐) | Confirmed in sprint_tracking.md? |
|---------------|-----------|--------------|----------------------------------|
| T0x | yes | ☑ | yes |

- **Verdict:** ✅ All dependencies complete — cleared to start.
  / ⛔ Blocked — <which dependency is incomplete>. **Refuse and report.**

## Critical Path & Parallelization
- **On the critical path?** Yes / No — <which downstream tasks depend on this>.
- **Parallelizable with:** <task IDs, or "none">.

## Architecture Alignment
- **Layers touched:** <e.g. `app/core/`, `tests/integration/`>
- **Clean Architecture respected?** Yes — <one-line why> (inner rings don't import outer).
- **Invariants in play:** <HITL? privacy? no-fabrication? type-safety? — or "none for this task">.
- **ADR required?** Yes (ADR-00NN — <title>) / No — follows existing pattern.

## Technical Approach
<2–3 paragraphs on *how* it will be implemented.>
- **Key decisions:** <choices made and why, over the alternatives>.
- **Risks / unknowns:** <edge cases, version compatibility, platform quirks>.

## Deliverables Checklist
- [ ] `path/one` — <description>
- [ ] `path/two` — <description>
- [ ] `tests/.../test_*.py` — <description>
- [ ] Docs updated: <which docs>
- [ ] ADR (if needed): ADR-00NN

## Validation Plan
- **Format/lint:** `cd backend && uv run ruff format --check . && uv run ruff check .`
- **Types:** `uv run mypy app/`
- **Tests:** `uv run pytest tests/ --cov=app --cov-report=term-missing` (target ≥ 70% Phase 1)
- **Other:** <task-specific checks, e.g. `alembic upgrade head` then empty autogenerate diff>

## Estimates (from the task card)
- **Complexity:** N/10
- **Risk:** N/10
- **Estimated context usage:** Low / Medium / High

## Definition of Done
- [ ] Code complete — no placeholders, `TODO`s, or stubs.
- [ ] `ruff` + `mypy` clean.
- [ ] Tests written & green; coverage ≥ target.
- [ ] Google-style docstrings present.
- [ ] Affected docs updated; ADR written if needed.
- [ ] Journal entry appended (`Commit Hash: _pending_`).
- [ ] Conventional Commit created on a task branch.
- [ ] Commit verified; hash backfilled into the journal.

## Next Steps
1. Confirm this plan with the user.
2. Implement following the eight-step workflow.
3. Run `/review-task`, then `/close-task`.
```

---

## See Also

- [`../commands/start-task.md`](../commands/start-task.md) — Produces this plan.
- [`../../docs/sprint_s1_execution_plan.md`](../../docs/sprint_s1_execution_plan.md) — Source task cards.
- [`../context/project_rules.md`](../context/project_rules.md) — Constraints to honor.
