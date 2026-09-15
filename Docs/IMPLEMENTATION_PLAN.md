# 🛰️ getJob.ai — Master Technical Implementation Plan & Execution Tracker (V2 Production)

```
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│  PROJECT: getJob.ai (getaJob.ai)                     STAGE: Production-Ready Roadmap                   │
│  CORE ARCHITECTURE: 3D Multi-Tenant AI Career Copilot LAST UPDATED: 2026-09-15                          │
│  TOTAL PHASES: 7 (MVP Core: Phases 1, 2, 3, 4, 7)     TOTAL TASKS: 88 (MVP: 48 Tasks)                   │
│  OVERALL PROGRESS: [░░░░░░░░░░░░░░░░░░░░] 0% (0/88 Tasks Completed)                                   │
└────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

> [!IMPORTANT]
> **V2 PRODUCTION REFINEMENTS:**
> * **Zero-Ban Discovery:** Includes `company_registry.py` (500+ curated Greenhouse/Lever boards) + **Adzuna Global API** + Hacker News API + Unstop/Devpost RSS.
> * **Realistic Stale Filter:** Based on verified posting age decay (`posted_days > 21` ➔ 50% penalty, `> 45` ➔ 80% penalty) rather than unavailable applicant counts.
> * **Legally Clean Referral Engine:** Generates targeted LinkedIn search URLs (opens in user's own session) + AI outreach drafts, eliminating all PII/DPDP liability.
> * **Background Worker & DB Migrations:** `Redis + arq` worker queue, `alembic` migrations, and Dockerized PostgreSQL 16 with `pgvector`.
> * **Security & Metering:** JWT in `httpOnly` `SameSite=Lax` cookies, file upload size/MIME guardrails, and `CreditMeterService`.

---

## 📊 Master Phase Summary & Execution Radar

| Phase | Module Name | Scope Tier | Lead Area | Total Tasks | Status |
| :---: | :--- | :---: | :--- | :---: | :---: |
| **01** | **Scaffolding, Migrations, Queue & Auth** | `MVP Core - P0` | Backend / Infra / 3D | **12** | `⚪ NOT STARTED` |
| **02** | **3D Knowledge Base Onboarding (RAG)** | `MVP Core - P0` | AI / Vector / RAG | **14** | `⚪ NOT STARTED` |
| **03** | **Global Discovery Radar & Stale Decay** | `MVP Core - P0` | APIs / Seed Registry | **12** | `⚪ NOT STARTED` |
| **04** | **Validation & ATS Tailoring Studio (Typst)**| `MVP Core - P0` | Typst / LLM / PDFs | **14** | `⚪ NOT STARTED` |
| **05** | **1-Click & Assisted Auto-Apply Engine** | `Phase 2 - P1` | Playwright / Copilot | **14** | `⚪ NOT STARTED` |
| **06** | **LinkedIn Referral Generator & Drafter** | `Phase 2 - P1` | Contact Search & AI | **10** | `⚪ NOT STARTED` |
| **07** | **Real-Time Tracker & Analytics Hub** | `MVP Core - P0` | Database / Analytics | **12** | `⚪ NOT STARTED` |
| **ALL** | **Full System Lifecycle** | — | **End-to-End** | **88** | `⚪ NOT STARTED` |

---

# Phase 1: Scaffolding, Migrations, Background Queue & Auth

```
Phase Status: ⚪ NOT STARTED   |   Priority: P0 - Critical (MVP Core)   |   Tasks: 0/12 Completed
```

### 🎯 Objective
Set up Dockerized PostgreSQL with `pgvector`, Redis background worker (`arq`), Alembic database migrations, `httpOnly` cookie JWT authentication (`SameSite=Lax`), structured logging (`structlog`), quota metering (`CreditMeterService`), file upload validation (5MB max, MIME check), and **Screen 1: 3D Spline Hero Portal (`fcb3d45a-1ffd-474a-afbf-9643cc3a0d40`)** with "Get a Job" and Login/Signup modals.

### 📐 Phase 1 Technical Implementation Flowchart
```mermaid
flowchart TD
    subgraph Infra ["Local Infrastructure (Docker Compose)"]
        Docker_PG[("PostgreSQL 16 + pgvector (Port: 5432)")]
        Docker_Redis[("Redis 7 (Port: 6379)")]
    end

    subgraph S1_UI ["Screen 1: 3D Hero Portal (/)"]
        SplineHero["Spline 3D Scene (fcb3d45a-1ffd-474a-afbf-9643cc3a0d40)"]
        Title["Centered Title: 'Get a Job'"]
        AuthModals["Glassmorphic Login & Sign-Up Modals"]
    end

    subgraph BackendGateway ["FastAPI Core (backend/app/)"]
        EP_Auth["Auth Endpoints (/api/auth/register, /login, /me)"]
        Alembic["Alembic Schema Migrations (alembic upgrade head)"]
        Security["Bcrypt Hashing + httpOnly SameSite=Lax Cookie JWT"]
        CreditMeter["CreditMeterService (Tracks credits_remaining)"]
        ArqWorker["Arq Background Worker (app/worker.py)"]
        UploadGuard["UploadValidator (5MB limit & PDF/DOCX MIME check)"]
    end

    AuthModals --> EP_Auth --> Security --> Docker_PG
    Security -->|Set-Cookie: httpOnly JWT| AuthModals
    Alembic --> Docker_PG
    EP_Auth --> CreditMeter
    EP_Auth --> ArqWorker --> Docker_Redis
