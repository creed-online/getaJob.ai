# 🗂️ getJob.ai — Complete Project Structure & Codebase Architecture (V2 Production)

> **Document Purpose:** This document serves as the definitive reference guide for the directory organization, module responsibilities, and data flow across the **getJob.ai (getaJob.ai)** repository.

---

## 🌳 1. Master Repository Directory Tree

```
getaJob.ai/
│
├── docker-compose.yml                  # Local PostgreSQL 16 (with pgvector) & Redis 7
├── .gitignore                          # Root ignore rules for secrets and build artifacts
├── readme.md                           # Master GitHub hero README with Wiki links
│
├── Docs/                               # Architecture, planning & engineering blueprints
│   ├── workflow.md                     # Original product requirements & feature specification
│   ├── workflowArchitecture.pdf        # Visual high-level architecture diagram PDF
│   ├── sitemap.md                      # Simplified 3D user journey & screen specifications
│   ├── techStack.md                    # Finalized technology stack & library manifest
│   ├── IMPLEMENTATION_PLAN.md          # Master 88-task execution plan & phase tracker
│   └── projectStructure.md             # This document (Codebase layout reference)
│
├── backend/                            # FastAPI Python 3.11+ Async Backend
│   ├── alembic/                        # Alembic Database Migration Environment
│   │   ├── versions/                   # Migration revision scripts
│   │   └── env.py                      # Async SQLAlchemy migration runner
│   ├── alembic.ini                     # Alembic configuration
│   │
│   ├── app/
│   │   ├── api/                        # REST & SSE API Controllers (HTTP Endpoints)
│   │   │   ├── auth.py                 # Multi-tenant Auth (Register, Login, Me, Logout)
│   │   │   ├── profile.py              # Knowledge Base (Resume upload, GitHub sync, Papers)
│   │   │   ├── qa_vault.py             # Screening QA Vault CRUD operations
│   │   │   ├── discovery.py            # Scored Opportunity feed & trigger sync
│   │   │   ├── tailor.py               # ATS Score check (1–10) & resume/CV optimizer
│   │   │   ├── downloads.py            # Direct binary PDF downloads (Resume & Cover Letter)
│   │   │   ├── apply.py                # 1-Click Auto-Apply runner & prompt submission
│   │   │   ├── apply_stream.py         # Server-Sent Events (SSE) live browser progress stream
│   │   │   ├── referrals.py            # 5-Persona Referral search deeplinks & message drafter
│   │   │   └── tracker.py              # Application KPI metrics, table CRUD & Excel export
│   │   │
│   │   ├── core/                       # Core Infrastructure, Security & Database
│   │   │   ├── config.py               # Pydantic Settings (DB URL, Redis, API keys)
│   │   │   ├── database.py             # SQLAlchemy 2.0 Async session factory
│   │   │   ├── security.py             # Bcrypt password hashing & httpOnly SameSite=Lax JWT
│   │   │   └── upload_guard.py         # File size (5MB) & MIME type validator
│   │   │
│   │   ├── models/                     # SQLAlchemy Declarative Database Tables
│   │   │   ├── user.py                 # User model (subscription_tier & credits_remaining)
│   │   │   ├── profile.py              # UserProfile, WorkExperience, Project, ScreeningQA
│   │   │   ├── opportunity.py          # Opportunity model (Platform enum, score, stale flag)
│   │   │   ├── tailored_doc.py         # TailoredDocument, ATSReport, CoverLetter
│   │   │   ├── referral.py             # ReferralTarget model (Personas + search query + draft)
│   │   │   └── application.py          # Application tracking model (Statuses, audit dates)
│   │   │
│   │   ├── schemas/                    # Pydantic v2 Request/Response Validation Schemas
│   │   │   ├── auth.py                 # Register, Login, TokenResponse, UserUpdate schemas
│   │   │   ├── profile.py              # StructuredProfile, WorkExp, Project, QASchemas
│   │   │   ├── opportunity.py          # OpportunityResponse, FilterParams schemas
│   │   │   ├── tailor.py               # ATSReportSchema, TailoredResumeResponse schemas
│   │   │   ├── referral.py             # ReferralSearchResponse, ReferralListSchema
│   │   │   └── tracker.py              # ApplicationCreate, KPIMetricsResponse schemas
│   │   │
│   │   ├── services/                   # Business Logic, AI Models & Automation Engines
│   │   │   ├── ai_client.py            # Google Gemini 2.0 Flash SDK wrapper
│   │   │   ├── credit_meter.py         # Per-user quota & rate-limiting service
│   │   │   ├── rag_engine.py           # FastEmbed local vector engine (RAM / 384-dim)
│   │   │   ├── profile_structurer.py   # LLM parser transforming raw data into typed JSON
│   │   │   ├── ats_evaluator.py        # Code structural audit + LLM keyword critique (1–10)
│   │   │   ├── tailoring_engine.py     # Google XYZ bullet optimizer & keyword injector
│   │   │   ├── cover_letter_gen.py     # Role-targeted cover letter synthesizer
│   │   │   ├── pdf_compiler.py         # Typst PDF compiler service (<50ms compilation)
│   │   │   ├── referral_finder.py      # 5-Persona company search URL generator
│   │   │   ├── outreach_generator.py   # Concise (<75 words) cold outreach message drafter
│   │   │   ├── tracker_service.py      # Real-time auto-logger & KPI metric aggregations
│   │   │   ├── export_service.py       # openpyxl (.xlsx) & JSON export generators
│   │   │   ├── event_bus.py            # Async event dispatcher triggering post-apply steps
│   │   │   │
│   │   │   ├── parsers/                # Document Ingestion Parsers
│   │   │   │   ├── resume_parser.py    # PDF text/section layout parser (pdfplumber)
│   │   │   │   ├── docx_parser.py      # Microsoft Word resume parser (python-docx)
│   │   │   │   ├── github_crawler.py   # GitHub public API repo & README crawler
│   │   │   │   └── doc_parser.py       # Research paper & transcript course parser
│   │   │   │
│   │   │   ├── scrapers/               # Direct Public API & Feed Ingestion
│   │   │   │   ├── company_registry.py # 500+ Curated Greenhouse/Lever company board slugs
│   │   │   │   ├── adzuna_api.py       # Adzuna Global Search API client
│   │   │   │   ├── greenhouse_api.py   # Greenhouse public JSON boards client
│   │   │   │   ├── lever_api.py        # Lever public postings client
│   │   │   │   ├── hn_api.py           # Hacker News Firebase API "Who is Hiring?" client
│   │   │   │   ├── hackathon_rss.py    # Unstop & Devpost hiring competitions RSS parser
│   │   │   │   ├── normalizer.py       # SHA-256 deduplication hashing & HTML cleaner
│   │   │   │   └── stale_filter.py     # Posting age decay rule (>21 days decay)
│   │   │   │
│   │   │   └── browser_agent/          # Autonomous & Assisted Auto-Apply (Playwright)
│   │   │       ├── agent.py            # Async Chromium browser session manager
│   │   │       ├── form_inspector.py   # DOM interactive input/file element detector
│   │   │       ├── form_mapper.py      # Field label to User Knowledge Base resolver
│   │   │       ├── copilot_pack.py     # Pre-filled Answer Pack generator for Workday
│   │   │       ├── question_fallback.py# Unknown question pause & SSE notification handler
│   │   │       └── adapters/           # Greenhouse, Lever & Generic portal adapters
│   │   │
│   │   ├── templates/                  # ATS-Optimized Typst Templates
│   │   │   ├── resume_ats.typ          # Single-column machine-readable ATS resume
│   │   │   └── cover_letter.typ        # Formal executive letterhead cover letter
│   │   │
│   │   ├── worker.py                   # Arq background worker tasks (RAG, apply, sync)
│   │   └── main.py                     # FastAPI app instance, CORS middleware & routers
│   │
│   ├── storage/                        # Persistent file caches & generated artifacts
│   │   └── generated/                  # Compiled ATS Resume & CV PDFs
│   ├── tests/                          # Automated Pytest Suite
│   ├── requirements.txt                # Python package dependencies
│   └── .env.example                    # Environment configuration template
│
└── frontend/                           # Next.js 14 App Router + TypeScript
    ├── public/                         # Static assets, favicon & icons
    ├── src/
    │   ├── app/                        # Next.js 14 App Router Page Views
    │   │   ├── page.tsx                # Screen 1: 3D Spline Hero Portal ("Get a Job")
    │   │   ├── layout.tsx              # Root HTML Layout, Theme Provider & Global Fonts
    │   │   ├── globals.css             # Tailwind CSS styles & glassmorphism variables
    │   │   │
    │   │   ├── (auth)/                 # Unauthenticated Route Group
    │   │   │   ├── login/page.tsx      # Standalone Login fallback view
    │   │   │   └── register/page.tsx   # Standalone Registration fallback view
    │   │   │
    │   │   ├── onboarding/             # Screen 2: 3D Knowledge Base Onboarding
    │   │   │   └── page.tsx            # Resume dropzone, GitHub sync & QA Vault
    │   │   │
    │   │   ├── dashboard/              # Screen 3: Opportunity Discovery Radar
    │   │   │   └── page.tsx            # Scored feeds (Greenhouse, Lever, Unstop, HN)
    │   │   │
    │   │   ├── tailor/                 # Screen 4: Validation & ATS Tailoring Studio
    │   │   │   └── [id]/page.tsx       # ATS score meter (1-10), bullet diff & downloads
    │   │   │
    │   │   ├── referrals/              # Screen 5: LinkedIn Referral Search Hub
    │   │   │   └── [id]/page.tsx       # 5 Persona cards + 1-Click Copy Short Messages
    │   │   │
    │   │   └── tracker/                # Screen 6: Real-Time Tracker & Analytics
    │   │       └── page.tsx            # KPI cards (Today/Week/Month), table, Excel export
    │   │
    │   ├── components/                 # Reusable React & UI Components
    │   │   ├── 3d/                     # 3D WebGL & Spline Wrappers
    │   │   │   ├── hero-spline.tsx     # Spline Scene 1 Component (fcb3d45a-...)
    │   │   │   └── onboarding-3d.tsx   # Spline Scene 2 Component (67f56854-...)
    │   │   │
    │   │   ├── ui/                     # Shadcn UI primitives (Radix UI wrappers)
    │   │   ├── auth-modals.tsx         # Floating Glassmorphic Login & Sign-Up Modals
    │   │   ├── opportunity-card.tsx    # Scored Opportunity card with confidence gauge
    │   │   ├── resume-diff-editor.tsx  # Side-by-side bullet diff & keyword highlighter
    │   │   ├── download-bar.tsx        # 1-Click Tailored Resume & CV download bar
    │   │   ├── auto-apply-modal.tsx    # Live Playwright browser progress stepper modal
    │   │   ├── copilot-drawer.tsx      # Floating Assisted Apply drawer for Workday
    │   │   ├── custom-q-prompt.tsx     # Unknown question input prompt dialog
    │   │   ├── referral-card.tsx       # Referral persona card with direct LinkedIn search ↗
    │   │   ├── kpi-metrics-grid.tsx    # Tracker summary widgets (Today, Week, Month, Year)
    │   │   └── tracker-table.tsx       # TanStack sortable & filterable applications table
    │   │
    │   ├── lib/                        # Utility Functions & Client State
    │   │   ├── api.ts                  # Axios instance with auto credentials for cookies
    │   │   ├── auth-store.ts           # Zustand store for user session state
    │   │   └── utils.ts                # Class merging (clsx + tailwind-merge) & formatters
    │   │
    │   └── types/                      # TypeScript Global Type Definitions
    │       ├── user.ts                 # User & Tenant interfaces
    │       ├── profile.ts              # Structured Profile, Experience, Project types
    │       ├── opportunity.ts          # Opportunity & Filter types
    │       ├── referral.ts             # Referral Search & Message types
    │       └── tracker.ts              # Application Record & KPI types
    │
    ├── package.json                    # Node dependencies & frontend scripts
    ├── tailwind.config.ts              # Tailwind CSS theme, animations & colors
    └── tsconfig.json                   # TypeScript compiler configuration
```
