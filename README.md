# FreshFit — AI Job Matcher

**Live app → [vidyullathaks.github.io/FreshFit-Job-Matcher](https://vidyullathaks.github.io/FreshFit-Job-Matcher/)**

FreshFit is a single-page web app that pulls job postings from the last 24 hours, scores each one against your uploaded resume using an AI matching algorithm, and then rewrites your resume — in your own format — to be ATS-optimised for the role you want to apply to. No sign-up, no backend, no data ever leaves your browser.

---

## Why it exists

Most job boards surface the same listings regardless of your background. You still have to open dozens of tabs, manually judge whether a role fits, and then rewrite your resume from scratch for each application. FreshFit collapses all three steps — **find, match, tailor** — into one tool that you run in under a minute.

---

## First Time? Start Here

This is the recommended flow from zero to your first tailored application:

**Step 1 — Upload your resume**
Click **Upload Your Resume** on the welcome screen. You can upload a PDF or paste plain text. Give it a label like `Product Manager Resume` so you can identify it later. Click **Save**. The app will automatically start searching for jobs.

**Step 2 — Configure your job preferences (important — do this before fetching)**
Go to **Settings** and fill in:
- **Job Titles / Keywords** — enter your actual target role(s), comma-separated. Example: `Product Manager, Senior PM, Head of Product`. These drive the job search *and* the scoring, so be specific.
- **Preferred Location** — type `Remote`, `United States`, `United Kingdom`, etc. Jobs from other countries will be filtered out automatically.

**Step 3 — Find fresh jobs**
Click **Find Jobs** (top-right button). The app simultaneously queries up to 6 job sources. It takes 5–20 seconds. When it finishes, the Jobs tab shows all results sorted by fit score — highest matches first.

**Step 4 — Read the fit scores**
Each card shows a percentage score and a label:
- **Strong Fit (70%+)** — your resume closely matches the role's title and requirements. Apply.
- **Moderate Fit (45–69%)** — partial match; worth reviewing. The tailoring step can close the gap.
- **Low Fit (<45%)** — role is likely outside your target area. These appear at the bottom.

Focus on Strong Fit cards. If you see few or no Strong Fit results, see [Tips for Best Results](#tips-for-best-results) below.

**Step 5 — Review and tailor**
Click any job card → then click **View & Tailor →** → then **Tailor Resume with AI** (requires a free Groq key). The AI rewrites your resume in your own format, using the job's exact wording, trimmed to one page. Review the output carefully — it preserves your real experience but reshapes emphasis and language for ATS matching.

**Step 6 — Save as PDF**
Click **Print / Save as PDF**. In the print dialog, set the destination to **Save as PDF** (Chrome) or **Microsoft Print to PDF** (Windows). This produces a formatted, print-ready PDF you can attach directly to applications.

---

## Features

### 1. Resume Management
- Upload one or more resumes as **PDF** or plain text.
- PDF text is extracted client-side using PDF.js — no file ever leaves your browser.
- Resumes are automatically parsed for section headers (Summary, Experience, Education, Skills, Certifications, etc.) and stored in `localStorage`.
- You can add, edit, and delete resumes, and label them by role (e.g. "Senior PM Resume", "Data Analyst CV").
- **Multiple resumes:** If you have more than one resume, the app automatically picks the best-matching one for each job. You can also manually switch which resume to tailor from inside the tailor modal.

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

Results are deduplicated by URL, filtered by your configured location, sorted by fit score, and cached in `localStorage` so the last fetch is always available without re-fetching.

### 3. AI Fit Scoring
Every job is scored 5–99% against the best matching resume in your library. The algorithm runs entirely in the browser — no API call is needed:

**Score breakdown (100 points total):**
- **60 pts — Core role match:** Extracts the core job title by stripping domain qualifiers (e.g. "Product Manager - Renewable Energy Platform (AI-Native)" → `product manager`). Scores how many of the core role words and phrases appear in your resume.
- **30 pts — Requirements section match:** Finds the "Requirements / Qualifications / What you bring" section of the job description specifically and scores it against your resume — avoiding dilution from boilerplate "About Us" and "What you'll do" text.
- **10 pts — Remote bonus:** Jobs tagged or located as remote get a +10 boost.

Score labels: **Strong Fit** (≥70%) · **Moderate Fit** (≥45%) · **Low Fit** (<45%)

Cached scores are automatically re-evaluated on every page load using the latest algorithm, so stale scores from previous sessions are always refreshed.

### 4. Location Filtering
After fetching, jobs are filtered against your configured **Preferred Location**:
- If set to `Remote`, all jobs pass through.
- If set to a country (e.g. `United States`), jobs where the location is clearly in a different country (e.g. "Berlin, Germany", "London, UK", "Bangalore, India") are excluded. Remote/worldwide/blank locations always pass through.
- Supports smart matching for US, UK, Canada, and Australia (major cities, state abbreviations, country name variants).

### 5. Dashboard Filters
Once jobs are loaded, narrow down by:
- **Keyword** — free-text search across title, company, description, and tags.
- **Source** — filter to a specific job board (RemoteOK, LinkedIn, etc.).
- **Fit Score** — show only Strong Fit (70%+) or Moderate Fit (50%+).

### 6. Job Detail View
Click any job card to open a full detail panel showing the full job description, fit score, location, salary (if available), posted time, source, which of your resumes matched best, and a direct **Apply** link to the original posting.

### 7. AI Resume Tailoring (Groq)
When you click **Tailor Resume with AI** on a job:
1. Your best-matching resume and the job description are sent to **Groq's API** (free, no credit card) running the `llama-3.3-70b-versatile` model.
2. The AI rewrites your resume with:
   - Exact terminology mirrored from the job description (ATS keyword matching).
   - Your original section order, bullet style, date formats, and capitalisations preserved.
   - Content trimmed to fit on **one US Letter page** (0.55" margins).
   - Your real employer names, job titles, and degrees never fabricated or altered.
3. The tailored output appears on-screen. **Review it before using** — the AI is very good but always check that facts are accurate.
4. If you have multiple resumes, you can switch which one to tailor from using the dropdown at the top of the tailor modal, then click **Re-tailor** to regenerate.

### 8. Print-to-PDF
After tailoring, click **Print / Save as PDF** to open a clean print view with proper resume typography. In the browser print dialog, choose **Save as PDF** as the destination. The output is formatted with a "Tailored for: [role] at [company]" footer at the bottom.

### 9. PIN Lock Screen
An optional PIN can be set for shared or public computers. The PIN is hashed with SHA-256 via the browser's Web Crypto API — the plaintext is never stored anywhere. See [Troubleshooting](#troubleshooting) if you forget it.

---

## Setup

### Minimum setup (no API keys needed)
1. Open [the live app](https://vidyullathaks.github.io/FreshFit-Job-Matcher/) in your browser.
2. Click **Upload Your Resume** and upload a PDF or paste plain text.
3. Go to **Settings** → set your **Job Titles** (e.g. `Product Manager, Senior PM`) and **Preferred Location** (e.g. `United States`).
4. Click **Find Jobs**. RemoteOK, Arbeitnow, and Remotive return results with no keys required.

### Recommended: add Groq (free, for AI tailoring)
1. Create a free account at [console.groq.com](https://console.groq.com) — no credit card needed.
2. Generate an API key (starts with `gsk_`).
3. Paste it into **Settings → Groq API Key → Save Settings**.
4. The **Tailor Resume with AI** button on every job card will now be active.

### Optional: add Apify (free tier, for LinkedIn + company career pages)
Apify gives you ~$5 free credit per month with no credit card. This unlocks LinkedIn jobs and company career page scraping.

1. Sign up at [apify.com](https://apify.com).
2. In the Apify console go to **Settings → Integrations → API token**.
3. Copy the token and paste it into **Settings → Apify API Token → Save Settings**.
4. For **LinkedIn jobs**: the app automatically tries four scraper actors in sequence — `hireforce~linkedin-jobs-scraper` → `curious_coder~linkedin-job-scraper` → `anchor~linkedin-jobs-scraper` → `bebity~linkedin-jobs-scraper`. If all four fail, go to [apify.com/store](https://apify.com/store?search=linkedin+jobs), find a working actor, and paste its ID into **Settings → LinkedIn Scraper Actor ID**.
5. For **company career pages**: type the companies you want to track into **Settings → Target Companies** (e.g. `Google, Meta, Stripe, OpenAI`). Use the quick-add badge buttons to add popular companies in one click.

### Optional: add Adzuna (free, adds salary data)
1. Register at [developer.adzuna.com](https://developer.adzuna.com) — 100 free API calls/day.
2. Copy your **App ID** and **App Key**.
3. Enter both in **Settings → Adzuna → Save Settings**.
4. Salary ranges will now appear on matching job cards.

---

## Tips for Best Results

**Match your job titles to your actual target role.** The fit score is anchored on the core role — if your Settings say `Software Engineer` but you're a Product Manager, every PM job will score low. Set your titles to exactly what you're looking for: `Product Manager, Senior PM, Director of Product`.

**Be specific with location.** `United States` will filter out European jobs. `Remote` will include everything. If you want US + Remote, try `United States` — remote-tagged jobs always pass through regardless of country filter.

**Upload a detailed resume.** The more keywords your resume contains (skills, tools, methodologies, domain terms), the more it will overlap with job descriptions. A resume with a rich Skills section and quantified bullet points scores significantly higher.

**Use multiple resumes for different roles.** If you're open to both PM and data analyst roles, add one resume optimised for each and label them clearly. The app will auto-select the best-matching one per job.

**Re-fetch daily.** The 24-hour window is intentional — fresh postings are less competitive. Come back each day and click **Find Jobs** to see new listings.

**Review tailored resumes before sending.** The AI preserves your real facts but reshapes language and emphasis. Always check that every line is accurate before submitting.

---

## Troubleshooting

### No jobs are showing up
- Check **Settings → Job Titles** — make sure they reflect what you're searching for.
- Check **Settings → Preferred Location** — if it's set to a country with few postings in the free APIs (e.g. a niche market), try `Remote` to cast a wider net.
- The 24-hour window is strict. If few jobs were posted in the last 24 hours for your role on the free APIs, results will be sparse. Adding Apify (LinkedIn) usually solves this.
- Open the browser console (F12 → Console) to see per-source error messages.

### All my scores are low (below 45%)
- The most common cause: **Job Titles in Settings don't match your resume**. If Settings says `Software Engineer` but your resume says `Product Manager`, every engineering job will match poorly against a PM resume.
- Make sure your resume has a rich **Skills** section with specific tools and domain keywords (e.g. `Agile, JIRA, product roadmap, stakeholder management, OKRs`).
- Short or sparse resumes score lower because there are fewer keywords to match against.

### LinkedIn / Apify not returning results
- Check **Settings → Apify API Token** is correct and saved.
- Go to [console.apify.com/billing](https://console.apify.com/billing) — if your monthly $5 credit is exhausted, LinkedIn scraping is paused until the next billing cycle.
- The scraper actor may be unavailable. Go to [apify.com/store](https://apify.com/store?search=linkedin+jobs), find an active actor, and paste its ID into **Settings → LinkedIn Scraper Actor ID**.

### I see jobs from other countries even though I set "United States"
- Jobs with `Remote` or blank location always pass through — these are valid for remote workers in any country.
- Jobs with only a city name and no country (e.g. "Springfield") are included when the country can't be determined. This is intentional to avoid excluding legitimate US listings.

### The "Tailor Resume with AI" button is greyed out
You need a Groq API key. Get one free at [console.groq.com](https://console.groq.com), then add it in **Settings → Groq API Key**.

### The PDF didn't parse well (garbled text or missing content)
- PDFs with heavy formatting, columns, or embedded graphics extract less cleanly. Try copying the text from your PDF and pasting it directly into the text area when uploading.
- Once saved, go to **Resumes → Edit** to review the extracted text and fix any issues.

### I forgot my PIN
Clearing your browser's `localStorage` for the site resets the PIN — but it also wipes all your resumes, jobs, and settings. In Chrome: open DevTools (F12) → Application tab → Local Storage → right-click the site → Clear. Then reload the page.

### Data disappeared after I cleared browser history
FreshFit stores everything in `localStorage`. "Clear browsing data" with the "Cookies and site data" option checked will wipe it. There is no cloud backup — all data lives only in your browser.

---

## Privacy

- **No backend.** There is no server. The app runs entirely in your browser.
- **No tracking.** No analytics, no cookies, no external data collection of any kind.
- **All data stays local.** Resumes, job results, and settings are stored only in your browser's `localStorage`. Clearing browser data removes everything permanently.
- **API keys are local.** Keys are stored in `localStorage` and sent directly from your browser to the respective APIs (Groq, Apify, Adzuna). They never pass through any intermediary server.
- **The PIN is hashed.** The PIN lock uses SHA-256 via the browser's Web Crypto API. The plaintext PIN is never stored anywhere.
- **Data is not portable.** Because everything lives in one browser's localStorage, your data does not sync across devices or browsers. There is no export feature.

---

## Browser Compatibility

| Browser | Support |
|---|---|
| Chrome 90+ | Full support (recommended) |
| Edge 90+ | Full support |
| Firefox 90+ | Full support |
| Safari 15+ | Full support |
| Mobile browsers | Functional but layout is desktop-optimised |
| Internet Explorer | Not supported |

---

## Limitations & Known Constraints

| Constraint | Detail |
|---|---|
| **Free API quotas** | Apify gives ~$5/month free; Adzuna gives 100 calls/day. Heavy use will exhaust these. |
| **24-hour job window** | Only jobs posted within the last 24 hours are fetched. Older jobs are filtered out intentionally. |
| **RemoteOK** | Returns remote-only jobs. Tag availability varies by role; niche roles may return few or no results. |
| **Arbeitnow** | International job board — returns mostly European jobs. The location filter removes non-matching results for US/UK/Canada/AU searches. |
| **Remotive** | Remote-only jobs grouped by category. The app maps job title keywords to the closest category automatically. |
| **LinkedIn actor stability** | Apify's LinkedIn scraper actors are community-maintained and can go offline without notice. The 4-actor fallback chain handles this, but if all fail you must find a working actor at apify.com/store. |
| **Single HTML file** | The entire app is one file — easy to deploy and share, but not structured for large-scale code maintenance. |
| **No mobile layout** | The app is responsive but designed primarily for desktop use. |
| **No cross-device sync** | Data lives in one browser's localStorage. There is no account system or cloud sync. |

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

The entire app is a single self-contained HTML file with no build step, no `node_modules`, no bundler, and no backend. Open the file in any browser and it works.

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