```

### 📋 Phase 1 Granular Task Matrix

| Task ID | Status | Priority | Target File Path | Class / Function / Component | Dependencies / Notes |
| :---: | :---: | :---: | :--- | :--- | :--- |
| `TASK-1.01` | [ ] | `P0` | `docker-compose.yml` | PostgreSQL + `pgvector` and Redis 7 services | Local infrastructure setup with 100% dev/prod parity |
| `TASK-1.02` | [ ] | `P0` | `backend/requirements.txt` | Python dependencies | `fastapi`, `sqlalchemy`, `asyncpg`, `alembic`, `arq`, `redis`, `structlog` |
| `TASK-1.03` | [ ] | `P0` | `backend/alembic/` | Alembic initialization & `env.py` | `alembic init alembic` with asyncpg connection |
| `TASK-1.04` | [ ] | `P0` | `backend/app/core/config.py` | `class Settings(BaseSettings)` | Loads DB URL, Redis URL, JWT Secret, Gemini API Key |
| `TASK-1.05` | [ ] | `P0` | `backend/app/core/database.py` | Async sessionmaker with `asyncpg` | Connection pool manager for PostgreSQL |
| `TASK-1.06` | [ ] | `P0` | `backend/app/models/user.py` | `class User(Base)` | Multi-tenant model with `subscription_tier` & `credits_remaining` |
| `TASK-1.07` | [ ] | `P0` | `backend/app/core/security.py` | Password hashing & `httpOnly` cookie JWT helpers | Secure SameSite=Lax cookie auth with Next.js path proxy |
| `TASK-1.08` | [ ] | `P0` | `backend/app/services/credit_meter.py` | `class CreditMeterService` | Enforces monthly credits and rate limits per user |
| `TASK-1.09` | [ ] | `P0` | `backend/app/core/upload_guard.py` | `validate_upload_file(file, max_mb=5)` | File size and MIME type security validator |
| `TASK-1.10` | [ ] | `P0` | `backend/app/worker.py` | `class WorkerSettings` (Arq) | Async task worker handling offloaded background jobs |
| `TASK-1.11` | [ ] | `P0` | `frontend/src/app/page.tsx` | `HeroPortalView()` | **Screen 1:** Spline 3D Scene (`fcb3d45a-...`) + "Get a Job" |
| `TASK-1.12` | [ ] | `P0` | `frontend/src/components/auth-modals.tsx` | `LoginModal()`, `SignUpModal()` | Glassmorphic floating modals over 3D canvas |

---

# Phase 2: 3D Knowledge Base Onboarding (RAG Engine)

```
Phase Status: ⚪ NOT STARTED   |   Priority: P0 - Critical (MVP Core)   |   Tasks: 0/14 Completed
```

### 🎯 Objective
Build **Screen 2: Knowledge Base Onboarding (`/onboarding`)** with the clean 3D Spline Scene (`67f56854-67d0-4779-b5ed-371c2f5d169e`). Ingest Master Resumes (validated $\le 5\text{MB}$), GitHub repositories, academic papers, and screening QA vault. Process with Gemini 2.0 Flash into strict JSON and generate 384-dimensional dense embeddings in `pgvector`.

### 📐 Phase 2 Technical Implementation Flowchart
```mermaid
flowchart TD
    subgraph S2_UI ["Screen 2: 3D Knowledge Base (/onboarding)"]
        SplineScene2["Spline 3D Clean Backdrop (67f56854-67d0-4779-b5ed-371c2f5d169e)"]
        Dropzone["Master Resume Dropzone (PDF/DOCX max 5MB) + GitHub Sync + QA Vault"]
        BtnSave["[ Save & Discover Opportunities ➔ ] Button"]
    end

    subgraph BackgroundWorker ["Arq Task Queue (backend/app/worker.py)"]
        JobIngest["job_ingest_knowledge_base(user_id, file_path, github_url)"]
    end

    subgraph ParsersAndAI ["Parsers & AI Services"]
        P_PDF["ResumeParser (pdfplumber)"]
        P_GH["GitHubCrawler (httpx public API)"]
        LLM["ProfileExtractionService (Gemini 2.0 Flash JSON Schema)"]
        FastEmbed["FastEmbed (bge-small-en-v1.5)"]
    end

    subgraph Storage ["PostgreSQL + pgvector"]
        T_Profile[("user_profiles, work_experiences, projects")]
        T_Embeddings[("experience_embeddings (vector column)")]
    end

    Dropzone -->|POST /api/profile/upload| JobIngest
    JobIngest --> P_PDF --> LLM
    JobIngest --> P_GH --> LLM
    LLM --> T_Profile
    LLM --> FastEmbed --> T_Embeddings
    BtnSave -->|Profile Saved| Redirect["Auto-Redirect ➔ Screen 3: Discovery Radar (/dashboard)"]
