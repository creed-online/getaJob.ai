# 🛠️ getJob.ai — Finalized Technology Stack & Architecture Reference

> **Document Purpose:** This document defines the definitive, production-grade technology stack for **getJob.ai (getaJob.ai)**. It maps every subsystem to its specific frameworks, libraries, and exact packages, detailing the rationale and free-tier compatibility for each choice.

---

## 📊 1. Master Technology Stack Matrix

| Subsystem / Layer | Primary Technology | Core Libraries & Frameworks | Free-Tier & Cost Profile |
| :--- | :--- | :--- | :--- |
| **Frontend UI & Client** | **Next.js 14 (App Router)** | TypeScript, Tailwind CSS, Shadcn UI, Lucide Icons, TanStack Table, Recharts | Open Source (100% Free) |
| **Backend API & Orchestration** | **FastAPI (Python 3.11+)** | Uvicorn (ASGI), AsyncIO, Pydantic v2, Pydantic-Settings | Open Source (100% Free) |
| **Authentication & Multi-Tenancy** | **JWT & Bcrypt** | `python-jose`, `passlib[bcrypt]`, `email-validator` | Open Source (100% Free) |
| **Primary LLM & Reasoning Engine** | **Google Gemini 2.0 Flash / 1.5 Flash** | `google-genai` SDK | **100% FREE** (15 RPM, 1M context, 1.5k req/day) |
| **Ultra-Fast LLM & Classification** | **Groq Cloud (Llama 3.3 70B Versatile)**| `groq` SDK | **100% FREE** (30 RPM, 300+ tokens/sec) |
| **Vector Embeddings & RAG Search** | **FastEmbed (Local `bge-small-en-v1.5`)**| `fastembed`, `numpy`, `rank-bm25` | **100% FREE** (Runs locally in RAM/CPU) |
| **Document Ingestion & Parsers** | **Python Document Suite** | `pdfplumber`, `pypdf`, `python-docx`, `httpx` | Open Source (100% Free) |
| **Multi-Source Discovery Scrapers** | **Playwright + Parsers** | `playwright`, `playwright-stealth`, `beautifulsoup4`, `feedparser`, `lxml` | Open Source (100% Free) |
| **ATS Resume & CV Compilation** | **Typst Engine** | `typst-py` / `typst`, `jinja2` | **100% FREE** (Sub-50ms native compilation) |
| **Autonomous 1-Click Auto-Apply** | **Playwright Async Agent** | `playwright`, `asyncio`, FastAPI Server-Sent Events (SSE) | Open Source (100% Free) |
| **Database & Persistence Layer** | **SQLAlchemy 2.0 Async** | `aiosqlite` (Local Dev), `asyncpg` + `pgvector` (PostgreSQL) | Open Source (100% Free) |
| **Analytics & Data Export** | **Data Suite** | `openpyxl`, `pandas` | Open Source (100% Free) |
| **Testing & Quality Assurance** | **Pytest Suite** | `pytest`, `pytest-asyncio`, `pytest-cov`, `httpx` | Open Source (100% Free) |

---

## 💻 2. Layer-by-Layer Technical Breakdown

