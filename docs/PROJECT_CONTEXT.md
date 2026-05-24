```python?code_reference&code_event_index=2
content = """# Project Context: Unified Productivity System (Second Brain)

## 1. Project Overview
A custom, unified "Second Brain" designed to replace Notion, Todoist, and Apple Reminders. The core purpose is to centralize notes, tasks, quarterly planning, and financial tracking into a single relational database to allow for cross-domain data analysis and "conclusion drawing" (e.g., correlating spending with productivity or mood).

## 2. Core Philosophy: Unified Entity Model
Instead of siloed tables for tasks, notes, and projects, the system uses a **Node-based hierarchy**:
- **Everything is an `Entry`**: A Note, Task, Project, or Quarter is simply a different `type` of the same entity.
- **Recursive Hierarchy**: Entries use a `parent_id` to allow infinite nesting (e.g., Quarter > Project > Task > Sub-task).
- **Polymorphic Triggers**: Triggers (Time/Location) are detached logic that can be associated with any Entry.

## 3. Database Architecture (PostgreSQL)
The system uses a relational-document hybrid approach. Postgres is used for relational integrity, while **JSONB** is used for dynamic, Notion-like table features.

### SQL Schema
```sql

-- Enums
CREATE TYPE entry_type AS ENUM ('note', 'task', 'project', 'quarter', 'habit');
CREATE TYPE entry_status AS ENUM ('backlog', 'todo', 'in_progress', 'completed', 'archived');
CREATE TYPE trigger_type AS ENUM ('time', 'location');