```

### 📋 Phase 2 Granular Task Matrix

| Task ID | Status | Priority | Target File Path | Class / Function / Component | Dependencies / Notes |
| :---: | :---: | :---: | :--- | :--- | :--- |
| `TASK-2.01` | [ ] | `P0` | `backend/app/services/parsers/resume_parser.py` | `class PDFResumeParser` | Extracts text and layout sections via `pdfplumber` |
| `TASK-2.02` | [ ] | `P1` | `backend/app/services/parsers/docx_parser.py` | `class DocxResumeParser` | Word document text extractor |
| `TASK-2.03` | [ ] | `P0` | `backend/app/services/parsers/github_crawler.py` | `class GitHubRepoCrawler` | Fetches repositories, languages, stars, and READMEs |
| `TASK-2.04` | [ ] | `P1` | `backend/app/services/parsers/doc_parser.py` | `class AcademicDocParser` | Ingests research papers, abstracts, and course transcripts |
| `TASK-2.05` | [ ] | `P0` | `backend/app/services/ai_client.py` | `class AIClientService` | Google Gemini 2.0 Flash SDK client with strict JSON schema |
| `TASK-2.06` | [ ] | `P0` | `backend/app/services/profile_structurer.py` | `class ProfileExtractionService` | Transforms parsed text into typed `StructuredProfileSchema` |
| `TASK-2.07` | [ ] | `P0` | `backend/app/services/rag_engine.py` | `class LocalRAGEngine` | Generates 384-dim embeddings via `FastEmbed` |
| `TASK-2.08` | [ ] | `P0` | `backend/app/models/profile.py` | `UserProfile`, `WorkExperience`, `Project`, `ScreeningQA` | SQLAlchemy models storing structured profile & vector embeddings |
| `TASK-2.09` | [ ] | `P0` | `backend/app/schemas/profile.py` | `StructuredProfileSchema`, `QACreateSchema` | Pydantic validation schemas |
| `TASK-2.10` | [ ] | `P0` | `backend/app/api/profile.py` | `upload_resume()`, `sync_github()`, `get_profile_kb()` | REST endpoints enqueuing ingestion tasks |
| `TASK-2.11` | [ ] | `P1` | `backend/app/api/qa_vault.py` | `create_qa_item()`, `list_qa_items()` | Manages pre-filled screening QA answers |
| `TASK-2.12` | [ ] | `P0` | `frontend/src/app/onboarding/page.tsx` | `KnowledgeBaseOnboardingView()` | **Screen 2:** Spline 3D Scene (`67f56854-...`) + Upload Dropzone |
| `TASK-2.13` | [ ] | `P1` | `frontend/src/components/github-sync-modal.tsx` | `GitHubSyncModal()` | GitHub repo selector and sync dialog |
| `TASK-2.14` | [ ] | `P1` | `frontend/src/components/qa-vault-drawer.tsx` | `QAVaultDrawer()` | Sliding drawer for screening questions & salary/visa info |

---

# Phase 3: Global Discovery Radar & Stale Decay Engine

```
Phase Status: ⚪ NOT STARTED   |   Priority: P0 - Critical (MVP Core)   |   Tasks: 0/12 Completed
```

### 🎯 Objective
Build **Screen 3: Opportunity Discovery Radar (`/dashboard`)**. Ingests verified opportunities via **Adzuna Global Search API + Company Board Registry (500+ curated Greenhouse & Lever boards) + Hacker News API + Unstop/Devpost RSS**. Apply the **Posting Age Decay Stale Filter (`posted_days > 21`)** and compute 0–100% match confidence scores via `pgvector` Cosine Similarity + BM25 keyword matching.

### 📐 Phase 3 Technical Implementation Flowchart
```mermaid
flowchart TD
    subgraph DiscoverySources ["Global & ATS Ingestion (backend/app/services/scrapers/)"]
        SeedReg["Company Seed Registry (500+ curated company board slugs)"]
        API_Adzuna["Adzuna Global Search API (api.adzuna.com)"]
        API_GH["Greenhouse Public Boards JSON (boards-api.greenhouse.io)"]
        API_Lever["Lever Public Postings API (api.lever.co)"]
        API_HN["Hacker News 'Who is Hiring?' Firebase API"]
        RSS_Hack["Unstop & Devpost Official RSS Feeds"]
    end

    subgraph Pipeline ["Normalization & Age Decay Filtering"]
        Normalizer["OpportunityNormalizerService (SHA-256 Dedupe Hash)"]
        AgeDecayCheck{"STALE AGE DECAY RULE:\nposted_days > 21 (50% penalty)\nposted_days > 45 (80% penalty)"}
        ApplyDecay["Apply Stale Decay Multiplier (score * decay)"]
        FreshScore["Normal Fresh Score (Multiplier = 1.0)"]
    end

    subgraph ScoringEngine ["Hybrid Scoring Engine (pgvector Cosine + BM25)"]
        Formula["Final Confidence: (0.6 * VecSim + 0.4 * BM25) * DecayMultiplier\nScale: 0.0% – 100.0%"]
    end

    subgraph S3_UI ["Screen 3: Opportunity Radar (/dashboard)"]
        UI_Feed["Clean 3D Themed Opportunity Feed\n• Category Tabs (All, Jobs, Unstop Hackathons, Research, HN)\n• Match Confidence Badges (0–100%)\n• Stale Filter Toggle\n• Action: [ Review & Tailor Resume ➔ ]"]
    end

    SeedReg --> API_GH --> Normalizer
    SeedReg --> API_Lever --> Normalizer
    API_Adzuna --> Normalizer
    API_HN --> Normalizer
    RSS_Hack --> Normalizer

    Normalizer --> AgeDecayCheck
    AgeDecayCheck -->|Stale| ApplyDecay --> Formula
    AgeDecayCheck -->|Fresh| FreshScore --> Formula
    Formula --> UI_Feed
