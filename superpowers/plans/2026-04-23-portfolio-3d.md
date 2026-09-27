# 3D Portfolio Website — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build dark futuristic 3D portfolio website for Irzan Aldi Ananto with scroll-driven camera timeline, floating project cards, and particle effects.

**Architecture:** Single-page Next.js 15 app. 3D scenes via React Three Fiber (lazy loaded, SSR disabled). 5 scroll sections: Hero, About, Experience Timeline, Projects, Contact. Mobile gets 2D fallback. Deploy on Vercel free tier.

**Tech Stack:** Next.js 15, React 19, TypeScript, Three.js, @react-three/fiber, @react-three/drei, @react-three/postprocessing, Tailwind CSS 4, Framer Motion

**Spec:** `docs/superpowers/specs/2026-04-23-portfolio-3d-design.md`

---

## File Structure

```
portfolio-3d/
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
├── postcss.config.mjs
├── .env.local
├── public/
│   ├── textures/
│   │   ├── pos-placeholder.png
│   │   ├── inventory-placeholder.png
│   │   ├── cms-placeholder.png
│   │   ├── livestock-placeholder.png
│   │   └── sales-erp-placeholder.png
│   └── og-image.png
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Navbar.tsx
│   │   │   └── ScrollProgress.tsx
│   │   ├── sections/
│   │   │   ├── HeroSection.tsx
│   │   │   ├── AboutSection.tsx
│   │   │   ├── ExperienceSection.tsx
│   │   │   ├── ProjectsSection.tsx
│   │   │   └── ContactSection.tsx
│   │   ├── three/
│   │   │   ├── Scene.tsx
│   │   │   ├── ParticleField.tsx
│   │   │   ├── TechOrbit.tsx
│   │   │   ├── TimelinePath.tsx
│   │   │   ├── ProjectCard3D.tsx
│   │   │   └── FloatingShapes.tsx
│   │   └── ui/
│   │       ├── GlassCard.tsx
│   │       ├── Badge.tsx
│   │       ├── ProjectModal.tsx
│   │       └── Button.tsx
│   ├── data/
│   │   ├── projects.ts
│   │   ├── experience.ts
│   │   └── skills.ts
│   ├── hooks/
│   │   ├── useScrollProgress.ts
│   │   └── useWebGL.ts
│   └── lib/
│       └── utils.ts
└── __tests__/
    ├── data/
    │   ├── projects.test.ts
    │   ├── experience.test.ts
    │   └── skills.test.ts
    └── hooks/
        └── useScrollProgress.test.ts
```

---

### Task 1: Project Scaffold & Dependencies

**Files:** Create: `portfolio-3d/package.json`, `tsconfig.json`, `next.config.ts`, `tailwind.config.ts`, `.env.local`

- [ ] **Step 1: Create Next.js project**

```bash
cd ~/Downloads/PROJECT
npx create-next-app@latest portfolio-3d --typescript --tailwind --eslint --app --src-dir --no-import-alias --turbopack
```

Select defaults when prompted. This creates Next.js 15 + React 19 + Tailwind 4 + TypeScript.

- [ ] **Step 2: Install 3D dependencies**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm install three @react-three/fiber @react-three/drei @react-three/postprocessing framer-motion
npm install -D @types/three
```

- [ ] **Step 3: Install test dependencies**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm install -D jest @testing-library/react @testing-library/jest-dom ts-jest @types/jest jest-environment-jsdom
```

- [ ] **Step 4: Create jest.config.ts**

```typescript
// portfolio-3d/jest.config.ts
import type { Config } from 'jest';

const config: Config = {
  testEnvironment: 'jsdom',
  transform: {
    '^.+\\.tsx?$': ['ts-jest', { tsconfig: 'tsconfig.json' }],
  },
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  setupFilesAfterSetup: [],
};

export default config;
```

Add test script to `package.json`:

```json
"scripts": {
  "test": "jest",
  "test:watch": "jest --watch"
}
```

- [ ] **Step 5: Create .env.local**

```env
# portfolio-3d/.env.local
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

- [ ] **Step 6: Update next.config.ts for Three.js**

```typescript
// portfolio-3d/next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  transpilePackages: ['three'],
};

export default nextConfig;
```

- [ ] **Step 7: Verify dev server starts**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm run dev
```

Expected: App running on http://localhost:3000, default Next.js page visible.

- [ ] **Step 8: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git init
git add .
git commit -m "chore: scaffold Next.js 15 project with R3F and Tailwind"
```

---

### Task 2: Global Styles & Theme

**Files:** Modify: `src/app/globals.css`, `tailwind.config.ts`, `src/app/layout.tsx`

- [ ] **Step 1: Update globals.css with theme variables and base styles**

Replace contents of `src/app/globals.css`:

```css
/* portfolio-3d/src/app/globals.css */
@import "tailwindcss";

@theme {
  --color-bg-primary: #0a0a1a;
  --color-bg-card: rgba(255, 255, 255, 0.05);
  --color-accent-cyan: #00d4ff;
  --color-accent-purple: #7b2ff7;
  --color-text-primary: #f0f0f0;
  --color-text-secondary: #8892a0;

  --font-heading: 'Space Grotesk', sans-serif;
  --font-body: 'Inter', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}

html {
  scroll-behavior: smooth;
}

body {
  background-color: var(--color-bg-primary);
  color: var(--color-text-primary);
  font-family: var(--font-body);
  overflow-x: hidden;
}

/* Scrollbar styling */
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  background: var(--color-bg-primary);
}

::-webkit-scrollbar-thumb {
  background: var(--color-accent-cyan);
  border-radius: 3px;
}

/* Glass card utility */
.glass {
  background: var(--color-bg-card);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.08);
}

/* Glow utility */
.glow-cyan {
  box-shadow: 0 0 20px rgba(0, 212, 255, 0.3);
}

.glow-text-cyan {
  text-shadow: 0 0 10px rgba(0, 212, 255, 0.5);
}
```

- [ ] **Step 2: Update layout.tsx with fonts**

```typescript
// portfolio-3d/src/app/layout.tsx
import type { Metadata } from 'next';
import { Space_Grotesk, Inter, JetBrains_Mono } from 'next/font/google';
import './globals.css';

const spaceGrotesk = Space_Grotesk({
  subsets: ['latin'],
  variable: '--font-heading',
  display: 'swap',
});

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-body',
  display: 'swap',
});

const jetbrainsMono = JetBrains_Mono({
  subsets: ['latin'],
  variable: '--font-mono',
  display: 'swap',
});

export const metadata: Metadata = {
  title: 'Irzan Aldi Ananto — Fullstack Developer',
  description:
    'Fullstack Developer with 3+ years experience building web apps and APIs for retail and e-commerce platforms. NestJS, Next.js, Flutter, Laravel.',
  openGraph: {
    title: 'Irzan Aldi Ananto — Fullstack Developer',
    description:
      'Fullstack Developer with 3+ years experience building web apps and APIs for retail and e-commerce platforms.',
    url: process.env.NEXT_PUBLIC_SITE_URL,
    siteName: 'Irzan Aldi Ananto',
    type: 'website',
  },
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html
      lang="en"
      className={`${spaceGrotesk.variable} ${inter.variable} ${jetbrainsMono.variable}`}
    >
      <body className="antialiased">{children}</body>
    </html>
  );
}
```

- [ ] **Step 3: Verify fonts load**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm run dev
```

Open http://localhost:3000. Inspect `<html>` element — should have 3 font CSS variables. Background should be `#0a0a1a`.

- [ ] **Step 4: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/app/globals.css src/app/layout.tsx tailwind.config.ts
git commit -m "style: add dark futuristic theme, fonts, and glass utilities"
```

---

### Task 3: Data Layer

**Files:** Create: `src/data/projects.ts`, `src/data/experience.ts`, `src/data/skills.ts`, `__tests__/data/projects.test.ts`, `__tests__/data/experience.test.ts`, `__tests__/data/skills.test.ts`

- [ ] **Step 1: Write test for projects data**

```typescript
// portfolio-3d/__tests__/data/projects.test.ts
import { projects, type Project } from '@/data/projects';

describe('projects data', () => {
  it('should have 5 projects', () => {
    expect(projects).toHaveLength(5);
  });

  it('each project has required fields', () => {
    projects.forEach((project: Project) => {
      expect(project.id).toBeTruthy();
      expect(project.title).toBeTruthy();
      expect(project.company).toBeTruthy();
      expect(project.period).toBeTruthy();
      expect(project.description).toBeTruthy();
      expect(project.techStack.length).toBeGreaterThan(0);
      expect(project.images.length).toBeGreaterThan(0);
    });
  });

  it('has projects from both companies', () => {
    const companies = [...new Set(projects.map((p) => p.company))];
    expect(companies).toContain('The Body Shop Indonesia');
    expect(companies).toContain('HW Group');
  });
});
```

- [ ] **Step 2: Run test → verify fails**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm test -- __tests__/data/projects.test.ts
```

