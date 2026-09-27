# 3D Portfolio Website — Design Spec

**Author:** Irzan Aldi Ananto
**Date:** 2026-04-23
**Status:** Approved

---

## Overview

Dark futuristic 3D portfolio website for Irzan Aldi Ananto (Fullstack Developer). Scroll-driven single page with camera path timeline. Built with Next.js 15 + React Three Fiber. Deployed on Vercel (free tier).

**Target audience:** Recruiters + clients.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15, React 19, TypeScript |
| 3D Engine | Three.js via @react-three/fiber |
| 3D Helpers | @react-three/drei, @react-three/postprocessing |
| Styling | Tailwind CSS 4 |
| Animation | Framer Motion (2D), R3F (3D) |
| 3D Assets | Free .glb models (Sketchfab/Poly.pizza) + custom primitives |
| Deploy | Vercel (free tier) |

---

## Site Structure

Single page, 5 scroll sections:

### 1. Hero
- Full viewport
- 3D particle field (stars) background
- Floating geometric shapes (icosahedron, torus) with slow rotation
- Camera parallax on mouse move
- Content: Name "Irzan Aldi Ananto", title "Fullstack Developer", CTA scroll indicator

### 2. About
- Professional summary (from CV)
- 3D tech stack orbit — icons rotate around center sphere
- Contact info links (email, GitHub, LinkedIn)
- Location: Tangerang Selatan, Banten

### 3. Experience Timeline
- 3D camera-on-rail, scroll-linked
- Camera flies through space along curved path
- Glowing line connects timeline waypoints
- 2 stops:
  - **Stop 1 — HW Group** (Oct 2021 – Aug 2023): Backend Developer
    - RESTful APIs with Laravel
    - Customer management CMS (Livewire)
    - POS CMS (Vue.js)
  - **Stop 2 — The Body Shop Indonesia** (Aug 2023 – Present): Fullstack Developer
    - RESTful APIs with NestJS
    - Customer management CMS (Next.js)
    - Promo features (vouchers, campaigns)
    - POS system (Flutter)
    - Real-time inventory system

### 4. Projects
- Floating 3D cards at each timeline stop
- Card hover → tilt toward cursor (3D parallax)
- Card click → fly to center → modal overlay with detail

**HW Group projects:**
| Project | Description | Stack |
|---------|------------|-------|
| Livestock ERP | ERP for livestock management, stock tracking, auto reporting | Laravel, Livewire, MySQL |
| Sales & Purchasing ERP | Internal asset transactions, auto report generation | Laravel, Livewire, MySQL |

**TBS Indonesia projects:**
| Project | Description | Stack |
|---------|------------|-------|
| POS System | In-store transactions, product scanning, payment, sales reporting | Flutter, Dart |
| Realtime Inventory | Stock sync across stores + e-commerce platform | NestJS, PostgreSQL, WebSocket |
| E-commerce CMS | Product catalog, promotions, customer data management | Next.js, React, TypeScript |

**Card content:**
- Project name
- Description (1-2 sentences)
- Tech stack badges
- 1-3 placeholder mockup images (replaced later with real screenshots)
- Links (GitHub, live demo) if available

### 5. Contact
- Email: irzanaldi@gmail.com
- Phone: +(62) 895-2662-7141
- GitHub: github.com/irzanaldi
- LinkedIn: linkedin.com/in/irzan-aldi-ananto-688819214
- 3D floating social icons
- Subtle particle background

---

## Navigation

- Fixed navbar at top
- Section links: Hero, About, Experience, Projects, Contact
- Click → smooth scroll + camera animation to section
- Scroll progress bar (thin cyan line) at top

---

## Visual Design

### Color Palette — Dark Futuristic

| Role | Color | Value |
|------|-------|-------|
| Background | Deep dark blue-black | `#0a0a1a` |
| Primary accent | Electric cyan | `#00d4ff` |
| Secondary accent | Purple/violet | `#7b2ff7` |
| Text primary | White | `#f0f0f0` |
| Text secondary | Muted gray | `#8892a0` |
| Card background | Dark glass | `rgba(255,255,255,0.05)` |
| Glow/highlight | Cyan glow | `#00d4ff` with blur |

