# Project Structure

## Overview

The project uses a **monorepo structure** with a shared core package that separates platform-agnostic business logic from platform-specific UI. This ensures the codebase is **ready for React Native mobile app development** later without rewriting business logic, API clients, types, or state management.

### Key Principle: Shared Core

```
┌─────────────────────────────────────────────────────┐
│                  packages/core                        │
│  (Business logic, API client, types, hooks, utils)   │
│  Platform-agnostic — NO React DOM or browser APIs    │
└──────────────────────┬──────────────────────────────┘
                       │ imported by
          ┌────────────┼────────────┐
          ▼                         ▼
┌──────────────────┐     ┌──────────────────┐
│   apps/web       │     │   apps/mobile    │
│   (React + Vite) │     │  (React Native)  │
│   PWA, browser   │     │   iOS, Android   │
│   TailwindCSS    │     │   NativeWind     │
└──────────────────┘     └──────────────────┘
                          (Future — Phase 5+)
```

---

## Directory Layout

```
tyrepower-workforce-management/
├── .github/
│   └── workflows/
│       ├── ci.yml                    # Run tests on PR (all packages)
│       └── deploy.yml                # Deploy web app to Vercel on merge to main
├── docs/
│   ├── ARCHITECTURE.md
│   ├── API_DESIGN.md
│   ├── DATABASE_SCHEMA.md
│   ├── DEVELOPMENT_PLAN.md
│   ├── FEATURES.md
│   ├── PROJECT_STRUCTURE.md          # This file
│   └── SETUP_GUIDE.md
│
├── packages/
│   └── core/                         # ⭐ SHARED — reused by web AND mobile
│       ├── package.json
│       ├── tsconfig.json
│       ├── src/
│       │   ├── api/                  # Supabase API client layer
│       │   │   ├── client.ts         # Supabase client factory (platform-agnostic)
│       │   │   ├── auth.ts           # Auth API (login, signup, reset, invite)
│       │   │   ├── shifts.ts         # Clock-in/out API calls
│       │   │   ├── timesheets.ts     # Timesheet CRUD + approval
│       │   │   ├── sites.ts          # Sites API
│       │   │   ├── profiles.ts       # User profiles API
│       │   │   ├── notifications.ts  # Notifications API
│       │   │   ├── reports.ts        # Report generation API
│       │   │   └── invitations.ts    # Invitation API
│       │   ├── hooks/                # Platform-agnostic hooks (React core)
│       │   │   ├── useAuth.ts        # Auth state + actions
│       │   │   ├── useClockIn.ts     # Clock-in logic
│       │   │   ├── useClockOut.ts    # Clock-out logic
│       │   │   ├── useActiveSession.ts
│       │   │   ├── useTimesheets.ts  # Timesheet data + mutations
│       │   │   ├── useTimesheetApproval.ts
│       │   │   ├── useSites.ts
│       │   │   ├── useNotifications.ts
│       │   │   ├── useRealtime.ts    # Supabase realtime subscriptions
│       │   │   ├── useOvertime.ts    # Overtime calculation
│       │   │   ├── useReports.ts
│       │   │   ├── useUsers.ts
│       │   │   ├── useOnlineStatus.ts
│       │   │   └── useDashboardStats.ts
│       │   ├── types/                # All TypeScript types
│       │   │   ├── database.types.ts # Auto-generated from Supabase
│       │   │   ├── index.ts          # App domain types
│       │   │   ├── api.ts            # Request/response shapes
│       │   │   └── enums.ts          # Role, status, method enums
│       │   ├── utils/                # Pure utility functions
│       │   │   ├── dates.ts          # Date/time formatting, duration calc
│       │   │   ├── geofence.ts       # Haversine distance, within-radius check
│       │   │   ├── overtime.ts       # Threshold calculation logic
│       │   │   ├── formatting.ts     # Currency, hours display
│       │   │   ├── validation.ts     # Input validation rules
│       │   │   └── export.ts         # CSV/data formatting (not file I/O)
│       │   ├── constants/            # App-wide constants
│       │   │   ├── config.ts         # Default thresholds, timeouts
│       │   │   └── roles.ts          # Role permissions map
│       │   └── index.ts              # Package barrel export
│       └── tests/
│           ├── api/
│           ├── hooks/
│           └── utils/
│
├── apps/
│   └── web/                          # Web application (React + Vite PWA)
│       ├── package.json
│       ├── tsconfig.json
│       ├── vite.config.ts
│       ├── vitest.config.ts
│       ├── playwright.config.ts
│       ├── index.html
│       ├── public/
│       │   ├── favicon.ico
│       │   ├── manifest.json         # PWA manifest
│       │   ├── sw.js                 # Service worker (generated)
│       │   └── icons/                # PWA icons
│       ├── src/
│       │   ├── app/
│       │   │   ├── App.tsx           # Root component
│       │   │   ├── Router.tsx        # Web route definitions
│       │   │   └── providers/
│       │   │       ├── AuthProvider.tsx
│       │   │       ├── NotificationProvider.tsx
│       │   │       └── ThemeProvider.tsx
│       │   ├── components/
│       │   │   ├── ui/               # Web UI primitives (TailwindCSS)
│       │   │   │   ├── Button.tsx
│       │   │   │   ├── Card.tsx
│       │   │   │   ├── Input.tsx
│       │   │   │   ├── Modal.tsx
│       │   │   │   ├── Table.tsx
│       │   │   │   ├── Badge.tsx
│       │   │   │   ├── Spinner.tsx
│       │   │   │   └── Toast.tsx
│       │   │   ├── layout/           # Web layout components
│       │   │   │   ├── AppLayout.tsx
│       │   │   │   ├── Sidebar.tsx
│       │   │   │   ├── Header.tsx
│       │   │   │   └── MobileNav.tsx
│       │   │   └── shared/           # Web-specific shared components
│       │   │       ├── Map.tsx       # Mapbox GL JS (web only)
│       │   │       ├── QRScanner.tsx # react-qr-reader (web camera)
│       │   │       ├── QRDisplay.tsx
│       │   │       ├── NotificationBell.tsx
│       │   │       └── StatusBadge.tsx
│       │   ├── features/
│       │   │   ├── auth/
│       │   │   │   ├── pages/
│       │   │   │   │   ├── LoginPage.tsx
│       │   │   │   │   ├── SignupPage.tsx
│       │   │   │   │   ├── ResetPasswordPage.tsx
│       │   │   │   │   └── AcceptInvitationPage.tsx
│       │   │   │   └── components/
│       │   │   │       ├── LoginForm.tsx
│       │   │   │       └── ProtectedRoute.tsx
│       │   │   ├── clock/
│       │   │   │   ├── pages/
│       │   │   │   │   └── ClockPage.tsx
│       │   │   │   └── components/
│       │   │   │       ├── ClockInButton.tsx
│       │   │   │       ├── ClockOutButton.tsx
│       │   │   │       ├── ActiveSessionCard.tsx
│       │   │   │       ├── GPSStatusIndicator.tsx
│       │   │   │       └── QRClockIn.tsx
│       │   │   ├── timesheets/
│       │   │   │   ├── pages/
│       │   │   │   │   ├── TimesheetsPage.tsx
│       │   │   │   │   ├── TimesheetDetailPage.tsx
│       │   │   │   │   └── TimesheetApprovalPage.tsx
│       │   │   │   └── components/
│       │   │   │       ├── TimesheetCard.tsx
│       │   │   │       ├── TimesheetTable.tsx
│       │   │   │       ├── ApprovalActions.tsx
│       │   │   │       ├── BulkApprovalDialog.tsx
│       │   │   │       ├── UndoNotification.tsx
│       │   │   │       └── HoursSummary.tsx
│       │   │   ├── sites/
│       │   │   │   ├── pages/
│       │   │   │   │   ├── SitesPage.tsx
│       │   │   │   │   ├── SiteDetailPage.tsx
│       │   │   │   │   ├── SiteFormPage.tsx
│       │   │   │   │   └── SiteLandingPage.tsx
│       │   │   │   └── components/
│       │   │   │       ├── SiteCard.tsx
│       │   │   │       ├── SiteMap.tsx
│       │   │   │       ├── GeofenceEditor.tsx
│       │   │   │       └── QRCodeManager.tsx
│       │   │   ├── dashboard/
│       │   │   │   ├── pages/
│       │   │   │   │   ├── EmployeeDashboard.tsx
│       │   │   │   │   ├── ManagerDashboard.tsx
│       │   │   │   │   └── AdminDashboard.tsx
│       │   │   │   └── components/
│       │   │   │       ├── StatsCard.tsx
│       │   │   │       ├── AttendanceList.tsx
│       │   │   │       ├── PendingApprovals.tsx
│       │   │   │       ├── OvertimeWidget.tsx
│       │   │   │       └── LabourCostWidget.tsx
│       │   │   ├── reports/
│       │   │   │   ├── pages/
│       │   │   │   │   └── ReportsPage.tsx
│       │   │   │   └── components/
│       │   │   │       ├── AttendanceReport.tsx
│       │   │   │       ├── HoursReport.tsx
│       │   │   │       ├── OvertimeReport.tsx
│       │   │   │       └── ExportButton.tsx
│       │   │   ├── notifications/
│       │   │   │   └── components/
│       │   │   │       ├── NotificationList.tsx
│       │   │   │       └── NotificationItem.tsx
│       │   │   └── users/
│       │   │       ├── pages/
│       │   │       │   ├── UsersPage.tsx
│       │   │       │   ├── UserDetailPage.tsx
│       │   │       │   └── ProfilePage.tsx
│       │   │       └── components/
│       │   │           ├── UserTable.tsx
│       │   │           ├── UserForm.tsx
│       │   │           ├── InviteForm.tsx
│       │   │           └── RoleBadge.tsx
│       │   ├── platform/             # Web-specific platform implementations
│       │   │   ├── geolocation.ts    # Browser Geolocation API
│       │   │   ├── storage.ts        # localStorage / IndexedDB
│       │   │   ├── notifications.ts  # Web Push (FCM + service worker)
│       │   │   └── camera.ts         # getUserMedia for QR scanning
│       │   ├── styles/
│       │   │   └── globals.css       # TailwindCSS imports + custom styles
│       │   └── main.tsx              # Web app entry point
│       └── tests/
│           ├── unit/
│           ├── integration/
│           └── e2e/
│               ├── auth.spec.ts
│               ├── clock-in.spec.ts
│               ├── timesheets.spec.ts
│               └── invitations.spec.ts
│
├── supabase/
│   ├── config.toml                   # Supabase local config
│   ├── migrations/
│   │   ├── 20260601000000_create_organizations.sql
│   │   ├── 20260601000001_create_profiles.sql
│   │   ├── 20260601000002_create_sites.sql
│   │   ├── 20260601000003_create_site_managers.sql
│   │   ├── 20260601000004_create_geofences.sql
│   │   ├── 20260601000005_create_qr_codes.sql
│   │   ├── 20260601000006_create_time_entries.sql
│   │   ├── 20260601000007_create_timesheets.sql
│   │   ├── 20260601000008_create_notifications.sql
│   │   ├── 20260601000009_create_invitations.sql
│   │   ├── 20260601000010_create_audit_logs.sql
│   │   ├── 20260601000011_create_announcements.sql
│   │   ├── 20260601000012_enable_rls_policies.sql
│   │   └── 20260601000013_seed_initial_data.sql
│   ├── functions/
│   │   ├── clock-in/
│   │   │   └── index.ts
│   │   ├── clock-out/
│   │   │   └── index.ts
│   │   ├── submit-timesheet/
│   │   │   └── index.ts
│   │   ├── approve-timesheet/
│   │   │   └── index.ts
│   │   ├── bulk-approve-timesheets/
│   │   │   └── index.ts
│   │   ├── generate-qr-code/
│   │   │   └── index.ts
│   │   ├── validate-qr-token/
│   │   │   └── index.ts
│   │   ├── invite-employee/
│   │   │   └── index.ts
│   │   ├── accept-invitation/
│   │   │   └── index.ts
│   │   ├── get-dashboard-stats/
│   │   │   └── index.ts
│   │   ├── get-overtime-alerts/
│   │   │   └── index.ts
│   │   ├── generate-report/
│   │   │   └── index.ts
│   │   └── send-notification/
│   │       └── index.ts
│   └── seed.sql                      # Development seed data
│
├── .env.example
├── .eslintrc.cjs
├── .gitignore
├── .prettierrc
├── package.json                      # Root workspace config
├── pnpm-workspace.yaml               # Monorepo workspace definition
├── turbo.json                        # Turborepo config (optional)
├── tsconfig.base.json                # Shared TS config
├── COST_ANALYSIS.md
├── COST_ANALYSIS_SIMPLE.md
├── README.md
└── TECH_STACK_DETAILED.md
```

