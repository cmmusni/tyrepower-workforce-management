# Development Setup Guide

## Prerequisites

Ensure you have the following installed:

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 20+ LTS | JavaScript runtime |
| npm | 10+ | Package manager (comes with Node) |
| Git | 2.40+ | Version control |
| Supabase CLI | Latest | Local database & functions |
| Docker Desktop | Latest | Required by Supabase CLI |
| VS Code | Latest | IDE (recommended) |

---

## Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/<your-org>/tyrepower-workforce-management.git
cd tyrepower-workforce-management

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.example .env.local

# 4. Start Supabase locally (requires Docker)
npx supabase start

# 5. Run database migrations
npx supabase db reset

# 6. Start the development server
npm run dev
```

The app will be available at `http://localhost:5173`

---

## Environment Variables

Create `.env.local` with:

```env
# Supabase (local)
VITE_SUPABASE_URL=http://127.0.0.1:54321
VITE_SUPABASE_ANON_KEY=<your-local-anon-key>

# Supabase (production - DO NOT commit)
# VITE_SUPABASE_URL=https://<project>.supabase.co
# VITE_SUPABASE_ANON_KEY=<your-prod-anon-key>

# Mapbox
VITE_MAPBOX_TOKEN=<your-mapbox-token>

# Firebase (Push Notifications)
VITE_FIREBASE_API_KEY=<your-firebase-key>
VITE_FIREBASE_PROJECT_ID=<your-project-id>
VITE_FIREBASE_MESSAGING_SENDER_ID=<your-sender-id>
VITE_FIREBASE_APP_ID=<your-app-id>
VITE_FIREBASE_VAPID_KEY=<your-vapid-key>
```

> **Note:** After running `npx supabase start`, the CLI outputs the local anon key. Copy it into `.env.local`.

---

## Supabase Local Development

### Starting Supabase

```bash
# Start all Supabase services (DB, Auth, Storage, Edge Functions)
npx supabase start

# This outputs:
# API URL: http://127.0.0.1:54321
# GraphQL URL: http://127.0.0.1:54321/graphql/v1
# DB URL: postgresql://postgres:postgres@127.0.0.1:54322/postgres
# Studio URL: http://127.0.0.1:54323
# Anon key: eyJ...
# Service role key: eyJ...
```

### Supabase Studio (Database GUI)

Open `http://127.0.0.1:54323` for a local version of the Supabase Dashboard where you can:
- Browse tables
- Run SQL queries
- View RLS policies
- Manage auth users
- View storage buckets

### Database Migrations

```bash
# Create a new migration
npx supabase migration new <migration_name>

# Apply all migrations (resets local DB)
npx supabase db reset

# Generate types from database schema
npx supabase gen types typescript --local > src/types/database.types.ts

# Diff local changes to generate migration
npx supabase db diff --use-migra -f <migration_name>
```

### Edge Functions

```bash
# Create a new Edge Function
npx supabase functions new <function_name>

# Serve functions locally (hot reload)
npx supabase functions serve

# Deploy a function to production
npx supabase functions deploy <function_name>
```

### Stopping Supabase

```bash
npx supabase stop
```

---

## Project Scripts

```json
{
  "dev": "vite",
  "build": "tsc && vite build",
  "preview": "vite preview",
  "lint": "eslint . --ext ts,tsx",
  "lint:fix": "eslint . --ext ts,tsx --fix",
  "format": "prettier --write \"src/**/*.{ts,tsx,css}\"",
  "test": "vitest",
  "test:run": "vitest run",
  "test:coverage": "vitest run --coverage",
  "e2e": "playwright test",
  "types": "npx supabase gen types typescript --local > src/types/database.types.ts",
  "db:reset": "npx supabase db reset",
  "db:migrate": "npx supabase migration new"
}
```

---

## VS Code Extensions (Recommended)

Create `.vscode/extensions.json`:

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "bradlc.vscode-tailwindcss",
    "dsznajder.es7-react-js-snippets",
    "formulahendry.auto-rename-tag",
    "ms-playwright.playwright",
    "supabase.supabase-vscode"
  ]
}
```

---

## External Service Setup

### 1. Supabase Cloud Project

1. Go to [supabase.com](https://supabase.com) → New Project
2. Enable the PostGIS extension: SQL Editor → `CREATE EXTENSION postgis;`
3. Copy the project URL and anon key
4. Link local CLI: `npx supabase link --project-ref <project-id>`

### 2. Mapbox Account

1. Sign up at [mapbox.com](https://www.mapbox.com/)
2. Create an access token (public scope for client-side)
3. Add token to `.env.local`
4. Free tier: 25,000 map loads/month

### 3. Firebase (Push Notifications)

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Create a project (disable Analytics if not needed)
3. Add a Web App
4. Enable Cloud Messaging
5. Generate a VAPID key pair (Project Settings → Cloud Messaging)
6. Copy config values to `.env.local`

### 4. Postmark (Email - Phase 3)

1. Sign up at [postmarkapp.com](https://postmarkapp.com)
2. Create a server
3. Get the Server API Token
4. Add to Supabase Edge Function secrets:
   ```bash
   npx supabase secrets set POSTMARK_API_TOKEN=<token>
   ```

### 5. Vercel (Deployment)

1. Sign up at [vercel.com](https://vercel.com)
2. Import the GitHub repository
3. Set environment variables in Vercel Dashboard
4. Configure: Framework = Vite, Build Command = `npm run build`, Output = `dist`
5. Auto-deploys on push to `main`

---

## Git Workflow

### Branches

| Branch | Purpose |
|--------|---------|
| `main` | Production-ready code (auto-deploys) |
| `develop` | Integration branch for features |
| `feature/<name>` | Individual feature branches |
| `fix/<name>` | Bug fix branches |

### Commit Convention

```
type(scope): description

feat(clock-in): add GPS geofence validation
fix(timesheet): correct overtime calculation
chore(deps): update supabase-js to v2.40
docs(api): add clock-out endpoint docs
```

Types: `feat`, `fix`, `chore`, `docs`, `style`, `refactor`, `test`

### Pull Request Flow

1. Create feature branch from `develop`
2. Implement changes with commits
3. Push and create PR to `develop`
4. Self-review checklist
5. Merge to `develop`
6. When sprint complete, merge `develop` → `main`

---

## Troubleshooting

### Supabase won't start
```bash
# Ensure Docker Desktop is running
docker ps

# Reset Supabase
npx supabase stop --no-backup
npx supabase start
```

### PostGIS not available
```sql
-- Run in Supabase Studio SQL Editor
CREATE EXTENSION IF NOT EXISTS postgis;
```

### Type errors after schema change
```bash
# Regenerate TypeScript types
npx supabase gen types typescript --local > src/types/database.types.ts
```

### GPS not working in development
- Chrome requires HTTPS for Geolocation API (except localhost)
- Use `http://localhost:5173` (not 127.0.0.1)
- Check browser permissions

### Hot reload not working
```bash
# Clear Vite cache
rm -rf node_modules/.vite
npm run dev
```
