# getJob.ai — Detailed Build Guide

> Companion to `IMPLEMENTATION_PLAN.md` (V2 Production). That document says *what* to build.
> This document says *what order to build it in, how to test each piece before moving on,
> and what will break if you skip a step*. Work top to bottom. Don't start a stage until
> the previous stage's "Done when" checklist is fully true — not "mostly working."

**How to use this:** each stage has an ordered task list. Each task names the exact file,
what it needs to contain at a minimum, and a manual test you run in a terminal or browser
before touching the next task. Do not build a UI screen before its backend endpoint returns
real data via `curl`. The UI is the last five minutes of a stage, not the first.

---

## Stage 0 — Pre-flight (half a day)

Nothing here produces app code. It exists so you don't discover a broken dependency three
days into Stage 2.

### 0.1 Pin real versions
Every version in `techStack.md` was current as of the doc's writing — check each before you
install anything, because pins drift:
```bash
pip index versions google-genai
pip index versions fastembed
pip index versions playwright
pip index versions typst          # and separately: pip index versions typst-py
npm view next versions --json | tail -5
npm view zustand version
```
Decide `typst` vs `typst-py` now (they're different packages with different APIs) and use
that name consistently in every doc and every import. Write the final pins into
`requirements.txt` and `package.json` before Stage 1 — don't install ad hoc as you go, or
you'll get a different resolution on your laptop vs. wherever you deploy.

### 0.2 Get every key you'll need for the whole MVP, now
Doing this once avoids a mid-Stage-3 wait for an API key approval.
- **Google AI Studio** → `GEMINI_API_KEY`
- **Groq Console** → `GROQ_API_KEY`
- **Adzuna Developer Portal** → `ADZUNA_APP_ID` + `ADZUNA_APP_KEY` (approval can take a day — apply now even though you don't need it until Stage 3)
- **GitHub** → personal access token, `repo` + `read:user` scope only, for `GITHUB_TOKEN`
- Put all of these in `backend/.env` immediately, and `backend/.env.example` with blank values, and confirm `.env` is in `.gitignore` before your first commit.

### 0.3 Prove the infra works before writing app code
```yaml
# docker-compose.yml — minimum viable version
services:
  db:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_USER: getjob
      POSTGRES_PASSWORD: devpassword
      POSTGRES_DB: getjob_dev
    ports: ["5432:5432"]
    volumes: ["pgdata:/var/lib/postgresql/data"]
  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
volumes:
  pgdata:
```
```bash
docker-compose up -d
psql postgresql://getjob:devpassword@localhost:5432/getjob_dev -c "CREATE EXTENSION IF NOT EXISTS vector;"
psql postgresql://getjob:devpassword@localhost:5432/getjob_dev -c "SELECT '[1,2,3]'::vector;"
redis-cli ping
```
If the `vector` cast returns a row and Redis returns `PONG`, the foundation is real. If
`pgvector/pgvector:pg16` isn't pullable, fall back to plain `postgres:16` and
`CREATE EXTENSION vector` will fail loudly — that's your signal to switch images, not debug
further.

### 0.4 Apply the doc corrections before you start reading the plan as truth
- Fix the age-decay branch order in `techStack.md` §2.1 — check `>45` before `>21`, or every posting older than 45 days silently gets the *milder* 50% penalty instead of the intended 80%.
- Add `apply_method: enum("autonomous","assisted","external_link")` to the `Opportunity` model description in `projectStructure.md` — Adzuna results mostly redirect to aggregators, not clean forms, and Screen 4's apply button needs to know which UI to show.
- Reconcile `services/opportunity_normalizer.py` (IMPLEMENTATION_PLAN) vs `services/scrapers/normalizer.py` (projectStructure) — pick one path, edit both docs to match.
- Add one line to the top of `workflow.md`: *"Superseded by V2 — retained for original product intent only. Discovery, referrals, and stale-filter sections no longer reflect the build."*
- Decide Ashby in/out. If out, remove `ashby_api.py` from `techStack.md`'s subsystem table. If in, add a task ID for it in Phase 3.
- Pick one word count for outreach messages (`<75 words` vs `50-word`) and make sitemap.md and techStack.md agree.

### 0.5 Decide deployment targets now (constrains later choices)
Vercel cannot run FastAPI + Playwright. Decide now so `main.py`'s CORS config and the
Next.js path-proxy rewrite are written against real domains from the start:
- **Frontend:** Vercel
- **Backend:** Railway or Render, Docker image based on `mcr.microsoft.com/playwright/python` (ships Chromium + deps preinstalled — building your own Playwright image from scratch wastes a day)
- **Postgres:** Neon or Supabase (both have managed pgvector)
- **Redis:** Upstash

**Done when:** `psql` returns a vector row, `redis-cli ping` returns `PONG`, every `.env` key
is populated with a real value, and the six doc corrections above are made.

---

## Stage 1 — Walking skeleton (Phase 1, ~1 week)

Goal: a user can register, log in, land on an empty `/onboarding` page, and a background
job can be triggered and observed completing. Nothing AI-related yet.

### Order of construction
1. **`docker-compose.yml`** — already done in Stage 0.
2. **`backend/app/core/config.py`** — `Settings(BaseSettings)` loading `DATABASE_URL`, `REDIS_URL`, `JWT_SECRET`, `GEMINI_API_KEY`, `GROQ_API_KEY`, `ADZUNA_APP_ID`, `ADZUNA_APP_KEY`, `GITHUB_TOKEN`, `COOKIE_DOMAIN`, `CORS_ORIGINS`.
   - Test: `python -c "from app.core.config import settings; print(settings.DATABASE_URL)"` prints the real value, not `None`.
3. **`backend/app/core/database.py`** — async engine + `get_db_session()` dependency.
   - Test: a throwaway script that opens a session and runs `SELECT 1`.
4. **`alembic init alembic`**, then configure `env.py` for async + point it at your `Base.metadata`.
   - This must exist *before* your first model, so the first migration is `alembic revision --autogenerate` against an empty schema, not a hand-written one.
5. **`backend/app/models/user.py`** — `User` table: `id (UUID)`, `email (unique)`, `full_name`, `phone`, `hashed_password`, `subscription_tier (default "free")`, `credits_remaining (default 5)`, `created_at`.
6. **First migration**: `alembic revision --autogenerate -m "create users table"`, then `alembic upgrade head`. Confirm the table exists: `psql ... -c "\d users"`.
7. **`backend/app/core/security.py`** — `get_password_hash()`, `verify_password()`, `create_access_token()`, `decode_access_token()`.
8. **`backend/app/schemas/auth.py`** — `UserRegisterSchema`, `UserLoginSchema`, `TokenResponse`. Use Pydantic's `EmailStr` and a regex for phone.
9. **`backend/app/api/auth.py`** — `register()`, `login()` (sets `httpOnly`, `SameSite=Lax`, `Secure` cookie), `get_me()`, `logout()` (clears cookie).
10. **`backend/app/main.py`** — FastAPI instance, CORS configured for your real frontend origin (from Stage 0.5), router mounting, global exception handler.
    - Test end-to-end with curl before any frontend exists:
    ```bash
    curl -c cookies.txt -X POST localhost:8000/api/auth/register -H "Content-Type: application/json" \
      -d '{"email":"you@test.com","password":"testpass123","full_name":"Test","phone":"+911234567890"}'
    curl -b cookies.txt localhost:8000/api/auth/me
    ```
    Second call must return your user, proving the cookie round-trips.
11. **`backend/app/worker.py`** — arq `WorkerSettings` with one dummy job, e.g. `async def ping(ctx): print("worker alive")`.
    - Run it in a second terminal: `arq app.worker.WorkerSettings`. Enqueue the job from a throwaway script or a temporary `/debug/ping` endpoint and confirm the print shows up in the worker terminal, not the API terminal. This is the check that proves your queue architecture actually works before you build anything real on top of it.
12. **`backend/app/services/credit_meter.py`** — `CreditMeterService.check_and_decrement(user_id, cost)`. Minimal version: read `credits_remaining`, raise `HTTPException(402)` if insufficient, decrement otherwise. Wire it nowhere yet — just have it ready.
13. **`backend/app/core/upload_guard.py`** — `validate_upload_file(file, max_mb=5)` checking size and MIME (`application/pdf`, the docx MIME string). Also nowhere yet — ready for Stage 2.
14. **Frontend scaffold**: `npx create-next-app@latest` with the pinned versions, Tailwind + Shadcn init, `next.config.js` rewrite `/api/:path*` → your backend URL (this is what makes `SameSite=Lax` cookies work across dev ports).
15. **`frontend/src/lib/api.ts`** — Axios instance with `withCredentials: true`.
16. **`frontend/src/lib/auth-store.ts`** — Zustand store holding user object (not the token — it's in the cookie, invisible to JS by design).
17. **`frontend/src/components/auth-modals.tsx`** + **`frontend/src/app/page.tsx`** — plain dark hero, no Spline yet. Title, tagline, two buttons, two modals wired to the API.
18. **`frontend/src/app/onboarding/page.tsx`** — can be a literal empty page with "Onboarding" as its only content. It exists to prove the redirect works.

**Manual smoke test for the whole stage:** open the browser, register a new user through the
UI, confirm you land on `/onboarding`, refresh the page and confirm you're still logged in
(cookie persistence), then hit a `/debug/ping`-style endpoint that enqueues the dummy arq job
and watch it fire in the worker terminal.

**Done when:** register → login → cookie set → `/onboarding` renders in the *browser*, and the
dummy job fires from a real HTTP request, not just a script.

---

## Stage 2 — Knowledge base (Phase 2, ~1.5 weeks)

Goal: upload a resume, get back structured data and vector embeddings sitting in Postgres.
This is the stage where AI output quality determines whether the rest of the product works —
spend real time on the prompt before wiring any UI.

### Order of construction
1. **`backend/app/services/parsers/resume_parser.py`** — `PDFResumeParser.extract_text()` using `pdfplumber`. Test standalone first: run it against your own resume PDF from a script, print the raw text, and eyeball whether section boundaries survived.
2. **`backend/app/services/parsers/docx_parser.py`** — same idea with `python-docx`.
3. **`backend/app/services/ai_client.py`** — thin wrapper around the Gemini SDK using the version you pinned in Stage 0. Confirm the current SDK's call shape before writing this — `google-genai` changed argument names across minor versions, so copy from the SDK's own README, not from memory.
4. **`backend/app/schemas/profile.py`** — `StructuredProfileSchema` **written first, before the prompt**. Define exactly what fields you need (name, contact, work experience list with company/title/dates/bullets, education, skills, projects) as a strict Pydantic model.
5. **`backend/app/services/profile_structurer.py`** — `ProfileExtractionService` that sends parsed text to Gemini with `response_schema` set to your Pydantic schema, gets back typed JSON.
   - **Test before any DB or UI work:** write a standalone script that runs your own resume through parser → structurer → prints the resulting object. Then do the same for three resumes you download from a template site (different formats, different section orders). If the schema breaks on any of them, fix the prompt now — this is the cheapest point in the whole project to fix it.
6. **`backend/app/services/parsers/github_crawler.py`** — `crawl_user_repos()`, `fetch_readme()` via `httpx` against the GitHub REST API using your PAT.
7. **`backend/app/services/parsers/doc_parser.py`** — research papers/transcripts, reuses the PDF parser with a different downstream prompt.
8. **`backend/app/models/profile.py`** — `UserProfile`, `WorkExperience`, `Project`, `ScreeningQA` tables. Include an `embedding` column (`Vector(384)`, matching FastEmbed's `bge-small-en-v1.5` output dimension) on whichever table needs semantic search — likely `Project` and `WorkExperience`.
9. **Second migration**: `alembic revision --autogenerate -m "profile tables"` → `alembic upgrade head`.
10. **`backend/app/services/rag_engine.py`** — `LocalRAGEngine.embed_text()`, `embed_batch()` wrapping FastEmbed, plus a `cosine_similarity_search()` that queries via pgvector's `<->` operator, not Python-side cosine — you already decided against the SQLite/pgvector split, don't reintroduce it here by computing similarity in Python.
11. **Wire `job_ingest_resume_rag` into `worker.py`**: parse → structure → save rows → embed → save vectors, all as one arq job so the HTTP request returns immediately.
12. **`backend/app/api/profile.py`** — `upload_resume()` (runs `upload_guard` first, then enqueues the job and returns immediately with a job/status id), `sync_github()`, `get_profile_kb()`.
13. **`backend/app/api/qa_vault.py`** — CRUD for screening Q&A, no AI involved, straightforward.
14. **Frontend**: `frontend/src/app/onboarding/page.tsx` real version — dropzone, GitHub connect button, QA vault drawer, "Save & Discover" button. Poll or SSE the job status so the button shows real progress instead of a fixed spinner.

**Manual test:** upload your actual resume through the browser UI, then query Postgres
directly:
```sql
SELECT full_name, company, title FROM work_experiences WHERE user_id = '<your-id>';
SELECT id, embedding IS NOT NULL AS has_vector FROM projects WHERE user_id = '<your-id>';
```

**Done when:** the structured rows match your real resume with no hallucinated companies or
dates, and every project row has a non-null embedding.

---

## Stage 3 — Discovery (Phase 3, ~1.5 weeks)

Goal: a dashboard of real job postings, scored against your actual profile from Stage 2, in
an order that makes sense to you as a human looking at it.

### Order of construction
1. **`backend/app/services/scrapers/company_registry.py`** — start with **30 companies you verify by hand**, not 500. For each: `curl https://boards-api.greenhouse.io/v1/boards/{slug}/jobs` and confirm you get JSON back before adding it to the list. Greenhouse slugs are usually the lowercase company name but not reliably — check each one. Do the same for Lever's `api.lever.co/v0/postings/{slug}`. This manual verification pass *is* the real work of this task; don't skip it by guessing slugs.
2. **`backend/app/services/scrapers/greenhouse_api.py`** — `GreenhouseAPIClient.fetch_jobs(slug)`, thin `httpx` wrapper, one function.
3. **`backend/app/services/scrapers/lever_api.py`** — same shape for Lever.
4. **`backend/app/services/scrapers/normalizer.py`** (pick this path per your Stage-0 doc fix) — `OpportunityNormalizerService`: SHA-256 hash of `company + title + location` for dedupe, HTML-strip descriptions.
5. **`backend/app/models/opportunity.py`** — `Opportunity` table: `platform (enum)`, `company`, `title`, `location`, `description`, `url`, `posted_at`, `confidence_score`, `apply_method (enum: autonomous/assisted/external_link)` — this last field is the fix from Stage 0.4; set it based on which scraper produced the row (Greenhouse/Lever → `autonomous`, everything else → `external_link` for now).
6. **Third migration.**
7. **`backend/app/services/stale_filter.py`** — `StaleAgeDecayService`, decay multiplier by posting age, **check the `>45` branch before `>21`** (the fix from Stage 0.4).
8. **`backend/app/services/scoring_engine.py`** — `OpportunityScoringService`: pgvector cosine similarity between the JD embedding and the user's project/experience embeddings, blended with BM25 keyword overlap via `rank-bm25`, multiplied by the stale decay factor. Land on one formula and write it down in a comment — you'll want to tune weights later and need to know what you started with.
9. **Wire `job_sync_discovery_feeds` into `worker.py`** — loops the company registry, calls both API clients, normalizes, stale-filters, scores, upserts.
10. **`backend/app/api/discovery.py`** — `get_feed()` (paginated, sorted by score), `trigger_sync()` (enqueues the job).
11. **Frontend**: `frontend/src/app/dashboard/page.tsx` and `opportunity-card.tsx` — category pills, confidence badge, stale indicator, click → `/tailor/[id]`.
12. Only after Greenhouse + Lever are visibly working end-to-end: add `adzuna_api.py`, `hn_api.py`, `hackathon_rss.py` — each is additive and independent, don't let them block the core loop.

**Manual test:** trigger a sync, then look at the dashboard as a human. Do the top-scored jobs
actually look like a fit for your resume? If the ranking looks random, your scoring formula
or your embeddings from Stage 2 are wrong — debug here, don't push forward with a broken
signal.

**Done when:** the dashboard shows real postings from at least Greenhouse and Lever, sorted
in an order you'd trust, with visibly different scores for a clearly-relevant vs.
clearly-irrelevant posting.

---

## Stage 4 — Tailoring studio (Phase 4, ~2 weeks)

Goal: click a job, get back an ATS score, an optimized resume, and a downloadable PDF you
would genuinely submit.

### Order of construction
1. **`backend/app/templates/resume_ats.typ`** — write this **by hand first**, with your own resume's data hardcoded in, before any Python touches it. Compile manually: `typst compile resume_ats.typ test.pdf`, then `pdftotext test.pdf - ` and confirm the extracted text is clean and in the right order (this is what an ATS parser will see). Iterate on the template until this is true — this is a design problem, not a code problem, and it's much faster to iterate on directly in Typst than through a Python wrapper.
2. **`backend/app/services/pdf_compiler.py`** — `PDFCompilerService` wrapping `typst-py` (or `typst`, per your Stage-0 decision), templated with Jinja2, taking structured resume data and producing bytes.
3. **`backend/app/services/ats_evaluator.py`** — build the **deterministic half first**: does the PDF have selectable text (re-extract and check length > 0), standard section headers, parseable dates, single-column layout (heuristic: consistent left-margin x-coordinates from `pdfplumber`). This part needs no LLM and should be fully unit-testable. Add the **LLM keyword/impact critique** second, as a separate function that produces a sub-score, and combine the two into the final 1–10 with a documented formula (same discipline as the scoring engine in Stage 3 — write the formula in a comment).
4. **`backend/app/services/tailoring_engine.py`** — `ResumeOptimizerService`: pulls top-matching projects via `rag_engine` from Stage 2, rewrites bullets into Google-XYZ format, injects missing keywords the ATS evaluator flagged.
5. **`backend/app/templates/cover_letter.typ`** + **`backend/app/services/cover_letter_gen.py`**.
6. **`backend/app/models/tailored_doc.py`** — `TailoredDocument`, `ATSReport`. Fourth migration.
7. **`backend/app/schemas/tailor.py`**, then **`backend/app/api/tailor.py`** (`evaluate_ats()`, `optimize_resume()`, `generate_cv()`) and **`backend/app/api/downloads.py`**.
8. **Frontend**: `frontend/src/app/tailor/[id]/page.tsx`, score gauge, download bar. Build `resume-diff-editor.tsx` last — it's the least load-bearing part of this screen.

**Manual test:** run a real job posting from your Stage-3 dashboard through the full pipeline,
download the PDF, open it, and ask yourself honestly whether you'd attach it to a real
application. Also paste its text into any free online ATS-checker to sanity-check your
deterministic scorer isn't wildly out of line with an external tool.

**Done when:** you would actually submit the generated PDF to a real employer, and the ATS
score changes sensibly when you deliberately remove a keyword from the job description test
input.

---

## Stage 5 — Tracker (Phase 7, ~1 week)

Goal: straightforward CRUD and aggregation — the least risky stage in the MVP. No AI calls.

### Order of construction
1. **`backend/app/models/application.py`** — `Application`: `company`, `role`, `platform`, `status (enum)`, `applied_at`, `notes`. Fifth migration.
2. **`backend/app/services/tracker_service.py`** — `log_application_event()` (called from wherever applications get created — for now, a manual-add endpoint, since auto-apply is Stage 7), `get_kpi_metrics()` (SQL date-range aggregations — do this in SQL, not by loading all rows into Python), `list_user_applications()`.
3. **`backend/app/services/export_service.py`** — `export_to_excel()` via `openpyxl`, `export_to_json()`.
4. **`backend/app/schemas/tracker.py`**, then **`backend/app/api/tracker.py`**.
5. **Frontend**: `kpi-cards.tsx`, `data-table.tsx` (TanStack, sortable/filterable), `add-application-modal.tsx`, `export-dropdown.tsx`.

**Manual test:** add a handful of applications with different dates and statuses through the
UI, confirm the KPI counters match a manual count, export both formats and open them.

**Done when: this is your MVP ship line.** Everything above (Stages 1–5) is a complete,
usable product: upload a resume, find real jobs, get a tailored ATS-clean PDF, track what you
applied to. **Deploy it and use it for two weeks of real job applications before writing
another line of code.** You will find more real bugs in two weeks of genuine use than in
another month of building — and you'll form an informed opinion about whether Stage 7
(auto-apply) is even worth its risk before you sink two more weeks into it.

---

## Stage 6 — Harden before anyone else touches it

Do this before you deploy, or before anyone besides you logs in — whichever comes first.

- **Password reset + email delivery** — Resend's free tier is enough for this volume. No task for this exists anywhere in the 88-task plan; add it.
- **Login rate limiting** — credits protect against LLM cost abuse, not brute-force login attempts. A simple per-IP + per-email attempt counter in Redis is enough.
- **Error tracking** — Sentry free tier, wired into both FastAPI and Next.js.
- **Frontend error boundaries** — so a single broken component doesn't white-screen the whole app.
- **Actually deploy** to the targets decided in Stage 0.5. Confirm the cookie survives the real cross-domain setup (`SameSite=Lax` + the Next.js proxy rewrite) — this is the single most common thing that works in local dev and silently breaks in production.
- **CI** running `pytest` on every push, even just GitHub Actions with no deploy step yet.

**Done when:** you can hand the deployed URL to a friend and they can register, upload a
resume, and see a dashboard without you standing over their shoulder.

---

## Stage 7 — Auto-apply (Phase 5, post-MVP)

Only start this once Stage 5's two-week real-use period has told you it's worth building.

### Order of construction
1. **The Redis pub/sub bridge, first, before any Playwright code.** This is the piece that got missed in the doc review: the SSE endpoint lives in the API process, the browser agent runs in the arq worker process, and `asyncio.Event` does not cross processes. Build and test this in isolation:
   - Worker publishes progress to a Redis channel `apply:{run_id}`.
   - `backend/app/api/apply_stream.py`'s SSE endpoint subscribes to that channel and forwards messages to the browser.
   - The user's answer to an unknown question goes back via a Redis key the worker is blocking on (`BLPOP` or similar), not an in-process event.
   - Test this with a fake job that just publishes three messages a second apart and confirm the frontend receives them live, before wiring any actual browser automation to it.
2. **`backend/app/services/browser_agent.py`** — `PlaywrightBrowserAgent`, stealth mode, using the Playwright-prebuilt Docker image from Stage 0.5 so you're not fighting Chromium installs.
3. **`backend/app/services/form_inspector.py`** and **`form_mapper.py`**.
4. **`backend/app/services/portal_adapters/greenhouse.py`** — build and fully test against **one real Greenhouse-hosted job posting end to end**, including the unknown-question pause/resume path, before writing the Lever adapter. Getting one portal completely solid tells you far more than getting two portals half-working.
5. **`backend/app/services/portal_adapters/lever.py`** — once Greenhouse is solid.
6. **`backend/app/services/question_fallback.py`**, **`backend/app/api/apply.py`**, **`backend/app/services/event_bus.py`** (auto-triggers the tracker log on successful submit).
7. **Frontend**: `auto-apply-modal.tsx` with live step indicators fed by the SSE bridge, `custom-question-prompt.tsx`.
8. **`backend/app/services/browser_agent/copilot_pack.py`** for Workday/Taleo — this is independent of the Playwright automation and considerably cheaper to build (it's a formatted answer pack + clipboard buttons, no DOM automation). Worth doing early in this stage, even in parallel, if the Greenhouse adapter is fighting you.

**Done when:** a real application to a real Greenhouse-hosted job gets submitted end to end,
including successfully pausing on an unknown custom question and resuming after you answer
it in the UI.

---

## Stage 8 — Referrals (Phase 6), then 3D polish

Do these last — both are genuinely optional polish at this point in the product, and both
are cheap now that the risky parts of each are resolved (referrals has no PII/legal exposure
left; the app already works without Spline).

### Referrals (~2 days)
1. **`backend/app/services/referral_finder.py`** — `generate_linkedin_search_urls()`: five URL-encoded LinkedIn people-search links per persona (recruiter, eng manager, peer, university recruiter, alumni), built from company name + role keywords. No scraping, no stored PII.
2. **`backend/app/services/outreach_generator.py`** — short Groq-drafted message per persona, using the word count you locked in during Stage 0.4.
3. **`backend/app/models/referral.py`** — stores the generated search query and message, not any person's data.
4. **`backend/app/api/referrals.py`**, then `frontend/src/app/referrals/[id]/page.tsx` + `referral-card.tsx` with the deeplink and copy-message button.

### 3D polish (last, always)
- Add Spline to the hero and onboarding screens only after everything else works.
- **Measure LCP on a real mid-range Android device before committing**, not just on your dev laptop. If it's bad, keep the plain hero for the landing page (where speed matters most for a first impression) and reserve Spline for onboarding (where the user is already committed and waiting is more tolerable).

---

## Quick-reference: what "done" means at each MVP gate

| Stage | You know it's done when... |
|---|---|
| 0 | `psql` runs a vector query, Redis pings, all keys are populated |
| 1 | Register→login→cookie→`/onboarding` works in a real browser; a real HTTP call fires a background job |
| 2 | Your real resume produces correct structured rows and non-null embeddings in Postgres |
| 3 | The dashboard ranks a clearly-relevant job above a clearly-irrelevant one, using real Greenhouse/Lever data |
| 4 | You'd actually submit the generated PDF to a real employer |
| 5 | **MVP ships.** You use it for two real weeks before Stage 6+ |
| 6 | A friend can use the deployed app without you present |
| 7 | One real Greenhouse application submits end-to-end, including the pause/resume flow |
| 8 | Referral deeplinks and 3D scenes are additive polish on a product that already works without them |

## General rules for the whole build

- **Backend before frontend, every time.** Endpoint → `curl` → confirm real data → then UI.
- **One real Greenhouse company before "500+ companies."** One real Greenhouse adapter before "also Lever."
- **Write the Pydantic schema before the LLM prompt that fills it**, in every stage that touches an LLM.
- **Any formula you write (scoring, decay, ATS blend) gets a comment explaining it.** You will tune these later and need to remember what you started with.
- **Commit with task IDs** (`TASK-2.07: local RAG engine`) so `IMPLEMENTATION_PLAN.md`'s tracker table reflects reality without you maintaining it separately.
