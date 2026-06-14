# Claude Code Operating System (CCOS) — JobHunt AI

> This directory contains the Claude Code Operating System: reusable context, command definitions, and
> templates that reduce prompt repetition and enforce consistency across all Claude Code sessions on
> this project.

**Purpose:** Every contributor (human or AI) should be able to complete work on JobHunt AI without
re-reading PLAN.md in full. The CCOS is a distilled, ready-to-use reference layer that fits in a
single Claude Code context, reducing session friction and ensuring alignment with project standards.

---

## Directory Structure

> The repo-root [`CLAUDE.md`](../CLAUDE.md) is **auto-loaded by Claude Code at session start** and is
> the entry point to this directory. `.claude/HANDOFF.md` (when present) is the single *open*
> session-handoff note.

```
CLAUDE.md                        # Auto-loaded operating brief (repo root) — read first
.claude/
├── README.md                    # This file
├── HANDOFF.md                   # (transient) open session-handoff note, if any
├── context/
│   ├── project_rules.md         # Architecture constraints, non-negotiable principles
│   ├── architecture_summary.md  # Quick-reference system architecture
│   ├── sprint_status.md         # Current sprint + upcoming tasks
│   └── coding_checklist.md      # Code review + DoD checklist
├── commands/
│   ├── prime-context.md         # PRIME-CONTEXT: session-start orientation ritual
│   ├── start-task.md            # START-TASK workflow + requirements
│   ├── review-task.md           # REVIEW-TASK workflow + checklist
│   ├── security-review.md       # SECURITY-REVIEW scope + checklist
│   ├── close-task.md            # CLOSE-TASK workflow + enforcement
│   ├── update-journal.md        # How to update dev_journal.md
│   ├── create-adr.md            # CREATE-ADR workflow + template
│   └── session-handoff.md       # SESSION-HANDOFF: continuity at session end
└── templates/
    ├── task_plan.md             # Implementation plan template
    ├── adr_template.md          # ADR template (matches docs/decision_log.md)
    ├── journal_entry.md         # Journal entry template (matches dev_journal.md)
    ├── review_report.md         # Review report template
    ├── security_report.md       # Security review report template
    ├── session_handoff.md       # Session handoff note template
    └── sprint_retro.md          # Sprint retrospective template
```

---

## Quick Start for Claude Code Sessions

**Every session begins the same way:** the repo-root [`CLAUDE.md`](../CLAUDE.md) is auto-loaded, then
run **`/prime-context`** to orient to the live state and the next ready task. Only then start work.

After orienting, a human asking you to work on this project will typically either:

1. **Reference a task directly:** "Implement T05 (Async DB engine) from S1." ← You read the task card
   from [`docs/sprint_s1_execution_plan.md`](../docs/sprint_s1_execution_plan.md).
2. **Use a CCOS command:** "Use /start-task to plan T05." ← You use the workflow in
   [`commands/start-task.md`](commands/start-task.md).
3. **Read the full project:** "Before we start, read PLAN.md, the governance docs, and the S1 plan." ←
   You're thorough and context-rich.

In all cases, the CCOS files serve as a **fast, consistent reference** — not a replacement for
`docs/` and `PLAN.md`, but a complement.

---

## File Guide

### Context Files

| File | What it is | When to read it |
|------|-----------|-----------------|
| [`context/project_rules.md`](context/project_rules.md) | Distilled architecture constraints + non-negotiable pillars + coding standards. | Before implementing anything. |
| [`context/architecture_summary.md`](context/architecture_summary.md) | One-page system overview: layers, components, data flow, key decisions. | Before designing a feature or placing code. |
| [`context/sprint_status.md`](context/sprint_status.md) | Current sprint (S1), DoD checklist, upcoming tasks, blocked dependencies. | At session start to understand the roadmap. |
| [`context/coding_checklist.md`](context/coding_checklist.md) | Quality checklist for code review + Definition of Done verification. | Before marking a task complete. |

