# UI Implementation Plan: Second Brain

## 1. Tech Stack & Architecture
- **Framework:** Next.js (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS (v4) with CSS variables for dynamic theming
- **Icons:** Lucide React (clean, minimalist icons)
- **Rich Text Editor:** TipTap headless editor for daily reflections and notes
- **State & Data Fetching:** Native fetch client with typed backend wrapper (`/lib/api.ts`)

---

## 2. Design Language: Minimalist Monochrome
Following a sleek, distraction-free aesthetic:
- **Backgrounds:** Crisp light surfaces (`--background: #ffffff`, `--surface: #ffffff`) and deep OLED dark surfaces (`--background: #000000`, `--surface: #0a0a0a`).
- **Typography:** Geist font family with high legibility, clean hierarchy, and strict proportional scaling.
- **Borders & Separators:** Thin, subtle borders (`border-border`) preserving spatial structure without visual noise.
- **Interactions:**
  - Micro-animations on hover (subtle scale shifts, hover pill highlights).
  - Expandable sidebar with smooth transitions.
  - Minimalist custom SVG illustrations (e.g. `PottedPlant`).

---

## 3. Application Layout & Navigation

### Primary Sidebar Navigation (`components/Sidebar.tsx`)
- Persistent collapsible left sidebar with theme toggle and expandable label drawer.
- **Navigation Links:**
  - **Home** (`/`) — Icon: `Home`
  - **Journaling** (`/journal`) — Icon: `BookOpen`
  - **Daily Notes** (`/daily-notes`) — Icon: `NotebookPen`
  - **Projects** (`/projects`) — Icon: `FolderKanban`
  - **Tasks** (`/tasks`) — Icon: `CheckSquare`
- **Bottom Actions:** Dark/Light mode toggle persisted via cookie (`app/actions.ts`).

---

## 4. Routing Structure

```text
assistant-ui/
├── app/
│   ├── layout.tsx             # Root layout wrapping Sidebar and theme provider
│   ├── page.tsx               # Minimalist Dashboard with welcome banner & potted plant
│   ├── tasks/
│   │   └── page.tsx           # Tasks views (Today, Weekly, Inbox)
│   ├── journal/
│   │   └── page.tsx           # Daily journaling interface with TipTap editor & history
│   ├── daily-notes/
│   │   └── page.tsx           # Daily scratchpad notes
│   ├── projects/
│   │   ├── page.tsx           # Project overview and creation
│   │   └── [id]/page.tsx      # Project detail view (child tasks & notes)
│   └── globals.css            # Tailwind tokens, dark mode variables, TipTap styles
```

---

## 5. Feature Deep Dive

### A. Dashboard (`/`)
- Focused, serene welcome screen greeting the user.
- High-contrast typography paired with a handcrafted vector potted plant illustration that scales responsively across mobile, tablet, and desktop viewports.

### B. Tasks (`/tasks`)
- **View Modes:**
  - **Today**: Filtered by tasks scheduled for the current day.
  - **Weekly**: Filtered by tasks scheduled between Monday and Sunday of the current week.
  - **Inbox**: All active tasks.
- **Task Management Modal (`TaskOverlay`)**:
  - Task title and rich notes/content.
  - Priority badge selection (Low, Medium, High).
  - Date scheduler.
  - Interactive subtasks list with instant child creation.

### C. Journaling (`/journal`)
- TipTap block-based rich text editor supporting headings, lists, quotes, and code blocks.
- Debounced auto-save directly to backend `/api/put` with `action: "journal"`.
- History sidebar for reviewing previous reflections.

### D. Projects (`/projects`)
- High-level overview of active initiatives.
- Automatic completion percentage rollups based on child tasks.
- Detail page (`/projects/[id]`) aggregating project tasks, status badges, and project notes.
