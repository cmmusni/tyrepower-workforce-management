# Tyrepower Workforce Management Platform

A modern workforce management solution built for Tyrepower locations — enabling GPS-verified clock-in/out, QR code attendance, automated timesheets, and real-time dashboards for multi-site operations.

---

## Key Features

- **GPS Geofence Clock-In/Out** — Employees clock in only when within site boundaries
- **QR Code Attendance** — Scan site-specific QR codes as an alternative clock-in method
- **Automated Timesheets** — Weekly timesheets auto-generated from time entries
- **Approval Workflows** — Managers approve/reject timesheets with one click
- **Real-Time Dashboards** — Live attendance, overtime alerts, and site overviews
- **Multi-Location Support** — Manage multiple Tyrepower sites from one platform
- **PWA** — Installable on phones, works offline for core features
- **Role-Based Access** — Employee, Site Manager, and Super Admin roles

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React.js + TypeScript + TailwindCSS |
| Backend | Supabase (PostgreSQL + PostGIS + Edge Functions) |
| Auth | Supabase Auth (JWT + RLS) |
| Maps | Mapbox GL JS |
| Hosting | Vercel (frontend) + Supabase Cloud (backend) |
| Notifications | Firebase Cloud Messaging + Postmark |
| CI/CD | GitHub Actions |

---

## Quick Start

```bash
# Clone
git clone https://github.com/<your-org>/tyrepower-workforce-management.git
cd tyrepower-workforce-management

# Install
npm install

# Environment setup
cp .env.example .env.local
# Edit .env.local with your Supabase & Mapbox keys

# Start Supabase locally (requires Docker)
npx supabase start

# Run migrations
npx supabase db reset

# Start dev server
npm run dev
```

Open `http://localhost:5173`

---

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/ARCHITECTURE.md) | System design, data flows, security model |
| [Database Schema](docs/DATABASE_SCHEMA.md) | Tables, relationships, RLS policies, key queries |
| [API Design](docs/API_DESIGN.md) | REST endpoints, Edge Functions, realtime subscriptions |
| [Features](docs/FEATURES.md) | Complete feature specs with acceptance criteria |
| [Development Plan](docs/DEVELOPMENT_PLAN.md) | 23-week sprint plan with 5 phases and milestones |
| [Roadmap & Timeline](docs/ROADMAP.md) | Mobile app development plan (React Native, future) |
| [Setup Guide](docs/SETUP_GUIDE.md) | Environment setup, tools, and troubleshooting |
| [Project Structure](docs/PROJECT_STRUCTURE.md) | Monorepo layout and architecture patterns |
| [Tech Stack Detail](TECH_STACK_DETAILED.md) | Technology choices and rationale |
| [Cost Analysis](COST_ANALYSIS.md) | Scaling costs per tier |
| [Cost Analysis (Simple)](COST_ANALYSIS_SIMPLE.md) | Simplified cost overview |

---

## Development Phases

| Phase | Duration | Focus | Status |
|-------|----------|-------|--------|
| **Phase 0: Design** | Week 1 | Branding (from Nina/Lae) + MVP Wireframes | ⏳ Blocked (awaiting branding) |
| **Phase 1: MVP** | Weeks 2-7 | Auth, Sites, Clock-In/Out + Breaks, QR, Timesheets, Dashboards | Not Started |
| **Phase 2: Production** | Weeks 8-14 | Leave, Holidays, Invitations, Payroll Export, Push, PWA, Audit | Not Started |
| **Testing & QA** | Weeks 15-16 | Full testing, cross-device, stakeholder UAT, sign-off | Not Started |
| **Phase 3: Enhancement** | Weeks 17-21 | Offline, Feature Flags, Data Migration, Rollout | Not Started |
| **Phase 4: Rostering & AI** | Weeks 22-27+ | Roster Management, AI Insights, Auto-Scheduling | Not Started |

---

## Project Structure (Summary)

```
packages/core/    # Shared business logic (platform-agnostic)
├── api/          # Supabase API client layer
├── hooks/        # Custom hooks (useAuth, useClockIn, etc.)
├── types/        # TypeScript types
├── utils/        # Pure utility functions
└── constants/    # App configuration

apps/web/         # Web application (React + Vite + TailwindCSS)
├── components/   # Web UI components
├── features/     # Feature modules (auth, clock, timesheets, etc.)
├── platform/     # Browser-specific APIs (GPS, camera, push)
└── styles/       # Global styles

supabase/
├── migrations/   # Database migrations
├── functions/    # Edge Functions
└── seed.sql      # Development seed data
```

See [Project Structure](docs/PROJECT_STRUCTURE.md) for the full monorepo layout.

---

## Scripts

```bash
pnpm dev             # Start web dev server
pnpm build           # Production build (all packages)
pnpm lint            # Run ESLint across workspace
pnpm test            # Run unit tests (Vitest)
pnpm e2e             # Run Playwright E2E tests
pnpm types           # Regenerate Supabase types
pnpm db:reset        # Reset local database
```

---

## Cost

| Scale | Monthly Cost |
|-------|-------------|
| Development (1 location, <150 employees) | **$0** |
| Early (2-3 locations, <300 employees) | **$0** |
| Growth (3 locations, 300-500 employees) | **$0-25** |
| Expansion (4-5 locations, 500+ employees) | **$30-55** |
| Enterprise (5+ locations, 800+ employees) | **$65-95** |

---

## License

Private — All rights reserved.

---

**Author:** Clifford Musni  
**Date:** June 2026
