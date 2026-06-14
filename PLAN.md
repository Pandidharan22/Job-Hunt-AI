# 🎯 JobHunt AI — Complete Project Plan

> **AI-Native, Open Source, Self-Deployable Job Hunting Automation Platform**
>
> Autonomous multi-agent system that searches, applies, tracks, networks, and closes job opportunities — entirely on autopilot with human-in-the-loop controls.

---

## 📋 Table of Contents

1. [Product Vision](#1-product-vision)
2. [Core Features](#2-core-features)
3. [Engineering Principles](#3-engineering-principles)
4. [System Architecture](#4-system-architecture)
5. [Technology Stack](#5-technology-stack)
6. [Project File Structure](#6-project-file-structure)
7. [Database Schema](#7-database-schema)
8. [AI Agent Design](#8-ai-agent-design)
9. [API Endpoints](#9-api-endpoints)
10. [Phase Roadmap](#10-phase-roadmap)
11. [Sprint Breakdown](#11-sprint-breakdown)
12. [Deployment Strategy](#12-deployment-strategy)
13. [Commercial Strategy](#13-commercial-strategy)
14. [Risks & Mitigations](#14-risks--mitigations)
15. [Definition of Done](#15-definition-of-done)
16. [Claude Code Prompting Guide](#16-claude-code-prompting-guide)

---

## 1. Product Vision

### Problem

Job hunting is a full-time job itself — searching across boards, tailoring resumes per role, writing cold emails, filling out forms, following up, networking for referrals, tracking dozens of applications simultaneously. It's repetitive, time-consuming, and deeply inefficient.

### Solution

**JobHunt AI** is an autonomous, multi-agent platform that handles the entire job hunting pipeline end-to-end:

- **Discovers** relevant jobs from multiple sources based on your preferences
- **Tailors** your resume and cover letter per job description using a local LLM
- **Applies** to roles via browser automation (Playwright)
- **Tracks** every application through its full lifecycle
- **Notifies** you on every status change (rejection, interview, offer)
- **Reaches out** for referrals at target companies automatically
- **Cold emails** startup founders and notifies you when they respond
- **Assists** you in closing the deal when an opportunity arises

### Design Philosophy

- **Agent-First:** Every major function is an autonomous LLM agent, not a script
- **Privacy-First:** All data stays on your machine; local LLMs via Ollama
- **Human-in-the-Loop:** Agent proposes, human approves before any send/submit
- **Open Source:** MIT licensed, community-driven, zero vendor lock-in
- **Self-Hostable:** Single `docker-compose up` on any PC or server

---

## 2. Core Features

### 2.1 Job Discovery & Matching

- Scrapes LinkedIn, Indeed, Glassdoor, Wellfound (AngelList), Hacker News "Who's Hiring"
- Semantic matching using vector embeddings (Qdrant) against user profile
- User preference model: role title, tech stack, location, salary range, remote/hybrid/onsite, company size, industry
- Deduplication via URL hash; freshness TTL = 24 hours
- Relevance scoring displayed in UI (0–100)

### 2.2 Resume Tailoring & Cover Letter Generation

- LLM reads the full JD and extracts required/preferred keywords
- Rewrites resume bullet points to match JD language — **never fabricates, only rephrases**
- ATS score check (keyword match %) after each rewrite iteration
- Iterates until ATS score ≥ configured threshold (default: 80)
- Generates a unique, context-aware cover letter per role
- Exports tailored resume as PDF + DOCX
- Stores every version linked to its job application (full version history)

### 2.3 Autonomous Application Engine

- Browser automation via Playwright for:
  - LinkedIn Easy Apply
  - Greenhouse ATS
  - Lever ATS
  - Workday
  - iCIMS
  - Ashby
  - Generic HTML forms (LLM-guided)
- Human-in-the-loop confirmation: agent sends a preview to user before final submit
- Screenshots application confirmation screen as proof
- Stores application ID, submission timestamp, portal used

### 2.4 Application Tracking

- Kanban board: `Wishlist → Applied → Screening → Interview → Assessment → Offer → Closed`
- Full status history with timestamps per application
- IMAP email watcher monitors inbox for status updates
- LLM classifies incoming emails: `Rejection | Interview Invite | Offer | Assessment | Info Request | Follow-up | Other`
- Auto-updates application status on classification
- Low-confidence emails flagged for manual user review
- Deadline reminders + follow-up nudges if no response in N days (configurable)
- On interview invite: auto-generates company research brief (recent news, tech stack, culture)

### 2.5 Real-Time Notifications

- WebSocket gateway pushes status changes to open web/desktop app instantly
- Self-hosted **Ntfy** push notifications → iOS, Android, desktop toast
- Notification types: New jobs found, Application submitted, Status changed, Reply received, Deal closed
- Weekly email digest: summary of all activity
- Per-notification-type on/off toggle in settings

### 2.6 Referral Outreach

- After each application, Referral Agent finds employees at the target company via LinkedIn scraping
- Contact scoring algorithm: alumni > mutual connections > same-title engineers > general employees
- LLM drafts hyper-personalized referral request per contact (name, role, shared context)
- 3-touch sequence: initial message → 5-day follow-up → 10-day final nudge
- Sends via email or LinkedIn DM
- Tracks reply status; notifies user when a referral contact responds
- User approval required before each message is sent (configurable)

### 2.7 Cold Email Engine (Startups & Founders)

- Discovers startups from: Crunchbase, YC company directory, AngelList/Wellfound, Product Hunt
- Finds founder/hiring manager emails via Hunter.io API (free tier) + email pattern guessing
- Company research pipeline: recent funding news, product page, founder LinkedIn bio, blog posts
- LLM writes hyper-personalized 5-sentence cold email: Hook → Relevance → Value → Ask → CTA
- Email sending via user's own SMTP (Gmail, Outlook, custom domain)
- 2-touch sequence: initial email → 7-day follow-up if no reply
- Bounce detection and blacklist management
- Reply detection via IMAP; immediately notifies user
- Volume cap: configurable (default ≤ 20 cold emails/day) for legal/ethical compliance

### 2.8 Deal Closer & Communication Assistant

- Activated when a promising reply arrives (startup founder, recruiter, hiring manager)
- Reads full email thread, assesses intent: `Interested | Scheduling | Offer | Negotiating | Ghosted | Closed Won | Closed Lost`
- Drafts contextual reply; user edits and approves before sending
- Negotiation coach: if salary/equity discussed, suggests counter-offer language and ranges
- Deal status pipeline: `Replied → Interested → Call Scheduled → Offer → Negotiating → Closed`
- Notifies user: 🎉 "Offer received from {Company}! Review and respond." on deal close
- In-app conversation view: full email thread + AI draft panel side-by-side

### 2.9 Analytics Dashboard

- Response rate, interview conversion rate, offer rate
- Average time-to-response per company type/size
- Best performing resume versions (which tailoring approaches get more replies)
- Best performing cold email subject lines
- Weekly activity digest
- Pattern insights: "You get 3x more replies applying Monday morning vs Friday"

---

## 3. Engineering Principles

| Principle | Implementation |
|-----------|---------------|
| **Agent-First** | Every major function is a LangGraph agent with tools, memory, and decision loop |
| **Event-Driven** | Celery + Redis message queue; every action is an async task |
| **Testable** | 90%+ test coverage target; pytest for backend, Playwright for E2E |
| **Secure** | Credentials in encrypted local vault; no third-party data sharing |
| **Observable** | Prometheus metrics + Grafana dashboards + structured logging (Loguru) |
| **Self-Hostable** | Single `docker-compose up`; works offline with local LLMs |
| **Type-Safe** | Full mypy strict mode; Pydantic v2 everywhere |
| **Documented** | OpenAPI auto-docs; MkDocs site; inline docstrings; ADR records |

---

## 4. System Architecture

### 4.1 High-Level Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                        │
│  React Web UI  │  Tauri Desktop App  │  Expo Mobile App     │
│                │  Ntfy Notifications │                      │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP / WebSocket
┌──────────────────────────▼──────────────────────────────────┐
│                  API GATEWAY LAYER                           │
│  FastAPI REST API  │  WebSocket Gateway  │  JWT Auth        │
│  OpenAPI Docs      │  Rate Limiter       │  CORS            │
└──────────────────────────┬──────────────────────────────────┘
                           │ Python function calls
┌──────────────────────────▼──────────────────────────────────┐
│              AGENT ORCHESTRATION LAYER                       │
│  Job Scout  │  Resume Tailor  │  Apply Agent  │  Email Parser│
│  Referral   │  Cold Email     │  Deal Closer  │  Tracker     │
└──────────────────────────┬──────────────────────────────────┘
                           │ Task dispatch
┌──────────────────────────▼──────────────────────────────────┐
│                TASK / MESSAGE QUEUE                          │
│  Celery Workers  │  Redis Broker  │  Celery Beat (cron)     │
│  Flower Monitor  │                │                         │
└──────────────────────────┬──────────────────────────────────┘
                           │ DB / AI calls
┌──────────────────────────▼──────────────────────────────────┐
│               DATA + AI LAYER                                │
│  PostgreSQL 16  │  Qdrant (vectors)  │  Redis (cache)       │
│  MinIO (files)  │  Ollama (LLM)      │  Vault (secrets)     │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP / browser
┌──────────────────────────▼──────────────────────────────────┐
│          SCRAPING / BROWSER AUTOMATION LAYER                 │
│  Playwright  │  Crawl4AI  │  Proxy Rotator  │  IMAP Watcher │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Job Application Data Flow (End-to-End)

```
1.  Celery Beat triggers Job Scout Agent every 6 hours
2.  Crawl4AI scrapes LinkedIn / Indeed / Wellfound / HN
3.  Raw job data → LLM extracts structured fields (title, company, salary, stack, JD text)
4.  JD text embedded (sentence-transformers) → stored in Qdrant
5.  Semantic similarity scored against user profile embedding
6.  Top matches (score > threshold) saved to PostgreSQL jobs table
7.  WebSocket push: "12 new jobs found" → UI updates
8.  User reviews → clicks "Apply" (manual) OR auto-apply runs on schedule
9.  Resume Tailor Agent: reads JD → rewrites resume → ATS score check → generates cover letter
10. Tailored resume PDF exported → stored in MinIO
11. Apply Agent: Playwright opens job URL → detects ATS type → fills form → uploads resume
12. Confirmation modal sent to user via WebSocket → user approves
13. Apply Agent submits form → screenshots confirmation → stores in MinIO
14. Application record created in PostgreSQL: status = "Applied"
15. Referral Agent triggered: finds employees at company → drafts messages → queues outreach
16. Celery Beat runs Email Parser every 15 minutes: checks IMAP for new emails
17. LLM classifies email → matches to application by company name / sender domain
18. Status updated in PostgreSQL: "Interview Invite" → StatusHistory event created
19. WebSocket push to UI + Ntfy push notification to mobile/desktop
20. Deal Closer Agent activated → drafts reply → notifies user to review + approve
21. User approves → email sent → deal tracked until Closed Won/Lost
```

### 4.3 Agent Communication Pattern

```
FastAPI Route
     │
     ▼
Celery Task (async)
     │
     ▼
Agent.run(context: dict)
     │
     ├── LangGraph State Machine
     │        ├── Node: call_tool(tool_name, args)
     │        ├── Node: llm_decision(prompt, state)
     │        ├── Edge: conditional routing
     │        └── Node: persist_result(db)
     │
     └── Returns: AgentResult(status, data, next_action)
```

---

## 5. Technology Stack

### 5.1 Backend (Python / FastAPI)

| Package | Version | Purpose |
|---------|---------|---------|
| `fastapi` | latest | REST API framework — async, Pydantic-native, OpenAPI auto-docs |
| `uvicorn[standard]` | latest | ASGI server with uvloop |
| `sqlalchemy[asyncio]` | 2.x | Async ORM with PostgreSQL |
| `alembic` | latest | Database migration management |
| `pydantic` | v2 | Data validation, settings management, serialization |
| `pydantic-settings` | latest | Environment-based config with `.env` support |
| `celery[redis]` | latest | Distributed async task queue |
| `redis` | latest | Celery broker + cache + pub/sub |
| `fastapi-users[sqlalchemy]` | latest | JWT auth, OAuth2, user management |
| `python-jose[cryptography]` | latest | JWT token handling |
| `passlib[bcrypt]` | latest | Password hashing |
| `httpx` | latest | Async HTTP client (for API calls, webhooks) |
| `loguru` | latest | Structured logging with JSON output |
| `pytest` | latest | Test framework |
| `pytest-asyncio` | latest | Async test support |
| `pytest-cov` | latest | Coverage reporting |
| `ruff` | latest | Linter + formatter (replaces flake8, black, isort) |
| `mypy` | latest | Static type checking |

### 5.2 AI / Agent Layer

| Package | Version | Purpose |
|---------|---------|---------|
| `langchain` | latest | LLM tool abstractions, prompt templates, chains |
| `langgraph` | latest | Stateful multi-step agent workflows as graphs |
| `langchain-community` | latest | Ollama integration, community tools |
| `ollama` | latest | Python client for local Ollama LLM server |
| `sentence-transformers` | latest | Text embeddings for semantic job matching |
| `qdrant-client` | latest | Vector DB client for semantic search |
| `tiktoken` | latest | Token counting for prompt management |

**LLM Models (via Ollama):**
- `llama3.1:8b` — Fast tasks: classification, matching, form field reasoning
- `llama3.1:70b` — Quality tasks: resume rewriting, cold emails, deal closing
- `nomic-embed-text` — Text embeddings (fast, small, purpose-built)

### 5.3 Browser Automation & Scraping

| Package | Version | Purpose |
|---------|---------|---------|
| `playwright` | latest | Browser automation for job applications |
| `crawl4ai` | latest | LLM-native web crawler for job board scraping |
| `beautifulsoup4` | latest | HTML parsing fallback |
| `aioimaplib` | latest | Async IMAP for email monitoring |
| `aiosmtplib` | latest | Async SMTP for email sending |

### 5.4 Storage & Infrastructure

| Tool | Purpose |
|------|---------|
| `PostgreSQL 16` | Primary relational database |
| `Redis 7` | Celery broker, session cache, rate limiting, pub/sub |
| `Qdrant` | Vector database for semantic job matching |
| `MinIO` | Self-hosted S3-compatible object storage (resumes, screenshots) |
| `minio` (Python SDK) | MinIO client |

### 5.5 Frontend (React)

| Package | Purpose |
|---------|---------|
| `react` v19 | UI framework |
| `vite` | Build tool |
| `typescript` | Type safety |
| `tailwindcss` | Utility CSS |
| `shadcn/ui` | Component library (Kanban, tables, modals, forms) |
| `@tanstack/react-query` v5 | Server state management + caching |
| `@tanstack/react-table` | Data tables for job listings |
| `zustand` | Global client state management |
| `react-beautiful-dnd` | Kanban drag-and-drop |
| `recharts` | Analytics charts |
| `react-router-dom` v6 | Client-side routing |
| `lucide-react` | Icon library |
| `axios` | HTTP client with interceptors |

### 5.6 Desktop & Mobile

| Tool | Purpose |
|------|---------|
| `Tauri 2` | Wraps React web app into native desktop app (Windows/macOS/Linux) |
| `Expo` (React Native) | iOS + Android mobile app with push notifications |
| `Ntfy` | Self-hosted push notification server (iOS + Android + Desktop) |

### 5.7 DevOps & Observability

| Tool | Purpose |
|------|---------|
| `Docker + Docker Compose` | Containerization; single-command local deploy |
| `GitHub Actions` | CI/CD: test, lint, build, publish Docker images |
| `Prometheus` | Metrics collection from all services |
| `Grafana` | Metrics dashboards + alerting |
| `Flower` | Celery task monitoring dashboard |
| `Nginx` | Reverse proxy, SSL termination, static file serving |
| `uv` | Ultra-fast Python package manager (replaces pip/poetry) |

---

## 6. Project File Structure

```
jobhunt-ai/
│
├── apps/                          # Frontend applications
│   ├── web/                       # React web frontend
│   │   ├── src/
│   │   │   ├── components/        # Reusable UI components
│   │   │   │   ├── ui/            # shadcn/ui base components
│   │   │   │   ├── KanbanBoard/   # Application tracker board
│   │   │   │   ├── JobCard/       # Job listing card
│   │   │   │   ├── ResumeEditor/  # Resume preview + diff view
│   │   │   │   ├── EmailThread/   # Conversation + AI draft panel
│   │   │   │   └── Notification/  # Toast + notification center
│   │   │   ├── pages/
│   │   │   │   ├── Dashboard.tsx      # Overview: stats + recent activity
│   │   │   │   ├── Jobs.tsx           # Job feed with filters
│   │   │   │   ├── Applications.tsx   # Kanban tracker
│   │   │   │   ├── Outreach.tsx       # Cold emails + referrals
│   │   │   │   ├── Conversations.tsx  # Deal closer + email threads
│   │   │   │   ├── Analytics.tsx      # Charts + insights
│   │   │   │   └── Settings.tsx       # Preferences + credentials
│   │   │   ├── hooks/
│   │   │   │   ├── useJobs.ts
│   │   │   │   ├── useApplications.ts
│   │   │   │   ├── useWebSocket.ts
│   │   │   │   └── useNotifications.ts
│   │   │   ├── stores/
│   │   │   │   ├── authStore.ts
│   │   │   │   └── uiStore.ts
│   │   │   ├── lib/
│   │   │   │   ├── api.ts         # Axios instance + interceptors
│   │   │   │   ├── ws.ts          # WebSocket client
│   │   │   │   └── utils.ts
│   │   │   ├── App.tsx
│   │   │   └── main.tsx
│   │   ├── package.json
│   │   ├── vite.config.ts
│   │   └── tsconfig.json
│   │
│   ├── desktop/                   # Tauri 2 desktop app
│   │   ├── src-tauri/
│   │   │   ├── src/
│   │   │   │   └── main.rs        # Tauri entry + native plugins
│   │   │   ├── Cargo.toml
│   │   │   └── tauri.conf.json
│   │   └── package.json
│   │
│   └── mobile/                    # Expo React Native
│       ├── app/                   # Expo Router pages
│       │   ├── (tabs)/
│       │   │   ├── index.tsx      # Dashboard
│       │   │   ├── applications.tsx
│       │   │   └── notifications.tsx
│       │   └── _layout.tsx
│       ├── components/
│       ├── app.json
│       └── package.json
│
├── backend/                       # Python FastAPI backend
│   ├── app/
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── __init__.py
│   │   │       ├── router.py          # Aggregates all routers
│   │   │       ├── auth.py            # POST /auth/login, /auth/register
│   │   │       ├── jobs.py            # GET /jobs, POST /jobs/search, GET /jobs/{id}
│   │   │       ├── applications.py    # CRUD /applications, PATCH /applications/{id}/status
│   │   │       ├── profile.py         # GET/PUT /profile, POST /profile/resume
│   │   │       ├── preferences.py     # GET/PUT /preferences
│   │   │       ├── outreach.py        # /outreach/cold-emails, /outreach/referrals
│   │   │       ├── conversations.py   # GET /conversations, POST /conversations/{id}/reply
│   │   │       ├── analytics.py       # GET /analytics/summary, /analytics/weekly
│   │   │       ├── notifications.py   # GET /notifications, PUT /notifications/settings
│   │   │       └── ws.py              # WebSocket: ws://host/api/v1/ws
│   │   │
│   │   ├── agents/                    # LangGraph AI agents
│   │   │   ├── __init__.py
│   │   │   ├── base_agent.py          # BaseAgent class: LLM client, tools, logger, run()
│   │   │   ├── job_scout.py           # JobScoutAgent: discovers + scores jobs
│   │   │   ├── resume_tailor.py       # ResumeTailorAgent: rewrites resume + cover letter
│   │   │   ├── apply_agent.py         # ApplyAgent: Playwright form automation
│   │   │   ├── email_parser.py        # EmailParserAgent: IMAP watcher + classifier
│   │   │   ├── referral_agent.py      # ReferralAgent: find contacts + draft messages
│   │   │   ├── cold_email.py          # ColdEmailAgent: startup discovery + email writing
│   │   │   ├── deal_closer.py         # DealCloserAgent: thread analysis + reply drafting
│   │   │   └── tracker.py             # TrackerAgent: analytics computation
│   │   │
│   │   ├── tasks/                     # Celery task definitions
│   │   │   ├── __init__.py
│   │   │   ├── scout_tasks.py         # run_job_scout, run_semantic_match
│   │   │   ├── apply_tasks.py         # apply_to_job, confirm_application
│   │   │   ├── resume_tasks.py        # tailor_resume, generate_cover_letter
│   │   │   ├── email_tasks.py         # check_email_inbox, classify_email
│   │   │   ├── outreach_tasks.py      # run_referral_outreach, run_cold_email_campaign
│   │   │   ├── notify_tasks.py        # send_push_notification, send_weekly_digest
│   │   │   └── analytics_tasks.py     # compute_weekly_analytics
│   │   │
│   │   ├── models/                    # SQLAlchemy ORM models
│   │   │   ├── __init__.py
│   │   │   ├── user.py                # User, UserPreferences
│   │   │   ├── resume.py              # Resume, ResumeVersion
│   │   │   ├── job.py                 # Job, Company
│   │   │   ├── application.py         # Application, StatusHistory
│   │   │   ├── outreach.py            # ColdEmail, Referral, Contact
│   │   │   ├── conversation.py        # Conversation, Message, DealStatus
│   │   │   └── notification.py        # Notification, NotificationSettings
│   │   │
│   │   ├── schemas/                   # Pydantic v2 request/response schemas
│   │   │   ├── __init__.py
│   │   │   ├── job.py
│   │   │   ├── application.py
│   │   │   ├── user.py
│   │   │   ├── outreach.py
│   │   │   └── conversation.py
│   │   │
│   │   ├── services/                  # External integrations (no business logic)
│   │   │   ├── __init__.py
│   │   │   ├── scraper.py             # Crawl4AI wrapper: scrape_jobs(url) → List[JobData]
│   │   │   ├── browser.py             # Playwright wrapper: fill_form(), submit(), screenshot()
│   │   │   ├── llm.py                 # Ollama client: complete(), embed(), chat()
│   │   │   ├── vector_store.py        # Qdrant: upsert_job(), search_similar(), delete()
│   │   │   ├── email.py               # IMAP: watch_inbox(); SMTP: send_email()
│   │   │   ├── storage.py             # MinIO: upload_file(), get_url(), delete_file()
│   │   │   ├── notify.py              # Ntfy: send_push(title, body, topic)
│   │   │   └── pdf.py                 # WeasyPrint: render resume HTML → PDF
│   │   │
│   │   └── core/                      # App infrastructure
│   │       ├── __init__.py
│   │       ├── config.py              # Pydantic Settings: all env vars
│   │       ├── database.py            # Async SQLAlchemy engine + session factory
│   │       ├── celery.py              # Celery app factory + Beat schedule
│   │       ├── security.py            # JWT utils, password hashing
│   │       ├── exceptions.py          # Custom HTTP exceptions
│   │       ├── middleware.py          # Logging, timing, CORS middleware
│   │       └── websocket.py           # WebSocket connection manager + broadcast
│   │
│   ├── alembic/                       # Database migrations
│   │   ├── versions/
│   │   │   └── 0001_initial_schema.py
│   │   └── env.py
│   │
│   ├── tests/
│   │   ├── unit/
│   │   │   ├── test_agents/
│   │   │   ├── test_services/
│   │   │   └── test_models/
│   │   ├── integration/
│   │   │   ├── test_api/
│   │   │   └── test_celery/
│   │   ├── e2e/
│   │   │   └── test_apply_flow.py     # Playwright E2E test
│   │   └── conftest.py                # Pytest fixtures, test DB setup
│   │
│   ├── main.py                        # FastAPI app factory: create_app()
│   ├── pyproject.toml                 # uv + ruff + mypy + pytest config
│   ├── .env.example                   # Template for all required env vars
│   └── Dockerfile
│
├── infra/                             # Infrastructure configuration
│   ├── docker-compose.yml             # Full local development stack
│   ├── docker-compose.prod.yml        # Production overrides
│   ├── nginx/
│   │   └── nginx.conf                 # Reverse proxy + SSL config
│   ├── prometheus/
│   │   └── prometheus.yml             # Scrape config for all services
│   └── grafana/
│       └── dashboards/
│           ├── agents.json            # Agent activity dashboard
│           └── applications.json      # Application funnel dashboard
│
├── docs/                              # MkDocs documentation
│   ├── index.md
│   ├── getting-started.md
│   ├── self-hosting.md
│   ├── configuration.md
│   ├── api-reference.md
│   └── agents.md
│
├── scripts/
│   ├── install.sh                     # One-click installer for Linux/macOS
│   ├── install.ps1                    # One-click installer for Windows
│   └── seed_data.py                   # Dev seed data script
│
├── .github/
│   └── workflows/
│       ├── ci.yml                     # Run tests on every PR
│       ├── release.yml                # Tag-based release automation
│       └── docker-publish.yml         # Publish to GHCR on release
│
├── .env.example
├── docker-compose.yml                 # Symlink to infra/docker-compose.yml
├── README.md
├── PLAN.md                            # This file
├── CONTRIBUTING.md
├── LICENSE                            # MIT
└── mkdocs.yml
```

---

## 7. Database Schema

### 7.1 Core Tables

```sql
-- Users
CREATE TABLE users (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email       VARCHAR(255) UNIQUE NOT NULL,
    hashed_password VARCHAR(255) NOT NULL,
    full_name   VARCHAR(255),
    is_active   BOOLEAN DEFAULT true,
    created_at  TIMESTAMPTZ DEFAULT now(),
    updated_at  TIMESTAMPTZ DEFAULT now()
);

-- User Preferences
CREATE TABLE user_preferences (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
    target_roles    TEXT[],              -- ["Software Engineer", "Backend Engineer"]
    tech_stack      TEXT[],              -- ["Python", "FastAPI", "PostgreSQL"]
    locations       TEXT[],              -- ["Remote", "Chennai", "Bangalore"]
    salary_min      INTEGER,             -- in USD/year
    salary_max      INTEGER,
    remote_only     BOOLEAN DEFAULT false,
    company_sizes   TEXT[],              -- ["startup", "mid", "enterprise"]
    industries      TEXT[],              -- ["fintech", "saas", "ai"]
    excluded_companies TEXT[],           -- Companies to skip
    auto_apply      BOOLEAN DEFAULT false,
    apply_limit_per_day INTEGER DEFAULT 10,
    require_approval BOOLEAN DEFAULT true,  -- Human-in-the-loop
    updated_at      TIMESTAMPTZ DEFAULT now()
);

-- Companies
CREATE TABLE companies (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name        VARCHAR(255) NOT NULL,
    domain      VARCHAR(255),
    website     VARCHAR(500),
    linkedin_url VARCHAR(500),
    size        VARCHAR(50),             -- startup, small, mid, large, enterprise
    industry    VARCHAR(100),
    location    VARCHAR(255),
    description TEXT,
    metadata    JSONB DEFAULT '{}',      -- funding, yc_batch, crunchbase_id, etc.
    created_at  TIMESTAMPTZ DEFAULT now()
);

-- Jobs
CREATE TABLE jobs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id      UUID REFERENCES companies(id),
    title           VARCHAR(255) NOT NULL,
    url             VARCHAR(1000) UNIQUE NOT NULL,
    url_hash        VARCHAR(64) UNIQUE NOT NULL,   -- SHA256 for dedup
    source          VARCHAR(50) NOT NULL,          -- linkedin, indeed, wellfound, hn
    location        VARCHAR(255),
    salary_min      INTEGER,
    salary_max      INTEGER,
    salary_currency VARCHAR(10) DEFAULT 'USD',
    job_type        VARCHAR(50),                   -- full-time, contract, part-time
    remote_type     VARCHAR(50),                   -- remote, hybrid, onsite
    description_raw TEXT,
    description_parsed JSONB,                      -- structured: requirements, benefits, stack
    ats_type        VARCHAR(50),                   -- greenhouse, lever, workday, linkedin, generic
    apply_url       VARCHAR(1000),
    relevance_score FLOAT,                         -- 0-100 match against user profile
    is_active       BOOLEAN DEFAULT true,
    posted_at       TIMESTAMPTZ,
    fetched_at      TIMESTAMPTZ DEFAULT now(),
    expires_at      TIMESTAMPTZ
);

-- Resumes
CREATE TABLE resumes (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id     UUID REFERENCES users(id) ON DELETE CASCADE,
    name        VARCHAR(255) NOT NULL,             -- "Master Resume", "Backend Focus"
    content_raw TEXT NOT NULL,                     -- Original resume text
    content_json JSONB,                            -- Parsed sections
    file_path   VARCHAR(500),                      -- MinIO path to PDF/DOCX
    is_master   BOOLEAN DEFAULT false,
    created_at  TIMESTAMPTZ DEFAULT now()
);

-- Resume Versions (tailored per application)
CREATE TABLE resume_versions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    resume_id       UUID REFERENCES resumes(id),
    application_id  UUID,                          -- FK to applications (set after creation)
    job_id          UUID REFERENCES jobs(id),
    content_raw     TEXT NOT NULL,                 -- Tailored resume text
    content_json    JSONB,                         -- Parsed sections
    cover_letter    TEXT,
    ats_score       INTEGER,                       -- 0-100
    keywords_matched TEXT[],
    file_path_pdf   VARCHAR(500),
    file_path_docx  VARCHAR(500),
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Applications
CREATE TABLE applications (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
    job_id          UUID REFERENCES jobs(id),
    resume_version_id UUID REFERENCES resume_versions(id),
    status          VARCHAR(50) NOT NULL DEFAULT 'wishlist',
    -- wishlist | applying | applied | screening | interview | assessment | offer | rejected | withdrawn | closed_won
    applied_at      TIMESTAMPTZ,
    ats_type        VARCHAR(50),
    confirmation_url VARCHAR(500),
    screenshot_path VARCHAR(500),                  -- MinIO path to confirmation screenshot
    notes           TEXT,
    metadata        JSONB DEFAULT '{}',            -- application ID from ATS, etc.
    created_at      TIMESTAMPTZ DEFAULT now(),
    updated_at      TIMESTAMPTZ DEFAULT now()
);

-- Application Status History
CREATE TABLE status_history (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    application_id  UUID REFERENCES applications(id) ON DELETE CASCADE,
    previous_status VARCHAR(50),
    new_status      VARCHAR(50) NOT NULL,
    changed_by      VARCHAR(50) DEFAULT 'agent',   -- agent, user, email_parser
    source_email_id UUID,                          -- FK to email that triggered change
    notes           TEXT,
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Contacts (for referrals and cold email targets)
CREATE TABLE contacts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id      UUID REFERENCES companies(id),
    full_name       VARCHAR(255),
    title           VARCHAR(255),
    email           VARCHAR(255),
    linkedin_url    VARCHAR(500),
    type            VARCHAR(50),                   -- employee, founder, recruiter, hiring_manager
    score           INTEGER DEFAULT 0,             -- referral priority score
    email_verified  BOOLEAN DEFAULT false,
    last_contacted  TIMESTAMPTZ,
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Referral Outreach
CREATE TABLE referrals (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    application_id  UUID REFERENCES applications(id),
    contact_id      UUID REFERENCES contacts(id),
    user_id         UUID REFERENCES users(id),
    status          VARCHAR(50) DEFAULT 'queued',
    -- queued | pending_approval | sent | followed_up | replied | referred | declined | no_response
    channel         VARCHAR(50),                   -- email, linkedin_dm
    message_draft   TEXT,
    message_sent    TEXT,
    sent_at         TIMESTAMPTZ,
    replied_at      TIMESTAMPTZ,
    follow_up_count INTEGER DEFAULT 0,
    next_followup_at TIMESTAMPTZ,
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Cold Emails
CREATE TABLE cold_emails (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id),
    contact_id      UUID REFERENCES contacts(id),
    company_id      UUID REFERENCES companies(id),
    subject         VARCHAR(500),
    body_draft      TEXT,
    body_sent       TEXT,
    status          VARCHAR(50) DEFAULT 'queued',
    -- queued | pending_approval | sent | bounced | replied | follow_up_sent | deal_opened | deal_closed | unsubscribed
    sent_at         TIMESTAMPTZ,
    replied_at      TIMESTAMPTZ,
    follow_up_count INTEGER DEFAULT 0,
    next_followup_at TIMESTAMPTZ,
    company_research JSONB,                        -- Research used to personalize
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Conversations (active threads with founders/recruiters)
CREATE TABLE conversations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id),
    application_id  UUID REFERENCES applications(id),
    cold_email_id   UUID REFERENCES cold_emails(id),
    company_id      UUID REFERENCES companies(id),
    contact_id      UUID REFERENCES contacts(id),
    subject         VARCHAR(500),
    deal_status     VARCHAR(50) DEFAULT 'replied',
    -- replied | interested | call_scheduled | offer | negotiating | closed_won | closed_lost | ghosted
    last_message_at TIMESTAMPTZ,
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Messages within a Conversation
CREATE TABLE messages (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE,
    direction       VARCHAR(10) NOT NULL,           -- inbound, outbound
    from_email      VARCHAR(255),
    to_email        VARCHAR(255),
    subject         VARCHAR(500),
    body_text       TEXT,
    body_html       TEXT,
    ai_draft        TEXT,                          -- AI-suggested reply
    imap_uid        VARCHAR(100),                  -- For dedup
    sent_at         TIMESTAMPTZ,
    received_at     TIMESTAMPTZ,
    created_at      TIMESTAMPTZ DEFAULT now()
);

-- Notifications
CREATE TABLE notifications (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID REFERENCES users(id) ON DELETE CASCADE,
    type            VARCHAR(100) NOT NULL,
    -- jobs_found | application_submitted | status_changed | referral_replied |
    -- cold_email_replied | deal_closed | follow_up_due | interview_tomorrow
    title           VARCHAR(255) NOT NULL,
    body            TEXT,
    link            VARCHAR(500),                  -- Deep link to relevant page
    is_read         BOOLEAN DEFAULT false,
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ DEFAULT now()
);
```

### 7.2 Key Indexes

```sql
CREATE INDEX idx_jobs_url_hash ON jobs(url_hash);
CREATE INDEX idx_jobs_relevance ON jobs(relevance_score DESC);
CREATE INDEX idx_jobs_source ON jobs(source);
CREATE INDEX idx_applications_user_status ON applications(user_id, status);
CREATE INDEX idx_status_history_application ON status_history(application_id, created_at DESC);
CREATE INDEX idx_notifications_user_unread ON notifications(user_id, is_read, created_at DESC);
CREATE INDEX idx_messages_conversation ON messages(conversation_id, received_at DESC);
CREATE INDEX idx_cold_emails_status ON cold_emails(user_id, status);
```

---

## 8. AI Agent Design

### 8.1 Base Agent Contract

```python
# backend/app/agents/base_agent.py

from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Any

@dataclass
class AgentResult:
    status: str           # "success" | "failed" | "pending_approval" | "skipped"
    data: dict[str, Any]
    next_action: str | None = None
    error: str | None = None

class BaseAgent(ABC):
    """All agents extend this. Provides LLM client, tools, structured logger."""

    def __init__(self, llm_model: str, tools: list, db_session, settings):
        self.llm = OllamaClient(model=llm_model)
        self.tools = tools
        self.db = db_session
        self.settings = settings
        self.logger = get_logger(self.__class__.__name__)

    @abstractmethod
    async def run(self, context: dict[str, Any]) -> AgentResult:
        """Entry point called by Celery task."""
        ...
```

### 8.2 Job Scout Agent

**File:** `backend/app/agents/job_scout.py`
**Model:** `llama3.1:8b` (speed-critical)
**Trigger:** Celery Beat every 6 hours

**LangGraph Flow:**
```
START
  │
  ▼
load_user_preferences
  │
  ▼
scrape_job_boards (parallel: LinkedIn, Indeed, Wellfound, HN)
  │
  ▼
parse_and_structure_jobs (LLM: extract title, company, salary, stack, type)
  │
  ▼
embed_job_descriptions (nomic-embed-text)
  │
  ▼
score_against_user_profile (Qdrant cosine similarity)
  │
  ▼
filter_by_threshold (score > preferences.min_relevance, default 60)
  │
  ▼
dedup_against_existing (url_hash lookup in PostgreSQL)
  │
  ▼
persist_new_jobs (INSERT to jobs table)
  │
  ▼
trigger_notification ("X new jobs found matching your profile")
  │
  ▼
END → AgentResult(status="success", data={"new_jobs": N})
```

**Tools:**
- `scrape_linkedin(query, location, limit)` → raw job list
- `scrape_indeed(query, location, limit)` → raw job list
- `scrape_wellfound(role, stack, remote)` → raw job list
- `scrape_hn_hiring(month_post_url)` → raw job list
- `embed_text(text)` → vector (via Ollama nomic-embed-text)
- `search_qdrant(vector, top_k)` → similar jobs
- `save_job_to_db(job_data)` → UUID

### 8.3 Resume Tailor Agent

**File:** `backend/app/agents/resume_tailor.py`
**Model:** `llama3.1:70b` (quality-critical)
**Trigger:** `tailor_resume` Celery task (before each application)

**LangGraph Flow:**
```
START
  │
  ▼
load_master_resume + load_job_description
  │
  ▼
extract_jd_requirements (LLM: required skills, preferred skills, keywords, role context)
  │
  ▼
analyze_resume_gaps (LLM: what's missing, what needs emphasis)
  │
  ▼
plan_rewrites (LLM: list of specific bullet-point changes)
  │
  ▼
rewrite_sections (LLM: rewrite each planned section — NO fabrication)
  │
  ▼
ats_score_check (keyword match %)
  │
  ├── score < 80? → loop back to plan_rewrites (max 3 iterations)
  │
  └── score >= 80? → continue
  │
  ▼
generate_cover_letter (LLM: personalized, references JD specifics)
  │
  ▼
export_to_pdf_and_docx (WeasyPrint + python-docx)
  │
  ▼
save_resume_version (PostgreSQL + MinIO)
  │
  ▼
END → AgentResult(status="success", data={"resume_version_id": UUID, "ats_score": 85})
```

**System Prompt (critical):**
```
You are a professional resume writer. Your ONLY job is to rephrase and reorder the user's existing
experience to better match the job description. You MUST NOT:
- Add experience, skills, or achievements the user has not listed
- Change dates, company names, or job titles
- Fabricate metrics or numbers

You MAY:
- Reword bullet points to use the JD's exact terminology
- Reorder bullet points to lead with most relevant items
- Add skills the user mentioned but are also in the JD
- Adjust the summary section to reflect the target role
```

### 8.4 Apply Agent

**File:** `backend/app/agents/apply_agent.py`
**Model:** `llama3.1:8b` (form field reasoning)
**Trigger:** `apply_to_job` Celery task

**ATS Adapters (Playwright strategies):**

| ATS | Detection Method | Strategy |
|-----|-----------------|----------|
| LinkedIn Easy Apply | URL contains `linkedin.com/jobs` | Click "Easy Apply", fill modal steps |
| Greenhouse | URL contains `boards.greenhouse.io` | Direct form fill, file upload |
| Lever | URL contains `jobs.lever.co` | Form fill + resume upload |
| Workday | URL contains `myworkdayjobs.com` | Multi-step wizard navigation |
| iCIMS | URL contains `icims.com` | Account creation + form fill |
| Ashby | URL contains `ashbyhq.com` | Single-page form |
| Generic | Fallback | LLM reads DOM, identifies form fields, fills intelligently |

**LangGraph Flow:**
```
START
  │
  ▼
load_application_context (job, resume_version, user_profile)
  │
  ▼
launch_stealth_browser (Playwright + stealth plugin)
  │
  ▼
navigate_to_apply_url
  │
  ▼
detect_ats_type (URL pattern + DOM analysis)
  │
  ▼
execute_ats_adapter (fills form fields, uploads resume PDF)
  │
  ▼
reach_confirmation_page_or_final_step
  │
  ▼
generate_preview_summary (what will be submitted)
  │
  ▼
send_approval_request_to_user (WebSocket push → UI modal)
  │
  ▼
wait_for_user_approval (timeout = 24h; if expired → skip)
  │
  ├── Rejected → abort, status = "skipped"
  │
  └── Approved → submit_form()
  │
  ▼
screenshot_confirmation_page → MinIO
  │
  ▼
update_application_status (status = "applied")
  │
  ▼
END → AgentResult(status="success", data={"confirmation_url": ..., "screenshot": ...})
```

### 8.5 Email Parser Agent

**File:** `backend/app/agents/email_parser.py`
**Model:** `llama3.1:8b` (classification)
**Trigger:** Celery Beat every 15 minutes

**Classification Labels & Actions:**

| Label | Trigger | Action |
|-------|---------|--------|
| `rejection` | "unfortunately", "moved forward with other candidates" | Status → `rejected` |
| `interview_invite` | "schedule", "interview", "call", "chat" | Status → `interview`, generate prep brief |
| `offer` | "offer letter", "pleased to offer", "compensation" | Status → `offer`, notify urgently |
| `assessment` | "coding challenge", "take-home", "hackerrank" | Status → `assessment`, add deadline |
| `info_request` | Questions about experience, portfolio, availability | Draft reply, notify user |
| `follow_up` | "just checking in", "following up" | Note in timeline |
| `other` | Everything else | Log only |

### 8.6 Referral Agent

**File:** `backend/app/agents/referral_agent.py`
**Model:** `llama3.1:70b` (personalization)
**Trigger:** After each successful application

**Contact Scoring Algorithm:**
```python
def score_contact(contact, user_profile, job) -> int:
    score = 0
    if is_alumni(contact, user_profile):          score += 40
    if has_mutual_connection(contact, user_profile): score += 30
    if contact.title == job.title:                score += 20  # Same role
    if "engineer" in contact.title.lower():       score += 10
    if contact.email_verified:                    score += 15
    if contact.type == "recruiter":               score -= 10  # Lower priority vs peer
    return min(score, 100)
```

**Outreach Sequence:**
1. Day 0: Send initial personalized message (pending user approval)
2. Day 5: Follow-up if no reply ("Just wanted to bump this up...")
3. Day 10: Final nudge ("No worries if you're busy — last message from me!")

### 8.7 Cold Email Agent

**File:** `backend/app/agents/cold_email.py`
**Model:** `llama3.1:70b` (research + writing)
**Trigger:** Manual trigger or weekly Celery Beat

**Company Research Pipeline:**
```
1. scrape_crunchbase(company_name) → funding history, investors, headcount
2. scrape_yc_directory(batch) → YC-specific context
3. fetch_company_blog(domain) → recent posts (last 3)
4. fetch_founder_linkedin(linkedin_url) → role history, shared interests
5. google_recent_news(company_name, last_30_days) → latest announcements
6. LLM synthesizes → company_research: {hook, value_prop, personalization_hooks}
```

**Cold Email Template Structure (enforced via prompt):**
```
Line 1 (Hook): Something specific about the company/founder that shows real research
Line 2-3 (Relevance + Value): Why you specifically, what you bring to THEIR problem
Line 4 (Ask): One clear, low-friction ask (15-minute call / reply with interest)
Line 5 (CTA): Single closing line, no pressure
```

### 8.8 Deal Closer Agent

**File:** `backend/app/agents/deal_closer.py`
**Model:** `llama3.1:70b` (negotiation + writing)
**Trigger:** Activated when inbound reply detected in Conversations

**Intent Classification:**
```
Interested     → "sounds great", "tell me more", "love your background"
Scheduling     → "let's hop on a call", "when are you free", "calendly"
Offer          → "we'd like to extend an offer", "compensation package"
Negotiating    → counter-offer exchanges, equity discussions
Ghosted        → no reply for 14+ days after interest shown
Closed Won     → signed offer letter, verbal acceptance confirmed
Closed Lost    → "decided to go another direction", withdrawal
```

**Negotiation Coach triggers when:**
- Offer letter received → shows market rate benchmarks (scraped from levels.fyi)
- Salary number mentioned → suggests counter-offer (1.1–1.2x as starting point)
- Equity discussed → explains vesting cliff, strike price considerations

---

## 9. API Endpoints

### 9.1 Authentication

```
POST   /api/v1/auth/register          Create new user account
POST   /api/v1/auth/login             Return JWT access + refresh token
POST   /api/v1/auth/refresh           Refresh access token
POST   /api/v1/auth/logout            Invalidate refresh token
```

### 9.2 Jobs

```
GET    /api/v1/jobs                   List jobs (filters: status, source, score_min, search)
GET    /api/v1/jobs/{id}              Get job detail
POST   /api/v1/jobs/search            Trigger manual job search
DELETE /api/v1/jobs/{id}              Remove job from feed (hide)
POST   /api/v1/jobs/{id}/save         Save to wishlist
```

### 9.3 Applications

```
GET    /api/v1/applications           List applications (filters: status, company, date_range)
GET    /api/v1/applications/{id}      Get application detail + status history
POST   /api/v1/applications           Create manual application entry
PATCH  /api/v1/applications/{id}      Update status, notes
DELETE /api/v1/applications/{id}      Remove application
POST   /api/v1/applications/{id}/apply   Trigger apply agent for a job
POST   /api/v1/applications/{id}/approve-submission   Approve apply agent's submission
POST   /api/v1/applications/{id}/reject-submission    Reject apply agent's submission
```

### 9.4 Profile & Resume

```
GET    /api/v1/profile                Get user profile
PUT    /api/v1/profile                Update profile
GET    /api/v1/profile/preferences    Get job preferences
PUT    /api/v1/profile/preferences    Update job preferences
GET    /api/v1/profile/resumes        List all resumes
POST   /api/v1/profile/resumes        Upload new master resume
GET    /api/v1/profile/resumes/{id}   Get resume + all versions
POST   /api/v1/profile/resumes/{id}/tailor   Trigger tailor agent for a job
GET    /api/v1/profile/resumes/versions/{id}/download   Download PDF or DOCX
```

### 9.5 Outreach

```
GET    /api/v1/outreach/referrals     List all referral outreach
POST   /api/v1/outreach/referrals     Trigger referral agent for an application
PATCH  /api/v1/outreach/referrals/{id}/approve   Approve + send message
PATCH  /api/v1/outreach/referrals/{id}/reject    Reject message

GET    /api/v1/outreach/cold-emails   List cold email campaigns
POST   /api/v1/outreach/cold-emails   Trigger cold email agent
PATCH  /api/v1/outreach/cold-emails/{id}/approve   Approve + send email
PATCH  /api/v1/outreach/cold-emails/{id}/reject    Reject email
GET    /api/v1/outreach/cold-emails/{id}/research  View company research used
```

### 9.6 Conversations

```
GET    /api/v1/conversations          List all active conversations
GET    /api/v1/conversations/{id}     Get full thread + messages
POST   /api/v1/conversations/{id}/draft-reply    Ask AI to draft a reply
POST   /api/v1/conversations/{id}/send-reply     Approve and send reply
PATCH  /api/v1/conversations/{id}/status         Update deal status manually
```

### 9.7 Notifications

```
GET    /api/v1/notifications          List notifications (unread first)
PATCH  /api/v1/notifications/{id}/read   Mark as read
PATCH  /api/v1/notifications/read-all    Mark all as read
GET    /api/v1/notifications/settings    Get notification preferences
PUT    /api/v1/notifications/settings    Update notification preferences
```

### 9.8 Analytics

```
GET    /api/v1/analytics/summary      Overall stats (applied, interviews, offers, response_rate)
GET    /api/v1/analytics/weekly       Week-by-week breakdown
GET    /api/v1/analytics/funnel       Application → interview → offer conversion
GET    /api/v1/analytics/patterns     AI-detected patterns + recommendations
```

### 9.9 WebSocket

```
WS     /api/v1/ws                     Real-time event stream (JWT auth in query param)
```

**WebSocket Event Types:**
```json
{ "type": "jobs_found",         "data": { "count": 12 } }
{ "type": "application_submitted", "data": { "job_id": "...", "company": "..." } }
{ "type": "status_changed",     "data": { "application_id": "...", "new_status": "interview" } }
{ "type": "reply_received",     "data": { "conversation_id": "...", "from": "founder@startup.com" } }
{ "type": "approval_required",  "data": { "type": "apply", "application_id": "...", "preview": {...} } }
{ "type": "deal_closed",        "data": { "conversation_id": "...", "company": "..." } }
```

---

## 10. Phase Roadmap

> **No fixed timeline. Ship each phase when it's done and tested. Quality over speed.**

### Phase 1 — Core Foundation & Job Discovery

**Goal:** Working monorepo, infrastructure, database, auth, and first agent (Job Scout) running end-to-end.

**Deliverables:**
- [ ] Monorepo initialized with `uv`, `ruff`, `mypy`, `pytest`, GitHub Actions CI
- [ ] `docker-compose.yml` with: FastAPI, PostgreSQL, Redis, Celery, Celery Beat, Qdrant, MinIO, Flower, Nginx
- [ ] Full database schema implemented with Alembic migrations
- [ ] FastAPI app factory with JWT auth (FastAPI-Users)
- [ ] User registration, login, profile, preferences CRUD
- [ ] Resume upload → MinIO storage
- [ ] **Job Scout Agent v1:** Crawl4AI scrapers for LinkedIn, Indeed, Wellfound, HN
- [ ] Qdrant vector store: embed JDs + user profile; semantic similarity scoring
- [ ] Celery Beat schedule: run scout every 6 hours; dedup via URL hash
- [ ] React web shell: job feed page + preferences settings page
- [ ] REST API: `/jobs`, `/preferences`

**Definition of Done:**
- System finds and displays relevant jobs automatically
- All services start with `docker-compose up`
- 70%+ test coverage on backend

---

### Phase 2 — Resume Tailoring + Auto-Apply Engine

**Goal:** LLM tailors resume per JD, and Playwright submits applications to real job boards.

**Deliverables:**
- [ ] Ollama integration with `llama3.1:8b` and `llama3.1:70b`
- [ ] **Resume Tailor Agent:** LangGraph pipeline (parse JD → extract keywords → rewrite → ATS score → iterate → export PDF)
- [ ] Resume versioning: each tailored version stored in MinIO + PostgreSQL
- [ ] Cover letter generation per role
- [ ] **Apply Agent:** Playwright adapters for LinkedIn Easy Apply, Greenhouse, Lever, Workday
- [ ] Generic form-filling fallback (LLM-guided)
- [ ] Anti-detection: Playwright stealth, random delays, proxy rotation support
- [ ] Human-in-the-loop confirmation: modal before every submission
- [ ] Application confirmation screenshots → MinIO
- [ ] Kanban board UI launched (drag-and-drop status management)
- [ ] Application detail page: timeline, resume used, screenshot

**Definition of Done:**
- System tailors resume, generates cover letter, and submits to at least 3 ATS types
- User approval gate works correctly
- All applied applications appear in Kanban board

---

### Phase 3 — Email Parsing + Status Tracking + Notifications

**Goal:** Close the feedback loop — detect status changes from inbox, push real-time notifications everywhere.

**Deliverables:**
- [ ] **Email Parser Agent:** `aioimaplib` IMAP watcher (Celery Beat every 15 min)
- [ ] LLM email classifier: Rejection | Interview | Offer | Assessment | Info Request
- [ ] Company/sender domain → application matching
- [ ] Auto status updates in PostgreSQL + StatusHistory events
- [ ] FastAPI WebSocket gateway + Redis pub/sub for real-time push
- [ ] Self-hosted Ntfy: iOS + Android + desktop toast notifications
- [ ] Notification preference settings (per-type on/off)
- [ ] Interview invite trigger: auto-generate company research brief
- [ ] Follow-up reminders: nudge after N days of no response
- [ ] Application detail page: full email thread view + timeline
- [ ] Weekly email digest (Celery Beat, every Sunday)

**Definition of Done:**
- Rejection email → automatic status update in UI within 15 minutes
- WebSocket push works in open browser tab
- Ntfy notification received on mobile

---

### Phase 4 — Referral & Cold Email Outreach

**Goal:** Active networking engine — find contacts, draft messages, run campaigns, detect replies.

**Deliverables:**
- [ ] **Referral Agent:** LinkedIn employee scraper (Crawl4AI), contact scoring algorithm
- [ ] Personalized referral message generation (LLM)
- [ ] 3-touch email/DM sequence with auto-scheduling
- [ ] Reply detection via IMAP; notification on reply
- [ ] User approval before each message sent
- [ ] Outreach tracking UI: sent, replied, referred
- [ ] **Cold Email Agent:** Crunchbase/YC/AngelList startup discovery
- [ ] Founder email finding: Hunter.io free tier + pattern guessing (`{first}@{domain}`, etc.)
- [ ] Company research pipeline (funding, product, blog, founder bio)
- [ ] Hyper-personalized cold email generation (5-sentence structure)
- [ ] SMTP sending via user's email account (Gmail OAuth / custom SMTP)
- [ ] 2-touch sequence (initial + 7-day follow-up)
- [ ] Bounce detection + unsubscribe tracking
- [ ] Reply → conversation thread created → user notified

**Definition of Done:**
- Cold email campaign sends, detects reply, creates conversation, notifies user
- Referral outreach sequence runs autonomously with approval gates

---

### Phase 5 — Deal Closer + Communication Assistant

**Goal:** When a conversation gets warm, become the user's intelligent co-pilot for closing the deal.

**Deliverables:**
- [ ] **Deal Closer Agent:** intent classification on all inbound messages
- [ ] Deal status pipeline (Replied → Interested → Call Scheduled → Offer → Negotiating → Closed)
- [ ] Draft reply generation with full thread context
- [ ] In-app conversation UI: full thread + AI draft panel side-by-side
- [ ] User approval workflow: draft → user edits → approve → send
- [ ] Negotiation coach: market rate lookup (levels.fyi scraping), counter-offer suggestions
- [ ] Deal Closed Won notification: 🎉 urgent push + email
- [ ] Analytics: deal conversion rate, avg. time-to-close
- [ ] Pipeline view: all active warm conversations in one board

**Definition of Done:**
- User can view a founder's reply, see AI draft, edit it, and send — all in-app
- Closed Won triggers correct notifications
- Deal status updates correctly across all states

---

### Phase 6 — Production Hardening + Commercial Launch

**Goal:** Make it bulletproof for thousands of self-hosting users, and launch commercially.

**Deliverables:**
- [ ] **Tauri 2 desktop app:** code-signed installers (Windows .exe, macOS .dmg, Linux .AppImage)
- [ ] **Expo mobile app:** iOS + Android build with Expo push notifications
- [ ] One-click installer: `curl -sSL install.jobhuntai.com | bash` + PowerShell equivalent
- [ ] Interactive setup wizard (first-run config: Ollama model selection, email credentials, preferences)
- [ ] Auto-update mechanism for Tauri app
- [ ] Prometheus metrics instrumentation on all agents + API endpoints
- [ ] Grafana dashboards: agent activity, application funnel, system health
- [ ] Rate limiting + circuit breakers on all scrapers (prevent IP banning)
- [ ] Retry logic with exponential backoff on all external calls
- [ ] Comprehensive security audit (OWASP Top 10 checklist)
- [ ] Credential encryption review (secrets stored in OS keychain via Tauri)
- [ ] Multi-user SaaS mode: user isolation, usage quotas per plan
- [ ] Stripe billing integration (cloud-hosted SaaS version)
- [ ] MkDocs documentation site: self-host guide, API reference, agent docs, FAQ
- [ ] GitHub release pipeline: automated Docker image publishing to GHCR
- [ ] Product Hunt launch kit + HN Show HN post
- [ ] `README.md` with quick-start, screenshots, demo GIF

**Definition of Done:**
- `docker-compose up` works on a fresh machine with zero manual steps
- Tauri app installs and connects to local backend automatically
- Documentation site live

---

## 11. Sprint Breakdown

> Sprints are feature-complete slices of work. No fixed duration — move to next when done + tested.

| Sprint | Name | Key Deliverables |
|--------|------|-----------------|
| **S1** | Project Bootstrap | Monorepo, Docker Compose, CI/CD, DB schema, Alembic migrations, JWT auth |
| **S2** | Job Scout Agent + React Shell | Crawl4AI scrapers, Qdrant semantic search, Celery Beat, job feed UI |
| **S3** | Resume Tailor Agent | Ollama integration, LangGraph pipeline, ATS scoring, PDF export, resume versioning |
| **S4** | Apply Agent | Playwright ATS adapters (LinkedIn/Greenhouse/Lever/Workday), approval modal, screenshots |
| **S5** | Email Parser + Status Engine | IMAP watcher, LLM classifier, auto status updates, application matching |
| **S6** | Real-time Notifications + Kanban UI | WebSocket gateway, Redis pub/sub, Ntfy push, Kanban drag-and-drop, notification settings |
| **S7** | Referral Agent | LinkedIn employee scraping, contact scoring, personalized messages, 3-touch sequences |
| **S8** | Cold Email Agent | Startup discovery, founder email finding, company research, LLM email writing, SMTP sending |
| **S9** | Deal Closer + Conversation UI | Intent classification, AI draft panel, approval flow, negotiation coach, deal pipeline |
| **S10** | Desktop (Tauri) + Mobile (Expo) | Native installers, system tray, mobile push, one-click install script |
| **S11** | Observability + Security | Prometheus/Grafana, rate limiting, circuit breakers, security audit, 90% test coverage |
| **S12** | Commercial Launch | Multi-user SaaS, Stripe, MkDocs docs, GHCR publishing, Product Hunt kit |

---

## 12. Deployment Strategy

### 12.1 Local Self-Host (Docker Compose)

**Prerequisites:** Docker Desktop, 8GB+ RAM, 50GB SSD

```bash
git clone https://github.com/yourusername/jobhunt-ai
cd jobhunt-ai
cp .env.example .env          # Edit with your email credentials
docker-compose up -d
# Open http://localhost:8080
```

**Services started by Docker Compose:**

| Service | Port | Description |
|---------|------|-------------|
| `api` | 8000 | FastAPI backend |
| `web` | 5173 | React frontend (Vite dev) / Nginx (prod) |
| `postgres` | 5432 | PostgreSQL database |
| `redis` | 6379 | Redis broker + cache |
| `celery-worker` | — | Async task worker |
| `celery-beat` | — | Cron scheduler |
| `flower` | 5555 | Celery monitoring UI |
| `qdrant` | 6333 | Vector database |
| `minio` | 9000 | Object storage |
| `ntfy` | 2586 | Push notification server |
| `ollama` | 11434 | Local LLM server |
| `nginx` | 8080 | Reverse proxy |
| `prometheus` | 9090 | Metrics collection |
| `grafana` | 3000 | Metrics dashboards |

### 12.2 Required Environment Variables

```bash
# backend/.env.example

# Database
DATABASE_URL=postgresql+asyncpg://postgres:password@postgres:5432/jobhuntai

# Redis
REDIS_URL=redis://redis:6379/0

# Security
SECRET_KEY=your-secret-key-min-32-chars
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# LLM (Ollama)
OLLAMA_BASE_URL=http://ollama:11434
LLM_FAST_MODEL=llama3.1:8b
LLM_QUALITY_MODEL=llama3.1:70b
EMBED_MODEL=nomic-embed-text

# Vector DB
QDRANT_URL=http://qdrant:6333

# Object Storage (MinIO)
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET_RESUMES=resumes
MINIO_BUCKET_SCREENSHOTS=screenshots

# Email (User's own account — for IMAP watching + SMTP sending)
EMAIL_ADDRESS=user@gmail.com
EMAIL_PASSWORD=app-specific-password  # Gmail App Password
IMAP_HOST=imap.gmail.com
IMAP_PORT=993
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587

# Notifications
NTFY_URL=http://ntfy:2586
NTFY_TOPIC=jobhuntai-alerts

# External APIs (all optional / free tier)
HUNTER_API_KEY=optional-for-email-finding
PROXYCRAWL_KEY=optional-for-proxy-rotation

# Rate Limits
MAX_APPLIES_PER_DAY=10
MAX_COLD_EMAILS_PER_DAY=20
REQUIRE_APPLY_APPROVAL=true

# Feature Flags
AUTO_APPLY_ENABLED=false      # Safety: disabled by default
COLD_EMAIL_ENABLED=false      # Safety: disabled by default
```

### 12.3 Desktop App (Tauri 2)

```bash
cd apps/desktop
npm install
npm run tauri build          # Creates installer in src-tauri/target/release/bundle/
```

- Windows: `.exe` installer (NSIS)
- macOS: `.dmg` installer (code-signed with Apple Dev cert)
- Linux: `.AppImage` + `.deb`

### 12.4 Mobile App (Expo)

```bash
cd apps/mobile
npm install
npx expo start              # Dev mode pointing to local backend
npx eas build               # Production build for App Store / Play Store
```

Configure `EXPO_PUBLIC_API_URL` to point to your self-hosted instance.

### 12.5 Cloud SaaS Deployment

For commercial multi-user deployment:
- **VPS:** Hetzner CX32 (4 vCPU, 8GB RAM, €12/mo) minimum per user shard
- **Database:** Self-hosted PostgreSQL on Hetzner or Supabase Pro
- **LLM:** Groq API (`GROQ_API_KEY`) as fast, cheap alternative to local Ollama — drop-in via env var
- **Scaling:** Celery workers scale horizontally; add more worker containers as user count grows
- **CDN:** Cloudflare for frontend, SSL, DDoS protection

---

## 13. Commercial Strategy

### 13.1 Pricing Model (Open Core)

| Tier | Price | Target | LLM | Limits |
|------|-------|--------|-----|--------|
| **Self-Host** | Free forever | Developers, power users | Local Ollama | Unlimited |
| **Cloud Pro** | $19/mo | Non-technical job seekers | Hosted (Groq/Claude) | 500 apps/mo |
| **Recruiter** | $49/mo | Recruiters, career coaches | Hosted | 5 profiles, 2000 apps/mo |
| **Agency** | $199/mo | Staffing agencies | Hosted | 25 profiles, white-label |

### 13.2 Competitive Differentiation

| Feature | JobHunt AI | LazyApply | Simplify | AutoApply |
|---------|-----------|-----------|---------|-----------|
| Self-hostable | ✅ | ❌ | ❌ | ❌ |
| Local LLM (private) | ✅ | ❌ | ❌ | ❌ |
| Cold email founders | ✅ | ❌ | ❌ | ❌ |
| Deal closer | ✅ | ❌ | ❌ | ❌ |
| Open source | ✅ | ❌ | ❌ | ❌ |
| Referral outreach | ✅ | ❌ | ❌ | ❌ |
| Resume versioning | ✅ | ✅ | ✅ | ❌ |

### 13.3 Launch Channels

1. **GitHub** — Open source release; aim for 500 stars in week 1
2. **Hacker News** — "Show HN: I built an AI agent that applies to jobs for me"
3. **Product Hunt** — Full launch with screenshots, demo video, maker comment
4. **Reddit** — r/cscareerquestions, r/jobsearchhacks, r/MachineLearning, r/SideProject
5. **Twitter/X** — Build-in-public thread documenting the entire development journey
6. **Dev.to / Hashnode** — Deep-dive article: architecture, agents, tech decisions
7. **YouTube** — Full demo walkthrough video (5-7 minutes)

---

## 14. Risks & Mitigations

| Risk | Impact | Likelihood | Mitigation |
|------|--------|-----------|-----------|
| Bot detection by job boards (LinkedIn/Indeed blocking scrapers) | High | High | Playwright stealth mode, randomized delays (2–8s), proxy rotation support, per-domain rate limits, configurable daily limits |
| LLM hallucinations in resume (fabricated experience) | High | Medium | Strict system prompt ("only rephrase, never fabricate"), factual grounding via original resume injection, user preview before every apply |
| Local LLM too slow on CPU-only machines | Medium | High | 8B model for speed-critical tasks, 70B only for quality tasks; Groq API as fast remote fallback (env var swap, no code changes) |
| CAN-SPAM / GDPR compliance for cold emails | High | Medium | Unsubscribe footer on all emails, max 20/day default, user configurable, clear docs on legal responsibility |
| LinkedIn/Indeed Terms of Service violations | High | Medium | Docs clearly state user is responsible; configurable rate limits; human-in-the-loop mode; official API options noted |
| ATS form variability (hundreds of different portals) | Medium | High | LLM-guided generic fallback handles unknown forms; pre-built adapters for top 5 ATS; community-contributed adapter pattern |
| SMTP sending hitting spam filters | Medium | Medium | SPF/DKIM/DMARC setup guide in docs; warm-up schedule guidance; Resend/Mailgun as professional SMTP fallback; 20/day cap |
| Scope creep | Medium | Medium | Each phase is a shippable unit; MVP is done after Phase 2; strict sprint scope; phases 3-6 are additive |

---

## 15. Definition of Done

### MVP (Phase 2 Complete)

- [ ] User sets preferences → system automatically finds matching jobs every 6 hours
- [ ] System tailors resume per JD using local LLM → ATS score ≥ 80
- [ ] System generates personalized cover letter per role
- [ ] System applies to LinkedIn Easy Apply + Greenhouse jobs via Playwright
- [ ] User approval modal works correctly before any submission
- [ ] All applications tracked in Kanban board with status
- [ ] Entire stack starts with `docker-compose up` on a fresh machine
- [ ] Backend test coverage ≥ 70%

### Production Ready (Phase 6 Complete)

- [ ] All 8 agents operational and tested
- [ ] Email parser detects status changes within 15 minutes
- [ ] WebSocket push + Ntfy mobile notifications working
- [ ] Referral + cold email campaigns run autonomously with approval gates
- [ ] Deal closer drafts replies and notifies on deal close
- [ ] Tauri desktop app installs and works on Windows/macOS/Linux
- [ ] Expo mobile app on iOS + Android with push notifications
- [ ] Prometheus + Grafana dashboards live
- [ ] Backend test coverage ≥ 90%
- [ ] MkDocs documentation site live
- [ ] Security audit passed (OWASP Top 10 checklist)

---

## 16. Claude Code Prompting Guide

> Use these prompts when vibe-coding each component with Claude Code. Start a fresh context per sprint. Always attach relevant files.

### Initial Setup

```
I'm building JobHunt AI — an open-source autonomous job hunting platform.
Stack: Python 3.12, FastAPI, SQLAlchemy 2.0 async, Alembic, Celery, Redis, Pydantic v2.
Package manager: uv. Linting: ruff. Type checking: mypy (strict). Tests: pytest-asyncio.

Read PLAN.md for the full project context before we start.

Task: Initialize the monorepo structure exactly as described in Section 6 (File Structure)
of PLAN.md. Create all directories and placeholder __init__.py files. Set up pyproject.toml
with uv, ruff (line-length=100, select=ALL), mypy strict mode, and pytest config.
Create docker-compose.yml with all services from Section 12.1.
Create .env.example with all variables from Section 12.2.
```

### Database Schema

```
Context: JobHunt AI backend. Read PLAN.md Section 7 (Database Schema).

Task: Implement all SQLAlchemy 2.0 async models in backend/app/models/ exactly as defined
in the schema. Use mapped_column() with proper types. Add __tablename__, __repr__,
and created_at/updated_at with server_default. Create Alembic migration for the full schema.
All foreign keys should use UUID. Add all indexes from Section 7.2.
```

### Base Agent + Job Scout

```
Context: JobHunt AI. Read PLAN.md Section 8.1 and 8.2.

Task: Implement BaseAgent (backend/app/agents/base_agent.py) and JobScoutAgent
(backend/app/agents/job_scout.py) exactly as designed.

JobScoutAgent requirements:
- LangGraph state machine with the exact flow from Section 8.2
- Tools: scrape_linkedin, scrape_indeed, scrape_wellfound using Crawl4AI
- Embed JDs with nomic-embed-text via Ollama
- Store in Qdrant, score against user profile
- Dedup via url_hash (SHA256)
- Celery task: run_job_scout in backend/app/tasks/scout_tasks.py
- Write pytest unit tests for all methods (mock Crawl4AI and Ollama calls)
```

### Resume Tailor Agent

```
Context: JobHunt AI. Read PLAN.md Section 8.3 carefully — especially the system prompt
constraints and the LangGraph flow.

Task: Implement ResumeTailorAgent (backend/app/agents/resume_tailor.py).

Critical requirements:
- Use llama3.1:70b via Ollama
- LangGraph with iterative ATS score loop (max 3 iterations, target >= 80)
- System prompt MUST include the no-fabrication instruction verbatim from PLAN.md
- PDF export via WeasyPrint, DOCX via python-docx
- Store version in MinIO + create resume_versions record
- Celery task: tailor_resume(application_id, job_id) in resume_tasks.py
- Unit tests with mocked LLM responses
```

### Apply Agent

```
Context: JobHunt AI. Read PLAN.md Section 8.4.

Task: Implement ApplyAgent (backend/app/agents/apply_agent.py) with Playwright.

Requirements:
- ATS detection by URL pattern (table in Section 8.4)
- Adapters for: LinkedIn Easy Apply, Greenhouse, Lever, Workday
- Generic LLM-guided fallback for unknown forms
- Human-in-the-loop: send WebSocket event type "approval_required" before submit
- Wait for approval with 24h timeout
- Screenshot confirmation page → upload to MinIO
- Update application.status = "applied" on success
- Use playwright-stealth for anti-detection
- Random delays between 2-8 seconds on all interactions
- Integration test against a local mock ATS HTML page
```

### Email Parser Agent

```
Context: JobHunt AI. Read PLAN.md Section 8.5.

Task: Implement EmailParserAgent (backend/app/agents/email_parser.py).

Requirements:
- aioimaplib async IMAP connection
- Celery Beat: check_email_inbox every 15 minutes
- LLM classification using the labels + trigger keywords from the table in Section 8.5
- Match emails to applications by company name / sender domain
- Create StatusHistory record on status change
- Trigger WebSocket broadcast (type: "status_changed")
- Trigger Ntfy push notification
- Low-confidence emails (< 0.7) flagged for manual review, NOT auto-updated
- Unit tests with sample email fixtures for each classification label
```

### Cold Email Agent

```
Context: JobHunt AI. Read PLAN.md Section 8.7.

Task: Implement ColdEmailAgent (backend/app/agents/cold_email.py).

Requirements:
- Crawl4AI scrapers for Crunchbase, YC directory, AngelList
- Hunter.io API integration (free tier) for email finding
- Email pattern fallback: first@domain, first.last@domain, flast@domain (try all, verify MX)
- Company research pipeline: funding + blog + LinkedIn + recent news (Crawl4AI)
- LLM email writing enforcing the 5-sentence structure from Section 8.7
- SMTP sending via aiosmtplib using user's configured credentials
- Celery task: run_cold_email_campaign
- Volume cap: MAX_COLD_EMAILS_PER_DAY from settings
- All emails pending_approval status by default — never auto-send without approval
- Reply detection: IMAP match by thread subject, create Conversation record
- Unit tests for research pipeline and email generation
```

### WebSocket + Notifications

```
Context: JobHunt AI. Read PLAN.md Sections 4.3 and 9.9.

Task: Implement the WebSocket gateway and notification system.

Requirements:
- FastAPI WebSocket endpoint at /api/v1/ws with JWT auth via query param
- ConnectionManager class: store active connections by user_id, broadcast to user
- Redis pub/sub: Celery tasks publish events, WebSocket manager subscribes
- All event types from Section 9.9 implemented
- Ntfy service (backend/app/services/notify.py): send_push(user_id, title, body, type)
- Notification model: create DB record + broadcast WS + send Ntfy for every event
- React hook: useWebSocket.ts — reconnect on disconnect, parse all event types
- Integration test: publish event from Celery task, assert WebSocket client receives it
```

### React Frontend

```
Context: JobHunt AI frontend. Read PLAN.md Sections 2 and 9 for UI requirements.

Task: Build the Kanban application tracker page (apps/web/src/pages/Applications.tsx).

Requirements:
- Columns: Wishlist | Applied | Screening | Interview | Assessment | Offer | Rejected
- Cards: company logo (from clearbit.com/logo/domain), job title, applied date, relevance score
- Drag-and-drop between columns (react-beautiful-dnd) — PATCH /applications/{id}
- Click card → application detail sheet: full timeline, resume version used, all emails, contacts
- Real-time: useWebSocket hook updates card status without page refresh
- Filter bar: by company, date range, source
- Empty state for each column with helpful CTA
- Fully responsive (mobile-first)
- shadcn/ui components + Tailwind only (no custom CSS)
```

### Docker Compose & Infrastructure

```
Context: JobHunt AI. Read PLAN.md Section 12.

Task: Write production-ready docker-compose.yml covering all services in Section 12.1.

Requirements:
- All services with health checks
- Volumes for persistent data (postgres, redis, qdrant, minio, ollama models)
- Ollama service: pulls llama3.1:8b and nomic-embed-text on startup via entrypoint script
- Celery worker + beat as separate services with shared backend code volume
- Nginx reverse proxy: / → web app, /api → FastAPI, /ws → WebSocket, /ntfy → Ntfy
- Environment variables from .env file via env_file directive
- networks: internal bridge (no external exposure except Nginx port 8080)
- docker-compose.prod.yml: resource limits, restart: always, production Nginx config
```

---

## Development Workflow

### Starting a new feature

```bash
git checkout -b feat/agent-name
cd backend
uv run pytest tests/ -x -q          # Ensure green before starting
# ... implement ...
uv run pytest tests/ --cov=app --cov-report=term-missing
uv run ruff check . && uv run mypy app/
git commit -m "feat(agents): add cold email agent with company research pipeline"
```

### Running the full stack locally

```bash
docker-compose up -d postgres redis qdrant minio   # Infrastructure only
cd backend && uv run uvicorn app.main:app --reload  # API hot reload
cd backend && uv run celery -A app.core.celery worker --loglevel=info
cd apps/web && npm run dev
```

### Pulling Ollama models

```bash
docker exec -it jobhuntai-ollama ollama pull llama3.1:8b
docker exec -it jobhuntai-ollama ollama pull llama3.1:70b    # ~40GB — only if 16GB+ RAM
docker exec -it jobhuntai-ollama ollama pull nomic-embed-text
```

---

## Contributing

See `CONTRIBUTING.md`. All contributions welcome — especially:
- New ATS adapters (Apply Agent)
- New job board scrapers (Job Scout Agent)
- Improved prompt templates
- Mobile app features

---

## License

MIT License — see `LICENSE` file.

---

*Built with Python, FastAPI, LangGraph, Ollama, Playwright, and a lot of coffee. ☕*
