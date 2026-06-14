# Contributing to JobHunt AI

First off — thank you for considering a contribution. JobHunt AI is open source (MIT) and
community-driven. This guide explains how we work so your contribution lands smoothly.

> **TL;DR:** Read [`docs/project_context.md`](docs/project_context.md) for context, follow the
> eight-step [development workflow](docs/development_workflow.md) for every change, write to the
> [coding standards](docs/coding_standards.md), use Conventional Commits, and append a
> [`dev_journal.md`](dev_journal.md) entry for every completed task.

---

## 1. Project Status

JobHunt AI is in **Phase 0 — Foundation**. The full specification lives in [`PLAN.md`](PLAN.md) and
the development operating system (this guide + the `docs/`) is in place. Application implementation
begins with **Sprint S1 — Project Bootstrap** (see [`docs/sprint_tracking.md`](docs/sprint_tracking.md)).
Until S1 lands, the tooling commands below describe the **target** setup the bootstrap sprint creates.

## 2. Read These First

| Document | Why |
|----------|-----|
| [`PLAN.md`](PLAN.md) | The complete master specification. The source of truth. |
| [`docs/project_context.md`](docs/project_context.md) | What the project is and the vocabulary we use. |
| [`docs/architecture.md`](docs/architecture.md) | How the system is structured and layered. |
| [`docs/coding_standards.md`](docs/coding_standards.md) | How to write code that will be accepted. |
| [`docs/development_workflow.md`](docs/development_workflow.md) | The process every change must follow. |
| [`docs/decision_log.md`](docs/decision_log.md) | Why the major technical choices were made. |

## 3. Ground Rules

- **Quality is non-negotiable.** No placeholder code, no `TODO`/`FIXME` comments, no mock
  implementations unless explicitly requested. See the
  [Quality Rules](docs/coding_standards.md#1-quality-rules-absolute).
- **Respect the architecture.** Keep business logic in agents, integrations in services, and
  dependencies pointing inward ([`docs/architecture.md`](docs/architecture.md)).
- **Never weaken safety to pass a check.** The human-in-the-loop approval gates and the
  no-fabrication resume rule are invariants, not obstacles.
- **Report honestly.** If tests fail, say so. Don't skip or weaken tests to go green.

## 4. Development Environment Setup

> Available once Sprint S1 has scaffolded the repo. Prerequisites: Docker Desktop, Python 3.12,
> Node.js 20+, and [`uv`](https://github.com/astral-sh/uv).

```bash
# 1. Fork and clone
git clone https://github.com/<your-username>/Job-Hunt-AI
cd Job-Hunt-AI

# 2. Configure environment
cp .env.example .env          # fill in your own values; never commit real secrets

# 3. Start the infrastructure
docker-compose up -d postgres redis qdrant minio

# 4. Backend
cd backend
uv sync                       # install dependencies
uv run alembic upgrade head   # apply migrations
uv run uvicorn app.main:app --reload

# 5. Frontend (separate terminal)
cd apps/web
npm install
npm run dev
```

## 5. The Contribution Workflow

Every change follows the **eight-step workflow** in
[`docs/development_workflow.md`](docs/development_workflow.md). In short:

1. **Analyze** the requirement against `PLAN.md` and the sprint board.
2. **Implement** complete, real code.
3. **Validate** — `ruff` + `mypy` (and frontend lint/typecheck) pass clean.
4. **Test** — write tests; the suite is green; coverage meets the phase target.
5. **Document** — update every affected doc.
6. **Journal** — append a [`dev_journal.md`](dev_journal.md) entry.
7. **Commit** — Conventional Commit on a feature branch.
8. **Verify** — confirm the commit and record its hash in the journal.

A change is not "done" until every box in the
[Definition of Done](docs/development_workflow.md#7-definition-of-done-per-task) is ticked.

## 6. Branching

Branch off an up-to-date `main`, named with a Conventional-Commit-type prefix:

```
feat/job-scout-agent      fix/imap-reconnect      docs/architecture-update
refactor/agent-base       test/resume-coverage    chore/bump-ruff
```

`main` is always releasable and green — never push broken or experimental work to it.

## 7. Commit Messages — Conventional Commits

All commits use [Conventional Commits](docs/development_workflow.md#4-conventional-commits-mandatory):

```
<type>(<scope>): <imperative summary>

<why, wrapped ~72 cols>

Co-Authored-By: <name> <email>
```

Allowed types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`. Examples:

```
feat(agents): add cold email agent with company research pipeline
fix(api): reconnect IMAP watcher after idle timeout
docs(workflow): clarify the commit-hash backfill convention
```

## 8. Pull Requests

- One logical change per PR; keep it focused and reviewable.
- The PR description states **what** changed and **why**, links the relevant sprint/ADR, and confirms
  the Definition of Done is met.
- CI (lint, type-check, tests) must be green before review.
- Include the `dev_journal.md` entry in the PR.
- Be responsive to review feedback; we review for correctness, architecture fit, and the Quality Rules.

## 9. Writing an ADR

If your change introduces or replaces a framework/datastore/service, sets a cross-cutting pattern, or
deviates from `PLAN.md`, add an [Architecture Decision Record](docs/decision_log.md) using the
template there. Routine feature work that follows existing patterns does not need an ADR.

## 10. Where Help Is Most Welcome

Per [`PLAN.md`](PLAN.md), high-value contribution areas are:

- **New ATS adapters** for the Apply Agent (iCIMS, Ashby, and more).
- **New job-board scrapers** for the Job Scout Agent.
- **Improved prompt templates** for any agent.
- **Mobile app features** (Expo).
- **Documentation** — clarity fixes, examples, the MkDocs site.

Good first issues will be labelled in the issue tracker once it is open.

## 11. Reporting Bugs & Requesting Features

- **Bugs:** open an issue with steps to reproduce, expected vs. actual behavior, logs, and your
  environment. Never paste real credentials, tokens, or full private emails.
- **Features:** open an issue describing the problem and the proposed solution; align it with a phase
  in [`PLAN.md`](PLAN.md) where possible.
- **Security issues:** do **not** open a public issue. Email the maintainer at
  `pandidharan7@gmail.com` with details and we will coordinate a fix.

## 12. Code of Conduct

Be respectful, constructive, and inclusive. Assume good faith, give actionable feedback, and keep
discussion focused on the work. Harassment or discrimination of any kind is not tolerated.

## 13. License

By contributing, you agree that your contributions are licensed under the project's
[MIT License](LICENSE).

---

Thank you for helping build a private, open, AI-native way to job hunt. 🚀