Expected: FAIL — module not found.

- [ ] **Step 3: Create projects.ts**

```typescript
// portfolio-3d/src/data/projects.ts
export interface Project {
  id: string;
  title: string;
  company: string;
  period: string;
  description: string;
  techStack: string[];
  images: string[];
  githubUrl?: string;
  liveUrl?: string;
}

export const projects: Project[] = [
  {
    id: 'pos-system',
    title: 'POS System',
    company: 'The Body Shop Indonesia',
    period: '2023 — Present',
    description:
      'Point of Sales application built with Flutter for in-store transactions. Handles product scanning, payment processing, and sales reporting across multiple retail outlets.',
    techStack: ['Flutter', 'Dart', 'REST API'],
    images: ['/textures/pos-placeholder.png'],
  },
  {
    id: 'realtime-inventory',
    title: 'Realtime Inventory',
    company: 'The Body Shop Indonesia',
    period: '2023 — Present',
    description:
      'Real-time inventory management system that synchronizes stock levels across physical stores and the e-commerce platform, ensuring accurate availability data.',
    techStack: ['NestJS', 'TypeScript', 'PostgreSQL', 'WebSocket'],
    images: ['/textures/inventory-placeholder.png'],
  },
  {
    id: 'ecommerce-cms',
    title: 'E-commerce CMS',
    company: 'The Body Shop Indonesia',
    period: '2023 — Present',
    description:
      'Content management system built with Next.js for managing the e-commerce platform, including product catalog, promotions, and customer data.',
    techStack: ['Next.js', 'React', 'TypeScript'],
    images: ['/textures/cms-placeholder.png'],
  },
  {
    id: 'livestock-erp',
    title: 'Livestock ERP',
    company: 'HW Group',
    period: 'Mar 2023',
    description:
      'Enterprise resource planning system for livestock management. Enables users to input and track stock data efficiently with automated reporting.',
    techStack: ['Laravel', 'PHP', 'Livewire', 'MySQL'],
    images: ['/textures/livestock-placeholder.png'],
  },
  {
    id: 'sales-purchasing-erp',
    title: 'Sales & Purchasing ERP',
    company: 'HW Group',
    period: 'Dec 2022 — Feb 2023',
    description:
      'ERP system supporting internal asset transactions with automatic report generation for accounting and finance departments.',
    techStack: ['Laravel', 'PHP', 'Livewire', 'MySQL'],
    images: ['/textures/sales-erp-placeholder.png'],
  },
];
```

- [ ] **Step 4: Run test → verify passes**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm test -- __tests__/data/projects.test.ts
```

Expected: PASS

- [ ] **Step 5: Write test for experience data**

```typescript
// portfolio-3d/__tests__/data/experience.test.ts
import { experiences, type Experience } from '@/data/experience';

describe('experience data', () => {
  it('should have 2 experiences', () => {
    expect(experiences).toHaveLength(2);
  });

  it('each experience has required fields', () => {
    experiences.forEach((exp: Experience) => {
      expect(exp.id).toBeTruthy();
      expect(exp.company).toBeTruthy();
      expect(exp.role).toBeTruthy();
      expect(exp.period).toBeTruthy();
      expect(exp.bullets.length).toBeGreaterThan(0);
      expect(exp.projectIds.length).toBeGreaterThan(0);
    });
  });

  it('experiences ordered chronologically (earliest first)', () => {
    expect(experiences[0].company).toBe('HW Group');
    expect(experiences[1].company).toBe('The Body Shop Indonesia');
  });
});
```

- [ ] **Step 6: Run test → verify fails**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm test -- __tests__/data/experience.test.ts
```

Expected: FAIL

- [ ] **Step 7: Create experience.ts**

```typescript
// portfolio-3d/src/data/experience.ts
export interface Experience {
  id: string;
  company: string;
  role: string;
  period: string;
  location: string;
  bullets: string[];
  projectIds: string[];
}

export const experiences: Experience[] = [
  {
    id: 'hw-group',
    company: 'HW Group',
    role: 'Backend Developer',
    period: 'Oct 2021 — Aug 2023',
    location: 'Indonesia',
    bullets: [
      'Designed and developed RESTful APIs using Laravel for internal operations and client-facing services.',
      'Built customer management CMS using Livewire, streamlining data operations.',
      'Contributed to POS CMS development using Vue.js with ongoing maintenance.',
    ],
    projectIds: ['livestock-erp', 'sales-purchasing-erp'],
  },
  {
    id: 'tbs-indonesia',
    company: 'The Body Shop Indonesia',
    role: 'Fullstack Developer',
    period: 'Aug 2023 — Present',
    location: 'Indonesia',
    bullets: [
      'Designed and implemented RESTful APIs using NestJS for backend services.',
      'Developed customer management CMS with Next.js for operations team.',
      'Built promotional features (vouchers, campaigns) for e-commerce website.',
      'Maintained and enhanced POS system using Flutter for in-store transactions.',
      'Developed real-time inventory system to sync stock across stores and e-commerce.',
    ],
    projectIds: ['pos-system', 'realtime-inventory', 'ecommerce-cms'],
  },
];
```

- [ ] **Step 8: Run test → verify passes**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm test -- __tests__/data/experience.test.ts
```

Expected: PASS

- [ ] **Step 9: Write test for skills data**

```typescript
// portfolio-3d/__tests__/data/skills.test.ts
import { skillCategories, type SkillCategory } from '@/data/skills';

describe('skills data', () => {
  it('should have 5 categories', () => {
    expect(skillCategories).toHaveLength(5);
  });

  it('each category has items', () => {
    skillCategories.forEach((cat: SkillCategory) => {
      expect(cat.category).toBeTruthy();
      expect(cat.items.length).toBeGreaterThan(0);
    });
  });
});
```

- [ ] **Step 10: Run test → verify fails**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm test -- __tests__/data/skills.test.ts
```

Expected: FAIL

- [ ] **Step 11: Create skills.ts**

```typescript
// portfolio-3d/src/data/skills.ts
export interface SkillCategory {
  category: string;
  items: string[];
}

export const skillCategories: SkillCategory[] = [
  { category: 'Languages', items: ['JavaScript', 'TypeScript', 'PHP', 'Dart'] },
  { category: 'Backend', items: ['NestJS', 'Laravel', 'Node.js', 'Express.js'] },
  { category: 'Frontend', items: ['Next.js', 'React', 'Vue.js', 'Livewire', 'Flutter'] },
  { category: 'Database', items: ['MySQL', 'PostgreSQL', 'MongoDB', 'Redis'] },
  { category: 'Tools', items: ['Git', 'Docker', 'REST API', 'Linux', 'Postman'] },
];
```

- [ ] **Step 12: Run all data tests → verify passes**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm test -- __tests__/data/
```

Expected: 3 suites, all PASS

- [ ] **Step 13: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/data/ __tests__/data/
git commit -m "feat: add portfolio data layer (projects, experience, skills)"
```

---

### Task 4: Utility Hooks

**Files:** Create: `src/hooks/useScrollProgress.ts`, `src/hooks/useWebGL.ts`, `src/lib/utils.ts`

- [ ] **Step 1: Create useScrollProgress hook**

```typescript
// portfolio-3d/src/hooks/useScrollProgress.ts
'use client';

import { useState, useEffect } from 'react';

export function useScrollProgress() {
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    function handleScroll() {
      const scrollTop = window.scrollY;
      const docHeight = document.documentElement.scrollHeight - window.innerHeight;
      const scrolled = docHeight > 0 ? scrollTop / docHeight : 0;
      setProgress(Math.min(Math.max(scrolled, 0), 1));
    }

    window.addEventListener('scroll', handleScroll, { passive: true });
    handleScroll();
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  return progress;
}
```

- [ ] **Step 2: Create useWebGL hook**

```typescript
// portfolio-3d/src/hooks/useWebGL.ts
'use client';

import { useState, useEffect } from 'react';

export function useWebGL() {
  const [supported, setSupported] = useState(true);

  useEffect(() => {
    try {
      const canvas = document.createElement('canvas');
      const gl =
        canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
      setSupported(!!gl);
    } catch {
      setSupported(false);
    }
  }, []);

  return supported;
}
```

- [ ] **Step 3: Create utils.ts**

```typescript
// portfolio-3d/src/lib/utils.ts
export function cn(...classes: (string | undefined | false)[]) {
  return classes.filter(Boolean).join(' ');
}

export function lerp(start: number, end: number, t: number) {
  return start + (end - start) * t;
}
```

