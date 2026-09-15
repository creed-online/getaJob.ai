# 🛠️ getJob.ai — Finalized Technology Stack & Architecture Reference (Refactored)

> **Document Purpose:** This document defines the robust, production-grade technology stack for **getJob.ai**. Refactored to eliminate anti-bot failure points, introduce background worker queues, ensure database migration safety, and guarantee 100% dev/prod vector parity.

---

## 📊 1. Master Technology Stack Matrix

| Subsystem / Layer | Primary Technology | Core Libraries & Frameworks | Purpose & Architectural Rationale |
| :--- | :--- | :--- | :--- |
| **Frontend Framework** | **Next.js 14 (App Router)** | TypeScript, Tailwind CSS, Shadcn UI, Lucide Icons, TanStack Table, Recharts | Server-side rendering, accessible glassmorphic UI, responsive tables |
| **Backend API Gateway** | **FastAPI (Python 3.11+)** | Uvicorn (ASGI), AsyncIO, Pydantic v2, Pydantic-Settings | High-throughput async REST and Server-Sent Events (SSE) |
| **Background Task Queue** | **Arq / Celery + Redis** | `arq==0.26.1` (or `celery==5.4.0`), `redis==5.0.8` | Offloads heavy LLM calls, RAG embedding, and browser sessions from the HTTP thread |
| **Database & Migrations** | **PostgreSQL + pgvector** | `sqlalchemy[asyncio]==2.0.35`, `asyncpg==0.29.0`, `alembic==1.13.2` | Native vector search in PostgreSQL with Alembic version-controlled schema migrations |
| **Local Dev Parity** | **Docker Compose** | Dockerized PostgreSQL 16 with `pgvector` extension + Redis | 100% development and production vector search parity |
| **Security & Authentication** | **JWT & httpOnly Cookies** | `python-jose[cryptography]==3.3.0`, `passlib[bcrypt]==1.7.4` | Stateless auth stored in secure `httpOnly` SameSite cookies (immune to XSS) |
| **Primary LLM (RAG & Tailor)**| **Google Gemini 2.0 Flash** | `google-genai` (Latest SDK) | 1,000,000 token context window for full Master Resume + JD analysis |
| **Fast LLM (Outreach Drafter)**| **Groq (Llama 3.3 70B)** | `groq==0.11.0` | 300+ tokens/sec generation for real-time outreach messages and classification |
| **Local Vector Search** | **FastEmbed (`bge-small-en-v1.5`)**| `fastembed==0.3.6`, `numpy` | 384-dimensional dense semantic embeddings computed in-memory |
| **Lexical Search (ATS Match)** | **Rank-BM25** | `rank-bm25==0.2.2` | Exact keyword and technical skill match density calculation |
| **Opportunity Discovery Ingestion**| **Direct Public APIs & Feeds** | Greenhouse API, Lever API, Ashby JSON, Adzuna API, HN Firebase API, Unstop/Devpost RSS | **100% reliable, zero bot-ban risk, clean structured JSON** |
| **Document Ingestion** | **Python Document Suite** | `pdfplumber==0.11.4`, `pypdf==4.3.1`, `python-docx==1.1.2`, `httpx==0.27.2` | Ingests Master Resumes, GitHub READMEs, research papers, and transcripts |
| **ATS Resume Compilation** | **Typst Engine** | `typst-py==0.11.0` / `typst`, `jinja2==3.1.4` | Sub-50ms single-column machine-readable ATS PDF output |
| **1-Click & Assisted Auto-Apply**| **Playwright Async Agent** | `playwright==1.47.0`, `asyncio`, FastAPI SSE Stream | Direct auto-apply for Greenhouse/Lever + Assisted prefill copilot for complex forms |
| **5-Contact Referral Finder** | **Contact Discovery & Outreach** | `httpx`, `groq` / `gemini`, regex pattern enrichment | Discovers 5 key company contacts with contact info and 1-click copy short messages |
| **Analytics & Export** | **Data Suite** | `openpyxl==3.1.5`, `pandas==2.2.2` | Formatted Excel (`.xlsx`) and structured JSON data export |
| **Testing & Quality** | **Pytest Suite** | `pytest==8.3.3`, `pytest-asyncio==0.24.0`, `pytest-cov==5.0.0` | Automated unit and integration test suite |

