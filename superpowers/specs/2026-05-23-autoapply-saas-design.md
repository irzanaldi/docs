# AutoApply SaaS — Design Spec

## Overview

SaaS platform for automated job applications across any platform. Chrome extension records user's apply flow, replays it for subsequent jobs. Web dashboard manages resumes, cover letters, job tracking, and AI-powered content tailoring.

## Key Decisions

| # | Topic | Decision |
|---|-------|----------|
| 1 | Approach | Browser extension + cloud backend (hybrid) |
| 2 | Platform targeting | Configurable — user record & replay per platform |
| 3 | Revenue | Build first, monetize later |
| 4 | Resume | Template-based — multiple versions with tag matching |
| 5 | Tech stack | Next.js fullstack + Chrome Extension (Manifest V3) |
| 6 | Platform config | Record & replay + curated community templates |
| 7 | AI | LLM-powered cover letter generation + resume matching. Configurable provider (OpenAI/Anthropic). User provides own API key. |
| 8 | Scope | Full product — extension + backend + dashboard + AI |

## Tech Stack

- **Web App:** Next.js 14+ (App Router, API Routes, Server Components)
- **Database:** PostgreSQL + Prisma ORM
- **Auth:** NextAuth.js
- **Chrome Extension:** Manifest V3, React popup, TypeScript content scripts
- **AI:** Anthropic SDK / OpenAI SDK (configurable)
- **File Storage:** Local (dev) / S3-compatible (prod)
- **Styling:** Tailwind CSS + shadcn/ui

---

## 1. Architecture

```
┌─────────────────────────────────────────────┐
│              Next.js App                     │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐ │
│  │ Dashboard │  │ Resume   │  │ Job       │ │
│  │ Pages     │  │ Manager  │  │ Tracker   │ │
│  └──────────┘  └──────────┘  └───────────┘ │
│  ┌──────────────────────────────────────┐   │
│  │ API Routes (/api/*)                  │   │
│  │  auth, profiles, resumes, jobs,      │   │
│  │  templates, extension-sync, ai       │   │
│  └──────────────────────────────────────┘   │
│  ┌──────────────────────────────────────┐   │
│  │ Database: PostgreSQL (Prisma)        │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
            ▲
            │ REST API (X-API-Key auth)
            ▼
┌─────────────────────────────────────────────┐
│           Chrome Extension                   │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐ │
│  │ Recorder  │  │ Replayer │  │ Popup UI  │ │
│  │ (content  │  │ (content │  │ (React)   │ │
│  │  script)  │  │  script) │  │           │ │
│  └──────────┘  └──────────┘  └───────────┘ │
│  ┌──────────────────────────────────────┐   │
│  │ Background Service Worker            │   │
│  │  - sync with backend                 │   │
│  │  - queue management                  │   │
│  │  - template storage                  │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

---

## 2. Database Schema

```
users
+-- id (UUID, PK)
+-- name (varchar)
+-- email (varchar, unique)
+-- password (varchar, hashed)
+-- created_at (timestamp)
+-- updated_at (timestamp)

profiles
+-- id (UUID, PK)
+-- user_id (FK -> users, unique)
+-- full_name (varchar)
+-- phone (varchar, nullable)
+-- location (varchar, nullable)
+-- headline (varchar, nullable — "Senior Frontend Developer")
+-- summary (text, nullable — bio/about)
+-- linkedin_url (varchar, nullable)
+-- portfolio_url (varchar, nullable)
+-- github_url (varchar, nullable)
+-- created_at (timestamp)
+-- updated_at (timestamp)

resumes
+-- id (UUID, PK)
+-- user_id (FK -> users)
+-- name (varchar — "Frontend Resume", "Fullstack Resume")
+-- tags (JSON — ["react", "typescript", "frontend"])
+-- file_url (varchar — uploaded PDF path)
+-- is_default (boolean, default false)
+-- created_at (timestamp)
+-- updated_at (timestamp)