### Visual Effects
- **Glassmorphism** cards: backdrop-blur, subtle border, frosted glass
- **Glow effects**: cyan glow on hover, accent borders
- **Particles**: floating star/dot particles, subtle drift
- **Gradient mesh**: subtle gradient blobs in background
- **Scroll progress**: thin cyan line at navbar top

### Typography
- Headings: **Space Grotesk** (modern, geometric)
- Body: **Inter** (clean readability)
- Monospace accent: **JetBrains Mono** (tech stack badges)

### 3D Scene Aesthetic
- Dark environment, no harsh lighting
- Rim lighting: cyan/purple edge glow on objects
- Bloom post-processing on accent elements
- Fog/depth for sense of space

---

## 3D Scene Architecture

### Hero Scene
- `ParticleField`: 500-1000 star particles, random position in sphere, slow drift
- `FloatingShapes`: 3-5 geometric shapes (icosahedron, torus, octahedron), slow rotation, cyan/purple emissive material
- Camera: static position, slight parallax on mouse move (lerp)

### About Scene
- `TechOrbit`: center sphere + orbiting tech icon planes
- Icons: JS, TS, PHP, Dart, NestJS, Laravel, Next.js, React, Vue, Flutter, PostgreSQL, MongoDB, Docker, Git
- Auto-rotate, no user orbit control

### Timeline Scene
- `TimelinePath`: CatmullRom curve through 3D space
- Camera position + lookAt interpolated along curve based on scroll progress
- Waypoint markers (glowing spheres) at each company stop
- Glowing line (TubeGeometry) connecting waypoints
- Text labels (drei Text) at waypoints

### Project Cards
- `ProjectCard3D`: Plane geometry with rounded corners (drei RoundedBox)
- Front face: project screenshot texture (placeholder initially)
- Hover: card tilts toward cursor via pointer events
- Click: animate card to center → show HTML overlay modal (drei Html)

### Contact Scene
- Floating 3D social icons (GitHub, LinkedIn, Email, Phone)
- Reuse particle background from hero (lighter density)

---

## File Structure

```
portfolio-3d/
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
├── public/
│   ├── models/              # .glb 3D assets
│   ├── textures/            # project screenshots/placeholders
│   ├── fonts/
│   └── og-image.png         # Open Graph image
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
│   │   └── useScrollProgress.ts
│   └── lib/
│       └── utils.ts
└── .env.local
```

---

## Performance & SEO

- 3D Canvas: lazy loaded via `next/dynamic` with `{ ssr: false }`
- Fallback: 2D static view for no-WebGL devices (detect via `typeof window !== 'undefined' && !!window.WebGLRenderingContext`)
- `<Suspense>` wrapper + loading skeleton during 3D scene load
- Next.js Metadata API: title, description, OG image, Twitter card
- Lighthouse targets: Performance 80+, SEO 95+, Accessibility 90+
- Asset optimization:
  - `.glb` models compressed (draco)
  - Textures max 512x512 for decorative, 1024x1024 for project screenshots
  - Font subsetting (latin only)

---

## Responsive Design

| Breakpoint | Behavior |
|-----------|----------|
| Desktop (>1024px) | Full 3D experience, all effects |
| Tablet (768-1024px) | Reduced particles (50%), simplified 3D |
| Mobile (<768px) | 2D fallback layout, no 3D canvas. Framer Motion for animations instead |

Mobile gets clean 2D version — same content, same dark theme, Framer Motion animations. No degraded 3D.

---

## Data Schema

```typescript
// data/projects.ts
interface Project {
  id: string;
  title: string;
  company: string;
  period: string;
  description: string;
  techStack: string[];
  images: string[];       // placeholder paths initially
  githubUrl?: string;
  liveUrl?: string;
}

// data/experience.ts
interface Experience {
  id: string;
  company: string;
  role: string;
  period: string;
  location: string;
  bullets: string[];
  projects: string[];     // project IDs
}

// data/skills.ts
interface SkillCategory {
  category: string;       // "Languages", "Backend", etc.
  items: string[];
}
```

---

## Out of Scope (v1)

- Blog/articles section
- Dark/light theme toggle (dark only)
- CMS integration (data hardcoded in TS files)
- Analytics (add later)
- i18n (English only for v1)
- Contact form backend (direct links only)
