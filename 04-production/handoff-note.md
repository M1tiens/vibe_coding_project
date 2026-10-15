# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

**The Dashboard Nobody Reads** is a clickable retention prototype for a personal-trainer marketplace. It tests one hypothesis: a first screen that leads with one headline metric ("clients at risk of lapsing this week"), a plain-language takeaway, and one recommended action (one-tap check-in and rebooking) will make trainers act instead of bounce. Four screens cover client check-ins, weekly availability, daily bookings, and experiment evidence. Code is grouped by PRD screen, with data and interaction logic separated from display. **Everything is simulated**: no real messages are sent, no data is persisted, and experiment outcomes are illustrative, not measured.

## Architecture (plain language)

- **Frontend:** **Framework:** TanStack Start v1 (React 19, Vite 7), Tailwind CSS v4 with semantic tokens in `src/styles.css` (green health-and-sports palette, DM Sans + Manrope). - **Shell:** `src/features/workspace/global-shell.tsx` renders the sidebar (Overview, My calendar, Day bookings, The experiment), trainer profile and rating (Alex Carter, 4.9 / 128 reviews), and a demo label. - **Screens** (`src/features/<screen>/`):   | URL | Screen | Purpose |   |---|---|---|   | `/` | Action-First Retention Overview | Headline at-risk count, takeaway, one batch check-in action, per-client cards with check-in dialog |   | `/calendar` | Calendar Availability Management | Weekly hours, add time block (end > start validation), remove blocks |   | `/bookings` | Daily Bookings Operational View | Day stats, Accept / Cancel / Restore per booking |   | `/experiment` | Hypothesis Experiment Tracker | Baseline evidence (60% bounce, 12 charts, 6 clicks), user quotes, scenario switch, live click counter, kill switch ("Pause & pivot") | - **Per-feature pattern:** `data.ts` (fixtures, types, query options) → `use-<feature>.ts` (query + interaction state) → `<feature>.tsx` (presentational, takes a `model` prop). Route files in `src/routes/` are thin adapters (search params, head metadata).
- **Backend / data:** _____
- **Key flows:** 1. **Check-in:** Overview → "Check in with 5 clients" (or single client) → dialog with personalized message → cards marked "Check-in sent" → click counter increments on `/experiment`.
2. **Availability:** Calendar → add block → appears in day grid → remove.
3. **Bookings:** Day bookings → Accept pending / Cancel / Restore → stats update.
4. **Kill switch:** Experiment → "Still bouncing" scenario → "Pause & pivot" → "Experiment paused. Revisit the metric."
5. **States:** every data screen shows a skeleton for 1s; empty states ("No data available" / "Add new times for your availability." / "No bookings yet"); error state "Please try again later" with retry.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| - Feature-based structure with clean data/display separation. | solid | _____ |
| - Design tokens; no hard-coded colors. | solid | _____ |

## Risks & assumptions for the team

- **Metric validity:** assumes "at risk of lapsing" (no booking scheduled + declining momentum) is the right signal. The kill switch exists because it may not be.
- **Measurement:** bounce (<15s, no interaction) and action click-through need real analytics instrumentation before any conclusion.
- **Messaging:** real check-ins need consent, opt-out, rate limits, and a delivery provider.
- **Data:** risk scoring needs real session history; calendar and bookings need conflict handling and time zones.
- **Demo leakage:** simulated labels must stay visible so stakeholders don't read illustrative numbers as results.

## How to run it

```
un install
bun run dev        # http://localhost:8080
bunx vitest run    # tests
```

- Append `?state=empty` or `?state=error` on `/`, `/calendar`, or `/bookings` to view those states.
- Reload to reset demo changes.
- Keep the existing Vite config (framework plugins + Cloudflare-target build). Do not hand-edit `src/routeTree.gen.ts`.
```
