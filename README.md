# FreshFit — AI Job Matcher

**Live app → [vidyullathaks.github.io/FreshFit-Job-Matcher](https://vidyullathaks.github.io/FreshFit-Job-Matcher/)**

FreshFit is a single-page web app that pulls job postings from the last 24 hours, scores each one against your uploaded resume using an AI matching algorithm, and then rewrites your resume — in your own format — to be ATS-optimised for the role you want to apply to. No sign-up, no backend, no data ever leaves your browser.

---

## Why it exists

Most job boards surface the same listings regardless of your background. You still have to open dozens of tabs, manually judge whether a role fits, and then rewrite your resume from scratch for each application. FreshFit collapses all three steps — **find, match, tailor** — into one tool that you run in under a minute.

---

## What it does

### 1. Resume Management
- Upload one or more resumes as **PDF** or plain text.
- PDF text is extracted client-side using PDF.js — no file ever leaves your browser.
- Resumes are automatically parsed for section headers (Summary, Experience, Education, Skills, Certifications, etc.) and stored in `localStorage`.
- You can add, edit, and delete resumes, and label them by role (e.g. "Senior PM Resume", "Data Analyst CV").

### 2. Fresh Job Fetching (last 24 hours only)
Click **Find Jobs** to pull fresh postings from up to six sources simultaneously:

| Source | Type | API key needed? |
|---|---|---|
| **RemoteOK** | Remote-only jobs (all roles) | No |
| **Arbeitnow** | English-language international jobs | No |
| **Remotive** | Remote jobs by category | No |
| **Adzuna** | Broad job board with salary data | Optional (free) |
| **LinkedIn via Apify** | LinkedIn job search | Optional (free tier) |
| **Company Career Pages via Apify** | Google Jobs scraper for specific companies | Optional (free tier) |

Results are deduplicated by URL, **filtered by your configured location**, sorted by fit score, and cached in `localStorage` so the last fetch is always available without re-fetching.

### 3. AI Fit Scoring
Every job is scored 5–99% against the best matching resume in your library. The algorithm runs entirely in the browser (no API call needed):

**Score breakdown (100 points total):**
- **60 pts — Core role match:** Extracts the core job title by stripping domain qualifiers (e.g. "Product Manager - Renewable Energy Platform (AI-Native)" → `product manager`). Scores unigram matches (40%) + bigram phrase matches (60%) between the core role and your resume keywords.
- **30 pts — Requirements section match:** Finds the "Requirements / Qualifications / What you bring" section of the job description specifically and scores it against your resume keywords — avoiding dilution from boilerplate "About Us" and "What you'll do" text.
- **10 pts — Remote bonus:** Jobs tagged or located as remote get a +10 bonus.

Score labels: **Strong Fit** (≥70%) · **Moderate Fit** (≥45%) · **Low Fit** (<45%)

Cached scores are automatically **re-evaluated on every page load** using the latest algorithm, so stale scores from previous sessions are always refreshed.

### 4. Location Filtering
After fetching, jobs are filtered against your configured **Preferred Location**:
- If set to `Remote`, all jobs pass through.
- If set to a country (e.g. `United States`), jobs where the location is clearly in a different country (e.g. "Berlin, Germany", "London, UK", "Bangalore, India") are excluded. Remote/worldwide/blank locations always pass through.
- Supports smart matching for US, UK, Canada, and Australia out of the box (major cities, state abbreviations, and country name variants).

### 5. Dashboard Filters
Once jobs are loaded, the dashboard lets you narrow down by:
- **Keyword** — free-text search across title, company, description, and tags.
- **Source** — filter to a specific job board (RemoteOK, LinkedIn, etc.).
- **Fit Score** — show only Strong Fit (70%+) or Moderate Fit (50%+).

### 6. Job Detail View
Click any job card to open a full-screen detail panel showing:
- Full job description
- Fit score and label
- Location, posted time, source badge
- Salary (when available from Adzuna)
- Which resume matched best
- A direct **Apply** link to the original posting
- A **Tailor Resume with AI** button

### 7. AI Resume Tailoring (Groq)
When you click **Tailor Resume with AI** on a job:
1. Your best-matching resume and the job description are sent to **Groq's API** (free, no credit card needed) running the `llama-3.3-70b-versatile` model.
2. The AI rewrites your resume with:
   - Exact terminology mirrored from the job description (ATS keyword matching).
   - Your original section order, bullet style, date formats, and capitalisations preserved.
   - Content trimmed to fit on **one US Letter page** (0.55" margins).
   - Your real employer names, job titles, and qualifications never fabricated.
3. The tailored output appears on-screen for review.

### 8. Print-to-PDF
After tailoring, click **Print / Save as PDF** to open a clean print view formatted with proper typography and a "Tailored for: [role] at [company]" footer. Use the browser's **Save as PDF** print destination to generate a file ready to attach to applications.

### 9. PIN Lock Screen
An optional PIN can be set for shared or public computers. The PIN is hashed with **SHA-256** via the Web Crypto API — the plaintext is never stored. Forgetting the PIN requires clearing browser `localStorage`.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **UI framework** | [Tailwind CSS v3](https://tailwindcss.com) via CDN |
| **PDF parsing** | [PDF.js 3.11](https://mozilla.github.io/pdf.js/) via CDN — fully client-side |
| **AI inference** | [Groq API](https://console.groq.com) — `llama-3.3-70b-versatile`, free tier |
| **Job scraping** | [Apify](https://apify.com) — LinkedIn scraper actors + Google Jobs actor |
| **Free job APIs** | RemoteOK, Arbeitnow, Remotive, Adzuna |
| **CORS proxy** | `api.allorigins.win` (used for RemoteOK only) |
| **Storage** | Browser `localStorage` — all data stays in the user's browser |
| **Crypto** | Web Crypto API (`SubtleCrypto.digest`) for PIN hashing |
| **Deployment** | GitHub Pages (static, no server) |
| **Language** | Vanilla JavaScript (ES2020+), HTML5, CSS3 — zero build tools, zero dependencies |

---

## Repository Structure

```
FreshFit-Job-Matcher/
└── index.html          # The entire application — all HTML, CSS, and JS in one file
```

That's it. The entire app is a single self-contained HTML file with no build step, no `node_modules`, no bundler, and no backend. Open the file in any browser and it works.

### Inside `index.html`

The script is organised into numbered tasks:

| Section | What it covers |
|---|---|
| **Task 3** | App state object + `localStorage` persistence (`ff_resumes`, `ff_settings`, `ff_jobs`, `ff_last_fetch`) |
| **Task 4** | PDF parsing with PDF.js; section header detection with regex patterns |
| **Task 5** | Resume CRUD — add, edit, delete, save |
| **Task 6** | Settings form — read/write all configuration to state |
| **Task 7** | Job fetching — `fetchRemoteOK`, `fetchArbeitnow`, `fetchRemotive`, `fetchAdzuna`, `fetchApifyLinkedIn`, `fetchApifyCareerPages`; `isWithin12h` time filter; `isLocationMatch` country filter |
| **Task 8** | Fit scoring — `extractKeywords`, `extractCoreRole`, `extractRequirementsSection`, `scoreResumeVsJob`, `bestResumeFor`, `scoreLabel` |
| **Task 9** | Dashboard rendering — `getFilteredJobs`, `renderDashboard`, `renderJobCard` |
| **Task 10** | Job detail modal — `openJobDetail` |
| **Task 11** | Groq integration — `callGroq`, `buildTailorPrompt`, `tailorWithAI` |
| **Task 12** | Tailor modal + print-to-PDF layout |
| **Task 13** | Modal system, navigation (`showView`), toast notifications, PIN logic, `init` |

---

## Setup & Usage

### Minimum setup (no API keys needed)
1. Open [the live app](https://vidyullathaks.github.io/FreshFit-Job-Matcher/) in your browser.
2. Click **Upload Your Resume** and upload a PDF or paste plain text.
3. Go to **Settings** and set your **Job Titles** (e.g. `Product Manager, Senior PM`) and **Preferred Location** (e.g. `United States` or `Remote`).
4. Click **Find Jobs**. RemoteOK, Arbeitnow, and Remotive will return results immediately — no keys required.

### Recommended: add Groq (free, for AI tailoring)
1. Create a free account at [console.groq.com](https://console.groq.com) — no credit card.
2. Generate an API key and paste it into **Settings → Groq API Key**.
3. Click **Tailor Resume with AI** on any job card.

### Optional: add Apify (free tier, for LinkedIn + company career pages)
1. Sign up free at [apify.com](https://apify.com) — $5 free credit/month, no card needed.
2. Go to **Settings → Integrations → API token** in the Apify console.
3. Paste the token into **Settings → Apify API Token** in FreshFit.
4. The app automatically tries four LinkedIn scraper actors in sequence:
   `hireforce~linkedin-jobs-scraper` → `curious_coder~linkedin-job-scraper` → `anchor~linkedin-jobs-scraper` → `bebity~linkedin-jobs-scraper`.
   If all fail, search [apify.com/store](https://apify.com/store?search=linkedin+jobs) for a working actor and paste its ID.
5. For company career pages, add target companies (e.g. `Google, Meta, Stripe`) in Settings. The `apify~google-jobs-scraper` actor runs a cross-product search of your job titles × companies.

### Optional: add Adzuna (free, for salary data)
1. Register at [developer.adzuna.com](https://developer.adzuna.com) — 100 free calls/day.
2. Enter your App ID and App Key in **Settings → Adzuna**.

---

## Privacy

- **No backend.** There is no server. The app runs entirely in your browser.
- **No tracking.** No analytics, no cookies, no external data collection of any kind.
- **All data stays local.** Resumes, job results, and settings are stored only in your browser's `localStorage`. Clearing browser data removes everything permanently.
- **API keys are local.** Keys are stored in `localStorage` and sent directly from your browser to the respective APIs (Groq, Apify, Adzuna). They never pass through any intermediary server.
- **The PIN is hashed.** The PIN lock uses SHA-256 via the browser's Web Crypto API. The plaintext PIN is never stored anywhere.

---

## Limitations & Known Constraints

| Constraint | Detail |
|---|---|
| **Free API quotas** | Apify gives ~$5/month free; Adzuna gives 100 calls/day. Heavy use will exhaust these. |
| **24-hour job window** | Only jobs posted within the last 24 hours are fetched. Older jobs are filtered out intentionally. |
| **RemoteOK** | Returns remote-only jobs. Tag availability varies by role; PM roles may return fewer results. |
| **Arbeitnow** | International job board — returns mostly European jobs. The location filter removes non-matching country results. |
| **Remotive** | Remote-only jobs grouped by category. The app maps job title keywords to the closest category. |
| **LinkedIn actor stability** | Apify's LinkedIn scraper actors are maintained by the community and can go offline. The 4-actor fallback chain handles this, but if all fail you need to find a working actor at apify.com/store. |
| **Single HTML file** | The entire app is in one file — convenient to deploy and share, but not structured for large-scale code maintenance. |
| **No mobile-optimised layout** | The app is responsive but designed primarily for desktop use. |

---

## Local Development

No build step required.

```bash
git clone https://github.com/vidyullathaks/FreshFit-Job-Matcher.git
cd FreshFit-Job-Matcher
open index.html        # macOS
# or: xdg-open index.html   (Linux)
# or: start index.html       (Windows)
```

Because the app calls external APIs directly from the browser, it works from `file://` without a local server. If you hit CORS issues with a specific API during development, serve it with any static server:

```bash
npx serve .
# or: python3 -m http.server 8080
```
