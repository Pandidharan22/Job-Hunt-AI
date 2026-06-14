# Review Report Template

> Output template for the [`/review-task`](../commands/review-task.md) command. Copy the block below
> and fill it from the automated results and the manual checklist in
> [`context/coding_checklist.md`](../context/coding_checklist.md). Delete this header when producing
> the real report.

---

```markdown
## REVIEW-TASK Report: S1-TNN — <Task Name>

**Date:** YYYY-MM-DD
**Reviewer:** Claude Code
**Branch:** <type>/s1-tNN-<slug>
**Scope:** <files / areas reviewed>

### 1. Validation Results (Automated)

| Check | Command | Result |
|-------|---------|--------|
| Format | `uv run ruff format --check .` | ✓ PASS / ✗ FAIL |
| Lint | `uv run ruff check .` | ✓ PASS / ✗ FAIL |
| Types | `uv run mypy app/` | ✓ PASS / ✗ FAIL |
| Tests | `uv run pytest tests/ --cov=app` | ✓ PASS (N tests, X% cov) / ✗ FAIL |

### 2. Checklist Results (Manual)

| Category | Checked | Notes |
|----------|---------|-------|
| Code Quality | N/M | <no placeholders/TODOs/print/secrets; docstrings present> |
| Architecture & Layering | N/M | <Clean Architecture respected; services have no business logic> |
| Async Correctness | N/M | <no blocking I/O on async paths; sessions not shared> |
| Testing | N/M | <unit + integration; mocks external; coverage ≥ target; no skips> |
| Documentation | N/M | <sprint board, .env.example, docstrings, ADR if needed> |
| Security | N/M / N/A | <secrets, validation, SQL, JWT, HITL — or N/A> |
| Git & Commits | N/M | <Conventional Commit; one logical change> |

### 3. Security Review (if applicable)

- **Ran `/security-review`?** Yes / No / N/A
- **Worst severity:** Critical / High / Medium / Low / None
- **Summary:** <one line, or "N/A — no auth/secrets/outbound touched">

### 4. Issues Found

1. **[Severity]** <issue> → <required fix>
2. ...

(Or "None — all checks pass.")

### 5. Final Verdict

- **✅ PASS** — Ready for `/close-task`.
- **❌ FAIL** — Fix the issues above and re-run `/review-task`.

### 6. Next Step

→ `/close-task` (if PASS) — runs Gates 1–6 and lands the commit.
→ Fix & re-review (if FAIL).
```

---

## Verdict Rules

- **Any** failed automated check (format / lint / types / tests) ⇒ overall **FAIL**.
- **Any** Critical/High security finding ⇒ overall **FAIL**.
- Never record PASS while a check is red. Report red as red, with the actual output.

## See Also

- [`../commands/review-task.md`](../commands/review-task.md) — The workflow that produces this report.
- [`../context/coding_checklist.md`](../context/coding_checklist.md) — The master checklist.
- [`../commands/security-review.md`](../commands/security-review.md) — Deep security pass.
- [`../commands/close-task.md`](../commands/close-task.md) — The finalization step after a PASS.