---

## 💻 2. Detailed Subsystem Specifications

### 2.1 Background Task Queue Architecture (Redis + Arq)
To prevent server memory exhaustion and long HTTP timeouts:
* **HTTP API Worker (FastAPI):** Accepts requests, validates inputs, and immediately enqueues long-running jobs.
* **Background Worker (`arq`):**
  * `job_ingest_resume_rag`: Chunks and vectorizes Master Resume.
  * `job_tailor_resume_typst`: Runs Gemini bullet optimizer and compiles Typst PDF.
  * `job_run_auto_apply`: Launches headless Playwright session.
  * `job_sync_discovery_feeds`: Ingests Greenhouse, Lever, Unstop, and Hacker News feeds.

---

### 2.2 Direct Public API Ingestion (Replacing Fragile Scraping)
Instead of fighting anti-bot defenses on LinkedIn/Indeed, getJob.ai queries verified, public structured endpoints:
1. **Greenhouse Public Boards:** `https://boards-api.greenhouse.io/v1/boards/{company}/jobs` (Returns 100% valid JSON per company).
2. **Lever Public Postings:** `https://api.lever.co/v0/postings/{company}`.
3. **Ashby Job Boards:** `https://jobs.ashbyhq.com/api/non-app-graphql`.
4. **Hacker News "Who is Hiring?":** Official Hacker News Firebase API (`https://hacker-news.firebaseio.com/v0/`).
5. **Adzuna & RemoteOK APIs:** Free developer API feeds.
6. **Hiring Hackathons & Competitions:** Official RSS/JSON feeds from Unstop, Devpost, and Kaggle.

---

### 2.3 Hybrid Auto-Apply Strategy
1. **1-Click Autonomous Apply (Greenhouse & Lever):**
   * Greenhouse and Lever have stable, predictable DOM structures. Playwright automatically maps candidate fields, uploads the Typst PDF, and submits.
2. **Assisted Apply / Copilot Mode (Workday, Taleo & Custom Multi-Step Portals):**
   * Generates a pre-filled **Answer Pack** containing all screening answers, tailored Google XYZ bullets, and the downloadable PDF.
   * Provides 1-click clipboard triggers so the user never spends time retyping answers.

---

### 2.4 Deterministic ATS Parseability Engine
* **Code-Based Structural Verification:** Checks if the PDF contains selectable text, standard section headers (`Education`, `Experience`, `Skills`), parseable dates (`MMM YYYY`), and single-column formatting.
* **LLM Keyword & Impact Analysis:** Gemini/Groq evaluates semantic keyword overlap against the JD and scores bullet impact using the Google XYZ formula.

---

### 2.5 Security & Token Metering
* **`httpOnly` Cookies:** JWT access and refresh tokens are stored in secure, encrypted `httpOnly` cookies with `SameSite=Strict`.
* **Token Metering (`CreditMeterService`):** Tracks per-user credit consumption (`credits_remaining`) on every auto-apply and tailoring action.
* **User-Provided API Key Override:** Users can optionally supply their own free Google Gemini API key in `/settings` for unlimited personal operations.

---

## 📦 3. Complete Updated `backend/requirements.txt`

```ini
# Core Web Framework & Server
fastapi==0.115.0
uvicorn[standard]==0.30.6
pydantic==2.8.2
pydantic-settings==2.4.0

# Database & Migrations
sqlalchemy[asyncio]==2.0.35
asyncpg==0.29.0
alembic==1.13.2

# Background Worker & Redis
arq==0.26.1
redis==5.0.8

# Security & Authentication
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.9
email-validator==2.2.0

# AI & LLM SDKs (100% Free Tiers)
google-genai>=0.1.1
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

# Analytics & Export
openpyxl==3.1.5
pandas==2.2.2

# Testing & Quality Assurance
pytest==8.3.3
pytest-asyncio==0.24.0
pytest-cov==5.0.0
```
