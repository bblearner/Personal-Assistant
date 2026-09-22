# Tasks Feature Implementation Plan & Architecture

## 1. Overview
The Tasks feature serves as the central hub for managing tasks, structured into a two-column responsive layout:
- **Sidebar Navigation (`components/tasks/Sidebar.tsx`)**: Quick switcher for filtering tasks by category (*Today*, *This Week*, and *Inbox*), along with progress summaries.
- **Main Content Area**: Displays the filtered list of tasks along with an overall progress bar, inline task creation, completion toggles, and detail overlay modals.

All task data relies on the unified `entries` table using `type = "task"`.

---

## 2. Timezone & Date Strategy
The database stores all timestamps in **UTC**, while the frontend works in the user's **local timezone**.
- The client calculates the start and end of the local time window (e.g. today from `00:00:00` to `23:59:59`).
- Bounds are formatted as ISO UTC strings (`.toISOString()`).
- Requests are dispatched to the backend via `GET /api/get?action=task.timerange&data={"startDate":"...","endDate":"..."}`.
- Overdue tasks are retrieved via `action: "task.overdue_timerange"`, which pulls both tasks scheduled for the window and past uncompleted tasks.

---

## 3. Supported Views

### A. Today's Tasks
- **Query**: Tasks scheduled for the current calendar date (plus past overdue tasks).
- **Header**: "Today's Tasks" with completion progress percentage.
- **List**: Renders tasks with checkbox completion toggle, priority tag, and title.
- **Actions**: Quick addition of new tasks with default scheduled date set to today.

### B. This Week's Tasks
- **Query**: Tasks scheduled between Monday and Sunday of the active week.
- **Header**: "This Week's Tasks" with weekly progress bar.

### C. Inbox
- **Query**: All active tasks regardless of date (`action: "task.inbox"`), excluding archived tasks.
- **Header**: "Inbox".

---

## 4. Components

### A. Task Item
- Displays completion checkbox, title, priority badge, and schedule indicator.
- Clicking any task opens the `TaskOverlay` modal for full editing.
- Completing a parent task automatically updates its status to `completed`.

### B. Task Modal (`TaskOverlay`)
- **Title**: Inline editable title.
- **Content**: Detailed Markdown notes/description.
- **Priority Selector**: Low, Medium, High (stored in `entry.data.priority`).
- **Schedule**: Date and time selector (stored in `entry.data.scheduled_at`).
- **Child Subtasks**:
  - List of child tasks where `parent_id == task.id`.
  - Inline completion toggle for subtasks.
  - "Add subtask" button opening `NewChildOverlay`.
  - Automatic completion rollup: parent task progress updates as subtasks are finished.

### C. Add Subtask Dialog (`NewChildOverlay`)
- Clean modal allowing rapid entry of subtask title.
- Automatically links `parent_id` to the parent task.

---

## 5. Data Model (`TaskEntry` & `TaskData`)

```typescript
export type TaskData = {
  status?: "backlog" | "todo" | "in_progress" | "completed" | "archived";
  progress?: number;
  scheduled_at?: string | null;
  deadline_at?: string | null;
  completed_at?: string | null;
  priority?: "low" | "medium" | "high";
};

export type TaskEntry = {
  id: string;
  type: string;
  parent_id?: string | null;
  title: string;
  content?: string;
  data?: TaskData;
  created_at: string;
  children?: TaskEntry[];
};
```