```

### 📋 Phase 3 Granular Task Matrix

| Task ID | Status | Priority | Target File Path | Class / Function / Component | Dependencies / Notes |
| :---: | :---: | :---: | :--- | :--- | :--- |
| `TASK-3.00` | [ ] | `P0` | `backend/app/services/scrapers/company_registry.py` | `class CompanyBoardRegistry` | Curated seed dataset of 500+ Greenhouse/Lever board slugs |
| `TASK-3.01` | [ ] | `P0` | `backend/app/services/scrapers/adzuna_api.py` | `class AdzunaAPIClient` | Queries global search index across thousands of active jobs |
| `TASK-3.02` | [ ] | `P0` | `backend/app/services/scrapers/greenhouse_api.py` | `class GreenhouseAPIClient` | Ingests public job listings from Greenhouse JSON boards |
| `TASK-3.03` | [ ] | `P0` | `backend/app/services/scrapers/lever_api.py` | `class LeverAPIClient` | Ingests public job postings from Lever API |
| `TASK-3.04` | [ ] | `P1` | `backend/app/services/scrapers/hn_api.py` | `class HackerNewsHiringClient` | Fetches monthly "Who is hiring?" jobs via Firebase API |
| `TASK-3.05` | [ ] | `P0` | `backend/app/services/scrapers/hackathon_rss.py` | `class HackathonFeedParser` | Ingests Unstop, Devpost, and Kaggle competition feeds |
| `TASK-3.06` | [ ] | `P0` | `backend/app/services/opportunity_normalizer.py` | `class OpportunityNormalizerService` | Generates SHA-256 dedupe hash & cleans descriptions |
| `TASK-3.07` | [ ] | `P0` | `backend/app/services/stale_filter.py` | `class StaleAgeDecayService` | Posting age decay rule (`>21 days` 50% decay, `>45 days` 80% decay) |
| `TASK-3.08` | [ ] | `P0` | `backend/app/services/scoring_engine.py` | `class OpportunityScoringService` | Computes hybrid `pgvector` Cosine + BM25 match score |
| `TASK-3.09` | [ ] | `P0` | `backend/app/models/opportunity.py` | `class Opportunity(Base)` | Database model storing opportunities and confidence scores |
| `TASK-3.10` | [ ] | `P0` | `backend/app/api/discovery.py` | `get_feed()`, `trigger_sync()` | Endpoints for fetching scored listings feed |
| `TASK-3.11` | [ ] | `P0` | `frontend/src/app/dashboard/page.tsx` | `OpportunityRadarView()` | **Screen 3:** Clean 3D themed feed with category tabs & search |

---

# Phase 4: Opportunity Validation & ATS Resume/CV Tailoring Studio

```
Phase Status: ⚪ NOT STARTED   |   Priority: P0 - Critical (MVP Core)   |   Tasks: 0/14 Completed
```

### 🎯 Objective
Build **Screen 4: Tailoring Studio (`/tailor/[id]`)**. Evaluates ATS score using deterministic code-based parseability + LLM keyword/impact analysis. If score is $< 8$, automatically retrieves relevant projects from RAG, injects required keywords, rewrites bullets using the Google XYZ formula, generates a custom Cover Letter, compiles ATS-perfect Typst PDFs in $<50\text{ms}$, and provides direct download buttons.

### 📐 Phase 4 Technical Implementation Flowchart
```mermaid
flowchart TD
    subgraph TriggerAction ["Screen 4: Validation Studio Trigger"]
        UserClick["User clicks 'Review & Tailor Resume' on an Opportunity"]
    end

    subgraph ATSEvaluator ["ATS Evaluator (backend/app/services/ats_evaluator.py)"]
        CodeAudit["Deterministic Structural Audit in Python:\n• Selectable text extractability\n• Standard section headers\n• Parseable date formats (MMM YYYY)"]
        LLMAudit["LLM Semantic Critique (Gemini 2.0 Flash):\n• Keyword Match Density (40%)\n• Action Metrics & Google XYZ (30%)\n• Section Relevancy (20%)"]
        CheckScore{"Is ATS Score < 8.0?"}
    end

    subgraph OptimizerEngine ["Dynamic Tailoring (backend/app/services/)"]
        RAG_Query["Query pgvector for Most Relevant Achievements"]
        Optimizer["ResumeOptimizerService (Gemini 2.0 Flash / Groq):\n• Highlight real matching achievements\n• Inject missing JD keywords without hallucination\n• Elevate ATS Score to > 8.5/10"]
        CoverGen["CoverLetterGeneratorService:\n• Company-aligned formal cover letter"]
    end

    subgraph TypstCompiler ["Document Compilation (backend/app/services/pdf_compiler.py)"]
        CompileResume["Typst Resume Compiler (resume_ats.typ) -> Single-Column PDF"]
        CompileCV["Typst Cover Letter Compiler (cover_letter.typ) -> PDF"]
    end

    subgraph S4_UI ["Screen 4: Tailoring Studio UI (/tailor/[id])"]
        ScoreGauge["ATS Score Meter (e.g. 6.1 ➔ 9.3 / 10)"]
        DL_Resume["📥 Download Tailored Resume (PDF) Button"]
        DL_CV["📥 Download Tailored Cover Letter (PDF) Button"]
        BtnApply["[ 1-Click Auto-Apply 🚀 ] Button"]
    end

    UserClick --> CodeAudit --> LLMAudit --> CheckScore
    CheckScore -->|Yes: Score < 8| RAG_Query --> Optimizer --> CompileResume
    CheckScore -->|No: Score >= 8| CompileResume
    Optimizer --> CoverGen --> CompileCV

    CompileResume --> DL_Resume
    CompileCV --> DL_CV
    LLMAudit --> ScoreGauge
    CompileResume --> BtnApply
