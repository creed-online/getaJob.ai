# 🗺️ getJob.ai — Simplified 3D Application Sitemap & User Flow (V2 Production)

> **Design Philosophy:** Ultra-clean, minimal, and immersive. Every page is centered around interactive 3D visual environments (Spline / Three.js) with zero visual clutter, leading the user smoothly from onboarding to auto-application.

---

## 🚀 1. The Simplified 3D User Journey

```mermaid
flowchart TD
    subgraph S1 ["Screen 1: 3D Hero Portal (Spline Scene 1)"]
        Hero3D["Interactive 3D Visual Atmosphere"]
        HeroTitle["Centered Heading: 'Get a Job'"]
        BtnAuth["[ Login ]   [ Sign Up ] Buttons"]
    end

    subgraph S2 ["Screen 2: 3D Knowledge Base Onboarding (Spline Scene 2)"]
        KB3D["Clean 3D Backdrop (Zero Distraction)"]
        KBUpload["Upload Knowledge Base:\n• Master Resume (PDF/DOCX max 5MB)\n• GitHub Repos Connect\n• Research Papers & Transcripts (Optional)\n• Screening QA Vault"]
        BtnSave["[ Save & Discover Opportunities ➔ ]"]
    end

    subgraph S3 ["Screen 3: Opportunity Discovery Radar"]
        Radar3D["Clean 3D Themed Opportunity Feed"]
        Cards["Scored Cards (Adzuna Global Index + 500+ Company Boards + HN + Unstop)\n• Confidence Score Gauge (0–100%)\n• Posting Age Decay Filter (>21 days)"]
        BtnTailor["[ Review & Tailor Resume ➔ ]"]
    end

    subgraph S4 ["Screen 4: Validation & Tailoring Studio"]
        Studio["Deterministic Structural Audit + ATS Score (1–10)\n• Auto-Tuned Bullets (Google XYZ)\n• Company-Aligned Cover Letter\n• 📥 Direct Download Tailored Resume & CV (Typst PDF)"]
        BtnApply["[ 1-Click Auto-Apply 🚀 ]"]
    end

    subgraph S5 ["Screen 5: Auto-Apply & LinkedIn Referral Hub"]
        AutoApply["1-Click Apply for Greenhouse/Lever + Assisted Copilot for Workday\n(Pause only if custom unknown question)"]
        ReferralCards["5 Persona Referral Cards (Recruiter, Lead, Peer, Alumni)\n• Direct LinkedIn Search Deeplink (Opens user session ↗)\n• 📋 1-Click Copy Short Outreach Message"]
    end

    subgraph S6 ["Screen 6: Real-Time Application Tracker"]
        Tracker["KPI Counters (Today/Week/Month/Year)\n• Sortable Status Table (Applied, OA, Interview, Offer)\n• 📥 1-Click Export to Excel (.xlsx) & JSON"]
    end

    Hero3D --> HeroTitle --> BtnAuth
    BtnAuth -->|Auth Success (httpOnly Cookie)| KB3D --> KBUpload --> BtnSave
    BtnSave --> Radar3D --> Cards --> BtnTailor
    BtnTailor --> Studio --> BtnApply
    BtnApply --> AutoApply --> ReferralCards
    AutoApply --> Tracker
```

---

## 🖥️ 2. Detailed Screen-by-Screen Breakdown

### 🌟 Screen 1: 3D Hero Portal (`/`)
* **Visual Atmosphere:** Full-screen interactive 3D WebGL / Spline scene (`fcb3d45a-1ffd-474a-afbf-9643cc3a0d40`).
* **Center Layout:**
  * Bold typography in the dead center: **`Get a Job`**.
  * Tagline: *Your autonomous AI career copilot.*
  * Two sleek, glassmorphic action buttons directly beneath:
    * **`[ 🔐 Login ]`** ➔ Opens glass modal (Sets secure `httpOnly` `SameSite=Lax` cookie).
    * **`[ ✨ Sign Up ]`** ➔ Opens registration modal (Email, Name, Phone, Password, Confirm Password).
* **Action:** Upon successful authentication, the user is transitioned to **Screen 2**.

---

