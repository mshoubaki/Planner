# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- `pnpm --filter @workspace/api-server run dev` — run API server locally

## Artifacts

- `mpp-viewer` (web) — Single-page app to import & view Microsoft Project
  files (XML, .mpp), Excel/CSV. Renders a Gantt with collapsible hierarchy,
  multi-level zoom, theme/display settings, auto-hiding resizable side
  panes, and in-place task editing (incl. drag-to-edit Gantt bars).
  Excel/CSV parsing is template-free: `parseExcelFile` in
  `src/lib/parseProject.ts` auto-detects the best sheet, scores rows for the
  header, fuzzy-matches columns to roles (name/start/finish/duration/%
  complete/parent/wbs/outline/resources/predecessors/notes/milestone/…),
  sniffs date columns when headers are unrecognized, and infers task
  hierarchy from (in priority) explicit Parent refs, Outline Level, dotted
  WBS / outline numbers, or leading-whitespace indentation in the name.
  Task pane supports: add task / sub-task, indent / outdent (toolbar
  buttons **and** Tab / Shift+Tab keyboard shortcuts when focus is on a
  row), delete (with descendants), inline rename via double-click, and
  HTML5 drag-and-drop reorder with three drop zones per row (top 25% =
  drop **before**, middle 50% = drop **into** as child, bottom 25% = drop
  **after**); cycle prevention blocks dropping a task into its own
  descendants. The `rebuildTaskTree` helper in `App.tsx` re-derives
  outline levels, outline numbers, sequential IDs, summary flags, and
  bottom-up date / % rollups after every structural change.
  Task table columns (#, Start, Finish, Duration, %) are user-resizable
  via header dividers (widths persisted in `localStorage` under
  `mpp-viewer-tasktable-cols-v1`). Vertical scrolling is synchronized
  between the task table and the Gantt chart (each pane forwards a
  scroll handle via `forwardRef`; an rAF-guarded source ref in `App.tsx`
  prevents feedback loops). Date inputs in the Details pane accept
  pasted dates in many human formats (ISO, US m/d/y, EU d/m/y, "Apr 30
  2026", etc.); the Finish picker has `min={start}` and auto-prefills
  with Start so its calendar opens at the correct month. New tasks
  default Start to today. An **Export Excel** toolbar button writes a
  round-trippable `.xlsx` (`exportProjectToExcel` in `parseProject.ts`)
  matching the columns the importer understands.
  A `CalendarProvider` (`src/lib/calendar.tsx`, persisted under
  `mpp-viewer-calendar-v1`) holds **workdays** (default Mon–Fri) and
  **blackout dates** (each `{date, label?}`); the Settings dialog
  exposes both. The Gantt shades non-working days (off-workdays and
  blackouts) with light grey bands, and blackout cells get a tooltip
  with the label.
- `api-server` (express) — Backs the viewer's `.mpp` import via
  `POST /api/mpp/convert`, which streams the binary to MPXJ (Java) and
  returns MSPDI XML. MPXJ runtime lives in
  `artifacts/api-server/vendor/mpxj/` (mpxj.jar + lib/, ~16MB) and requires
  the system `openjdk` package.
- `mockup-sandbox` (design) — component preview server.

## Notes

- Spawning external binaries from the API server requires removing the
  Replit `LD_AUDIT` env var on the child process; otherwise spawn fails
  with `ENOENT`. See `artifacts/api-server/src/routes/mpp.ts`.

See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.
