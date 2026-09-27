# AutoApply SaaS — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a SaaS platform with Chrome extension that automates job applications across any platform using record & replay, with AI-powered resume tailoring.

**Architecture:** Next.js 14 fullstack app (API routes + dashboard) with PostgreSQL/Prisma for data, NextAuth for auth, and a Chrome Extension (Manifest V3) with React popup + content scripts for recording/replaying apply flows. AI features via configurable LLM provider (OpenAI/Anthropic).

**Tech Stack:** Next.js 14, Prisma, PostgreSQL, NextAuth.js, Tailwind + shadcn/ui, Chrome Extension Manifest V3, React, TypeScript, Anthropic/OpenAI SDK

**Spec:** `docs/superpowers/specs/2026-05-23-autoapply-saas-design.md`

**Project root:** `/Users/irzan-aldi/Downloads/PROJECT/Portofolio/autoapply`

---

## File Structure

### Web App (`web/`)

```
web/
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx                          (landing)
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── register/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx                    (sidebar + auth guard)
│   │   │   ├── dashboard/page.tsx
│   │   │   ├── profile/page.tsx
│   │   │   ├── resumes/page.tsx
│   │   │   ├── cover-letters/page.tsx
│   │   │   ├── jobs/page.tsx
│   │   │   ├── jobs/[id]/page.tsx
│   │   │   ├── templates/page.tsx
│   │   │   ├── templates/[id]/page.tsx
│   │   │   └── settings/page.tsx
│   │   └── api/
│   │       ├── auth/[...nextauth]/route.ts
│   │       ├── auth/register/route.ts
│   │       ├── profile/route.ts
│   │       ├── resumes/route.ts
│   │       ├── resumes/[id]/route.ts
│   │       ├── cover-letters/route.ts
│   │       ├── cover-letters/[id]/route.ts
│   │       ├── jobs/route.ts
│   │       ├── jobs/stats/route.ts
│   │       ├── jobs/[id]/route.ts
│   │       ├── jobs/[id]/logs/route.ts
│   │       ├── templates/route.ts
│   │       ├── templates/community/route.ts
│   │       ├── templates/[id]/route.ts
│   │       ├── api-keys/route.ts
│   │       ├── api-keys/[id]/route.ts
│   │       ├── settings/route.ts
│   │       ├── ai/analyze-job/route.ts
│   │       ├── ai/match-resume/route.ts
│   │       ├── ai/generate-cover/route.ts
│   │       ├── ai/suggest-resume/route.ts
│   │       └── ext/
│   │           ├── profile/route.ts
│   │           ├── resumes/route.ts
│   │           ├── templates/route.ts
│   │           ├── jobs/route.ts
│   │           └── apply-logs/route.ts
│   ├── components/
│   │   ├── ui/                               (shadcn)
│   │   ├── layout/
│   │   │   ├── sidebar.tsx
│   │   │   ├── header.tsx
│   │   │   └── auth-guard.tsx
│   │   └── shared/
│   │       ├── data-table.tsx
│   │       ├── file-upload.tsx
│   │       ├── tag-input.tsx
│   │       └── status-badge.tsx
│   ├── lib/
│   │   ├── prisma.ts
│   │   ├── auth.ts                           (NextAuth config)
│   │   ├── api-key.ts                        (validate X-API-Key)
│   │   ├── upload.ts                         (file storage helper)
│   │   ├── utils.ts
│   │   └── ai/
│   │       ├── provider.ts                   (LLM abstraction)
│   │       ├── job-analyzer.ts
│   │       ├── resume-matcher.ts
│   │       └── cover-generator.ts
│   └── types/
│       └── index.ts
├── public/
├── .env.example
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

### Chrome Extension (`extension/`)

```
extension/
├── manifest.json
├── package.json
├── tsconfig.json
├── webpack.config.js
├── src/
│   ├── background/
│   │   └── service-worker.ts
│   ├── content/
│   │   ├── recorder.ts
│   │   ├── replayer.ts
│   │   ├── detector.ts
│   │   └── selector.ts                      (smart selector generator)
│   ├── popup/
│   │   ├── index.html
│   │   ├── index.tsx
│   │   ├── App.tsx
│   │   ├── pages/
│   │   │   ├── Dashboard.tsx
│   │   │   ├── Record.tsx
│   │   │   ├── Queue.tsx
│   │   │   └── Settings.tsx
│   │   └── components/
│   │       ├── StatusBadge.tsx
│   │       └── JobCard.tsx
│   └── shared/
│       ├── types.ts
│       ├── api-client.ts
│       └── storage.ts
└── dist/                                    (build output)
```

---

## Task 1: Project Setup + Prisma Schema

**Files:**
- Create: `autoapply/web/` (Next.js project)
- Create: `autoapply/web/prisma/schema.prisma`
- Create: `autoapply/web/.env.example`

- [ ] **Step 1: Create Next.js project**

```bash
cd /Users/irzan-aldi/Downloads/PROJECT/Portofolio
mkdir autoapply && cd autoapply
npx create-next-app@latest web --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --no-turbopack
```

- [ ] **Step 2: Install dependencies**

```bash
cd web
npm install prisma @prisma/client next-auth @auth/prisma-adapter bcryptjs
npm install @anthropic-ai/sdk openai
npm install uuid
npm install -D @types/bcryptjs @types/uuid
npx prisma init
npx shadcn@latest init -d
npx shadcn@latest add button card dialog dropdown-menu input label select table tabs textarea badge separator avatar toast
```

- [ ] **Step 3: Write Prisma schema**

`web/prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(uuid())
  name          String
  email         String    @unique
  password      String
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  profile       Profile?
  resumes       Resume[]
  coverLetters  CoverLetter[]
  jobs          Job[]
  templates     PlatformTemplate[]
  apiKeys       ApiKey[]
  settings      UserSettings?
  aiGenerations AiGeneration[]

  @@map("users")
}