cover_letters
+-- id (UUID, PK)
+-- user_id (FK -> users)
+-- name (varchar — "Generic", "Startup-focused")
+-- content (text — template with {{company}}, {{position}} placeholders)
+-- tags (JSON)
+-- is_default (boolean, default false)
+-- created_at (timestamp)
+-- updated_at (timestamp)

platform_templates
+-- id (UUID, PK)
+-- user_id (FK -> users, nullable — null = official template)
+-- platform_name (varchar — "LinkedIn", "JobStreet")
+-- platform_url_pattern (varchar — "linkedin.com/jobs/*/apply")
+-- name (varchar — "LinkedIn Easy Apply")
+-- steps (JSON — recorded action steps)
+-- field_mapping (JSON — maps form fields to profile data)
+-- is_public (boolean, default false)
+-- is_verified (boolean, default false — curated by admin)
+-- version (integer, default 1)
+-- created_at (timestamp)
+-- updated_at (timestamp)

jobs
+-- id (UUID, PK)
+-- user_id (FK -> users)
+-- title (varchar)
+-- company (varchar)
+-- location (varchar, nullable)
+-- url (varchar — job posting URL)
+-- platform (varchar — "LinkedIn", "JobStreet", etc.)
+-- description (text, nullable)
+-- status (enum: saved, applying, applied, interview, rejected, offered)
+-- resume_id (FK -> resumes, nullable)
+-- cover_letter_id (FK -> cover_letters, nullable)
+-- applied_at (timestamp, nullable)
+-- notes (text, nullable)
+-- created_at (timestamp)
+-- updated_at (timestamp)

apply_logs
+-- id (UUID, PK)
+-- job_id (FK -> jobs)
+-- template_id (FK -> platform_templates)
+-- status (enum: queued, in_progress, success, failed, needs_human)
+-- error_message (text, nullable)
+-- screenshot_url (varchar, nullable — capture on failure)
+-- started_at (timestamp, nullable)
+-- completed_at (timestamp, nullable)
+-- created_at (timestamp)

api_keys
+-- id (UUID, PK)
+-- user_id (FK -> users)
+-- key (varchar, unique — random token)
+-- name (varchar — "My Extension")
+-- last_used_at (timestamp, nullable)
+-- is_active (boolean, default true)
+-- created_at (timestamp)

user_settings
+-- id (UUID, PK)
+-- user_id (FK -> users, unique)
+-- ai_provider (enum: openai, anthropic, nullable)
+-- ai_api_key (varchar, encrypted, nullable)
+-- ai_model (varchar, nullable — "gpt-4o-mini", "claude-sonnet-4-6")
+-- ai_enabled (boolean, default false)
+-- updated_at (timestamp)

ai_generations
+-- id (UUID, PK)
+-- user_id (FK -> users)
+-- job_id (FK -> jobs, nullable)
+-- type (enum: cover_letter, resume_suggestion, keyword_analysis)
+-- input_data (JSON — job desc, resume used, template used)
+-- output (text — generated content)
+-- model_used (varchar)
+-- tokens_used (integer)
+-- created_at (timestamp)
```

---

## 3. Chrome Extension — Record & Replay Engine

### Recorder

Content script captures DOM interactions when user clicks "Record":

- Click events → CSS selector + element text
- Input/type events → CSS selector + field label
- File upload events → CSS selector + field type
- Select/dropdown → CSS selector + option value
- Page navigations → URL pattern

Stored as JSON:

```json
{
  "platform": "LinkedIn",
  "url_pattern": "linkedin.com/jobs/*/apply",
  "steps": [
    { "action": "click", "selector": "button.jobs-apply-button", "label": "Apply" },
    { "action": "wait", "ms": 2000 },
    { "action": "fill", "selector": "input[name='phone']", "field": "profile.phone" },
    { "action": "fill", "selector": "input[name='email']", "field": "profile.email" },
    { "action": "upload", "selector": "input[type='file']", "field": "resume.file" },
    { "action": "fill", "selector": "textarea.cover-letter", "field": "cover_letter.content" },
    { "action": "click", "selector": "button[aria-label='Submit']", "label": "Submit" },
    { "action": "wait_for", "selector": ".success-message", "timeout": 5000 }
  ],
  "field_mapping": {
    "profile.phone": "Phone Number",
    "profile.email": "Email",
    "resume.file": "Resume Upload",
    "cover_letter.content": "Cover Letter"
  }
}
```

### Smart Selector Strategy

Recorder captures multiple selector variants per element. Replayer tries each:

```
Priority:
1. data-testid or aria-label (most stable)
2. id attribute
3. unique class combination
4. XPath with text content
```

### Replayer

Executes steps sequentially with safety:

```
For each step:
  1. Wait for selector (max 10s timeout)
  2. Execute action (click/fill/upload)
  3. Verify success (element state changed)
  4. If CAPTCHA → pause, notify user, resume after solved
  5. If element not found → pause, allow skip/retry/abort
  6. Screenshot on failure → save to apply_logs