---

## Architecture Patterns

### Monorepo with Shared Core (Mobile-Ready)

The codebase is split into three layers:

| Layer | Path | Contains | Platform |
|-------|------|----------|----------|
| **Core** | `packages/core/` | API client, hooks, types, utils, constants | Any (React/React Native) |
| **Web App** | `apps/web/` | UI components, pages, routing, styles, platform APIs | Browser (React DOM) |
| **Mobile App** | `apps/mobile/` *(future)* | Native UI, navigation, platform APIs | iOS/Android (React Native) |

### What Goes in `packages/core/` vs `apps/web/`

```
packages/core/ (SHARED — no DOM, no browser APIs, no CSS)
├── api/        → Supabase client calls (fetch data, mutations)
├── hooks/      → useClockIn, useTimesheets, useAuth (state + logic)
├── types/      → TypeScript interfaces, enums, generated DB types
├── utils/      → Pure functions (date math, geofence calc, formatting)
└── constants/  → Config values, role maps, thresholds

apps/web/ (WEB ONLY — browser-specific)
├── components/ → React DOM components (TailwindCSS styled)
├── features/   → Page routes + feature-specific UI
├── platform/   → Browser Geolocation, IndexedDB, Web Push, Camera
└── styles/     → TailwindCSS, CSS files
```