### 2.1 Frontend Architecture (`frontend/`)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          Next.js 14 App Router                          │
│   ├── (auth)/login & register         ├── profile/ (Resume & GitHub)    │
│   ├── dashboard/ (Opportunity Radar)  ├── tailor/[id] (ATS Studio)      │
│   ├── referrals/[id] (5-Contact Hub)  └── tracker/ (Kanban & Table)     │
└─────────────────────────────────────────────────────────────────────────┘
```

* **Core Framework:** **Next.js 14 (React 18 + TypeScript)**
  * *Why:* Server-Side Rendering (SSR) for fast initial load, file-system routing, native Server Actions, and optimal developer experience.
* **Styling & Design System:** **Tailwind CSS + Shadcn UI (Radix UI Primitives)**
  * *Libraries:* `tailwindcss@3.4.10`, `@radix-ui/react-dialog`, `@radix-ui/react-dropdown-menu`, `@radix-ui/react-tabs`, `@radix-ui/react-toast`, `lucide-react@0.439.0`.
  * *Why:* Clean, dark-mode-first aesthetic with accessible, fully customizable UI components.
* **State Management:** **Zustand (`zustand@4.5.5`)**
  * *Why:* Minimalist, boilerplate-free state store for managing user session, active tenant data, and live application progress.
* **Form Handling & Validation:** **React Hook Form + Zod**
  * *Libraries:* `react-hook-form@7.53.0`, `@hookform/resolvers@3.9.0`, `zod@3.23.8`.
  * *Why:* Type-safe form validation with zero unnecessary re-renders.
* **Data Table & Analytics Widgets:** **TanStack Table + Recharts**
  * *Libraries:* `@tanstack/react-table@8.20.5`, `recharts@2.12.7`.
  * *Why:* Powerful client-side sorting, filtering, and pagination for the Application Tracker, plus responsive gauges for ATS score meters.
* **HTTP Client & Real-Time Streaming:** **Axios + Native EventSource (SSE)**
  * *Libraries:* `axios@1.7.7`.
  * *Why:* Interceptors for automatic JWT Bearer token injection, and native SSE support for live streaming Playwright browser progress.

---

### 2.2 Backend API & Core Services (`backend/`)

* **Framework:** **FastAPI (`fastapi==0.115.0`)**
  * *ASGI Server:* `uvicorn[standard]==0.30.6`.
  * *Why:* High concurrency with native `async/await`, automatic OpenAPI/Swagger documentation, and native Pydantic validation.
* **Data Validation & Configuration:** **Pydantic v2 (`pydantic==2.8.2`, `pydantic-settings==2.4.0`)**
  * *Why:* High-speed C-based validation for request payloads, environment variables, and LLM structured schemas.
* **Authentication & Multi-Tenant Security:**
  * *Libraries:* `python-jose[cryptography]==3.3.0` (JWT token generation & validation), `passlib[bcrypt]==1.7.4` (salted password hashing), `email-validator==2.2.0`.
  * *Why:* Industry-standard stateless security enforcing row-level tenant separation on every database query.

---

### 2.3 AI, LLM & RAG Engine (100% Free Tier)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              AI CORE                                    │
│                                                                         │
│  ┌───────────────────────┐  ┌───────────────────────┐  ┌─────────────┐  │
│  │ Gemini 2.0 Flash      │  │ Groq (Llama 3.3 70B)  │  │ FastEmbed   │  │
│  │ • 1M Context Window   │  │ • 300+ tokens/sec     │  │ • Local RAM │  │
│  │ • Profile Structuring │  │ • ATS Score Classif.  │  │ • BGE-Small │  │
│  │ • Resume & CV Tailor  │  │ • Referral Messages   │  │ • 384-dim   │  │
│  └───────────────────────┘  └───────────────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

1. **Primary Reasoning & Document Synthesis Engine:**
   * **Google Gemini 2.0 Flash / 1.5 Flash (`google-genai==0.1.1`)**
   * *Role:* Ingests entire Master Resumes, GitHub READMEs, and complex JDs in a single prompt (up to 1,000,000 tokens). Generates strictly typed JSON outputs via `response_schema`.
   * *Cost:* **100% Free** on Google AI Studio (15 RPM, 1,500 requests/day).
2. **High-Speed Classifier & Referral Drafter:**
   * **Groq Cloud — Llama 3.3 70B Versatile (`groq==0.10.0`)**
   * *Role:* Sub-second token generation for real-time ATS scoring reports, missing keyword detection, and concise (<75 words) cold outreach messages.
   * *Cost:* **100% Free** (30 RPM, 14,400 requests/day).
3. **Local Vector Embeddings Engine (Zero External API Dependency):**
   * **FastEmbed (`fastembed==0.3.4`)** running `bge-small-en-v1.5` (or `all-MiniLM-L6-v2`)
   * *Role:* Converts candidate work history and projects into 384-dimensional dense semantic vectors directly in local Python memory.
   * *Cost:* **100% Free** (Runs on CPU/GPU with zero API latency and zero cost).
4. **Lexical Keyword Matcher:**
   * **Rank-BM25 (`rank-bm25==0.2.2`)**
   * *Role:* Performs sparse exact-keyword matching for mandatory technical skills (e.g., `"Kubernetes"`, `"PyTorch"`, `"FastAPI"`) to complement dense semantic embeddings.

---

### 2.4 Document Ingestion & Parsers

* **PDF Parsing & Layout Analysis:** `pdfplumber==0.11.4`, `pypdf==4.3.1`.
  * *Role:* Extracts text, tables, dates, and section boundaries from uploaded Master Resumes, research papers, and university transcripts.
* **Word Document Parsing:** `python-docx==1.1.2`.
  * *Role:* Parses `.docx` format resumes.
* **GitHub API & Repo Ingestion:** `httpx==0.27.2`.
  * *Role:* Asynchronously fetches repository lists, language distributions, commit activity, and `README.md` files for proof-of-work extraction.

---

### 2.5 Multi-Source Discovery Scrapers

* **Dynamic JS Web Scraping:** `playwright==1.47.0`, `playwright-stealth==1.0.6`.
  * *Role:* Headless Chromium automation for scraping dynamic job boards (LinkedIn public listings, Indeed, Wellfound).
* **HTML & RSS Feed Ingestion:** `beautifulsoup4==4.12.3`, `lxml==5.3.0`, `feedparser==6.0.11`.
  * *Role:* High-speed XML/HTML parsing for Unstop hackathons, Devpost challenges, CERN/NSF fellowship feeds, and Hacker News ("Who is hiring?").

---

### 2.6 ATS Resume & Cover Letter Compilation Engine

* **Engine:** **Typst (`typst-py==0.11.0` / `typst`) + Jinja2 (`jinja2==3.1.4`)**
  * *Why Typst over LaTeX?*
    * **100x Faster:** Compiles in **< 50 milliseconds** (LaTeX takes 2–5 seconds).
    * **Zero Bloat:** Lightweight Python binary without needing a 4GB TeX Live installation.
    * **Pristine ATS Output:** Emits clean, single-column, standard-font PDFs with perfect text extractability (guaranteed 95+ score on Workday, Taleo, Greenhouse ATS).

---

### 2.7 Autonomous 1-Click Auto-Apply Browser Engine

* **Framework:** **Playwright Python Async (`playwright==1.47.0`)**
  * *Form Field Mapper:* Heuristic & semantic DOM attribute matching (`id`, `name`, `aria-label`, `placeholder`, `autocomplete`).
  * *Portal Adapters:* Custom handler classes for **Greenhouse**, **Lever**, **Workday**, and **Generic/Custom forms**.
  * *Human-in-the-Loop Pause:* Utilizes `asyncio.Event` with Server-Sent Events (SSE) to pause submission and prompt the user in the UI when encountering unknown custom screening questions.

---

### 2.8 Database & Persistence Layer

* **ORM & Database Driver:** **SQLAlchemy 2.0 Async (`sqlalchemy[asyncio]==2.0.34`)**
  * *Development DB:* **SQLite with async driver (`aiosqlite==0.20.0`)** — Zero configuration, instant local startup.
  * *Production DB:* **PostgreSQL (`asyncpg==0.29.0`) + `pgvector`** — Robust relational storage with native vector search.
* **File Storage:** Local structured filesystem (`backend/storage/generated/`) designed with S3-compatible path abstractions (`/resumes/{user_id}/{doc_id}.pdf`).

---

### 2.9 Analytics & Export Engine

* **Excel File Generation:** **`openpyxl==3.1.5`** + **`pandas==2.2.2`**
  * *Role:* Asynchronously compiles applied applications into formatted `.xlsx` spreadsheets with auto-styled headers and status color highlights.
* **JSON Export:** Native Python `json` serializer with ISO-8601 timestamps.

---

### 2.10 Testing & Code Quality Suite

* **Testing Framework:** `pytest==8.3.2`, `pytest-asyncio==0.24.0`, `pytest-cov==5.0.0`.
* **API Testing Client:** `httpx==0.27.2` (`AsyncClient`).
* **Linter & Formatter:** **Ruff** (Python) + **ESLint / Prettier** (TypeScript).

---

## 📦 3. Complete Dependency Manifest

### Backend `requirements.txt`
```ini
# Core Web Framework
fastapi==0.115.0
uvicorn[standard]==0.30.6
pydantic==2.8.2
pydantic-settings==2.4.0