```

### 📋 Phase 4 Granular Task Matrix

| Task ID | Status | Priority | Target File Path | Class / Function / Component | Dependencies / Notes |
| :---: | :---: | :---: | :--- | :--- | :--- |
| `TASK-4.01` | [ ] | `P0` | `backend/app/services/ats_evaluator.py` | `class ATSEvaluatorService` | Deterministic structural audit + LLM keyword critique |
| `TASK-4.02` | [ ] | `P0` | `backend/app/services/rag_engine.py` | `get_top_projects_for_jd()` | Retrieves candidate's best matching projects via `pgvector` |
| `TASK-4.03` | [ ] | `P0` | `backend/app/services/tailoring_engine.py` | `class ResumeOptimizerService` | Rewrites bullets into Google XYZ impact format & injects keywords |
| `TASK-4.04` | [ ] | `P1` | `backend/app/services/cover_letter_gen.py` | `class CoverLetterGeneratorService` | Generates role-targeted, company-aligned cover letter |
| `TASK-4.05` | [ ] | `P0` | `backend/app/templates/resume_ats.typ` | ATS Typst Template | Single-column, clean typography, 100% ATS parseable |
| `TASK-4.06` | [ ] | `P1` | `backend/app/templates/cover_letter.typ` | Cover Letter Typst Template | Clean executive letterhead layout |
| `TASK-4.07` | [ ] | `P0` | `backend/app/services/pdf_compiler.py` | `class PDFCompilerService` | Compiles Typst templates to binary PDF via `typst-py` (<50ms) |
| `TASK-4.08` | [ ] | `P1` | `backend/app/models/tailored_doc.py` | `TailoredDocument`, `ATSReport` | Models storing optimized resumes, cover letters, and scores |
| `TASK-4.09` | [ ] | `P1` | `backend/app/schemas/tailor.py` | `ATSReportSchema`, `TailorResponseSchema` | Pydantic schemas for scoring & tailored outputs |
| `TASK-4.10` | [ ] | `P0` | `backend/app/api/tailor.py` | `evaluate_ats()`, `optimize_resume()`, `generate_cv()` | Endpoints powering the Tailoring Studio |
| `TASK-4.11` | [ ] | `P0` | `backend/app/api/downloads.py` | `download_resume()`, `download_cv()` | Direct 1-click download endpoints returning PDF files |
| `TASK-4.12` | [ ] | `P0` | `frontend/src/app/tailor/[id]/page.tsx` | `TailoringStudioView()` | **Screen 4:** ATS score meter, keyword diff preview & download buttons |
| `TASK-4.13` | [ ] | `P1` | `frontend/src/components/resume-diff-editor.tsx` | `ResumeDiffEditor()` | Visual side-by-side diff highlighting injected keywords |
| `TASK-4.14` | [ ] | `P0` | `frontend/src/components/download-bar.tsx` | `DownloadActionBar()` | Direct download buttons for Tailored Resume & CV |

---

# Phase 5: 1-Click & Assisted Auto-Apply Engine (Browser Agent)

```
Phase Status: ⚪ NOT STARTED   |   Priority: P1 - High (Post-MVP)   |   Tasks: 0/14 Completed
```

### 🎯 Objective
Automate application submissions via Playwright. Support **1-Click Direct Auto-Apply for Greenhouse & Lever** (clean single-page DOMs) and **Assisted Apply / Copilot Answer Pack for complex multi-step portals (Workday, Taleo)** to guarantee zero bot-bans.

### 📐 Phase 5 Technical Implementation Flowchart
```mermaid
flowchart TD
    subgraph Trigger ["Screen 4: Apply Trigger"]
        Start["User clicks '1-Click Auto-Apply 🚀'"]
    end

    subgraph PortalDetection ["Portal Type Detection"]
        Detect{"Is Portal Greenhouse or Lever?"}
    end

    subgraph DirectAutoApply ["Direct 1-Click Auto-Apply Mode (Playwright)"]
        Launch["Launch Async Playwright Browser (Stealth Mode)"]
        AutoFill["Auto-Fill Fields & Attach Tailored Typst PDF"]
        SubmitDirect["Submit Application & Capture Confirmation"]
    end

    subgraph AssistedCopilot ["Assisted Apply / Copilot Mode (Workday / Complex)"]
        GenPack["Generate Answer Pack (Tailored text + QA answers)"]
        DrawerUI["Open Floating Copilot Drawer with 1-Click Clipboard Triggers"]
        ManualConfirm["User reviews pre-filled answers & clicks final Submit"]
    end

    subgraph SequentialTriggers ["Post-Apply Triggers"]
        TriggerStep6["Auto-Trigger Screen 5: LinkedIn Referral Generator"]
        TriggerStep7["Auto-Trigger Screen 6: Real-Time Application Tracker"]
    end

    Start --> Detect
    Detect -->|Yes: Greenhouse/Lever| Launch --> AutoFill --> SubmitDirect --> TriggerStep6
    Detect -->|No: Workday/Custom| GenPack --> DrawerUI --> ManualConfirm --> TriggerStep6
    SubmitDirect --> TriggerStep7
    ManualConfirm --> TriggerStep7
