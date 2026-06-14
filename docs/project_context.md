# Project Context — JobHunt AI

> **Read this first.** This document is the canonical, human- and agent-readable summary of *what*
> JobHunt AI is, *why* it exists, and *how* we think about it. It is the entry point for every
> contributor (human or AI) before touching the codebase.

**Source-of-truth hierarchy**

1. [`PLAN.md`](../PLAN.md) — the complete master specification (vision, features, schema, agents,
   API, roadmap). When this document and `PLAN.md` disagree, **`PLAN.md` wins** and this document
   must be corrected.
2. This document — a navigable summary and onboarding context layer over `PLAN.md`.
3. The other files in [`docs/`](.) — deep detail on architecture, standards, workflow, and history.

---

## 1. Elevator Pitch

**JobHunt AI** is an AI-native, open-source, self-deployable platform that automates the entire job
hunting pipeline end-to-end. It is a multi-agent system that **discovers** relevant jobs, **tailors**
your resume and cover letter per role, **applies** via browser automation, **tracks** every
application's lifecycle, **notifies** you of every change, **reaches out** for referrals,
**cold-emails** founders, and **assists** you in closing the deal — autonomously, with
human-in-the-loop controls, running privately on your own machine.

## 2. The Problem

Job hunting is itself a full-time job: searching across boards, tailoring resumes per role, writing
cold emails, filling out forms, following up, networking for referrals, and tracking dozens of
applications simultaneously. It is repetitive, time-consuming, and deeply inefficient.

## 3. The Solution

A pipeline of autonomous LLM agents — not scripts — each owning one stage of the job hunt, connected
by an event-driven task queue and a shared database, all runnable with a single `docker-compose up`
on a user's own hardware using local LLMs.

## 4. Design Philosophy (Non-Negotiable Pillars)

| Pillar | What it means in practice |
|--------|---------------------------|
| **Agent-First** | Every major function is an autonomous LangGraph agent with tools, memory, and a decision loop — not a linear script. |
| **Privacy-First** | All data stays on the user's machine. LLM inference runs locally via Ollama by default. No third-party data sharing. |
| **Human-in-the-Loop (HITL)** | The agent *proposes*; the human *approves* before anything is sent or submitted. Outbound actions are gated by default. |
| **Open Source** | MIT licensed, community-driven, zero vendor lock-in. |
| **Self-Hostable** | A single `docker-compose up` on any PC or server. Works offline. |

## 5. Target Users

- **Developers & power users** — self-host the full stack for free, keep all data private.
- **Active job seekers** — automate the repetitive 80% of the hunt, focus on interviews and closing.
- **Recruiters & career coaches** — manage multiple candidate profiles (commercial tiers).
- **Contributors** — extend ATS adapters, job-board scrapers, prompts, and the mobile app.

## 6. Core Capabilities (Summary)

Full detail in [`PLAN.md` §2](../PLAN.md). The platform is organized into nine capability areas:

1. **Job Discovery & Matching** — multi-source scraping, semantic matching via vector embeddings,
   relevance scoring, deduplication.
2. **Resume Tailoring & Cover Letters** — JD-aware rewriting that *rephrases, never fabricates*; ATS
   scoring with iterative improvement; PDF/DOCX export; full version history.
3. **Autonomous Application Engine** — Playwright automation across LinkedIn Easy Apply, Greenhouse,
   Lever, Workday, iCIMS, Ashby, and generic forms, with HITL confirmation and proof screenshots.
4. **Application Tracking** — Kanban lifecycle, full status history, IMAP email watcher with LLM
   classification, deadline and follow-up reminders.
5. **Real-Time Notifications** — WebSocket push, self-hosted Ntfy mobile/desktop push, weekly digest.
6. **Referral Outreach** — contact discovery and scoring, personalized multi-touch sequences.
7. **Cold Email Engine** — startup/founder discovery, research-driven personalization, volume caps
   for legal/ethical compliance.
8. **Deal Closer & Communication Assistant** — intent analysis, AI-drafted replies, negotiation
   coaching, deal pipeline.
9. **Analytics Dashboard** — response/interview/offer rates, time-to-response, pattern insights.

## 7. The Agents at a Glance

Eight autonomous agents form the orchestration layer. Detail in [`PLAN.md` §8](../PLAN.md) and
[`architecture.md`](architecture.md).

