# Second Brain (Personal Assistant)

A unified personal productivity system designed to centralize tasks, notes, projects, and daily reflections into a single relational database with a clean, minimalist user interface.

---

## Monorepo Projects

| Project | Directory | Technology | Description |
|---|---|---|---|
| **Backend** | [`assistant-backend/`](file:///Users/bhavyejain/Projects/assistant/assistant-backend/README.md) | Go 1.23+, PostgreSQL | REST API server with action-based dispatcher, auth middleware, and business services |
| **Frontend** | [`assistant-ui/`](file:///Users/bhavyejain/Projects/assistant/assistant-ui/README.md) | Next.js 15, TypeScript, Tailwind v4 | Minimalist web application with dark/light themes, TipTap editor, and task management |
| **Database** | [`assistant-db/`](file:///Users/bhavyejain/Projects/assistant/assistant-db/README.md) | PostgreSQL, Supabase, Docker | Database schema (`entries` table), reset script (`nuke.bash`), and data seeders |
| **Mobile** | `assistant-mobile/` | React Native / Expo, TypeScript | Mobile companion application |

---

## Quick Start (Docker Compose)

Start the entire stack locally with Docker Compose:

```bash
docker compose up -d
```

- **Web UI:** [http://localhost:3000](http://localhost:3000)
- **Backend API:** [http://localhost:8080](http://localhost:8080)
- **Database:** `localhost:5432` (`second_brain`)

### Seeding Data
Once the containers are running:
```bash
cd assistant-db
./seed-projects.sh
./seed-tasks.sh
./seed-data.sh
```

---

## Documentation

Detailed architecture guides and technical references can be found in the [`docs/`](file:///Users/bhavyejain/Projects/assistant/docs) directory:
- [Project Context & Data Model](file:///Users/bhavyejain/Projects/assistant/docs/PROJECT_CONTEXT.md)
- [Mission & Implementation Status](file:///Users/bhavyejain/Projects/assistant/docs/mission.md)
- [UI Architecture Plan](file:///Users/bhavyejain/Projects/assistant/docs/UI_PLAN.md)
- [Tasks Feature Architecture](file:///Users/bhavyejain/Projects/assistant/docs/tasks_implementation_plan.md)
- [Production Deployment Plan](file:///Users/bhavyejain/Projects/assistant/docs/DEPLOYMENT_PLAN.md)
