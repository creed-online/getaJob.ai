<div align="center">

# 🚀 getJob.ai — Autonomous 3D AI Career Copilot

[![Next.js](https://img.shields.io/badge/Next.js_14-black?style=for-the-badge&logo=next.js&logoColor=white)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)](https://fastapi.tiangolo.com/)
[![Gemini](https://img.shields.io/badge/Gemini_2.0_Flash-4285F4?style=for-the-badge&logo=google&logoColor=white)](https://aistudio.google.com/)
[![Playwright](https://img.shields.io/badge/Playwright-2EAD33?style=for-the-badge&logo=playwright&logoColor=white)](https://playwright.dev/)
[![Typst](https://img.shields.io/badge/Typst_PDF-239DAD?style=for-the-badge)](https://typst.app/)

<p align="center">
  <b>Stop manually applying to hundreds of jobs. Let AI discover opportunities, tune your resume for ATS, auto-fill applications, and discover strategic referrals in 1 click.</b>
</p>

[Explore Documentation](./docs/IMPLEMENTATION_PLAN.md) • [View Tech Stack](./docs/techStack.md) • [Sitemap Flow](./docs/sitemap.md)

</div>

---

### ✨ Key Features

- 🌐 **Multi-Channel Opportunity Radar:** Aggregates traditional jobs, hiring hackathons (Unstop, Devpost), and research fellowships (CERN, NSF REU, Google/MS Research).
- 🚫 **Stale Opportunity Filter:** Automatically filters ghost jobs (>1,000 applicants & posted >2 weeks ago).
- 🎯 **RAG Resume & CV Tailoring Studio:** Evaluates baseline ATS score (1–10). If < 8, automatically injects required keywords and rewrites bullets using the Google XYZ impact formula without hallucinating.
- 📑 **Sub-50ms ATS PDF Compilation:** Compiles pixel-perfect, machine-readable single-column resumes using Typst.
- 🤖 **1-Click Auto-Apply Engine:** Playwright browser agent navigates multi-page portal forms gracefully, pausing only when encountering unanswerable custom questions.
- 🤝 **5-Contact Referral Finder:** Post-application discovery of 5 key company contacts (Recruiters, Leads, Peers, Alumni) with pre-crafted 1-click copy outreach messages.
- 📊 **Real-Time Application Tracker:** Live KPI counters (Today, Week, Month, Year), status tracking table, and 1-click Excel (.xlsx) / JSON export.