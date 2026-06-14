# Architecture Summary — JobHunt AI

> One-page system overview. For deep detail, read [`docs/architecture.md`](../../docs/architecture.md)
> and [`PLAN.md` §4–§9](../../PLAN.md).

---

## What is JobHunt AI?

An autonomous, multi-agent platform that automates the entire job hunting pipeline: discovers
relevant jobs, tailors resumes, applies via browser automation, tracks applications, detects status
changes via email, runs referral outreach, sends cold emails to founders, and assists in closing
deals. All with human approval gates, running privately on the user's machine.

---

## System Architecture

### Layers (High to Low)

```
┌─────────────────────────────────────────────────────┐
│ PRESENTATION                                        │
│ React Web · Tauri Desktop · Expo Mobile · Ntfy     │
└─────────────────┬───────────────────────────────────┘
                  │ HTTP / WebSocket / Push
┌─────────────────▼───────────────────────────────────┐
│ API GATEWAY                                         │
│ FastAPI REST · WebSocket (Redis pub/sub) · JWT Auth│
│ Request validation (Pydantic) · Rate limiting       │
└─────────────────┬───────────────────────────────────┘
                  │ Python function calls
┌─────────────────▼───────────────────────────────────┐
│ AGENTS (LangGraph State Machines)                   │
│ Job Scout · Resume Tailor · Apply · Email Parser   │
│ Referral · Cold Email · Deal Closer · Tracker      │
└─────────────────┬───────────────────────────────────┘
                  │ Task dispatch
┌─────────────────▼───────────────────────────────────┐
│ TASK QUEUE                                          │
│ Celery Workers (async tasks) · Redis Broker        │
│ Celery Beat (cron scheduler)                        │
└─────────────────┬───────────────────────────────────┘
                  │ DB / AI calls
┌─────────────────▼───────────────────────────────────┐
│ DATA + AI                                           │
│ PostgreSQL 16 · Qdrant · Redis · MinIO · Ollama    │
│ Vault (secrets) · OS Keychain (credentials)         │
└─────────────────┬───────────────────────────────────┘
                  │ HTTP / browser control
┌─────────────────▼───────────────────────────────────┐
│ INTEGRATION LAYER                                   │
│ Playwright (browser) · Crawl4AI (scraping)         │
│ aioimaplib (email) · aiosmtplib (sending)          │
└─────────────────────────────────────────────────────┘
```

**Rule:** Dependencies point downward only. API calls Agents; Agents call Services; Services call
external systems. Never the reverse.

---

## Core Concepts

### 1. Agents (Autonomous Decision Makers)

Each agent is a **LangGraph state machine** that extends `BaseAgent`:

```python
class BaseAgent(ABC):
    async def run(self, context: dict) -> AgentResult:
        # LangGraph nodes: call tools, make LLM decisions, persist results
        # Returns: AgentResult(status, data, next_action, error)
```

**Eight agents:**

| Agent | Responsibility | Model |
|-------|---|---|
| **Job Scout** | Discover + scrape + embed + score + persist jobs | `llama3.1:8b` |
| **Resume Tailor** | Rewrite resume to JD, ATS score, generate cover letter | `llama3.1:70b` |
| **Apply** | Drive Playwright across ATS portals, HITL approval | `llama3.1:8b` |
| **Email Parser** | Watch IMAP, classify emails, update application status | `llama3.1:8b` |
| **Referral** | Find/score contacts, draft personalized messages | `llama3.1:70b` |
| **Cold Email** | Discover startups, research, write 5-sentence emails | `llama3.1:70b` |
| **Deal Closer** | Classify intent, draft replies, coach negotiation | `llama3.1:70b` |
| **Tracker** | Compute analytics and pattern insights | — |

Each agent is **idempotent** and **retriable**. Celery handles retries + exponential backoff.

### 2. Data Model (PostgreSQL)

**Core tables:**

- **`users`** — user accounts, auth hashes.
- **`user_preferences`** — job search criteria (role, tech stack, location, salary, remote, etc.).
- **`jobs`** — discovered job postings with relevance scores.
- **`applications`** — one record per job a user applied to; status lifecycle.
- **`resumes`** — master resume + resume versions (one per application).
- **`status_history`** — audit trail of application status changes.
- **`cold_emails`** — outbound cold email campaigns.
- **`referrals`** — referral outreach sequences.
- **`conversations`** — active email threads with founders/recruiters.
- **`messages`** — email/DM messages within conversations.
- **`notifications`** — activity notifications.

**Full schema:** [`PLAN.md` §7](../../PLAN.md).

### 3. Semantic Matching (Qdrant)

Jobs are embedded using `nomic-embed-text`. User profile is also embedded. Cosine similarity scores
each job against the profile. Top matches (score > threshold) are surfaced. Updates every 6 hours via
Job Scout.

### 4. Human-in-the-Loop (HITL)

Before **any** outbound action (apply, email, DM):

1. Agent prepares a draft/preview.
2. WebSocket pushes `approval_required` event to the UI.
3. User reviews in a modal and approves or rejects.
4. If approved, agent executes; otherwise, skips.

**Default:** approval required. **Optional:** auto-approve (on user preference, capped).

### 5. Event-Driven Architecture

```
FastAPI Route
     ↓ (validates, authorizes)
Celery Task (async, retriable)
     ↓ (thin wrapper)
Agent.run(context)
     ├── LangGraph State Machine (tools, decisions, persistence)
     └── AgentResult(status, data, next_action)
     ↓ (Redis pub/sub)
WebSocket ConnectionManager
     ↓ (Redis pub/sub broadcast)
All connected clients (real-time update)
     ↓
Ntfy (mobile/desktop push)
```

---

## Technology Stack