model Profile {
  id           String  @id @default(uuid())
  userId       String  @unique @map("user_id")
  fullName     String  @map("full_name")
  phone        String?
  location     String?
  headline     String?
  summary      String?
  linkedinUrl  String? @map("linkedin_url")
  portfolioUrl String? @map("portfolio_url")
  githubUrl    String? @map("github_url")
  createdAt    DateTime @default(now()) @map("created_at")
  updatedAt    DateTime @updatedAt @map("updated_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("profiles")
}

model Resume {
  id        String   @id @default(uuid())
  userId    String   @map("user_id")
  name      String
  tags      Json     @default("[]")
  fileUrl   String   @map("file_url")
  isDefault Boolean  @default(false) @map("is_default")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  user User  @relation(fields: [userId], references: [id], onDelete: Cascade)
  jobs Job[]

  @@map("resumes")
}

model CoverLetter {
  id        String   @id @default(uuid())
  userId    String   @map("user_id")
  name      String
  content   String
  tags      Json     @default("[]")
  isDefault Boolean  @default(false) @map("is_default")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  user User  @relation(fields: [userId], references: [id], onDelete: Cascade)
  jobs Job[]

  @@map("cover_letters")
}

model PlatformTemplate {
  id               String   @id @default(uuid())
  userId           String?  @map("user_id")
  platformName     String   @map("platform_name")
  platformUrlPattern String @map("platform_url_pattern")
  name             String
  steps            Json
  fieldMapping     Json     @map("field_mapping")
  isPublic         Boolean  @default(false) @map("is_public")
  isVerified       Boolean  @default(false) @map("is_verified")
  version          Int      @default(1)
  createdAt        DateTime @default(now()) @map("created_at")
  updatedAt        DateTime @updatedAt @map("updated_at")

  user      User?      @relation(fields: [userId], references: [id], onDelete: SetNull)
  applyLogs ApplyLog[]

  @@map("platform_templates")
}

enum JobStatus {
  saved
  applying
  applied
  interview
  rejected
  offered
}

model Job {
  id            String    @id @default(uuid())
  userId        String    @map("user_id")
  title         String
  company       String
  location      String?
  url           String
  platform      String
  description   String?
  status        JobStatus @default(saved)
  resumeId      String?   @map("resume_id")
  coverLetterId String?   @map("cover_letter_id")
  appliedAt     DateTime? @map("applied_at")
  notes         String?
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  user         User         @relation(fields: [userId], references: [id], onDelete: Cascade)
  resume       Resume?      @relation(fields: [resumeId], references: [id], onDelete: SetNull)
  coverLetter  CoverLetter? @relation(fields: [coverLetterId], references: [id], onDelete: SetNull)
  applyLogs    ApplyLog[]
  aiGenerations AiGeneration[]

  @@map("jobs")
}

enum ApplyLogStatus {
  queued
  in_progress
  success
  failed
  needs_human
}

model ApplyLog {
  id            String         @id @default(uuid())
  jobId         String         @map("job_id")
  templateId    String         @map("template_id")
  status        ApplyLogStatus @default(queued)
  errorMessage  String?        @map("error_message")
  screenshotUrl String?        @map("screenshot_url")
  startedAt     DateTime?      @map("started_at")
  completedAt   DateTime?      @map("completed_at")
  createdAt     DateTime       @default(now()) @map("created_at")

  job      Job              @relation(fields: [jobId], references: [id], onDelete: Cascade)
  template PlatformTemplate @relation(fields: [templateId], references: [id])

  @@map("apply_logs")
}

model ApiKey {
  id         String    @id @default(uuid())
  userId     String    @map("user_id")
  key        String    @unique
  name       String
  lastUsedAt DateTime? @map("last_used_at")
  isActive   Boolean   @default(true) @map("is_active")
  createdAt  DateTime  @default(now()) @map("created_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("api_keys")
}

enum AiProvider {
  openai
  anthropic
}

model UserSettings {
  id         String      @id @default(uuid())
  userId     String      @unique @map("user_id")
  aiProvider AiProvider? @map("ai_provider")
  aiApiKey   String?     @map("ai_api_key")
  aiModel    String?     @map("ai_model")
  aiEnabled  Boolean     @default(false) @map("ai_enabled")
  updatedAt  DateTime    @updatedAt @map("updated_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("user_settings")
}

enum AiGenerationType {
  cover_letter
  resume_suggestion
  keyword_analysis
}

model AiGeneration {
  id         String           @id @default(uuid())
  userId     String           @map("user_id")
  jobId      String?          @map("job_id")
  type       AiGenerationType
  inputData  Json             @map("input_data")
  output     String
  modelUsed  String           @map("model_used")
  tokensUsed Int              @map("tokens_used")
  createdAt  DateTime         @default(now()) @map("created_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  job  Job? @relation(fields: [jobId], references: [id], onDelete: SetNull)

  @@map("ai_generations")
}
```

- [ ] **Step 4: Create .env.example**

```
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/autoapply?schema=public"
NEXTAUTH_SECRET="change-me-in-production"
NEXTAUTH_URL="http://localhost:3000"
```

- [ ] **Step 5: Create Prisma client singleton**

`web/src/lib/prisma.ts`:
```typescript
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const prisma = globalForPrisma.prisma || new PrismaClient();

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

- [ ] **Step 6: Run migration**

```bash
cd web && npx prisma migrate dev --name init
```

- [ ] **Step 7: Init git and commit**

```bash
cd /Users/irzan-aldi/Downloads/PROJECT/Portofolio/autoapply
git init && git add web/ && git commit -m "feat: project setup — Next.js + Prisma + PostgreSQL schema"
```

---

## Task 2: Auth (NextAuth + Register/Login)

**Files:**
- Create: `web/src/lib/auth.ts`
- Create: `web/src/app/api/auth/[...nextauth]/route.ts`
- Create: `web/src/app/api/auth/register/route.ts`
- Create: `web/src/app/(auth)/login/page.tsx`
- Create: `web/src/app/(auth)/register/page.tsx`
- Create: `web/src/app/(auth)/layout.tsx`

NextAuth config with Credentials provider (email + password). Register endpoint hashes password with bcryptjs. Login/register pages with forms. Session includes user id.

Auth guard middleware: redirect unauthenticated users from dashboard routes to /login.

- [ ] **Steps:** Create auth config, API routes, login/register pages, middleware
- [ ] **Commit:** `feat: add auth — NextAuth credentials provider, register/login pages`

---

## Task 3: Dashboard Layout + Profile + Settings + API Keys

**Files:**
- Create: `web/src/app/(dashboard)/layout.tsx` (sidebar + header)
- Create: `web/src/components/layout/sidebar.tsx`
- Create: `web/src/app/(dashboard)/profile/page.tsx`
- Create: `web/src/app/(dashboard)/settings/page.tsx`
- Create: `web/src/app/api/profile/route.ts`
- Create: `web/src/app/api/settings/route.ts`
- Create: `web/src/app/api/api-keys/route.ts`
- Create: `web/src/app/api/api-keys/[id]/route.ts`
- Create: `web/src/lib/api-key.ts`

Dashboard layout with sidebar navigation (all pages listed in spec). Profile page: edit full_name, phone, location, headline, summary, URLs. Settings page: AI provider config + API key management (generate, list, delete, copy).

`api-key.ts` — middleware helper to validate X-API-Key header, resolve user.

- [ ] **Steps:** Create layout, sidebar, profile CRUD, settings CRUD, API key management
- [ ] **Commit:** `feat: add dashboard layout, profile, settings, API key management`

---

## Task 4: Resume CRUD

**Files:**
- Create: `web/src/app/api/resumes/route.ts` (GET list, POST create multipart)
- Create: `web/src/app/api/resumes/[id]/route.ts` (GET, PUT, DELETE)
- Create: `web/src/app/(dashboard)/resumes/page.tsx`
- Create: `web/src/lib/upload.ts`
- Create: `web/src/components/shared/file-upload.tsx`
- Create: `web/src/components/shared/tag-input.tsx`

Resume page: table of resumes with name, tags, default badge, actions. Upload PDF (multipart, store locally in `public/uploads/resumes/`). Tag input component for adding/removing tags. Set default resume.

- [ ] **Steps:** Create upload helper, API routes, resume page with file upload + tags
- [ ] **Commit:** `feat: add resume management — upload PDF, tags, default selection`

---

## Task 5: Cover Letter CRUD

**Files:**
- Create: `web/src/app/api/cover-letters/route.ts`
- Create: `web/src/app/api/cover-letters/[id]/route.ts`
- Create: `web/src/app/(dashboard)/cover-letters/page.tsx`

Cover letter page: table with name, tags, default, actions. Create/edit dialog with textarea supporting `{{company}}`, `{{position}}`, `{{job_title}}` placeholders. Preview rendered template with sample data.

- [ ] **Steps:** Create API routes, cover letter page with placeholder template editor
- [ ] **Commit:** `feat: add cover letter management — templates with placeholders`

---

## Task 6: Job Tracker

**Files:**
- Create: `web/src/app/api/jobs/route.ts` (GET with filters, POST)
- Create: `web/src/app/api/jobs/stats/route.ts` (GET dashboard stats)
- Create: `web/src/app/api/jobs/[id]/route.ts` (GET, PUT, DELETE)
- Create: `web/src/app/api/jobs/[id]/logs/route.ts` (GET logs)
- Create: `web/src/app/(dashboard)/jobs/page.tsx` (table + kanban toggle)
- Create: `web/src/app/(dashboard)/jobs/[id]/page.tsx` (detail + logs)
- Create: `web/src/components/shared/status-badge.tsx`

Jobs page with two view modes:
- **Table:** sortable columns — company, title, platform, status, applied date
- **Kanban:** drag-and-drop columns — Saved → Applying → Applied → Interview → Offered/Rejected

Job detail page: full info + apply log history table. Status badge component with colors per status.

Filter by: status, platform, search text.

- [ ] **Steps:** Create API routes, job tracker with table/kanban views, detail page
- [ ] **Commit:** `feat: add job tracker — table/kanban views, status tracking, apply logs`

---

## Task 7: Template Management

**Files:**
- Create: `web/src/app/api/templates/route.ts`
- Create: `web/src/app/api/templates/community/route.ts`
- Create: `web/src/app/api/templates/[id]/route.ts`
- Create: `web/src/app/(dashboard)/templates/page.tsx`
- Create: `web/src/app/(dashboard)/templates/[id]/page.tsx`

Templates page: tabs — My Templates / Community. Table with platform name, template name, version, public badge, verified badge. Template detail page: view recorded steps as readable list, edit field mapping, test replay (dry run).

Community page: browse public + verified templates, "clone" button copies to user's templates.

- [ ] **Steps:** Create API routes, template list with tabs, detail page with step viewer
- [ ] **Commit:** `feat: add template management — my templates, community browse, step viewer`

---

## Task 8: AI Module

**Files:**
- Create: `web/src/lib/ai/provider.ts` (LLM abstraction layer)
- Create: `web/src/lib/ai/job-analyzer.ts`
- Create: `web/src/lib/ai/resume-matcher.ts`
- Create: `web/src/lib/ai/cover-generator.ts`
- Create: `web/src/app/api/ai/analyze-job/route.ts`
- Create: `web/src/app/api/ai/match-resume/route.ts`
- Create: `web/src/app/api/ai/generate-cover/route.ts`
- Create: `web/src/app/api/ai/suggest-resume/route.ts`

**Provider abstraction** (`provider.ts`):
```typescript
interface LLMProvider {
  generateText(prompt: string, systemPrompt?: string): Promise<{ text: string; tokensUsed: number }>;
}

function createProvider(provider: 'openai' | 'anthropic', apiKey: string, model: string): LLMProvider
```

Supports OpenAI and Anthropic SDKs behind single interface. User's AI settings (provider, key, model) read from user_settings table.

**Job analyzer:** Extract skills, keywords, experience level, tone from job description.
**Resume matcher:** Score each resume against job analysis using tag overlap + LLM scoring.
**Cover generator:** Generate cover letter from user's template + job context + profile data. Replace `{{placeholders}}` + add job-specific content.

Each AI endpoint checks `ai_enabled` + `ai_api_key` — returns 403 if not configured.

- [ ] **Steps:** Create LLM provider abstraction, implement 4 AI services, create API routes
- [ ] **Commit:** `feat: add AI module — job analysis, resume matching, cover letter generation`

---

## Task 9: Dashboard Stats + Landing Page

**Files:**
- Create: `web/src/app/(dashboard)/dashboard/page.tsx`
- Create: `web/src/app/page.tsx` (landing)

Dashboard: stat cards (total applied, this week, success rate, needs action), status pie chart (Recharts), recent activity feed (last 10 apply logs).

Landing page: hero section explaining product, feature highlights (record & replay, AI tailoring, job tracking), CTA buttons (register/login). Simple, clean.

- [ ] **Steps:** Create dashboard with stats, create landing page
- [ ] **Commit:** `feat: add dashboard stats and landing page`

---

## Task 10: Extension API Endpoints

**Files:**
- Create: `web/src/app/api/ext/profile/route.ts`
- Create: `web/src/app/api/ext/resumes/route.ts`
- Create: `web/src/app/api/ext/templates/route.ts`
- Create: `web/src/app/api/ext/jobs/route.ts`
- Create: `web/src/app/api/ext/apply-logs/route.ts`

All extension endpoints authenticate via `X-API-Key` header (using `lib/api-key.ts`). Return data needed by extension: profile fields for form filling, resume file URLs for upload, user's templates, and accept job creation + apply log submissions.

- [ ] **Steps:** Create 5 extension API routes with API key auth
- [ ] **Commit:** `feat: add extension sync API endpoints with API key auth`

---

## Task 11: Chrome Extension Setup + Popup UI

**Files:**
- Create: `autoapply/extension/` (entire project)

- [ ] **Step 1: Init extension project**

```bash
cd /Users/irzan-aldi/Downloads/PROJECT/Portofolio/autoapply
mkdir extension && cd extension
npm init -y
npm install react react-dom
npm install -D typescript webpack webpack-cli ts-loader css-loader style-loader
npm install -D copy-webpack-plugin html-webpack-plugin
npm install -D @types/react @types/react-dom @types/chrome
```

- [ ] **Step 2: Create manifest.json (Manifest V3)**

```json
{
  "manifest_version": 3,
  "name": "AutoApply",
  "version": "1.0.0",
  "description": "Auto-apply to jobs with record & replay",
  "permissions": ["activeTab", "storage", "tabs", "scripting"],
  "host_permissions": ["<all_urls>"],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [{
    "matches": ["<all_urls>"],
    "js": ["content.js"],
    "run_at": "document_idle"
  }],
  "action": {
    "default_popup": "popup.html",
    "default_icon": { "16": "icon16.png", "48": "icon48.png", "128": "icon128.png" }
  }
}
```

- [ ] **Step 3: Create webpack config for building extension**

Bundle 3 entry points: background (service-worker), content (recorder+replayer+detector), popup (React app).

- [ ] **Step 4: Create popup React app**

4 pages in popup: Dashboard (quick stats, connection status), Record (start/stop recording, step preview), Queue (pending applies, status), Settings (API key input, connection test).

Navigation via simple tab bar at bottom of popup.

- [ ] **Step 5: Create shared utilities**

`shared/api-client.ts` — fetch wrapper that reads API key from chrome.storage, sends X-API-Key header.
`shared/storage.ts` — chrome.storage.local wrapper for API key, settings, cached data.
`shared/types.ts` — TypeScript types for steps, templates, jobs (matches web app types).

- [ ] **Step 6: Create background service worker**

Handles: message passing between popup ↔ content scripts, sync with backend, queue management for batch applies.

- [ ] **Commit:** `feat: create Chrome extension — Manifest V3, React popup, shared utilities`

---

## Task 12: Extension Recorder

**Files:**
- Create: `extension/src/content/recorder.ts`
- Create: `extension/src/content/selector.ts`

**Recorder** — content script that captures user interactions:
- Listen for click, input, change, submit events on document
- For each event, generate smart selectors (data-testid → aria-label → id → class → xpath)
- Build step object: `{ action, selectors[], label, field?, value? }`
- Handle file inputs specially (detect upload, mark as `resume.file` or `cover_letter.file`)
- Detect page navigations
- Send steps to popup via chrome.runtime.sendMessage
- Start/stop controlled by messages from popup

**Smart selector generator** (`selector.ts`):
- Generate multiple selector strategies per element
- Priority: data-testid, aria-label, id, nth-child path, text-based xpath
- Return array of selectors ordered by stability

- [ ] **Steps:** Create recorder content script + smart selector generator
- [ ] **Commit:** `feat: add extension recorder — DOM event capture with smart selectors`

---

## Task 13: Extension Replayer + Detector

**Files:**
- Create: `extension/src/content/replayer.ts`
- Create: `extension/src/content/detector.ts`

**Replayer** — executes recorded steps:
- For each step: try selectors in priority order until one matches
- Execute action: click (dispatchEvent), fill (set value + dispatch input/change), upload (create File + set on input), select (set value)
- Wait between steps (configurable delay)
- Timeout per step (10s default)
- On failure: screenshot via `chrome.tabs.captureVisibleTab`, pause, notify popup
- On CAPTCHA detected: pause, show notification, wait for user to solve, resume
- On success: send completion status to popup → popup syncs to backend

**Detector** (`detector.ts`):
- CAPTCHA detection: look for known CAPTCHA elements (reCAPTCHA iframe, hCaptcha div, common class names)
- Success detection: look for success messages, redirect patterns, "application submitted" text
- Configurable per template via optional `successSelector` and `captchaSelector` fields

- [ ] **Steps:** Create replayer engine + CAPTCHA/success detector
- [ ] **Commit:** `feat: add extension replayer — step execution with CAPTCHA handling`

---

## Task 14: Final Integration + Verification

- [ ] **Step 1: Build extension**

```bash
cd extension && npm run build
```

Load unpacked in Chrome: `chrome://extensions` → Developer mode → Load unpacked → select `extension/dist/`

- [ ] **Step 2: Test web app**

```bash
cd web && npm run build && npm run start
```

Verify: auth flow, CRUD operations, AI endpoints (with API key configured), dashboard stats.

- [ ] **Step 3: Test extension ↔ backend sync**

- Set API key in extension
- Verify connection test passes
- Record a test flow on any website
- Save template
- Replay template

- [ ] **Step 4: Final commit**

```bash
cd /Users/irzan-aldi/Downloads/PROJECT/Portofolio/autoapply
git add .
git commit -m "feat: complete AutoApply SaaS — web dashboard + Chrome extension + AI"
```
