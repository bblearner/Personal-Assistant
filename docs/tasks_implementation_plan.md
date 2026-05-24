# Tasks Feature Implementation Plan

## 1. Overview
The Tasks page will serve as the central hub for managing tasks, structured into a two-column layout:
- **Main Content Area**: Displays the currently selected list of tasks (Today, Weekly, Shopping, or Inbox) along with a progress bar and an "Add a task" button.
- **Sidebar Navigation**: Contains links (or buttons) to switch views between "This week", "Shopping", and "Inbox", complete with their own progress summaries where applicable.

All task data will rely on the `entries` table using `type = "task"`.

---

## 2. Timezone & Data Strategy
Since the database stores timestamps in **UTC** and the UI works in **local time**, all time-based queries will be calculated on the client to get the exact local time bounds, which are then converted to UTC for the backend request.

**Example for "Today's Tasks":**
1. Get start of today (e.g., `00:00:00` local).
2. Get end of today (e.g., `23:59:59` local).
3. Convert both to UTC using `.toISOString()`.
4. Send to backend: `GET /api/tasks?scheduled_start=<utc_start>&scheduled_end=<utc_end>`.

---

## 3. UI Layout & Routing
To support seamless navigation and URL sharing, we will use query parameters or sub-routes to control the main view.

- **`/tasks`** (or `?view=today`): Shows Today's Tasks.
- **`/tasks?view=weekly`**: Shows Weekly Tasks.
- **`/tasks?view=shopping`**: Shows Shopping Tasks.
- **`/tasks?view=inbox`**: Shows the Inbox.

### Sidebar Navigation
- **This Week**: Links to `?view=weekly`. Shows a progress bar based on tasks scheduled this week.
- **Shopping**: Queries for tasks containing the "shopping" tag (via `entry_tags` / tag system). If > 0, renders a button linking to `?view=shopping` with a progress bar.
- **Inbox**: Links to `?view=inbox`.

---

## 4. Main Views Breakdown

### A. Today's Tasks (Default View)
- **Query**: Fetch tasks where `ScheduledAt` falls within the local "today" (converted to UTC). 
- **Header**: "Today's Tasks" alongside a Progress Bar.
- **Progress Calculation**: `(Completed Tasks Today) / (Total Tasks Scheduled Today) * 100`.
- **List**: Renders tasks using the `Lightweight Task Component`.
- **Footer**: "Add a task" inline button to quickly append a new task to today's list.

### B. Weekly Tasks
- **Query**: Fetch tasks where `ScheduledAt` falls between this week's Monday and Sunday (local bounds converted to UTC).
- **Header**: "This Week's Tasks" with Progress Bar.
- **List & Footer**: Same structure as Today's Tasks.

### C. Shopping
- **Query**: Fetch tasks that have a relation in the tag system with the tag name "shopping".
- **Header**: "Shopping List" with Progress Bar.
- **List & Footer**: Same structure as Today's Tasks.

### D. Inbox
- **Query**: Fetch all tasks (regardless of date) but exclude tasks where `status = 'archived'`.
- **Header**: "Inbox".
- **Layout**: Renders the reusable `KanbanBoard` component.
  - Passes the fetched tasks to the board.
  - The board manages columns based on `entry_status` (Backlog, Todo, In Progress, Completed).

---

## 5. Reusable Components

### A. Lightweight Task Component (`TaskListItem`)
- **Purpose**: The default item shown in lists.
- **Visuals**: Behaves as a clickable button/row. Shows Title, Scheduled Time, and Priority.
- **Interactions**: 
  - Clicking opens the `TaskOverlay` component.
  - Can handle auto-saving if inline quick-edits (like toggling status) are supported.

### B. Overlay Task Component (`TaskOverlay`)
- **Purpose**: A detailed, centered Modal dialog layered over the main screen.
- **Fields**: 
  - Editable Title
  - Editable Content (Description)
  - Priority Selector (Saved within the `metadata` JSONB column, e.g., `{"priority": "high"}`).
  - Scheduled Date/Time Picker
  - Child Tasks (sub-tasks rendered as clickable buttons)
  - Status/Other details
- **Auto-Saving**: Any change to title, content, priority, etc., triggers a debounced auto-save to the backend, similar to the Journal component.

### C. Kanban Board Component (`KanbanBoard`)
- **Purpose**: A reusable board component that organizes tasks into columns based on their `status`.
- **Functionality**:
  - Displays dynamic columns (e.g., Backlog, Todo, In Progress, Completed).
  - Uses the `TaskListItem` or a specific board-card component for individual tasks.
  - (Optional but recommended) Supports drag-and-drop to easily change the status of a task.
  - Handles updating the task status via the API when moved between columns.
