# Mission: Second Brain — Full-Stack Architecture & Progress

## Overview
Connect the Go backend (`assistant-backend`) to the Next.js frontend (`assistant-ui`) into a cohesive, production-grade Second Brain personal productivity system.

---

## Repositories & Modules

| Module | Path | Stack | Description |
|---|---|---|---|
| `assistant-backend` | `assistant-backend/` | Go 1.23+, PostgreSQL | REST API server, business logic services, data store |
| `assistant-ui` | `assistant-ui/` | Next.js (App Router), TS, Tailwind v4 | Web client with minimalist monochrome design |
| `assistant-db` | `assistant-db/` | PostgreSQL, Supabase, Docker | Database schema, seeders, and migration tooling |
| `assistant-mobile` | `assistant-mobile/` | React Native / Expo, TypeScript | Mobile client companion |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│  assistant-ui (Next.js, :3000)                          │
│  ┌──────────┐ ┌────────┐ ┌─────────┐ ┌──────────────┐ │
│  │Dashboard │ │ Tasks  │ │ Journal │ │   Projects   │ │
│  └────┬─────┘ └───┬────┘ └────┬────┘ └──────┬───────┘ │
│       └───────────┴───────────┴──────────────┘          │
│            Typed API Client (/lib/api.ts)               │
└───────────────────────┬─────────────────────────────────┘
                        │ HTTP/JSON (Bearer Auth)
┌───────────────────────▼─────────────────────────────────┐
│  assistant-backend (Go HTTP Server, :8080)              │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Routes (/api/put, /api/get, /api/delete)        │   │
│  │  Application Layer (Tasks, Projects, Journal...) │   │
│  │  Store Layer (store.Entry, PostgreSQL)           │   │
│  └──────────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────────┘
                        │ SQL
┌───────────────────────▼─────────────────────────────────┐
│  PostgreSQL (Docker, :5432)                             │
│  Unified `entries` table (with JSONB `data` payload)    │
└─────────────────────────────────────────────────────────┘
```

---

## Implementation Status

### Phase 1 — Database & Store Layer
- [x] Unified `entries` schema with recursive parent-child hierarchy (`parent_id`)
- [x] Removed legacy, unused tables (`triggers`, `table_definitions`, `table_rows`, `logs`, `tags`, `entry_tags`)
- [x] PostgreSQL indexes for parent IDs, types, and journal dates
- [x] Seeding scripts for projects, tasks, and journal history
- [x] Automated database reset script (`nuke.bash`)

### Phase 2 — Go Backend Service
- [x] Scaffolding and graceful shutdown (`cmd/server/main.go`)
- [x] Central HTTP router with `/api/put`, `/api/get`, `/api/delete` and `/health`
- [x] Security middleware: Bearer token authentication, CORS restriction, structured `slog` logging
- [x] `TaskService`: CRUD, overdue calculations, inbox queries, date-range filtering
- [x] `ProjectService`: Project listing, child tasks/notes aggregation, progress rollups
- [x] `JournalService`: Daily reflection saving and date-range retrieval
- [x] `DailyNotesService`: Quick note capture and date queries
- [x] Unit and integration test suite passing against PostgreSQL

### Phase 3 — Next.js Web Interface
- [x] Minimalist monochrome design system (Geist font, Tailwind CSS v4, dark/light theme)
- [x] Dashboard page (`/`) with welcome banner and potted plant component
- [x] Tasks interface (`/tasks`) with Today, This Week, and Inbox views, progress bars, and modal overlay
- [x] Journaling interface (`/journal`) with TipTap block rich-text editor and timeline history
- [x] Daily Notes interface (`/daily-notes`)
- [x] Projects interface (`/projects`) with project detail views and child task lists
- [x] Typed API client wrapper with authentication headers and server/client endpoint routing