| Layer | Tech | Why |
|-------|------|-----|
| **Backend** | Python 3.12, FastAPI, SQLAlchemy 2.0 (async) | Type-safe, async-native, mature ecosystem |
| **Database** | PostgreSQL 16 | ACID, full-text search, JSONB, JSON operators |
| **Vector DB** | Qdrant | Self-hosted, semantic search, small footprint |
| **Storage** | MinIO (S3-compatible) | Self-hosted, resumes + screenshots, S3 semantics |
| **Agents** | LangGraph, Ollama (local LLMs) | Stateful multi-step agents; local = privacy-first |
| **Task Queue** | Celery + Redis | Async, distributed, scheduling, retries |
| **LLMs** | `llama3.1:8b` (fast), `llama3.1:70b` (quality), `nomic-embed-text` | Open, self-hosted, good quality-speed tradeoff |
| **Browser Automation** | Playwright | Fast, stealth-capable, multi-browser |
| **Web Scraping** | Crawl4AI | LLM-native, extracts structured data |
| **Email** | aioimaplib, aiosmtplib | Async, native Python |
| **Frontend** | React 19, Vite, TypeScript, Tailwind, shadcn/ui | Type-safe, fast, component library |
| **Desktop** | Tauri 2 | Native Rust wrapper, small binary |
| **Mobile** | Expo (React Native) | JS, fast iteration |
| **DevOps** | Docker Compose, GitHub Actions, Prometheus, Grafana | Local dev, CI/CD, observability |

---

## Clean Architecture Mapping

The backend folder structure maps to Clean Architecture rings (inner → outer):

```
app/models/   ← Entities (SQLAlchemy ORM, data shapes)
app/schemas/  ← Request/response models (Pydantic)
app/agents/   ← Use cases (LangGraph decision logic)
app/api/      ← Interface adapters (FastAPI routes)
app/tasks/    ← Interface adapters (Celery tasks)
app/services/ ← Frameworks & drivers (integrations, no business logic)
app/core/     ← Frameworks & drivers (config, DB, auth, logging, exceptions)
```

**Dependency rule:** Inner never imports outer. API → Agents → Services → External.

---

## Data Flow (Job Application Example)

1. **Discover** — Beat triggers Job Scout every 6h.
   - Crawl4AI scrapes LinkedIn/Indeed/Wellfound/HN.
   - LLM extracts job structure (title, company, salary, description, stack).
2. **Match** — Job Scout's embedding + scoring step (per [`PLAN.md` §8.2](../../PLAN.md)).
   - Job description embedded with `nomic-embed-text`.
   - Scored against user profile in Qdrant.
   - Top matches (score > threshold) saved to PostgreSQL.
   - WebSocket push: "12 new jobs found."
3. **Apply** — User clicks "Apply" or auto-apply runs on schedule.
   - Resume Tailor rewrites resume to JD, loops on ATS score until ≥80%.
   - Generates cover letter, exports PDF/DOCX to MinIO.
   - Apply Agent drives Playwright through ATS.
   - Sends preview WebSocket event; user approves.
   - Submits form, screenshots confirmation to MinIO.
   - Application status = "applied".
4. **Outreach** — Referral Agent runs post-apply.
   - Finds LinkedIn employees at target company.
   - Scores by alumni/mutual connections/role match.
   - Drafts personalized message.
   - Queues 3-touch sequence (Day 0 / 5 / 10).
5. **Track** — Email Parser checks IMAP every 15m.
   - LLM classifies email (rejection, interview, offer, assessment, info request).
   - Matches to application by company/sender domain.
   - Updates status, creates StatusHistory event.
   - WebSocket push + Ntfy mobile notification.
6. **Close** — Deal Closer activated on warm reply.
   - Reads full thread, classifies intent (interested, scheduling, offer, negotiating).
   - Drafts contextual reply.
   - User edits + approves in-app.
   - Email sent; deal tracked.

---

## Invariants (Never Violate)

1. **HITL on all outbound actions.** No email sent, no application submitted without user approval.
2. **Privacy-first.** Local Ollama by default. Cloud is opt-in.
3. **No resume fabrication.** Only rephrase; never add experience.
4. **Type-safe end-to-end.** mypy strict, Pydantic v2, TypeScript strict.
5. **Clean Architecture.** Dependencies point inward. Services have zero business logic.

---

## Deployment

### Local (Docker Compose)

```bash
docker-compose up
# Starts: postgres, redis, qdrant, minio, api, worker, beat, flower, nginx
# Open: http://localhost:8080
```

### Cloud SaaS (Future, Phase 6)

- Hetzner CX32 (4 vCPU, 8GB RAM) per user shard.
- Hosted PostgreSQL (Supabase) or self-hosted.
- Groq API as fast LLM fallback (no local Ollama).
- Celery workers scale horizontally.
- Cloudflare CDN + DDoS protection.

---

## Key Decision Records (ADRs)

| ADR | Decision | Impact |
|-----|----------|--------|
| ADR-0001 | Documentation-driven dev OS | Every task has a journal entry + ADR for major decisions |
| ADR-0005 | Local-first Ollama (privacy) | Users can self-host; no vendor lock-in |
| ADR-0009 | HITL approval gates | Safety; users retain control |
| ADR-0012 | Clean Architecture | Testable, swappable, scalable |

---

## See Also

- [`project_rules.md`](project_rules.md) — Constraints and non-negotiable principles.
- [`sprint_status.md`](sprint_status.md) — Current sprint + tasks.
- [`coding_checklist.md`](coding_checklist.md) — Code review checklist.
- [`../../docs/architecture.md`](../../docs/architecture.md) — Full architecture documentation.
- [`../../PLAN.md`](../../PLAN.md) — Complete specification.
