# 🛠️ getJob.ai — Finalized Technology Stack & Architecture Reference (V2.1 Production Final)

> **Document Purpose:** This document defines the production-hardened technology stack and deployment architecture for **getJob.ai**.

---

## 📊 1. Master Technology Stack Matrix

| Subsystem / Layer | Primary Technology | Core Libraries & Frameworks | Purpose & Architectural Rationale |
| :--- | :--- | :--- | :--- |
| **Frontend Framework** | **Next.js 14 (App Router)** | TypeScript, Tailwind CSS, Shadcn UI, Lucide Icons, TanStack Table, Recharts | Server-side rendering, accessible glassmorphic UI, responsive tables |
| **Backend API Gateway** | **FastAPI (Python 3.11+)** | Uvicorn (ASGI), AsyncIO, Pydantic v2, `slowapi==0.1.9` | Async REST & SSE with brute-force rate-limiting (`5/min` on login) |
| **Background Task Queue** | **Arq + Redis (with Pub/Sub IPC)** | `arq==0.26.1`, `redis==5.0.8` | Offloads heavy LLM calls, RAG embedding & Playwright sessions with cross-process Redis Pub/Sub |
| **Database & Migrations** | **PostgreSQL 16 + pgvector** | `sqlalchemy[asyncio]==2.0.35`, `asyncpg==0.29.0`, `alembic==1.13.2` | Native vector search in PostgreSQL with Alembic version-controlled schema migrations |
| **Local Dev Infrastructure** | **Docker Compose** | Dockerized PostgreSQL 16 (`pgvector`) + Redis 7 | 100% development and production vector search parity |
| **Security & Authentication** | **JWT, httpOnly Cookies & Email**| `python-jose[cryptography]==3.3.0`, `passlib[bcrypt]==1.7.4`, `resend==2.4.0` | Stateless auth in `httpOnly` `SameSite=Lax` cookies + Resend for password resets |
| **Primary LLM (RAG & Tailor)**| **Google Gemini 2.0 Flash** | `google-genai==2.23.0` (model: `gemini-2.0-flash`) | 1,000,000 token context window for full Master Resume + JD analysis |
| **Fast LLM (Outreach Drafter)**| **Groq (Llama 3.3 70B)** | `groq==0.11.0` (model: `llama-3.3-70b-versatile`) | 300+ tokens/sec generation for real-time outreach messages (<75 words) |
| **Local Vector Search** | **FastEmbed (`bge-small-en-v1.5`)**| `fastembed==0.3.6`, `numpy` | 384-dimensional dense semantic embeddings computed in-memory |
| **Lexical Search (ATS Match)** | **Rank-BM25** | `rank-bm25==0.2.2` | Exact keyword and technical skill match density calculation |
| **Global Discovery APIs** | **Adzuna + ATS Feeds + RSS** | Adzuna Global API, Greenhouse JSON, Lever API, HN Firebase API, Unstop/Devpost RSS | **100% reliable, zero bot-ban risk, global index + company seed registry** |
| **Document Ingestion** | **Python Document Suite** | `pdfplumber==0.11.4`, `pypdf==4.3.1`, `python-docx==1.1.2`, `httpx==0.27.2` | Ingests Master Resumes, GitHub READMEs, and papers with 5MB & MIME validation |
| **ATS Resume Compilation** | **Typst Engine** | `typst-py==0.11.0` / `typst`, `jinja2==3.1.4` | Sub-50ms single-column machine-readable ATS PDF output |
| **Hybrid Auto-Apply Engine** | **Playwright Async + Copilot**| `playwright==1.47.0`, `asyncio`, Redis Pub/Sub SSE Bridge | 1-Click for Greenhouse/Lever, Copilot Answer Pack for Workday, Direct Link for Adzuna |
| **Referral Search & Outreach** | **LinkedIn Search Generator** | `httpx`, `groq` / `gemini`, URL query encoders | Generates targeted LinkedIn search URLs + customized cold connection messages (<75 words) |
| **Analytics & Export** | **Data Suite** | `openpyxl==3.1.5`, `pandas==2.2.2` | Formatted Excel (`.xlsx`) and structured JSON data export |
| **Logging & Quality** | **Pytest & Structured Logs** | `pytest==8.3.3`, `pytest-asyncio==0.24.0`, `pytest-cov==5.0.0`, `structlog==24.4.0` | Automated test suite with JSON structured logging |

