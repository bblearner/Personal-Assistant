# Mission: Second Brain — Full-Stack Integration

## Overview
Connect the Go backend (`assistant`) to the Next.js frontend (`assistant-ui`) into a cohesive, working Second Brain application. The backend exposes a REST API; the frontend consumes it with a polished, production-quality UI.

---

## Repositories

| Repo | Path | Stack |
|---|---|---|
| `assistant` | `/Users/bhavyejain/Projects/assistant` | Go, PostgreSQL (Docker) |
| `assistant-ui` | `/Users/bhavyejain/Projects/assistant-ui` | Next.js 15, TypeScript, Tailwind |

---

## Current State

### `assistant` (Backend)
- ✅ **Store layer** (`internal/store`): Full CRUD for `entries`, `logs`, `tables`, `tags`, `triggers`
- ✅ **Application layer** (`internal/application`): `JournalService`, `TaskService`, `FinanceService`, `TriggerService`, `TagService`, `TableService`
- ❌ **HTTP layer**: No REST API server yet — no `cmd/` HTTP entrypoint, no routing, no handlers
- ✅ **Database**: PostgreSQL via Docker Compose, schema in `init.sql`

### `assistant-ui` (Frontend)
- ✅ Next.js 15 app router scaffolded
- ✅ Tailwind CSS configured
- ❌ **UI**: Default Next.js placeholder page — no real components or views built yet
- ❌ **API client**: No backend integration

---

## Mission Goals

### Phase 1 — Backend: REST API Server
Build an HTTP server in `assistant` that exposes all application-layer services.

- [ ] **1.1** Add a web framework (e.g., `net/http` stdlib or `chi`) and wire up a server in `cmd/server/main.go`
- [ ] **1.2** Implement REST handlers for **Entries** (`GET /entries`, `POST /entries`, `PATCH /entries/:id`, `DELETE /entries/:id`)
- [ ] **1.3** Implement REST handlers for **Tasks** (wrapper around entries with `type=task`)
- [ ] **1.4** Implement REST handlers for **Journal** (`POST /journal`)
- [ ] **1.5** Implement REST handlers for **Finance** (`GET /budgets`, `POST /budgets`, `POST /budgets/:id/transactions`, `POST /budgets/:id/reset`)
- [ ] **1.6** Implement REST handlers for **Tags** and **Triggers**
- [ ] **1.7** Add CORS middleware (allow `localhost:3000` in dev)
- [ ] **1.8** Add JSON error responses and structured logging

### Phase 2 — Frontend: Core UI
Replace the placeholder in `assistant-ui` with real views.

- [ ] **2.1** Design system: define color tokens, typography, spacing in `globals.css`
- [ ] **2.2** API client: typed `fetch` wrapper pointing at the Go backend (`NEXT_PUBLIC_API_URL`)
- [ ] **2.3** **Dashboard** page — summary cards (tasks due today, budget health, recent journal entries)
- [ ] **2.4** **Tasks** page — list, create, update, delete tasks; status badges
- [ ] **2.5** **Journal** page — daily journal entry creation and history
- [ ] **2.6** **Finance** page — budget buckets, add transactions, spending progress bars
- [ ] **2.7** **Tables** page — dynamic Notion-lite table view
- [ ] **2.8** Navigation sidebar with route links

### Phase 3 — Integration & Polish
- [ ] **3.1** End-to-end smoke test: create a task in the UI, confirm it persists in Postgres
- [ ] **3.2** Dark mode (Tailwind `dark:` classes, system preference)
- [ ] **3.3** Optimistic UI updates and loading skeletons
- [ ] **3.4** Error boundaries and toast notifications
- [ ] **3.5** Docker Compose: add `assistant` service so `docker compose up` starts DB + API together

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│  assistant-ui (Next.js, :3000)                          │
│  ┌──────────┐ ┌────────┐ ┌─────────┐ ┌──────────────┐ │
│  │Dashboard │ │ Tasks  │ │ Journal │ │   Finance    │ │
│  └────┬─────┘ └───┬────┘ └────┬────┘ └──────┬───────┘ │
│       └───────────┴───────────┴──────────────┘          │
│                   API Client (fetch)                     │
└───────────────────────┬─────────────────────────────────┘
                        │ HTTP/JSON
┌───────────────────────▼─────────────────────────────────┐
│  assistant (Go HTTP Server, :8080)                      │
│  ┌──────────────────────────────────────────────────┐   │
│  │  Handlers  →  Application Layer  →  Store Layer  │   │
│  └──────────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────────┘
                        │ SQL
┌───────────────────────▼─────────────────────────────────┐
│  PostgreSQL (Docker, :5432)                             │
└─────────────────────────────────────────────────────────┘
```

---

## Starting Point
**Recommended first task**: Phase 1.1 — scaffold the HTTP server in `cmd/server/main.go` with a health check endpoint (`GET /health`), wired to the existing DB connection used in tests.