### 💾 Screen 2: 3D Knowledge Base Onboarding (`/onboarding`)
* **Visual Atmosphere:** Immersive, clean 3D backdrop based on Spline scene (`67f56854-67d0-4779-b5ed-371c2f5d169e`).
* **Design Rule:** **Zero clutter.** 100% focused on ingesting career memory with strict upload guardrails:
  * **Master Resume Dropzone:** Drag & drop PDF/DOCX (max 5MB, MIME checked) with instant parsing.
  * **GitHub Repositories:** One-click GitHub sync to analyze codebases and READMEs.
  * **Research Papers & Transcripts (Optional):** Attach academic publications and coursework.
  * **Screening QA Vault (Optional):** Pre-fill salary expectations, visa status, and behavioral answers.
* **Primary Action:** **`[ Save & Discover Opportunities ➔ ]`** ➔ Enqueues `arq` background vectorization and advances to Screen 3.

---

### 🌐 Screen 3: Opportunity Discovery Radar (`/dashboard`)
* **Visual Theme:** Consistent 3D dark-mode aesthetic with ambient lighting.
* **Data Sources (Zero-Ban & Reliable):**
  * **Adzuna Global Search API:** Covers thousands of active global job postings.
  * **Company Board Registry:** 500+ curated Greenhouse (`boards-api.greenhouse.io`) and Lever (`api.lever.co`) boards.
  * **Hacker News "Who is Hiring?":** Official Firebase JSON API.
  * **Unstop & Devpost:** Official RSS feeds for hackathons and hiring challenges.
* **Key Card Attributes:**
  * Job/Hackathon Title, Company, Location.
  * **Match Confidence Gauge (0–100%)** based on `pgvector` Cosine Similarity + BM25 keyword overlap.
  * **Posting Age Decay Filter:** Decays score if posted $>21$ days ago.
* **Primary Action:** Clicking any card opens the **Validation & Tailoring Studio**.

---

### ✍️ Screen 4: Opportunity Validation & Tailoring Studio (`/tailor/[id]`)
* **Visual Theme:** Minimalist split-panel view with real-time score indicators.
* **Content:**
  * **ATS Score Gauge (Scale 1–10):** Deterministic structural check (selectable text, standard headers) + Gemini keyword critique.
  * **Automatic Optimization (if Score < 8):** Injects required keywords and rewrites bullets into the Google XYZ impact format.
  * **Cover Letter Generator:** Custom letter matched to the company and role.
  * **Download Bar:** 
    * `📥 Download Tailored Resume (PDF)` (Compiled via Typst in <50ms).
    * `📥 Download Tailored CV / Cover Letter (PDF)`.
* **Primary Action:** **`[ 1-Click Auto-Apply 🚀 ]`** button.

---

### 🤖 Screen 5: 1-Click Auto-Apply & LinkedIn Referral Hub (`/referrals/[id]`)
* **Auto-Apply Flow:**
  * **Greenhouse & Lever:** Autonomous Playwright browser agent auto-fills fields, attaches the tailored Typst PDF, and submits.
  * **Workday & Complex Portals:** Assisted Apply Copilot drawer with pre-filled Answer Pack and 1-click clipboard triggers.
  * **Human-in-the-loop safety:** If an unknown custom question appears, the UI cleanly pauses and prompts the user for the answer.
* **LinkedIn Referral Hub (Accessible on-demand or post-apply):**
  * Displays 5 Persona Cards (Technical Recruiter, Engineering Manager, Team Peer, University Recruiter, Alumni).
  * Generates **Direct LinkedIn Search Deeplinks** (opens real-time search inside the user's logged-in session ↗).
  * Features a **`📋 1-Click Copy Short Message`** button with a concise (<75 words) personalized cold outreach note.

---

### 📊 Screen 6: Real-Time Application Tracker (`/tracker`)
* **Visual Theme:** Unified 3D glassmorphic data dashboard.
* **Content:**
  * **Real-time KPI metric counters:** `Applied Today`, `Applied This Week`, `Applied This Month`, `Applied This Year`.
  * **Sortable Data Table:** Filter by date, search companies, and update application outcomes (`Applied`, `No Response`, `Online Assessment`, `Interview`, `Rejection`, `Offer`).
  * **Export Actions:** `📥 Export to Excel (.xlsx)` and `📥 Export to JSON`.