---

## 🚀 2. Production Deployment Blueprint

```
┌────────────────────────────────────────────────────────────────────────┐
│                        PRODUCTION ARCHITECTURE                         │
│                                                                        │
│  [ FRONTEND ] ➔ Vercel (Next.js 14 App Router)                         │
│                  Path rewrite: /api/:path* ➔ Railway Backend           │
│                                                                        │
│  [ BACKEND ]  ➔ Railway / Render / Fly.io                              │
│                  Docker Image: mcr.microsoft.com/playwright/python     │
│                  Runs: 1) FastAPI ASGI Web Server                      │
│                        2) Arq Background Worker (Playwright & RAG)     │
│                                                                        │
│  [ DATABASE ] ➔ Supabase / Neon (Managed PostgreSQL 16 + pgvector)     │
│                                                                        │
│  [ QUEUE ]    ➔ Upstash Redis (Serverless Free Tier) / Redis 7         │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 💻 3. Key Technical Specifications

### 3.1 Redis Pub/Sub Cross-Process SSE Bridge
Because FastAPI and the `arq` worker run in separate OS processes:
1. **Worker ➔ Client (Progress Stream):** `job_run_auto_apply` publishes step events to Redis channel `apply_progress:{session_id}`. The FastAPI endpoint `GET /api/apply/stream/{session_id}` subscribes and yields Server-Sent Events (SSE) to the browser.
2. **Client ➔ Worker (Question Answer):** When encountering an unknown question, the worker publishes a prompt event and blocks on `r.brpop(f"apply_answer:{session_id}", timeout=120)`. When the user submits the answer via `POST /api/apply/answer-prompt`, FastAPI pushes to `r.lpush(f"apply_answer:{session_id}", answer)`, instantly resuming the worker.

---

### 3.2 Opportunity Apply Methods (`apply_method` Enum)
To ensure the UI renders the correct action button:
* **`AUTONOMOUS`** (Greenhouse & Lever): Supported for 1-Click Auto-Apply via Playwright.
* **`ASSISTED`** (Workday, Taleo, Custom forms): Generates a pre-filled Copilot Answer Pack with 1-click clipboard triggers.
* **`EXTERNAL_REDIRECT`** (Adzuna / Aggregator links): Opens direct application link in new tab + auto-logs card in Tracker.

---

### 3.3 Posting Age Decay Calculation (Fixed Order)
```python
def compute_stale_decay(posted_days: int) -> float:
    # Must evaluate oldest age thresholds FIRST
    if posted_days > 45:
        return 0.2  # 80% confidence penalty for postings > 45 days
    elif posted_days > 21:
        return 0.5  # 50% confidence penalty for postings > 21 days
    return 1.0      # Fresh listing (<= 21 days)
```

---

## 📦 4. Complete Updated `backend/requirements.txt`

```ini
# Core Web Framework & Server
fastapi==0.115.0
uvicorn[standard]==0.30.6
pydantic==2.8.2
pydantic-settings==2.4.0
slowapi==0.1.9

# Database & Migrations
sqlalchemy[asyncio]==2.0.35
asyncpg==0.29.0
alembic==1.13.2

# Background Worker & Redis
arq==0.26.1
redis==5.0.8

# Security, Auth & Email
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.9
email-validator==2.2.0
resend==2.4.0

# AI & LLM SDKs (Latest Releases)
google-genai==2.23.0
groq==0.11.0
fastembed==0.3.6
rank-bm25==0.2.2

# Document Parsers & Compilers
pdfplumber==0.11.4
pypdf==4.3.1
python-docx==1.1.2
typst==0.11.0
jinja2==3.1.4

# API Ingestion & Browser Automation
playwright==1.47.0
beautifulsoup4==4.12.3
lxml==5.3.0
feedparser==6.0.11
httpx==0.27.2

# Analytics, Logging & Export
openpyxl==3.1.5
pandas==2.2.2
structlog==24.4.0

# Testing & Quality Assurance
pytest==8.3.3
pytest-asyncio==0.24.0
pytest-cov==5.0.0
```