```

### Extension File Structure

```
chrome-extension/
├── manifest.json              (Manifest V3)
├── background/
│   └── service-worker.ts      (sync, queue, auth)
├── content/
│   ├── recorder.ts            (DOM event capture + step generation)
│   ├── replayer.ts            (step executor + error handling)
│   └── detector.ts            (CAPTCHA detection, success detection)
├── popup/
│   ├── App.tsx                (React popup UI)
│   ├── pages/
│   │   ├── Dashboard.tsx      (status, quick stats)
│   │   ├── Record.tsx         (start/stop recording)
│   │   ├── Queue.tsx          (pending applies)
│   │   └── Settings.tsx       (API key, preferences)
│   └── components/
└── shared/
    ├── types.ts               (shared with web app)
    ├── api-client.ts          (backend communication)
    └── storage.ts             (chrome.storage wrapper)
```

---

## 4. Web Dashboard

### Pages

```
/                        → Landing page
/login                   → Auth
/register                → Auth
/dashboard               → Overview stats + recent activity
/profile                 → Edit profile (data injected into forms)
/resumes                 → CRUD resume versions + tags + upload PDF
/cover-letters           → CRUD cover letter templates with placeholders
/jobs                    → Job tracker (kanban or table view)
/jobs/:id                → Job detail + apply log history
/templates               → My templates + community templates
/templates/:id           → Template detail (view steps, edit)
/settings                → Account, API key management, AI config
```

### Dashboard Stats

- Total applied count
- This week count
- Success rate (applied / attempted)
- Needs action count (failed/needs_human)
- Status breakdown pie chart
- Recent activity feed

### Job Tracker

Two view modes:
- **Table** — sortable: company, title, platform, status, applied date, resume used
- **Kanban** — drag between: Saved → Applying → Applied → Interview → Offered/Rejected

### Extension ↔ Backend Auth

```
1. User logs in on dashboard
2. Dashboard generates API key (Settings page)
3. User copies key to extension popup Settings
4. Extension sends: X-API-Key header on all requests
5. Backend validates key → identifies user
```

---

## 5. AI Resume Tailoring

### Flow

```
User selects job to apply
    |
    +-- System fetches job description (scraped from URL or pasted)
    |
    +-- POST /api/ai/analyze-job → extracts skills, keywords, tone
    |
    +-- POST /api/ai/match-resume → picks best resume by tag match + LLM scoring
    |
    +-- POST /api/ai/generate-cover → tailored cover letter from template + job context
    |
    +-- User reviews → approve/edit → apply
```

### API Endpoints

```
POST /api/ai/analyze-job        — extract skills/keywords from job description
POST /api/ai/match-resume       — pick best resume for job (tag + LLM scoring)
POST /api/ai/generate-cover     — generate tailored cover letter
POST /api/ai/suggest-resume     — suggest resume improvements for job
```

### LLM Configuration

User sets in Settings page:
- Provider: OpenAI or Anthropic
- API key (encrypted at rest)
- Model selection
- Enable/disable toggle

If no API key → AI features hidden, manual mode only.

### Extension AI Integration

```
Extension popup → "Apply with AI" button
    |
    +-- Content script scrapes job description from page
    +-- Sends to: POST /api/ai/generate-cover
    +-- Gets tailored cover letter back
    +-- Auto-fills cover letter field in apply form
    +-- User reviews → confirm → submit