```

### 📋 Phase 5 Granular Task Matrix

| Task ID | Status | Priority | Target File Path | Class / Function / Component | Dependencies / Notes |
| :---: | :---: | :---: | :--- | :--- | :--- |
| `TASK-5.01` | [ ] | `P0` | `backend/app/services/browser_agent/agent.py` | `class PlaywrightBrowserAgent` | Manages async browser session with stealth plugin |
| `TASK-5.02` | [ ] | `P0` | `backend/app/services/browser_agent/form_inspector.py` | `class FormInspectorService` | Detects input fields and file upload elements in DOM |
| `TASK-5.03` | [ ] | `P0` | `backend/app/services/browser_agent/form_mapper.py` | `class FieldMapperService` | Maps DOM labels to User KB fields |
| `TASK-5.04` | [ ] | `P0` | `backend/app/services/browser_agent/adapters/greenhouse.py` | `class GreenhouseAdapter` | Handles Greenhouse single & multi-step forms |
| `TASK-5.05` | [ ] | `P0` | `backend/app/services/browser_agent/adapters/lever.py` | `class LeverAdapter` | Handles Lever application pages |
| `TASK-5.06` | [ ] | `P1` | `backend/app/services/browser_agent/copilot_pack.py` | `class AnswerPackGenerator` | Compiles pre-filled Answer Pack for Workday/custom portals |
| `TASK-5.07` | [ ] | `P1` | `backend/app/services/browser_agent/adapters/generic.py` | `class GenericFormAdapter` | Heuristic fallback DOM walker for custom portals |
| `TASK-5.08` | [ ] | `P0` | `backend/app/services/browser_agent/question_fallback.py` | `class QuestionFallbackHandler` | Pauses session on low confidence & notifies user |
| `TASK-5.09` | [ ] | `P1` | `backend/app/api/apply_stream.py` | `stream_application_progress()` | SSE stream sending live step updates to frontend |
| `TASK-5.10` | [ ] | `P0` | `backend/app/api/apply.py` | `start_apply()`, `submit_question_answer()` | Endpoints controlling auto-apply lifecycle |
| `TASK-5.11` | [ ] | `P1` | `frontend/src/components/auto-apply-modal.tsx` | `AutoApplyModal()` | Modal showing live progress step indicators |
| `TASK-5.12` | [ ] | `P0` | `frontend/src/components/custom-q-prompt.tsx` | `QuestionPromptDialog()` | Interactive prompt modal for answering unknown questions |
| `TASK-5.13` | [ ] | `P1` | `frontend/src/components/copilot-drawer.tsx` | `AssistedApplyDrawer()` | Floating sidebar with 1-click clipboard answers for Workday |
| `TASK-5.14` | [ ] | `P0` | `backend/app/services/event_bus.py` | `class ApplicationEventBus` | Automatically triggers Phase 6 & Phase 7 on submit |

---

# Phase 6: LinkedIn Referral Search Generator & Outreach Drafter

```
Phase Status: ⚪ NOT STARTED   |   Priority: P1 - High (Post-MVP)   |   Tasks: 0/10 Completed
```

### 🎯 Objective
Build **Screen 5: Referral Search Hub (`/referrals/[id]`)**. Accessible on-demand or immediately post-application. Identifies 5 key employee personas (Recruiter, Engineering Lead, Team Peer, University Recruiter, Alumni) at the company, generates encoded direct LinkedIn Search URLs (opening in the user's logged-in session), and crafts tailored 50-word cold outreach messages with a 1-click copy action.

### 📐 Phase 6 Technical Implementation Flowchart
```mermaid
flowchart TD
    subgraph Trigger ["Trigger (On-Demand or Post-Apply)"]
        PostApply["User selects Referral Hub on an Opportunity"]
    end

    subgraph PersonaEngine ["Persona & Search URL Generator (backend/app/services/referral_finder.py)"]
        ExtractTarget["Extract Company Name & Applied Role Department"]
        BuildSearches["Generate 5 Targeted LinkedIn Search URLs:\n1. Technical Recruiter Search URL\n2. Engineering Manager in Dept URL\n3. Senior Engineer / Team Peer URL\n4. University Recruiter URL\n5. College Alumni at Company URL"]
    end

    subgraph MessageGenerator ["Outreach Message Synthesizer (backend/app/services/outreach_generator.py)"]
        Synthesize["Gemini 2.0 / Groq Drafter:\n• Personalize with persona role & company tech stack\n• Short, simple, high-converting (<75 words)\n• Mentions candidate's strongest matching project"]
    end

    subgraph S5_UI ["Screen 5: Referral Hub UI (/referrals/[id])"]
        RenderCards["Display 5 Interactive Persona Cards:\n• Persona Badge (Recruiter, Lead, Peer, Alumni)\n• Direct LinkedIn Search Link (Opens user's LinkedIn ↗)\n• Tailored Message Textarea\n• 📋 1-Click 'Copy Message' Button"]
    end

    PostApply --> ExtractTarget --> BuildSearches --> Synthesize --> RenderCards
