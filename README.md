<div align="center">

# 🎯 JobHunt AI

**AI-Native, Open Source, Self-Deployable Job Hunting Automation Platform**

An autonomous multi-agent system that searches, applies, tracks, networks, and closes job
opportunities — on autopilot, with human-in-the-loop controls, running privately on your own machine.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Status: Foundation](https://img.shields.io/badge/status-foundation%20(pre--alpha)-orange.svg)](docs/sprint_tracking.md)
[![Python](https://img.shields.io/badge/python-3.12-3776AB.svg)](https://www.python.org/)
[![Backend: FastAPI](https://img.shields.io/badge/backend-FastAPI-009688.svg)](https://fastapi.tiangolo.com/)
[![Agents: LangGraph](https://img.shields.io/badge/agents-LangGraph-1C3C3C.svg)](https://www.langchain.com/langgraph)

</div>

---

> ### ⚠️ Project Status: Foundation Phase (Pre-Alpha)
>
> The complete specification lives in **[`PLAN.md`](PLAN.md)** and the **development operating
> system** (docs, workflow, decision log, journal) is now in place. **Application code has not yet
> been implemented** — the first coding sprint is
> **[S1 — Project Bootstrap](docs/sprint_tracking.md)**. The quick start below describes the
> *target* experience once Phase 1 ships. Follow [`docs/sprint_tracking.md`](docs/sprint_tracking.md)
> for live progress.

## What Is JobHunt AI?

Job hunting is a full-time job in itself — searching across boards, tailoring resumes per role,
writing cold emails, filling out endless forms, following up, networking for referrals, and tracking
dozens of applications at once. JobHunt AI hands that work to a team of autonomous AI agents that:

- 🔍 **Discover** relevant jobs from LinkedIn, Indeed, Glassdoor, Wellfound, and HN "Who's Hiring"
- ✍️ **Tailor** your resume and cover letter per job — *rephrasing, never fabricating*
- 🤖 **Apply** via browser automation across major ATS portals, with your approval before submit
- 📊 **Track** every application through its full lifecycle on a Kanban board
- 📬 **Detect** status changes by reading your inbox and classifying replies
- 🔔 **Notify** you in real time on web, desktop, and mobile
- 🤝 **Reach out** for referrals at target companies
- 💌 **Cold-email** founders with hyper-personalized, researched messages
- 🎉 **Close** the deal — drafting replies and coaching negotiation

## Why It's Different

| Principle | What it means |
|-----------|---------------|
| 🧠 **Agent-First** | Every capability is an autonomous LangGraph agent, not a brittle script. |
| 🔒 **Privacy-First** | All data stays on your machine; LLMs run locally via Ollama by default. |
| ✋ **Human-in-the-Loop** | The agent proposes; you approve before anything is sent or submitted. |
| 📖 **Open Source** | MIT licensed, community-driven, zero vendor lock-in. |
| 🐳 **Self-Hostable** | A single `docker-compose up`. Works offline. |

## Architecture at a Glance

```
Presentation   →  React Web · Tauri Desktop · Expo Mobile · Ntfy Push
API Gateway    →  FastAPI REST · WebSocket Gateway · JWT Auth
Agents         →  Job Scout · Resume Tailor · Apply · Email Parser ·
                  Referral · Cold Email · Deal Closer · Tracker
Task Queue     →  Celery Workers · Redis Broker · Celery Beat
Data + AI      →  PostgreSQL · Qdrant · Redis · MinIO · Ollama
Automation     →  Playwright · Crawl4AI · IMAP/SMTP Watcher
```

Full detail in [`docs/architecture.md`](docs/architecture.md) and [`PLAN.md` §4](PLAN.md).

## Tech Stack

| Layer | Technologies |
|-------|--------------|
| **Backend** | Python 3.12, FastAPI, SQLAlchemy 2.0 (async), Alembic, Pydantic v2, Celery, Redis |
| **AI / Agents** | LangChain, LangGraph, Ollama (`llama3.1:8b/70b`, `nomic-embed-text`), Qdrant |
| **Automation** | Playwright, Crawl4AI, aioimaplib, aiosmtplib |
| **Frontend** | React 19, Vite, TypeScript, Tailwind, shadcn/ui, TanStack Query, Zustand |
| **Desktop / Mobile** | Tauri 2, Expo (React Native), Ntfy |
| **DevOps** | Docker Compose, GitHub Actions, Prometheus, Grafana, Nginx, `uv` |

## Quick Start (Target — Phase 1)

> Requires Docker Desktop, 8 GB+ RAM, and ~50 GB free disk. Available once **S1** lands.

```bash
git clone https://github.com/Pandidharan22/Job-Hunt-AI
cd Job-Hunt-AI
cp .env.example .env          # add your email credentials and preferences
docker-compose up -d
# Open http://localhost:8080
```

See [`PLAN.md` §12](PLAN.md) for the full service list, ports, and environment variables.

## Roadmap

| Phase | Theme | Status |
|-------|-------|--------|
| **0** | Project Governance & Operating System | ✅ Done |
| **1** | Core Foundation & Job Discovery | ⏳ Next |
| **2** | Resume Tailoring + Auto-Apply Engine | ☐ Planned |
| **3** | Email Parsing + Status Tracking + Notifications | ☐ Planned |
| **4** | Referral & Cold Email Outreach | ☐ Planned |
| **5** | Deal Closer + Communication Assistant | ☐ Planned |
| **6** | Production Hardening + Commercial Launch | ☐ Planned |

Live, sprint-level progress: [`docs/sprint_tracking.md`](docs/sprint_tracking.md).

## Documentation

| Document | What's inside |
|----------|---------------|
| [`PLAN.md`](PLAN.md) | The complete master specification — vision, features, schema, agents, API, roadmap. |
| [`docs/project_context.md`](docs/project_context.md) | Canonical context and domain glossary. Start here. |
| [`docs/architecture.md`](docs/architecture.md) | System architecture, layers, Clean Architecture mapping. |
| [`docs/coding_standards.md`](docs/coding_standards.md) | Quality rules and language conventions. |
| [`docs/development_workflow.md`](docs/development_workflow.md) | The mandatory eight-step task workflow + Git rules. |
| [`docs/sprint_tracking.md`](docs/sprint_tracking.md) | Live sprint and task board. |
| [`docs/decision_log.md`](docs/decision_log.md) | Architecture Decision Records (ADRs). |
| [`dev_journal.md`](dev_journal.md) | The complete chronological engineering history. |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to contribute. |
| [`CLAUDE.md`](CLAUDE.md) | Auto-loaded operating brief for Claude Code sessions. |
| [`.claude/README.md`](.claude/README.md) | The Claude Code Operating System (CCOS): session commands, context, and templates. |

## How We Build

This project runs on a documented **development operating system**: every task follows an eight-step
workflow (analyze → implement → validate → test → document → journal → commit → verify), every
significant decision is recorded as an ADR, and every completed task appends an entry to the
engineering journal. See [`docs/development_workflow.md`](docs/development_workflow.md).

Because this repo is built primarily with Claude Code, that workflow is operationalized as the
**Claude Code Operating System** ([`.claude/`](.claude/README.md)) — reusable session commands
(`/prime-context`, `/start-task`, `/review-task`, `/security-review`, `/close-task`, …), context
files, and templates, fronted by the auto-loaded [`CLAUDE.md`](CLAUDE.md) — so every session stays
consistent without re-deriving the rules.

## Contributing

Contributions are welcome — especially new ATS adapters, job-board scrapers, prompt improvements, and
mobile features. Please read [`CONTRIBUTING.md`](CONTRIBUTING.md) first.

## License

[MIT](LICENSE) © 2026 Pandidharan G R ([@Pandidharan22](https://github.com/Pandidharan22))

---

<div align="center">
<sub>Built with Python, FastAPI, LangGraph, Ollama, Playwright — and a lot of coffee. ☕</sub>
</div>
