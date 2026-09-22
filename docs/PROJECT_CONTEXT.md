# Project Context: Unified Productivity System (Second Brain)

## 1. Project Overview
A custom, unified "Second Brain" designed to replace Notion, Todoist, and Apple Reminders. The core purpose is to centralize notes, tasks, projects, quarterly planning, and daily reflections into a single relational database with an intuitive, minimalist user interface.

## 2. Core Philosophy: Unified Entity Model
Instead of siloed tables for tasks, notes, and projects, the system uses a **Node-based hierarchy**:
- **Everything is an `Entry`**: A Note, Task, Project, Journal, Daily Note, Activity, or Quarter is simply a different `type` of the same entity.
- **Recursive Hierarchy**: Entries use a `parent_id` to allow infinite nesting (e.g., Quarter > Project > Task > Sub-task).
- **Flexible JSONB Data**: Type-specific attributes are stored within the `data` JSONB column, avoiding schema migrations for new attributes.

## 3. Database Architecture (PostgreSQL)
The database uses a relational-document hybrid approach: PostgreSQL provides relational integrity and cascading parent-child deletions, while **JSONB** allows flexible attributes.

### SQL Schema (`init.sql`)
```sql
-- Enums
CREATE TYPE entry_type AS ENUM ('note', 'journal', 'activity', 'bucket', 'project', 'habit', 'task');

-- 1. Core Entries Table
CREATE TABLE entries (
    id UUID PRIMARY KEY,
    parent_id UUID REFERENCES entries(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    content TEXT,
    type entry_type NOT NULL,
    data JSONB, -- Flexible store for type-specific fields
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_entries_parent ON entries (parent_id);
CREATE INDEX idx_entries_type ON entries (type);
CREATE INDEX idx_entries_journal_date
    ON entries ((data->>'journal_date'), created_at DESC)
    WHERE type = 'journal';

-- Enable Row Level Security (RLS) to resolve Supabase security warnings
ALTER TABLE entries ENABLE ROW LEVEL SECURITY;
```

---

## 4. Entity Types & Data Schemas

### A. Tasks (`type = 'task'`)
Tasks represent actionable items that can be scheduled, organized by status, prioritized, and nested under projects or parent tasks.

**`data` JSONB Structure:**
```json
{
  "status": "todo",         // "backlog" | "todo" | "in_progress" | "completed" | "archived"
  "priority": "high",       // "low" | "medium" | "high"
  "scheduled_at": "2026-09-13T12:00:00Z",
  "deadline_at": null,
  "completed_at": null,
  "progress": 0
}
```

### B. Projects (`type = 'project'`)
Projects group related tasks and notes together. Child entries reference the project ID via `parent_id`.
- Automatic progress rollup calculates the percentage of completed tasks under a project.

### C. Journal Entries (`type = 'journal'`)
Daily reflection entries. The `data` JSONB column stores `"journal_date": "YYYY-MM-DD"`, which is indexed for rapid date-range retrieval.

### D. Daily Notes (`type = 'note'`)
Quick daily scratchpad notes associated with a specific date.

### E. Activity Tracking (`type = 'activity'`)
Tracks time spent on activity categories.
```json
{
  "time_entries": [
    { "date": "2026-07-28T23:50:00Z", "duration": 3600 }
  ],
  "total_duration": 3600
}
```

---

## 5. Application Layer Services

The backend (`internal/application`) encapsulates domain logic around the unified `store`:
- **`TaskService`**: Task creation, date-range queries (`task.timerange`, `task.overdue_timerange`), inbox listing, and cascade deletion.
- **`ProjectService`**: Project portfolio management, fetching child tasks and notes (`project.children`), and progress tracking.
- **`JournalService`**: Saving and querying reflections across date ranges.
- **`DailyNotesService`**: Fast note creation and date-based retrieval.
- **`NoteService`**: General note management.
- **`ActivityService`**: Activity logging and duration aggregation.

---

## 6. API Architecture & Action Contracts

The backend provides a unified HTTP dispatcher where requests are routed using an `action` string:
- `POST /api/put`
- `GET /api/get`
- `DELETE /api/delete`
- `GET /health`

### Request Examples

#### 1. Save / Update Task (`POST /api/put`)
```json
{
  "action": "task",
  "data": {
    "entry": {
      "type": "task",
      "title": "Review pull request",
      "parent_id": "<optional_parent_id>",
      "data": {
        "status": "in_progress",
        "priority": "medium",
        "scheduled_at": "2026-09-13T12:00:00Z"
      }
    }
  }
}
```

#### 2. Query Tasks by Date Range (`GET /api/get`)
```
GET /api/get?action=task.timerange&data={"startDate":"2026-09-13T00:00:00Z","endDate":"2026-09-13T23:59:59Z"}
```

#### 3. Fetch Project Children (`GET /api/get`)
```
GET /api/get?action=project.children&data={"id":"<project_uuid>"}
```

---

## 7. Technology Stack Summary

| Component | Technology |
|---|---|
| **Database** | PostgreSQL 15+ (Supabase compatible) |
| **Backend** | Go 1.23+, Standard Library `net/http`, `log/slog` |
| **Frontend** | Next.js 15+ (App Router), TypeScript, Tailwind CSS v4, TipTap |
| **Mobile** | React Native / Expo (in `assistant-mobile`) |
| **Deployment** | Docker Compose for local development; Vercel (UI), Railway/Render (API), Neon/Supabase (DB) for production |