- [ ] **Step 4: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/hooks/ src/lib/
git commit -m "feat: add scroll progress, WebGL detection hooks, and utils"
```

---

### Task 5: UI Components

**Files:** Create: `src/components/ui/GlassCard.tsx`, `src/components/ui/Badge.tsx`, `src/components/ui/Button.tsx`, `src/components/ui/ProjectModal.tsx`

- [ ] **Step 1: Create GlassCard**

```typescript
// portfolio-3d/src/components/ui/GlassCard.tsx
import { cn } from '@/lib/utils';

interface GlassCardProps {
  children: React.ReactNode;
  className?: string;
  hover?: boolean;
}

export function GlassCard({ children, className, hover = true }: GlassCardProps) {
  return (
    <div
      className={cn(
        'glass rounded-2xl p-6',
        hover && 'transition-all duration-300 hover:glow-cyan hover:border-[var(--color-accent-cyan)]/30',
        className
      )}
    >
      {children}
    </div>
  );
}
```

- [ ] **Step 2: Create Badge**

```typescript
// portfolio-3d/src/components/ui/Badge.tsx
import { cn } from '@/lib/utils';

interface BadgeProps {
  children: React.ReactNode;
  className?: string;
}

export function Badge({ children, className }: BadgeProps) {
  return (
    <span
      className={cn(
        'inline-flex items-center px-3 py-1 rounded-full text-xs',
        'font-[family-name:var(--font-mono)]',
        'bg-[var(--color-accent-cyan)]/10 text-[var(--color-accent-cyan)]',
        'border border-[var(--color-accent-cyan)]/20',
        className
      )}
    >
      {children}
    </span>
  );
}
```

- [ ] **Step 3: Create Button**

```typescript
// portfolio-3d/src/components/ui/Button.tsx
import { cn } from '@/lib/utils';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'ghost';
  children: React.ReactNode;
}

export function Button({ variant = 'primary', children, className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        'px-6 py-3 rounded-xl font-medium transition-all duration-300 cursor-pointer',
        variant === 'primary' &&
          'bg-[var(--color-accent-cyan)] text-[var(--color-bg-primary)] hover:shadow-[0_0_20px_rgba(0,212,255,0.4)]',
        variant === 'ghost' &&
          'border border-[var(--color-accent-cyan)]/30 text-[var(--color-accent-cyan)] hover:bg-[var(--color-accent-cyan)]/10',
        className
      )}
      {...props}
    >
      {children}
    </button>
  );
}
```

- [ ] **Step 4: Create ProjectModal**

```typescript
// portfolio-3d/src/components/ui/ProjectModal.tsx
'use client';

import { motion, AnimatePresence } from 'framer-motion';
import { Badge } from './Badge';
import { Button } from './Button';
import type { Project } from '@/data/projects';

interface ProjectModalProps {
  project: Project | null;
  onClose: () => void;
}

