# CLAUDE.md — JobHunt AI Operating Brief

> **Auto-loaded into every Claude Code session.** This is the high-signal orientation layer. Read it
> first, then use the [Claude Code Operating System](.claude/README.md) (`.claude/`) for commands and
> templates. When this file and [`PLAN.md`](PLAN.md) disagree, **`PLAN.md` wins**.

---

## What this project is

**JobHunt AI** — an AI-native, open-source, self-hostable, multi-agent platform that automates the
job-hunting pipeline (discover → tailor → apply → track → outreach → close), with human approval
gates, running privately on the user's machine. Full spec: [`PLAN.md`](PLAN.md).

## Where we are right now

- **Phase:** Phase 1 — Core Foundation & Job Discovery
- **Sprint:** **S1 — Project Bootstrap** · status **◐ In progress**
- **Next task:** **S1-T01B — Monorepo directory skeleton** (T01A repo-hygiene files are done)
- **State of `main`:** governance + CCOS docs + repo-hygiene files
  (`.gitattributes`/`.editorconfig`/`.gitignore`). **No application code has been written yet.**
- **Live status:** [`.claude/context/sprint_status.md`](.claude/context/sprint_status.md) ·
  [`docs/sprint_s1_execution_plan.md`](docs/sprint_s1_execution_plan.md)

> Keep this section current. Update it at sprint/task transitions (it is the first thing every
> session reads).

## Non-negotiable invariants (violating these is a bug, not a choice)

1. **Human-in-the-Loop on every outbound action.** No application submitted, no email/DM sent without
   an explicit approval gate (on by default). [ADR-0009]
2. **Privacy-first.** Local Ollama inference by default; cloud LLM is opt-in. [ADR-0005]
3. **No resume fabrication.** The Resume Tailor only rephrases/reorders real content — never invents
   experience, dates, titles, or metrics. The no-fabrication system prompt is verbatim and mandatory.
4. **Clean Architecture.** Dependencies point inward; `services/` hold **zero** business logic. [ADR-0012]
5. **Type-safe end-to-end.** `mypy --strict`, Pydantic v2, TypeScript strict.
6. **Quality rules.** No placeholder code, no `TODO`/`FIXME`, no mock implementations unless requested,
   simplicity over abstraction.

Details: [`.claude/context/project_rules.md`](.claude/context/project_rules.md).

## The eight-step task workflow (never skip a step)

`1 Analyze → 2 Implement → 3 Validate (ruff+mypy) → 4 Test (pytest) → 5 Document → 6 Journal → 7 Commit → 8 Verify`

If a step fails, **stop and report it honestly** — never fake green, skip a test, or weaken a check.
Full definition: [`docs/development_workflow.md`](docs/development_workflow.md).

## Git rules

- **Conventional Commits** only: `<type>(<scope>): <summary>` (`feat`/`fix`/`refactor`/`docs`/`test`/`chore`).
- **Branch per task** off `main`: `<type>/s1-tNN-<slug>`. `main` is always green.
- Every commit message ends with: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
- Commit/push **only when the user asks**. If on `main`, branch first.

## CCOS commands (in `.claude/commands/`)

| Command | Use it to |
|---------|-----------|
| `/prime-context` | **Start here every session** — orient to current state and the next ready task. |
| `/start-task` | Plan a task; refuse if dependencies are incomplete. |
| `/review-task` | Verify code against architecture, standards, tests, docs, commits. |
| `/security-review` | Audit auth, secrets, env, SQL/prompt injection, logging, HITL. |
| `/close-task` | Enforce validation → tests → docs → journal → commit → verify. |
| `/update-journal` | Append a complete `dev_journal.md` entry. |
| `/create-adr` | Decide on and record an Architecture Decision Record. |
| `/session-handoff` | Write a continuity note before ending a session mid-task. |

## Cross-session discipline (how we stay consistent across many sessions)

Sessions start cold and have no memory of prior ones. To prevent drift:

- **Begin with `/prime-context`** — never assume you know the current task; read the live status.
- **Trust the journal + git, not memory.** [`dev_journal.md`](dev_journal.md) + `git log` are the
  source of truth for what actually happened.
- **One task at a time.** Finish a task through all eight steps (or `/session-handoff`) before
  starting another. Don't leave half-done work without a handoff note.
- **Backfill commit hashes.** A journal entry's `Commit Hash` starts `_pending_`; record the real
  hash after committing (a commit can't contain its own hash).
- **Keep CCOS in sync with `docs/`.** The `.claude/context/` files distill `docs/` and `PLAN.md`; if
  you change a doc, update the matching CCOS file (and this section's "Where we are" block).

## Key documents

- [`PLAN.md`](PLAN.md) — master specification (source of truth).
- [`.claude/README.md`](.claude/README.md) — the CCOS index.
- [`docs/architecture.md`](docs/architecture.md) · [`docs/coding_standards.md`](docs/coding_standards.md) ·
  [`docs/development_workflow.md`](docs/development_workflow.md) · [`docs/decision_log.md`](docs/decision_log.md)
- [`docs/sprint_s1_execution_plan.md`](docs/sprint_s1_execution_plan.md) — the 21 S1 task cards.

> **Do not begin Sprint S1 implementation unless explicitly asked.** Default to planning, review, and
> CCOS work; wait for the user to direct coding.