### Rule: If It Doesn't Need a Browser, It Goes in Core

| Example | Goes in... | Reason |
|---------|-----------|--------|
| `useClockIn()` hook | `packages/core/hooks/` | Logic is the same on web and mobile |
| `<ClockInButton />` | `apps/web/features/clock/` | UI rendering is platform-specific |
| `supabase.from('shifts').insert(...)` | `packages/core/api/` | API calls are identical |
| `navigator.geolocation.getCurrentPosition()` | `apps/web/platform/` | Browser-only API |
| `calculateDistance(lat1, lon1, lat2, lon2)` | `packages/core/utils/` | Pure math — no platform deps |
| Mapbox GL JS map component | `apps/web/components/shared/` | Web-only rendering library |

### Feature-Based Organization (within apps/web)

Each feature module contains:
- **pages/** — Route-level components (full page views)
- **components/** — Feature-specific UI components

Hooks and API logic live in `packages/core/` and are imported.

### Import Rules

```typescript
// ✅ Import from core package (shared logic)
import { useClockIn } from '@tyrepower/core/hooks/useClockIn';
import { clockInApi } from '@tyrepower/core/api/shifts';
import type { Shift } from '@tyrepower/core/types';
import { calculateDistance } from '@tyrepower/core/utils/geofence';

// ✅ Import from web UI layer
import { Button } from '@/components/ui/Button';
import { ClockInButton } from '@/features/clock/components/ClockInButton';

// ✅ Import platform-specific implementation
import { getCurrentPosition } from '@/platform/geolocation';

// ❌ NEVER import browser APIs in packages/core
import { navigator } from '...'; // NOT allowed in core

// ❌ NEVER import React DOM in packages/core
import { render } from 'react-dom'; // NOT allowed in core
```

### Path Aliases

```json
// packages/core/tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@tyrepower/core/*": ["src/*"]
    }
  }
}

// apps/web/tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@tyrepower/core/*": ["../../packages/core/src/*"]
    }
  }
}
```

### Workspace Configuration (pnpm-workspace.yaml)

```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

---

## Mobile App Readiness

### When React Native is added (future Phase 5+):

```
apps/
├── web/       ← already built
└── mobile/    ← new
    ├── package.json
    ├── app/                    # Expo Router or React Navigation
    ├── components/             # Native UI (NativeWind / RN StyleSheet)
    ├── features/               # Screen-level components
    ├── platform/               # Native GPS, Camera, Push, Biometrics
    └── ...
```

### What the mobile app reuses from core (zero rewrite):
- ✅ All API client functions (`packages/core/api/`)
- ✅ All hooks — useAuth, useClockIn, useTimesheets, etc. (`packages/core/hooks/`)
- ✅ All TypeScript types (`packages/core/types/`)
- ✅ All utility functions (`packages/core/utils/`)
- ✅ All constants and config (`packages/core/constants/`)
- ✅ All business logic and validation rules

### What the mobile app implements fresh:
- 🔨 Native UI components (React Native)
- 🔨 Navigation (React Navigation / Expo Router)
- 🔨 Platform APIs (native GPS, camera for QR, push via Expo)
- 🔨 Native styling (NativeWind or StyleSheet)
- 🔨 App store assets and config

### Estimated mobile effort (with core shared):
- Without core sharing: ~60-70% of web effort
- **With core sharing: ~30-40% of web effort** (UI only)

---

## Key Files

| File | Purpose |
|------|---------|
| `packages/core/src/api/client.ts` | Supabase client factory (shared) |
| `packages/core/src/hooks/useAuth.ts` | Auth state + login/logout (shared) |
| `packages/core/src/types/database.types.ts` | Auto-generated DB types (never edit) |
| `packages/core/src/utils/geofence.ts` | Distance calculation (shared) |
| `apps/web/src/app/Router.tsx` | Web route definitions and guards |
| `apps/web/src/platform/geolocation.ts` | Browser GPS implementation |
| `apps/web/src/platform/notifications.ts` | Web Push via FCM |
| `supabase/config.toml` | Local Supabase configuration |
| `supabase/migrations/` | Database migration files |
| `supabase/functions/` | Edge Functions (Deno runtime) |
| `pnpm-workspace.yaml` | Monorepo workspace definition |
| `tsconfig.base.json` | Shared TypeScript config |
