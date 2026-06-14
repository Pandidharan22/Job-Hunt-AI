# Sprint Tracking — JobHunt AI

> The live board for what is planned, in progress, and done. Update this file as part of **Step 5**
> of the [development workflow](development_workflow.md) whenever a deliverable's status changes.
> Sprint scope and deliverables are derived from [`PLAN.md` §10–§11](../PLAN.md); that file remains
> the authoritative specification.

**Last updated:** 2026-06-14

---

## Legend

| Symbol | Status |
|--------|--------|
| ☐ | Not started |
| ◐ | In progress |
| ☑ | Done |

## Current Focus

- **Phase:** Phase 0 — Project Governance & Operating System
- **Sprint:** **S0 — Foundation Setup** (☑ complete)
- **Next sprint:** **S1 — Project Bootstrap**

> No fixed timeline. Ship each sprint when it is done **and tested**. Quality over speed.

---

## Phase → Sprint Map

| Phase | Theme | Sprints |
|-------|-------|---------|
| **Phase 0** | Project Governance & Operating System | S0 |
| **Phase 1** | Core Foundation & Job Discovery | S1, S2 |
| **Phase 2** | Resume Tailoring + Auto-Apply Engine | S3, S4 |
| **Phase 3** | Email Parsing + Status Tracking + Notifications | S5, S6 |
| **Phase 4** | Referral & Cold Email Outreach | S7, S8 |
| **Phase 5** | Deal Closer + Communication Assistant | S9 |
| **Phase 6** | Production Hardening + Commercial Launch | S10, S11, S12 |

## Master Sprint Board

| Sprint | Name | Phase | Status |
|--------|------|-------|--------|
| **S0** | Foundation Setup (dev operating system) | 0 | ☑ Done |
| **S1** | Project Bootstrap | 1 | ☐ Not started |
| **S2** | Job Scout Agent + React Shell | 1 | ☐ Not started |
| **S3** | Resume Tailor Agent | 2 | ☐ Not started |
| **S4** | Apply Agent | 2 | ☐ Not started |
| **S5** | Email Parser + Status Engine | 3 | ☐ Not started |
| **S6** | Real-time Notifications + Kanban UI | 3 | ☐ Not started |
| **S7** | Referral Agent | 4 | ☐ Not started |
| **S8** | Cold Email Agent | 4 | ☐ Not started |
| **S9** | Deal Closer + Conversation UI | 5 | ☐ Not started |
| **S10** | Desktop (Tauri) + Mobile (Expo) | 6 | ☐ Not started |
| **S11** | Observability + Security | 6 | ☐ Not started |
| **S12** | Commercial Launch | 6 | ☐ Not started |

---

## Sprint Detail

### S0 — Foundation Setup ☑

**Goal:** Establish the project's development operating system before any application code is written.

- [x] `README.md` — public front door and quick start
- [x] `CONTRIBUTING.md` — contribution process
- [x] `docs/project_context.md` — canonical context
- [x] `docs/architecture.md` — architecture & layering
- [x] `docs/coding_standards.md` — quality and language rules
- [x] `docs/development_workflow.md` — the eight-step workflow + Git rules
- [x] `docs/sprint_tracking.md` — this board
- [x] `docs/decision_log.md` — ADR records
- [x] `dev_journal.md` — engineering history + first entry

**Definition of Done:** All governance artifacts exist, are internally consistent, cross-reference
each other and `PLAN.md`, and are committed. ☑

---

### S1 — Project Bootstrap ☐  *(Phase 1)*

**Execution Plan:** [`docs/sprint_s1_execution_plan.md`](sprint_s1_execution_plan.md) (21 tasks,
dependency graph, critical path, risks, review checkpoints, ADRs).

**Goal:** A working, reproducible monorepo with infrastructure, database, and auth.

- [ ] Monorepo initialized with `uv`, `ruff`, `mypy`, `pytest`, GitHub Actions CI
- [ ] `docker-compose.yml` with FastAPI, PostgreSQL, Redis, Celery, Celery Beat, Qdrant, MinIO,
      Flower, Nginx