-- 1. Core Entries Table
CREATE TABLE entries (
    id UUID PRIMARY KEY,
    type entry_type NOT NULL,
    parent_id UUID REFERENCES entries(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    content TEXT, -- Markdown body
    status entry_status DEFAULT 'backlog',
    progress INTEGER DEFAULT 0,
    deadline_at TIMESTAMPTZ,
    scheduled_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 2. Notification Triggers
CREATE TABLE triggers (
    id UUID PRIMARY KEY,
    entry_id UUID NOT NULL REFERENCES entries(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    type trigger_type NOT NULL,
    config JSONB NOT NULL, -- { "time": "09:00", "repeat": "daily" } or { "lat": x, "lng": y, "radius": 100 }
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 3. Dynamic Table System (Notion-lite)
CREATE TABLE table_definitions (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    columns JSONB NOT NULL, -- Array: [{"name": "Price", "type": "currency"}, {"name": "Status", "type": "select"}]
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE table_rows (
    id UUID PRIMARY KEY,
    table_def_id UUID NOT NULL REFERENCES table_definitions(id) ON DELETE CASCADE,
    data JSONB NOT NULL, -- Key-value pairs: {"Price": 15.99, "Service": "Netflix"}
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 4. Analytics Logs (For Plotting Conclusions)
CREATE TABLE logs (
    id UUID PRIMARY KEY,
    entry_id UUID REFERENCES entries(id) ON DELETE SET NULL,
    event_type TEXT NOT NULL, -- e.g., 'task_completed', 'finance_added'
    numeric_value DECIMAL, -- For charting
    metadata JSONB,
    logged_at TIMESTAMPTZ DEFAULT NOW()
);

-- 5. Taxonomy
CREATE TABLE tags (
    id UUID PRIMARY KEY,
    label TEXT UNIQUE NOT NULL
);

CREATE TABLE entry_tags (
    entry_id UUID REFERENCES entries(id) ON DELETE CASCADE,
    tag_id UUID REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (entry_id, tag_id)
);
```

## 4. Key Feature Implementation Details

### A. Location & Time Triggers

#### Time Trigger Implementation

Time triggers are evaluated by `TriggerService.EvaluateAllTriggers()`, which is called on a schedule (every minute via a ticker). All firing logic is derived purely from the `TimeConfig` — there is no stored state like `next_run` or `last_fired`.

**Config schema** (JSONB stored as string):
```json
{ "time": "<UTC timestamp>", "repeat": 60 }
```
- `time`: The UTC timestamp of the first (or only) firing. Stored and compared in UTC.
- `repeat`: Interval in minutes for recurring triggers. Set to `0` for a one-shot trigger.

**Firing algorithm** (`ShouldFireTimeTrigger`):
- **Tolerance window**: ±5 minutes around any target firing time. The scheduler runs every minute, so this window ensures a trigger is never missed.
- **One-shot** (`repeat == 0`): Fires if `|now - config.time| ≤ 5 minutes`. Deactivated (`is_active = false`) after firing.
- **Recurring** (`repeat > 0`): Fires if `(now - config.time) % repeat ≤ 5 minutes`, i.e., if the current time falls within the tolerance window of the config time or any of its repeat boundaries. Remains active after firing.

**Timezone handling**: All timestamps are stored and compared in UTC. The client is responsible for converting their local time to UTC before creating a trigger.

#### Location Trigger Implementation

Location triggers use geofencing to fire a notification when the user enters a defined radius around a point. They are always **one-shot** — once fired, the trigger is deactivated (`is_active = false`) to avoid repeatedly pinging the user while they remain at the location.

**Config schema** (JSONB stored as string):
```json
{ "lat": 51.5033, "lng": -0.1196, "radius": 500 }
```
- `lat` / `lng`: GPS coordinates of the target location.
- `radius`: Geofence radius in meters.

**Firing algorithm** (`IsWithinRadius`):
- Uses the **Haversine formula** to calculate the great-circle distance between the user's current position (from `LocationService`) and the config coordinates.
- Fires if `haversineDistance(current, config) ≤ radius`.
- After firing, the trigger is deactivated. To re-arm, the user must create a new trigger.

### B. Dynamic Tables (Finances/Subscriptions)
- **Schema-less Rows**: The `table_rows.data` JSONB column stores user-defined fields.
- **Column Validation**: `TableRow.Save()` validates that every key in the row data matches a column defined in the parent `TableDefinition`. `TableDefinition.Save()` validates that every column has a non-empty `name` field.
- **Formulas**: Stored as strings in `table_definitions.columns` (e.g., `"{Cost} * 1.15"`) and parsed on the frontend using a math parser (like math.js).
- **Rollups**: SQL aggregation queries are used to sum/average JSONB fields for quarterly reviews.

#### Finance (Budget Tracking)

The `FinanceService` is a stateless service built on top of the dynamic tables system. It manages budget buckets and tracks spending via transaction logs.

**Budget Table** — a `TableDefinition` with name "Budget" and default columns:
```json
[{"name": "Bucket", "type": "string"}, {"name": "Amount", "type": "int"}, {"name": "Spent", "type": "int"}]
```

**BucketData schema** — typed struct used for validation and (de)serialization of budget rows:
- `Bucket` (string, required): Category name (e.g., "Restaurants", "Coffee")
- `Amount` (float64, required, > 0): Budgeted amount for the period
- `Spent` (float64, defaults to 0): Running total of spending

**Transactions** are stored as `Log` entries with:
- `event_type`: `"finance_transaction"`
- `numeric_value`: Transaction amount
- `metadata`: `{"bucket": "Coffee", "budgetId": "<table_def_id>"}`. If `budgetId` is present, the matching bucket row's `Spent` is incremented.

**Key methods**:
- `CreateBudgetTable(td)` — creates the budget table definition with defaults
- `AddBucket(tr)` — validates via `BucketData.Validate()`, normalizes, and saves
- `AddTransaction(log)` — saves the log, then updates the matching bucket's `Spent`
- `Reset(budgetTableId)` — sets `Spent = 0` on all rows in the budget table

**Design**: The service is fully stateless — all IDs (budget table, bucket) are passed by the caller. No state is stored on the `FinanceService` struct.

### C. The "Insight" Engine
- Uses the `logs` table to track every interaction (habit completion, spending, energy levels).
- **Correlation**: Queries join `logs` and `entry_tags` to analyze cross-domain data (e.g., "Do high-energy days lead to more impulse spending?").

## 5. Technical Recommendations
- **Database**: PostgreSQL (Supabase recommended for Auth/Real-time).
- **Frontend**: Web (React/Next.js) with TanStack Table for the dynamic table UI.
- **Plotting**: Recharts or D3.js for visual data analysis.
- **Sync**: Use an append-only log strategy for any changes meant to be plotted over time.

## 6. Application Layer Features
The `internal/application` package builds business logic and workflows on top of the generic `store` layer.
- **Daily Journaling**: A service to quickly generate daily reflections. Creates a `note` type Entry automatically marked as `completed`. Can optionally link to a habit entry (e.g., "Daily Journaling" habit) to track streaks.
- **Task Management**: A service to validate, create, and update tasks leveraging the underlying `store.Entry` structure. Includes default status assignments.
- **Project Management**: A service to validate, create, and update projects. Enforces the `project` entry type, validates that the title is present, defaults status to `todo`, and defaults scheduled dates to `now`.
- **Quarter Planning**: A service to validate, create, and update quarterly plans. Enforces the `quarter` entry type, validates that the title is present, defaults status to `todo`, and defaults scheduled dates to `now`.

## 7. API Architecture & Contracts

The backend utilizes a generic HTTP handler system mapping `PUT`, `GET`, and `DELETE` requests to specific service methods via action strings.

**Generic Request Format:**
```json
{
  "action": "<feature_name>",
  "data": { ... }
}
```

**Journal API Example:**
- **PUT `action: "journal"`**: Creates or updates a daily journal entry.
  ```json
  {
    "action": "journal",
    "data": {
      "entry": {
        "id": "<optional_uuid>",
        "content": "<journal entry text>"
      }
    }
  }
  ```
- **GET `action: "journal.timerange"`**: Retrieves journal entries within a specific timeframe.
  ```json
  {
    "action": "journal.timerange",
    "data": {
      "startDate": "2023-01-01T00:00:00Z",
      "endDate": "2023-12-31T23:59:59Z"
    }
  }
  ```
  *(Note: For GET requests, the payload is typically passed as a URL-encoded JSON string in the `data` query parameter: `?action=journal.timerange&data={...}`)*
"""