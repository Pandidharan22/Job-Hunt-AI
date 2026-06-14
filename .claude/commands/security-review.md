# SECURITY-REVIEW Command

> A specialized, project-specific security review for JobHunt AI. Run it on any change that touches
> authentication, authorization, secrets, environment variables, database queries, LLM prompts,
> logging, or **any outbound action**. It complements the general-purpose `/security-review` skill
> with rules that are specific to this project's invariants (HITL gates, no-fabrication, privacy-first).

---

## When to Use

- Any task touching `app/core/security.py`, `app/api/v1/auth.py`, `app/core/config.py`,
  `app/services/*`, `app/agents/*`, or DB query construction.
- Any task that sends email/DMs, submits forms, or otherwise acts outside the user's machine.
- As part of **CP-C** (after T14, authentication) and before merging any auth/outbound change.

```
You: "This change adds JWT auth. I'll run /security-review before /close-task."
```

---

## The Eight Review Areas

For each area: inspect, grep, and report findings with a severity
(**Critical / High / Medium / Low / Info**). A **Critical** finding blocks the merge.

### 1. Authentication

- [ ] Passwords hashed with `passlib[bcrypt]` — never stored or compared in plaintext.
- [ ] JWTs signed with the configured `SECRET_KEY` (≥ 32 chars) and `ALGORITHM` (HS256).
- [ ] Access tokens have a short expiry (`ACCESS_TOKEN_EXPIRE_MINUTES`); refresh flow is explicit.
- [ ] Tampered, expired, and malformed tokens are rejected (negative tests exist).
- [ ] No auth bypass paths; protected routes actually require a valid token.
- **Check:** `app/core/security.py`, `app/api/v1/auth.py`, `tests/integration/test_auth.py`.

### 2. Authorization

- [ ] Every resource is scoped to its owner — a user can only read/write their own `applications`,
      `resumes`, `conversations`, etc. (row-level ownership via `user_id`).
- [ ] No endpoint returns another user's data (IDOR check: requesting `/applications/{id}` of another
      user → 404/403).
- [ ] Admin/privileged actions (if any) are gated behind explicit role checks.
- **Check:** route dependencies, query filters (`WHERE user_id = current_user.id`).

### 3. Secrets

- [ ] No real secrets committed — only `.env.example` with placeholder values.
- [ ] No hardcoded API keys, passwords, tokens, or connection strings in code.
- [ ] Secrets are read via `pydantic-settings`, never inlined.
- **Check:** `grep -rEi "(api_key|secret|password|token)\s*=\s*['\"][^'\"]+" backend/app/` →
  only config field declarations, no literal credentials. Scan git history for accidental leaks.

### 4. Environment Variables

- [ ] Every variable in [`PLAN.md` §12.2](../../PLAN.md) is represented in `Settings` and
      `.env.example` (1:1, no drift).
- [ ] Required secrets have **no insecure defaults** (e.g. `SECRET_KEY` must be supplied, not
      defaulted to a known value in production paths).
- [ ] Feature-flag safety defaults are correct: `AUTO_APPLY_ENABLED=false`,
      `COLD_EMAIL_ENABLED=false`, `REQUIRE_APPLY_APPROVAL=true`.
- **Check:** `app/core/config.py` vs `.env.example` vs §12.2.

### 5. SQL Injection

- [ ] All queries go through the SQLAlchemy ORM / parameterized constructs — **no f-strings or
      string concatenation** building SQL.
- [ ] Any raw SQL uses bound parameters (`text(...).bindparams(...)`), never interpolated input.
- **Check:** `grep -rEn "execute\(|text\(|f\".*SELECT|%.*SELECT" backend/app/` → no interpolated SQL.

### 6. Prompt Injection

> Specific to this AI-native project. Untrusted text (job descriptions, scraped pages, inbound
> emails) flows into LLM prompts and must never be able to hijack agent behavior.