# Database & ORM
sqlalchemy[asyncio]==2.0.34
aiosqlite==0.20.0
asyncpg==0.29.0

# Security & Authentication
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.9
email-validator==2.2.0

# AI & LLM SDKs (100% Free Tiers)
google-genai==0.1.1
groq==0.10.0
fastembed==0.3.4
rank-bm25==0.2.2

# Document Parsers & Compilers
pdfplumber==0.11.4
pypdf==4.3.1
python-docx==1.1.2
typst==0.11.0
jinja2==3.1.4

# Web Scraping & Browser Automation
playwright==1.47.0
playwright-stealth==1.0.6
beautifulsoup4==4.12.3
lxml==5.3.0
feedparser==6.0.11
httpx==0.27.2

# Analytics & Export
openpyxl==3.1.5
pandas==2.2.2

# Testing & Quality
pytest==8.3.2
pytest-asyncio==0.24.0
pytest-cov==5.0.0
```

### Frontend `package.json` (Key Dependencies)
```json
{
  "dependencies": {
    "next": "14.2.10",
    "react": "18.3.1",
    "react-dom": "18.3.1",
    "typescript": "5.5.4",
    "tailwindcss": "3.4.10",
    "@radix-ui/react-dialog": "^1.1.1",
    "@radix-ui/react-dropdown-menu": "^2.1.1",
    "@radix-ui/react-tabs": "^1.1.0",
    "@radix-ui/react-toast": "^1.2.1",
    "lucide-react": "^0.439.0",
    "zustand": "^4.5.5",
    "axios": "^1.7.7",
    "react-hook-form": "^7.53.0",
    "@hookform/resolvers": "^3.9.0",
    "zod": "^3.23.8",
    "@tanstack/react-table": "^8.20.5",
    "recharts": "^2.12.7"
  }
}
```

---

## 🔒 4. Free-Tier API Key Configuration Reference

| Service | Environment Variable | Where to get it (100% Free) | Quotas / Limits |
| :--- | :--- | :--- | :--- |
| **Google Gemini API** | `GEMINI_API_KEY` | [Google AI Studio](https://aistudio.google.com/) | 15 RPM, 1M context, 1,500 req/day |
| **Groq Cloud API** | `GROQ_API_KEY` | [Groq Console](https://console.groq.com/) | 30 RPM, 14,400 req/day |
| **Local FastEmbed** | *None (Local)* | *Installed via pip* | Unlimited (Runs in RAM/CPU) |
| **GitHub API (Optional)** | `GITHUB_TOKEN` | [GitHub Personal Access Token](https://github.com/settings/tokens) | 5,000 req/hour (60 req/hr unauthenticated) |