- [ ] Full database schema implemented with Alembic migrations ([`PLAN.md` §7](../PLAN.md))
- [ ] FastAPI app factory with JWT auth (FastAPI-Users)
- [ ] `.env.example` with all variables from [`PLAN.md` §12.2](../PLAN.md)

**Definition of Done:** All services start with `docker-compose up`; auth works; migrations apply
cleanly.

---

### S2 — Job Scout Agent + React Shell ☐  *(Phase 1)*

**Goal:** The first agent runs end-to-end and surfaces jobs in a UI.

- [ ] Crawl4AI scrapers: LinkedIn, Indeed, Wellfound, HN "Who's Hiring"
- [ ] Qdrant vector store: embed JDs + user profile; semantic similarity scoring
- [ ] Celery Beat: run scout every 6 hours; dedup via URL hash
- [ ] `BaseAgent` + `JobScoutAgent` LangGraph flow ([`PLAN.md` §8.1–8.2](../PLAN.md))
- [ ] React web shell: job feed page + preferences settings page
- [ ] REST API: `/jobs`, `/preferences`
- [ ] User registration, login, profile, preferences CRUD; resume upload → MinIO

**Definition of Done:** System finds and displays relevant jobs automatically; backend coverage ≥ 70%.

---

### S3 — Resume Tailor Agent ☐  *(Phase 2)*

**Goal:** LLM tailors a resume per JD and exports it.

- [ ] Ollama integration with `llama3.1:8b` and `llama3.1:70b`
- [ ] Resume Tailor LangGraph pipeline with iterative ATS-score loop (max 3, target ≥ 80)
- [ ] No-fabrication system prompt included verbatim ([`PLAN.md` §8.3](../PLAN.md))
- [ ] Cover letter generation per role
- [ ] PDF export (WeasyPrint) + DOCX (python-docx)
- [ ] Resume versioning: each version stored in MinIO + PostgreSQL

**Definition of Done:** Tailored resume reaches ATS score ≥ 80 and is stored with full version history.

---

### S4 — Apply Agent ☐  *(Phase 2)*

**Goal:** Playwright submits applications with a human approval gate.

- [ ] ATS detection by URL pattern ([`PLAN.md` §8.4](../PLAN.md))
- [ ] Adapters: LinkedIn Easy Apply, Greenhouse, Lever, Workday
- [ ] Generic LLM-guided fallback for unknown forms
- [ ] Anti-detection: Playwright stealth, random 2–8s delays, proxy support
- [ ] HITL confirmation modal before every submission (WebSocket `approval_required`)
- [ ] Confirmation screenshots → MinIO
- [ ] Kanban board UI + application detail page

**Definition of Done:** Submits to ≥ 3 ATS types; approval gate works; applications appear in Kanban.

---

### S5 — Email Parser + Status Engine ☐  *(Phase 3)*

**Goal:** Detect status changes from the inbox automatically.

- [ ] `aioimaplib` IMAP watcher (Celery Beat every 15 min)
- [ ] LLM email classifier (Rejection | Interview | Offer | Assessment | Info Request | Follow-up)
- [ ] Company/sender domain → application matching
- [ ] Auto status updates + `status_history` events
- [ ] Low-confidence (< 0.7) emails flagged for manual review, not auto-updated
- [ ] Interview invite → auto-generate company research brief

**Definition of Done:** A rejection email yields an automatic status update within 15 minutes.

---

### S6 — Real-time Notifications + Kanban UI ☐  *(Phase 3)*

**Goal:** Push every change everywhere, in real time.

- [ ] FastAPI WebSocket gateway + Redis pub/sub
- [ ] All WebSocket event types ([`PLAN.md` §9.9](../PLAN.md))
- [ ] Self-hosted Ntfy: iOS + Android + desktop toast
- [ ] Notification preference settings (per-type on/off)
- [ ] Kanban drag-and-drop status management; `useWebSocket` live updates
- [ ] Follow-up reminders after N days; weekly email digest (Sunday Beat)

**Definition of Done:** WebSocket push works in an open tab; Ntfy push received on mobile.

---

### S7 — Referral Agent ☐  *(Phase 4)*

**Goal:** Find contacts and run approval-gated referral sequences.