```

---

## 6. API Endpoints Summary

### Auth
```
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/logout
GET    /api/auth/me
```

### Profile
```
GET    /api/profile
PUT    /api/profile
```

### Resumes
```
GET    /api/resumes
POST   /api/resumes              (multipart — file + metadata)
GET    /api/resumes/:id
PUT    /api/resumes/:id
DELETE /api/resumes/:id
```

### Cover Letters
```
GET    /api/cover-letters
POST   /api/cover-letters
GET    /api/cover-letters/:id
PUT    /api/cover-letters/:id
DELETE /api/cover-letters/:id
```

### Jobs
```
GET    /api/jobs                  (?status, ?platform, ?search)
POST   /api/jobs
GET    /api/jobs/:id
PUT    /api/jobs/:id
DELETE /api/jobs/:id
GET    /api/jobs/stats            (dashboard counts)
```

### Apply Logs
```
GET    /api/jobs/:id/logs
POST   /api/jobs/:id/logs         (from extension)
```

### Templates
```
GET    /api/templates             (?platform, ?is_public, ?search)
POST   /api/templates
GET    /api/templates/:id
PUT    /api/templates/:id
DELETE /api/templates/:id
GET    /api/templates/community   (public + verified)
```

### API Keys
```
GET    /api/api-keys
POST   /api/api-keys
DELETE /api/api-keys/:id
```

### Settings
```
GET    /api/settings
PUT    /api/settings
```

### AI
```
POST   /api/ai/analyze-job
POST   /api/ai/match-resume
POST   /api/ai/generate-cover
POST   /api/ai/suggest-resume
```

### Extension Sync (X-API-Key auth)
```
GET    /api/ext/profile           (get profile data for form filling)
GET    /api/ext/resumes           (get resumes for upload)
GET    /api/ext/templates         (get user's templates)
POST   /api/ext/jobs              (create job from extension)
POST   /api/ext/apply-logs        (log apply attempt)
```

---

## 7. Project Structure

```
autoapply/
├── web/                          (Next.js fullstack app)
│   ├── prisma/
│   │   └── schema.prisma
│   ├── src/
│   │   ├── app/
│   │   │   ├── (marketing)/      (landing page)
│   │   │   ├── (auth)/           (login, register)
│   │   │   ├── (dashboard)/      (protected pages)
│   │   │   │   ├── dashboard/
│   │   │   │   ├── profile/
│   │   │   │   ├── resumes/
│   │   │   │   ├── cover-letters/
│   │   │   │   ├── jobs/
│   │   │   │   ├── templates/
│   │   │   │   └── settings/
│   │   │   └── api/
│   │   │       ├── auth/
│   │   │       ├── profile/
│   │   │       ├── resumes/
│   │   │       ├── cover-letters/
│   │   │       ├── jobs/
│   │   │       ├── templates/
│   │   │       ├── api-keys/
│   │   │       ├── settings/
│   │   │       ├── ai/
│   │   │       └── ext/          (extension endpoints)
│   │   ├── components/
│   │   │   ├── ui/               (shadcn)
│   │   │   └── shared/
│   │   ├── lib/
│   │   │   ├── prisma.ts
│   │   │   ├── auth.ts
│   │   │   ├── ai/
│   │   │   │   ├── provider.ts   (LLM provider abstraction)
│   │   │   │   ├── job-analyzer.ts
│   │   │   │   ├── resume-matcher.ts
│   │   │   │   └── cover-generator.ts
│   │   │   └── utils.ts
│   │   └── types/
│   ├── public/
│   └── package.json
├── extension/                    (Chrome Extension)
│   ├── manifest.json
│   ├── background/
│   ├── content/
│   ├── popup/
│   ├── shared/
│   ├── package.json
│   └── tsconfig.json
└── shared/                       (shared types between web + extension)
    └── types.ts
```