```

### 📋 Phase 6 Granular Task Matrix

| Task ID | Status | Priority | Target File Path | Class / Function / Component | Dependencies / Notes |
| :---: | :---: | :---: | :--- | :--- | :--- |
| `TASK-6.01` | [ ] | `P1` | `backend/app/services/referral_finder.py` | `extract_company_metadata()` | Resolves target company name and department |
| `TASK-6.02` | [ ] | `P1` | `backend/app/services/referral_finder.py` | `generate_linkedin_search_urls()` | Generates 5 encoded LinkedIn search deeplinks |
| `TASK-6.03` | [ ] | `P1` | `backend/app/services/referral_finder.py` | `build_alumni_search_query()` | Custom search URL incorporating user's university |
| `TASK-6.04` | [ ] | `P0` | `backend/app/services/outreach_generator.py` | `class OutreachGeneratorService` | Crafts short, personalized cold messages (<75 words) |
| `TASK-6.05` | [ ] | `P1` | `backend/app/models/referral.py` | `class ReferralTarget(Base)` | Database model storing generated search queries and messages |
| `TASK-6.06` | [ ] | `P1` | `backend/app/schemas/referral.py` | `ReferralSearchResponse`, `ReferralListSchema` | Pydantic schemas for referral API |
| `TASK-6.07` | [ ] | `P1` | `backend/app/api/referrals.py` | `get_referral_searches()`, `regenerate_message()` | API endpoints for fetching and regenerating messages |
| `TASK-6.08` | [ ] | `P1` | `frontend/src/app/referrals/[id]/page.tsx` | `ReferralHubView()` | **Screen 5:** Displays 5 referral cards with direct search links |
| `TASK-6.09` | [ ] | `P0` | `frontend/src/components/referral-card.tsx` | `CopyMessageButton()` | 1-Click clipboard copy button with visual feedback |
| `TASK-6.10` | [ ] | `P2` | `frontend/src/components/referral-card.tsx` | `ReferralCard()` | Card with direct LinkedIn search link & message box |

---

# Phase 7: Real-Time Application Tracker & Analytics Hub

```
Phase Status: ⚪ NOT STARTED   |   Priority: P0 - Critical (MVP Core)   |   Tasks: 0/12 Completed
```

### 🎯 Objective
Build **Screen 6: Application Tracker & Analytics (`/tracker`)**. Real-time logging of all applications (auto-logged on submit or manually added). Compute KPI metrics (Today / Week / Month / Year), provide a sortable data table with outcome management (Rejection, Selection, No Response), and provide 1-click Excel (.xlsx) and JSON export.

### 📐 Phase 7 Technical Implementation Flowchart
```mermaid
flowchart TD
    subgraph DataInputs ["Application Inputs"]
        AutoLog["Auto-Logged from Phase 5 Application Completion"]
        ManualLog["User Manual Entry via Add Application Modal"]
    end

    subgraph TrackerServiceLayer ["Tracker Service (backend/app/services/tracker_service.py)"]
        Table_App[("Table: applications\n• id (UUID PK)\n• user_id (UUID FK)\n• company (VARCHAR)\n• role (VARCHAR)\n• platform (ENUM)\n• status (ENUM: Applied, No Response, OA, Interview, Rejection, Offer)\n• applied_at (TIMESTAMP)\n• notes (TEXT)")]
        
        KPICalc["Compute Real-Time KPI Aggregations:\n• Applied Today (created_at >= CURRENT_DATE)\n• Applied This Week (created_at >= 7_DAYS_AGO)\n• Applied This Month (created_at >= 30_DAYS_AGO)\n• Applied This Year (created_at >= 365_DAYS_AGO)"]
        
        StatusAgg["Compute Status Distribution:\n• Applied • No Response\n• Online Assessment • Interview\n• Rejection • Offer"]
    end

    subgraph ExportEngine ["Export Engine (backend/app/services/export_service.py)"]
        XLSX["openpyxl Spreadsheet Generator (.xlsx)"]
        JSON_Exp["JSON Serializer (.json)"]
    end

    subgraph S6_UI ["Screen 6: Tracker Hub (/tracker)"]
        KPICards["Real-Time Metric Cards (Today / Week / Month / Year)"]
        DataTable["TanStack Sortable & Filterable Table\n(Sort: Today, 1 Week Ago, 1 Month Ago, 1 Year Ago)"]
        InlineEdit["Inline Status Editor (Select Dropdown & Notes)"]
        ExportDropdown["📥 1-Click Export (Excel .xlsx / JSON)"]
    end

    AutoLog --> Table_App
    ManualLog --> Table_App

    Table_App --> KPICalc --> KPICards
    Table_App --> StatusAgg --> DataTable
    Table_App --> InlineEdit --> Table_App

    Table_App --> XLSX --> ExportDropdown
    Table_App --> JSON_Exp --> ExportDropdown