### Command Files

| File | What it does | When to use it |
|------|-------------|----------------|
| [`commands/prime-context.md`](commands/prime-context.md) | Session-start orientation: reads live status + git, finds the next ready task, detects drift. | **First, in every new session.** |
| [`commands/start-task.md`](commands/start-task.md) | Workflow for analyzing a task, identifying dependencies, and creating an implementation plan. | At the start of any implementation task. |
| [`commands/review-task.md`](commands/review-task.md) | Workflow for reviewing code against architecture, standards, tests, docs, and journal. | Before marking a task done. |
| [`commands/security-review.md`](commands/security-review.md) | Specialized review for auth, authz, secrets, env, SQL injection, prompt injection, logging, HITL gates. | For any task touching user data, credentials, or outbound actions. |
| [`commands/close-task.md`](commands/close-task.md) | Workflow to enforce all pre-commit checks: validation, tests, docs, journal, commit, verify. | At the very end of a task (Steps 3–8 of the workflow). |
| [`commands/update-journal.md`](commands/update-journal.md) | How to append a dev_journal.md entry with all required fields. | During task closeout (Step 6). |
| [`commands/create-adr.md`](commands/create-adr.md) | Workflow for deciding whether a task needs an ADR, and how to write one. | When a task makes a significant architectural decision. |
| [`commands/session-handoff.md`](commands/session-handoff.md) | Captures in-progress state so the next cold session can resume without loss. | When a task can't finish this session. |

### Template Files

| File | What it is | When to use it |
|------|-----------|----------------|
| [`templates/task_plan.md`](templates/task_plan.md) | Template for the implementation plan output by `/start-task`. | During task analysis (Step 1). |
| [`templates/adr_template.md`](templates/adr_template.md) | Template matching [`docs/decision_log.md`](../docs/decision_log.md) format. | When creating an ADR. |
| [`templates/journal_entry.md`](templates/journal_entry.md) | Template matching [`dev_journal.md`](../dev_journal.md) format with all required fields. | During task closeout (Step 6). |
| [`templates/review_report.md`](templates/review_report.md) | Template for structured code review output. | During task review (using `/review-task`). |
| [`templates/security_report.md`](templates/security_report.md) | Template for the `/security-review` report (8 areas + severities). | During a security review. |
| [`templates/session_handoff.md`](templates/session_handoff.md) | Template for the open handoff note (`.claude/HANDOFF.md`). | When ending a session mid-task. |
| [`templates/sprint_retro.md`](templates/sprint_retro.md) | Template for the end-of-sprint retrospective journal entry. | At sprint close (e.g. S1-T21). |

---

## Key Principles of the CCOS

1. **Distilled, not duplicate.** CCOS files summarize, reference, and operationalize — they don't
   duplicate PLAN.md or `docs/`. When CCOS and PLAN.md disagree, PLAN.md wins.
2. **Action-oriented.** Each command and template is a checklist or workflow, not just reading
   material.
3. **Fast context fit.** A fresh Claude Code session should be able to read all CCOS files in one
   context window and be ready to execute S1 tasks.
4. **Enforces the eight-step workflow.** Every command ties back to the eight-step task workflow in
   [`docs/development_workflow.md`](../docs/development_workflow.md).

---

## Reference

- [`../PLAN.md`](../PLAN.md) — Master specification (read in full before major changes).
- [`../docs/project_context.md`](../docs/project_context.md) — Project elevator pitch and context.
- [`../docs/architecture.md`](../docs/architecture.md) — Architecture, layers, and constraints.
- [`../docs/development_workflow.md`](../docs/development_workflow.md) — The eight-step task workflow.
- [`../docs/sprint_s1_execution_plan.md`](../docs/sprint_s1_execution_plan.md) — S1 task breakdown.
- [`../docs/coding_standards.md`](../docs/coding_standards.md) — Code quality rules.
- [`../dev_journal.md`](../dev_journal.md) — Project engineering history.