- [ ] Untrusted content (JD text, scraped HTML, email bodies) is clearly delimited and treated as
      **data**, not instructions, in prompts.
- [ ] System prompts are fixed server-side; user/scraped content cannot override them — especially the
      Resume Tailor's **no-fabrication** system prompt (must remain verbatim).
- [ ] LLM output that triggers actions (e.g. classification → status change, form-fill values) is
      validated/whitelisted before use; low-confidence results are flagged, not auto-applied.
- [ ] No scraped/email content is `eval`'d, executed, or used to build queries/paths unsanitized.
- **Check:** `app/agents/*`, prompt templates, `email_parser` confidence gating (< 0.7 → manual review).

### 7. Logging Leaks

- [ ] No secrets, tokens, passwords, full email bodies, or full resume content in logs.
- [ ] Loguru is the only logger; **no `print()`** in committed code.
- [ ] Exceptions don't serialize sensitive request bodies into logs.
- **Check:** `grep -rn "print(" backend/app/` → empty; review `logger.*` calls near auth/email/secrets.

### 8. Human-in-the-Loop (HITL) Requirements

> The project's hardest invariant ([ADR-0009](../../docs/decision_log.md)).

- [ ] **Every outbound action** (application submit, referral message, cold email, deal reply) passes
      an explicit approval gate before it executes.
- [ ] Approval is **on by default**; auto modes are opt-in and capped
      (`MAX_APPLIES_PER_DAY`, `MAX_COLD_EMAILS_PER_DAY`).
- [ ] The agent prepares a preview and emits `approval_required`; nothing sends on a rejected/expired
      approval.
- [ ] Cold/outreach emails include an unsubscribe footer and honor volume caps (CAN-SPAM/GDPR).
- **Check:** `app/agents/{apply_agent,referral_agent,cold_email,deal_closer}.py`, send paths in
  `app/services/email.py`.

> *For S1 specifically:* areas 5–8 are mostly forward-looking (agents/email arrive in S2–S5), but
> **1–4 and 7 apply now** to the auth, config, and database work.

---

## Output: Security Review Report

```markdown
## SECURITY-REVIEW Report: S1-TNN — <Task Name>

**Date:** YYYY-MM-DD · **Scope:** <files/areas reviewed>

| Area | Result | Severity (worst) |
|------|--------|------------------|
| 1. Authentication | ✓ / ✗ / N/A | — |
| 2. Authorization | ✓ / ✗ / N/A | — |
| 3. Secrets | ✓ / ✗ / N/A | — |
| 4. Environment variables | ✓ / ✗ / N/A | — |
| 5. SQL injection | ✓ / ✗ / N/A | — |
| 6. Prompt injection | ✓ / ✗ / N/A | — |
| 7. Logging leaks | ✓ / ✗ / N/A | — |
| 8. Human-in-the-loop | ✓ / ✗ / N/A | — |

### Findings
1. [Severity] <area>: <description> → <required fix>

(Or "None — all areas pass.")

### Verdict
**PASS** — safe to proceed to /close-task.
**FAIL** — fix Critical/High findings and re-run.
```

---

## Rules

- A **Critical** or **High** finding blocks the merge — fix it, don't defer it.
- **Never weaken a HITL gate, the no-fabrication prompt, or token validation to make a test pass.**
- Report findings honestly with evidence (file:line); don't claim "secure" without checking.

## See Also

- [`review-task.md`](review-task.md) — General review (calls this command when auth/secrets involved).
- [`../context/project_rules.md`](../context/project_rules.md) — Invariants (HITL, privacy, no fabrication).
- [`../../docs/decision_log.md`](../../docs/decision_log.md) — ADR-0005 (privacy), ADR-0009 (HITL).
- [`../../PLAN.md`](../../PLAN.md) — §12.2 env vars, §14 risks (compliance, ToS, prompt safety).
- Built-in `/security-review` skill — general OWASP-style pass; run alongside this for broad coverage.
