# Security Report Template

> Output template for the [`/security-review`](../commands/security-review.md) command. Copy the block
> below and fill it from the eight review areas. A **Critical** or **High** finding blocks the merge.
> Delete this header when producing the real report.

---

```markdown
## SECURITY-REVIEW Report: S1-TNN — <Task Name>

**Date:** YYYY-MM-DD
**Reviewer:** Claude Code
**Scope:** <files / modules / areas reviewed>
**Trigger:** <auth change / secrets / outbound action / checkpoint CP-C / routine>

### Area Results

| # | Area | Result | Worst Severity |
|---|------|--------|----------------|
| 1 | Authentication | ✓ PASS / ✗ FAIL / N/A | — |
| 2 | Authorization | ✓ PASS / ✗ FAIL / N/A | — |
| 3 | Secrets | ✓ PASS / ✗ FAIL / N/A | — |
| 4 | Environment variables | ✓ PASS / ✗ FAIL / N/A | — |
| 5 | SQL injection | ✓ PASS / ✗ FAIL / N/A | — |
| 6 | Prompt injection | ✓ PASS / ✗ FAIL / N/A | — |
| 7 | Logging leaks | ✓ PASS / ✗ FAIL / N/A | — |
| 8 | Human-in-the-loop | ✓ PASS / ✗ FAIL / N/A | — |

### Findings

| Severity | Area | Location (file:line) | Description | Required Fix |
|----------|------|----------------------|-------------|--------------|
| Critical/High/Medium/Low/Info | <area> | `path:line` | <what> | <fix> |

(Or "None — all in-scope areas pass.")

### Evidence (commands run)

```bash
grep -rEi "(api_key|secret|password|token)\s*=\s*['\"]" backend/app/   # secrets scan
grep -rEn "execute\(|text\(|f\".*SELECT" backend/app/                  # raw SQL scan
grep -rn "print(" backend/app/                                         # logging scan
```
<results / "clean">

### Verdict

- **✅ PASS** — no Critical/High findings; safe to proceed to `/close-task`.
- **❌ FAIL** — fix Critical/High findings and re-run `/security-review`.

### Notes

- HITL gates checked: <which outbound paths>.
- No-fabrication prompt intact: <yes / N/A>.
- Ran built-in `/security-review` skill alongside: <yes / no>.
```

---

## Severity Guide

| Severity | Meaning | Action |
|----------|---------|--------|
| **Critical** | Exploitable now; data loss, account takeover, or a broken invariant (e.g. outbound without HITL). | Block. Fix immediately. |
| **High** | Serious weakness, likely exploitable (e.g. secret in code, missing authz check). | Block. Fix before merge. |
| **Medium** | Weakness needing conditions to exploit. | Fix this task or file a tracked follow-up. |
| **Low** | Minor hardening opportunity. | Fix opportunistically. |
| **Info** | Observation, no action required. | Note only. |

## See Also

- [`../commands/security-review.md`](../commands/security-review.md) — The workflow that produces this report.
- [`review_report.md`](review_report.md) — General review report (has a security summary section).
- [`../context/project_rules.md`](../context/project_rules.md) — Invariants (HITL, privacy, no fabrication).