export function ProjectModal({ project, onClose }: ProjectModalProps) {
  return (
    <AnimatePresence>
      {project && (
        <motion.div
          className="fixed inset-0 z-50 flex items-center justify-center p-4"
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
        >
          {/* Backdrop */}
          <motion.div
            className="absolute inset-0 bg-black/70 backdrop-blur-sm"
            onClick={onClose}
          />

          {/* Modal */}
          <motion.div
            className="glass relative z-10 max-w-2xl w-full rounded-2xl p-8"
            initial={{ scale: 0.9, y: 20 }}
            animate={{ scale: 1, y: 0 }}
            exit={{ scale: 0.9, y: 20 }}
          >
            {/* Close button */}
            <button
              onClick={onClose}
              className="absolute top-4 right-4 text-[var(--color-text-secondary)] hover:text-[var(--color-text-primary)] cursor-pointer"
            >
              ✕
            </button>

            {/* Project screenshot placeholder */}
            <div className="w-full h-48 rounded-xl bg-[var(--color-accent-cyan)]/5 border border-[var(--color-accent-cyan)]/10 mb-6 flex items-center justify-center text-[var(--color-text-secondary)]">
              {project.images[0] ? (
                <img
                  src={project.images[0]}
                  alt={project.title}
                  className="w-full h-full object-cover rounded-xl"
                  onError={(e) => {
                    (e.target as HTMLImageElement).style.display = 'none';
                  }}
                />
              ) : (
                'Screenshot placeholder'
              )}
            </div>

            {/* Content */}
            <h3 className="text-2xl font-[family-name:var(--font-heading)] font-bold mb-1">
              {project.title}
            </h3>
            <p className="text-[var(--color-text-secondary)] text-sm mb-4">
              {project.company} · {project.period}
            </p>
            <p className="text-[var(--color-text-primary)] mb-6 leading-relaxed">
              {project.description}
            </p>

            {/* Tech stack */}
            <div className="flex flex-wrap gap-2 mb-6">
              {project.techStack.map((tech) => (
                <Badge key={tech}>{tech}</Badge>
              ))}
            </div>

            {/* Links */}
            <div className="flex gap-3">
              {project.githubUrl && (
                <a href={project.githubUrl} target="_blank" rel="noopener noreferrer">
                  <Button variant="ghost">GitHub</Button>
                </a>
              )}
              {project.liveUrl && (
                <a href={project.liveUrl} target="_blank" rel="noopener noreferrer">
                  <Button variant="primary">Live Demo</Button>
                </a>
              )}
            </div>
          </motion.div>
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

- [ ] **Step 5: Verify components render**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm run dev
```

Temporarily import and render `<GlassCard>`, `<Badge>`, `<Button>` in `page.tsx` to verify styling. Remove after verification.

- [ ] **Step 6: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/components/ui/
git commit -m "feat: add UI components (GlassCard, Badge, Button, ProjectModal)"
```

---

### Task 6: Layout Components (Navbar + ScrollProgress)

**Files:** Create: `src/components/layout/Navbar.tsx`, `src/components/layout/ScrollProgress.tsx`

- [ ] **Step 1: Create ScrollProgress**

```typescript
// portfolio-3d/src/components/layout/ScrollProgress.tsx
'use client';

import { useScrollProgress } from '@/hooks/useScrollProgress';

export function ScrollProgress() {
  const progress = useScrollProgress();

  return (
    <div className="fixed top-0 left-0 w-full h-[2px] z-50">
      <div
        className="h-full bg-[var(--color-accent-cyan)] shadow-[0_0_10px_var(--color-accent-cyan)]"
        style={{ width: `${progress * 100}%` }}
      />
    </div>
  );
}
```

- [ ] **Step 2: Create Navbar**

```typescript
// portfolio-3d/src/components/layout/Navbar.tsx
'use client';

import { useState, useEffect } from 'react';
import { cn } from '@/lib/utils';

const navItems = [
  { label: 'Home', href: '#hero' },
  { label: 'About', href: '#about' },
  { label: 'Experience', href: '#experience' },
  { label: 'Projects', href: '#projects' },
  { label: 'Contact', href: '#contact' },
];

export function Navbar() {
  const [scrolled, setScrolled] = useState(false);
  const [mobileOpen, setMobileOpen] = useState(false);

  useEffect(() => {
    function handleScroll() {
      setScrolled(window.scrollY > 50);
    }
    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  function handleClick(href: string) {
    setMobileOpen(false);
    const el = document.querySelector(href);
    el?.scrollIntoView({ behavior: 'smooth' });
  }

  return (
    <nav
      className={cn(
        'fixed top-0 left-0 w-full z-40 transition-all duration-300',
        scrolled ? 'glass py-3' : 'py-5'
      )}
    >
      <div className="max-w-6xl mx-auto px-6 flex items-center justify-between">
        <a
          href="#hero"
          className="font-[family-name:var(--font-heading)] font-bold text-lg text-[var(--color-accent-cyan)]"
          onClick={(e) => {
            e.preventDefault();
            handleClick('#hero');
          }}
        >
          IA
        </a>

        {/* Desktop nav */}
        <div className="hidden md:flex gap-8">
          {navItems.map((item) => (
            <a
              key={item.href}
              href={item.href}
              onClick={(e) => {
                e.preventDefault();
                handleClick(item.href);
              }}
              className="text-sm text-[var(--color-text-secondary)] hover:text-[var(--color-accent-cyan)] transition-colors"
            >
              {item.label}
            </a>
          ))}
        </div>

        {/* Mobile toggle */}
        <button
          className="md:hidden text-[var(--color-text-secondary)] cursor-pointer"
          onClick={() => setMobileOpen(!mobileOpen)}
        >
          {mobileOpen ? '✕' : '☰'}
        </button>
      </div>

      {/* Mobile menu */}
      {mobileOpen && (
        <div className="md:hidden glass mt-2 mx-6 rounded-xl p-4 flex flex-col gap-4">
          {navItems.map((item) => (
            <a
              key={item.href}
              href={item.href}
              onClick={(e) => {
                e.preventDefault();
                handleClick(item.href);
              }}
              className="text-sm text-[var(--color-text-secondary)] hover:text-[var(--color-accent-cyan)] transition-colors"
            >
              {item.label}
            </a>
          ))}
        </div>
      )}
    </nav>
  );
}
```

- [ ] **Step 3: Verify layout components render**

Temporarily add to `page.tsx`:

```typescript
import { Navbar } from '@/components/layout/Navbar';
import { ScrollProgress } from '@/components/layout/ScrollProgress';

export default function Home() {
  return (
    <>
      <ScrollProgress />
      <Navbar />
      <div style={{ height: '300vh', paddingTop: '100px' }}>
        <h1>Scroll to test</h1>
      </div>
    </>
  );
}
```

Expected: Navbar visible, scroll progress bar grows on scroll, navbar gets glass effect after scrolling.

- [ ] **Step 4: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/components/layout/
git commit -m "feat: add Navbar and ScrollProgress layout components"
```

---

### Task 7: 3D Scene Foundation

**Files:** Create: `src/components/three/Scene.tsx`, `src/components/three/ParticleField.tsx`, `src/components/three/FloatingShapes.tsx`

- [ ] **Step 1: Create Scene wrapper (Canvas + dynamic import)**

```typescript
// portfolio-3d/src/components/three/Scene.tsx
'use client';

import { Canvas } from '@react-three/fiber';
import { Preload } from '@react-three/drei';
import { Suspense } from 'react';

interface SceneProps {
  children: React.ReactNode;
  className?: string;
}

export function Scene({ children, className }: SceneProps) {
  return (
    <Canvas
      className={className}
      camera={{ position: [0, 0, 5], fov: 75 }}
      dpr={[1, 1.5]}
      gl={{ antialias: true, alpha: true }}
    >
      <Suspense fallback={null}>
        {children}
        <Preload all />
      </Suspense>
    </Canvas>
  );
}
```

- [ ] **Step 2: Create ParticleField**

```typescript
// portfolio-3d/src/components/three/ParticleField.tsx
'use client';

import { useRef, useMemo } from 'react';
import { useFrame } from '@react-three/fiber';
import * as THREE from 'three';

interface ParticleFieldProps {
  count?: number;
  radius?: number;
}

export function ParticleField({ count = 800, radius = 15 }: ParticleFieldProps) {
  const meshRef = useRef<THREE.Points>(null);

  const positions = useMemo(() => {
    const pos = new Float32Array(count * 3);
    for (let i = 0; i < count; i++) {
      const theta = Math.random() * Math.PI * 2;
      const phi = Math.acos(2 * Math.random() - 1);
      const r = radius * Math.cbrt(Math.random());
      pos[i * 3] = r * Math.sin(phi) * Math.cos(theta);
      pos[i * 3 + 1] = r * Math.sin(phi) * Math.sin(theta);
      pos[i * 3 + 2] = r * Math.cos(phi);
    }
    return pos;
  }, [count, radius]);

  useFrame((_, delta) => {
    if (meshRef.current) {
      meshRef.current.rotation.y += delta * 0.02;
      meshRef.current.rotation.x += delta * 0.01;
    }
  });

  return (
    <points ref={meshRef}>
      <bufferGeometry>
        <bufferAttribute
          attach="attributes-position"
          array={positions}
          count={count}
          itemSize={3}
        />
      </bufferGeometry>
      <pointsMaterial
        size={0.03}
        color="#00d4ff"
        transparent
        opacity={0.6}
        sizeAttenuation
        depthWrite={false}
      />
    </points>
  );
}
```

- [ ] **Step 3: Create FloatingShapes**

```typescript
// portfolio-3d/src/components/three/FloatingShapes.tsx
'use client';

import { useRef } from 'react';
import { useFrame } from '@react-three/fiber';
import { MeshDistortMaterial } from '@react-three/drei';
import * as THREE from 'three';

function FloatingShape({
  position,
  geometry,
  color,
  speed,
}: {
  position: [number, number, number];
  geometry: 'icosahedron' | 'torus' | 'octahedron';
  color: string;
  speed: number;
}) {
  const meshRef = useRef<THREE.Mesh>(null);

  useFrame((state) => {
    if (meshRef.current) {
      meshRef.current.rotation.x = state.clock.elapsedTime * speed * 0.3;
      meshRef.current.rotation.y = state.clock.elapsedTime * speed * 0.2;
      meshRef.current.position.y =
        position[1] + Math.sin(state.clock.elapsedTime * speed * 0.5) * 0.3;
    }
  });

  return (
    <mesh ref={meshRef} position={position}>
      {geometry === 'icosahedron' && <icosahedronGeometry args={[0.8, 1]} />}
      {geometry === 'torus' && <torusGeometry args={[0.6, 0.25, 16, 32]} />}
      {geometry === 'octahedron' && <octahedronGeometry args={[0.7, 0]} />}
      <MeshDistortMaterial
        color={color}
        emissive={color}
        emissiveIntensity={0.3}
        transparent
        opacity={0.15}
        wireframe
        distort={0.2}
        speed={1.5}
      />
    </mesh>
  );
}

export function FloatingShapes() {
  return (
    <group>
      <FloatingShape
        position={[-3, 1, -2]}
        geometry="icosahedron"
        color="#00d4ff"
        speed={0.8}
      />
      <FloatingShape
        position={[3.5, -0.5, -3]}
        geometry="torus"
        color="#7b2ff7"
        speed={0.6}
      />
      <FloatingShape
        position={[-1.5, -1.5, -1.5]}
        geometry="octahedron"
        color="#00d4ff"
        speed={1.0}
      />
      <FloatingShape
        position={[2, 2, -4]}
        geometry="icosahedron"
        color="#7b2ff7"
        speed={0.5}
      />
    </group>
  );
}
```

- [ ] **Step 4: Verify 3D renders**

Update `page.tsx` temporarily:

```typescript
'use client';

import dynamic from 'next/dynamic';
import { Navbar } from '@/components/layout/Navbar';
import { ScrollProgress } from '@/components/layout/ScrollProgress';

const Scene = dynamic(
  () => import('@/components/three/Scene').then((mod) => mod.Scene),
  { ssr: false }
);
const ParticleField = dynamic(
  () => import('@/components/three/ParticleField').then((mod) => mod.ParticleField),
  { ssr: false }
);
const FloatingShapes = dynamic(
  () => import('@/components/three/FloatingShapes').then((mod) => mod.FloatingShapes),
  { ssr: false }
);

export default function Home() {
  return (
    <>
      <ScrollProgress />
      <Navbar />
      <div className="h-screen relative">
        <div className="absolute inset-0">
          <Scene>
            <ambientLight intensity={0.2} />
            <ParticleField />
            <FloatingShapes />
          </Scene>
        </div>
        <div className="relative z-10 flex items-center justify-center h-full">
          <h1 className="text-4xl font-bold">3D Works!</h1>
        </div>
      </div>
    </>
  );
}
```

Expected: Particles floating, wireframe shapes rotating, dark background. No SSR errors.

- [ ] **Step 5: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/components/three/Scene.tsx src/components/three/ParticleField.tsx src/components/three/FloatingShapes.tsx
git commit -m "feat: add 3D scene foundation with particles and floating shapes"
```

---

### Task 8: Hero Section

**Files:** Create: `src/components/sections/HeroSection.tsx`

- [ ] **Step 1: Create HeroSection**

```typescript
// portfolio-3d/src/components/sections/HeroSection.tsx
'use client';

import dynamic from 'next/dynamic';
import { motion } from 'framer-motion';

const Scene = dynamic(
  () => import('@/components/three/Scene').then((mod) => mod.Scene),
  { ssr: false }
);
const ParticleField = dynamic(
  () => import('@/components/three/ParticleField').then((mod) => mod.ParticleField),
  { ssr: false }
);
const FloatingShapes = dynamic(
  () => import('@/components/three/FloatingShapes').then((mod) => mod.FloatingShapes),
  { ssr: false }
);

export function HeroSection() {
  return (
    <section id="hero" className="relative h-screen flex items-center justify-center overflow-hidden">
      {/* 3D Background */}
      <div className="absolute inset-0">
        <Scene>
          <ambientLight intensity={0.2} />
          <ParticleField count={800} />
          <FloatingShapes />
        </Scene>
      </div>

      {/* Content overlay */}
      <div className="relative z-10 text-center px-6">
        <motion.p
          className="text-[var(--color-accent-cyan)] font-[family-name:var(--font-mono)] text-sm mb-4 tracking-widest uppercase"
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.2 }}
        >
          Hello, I&apos;m
        </motion.p>

        <motion.h1
          className="text-5xl md:text-7xl font-[family-name:var(--font-heading)] font-bold mb-4"
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.4 }}
        >
          Irzan Aldi{' '}
          <span className="text-[var(--color-accent-cyan)] glow-text-cyan">Ananto</span>
        </motion.h1>

        <motion.p
          className="text-xl md:text-2xl text-[var(--color-text-secondary)] mb-8"
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.6 }}
        >
          Fullstack Developer
        </motion.p>

        <motion.p
          className="text-[var(--color-text-secondary)] max-w-lg mx-auto mb-12 leading-relaxed"
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ delay: 0.8 }}
        >
          3+ years building web apps & APIs for retail and e-commerce platforms.
        </motion.p>

        {/* Scroll indicator */}
        <motion.div
          className="absolute bottom-10 left-1/2 -translate-x-1/2"
          animate={{ y: [0, 10, 0] }}
          transition={{ repeat: Infinity, duration: 2 }}
        >
          <div className="w-6 h-10 rounded-full border-2 border-[var(--color-accent-cyan)]/40 flex justify-center pt-2">
            <div className="w-1 h-2 rounded-full bg-[var(--color-accent-cyan)]" />
          </div>
        </motion.div>
      </div>
    </section>
  );
}
```

- [ ] **Step 2: Verify hero renders**

Update `page.tsx`:

```typescript
import { Navbar } from '@/components/layout/Navbar';
import { ScrollProgress } from '@/components/layout/ScrollProgress';
import { HeroSection } from '@/components/sections/HeroSection';

export default function Home() {
  return (
    <>
      <ScrollProgress />
      <Navbar />
      <HeroSection />
    </>
  );
}
```

Expected: Full-screen hero with 3D background, name, title, scroll indicator animating.

- [ ] **Step 3: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/components/sections/HeroSection.tsx src/app/page.tsx
git commit -m "feat: add Hero section with 3D particle background"
```

---

### Task 9: About Section + TechOrbit

**Files:** Create: `src/components/three/TechOrbit.tsx`, `src/components/sections/AboutSection.tsx`

- [ ] **Step 1: Create TechOrbit 3D component**

```typescript
// portfolio-3d/src/components/three/TechOrbit.tsx
'use client';

import { useRef } from 'react';
import { useFrame } from '@react-three/fiber';
import { Text } from '@react-three/drei';
import * as THREE from 'three';

const techItems = [
  'JS', 'TS', 'PHP', 'Dart',
  'NestJS', 'Laravel', 'Next.js', 'React',
  'Vue', 'Flutter', 'PostgreSQL', 'Docker',
];

function OrbitItem({ label, index, total }: { label: string; index: number; total: number }) {
  const ref = useRef<THREE.Group>(null);
  const angle = (index / total) * Math.PI * 2;
  const radius = 2.5;
  const speed = 0.15;

  useFrame((state) => {
    if (ref.current) {
      const t = state.clock.elapsedTime * speed + angle;
      ref.current.position.x = Math.cos(t) * radius;
      ref.current.position.z = Math.sin(t) * radius;
      ref.current.position.y = Math.sin(t * 2) * 0.3;
      ref.current.lookAt(0, 0, 0);
    }
  });

  return (
    <group ref={ref}>
      <Text
        fontSize={0.2}
        color="#00d4ff"
        anchorX="center"
        anchorY="middle"
        font="/fonts/JetBrainsMono-Regular.ttf"
      >
        {label}
      </Text>
    </group>
  );
}

export function TechOrbit() {
  const groupRef = useRef<THREE.Group>(null);

  return (
    <group ref={groupRef}>
      {/* Center sphere */}
      <mesh>
        <sphereGeometry args={[0.5, 32, 32]} />
        <meshStandardMaterial
          color="#7b2ff7"
          emissive="#7b2ff7"
          emissiveIntensity={0.5}
          transparent
          opacity={0.3}
          wireframe
        />
      </mesh>

      {/* Orbiting tech labels */}
      {techItems.map((tech, i) => (
        <OrbitItem key={tech} label={tech} index={i} total={techItems.length} />
      ))}
    </group>
  );
}
```

- [ ] **Step 2: Download JetBrains Mono font for 3D Text**

```bash
mkdir -p ~/Downloads/PROJECT/portfolio-3d/public/fonts
curl -L -o /tmp/JetBrainsMono.zip "https://github.com/JetBrains/JetBrainsMono/releases/download/v2.304/JetBrainsMono-2.304.zip"
unzip -o /tmp/JetBrainsMono.zip -d /tmp/JetBrainsMono
cp /tmp/JetBrainsMono/fonts/ttf/JetBrainsMono-Regular.ttf ~/Downloads/PROJECT/portfolio-3d/public/fonts/
```

- [ ] **Step 3: Create AboutSection**

```typescript
// portfolio-3d/src/components/sections/AboutSection.tsx
'use client';

import dynamic from 'next/dynamic';
import { motion } from 'framer-motion';
import { Badge } from '@/components/ui/Badge';
import { skillCategories } from '@/data/skills';

const Scene = dynamic(
  () => import('@/components/three/Scene').then((mod) => mod.Scene),
  { ssr: false }
);
const TechOrbit = dynamic(
  () => import('@/components/three/TechOrbit').then((mod) => mod.TechOrbit),
  { ssr: false }
);

export function AboutSection() {
  return (
    <section id="about" className="relative min-h-screen py-24 px-6">
      <div className="max-w-6xl mx-auto grid md:grid-cols-2 gap-12 items-center">
        {/* Left: 3D Tech Orbit */}
        <motion.div
          className="h-[400px] w-full"
          initial={{ opacity: 0, x: -40 }}
          whileInView={{ opacity: 1, x: 0 }}
          viewport={{ once: true }}
          transition={{ duration: 0.8 }}
        >
          <Scene>
            <ambientLight intensity={0.3} />
            <pointLight position={[5, 5, 5]} intensity={0.5} />
            <TechOrbit />
          </Scene>
        </motion.div>

        {/* Right: About content */}
        <motion.div
          initial={{ opacity: 0, x: 40 }}
          whileInView={{ opacity: 1, x: 0 }}
          viewport={{ once: true }}
          transition={{ duration: 0.8 }}
        >
          <h2 className="text-3xl md:text-4xl font-[family-name:var(--font-heading)] font-bold mb-6">
            About <span className="text-[var(--color-accent-cyan)]">Me</span>
          </h2>

          <p className="text-[var(--color-text-secondary)] leading-relaxed mb-8">
            Fullstack Developer with 3+ years of experience building web applications
            and APIs for retail and e-commerce platforms. Hands-on expertise with
            Laravel, NestJS, Next.js, Vue.js, and Flutter. Currently working at
            The Body Shop Indonesia, building POS systems, real-time inventory, and
            e-commerce solutions.
          </p>

          {/* Skills grid */}
          <div className="space-y-4">
            {skillCategories.map((cat) => (
              <div key={cat.category}>
                <p className="text-sm text-[var(--color-text-secondary)] mb-2 font-[family-name:var(--font-mono)]">
                  {cat.category}
                </p>
                <div className="flex flex-wrap gap-2">
                  {cat.items.map((item) => (
                    <Badge key={item}>{item}</Badge>
                  ))}
                </div>
              </div>
            ))}
          </div>
        </motion.div>
      </div>
    </section>
  );
}
```

- [ ] **Step 4: Add AboutSection to page.tsx**

```typescript
// portfolio-3d/src/app/page.tsx
import { Navbar } from '@/components/layout/Navbar';
import { ScrollProgress } from '@/components/layout/ScrollProgress';
import { HeroSection } from '@/components/sections/HeroSection';
import { AboutSection } from '@/components/sections/AboutSection';

export default function Home() {
  return (
    <>
      <ScrollProgress />
      <Navbar />
      <HeroSection />
      <AboutSection />
    </>
  );
}
```

Expected: About section below hero. Tech orbit spinning on left. Skills grid on right.

- [ ] **Step 5: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/components/three/TechOrbit.tsx src/components/sections/AboutSection.tsx src/app/page.tsx public/fonts/
git commit -m "feat: add About section with 3D tech orbit"
```

---

### Task 10: Experience Timeline Section

**Files:** Create: `src/components/three/TimelinePath.tsx`, `src/components/sections/ExperienceSection.tsx`

- [ ] **Step 1: Create ExperienceSection (2D scroll-based with 3D accents)**

The timeline uses Framer Motion scroll animations with a glowing vertical line. 3D accents via small particle background.

```typescript
// portfolio-3d/src/components/sections/ExperienceSection.tsx
'use client';

import { motion } from 'framer-motion';
import { GlassCard } from '@/components/ui/GlassCard';
import { experiences } from '@/data/experience';

function TimelineItem({
  experience,
  index,
}: {
  experience: (typeof experiences)[number];
  index: number;
}) {
  const isLeft = index % 2 === 0;

  return (
    <motion.div
      className={`flex items-start gap-8 mb-16 ${isLeft ? 'md:flex-row' : 'md:flex-row-reverse'}`}
      initial={{ opacity: 0, x: isLeft ? -50 : 50 }}
      whileInView={{ opacity: 1, x: 0 }}
      viewport={{ once: true, margin: '-100px' }}
      transition={{ duration: 0.6 }}
    >
      {/* Content */}
      <div className="flex-1">
        <GlassCard>
          <div className="flex items-start justify-between mb-3 flex-wrap gap-2">
            <div>
              <h3 className="text-xl font-[family-name:var(--font-heading)] font-bold text-[var(--color-accent-cyan)]">
                {experience.company}
              </h3>
              <p className="text-[var(--color-text-secondary)] text-sm">
                {experience.role}
              </p>
            </div>
            <span className="text-xs font-[family-name:var(--font-mono)] text-[var(--color-accent-purple)] bg-[var(--color-accent-purple)]/10 px-3 py-1 rounded-full">
              {experience.period}
            </span>
          </div>

          <ul className="space-y-2">
            {experience.bullets.map((bullet, i) => (
              <li
                key={i}
                className="text-[var(--color-text-secondary)] text-sm leading-relaxed flex gap-2"
              >
                <span className="text-[var(--color-accent-cyan)] mt-1 shrink-0">▸</span>
                {bullet}
              </li>
            ))}
          </ul>
        </GlassCard>
      </div>
    </motion.div>
  );
}

export function ExperienceSection() {
  return (
    <section id="experience" className="relative min-h-screen py-24 px-6">
      <div className="max-w-4xl mx-auto">
        <motion.h2
          className="text-3xl md:text-4xl font-[family-name:var(--font-heading)] font-bold text-center mb-16"
          initial={{ opacity: 0, y: 20 }}
          whileInView={{ opacity: 1, y: 0 }}
          viewport={{ once: true }}
        >
          Work <span className="text-[var(--color-accent-cyan)]">Experience</span>
        </motion.h2>

        {/* Timeline line */}
        <div className="relative">
          {/* Glowing vertical line */}
          <div className="absolute left-1/2 top-0 bottom-0 w-[2px] bg-gradient-to-b from-[var(--color-accent-cyan)]/50 via-[var(--color-accent-purple)]/50 to-transparent hidden md:block" />

          {/* Timeline items */}
          {experiences.map((exp, i) => (
            <div key={exp.id} className="relative">
              {/* Dot on timeline */}
              <div className="absolute left-1/2 -translate-x-1/2 w-4 h-4 rounded-full bg-[var(--color-accent-cyan)] shadow-[0_0_12px_var(--color-accent-cyan)] hidden md:block z-10" />
              <TimelineItem experience={exp} index={i} />
            </div>
          ))}
        </div>
      </div>
    </section>
  );
}
```

- [ ] **Step 2: Create TimelinePath placeholder for future 3D enhancement**

```typescript
// portfolio-3d/src/components/three/TimelinePath.tsx
'use client';

import { useRef } from 'react';
import { useFrame } from '@react-three/fiber';
import * as THREE from 'three';

export function TimelinePath() {
  const lineRef = useRef<THREE.Line>(null);

  const points = [
    new THREE.Vector3(-3, 0, 0),
    new THREE.Vector3(-1, 1, -1),
    new THREE.Vector3(1, -0.5, 0),
    new THREE.Vector3(3, 0.5, -1),
  ];

  const curve = new THREE.CatmullRomCurve3(points);
  const curvePoints = curve.getPoints(50);

  useFrame((state) => {
    if (lineRef.current) {
      lineRef.current.rotation.y = Math.sin(state.clock.elapsedTime * 0.1) * 0.1;
    }
  });

  return (
    <line ref={lineRef as React.RefObject<THREE.Line>}>
      <bufferGeometry>
        <bufferAttribute
          attach="attributes-position"
          array={new Float32Array(curvePoints.flatMap((p) => [p.x, p.y, p.z]))}
          count={curvePoints.length}
          itemSize={3}
        />
      </bufferGeometry>
      <lineBasicMaterial color="#00d4ff" transparent opacity={0.4} />
    </line>
  );
}
```

- [ ] **Step 3: Add ExperienceSection to page.tsx**

```typescript
// portfolio-3d/src/app/page.tsx
import { Navbar } from '@/components/layout/Navbar';
import { ScrollProgress } from '@/components/layout/ScrollProgress';
import { HeroSection } from '@/components/sections/HeroSection';
import { AboutSection } from '@/components/sections/AboutSection';
import { ExperienceSection } from '@/components/sections/ExperienceSection';

export default function Home() {
  return (
    <>
      <ScrollProgress />
      <Navbar />
      <HeroSection />
      <AboutSection />
      <ExperienceSection />
    </>
  );
}
```

Expected: Timeline with 2 cards (HW Group, TBS Indonesia), glowing vertical line, cards animate in on scroll.

- [ ] **Step 4: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/components/three/TimelinePath.tsx src/components/sections/ExperienceSection.tsx src/app/page.tsx
git commit -m "feat: add Experience timeline section with scroll animations"
```

---

### Task 11: Projects Section + 3D Cards

**Files:** Create: `src/components/three/ProjectCard3D.tsx`, `src/components/sections/ProjectsSection.tsx`

- [ ] **Step 1: Create placeholder textures**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
mkdir -p public/textures

# Create simple placeholder PNGs (gray with project name)
# These are 800x450 placeholder images
for name in pos-placeholder inventory-placeholder cms-placeholder livestock-placeholder sales-erp-placeholder; do
  convert -size 800x450 xc:'#1a1a2e' -fill '#00d4ff' -gravity center -pointsize 36 -annotate 0 "${name}" "public/textures/${name}.png" 2>/dev/null || echo "ImageMagick not found — create placeholder images manually or skip (component handles missing images gracefully)"
done
```

If ImageMagick not available, skip — component has fallback for missing images.

- [ ] **Step 2: Create ProjectCard3D**

```typescript
// portfolio-3d/src/components/three/ProjectCard3D.tsx
'use client';

import { useRef, useState } from 'react';
import { useFrame } from '@react-three/fiber';
import { RoundedBox, Text } from '@react-three/drei';
import * as THREE from 'three';

interface ProjectCard3DProps {
  title: string;
  company: string;
  position: [number, number, number];
  onClick: () => void;
}

export function ProjectCard3D({
  title,
  company,
  position,
  onClick,
}: ProjectCard3DProps) {
  const meshRef = useRef<THREE.Mesh>(null);
  const [hovered, setHovered] = useState(false);

  useFrame((state) => {
    if (meshRef.current) {
      meshRef.current.position.y =
        position[1] + Math.sin(state.clock.elapsedTime * 0.8 + position[0]) * 0.1;

      const targetScale = hovered ? 1.08 : 1;
      meshRef.current.scale.lerp(
        new THREE.Vector3(targetScale, targetScale, targetScale),
        0.1
      );
    }
  });

  return (
    <group
      position={position}
      ref={meshRef as React.RefObject<THREE.Group> & React.RefObject<THREE.Mesh>}
      onClick={onClick}
      onPointerEnter={() => {
        setHovered(true);
        document.body.style.cursor = 'pointer';
      }}
      onPointerLeave={() => {
        setHovered(false);
        document.body.style.cursor = 'default';
      }}
    >
      <RoundedBox args={[2.2, 1.4, 0.05]} radius={0.08} smoothness={4}>
        <meshStandardMaterial
          color={hovered ? '#1a1a3e' : '#0f0f2a'}
          emissive={hovered ? '#00d4ff' : '#7b2ff7'}
          emissiveIntensity={hovered ? 0.15 : 0.05}
          transparent
          opacity={0.9}
        />
      </RoundedBox>

      {/* Title */}
      <Text
        position={[0, 0.2, 0.03]}
        fontSize={0.16}
        color="#f0f0f0"
        anchorX="center"
        anchorY="middle"
        maxWidth={1.8}
      >
        {title}
      </Text>

      {/* Company */}
      <Text
        position={[0, -0.15, 0.03]}
        fontSize={0.1}
        color="#8892a0"
        anchorX="center"
        anchorY="middle"
      >
        {company}
      </Text>

      {/* Glow border when hovered */}
      {hovered && (
        <RoundedBox args={[2.25, 1.45, 0.04]} radius={0.08} smoothness={4}>
          <meshBasicMaterial
            color="#00d4ff"
            transparent
            opacity={0.1}
            side={THREE.BackSide}
          />
        </RoundedBox>
      )}
    </group>
  );
}
```

- [ ] **Step 3: Create ProjectsSection**

```typescript
// portfolio-3d/src/components/sections/ProjectsSection.tsx
'use client';

import { useState } from 'react';
import dynamic from 'next/dynamic';
import { motion } from 'framer-motion';
import { GlassCard } from '@/components/ui/GlassCard';
import { Badge } from '@/components/ui/Badge';
import { ProjectModal } from '@/components/ui/ProjectModal';
import { projects, type Project } from '@/data/projects';

const Scene = dynamic(
  () => import('@/components/three/Scene').then((mod) => mod.Scene),
  { ssr: false }
);
const ProjectCard3D = dynamic(
  () => import('@/components/three/ProjectCard3D').then((mod) => mod.ProjectCard3D),
  { ssr: false }
);
const ParticleField = dynamic(
  () => import('@/components/three/ParticleField').then((mod) => mod.ParticleField),
  { ssr: false }
);

export function ProjectsSection() {
  const [selectedProject, setSelectedProject] = useState<Project | null>(null);

  const tbsProjects = projects.filter(
    (p) => p.company === 'The Body Shop Indonesia'
  );
  const hwProjects = projects.filter((p) => p.company === 'HW Group');

  return (
    <section id="projects" className="relative min-h-screen py-24 px-6">
      <div className="max-w-6xl mx-auto">
        <motion.h2
          className="text-3xl md:text-4xl font-[family-name:var(--font-heading)] font-bold text-center mb-4"
          initial={{ opacity: 0, y: 20 }}
          whileInView={{ opacity: 1, y: 0 }}
          viewport={{ once: true }}
        >
          Featured <span className="text-[var(--color-accent-cyan)]">Projects</span>
        </motion.h2>

        <motion.p
          className="text-center text-[var(--color-text-secondary)] mb-16"
          initial={{ opacity: 0 }}
          whileInView={{ opacity: 1 }}
          viewport={{ once: true }}
        >
          Click a card to see details
        </motion.p>

        {/* 3D Project Cards Scene */}
        <motion.div
          className="h-[500px] w-full mb-12 hidden md:block"
          initial={{ opacity: 0 }}
          whileInView={{ opacity: 1 }}
          viewport={{ once: true }}
          transition={{ duration: 1 }}
        >
          <Scene>
            <ambientLight intensity={0.4} />
            <pointLight position={[5, 5, 5]} intensity={0.5} />
            <ParticleField count={200} radius={10} />

            {/* TBS Projects - right side */}
            {tbsProjects.map((project, i) => (
              <ProjectCard3D
                key={project.id}
                title={project.title}
                company={project.company}
                position={[1.5, 1.2 - i * 1.6, 0]}
                onClick={() => setSelectedProject(project)}
              />
            ))}

            {/* HW Projects - left side */}
            {hwProjects.map((project, i) => (
              <ProjectCard3D
                key={project.id}
                title={project.title}
                company={project.company}
                position={[-1.5, 0.5 - i * 1.6, 0]}
                onClick={() => setSelectedProject(project)}
              />
            ))}
          </Scene>
        </motion.div>

        {/* Mobile fallback: 2D cards */}
        <div className="grid gap-6 md:hidden">
          {projects.map((project, i) => (
            <motion.div
              key={project.id}
              initial={{ opacity: 0, y: 30 }}
              whileInView={{ opacity: 1, y: 0 }}
              viewport={{ once: true }}
              transition={{ delay: i * 0.1 }}
            >
              <GlassCard
                className="cursor-pointer"
                hover
              >
                <button
                  className="w-full text-left cursor-pointer"
                  onClick={() => setSelectedProject(project)}
                >
                  <h3 className="text-lg font-[family-name:var(--font-heading)] font-bold mb-1">
                    {project.title}
                  </h3>
                  <p className="text-[var(--color-text-secondary)] text-xs mb-3">
                    {project.company} · {project.period}
                  </p>
                  <p className="text-[var(--color-text-secondary)] text-sm mb-4 line-clamp-2">
                    {project.description}
                  </p>
                  <div className="flex flex-wrap gap-2">
                    {project.techStack.map((tech) => (
                      <Badge key={tech}>{tech}</Badge>
                    ))}
                  </div>
                </button>
              </GlassCard>
            </motion.div>
          ))}
        </div>
      </div>

      {/* Project Modal */}
      <ProjectModal
        project={selectedProject}
        onClose={() => setSelectedProject(null)}
      />
    </section>
  );
}
```

- [ ] **Step 4: Add ProjectsSection to page.tsx**

```typescript
// portfolio-3d/src/app/page.tsx
import { Navbar } from '@/components/layout/Navbar';
import { ScrollProgress } from '@/components/layout/ScrollProgress';
import { HeroSection } from '@/components/sections/HeroSection';
import { AboutSection } from '@/components/sections/AboutSection';
import { ExperienceSection } from '@/components/sections/ExperienceSection';
import { ProjectsSection } from '@/components/sections/ProjectsSection';

export default function Home() {
  return (
    <>
      <ScrollProgress />
      <Navbar />
      <HeroSection />
      <AboutSection />
      <ExperienceSection />
      <ProjectsSection />
    </>
  );
}
```

Expected: 3D floating cards on desktop (TBS right, HW left), 2D card grid on mobile. Click card → modal opens with project detail.

- [ ] **Step 5: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/components/three/ProjectCard3D.tsx src/components/sections/ProjectsSection.tsx src/app/page.tsx
git commit -m "feat: add Projects section with 3D floating cards and modal"
```

---

### Task 12: Contact Section

**Files:** Create: `src/components/sections/ContactSection.tsx`

- [ ] **Step 1: Create ContactSection**

```typescript
// portfolio-3d/src/components/sections/ContactSection.tsx
'use client';

import { motion } from 'framer-motion';
import { Button } from '@/components/ui/Button';

const contactLinks = [
  {
    label: 'Email',
    value: 'irzanaldi@gmail.com',
    href: 'mailto:irzanaldi@gmail.com',
    icon: '✉',
  },
  {
    label: 'Phone',
    value: '+(62) 895-2662-7141',
    href: 'tel:+62895266271141',
    icon: '☎',
  },
  {
    label: 'GitHub',
    value: 'github.com/irzanaldi',
    href: 'https://github.com/irzanaldi',
    icon: '⌨',
  },
  {
    label: 'LinkedIn',
    value: 'Irzan Aldi Ananto',
    href: 'https://linkedin.com/in/irzan-aldi-ananto-688819214',
    icon: '🔗',
  },
];

export function ContactSection() {
  return (
    <section id="contact" className="relative min-h-[70vh] py-24 px-6 flex items-center">
      <div className="max-w-2xl mx-auto text-center w-full">
        <motion.h2
          className="text-3xl md:text-4xl font-[family-name:var(--font-heading)] font-bold mb-4"
          initial={{ opacity: 0, y: 20 }}
          whileInView={{ opacity: 1, y: 0 }}
          viewport={{ once: true }}
        >
          Get In <span className="text-[var(--color-accent-cyan)]">Touch</span>
        </motion.h2>

        <motion.p
          className="text-[var(--color-text-secondary)] mb-12 max-w-md mx-auto"
          initial={{ opacity: 0 }}
          whileInView={{ opacity: 1 }}
          viewport={{ once: true }}
          transition={{ delay: 0.2 }}
        >
          Open for opportunities. Feel free to reach out.
        </motion.p>

        <div className="grid sm:grid-cols-2 gap-4 mb-12">
          {contactLinks.map((link, i) => (
            <motion.a
              key={link.label}
              href={link.href}
              target={link.href.startsWith('http') ? '_blank' : undefined}
              rel={link.href.startsWith('http') ? 'noopener noreferrer' : undefined}
              className="glass rounded-xl p-5 text-left transition-all duration-300 hover:glow-cyan hover:border-[var(--color-accent-cyan)]/30 group"
              initial={{ opacity: 0, y: 20 }}
              whileInView={{ opacity: 1, y: 0 }}
              viewport={{ once: true }}
              transition={{ delay: 0.1 * i }}
            >
              <span className="text-2xl mb-2 block">{link.icon}</span>
              <p className="text-sm text-[var(--color-text-secondary)] mb-1">
                {link.label}
              </p>
              <p className="text-[var(--color-text-primary)] font-medium text-sm group-hover:text-[var(--color-accent-cyan)] transition-colors">
                {link.value}
              </p>
            </motion.a>
          ))}
        </div>

        {/* CTA */}
        <motion.div
          initial={{ opacity: 0 }}
          whileInView={{ opacity: 1 }}
          viewport={{ once: true }}
          transition={{ delay: 0.5 }}
        >
          <a href="mailto:irzanaldi@gmail.com">
            <Button variant="primary">Say Hello</Button>
          </a>
        </motion.div>

        {/* Footer */}
        <motion.p
          className="mt-16 text-xs text-[var(--color-text-secondary)]"
          initial={{ opacity: 0 }}
          whileInView={{ opacity: 1 }}
          viewport={{ once: true }}
        >
          Built with Next.js, Three.js & Framer Motion
        </motion.p>
      </div>
    </section>
  );
}
```

- [ ] **Step 2: Add ContactSection to page.tsx — final assembly**

```typescript
// portfolio-3d/src/app/page.tsx
import { Navbar } from '@/components/layout/Navbar';
import { ScrollProgress } from '@/components/layout/ScrollProgress';
import { HeroSection } from '@/components/sections/HeroSection';
import { AboutSection } from '@/components/sections/AboutSection';
import { ExperienceSection } from '@/components/sections/ExperienceSection';
import { ProjectsSection } from '@/components/sections/ProjectsSection';
import { ContactSection } from '@/components/sections/ContactSection';

export default function Home() {
  return (
    <>
      <ScrollProgress />
      <Navbar />
      <HeroSection />
      <AboutSection />
      <ExperienceSection />
      <ProjectsSection />
      <ContactSection />
    </>
  );
}
```

Expected: All 5 sections visible. Scroll through entire page. Navbar links work. Contact links clickable.

- [ ] **Step 3: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/components/sections/ContactSection.tsx src/app/page.tsx
git commit -m "feat: add Contact section and complete page assembly"
```

---

### Task 13: Mobile Responsive & WebGL Fallback

**Files:** Modify: `src/app/page.tsx`, `src/components/sections/HeroSection.tsx`, `src/components/sections/AboutSection.tsx`

- [ ] **Step 1: Create client wrapper page with WebGL detection**

```typescript
// portfolio-3d/src/app/page.tsx
import { Navbar } from '@/components/layout/Navbar';
import { ScrollProgress } from '@/components/layout/ScrollProgress';
import { HeroSection } from '@/components/sections/HeroSection';
import { AboutSection } from '@/components/sections/AboutSection';
import { ExperienceSection } from '@/components/sections/ExperienceSection';
import { ProjectsSection } from '@/components/sections/ProjectsSection';
import { ContactSection } from '@/components/sections/ContactSection';

export default function Home() {
  return (
    <>
      <ScrollProgress />
      <Navbar />
      <main>
        <HeroSection />
        <AboutSection />
        <ExperienceSection />
        <ProjectsSection />
        <ContactSection />
      </main>
    </>
  );
}
```

- [ ] **Step 2: Update HeroSection with mobile fallback**

In `HeroSection.tsx`, wrap 3D canvas in responsive visibility:

```typescript
// Add to HeroSection.tsx — wrap the 3D div:
{/* 3D Background - hidden on mobile */}
<div className="absolute inset-0 hidden md:block">
  <Scene>
    <ambientLight intensity={0.2} />
    <ParticleField count={800} />
    <FloatingShapes />
  </Scene>
</div>

{/* Mobile gradient fallback */}
<div className="absolute inset-0 md:hidden bg-gradient-to-br from-[#0a0a1a] via-[#0a1628] to-[#1a0a2e]" />
```

- [ ] **Step 3: Update AboutSection with mobile fallback**

In `AboutSection.tsx`, wrap TechOrbit:

```typescript
// Wrap the 3D div:
<motion.div
  className="h-[400px] w-full hidden md:block"
  // ... rest same
>
  <Scene>
    <ambientLight intensity={0.3} />
    <pointLight position={[5, 5, 5]} intensity={0.5} />
    <TechOrbit />
  </Scene>
</motion.div>

{/* Mobile: simple badge grid instead of 3D orbit */}
<div className="md:hidden mb-8">
  <div className="flex flex-wrap gap-2 justify-center">
    {['JS', 'TS', 'PHP', 'Dart', 'NestJS', 'Laravel', 'Next.js', 'React', 'Vue', 'Flutter', 'PostgreSQL', 'Docker'].map((tech) => (
      <Badge key={tech}>{tech}</Badge>
    ))}
  </div>
</div>
```

- [ ] **Step 4: Test responsive breakpoints**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm run dev
```

Test in browser:
- Desktop (>1024px): Full 3D experience
- Tablet (768-1024px): Reduced but functional
- Mobile (<768px): 2D fallback, no 3D canvas loaded

- [ ] **Step 5: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/components/sections/ src/app/page.tsx
git commit -m "feat: add mobile responsive fallbacks for 3D sections"
```

---

### Task 14: SEO & Performance Optimization

**Files:** Modify: `src/app/layout.tsx`, create: `public/og-image.png`

- [ ] **Step 1: Create OG image placeholder**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
# Create a simple OG placeholder (or use any 1200x630 dark image)
convert -size 1200x630 xc:'#0a0a1a' -fill '#00d4ff' -gravity center -pointsize 48 -annotate 0 "Irzan Aldi Ananto\nFullstack Developer" public/og-image.png 2>/dev/null || echo "Create public/og-image.png manually (1200x630, dark background with name)"
```

- [ ] **Step 2: Enhance metadata in layout.tsx**

Update metadata in `src/app/layout.tsx`:

```typescript
export const metadata: Metadata = {
  title: 'Irzan Aldi Ananto — Fullstack Developer',
  description:
    'Fullstack Developer with 3+ years experience building web apps and APIs for retail and e-commerce platforms. NestJS, Next.js, Flutter, Laravel.',
  keywords: [
    'Fullstack Developer',
    'NestJS',
    'Next.js',
    'Flutter',
    'Laravel',
    'React',
    'TypeScript',
    'Portfolio',
  ],
  authors: [{ name: 'Irzan Aldi Ananto' }],
  openGraph: {
    title: 'Irzan Aldi Ananto — Fullstack Developer',
    description:
      'Fullstack Developer with 3+ years experience building web apps and APIs for retail and e-commerce platforms.',
    url: process.env.NEXT_PUBLIC_SITE_URL,
    siteName: 'Irzan Aldi Ananto',
    images: [{ url: '/og-image.png', width: 1200, height: 630 }],
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Irzan Aldi Ananto — Fullstack Developer',
    description:
      'Fullstack Developer with 3+ years experience building web apps and APIs.',
    images: ['/og-image.png'],
  },
  robots: { index: true, follow: true },
};
```

- [ ] **Step 3: Verify build succeeds**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm run build
```

Expected: Build succeeds with no errors. Check output for bundle size.

- [ ] **Step 4: Commit**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add src/app/layout.tsx public/og-image.png
git commit -m "feat: add SEO metadata, OG image, and verify production build"
```

---

### Task 15: Vercel Deployment

**Files:** None new — deployment config.

- [ ] **Step 1: Verify build is clean**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
npm run build
npm run start
```

Open http://localhost:3000. Test all sections, navigation, modal, responsive.

- [ ] **Step 2: Push to GitHub**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
# Create GitHub repo first (if not exists)
gh repo create portfolio-3d --public --source=. --push
```

Or if repo already exists:

```bash
git remote add origin git@github.com:irzanaldi/portfolio-3d.git
git push -u origin main
```

- [ ] **Step 3: Deploy to Vercel**

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
cd ~/Downloads/PROJECT/portfolio-3d
vercel --prod
```

Follow prompts. Link to Vercel account. Auto-detects Next.js.

- [ ] **Step 4: Verify live deployment**

Open Vercel URL. Test:
- All sections render
- 3D works on desktop
- Mobile fallback works
- Navigation smooth
- Modal opens/closes
- SEO meta tags present (view source)

- [ ] **Step 5: Commit deployment config (if any)**

```bash
cd ~/Downloads/PROJECT/portfolio-3d
git add -A
git commit -m "chore: configure Vercel deployment"
git push
```

---

## Summary

| Task | Component | Key Output |
|------|-----------|-----------|
| 1 | Project Scaffold | Next.js 15 + R3F + Tailwind configured |
| 2 | Global Styles | Dark futuristic theme, fonts, glass utilities |
| 3 | Data Layer | Projects, experience, skills data + tests |
| 4 | Utility Hooks | useScrollProgress, useWebGL, utils |
| 5 | UI Components | GlassCard, Badge, Button, ProjectModal |
| 6 | Layout | Navbar + ScrollProgress |
| 7 | 3D Foundation | Scene, ParticleField, FloatingShapes |
| 8 | Hero Section | Full-screen hero with 3D background |
| 9 | About Section | TechOrbit + skills grid |
| 10 | Experience | Timeline with scroll animations |
| 11 | Projects | 3D floating cards + modal |
| 12 | Contact | Contact links + CTA |
| 13 | Responsive | Mobile 2D fallback |
| 14 | SEO | Metadata, OG image, build verification |
| 15 | Deploy | Vercel deployment |