- [ ] LinkedIn employee scraper (Crawl4AI) + contact scoring algorithm ([`PLAN.md` §8.6](../PLAN.md))
- [ ] Personalized referral message generation (LLM)
- [ ] 3-touch sequence (Day 0 / 5 / 10) with auto-scheduling
- [ ] Reply detection via IMAP; notify on reply
- [ ] User approval before each message; outreach tracking UI

**Definition of Done:** Referral sequence runs autonomously with working approval gates.

---

### S8 — Cold Email Agent ☐  *(Phase 4)*

**Goal:** Discover startups, research, and send approval-gated cold emails.

- [ ] Startup discovery: Crunchbase, YC directory, AngelList/Wellfound, Product Hunt
- [ ] Founder email finding: Hunter.io free tier + pattern guessing (MX-verified)
- [ ] Company research pipeline (funding, product, blog, founder bio, recent news)
- [ ] 5-sentence cold email generation ([`PLAN.md` §8.7](../PLAN.md))
- [ ] SMTP sending via `aiosmtplib`; 2-touch sequence (initial + 7-day)
- [ ] Volume cap (`MAX_COLD_EMAILS_PER_DAY`); bounce + unsubscribe handling
- [ ] Reply → conversation thread created → user notified

**Definition of Done:** A campaign sends, detects a reply, creates a conversation, and notifies the user.

---

### S9 — Deal Closer + Conversation UI ☐  *(Phase 5)*

**Goal:** Become the user's co-pilot for closing warm conversations.

- [ ] Intent classification on inbound messages ([`PLAN.md` §8.8](../PLAN.md))
- [ ] Deal status pipeline (Replied → Interested → Call Scheduled → Offer → Negotiating → Closed)
- [ ] Draft reply generation with full thread context
- [ ] In-app conversation UI: thread + AI draft panel side-by-side; edit → approve → send
- [ ] Negotiation coach: market-rate lookup, counter-offer suggestions
- [ ] Closed Won notification (urgent push + email); deal analytics

**Definition of Done:** User can view a reply, see/edit an AI draft, and send — all in-app; Closed Won
fires correct notifications.

---

### S10 — Desktop (Tauri) + Mobile (Expo) ☐  *(Phase 6)*

**Goal:** Native apps and one-click install.

- [ ] Tauri 2 desktop app: code-signed Windows/macOS/Linux installers; auto-update
- [ ] Expo mobile app: iOS + Android with push notifications
- [ ] One-click installer (`bash` + PowerShell) + interactive first-run setup wizard

**Definition of Done:** Tauri app installs and auto-connects to the local backend; mobile push works.

---

### S11 — Observability + Security ☐  *(Phase 6)*

**Goal:** Make it bulletproof.

- [ ] Prometheus metrics on all agents + endpoints; Grafana dashboards
- [ ] Rate limiting + circuit breakers on all scrapers
- [ ] Retry with exponential backoff on all external calls
- [ ] Security audit (OWASP Top 10); credential encryption review (OS keychain)
- [ ] Backend test coverage ≥ 90%

**Definition of Done:** Dashboards live; audit passed; coverage ≥ 90%.

---

### S12 — Commercial Launch ☐  *(Phase 6)*

**Goal:** Multi-user SaaS and public launch.

- [ ] Multi-user SaaS mode: user isolation, per-plan quotas
- [ ] Stripe billing integration
- [ ] MkDocs documentation site (self-host guide, API reference, agent docs, FAQ)
- [ ] GitHub release pipeline: automated Docker image publishing to GHCR
- [ ] Product Hunt launch kit + "Show HN" post; README with screenshots + demo GIF

**Definition of Done:** `docker-compose up` works on a fresh machine with zero manual steps;
documentation site live.

---

## Task Tracking Template

When breaking a sprint into tasks, track them here or in the issue tracker using this shape:

```
- [ ] <task name> — <one-line scope>  (owner, status, branch, commit)
```

Each completed task must also produce a [`dev_journal.md`](../dev_journal.md) entry per the
[development workflow](development_workflow.md).

## Change Log (this document)

| Date | Change |
|------|--------|
| 2026-06-14 | Created board; S0 marked done; S1–S12 seeded from `PLAN.md`. |
