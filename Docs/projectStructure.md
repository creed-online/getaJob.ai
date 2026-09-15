getaJob.ai/
│
├── Docs/                               # Project blueprints & documentation
│   ├── workflow.md
│   ├── sitemap.md
│   ├── techStack.md
│   ├── IMPLEMENTATION_PLAN.md
│   └── workflowArchitecture.pdf
│
├── backend/                            # FastAPI Python Async API
│   ├── app/
│   │   ├── api/                        # REST & SSE API Route Controllers
│   │   │   ├── auth.py                 # Login, Register, Credentials, Delete User
│   │   │   ├── profile.py              # Resume upload, GitHub sync, QA Vault
│   │   │   ├── discovery.py            # Opportunity radar feed & trigger crawl
│   │   │   ├── tailor.py               # ATS score check & Typst PDF compiler
│   │   │   ├── apply.py                # 1-Click Auto-Apply runner & SSE stream
│   │   │   ├── referrals.py            # 5-Contact Referral finder & messages
│   │   │   └── tracker.py              # KPI metrics, applications table & Excel export
│   │   │
│   │   ├── core/                       # Core Infrastructure & Security
│   │   │   ├── config.py               # Settings & API keys (Gemini, Groq)
│   │   │   ├── database.py             # Async SQLAlchemy session factory
│   │   │   └── security.py             # Bcrypt hashing & JWT token provider
│   │   │
│   │   ├── models/                     # SQLAlchemy Database Tables
│   │   │   ├── user.py                 # User model (with subscription_tier & credits)
│   │   │   ├── profile.py              # UserProfile, WorkExperience, Project, QA
│   │   │   ├── opportunity.py          # Opportunity table (Jobs, Unstop, Research, X)
│   │   │   ├── tailored_doc.py         # Tailored resumes, ATS reports, cover letters
│   │   │   ├── referral.py             # ReferralContact table
│   │   │   └── application.py          # Application tracking table (statuses, dates)
│   │   │
│   │   ├── schemas/                    # Pydantic v2 Request/Response Schemas
│   │   │   ├── auth.py
│   │   │   ├── profile.py
│   │   │   ├── opportunity.py
│   │   │   ├── tailor.py
│   │   │   ├── referral.py
│   │   │   └── tracker.py
│   │   │
│   │   ├── services/                   # Business Logic & AI Engines
│   │   │   ├── ai_client.py            # Gemini 2.0 Flash & Groq SDK wrapper
│   │   │   ├── rag_engine.py           # FastEmbed local vector engine (RAM)
│   │   │   ├── profile_structurer.py   # LLM parser (Raw text ➔ Structured JSON)
│   │   │   ├── ats_evaluator.py        # 1–10 ATS scoring rubric
│   │   │   ├── tailoring_engine.py     # Google XYZ bullet optimizer & keyword injector
│   │   │   ├── pdf_compiler.py         # Typst PDF compiler (sub-50ms)
│   │   │   ├── referral_finder.py      # 5-persona company employee search
│   │   │   ├── outreach_generator.py   # Concise (<75 words) cold message drafter
│   │   │   ├── tracker_service.py      # Real-time auto-logger & KPI aggregations
│   │   │   ├── export_service.py       # openpyxl (.xlsx) & JSON generator
│   │   │   │
│   │   │   ├── parsers/                # Document Ingestion Parsers
│   │   │   │   ├── resume_parser.py    # PDF text/section extractor (pdfplumber)
│   │   │   │   ├── docx_parser.py      # Word doc extractor
│   │   │   │   ├── github_crawler.py   # GitHub repo & README crawler
│   │   │   │   └── doc_parser.py       # Research paper & transcript parser
│   │   │   │
│   │   │   ├── scrapers/               # Multi-Channel Discovery Scrapers
│   │   │   │   ├── job_scraper.py      # LinkedIn, Indeed, Wellfound, Direct ATS
│   │   │   │   ├── hackathon_scraper.py# Unstop, Devpost, Kaggle challenges
│   │   │   │   ├── research_scraper.py # CERN, NSF REU, Google/MS Research
│   │   │   │   ├── social_scraper.py   # X/Twitter & Hacker News hiring feeds
│   │   │   │   ├── normalizer.py       # SHA-256 dedupe & HTML cleaner
│   │   │   │   └── stale_filter.py     # Stale rule (>1000 apps & >2 weeks old)
│   │   │   │
│   │   │   └── browser_agent/          # Autonomous Auto-Apply (Playwright)
│   │   │       ├── agent.py            # Async browser session manager
│   │   │       ├── form_inspector.py   # DOM interactive field detector
│   │   │       ├── form_mapper.py      # Field to User KB resolver
│   │   │       ├── question_fallback.py# Unknown question pause & SSE emitter
│   │   │       └── adapters/           # Greenhouse, Lever, Workday, Generic
│   │   │
│   │   ├── templates/                  # Typst Templates for ATS Documents
│   │   │   ├── resume_ats.typ          # Single-column ATS resume template
│   │   │   └── cover_letter.typ        # Formal cover letter template
│   │   │
│   │   └── main.py                     # FastAPI application entry & CORS
│   │
│   ├── storage/                        # Local generated PDFs & uploads cache
│   │   └── generated/
│   ├── tests/                          # Automated Pytest Suite
│   ├── requirements.txt                # Python dependencies
│   └── .env.example                    # Environment configuration template
│
├── frontend/                           # Next.js 14 (App Router) + TypeScript
│   ├── src/
│   │   ├── app/
│   │   │   ├── page.tsx                # Screen 1: 3D Spline Hero Portal ("Get a Job")
│   │   │   ├── onboarding/             # Screen 2: 3D Knowledge Base Upload
│   │   │   │   └── page.tsx
│   │   │   ├── dashboard/              # Screen 3: Opportunity Discovery Radar Feed
│   │   │   │   └── page.tsx
│   │   │   ├── tailor/[id]/            # Screen 4: ATS Tailoring Studio & Downloads
│   │   │   │   └── page.tsx
│   │   │   ├── referrals/[id]/         # Screen 5: 5-Contact Referral Hub
│   │   │   │   └── page.tsx
│   │   │   ├── tracker/                # Screen 6: Real-Time Tracker & Excel Export
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx              # Root Layout & Font Providers
│   │   │   └── globals.css             # Tailwind Dark Theme Styles
│   │   │
│   │   ├── components/                 # Reusable React Components
│   │   │   ├── 3d/                     # Spline & Three.js Canvas Wrappers
│   │   │   │   ├── hero-spline.tsx     # Spline Scene 1 Component
│   │   │   │   └── onboarding-spline.tsx# Spline Scene 2 Component
│   │   │   ├── ui/                     # Shadcn UI primitives (Dialog, Tabs, etc.)
│   │   │   ├── auth-modals.tsx         # Login & Sign Up Modals
│   │   │   ├── opportunity-card.tsx    # Scored Opportunity Cards
│   │   │   ├── auto-apply-modal.tsx    # Live Browser Form Walker Modal
│   │   │   ├── custom-q-prompt.tsx     # Unknown Question Prompt Dialog
│   │   │   ├── referral-card.tsx       # 5-Persona Contact Card + Copy Button
│   │   │   └── tracker-table.tsx       # TanStack sortable applications table
│   │   │
│   │   ├── lib/
│   │   │   ├── api.ts                  # Axios client with JWT interceptor
│   │   │   ├── auth-store.ts           # Zustand global authentication store
│   │   │   └── utils.ts                # Tailwind cn() and formatting helpers
│   │   │
│   │   └── types/                      # Shared TypeScript Interfaces
│   │       ├── user.ts
│   │       ├── opportunity.ts
│   │       └── application.ts
│   │
│   ├── public/                         # Static icons and assets
│   ├── package.json                    # Node dependencies
│   ├── tailwind.config.ts              # Tailwind CSS configuration
│   └── tsconfig.json                   # TypeScript configuration
│
├── .gitignore                          # Standard git ignore for secrets/builds
└── readme.md                           # Master GitHub README