Kanban Task Board
A Trello-style task board built with React + Vite. State-driven UI: the DOM
never gets touched directly — every add/delete/move action goes through
`useState`, and React re-renders the columns from that state.
Status
[x] Phase 1 — Base MVP & Component Logic
[x] Phase 2 — Inline editing, priority system, localStorage persistence
[x] Phase 3 — Drag-and-drop (dnd-kit), live search filtering (this delivery)
Getting started
```bash
npm install
npm run dev
```
Then open the printed local URL (typically `http://localhost:5173`).
To produce a production build:
```bash
npm run build
npm run preview
```
Architecture
```
src/
  App.jsx                 State owner: the `tasks` array + addTask/deleteTask/editTask/reorderTask
  components/
    Board.jsx               DndContext + sensors live here; owns the search query state
    Column.jsx               Droppable container (useDroppable) + SortableContext per column
    TaskCard.jsx              Draggable card (useSortable) with a dedicated drag handle
    AddTaskForm.jsx           Controlled input + priority dropdown for creating new tasks
    SearchBar.jsx             Controlled input for the live task filter
  data/
    columns.js               Single source of truth for column ids/titles/order
    priorities.js             Single source of truth for the priority levels
  utils/
    createId.js               Task id generation
```
Phase 2 additions
Inline editing — clicking a task's text swaps it for a text input
(`TaskCard`'s own `isEditing` state). `Enter` or blur commits via
`onEditTask`; `Escape` reverts. Empty edits are discarded rather than
saved, so a task can never be blanked by accident.
Priority system — `AddTaskForm` has a High/Medium/Low dropdown
(defaults to Medium). `TaskCard` applies a modifier class
(`task-card--high/medium/low`) that colors the card's left border
(red/yellow/green) and a small priority tag.
Persistence — `App.jsx` lazily reads `localStorage` on first render
and re-saves the full `tasks` array on every change via `useEffect`, so
a hard refresh restores the board exactly as it was left. Both read and
write are wrapped in `try/catch` so a corrupted value or unavailable
storage (private browsing, quota) degrades gracefully instead of
crashing the app.
Why one `tasks` array instead of three?
Each task is `{ id, text, status }`, where `status` is `"todo"`, `"inProgress"`,
or `"done"`. Columns are derived by filtering this one array rather than
kept as three separately-managed arrays. That means:
Move Task is a one-line update (`status` field changes) instead of
removing an item from one array and re-inserting it into another.
There's a single source of truth, so it's impossible for a task to end up
duplicated or missing across columns.
It sets up Phase 3 cleanly: this is the same data shape most drag-and-drop
libraries (including `dnd-kit`) expect.
Prop drilling
As the sprint asks: `App → Board → Column → TaskCard` is a deliberate,
explicit prop chain (no Context API yet) so state flow is easy to trace by
hand while learning the pattern.
Phase 3 additions
Drag-and-drop (dnd-kit) — the Move buttons are gone entirely, replaced
by physical dragging:
`Board.jsx` owns the single `DndContext`, sensors (`PointerSensor` with
a 5px activation threshold so clicks don't misfire as drags, plus
`KeyboardSensor` for keyboard-accessible dragging), and `onDragEnd`.
`Column.jsx` is a droppable target via `useDroppable` (so you can drop
onto an empty column) wrapping a `SortableContext` (so cards within
it can be reordered).
`TaskCard.jsx` is draggable via `useSortable`, but the drag listeners
are scoped to a small handle (the three-line grip icon) — not the
whole card — so clicking the task text still opens inline editing
instead of starting a drag.
All the actual state logic (deciding the new status, where to splice
the task back into the array) lives in one function, `App.jsx`'s
`reorderTask`, which handles both "moved to a different column" and
"reordered within the same column" the same way.
Live search — `Board.jsx` holds its own `searchQuery` state (it's a
display filter, not persisted data, so it doesn't belong in `App`) and
filters the tasks passed into each `Column` in real time as you type.
Design
Visual direction is a "drafting desk" — kraft-paper folders as columns,
tilted index cards as tasks — rather than a generic dashboard/card-grid
look. Tokens live in `src/index.css`.
Prerequisite compliance
Scaffolded with `npm create vite@latest` (React template), per the sprint's
prerequisite. No `create-react-app`.