```

### 📋 Phase 7 Granular Task Matrix

| Task ID | Status | Priority | Target File Path | Class / Function / Component | Dependencies / Notes |
| :---: | :---: | :---: | :--- | :--- | :--- |
| `TASK-7.01` | [ ] | `P0` | `backend/app/models/application.py` | `class Application(Base)` | Database model with status enums and timestamps |
| `TASK-7.02` | [ ] | `P0` | `backend/app/services/tracker_service.py` | `log_application_event()` | Real-time auto-logger called upon form submission |
| `TASK-7.03` | [ ] | `P0` | `backend/app/services/tracker_service.py` | `get_kpi_metrics()` | SQL aggregations for Today, Week, Month, Year |
| `TASK-7.04` | [ ] | `P0` | `backend/app/services/tracker_service.py` | `list_user_applications()` | Sortable query with date range & status filters |
| `TASK-7.05` | [ ] | `P1` | `backend/app/services/export_service.py` | `export_to_excel()` | Generates formatted Excel (`.xlsx`) via `openpyxl` |
| `TASK-7.06` | [ ] | `P2` | `backend/app/services/export_service.py` | `export_to_json()` | Generates structured JSON data file |
| `TASK-7.07` | [ ] | `P0` | `backend/app/schemas/tracker.py` | `ApplicationCreate`, `ApplicationUpdate`, `KPISchema` | Pydantic validation schemas |
| `TASK-7.08` | [ ] | `P0` | `backend/app/api/tracker.py` | `get_metrics()`, `list_apps()`, `create_manual()`, `patch_app()`, `export()` | REST endpoints for tracker lifecycle |
| `TASK-7.09` | [ ] | `P0` | `frontend/src/app/tracker/kpi-cards.tsx` | `KPIMetricsGrid()` | Metric cards for Today, Week, Month, Year counts |
| `TASK-7.10` | [ ] | `P0` | `frontend/src/app/tracker/data-table.tsx` | `ApplicationDataTable()` | **Screen 6:** TanStack Table with inline status dropdown & filters |
| `TASK-7.11` | [ ] | `P1` | `frontend/src/components/add-application-modal.tsx` | `AddApplicationModal()` | Dialog for adding manual application entries |
| `TASK-7.12` | [ ] | `P0` | `frontend/src/components/export-dropdown.tsx` | `ExportDropdown()` | 1-Click download triggers for Excel & JSON |

---

# Comprehensive Verification & Testing Suite

```bash
# ============================================================
# PHASE-BY-PHASE AUTOMATED TEST EXECUTION COMMANDS
# ============================================================

# 1. Verify Phase 1 (Auth, Migrations, Queue & 3D Hero Portal)
pytest tests/test_auth.py -v

# 2. Verify Phase 2 (Knowledge Base & RAG Vector Engine)
pytest tests/test_rag.py -v

# 3. Verify Phase 3 (Discovery APIs, Company Registry & Stale Decay)
pytest tests/test_discovery.py -v

# 4. Verify Phase 4 (ATS Evaluator & Typst PDF Compiler)
pytest tests/test_tailoring.py -v

# 5. Verify Phase 5 (Playwright Browser Auto-Apply)
pytest tests/test_apply.py -v

# 6. Verify Phase 6 (LinkedIn Referral Search Generator)
pytest tests/test_referrals.py -v

# 7. Verify Phase 7 (Real-Time Tracker & Excel Export)
pytest tests/test_tracker.py -v

# Run Full System Test Suite with Coverage
pytest tests/ -v --cov=app --cov-report=term-missing
```
