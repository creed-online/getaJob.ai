# 🛠️ getJob.ai — Finalized Technology Stack & Architecture Reference (V2 Production)

> **Document Purpose:** This document defines the definitive, production-hardened technology stack for **getJob.ai**. Refactored to eliminate anti-bot failure points, introduce background worker queues, ensure database migration safety, and guarantee 100% dev/prod vector parity.

---

## 📊 1. Master Technology Stack Matrix

| Subsystem / Layer | Primary Technology | Core Libraries & Frameworks | Purpose & Architectural Rationale |
| :--- | :--- | :--- | :--- |
| **Frontend Framework** | **Next.js 14 (App Router)** | TypeScript, Tailwind CSS, Shadcn UI, Lucide Icons, TanStack Table, Recharts | Server-side rendering, accessible glassmorphic UI, responsive tables |
| **Backend API Gateway** | **FastAPI (Python 3.11+)** | Uvicorn (ASGI), AsyncIO, Pydantic v2, Pydantic-Settings | High-throughput async REST and Server-Sent Events (SSE) |
| **Background Task Queue** | **Arq + Redis** | `arq==0.26.1`, `redis==5.0.8` | Offloads heavy LLM calls, RAG embedding, and browser sessions from the HTTP thread |
| **Database & Migrations** | **PostgreSQL + pgvector** | `sqlalchemy[asyncio]==2.0.35`, `asyncpg==0.29.0`, `alembic==1.13.2` | Native vector search in PostgreSQL with Alembic version-controlled schema migrations |
| **Local Dev Infrastructure** | **Docker Compose** | Dockerized PostgreSQL 16 with `pgvector` extension + Redis 7 | 100% development and production vector search parity |
| **Security & Authentication** | **JWT & httpOnly Cookies** | `python-jose[cryptography]==3.3.0`, `passlib[bcrypt]==1.7.4` | Stateless auth stored in secure `httpOnly` `SameSite=Lax` cookies with Next.js path proxy |
| **Primary LLM (RAG & Tailor)**| **Google Gemini 2.0 Flash** | `google-genai==0.1.1` (model: `gemini-2.0-flash`) | 1,000,000 token context window for full Master Resume + JD analysis |
| **Fast LLM (Outreach Drafter)**| **Groq (Llama 3.3 70B)** | `groq==0.11.0` (model: `llama-3.3-70b-versatile`) | 300+ tokens/sec generation for real-time outreach messages and classification |
| **Local Vector Search** | **FastEmbed (`bge-small-en-v1.5`)**| `fastembed==0.3.6`, `numpy` | 384-dimensional dense semantic embeddings computed in-memory |
| **Lexical Search (ATS Match)** | **Rank-BM25** | `rank-bm25==0.2.2` | Exact keyword and technical skill match density calculation |
| **Global Discovery APIs** | **Adzuna + ATS Feeds + RSS** | Adzuna API, Greenhouse JSON, Lever API, Ashby JSON, HN Firebase API, Unstop/Devpost RSS | **100% reliable, zero bot-ban risk, global index + company seed registry** |
| **Document Ingestion** | **Python Document Suite** | `pdfplumber==0.11.4`, `pypdf==4.3.1`, `python-docx==1.1.2`, `httpx==0.27.2` | Ingests Master Resumes, GitHub READMEs, research papers, and transcripts with size & MIME check |
| **ATS Resume Compilation** | **Typst Engine** | `typst-py==0.11.0` / `typst`, `jinja2==3.1.4` | Sub-50ms single-column machine-readable ATS PDF output |
| **1-Click & Assisted Apply** | **Playwright Async Agent** | `playwright==1.47.0`, `asyncio`, FastAPI SSE Stream | Direct auto-apply for Greenhouse/Lever + Assisted prefill copilot for complex forms |
| **Referral Search & Outreach** | **LinkedIn Search Generator** | `httpx`, `groq` / `gemini`, URL query encoders | Generates targeted LinkedIn search URLs + customized cold connection messages (100% legal, zero PII liability) |
| **Analytics & Export** | **Data Suite** | `openpyxl==3.1.5`, `pandas==2.2.2` | Formatted Excel (`.xlsx`) and structured JSON data export |
| **Logging & Quality** | **Pytest & Structured Logs** | `pytest==8.3.3`, `pytest-asyncio==0.24.0`, `pytest-cov==5.0.0`, `structlog==24.4.0` | Automated test suite with JSON structured logging |

---

## 💻 2. Detailed Subsystem Specifications

### 2.1 Discovery Engine: Global Index + Company Seed Registry
To guarantee a rich, fresh feed without scraping:
1. **Global Search Index (Adzuna API):** Provides a search index of thousands of fresh active postings worldwide.
2. **Company Seed Registry (`company_registry.py`):** Pre-curated dataset of 500+ top tech companies and startups with their Greenhouse (`boards-api.greenhouse.io/v1/boards/{company}/jobs`) and Lever (`api.lever.co/v0/postings/{company}`) board slugs.
3. **Tech Community Signals:** Official Hacker News "Who is Hiring?" Firebase JSON API + Unstop and Devpost RSS feeds.
4. **Stale Opportunity Decay Rule:**
   $$\text{Stale Penalty} = \begin{cases} 0.5 & \text{if } \text{posted\_days} > 21 \\ 0.2 & \text{if } \text{posted\_days} > 45 \\ 1.0 & \text{otherwise (Fresh)} \end{cases}$$

---

### 2.2 Legally Clean Referral Discovery (LinkedIn Search Generator)
* **How It Works:**
  * The AI extracts the company name and target role personas (e.g. *"Technical Recruiter at Stripe"*, *"Engineering Manager at Stripe"*, *"Software Engineer at Stripe alumni of IIT/BITS"*).
  * Generates direct, encoded LinkedIn search links:  
    `https://www.linkedin.com/search/results/people/?keywords=Technical+Recruiter+Stripe`
  * The user clicks the link to view the real-time search inside **their own authenticated LinkedIn session**.
  * The AI generates a customized, 50-word cold outreach message ready for the user to copy-paste.
* **Benefits:** Zero personal data scraped, zero DPDP/GDPR liability, 100% accurate real-time results.

---

### 2.3 Hybrid Auto-Apply Strategy
1. **1-Click Autonomous Apply (Greenhouse & Lever):**
   * Greenhouse and Lever have stable, predictable DOM structures. Playwright automatically maps candidate fields, uploads the Typst PDF, and submits.
2. **Assisted Apply / Copilot Mode (Workday, Taleo & Custom Multi-Step Portals):**
   * Generates a pre-filled **Answer Pack** containing all screening answers, tailored Google XYZ bullets, and the downloadable PDF.
   * Provides 1-click clipboard triggers so the user never spends time retyping answers.

---

### 2.4 Document Validation & Security Guardrails
* **Resume Upload Guardrails:** Strict validation rejecting files $> 5\text{MB}$ and non-PDF/DOCX MIME types (`application/pdf`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document`).
* **Authentication Security:** JWT cookies set with `httpOnly=True`, `SameSite=Lax`, `Secure=True`, with Next.js path rewrites `/api/:path* ➔ http://backend:8000/api/:path*`.
* **Token Metering (`CreditMeterService`):** Tracks per-user credit consumption (`credits_remaining`) on every auto-apply and tailoring action.

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

# AI & LLM SDKs
google-genai==0.1.1
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