| Agent | File | LLM | Trigger | Responsibility |
|-------|------|-----|---------|----------------|
| **Job Scout** | `job_scout.py` | `llama3.1:8b` | Beat / 6h | Discover, structure, embed, score, and persist new jobs. |
| **Resume Tailor** | `resume_tailor.py` | `llama3.1:70b` | Before each apply | Rewrite resume to JD, ATS-score loop, generate cover letter, export. |
| **Apply Agent** | `apply_agent.py` | `llama3.1:8b` | `apply_to_job` task | Drive Playwright across ATS portals; HITL approval before submit. |
| **Email Parser** | `email_parser.py` | `llama3.1:8b` | Beat / 15m | Watch IMAP, classify emails, auto-update application status. |
| **Referral** | `referral_agent.py` | `llama3.1:70b` | After each apply | Find/score contacts, draft personalized referral sequences. |
| **Cold Email** | `cold_email.py` | `llama3.1:70b` | Manual / weekly | Discover startups, research, write 5-sentence cold emails. |
| **Deal Closer** | `deal_closer.py` | `llama3.1:70b` | On inbound reply | Classify intent, draft replies, coach negotiation. |
| **Tracker** | `tracker.py` | — | Analytics tasks | Compute analytics and pattern insights. |

## 8. Engineering Principles

| Principle | Implementation |
|-----------|---------------|
| **Agent-First** | Every major function is a LangGraph agent with tools, memory, and a decision loop. |
| **Event-Driven** | Celery + Redis message queue; every action is an async task. |
| **Testable** | 90%+ coverage target; pytest for backend, Playwright for E2E. |
| **Secure** | Credentials in encrypted local vault; no third-party data sharing. |
| **Observable** | Prometheus metrics + Grafana dashboards + structured logging (Loguru). |
| **Self-Hostable** | Single `docker-compose up`; works offline with local LLMs. |
| **Type-Safe** | Full mypy strict mode; Pydantic v2 everywhere. |
| **Documented** | OpenAPI auto-docs; MkDocs site; inline docstrings; ADR records. |

## 9. Domain Glossary

| Term | Meaning |
|------|---------|
| **ADR** | Architecture Decision Record — a dated, immutable record of one significant technical decision (see [`decision_log.md`](decision_log.md)). |
| **Agent** | An autonomous LangGraph state machine with tools, an LLM, and a decision loop; extends `BaseAgent`. |
| **ATS** | Applicant Tracking System — the portal an employer uses to receive applications (Greenhouse, Lever, Workday, etc.). |
| **ATS Score** | Keyword-match percentage (0–100) between a tailored resume and a job description; target ≥ 80. |
| **Beat** | Celery Beat — the cron-style scheduler that triggers recurring agent runs. |
| **Cover Letter** | A per-role, context-aware letter generated by the Resume Tailor Agent. |
| **HITL** | Human-in-the-Loop — the approval gate a human must pass before an outbound action executes. |
| **JD** | Job Description — the raw and parsed text of a job posting. |
| **Relevance Score** | Semantic similarity (0–100) between a job and the user's profile, computed via Qdrant. |
| **Resume Version** | A tailored, immutable snapshot of a resume linked to one application. |
| **Sprint** | A feature-complete slice of work (see [`sprint_tracking.md`](sprint_tracking.md)); S0–S12. |
| **Vector Store** | Qdrant — holds embeddings of JDs and the user profile for semantic matching. |

## 10. Current Status

- **Phase:** Phase 0 — Project Governance & Operating System.
- **Sprint:** S0 — Foundation Setup (development operating system).
- **State:** Specification (`PLAN.md`) complete; development operating system being established.
  **Application implementation has not yet started.** The first coding sprint is
  **S1 — Project Bootstrap** ([`sprint_tracking.md`](sprint_tracking.md)).

## 11. Related Documents

| Document | Purpose |
|----------|---------|
| [`../PLAN.md`](../PLAN.md) | Complete master specification. |
| [`../README.md`](../README.md) | Public front door and quick start. |
| [`../CONTRIBUTING.md`](../CONTRIBUTING.md) | How to contribute. |
| [`architecture.md`](architecture.md) | System architecture, layers, and Clean Architecture mapping. |
| [`coding_standards.md`](coding_standards.md) | Code quality rules and language conventions. |
| [`development_workflow.md`](development_workflow.md) | The mandatory per-task workflow and Git rules. |
| [`sprint_tracking.md`](sprint_tracking.md) | Live sprint and task board. |
| [`decision_log.md`](decision_log.md) | Architecture Decision Records. |
| [`../dev_journal.md`](../dev_journal.md) | Chronological engineering history. |
