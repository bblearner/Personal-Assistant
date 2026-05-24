# UI Implementation Plan: Second Brain

## 1. Tech Stack & Architecture
- **Framework:** Next.js (App Directory)
- **Language:** TypeScript
- **Styling:** Tailwind CSS (Vanilla CSS where necessary for complex micro-animations)
- **Icons:** Lucide React (clean, minimalist, black/white friendly)
- **State Management/Data Fetching:** React Query or Next.js native fetching to interact with the backend API.

## 2. Design Language: Minimalist Monochrome
Following your constraints, the design will feel premium and responsive without relying on heavy colors.
- **Backgrounds:** Pure White (`bg-white`) for the main canvas.
- **Text & Icons:** Deep Black (`text-black`, `text-neutral-900`) for high contrast and readability.
- **Borders/Dividers:** Very subtle, thin black or gray lines (`border-black/10` or `border-neutral-200`) to separate sections without clutter.
- **Typography:** Modern, clean sans-serif (e.g., Inter, Geist, or Roboto) for high legibility.
- **Interactions (The "Wow" Factor):** 
  - Smooth micro-animations on hover (e.g., subtle scaling, underline expansions).
  - Glassmorphism or blurred active states for navigation to make the UI feel alive.
  - Seamless page transitions.

## 3. Application Layout
The core layout will feature a persistent Side Navigation panel and a main content area.

### Side Navigation (`/components/layout/SideNav.tsx`)
- **Container:** Fixed left sidebar, white background, separated from the main content by a delicate 1px border.
- **Header:** Simple text logo "Second Brain" or a minimalist geometric icon.
- **Navigation Links:**
  - **Home** (`/`) - Icon: `Home`
  - **Journaling** (`/journal`) - Icon: `BookOpen` or `PenTool`
  - **Finances** (`/finances`) - Icon: `PieChart` or `Wallet`
  - **Tasks** (`/tasks`) - Icon: `CheckSquare`
  - **Logging** (`/logs`) - Icon: `Activity` or `BarChart2`
- **Active State:** Bold text, black icon, and a subtle black background indicator (like a soft pill shape or left border).
- **Inactive State:** Dark gray text/icons, transitioning smoothly to black on hover.

## 4. Routing Structure (Next.js App Router)
```text
assistant-ui/
├── app/
│   ├── layout.tsx         # Contains the SideNav and main content wrapper
│   ├── page.tsx           # Homepage (TBD)
│   ├── journal/
│   │   └── page.tsx       # Journaling feature
│   ├── finances/
│   │   └── page.tsx       # Finances feature
│   ├── tasks/
│   │   └── page.tsx       # Tasks feature
│   └── logs/
│       └── page.tsx       # Logging/Analytics feature
```

## 5. Feature Deep Dive: Journaling
The journaling feature allows the user to quickly generate daily reflections (saved as `note` entries).

### A. UI Layout (`/journal`)
A clean, distraction-free writing environment.
- **Header:** "Journal" title with an elegant, minimalist date selector.
- **Two-Column Layout:**
  - **History Sidebar (Left):** A left-aligned column showing a snippet of the last 7 days of journal history.
  - **Main Area (Editor):** A large, minimalist text area for today's journal or the selected journal.

### B. Core Components
1. **`JournalEditor`:**
   - Powered by **TipTap** (a headless, block-based rich text editor). This aligns perfectly with the minimalist design as it brings no default styling, allowing full Tailwind customization.
   - Supports Markdown shortcuts (e.g., `#` for headings, `-` for lists) for a seamless, keyboard-driven "Second Brain" writing experience.
   - "Ghost" placeholder text (e.g., "What's on your mind today?").
   - Auto-save functionality with a discrete, fading "Saved at 10:42 AM" indicator at the bottom.
2. **`JournalHistory`:**
   - A vertical sidebar column on the left side displaying journal history for the last 7 days.
   - Fetches data using the `journal.timerange` API with `startDate` (7 days ago) and `endDate` (today).
   - Shows the date, a short snippet of the content, and smooth hover interactions.
3. **`DateRangeSelector`:**
   - Minimalist date picker to filter the history (e.g., "This Week", "Last 30 Days", or a custom range) to support the `journal.timerange` API.

### C. API Integration (Based on Backend Contracts)
- **Save Journal:**
  - Endpoint: `PUT` request.
  - Payload: `{ "action": "journal", "data": { "entry": { "content": "..." } } }`
- **Fetch Journals:**
  - Endpoint: `GET` request.
  - Query: `?action=journal.timerange&data={"startDate": "...", "endDate": "..."}`
  - Used to populate the `JournalTimeline`.

## 6. Implementation Steps (When Ready)
1. Initialize the Next.js project with Tailwind and TypeScript in the `assistant-ui` folder.
2. Set up global CSS to enforce the minimalist monochrome design system and typography.
3. Build the `SideNav` component and wrap it in the root `layout.tsx`.
4. Implement the `/journal` route with mocked components (Editor and Timeline).
5. Connect the Journal components to the Go backend API.
