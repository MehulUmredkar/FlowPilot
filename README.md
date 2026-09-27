# FlowPilot — Project Guide

A small business workflow & task management platform, built as three standalone HTML files. No build tools, no backend — everything runs directly in the browser.

## Files

| File | What it is |
|---|---|
| `flowpilot-login.html` | Sign in / create account page |
| `flowpilot-landing.html` | Marketing landing page |
| `flowpilot-app.html` | The actual workspace (dashboard, board, tasks, team) |

Each file is fully self-contained: HTML + CSS + JavaScript in one document. Open any of them directly in a browser — nothing to install.

## Design system

All three pages share one visual language, defined as CSS custom properties (variables) at the top of each `<style>` block:

```css
--bg: #f8fafc        /* page background */
--surface: #ffffff   /* card/panel background */
--surface2: #eef0fb  /* subtle fill, e.g. progress track */
--border: #e2e8f0    /* hairline borders */
--text: #0f172a      /* body text */
--muted: #64748b     /* secondary text */
--accent: #4f46e5    /* indigo — buttons, links, active states */
```

A `prefers-color-scheme: dark` media query (plus a `data-theme="dark"` override) swaps these to a dark navy palette with `--accent: #818cf8`.

Typography is **Inter** (falls back to system sans-serif). Buttons and inputs use ~10px border-radius; cards use ~12px.

**Status colors** are consistent everywhere: green = done/on track, orange = in progress/at risk, red = overdue/off track, gray = to do/not started.

## How the workspace app works (`flowpilot-app.html`)

### Data model

Everything lives in one JavaScript object called `state`:

```js
state = {
  members: [{ id, name }, ...],
  tasks: [{ id, title, assignee, deadline, priority, status }, ...],
  nextTaskId, nextMemberId   // counters for new IDs
}
```

- `priority` is `"high" | "medium" | "low"`
- `status` is `"todo" | "in-progress" | "done"`
- `deadline` is a plain date string like `"2026-10-02"`

### Persistence

`state` is saved to the browser's `localStorage` under the key `"taskflow-data-v1"` every time something changes, and reloaded from there on page load. This means:
- Data survives page refreshes and browser restarts
- Data is **per-browser, per-device** — it does not sync between people or devices
- Clearing browser storage wipes it (there's no server to fall back on)

### Tabs / views

A single-page layout with four `<section>` elements, toggled by `nav button` clicks (`dashboard`, `board`, `tasks`, `team`). Only one is visible at a time via a `.active` class.

1. **Dashboard** — stat cards (total/completed/overdue/team size), a "Portfolio health" panel (off track / at risk / on track counts + a segmented progress bar), a status breakdown, and per-person completion bars.
2. **Board** — tasks grouped into colored sections by status (Active/Planning/Completed), each task shown as a row with a colored status pill and an avatar circle.
3. **Tasks** — the create-task form and a full table with inline status editing and delete.
4. **Team** — add team members; each shows their task count and completion rate.

### Rendering pattern

There's no framework — each view has a `render___()` function (`renderDashboard`, `renderBoard`, `renderTasks`, `renderTeam`) that rebuilds its section's HTML from the current `state`. Any action that changes data (add task, change status, delete, add member) calls `save()` then re-runs all four render functions, so every view stays in sync automatically.

### Animation touches

- New table rows fade/slide in (staggered slightly per row)
- Deleted rows fade out before being removed from `state`
- Dashboard numbers count up from 0 instead of appearing instantly
- Progress bars animate their width in on render
- Toast notifications (bottom-right) confirm actions like "Task added"
- A `prefers-reduced-motion` media query disables all of this for users who've asked for less motion

## How the login page works (`flowpilot-login.html`)

A single form UI that toggles between "Sign up" and "Log in" modes via a `data-view` attribute on the root `<div id="app">`. CSS rules like `[data-view="login"] .signup-only { display: none; }` show/hide the right fields and copy. A small password-strength meter checks length/case/digits/symbols and colors four bar segments accordingly. No real authentication happens — the form doesn't submit anywhere yet.

## How the landing page works (`flowpilot-landing.html`)

Static marketing content: hero, an 8-item feature grid (mirrors the app's real features), a 3-step "how it works," a stats row, and a closing call-to-action. No JavaScript logic beyond smooth-scroll anchor links.

## What's NOT connected yet

These three pages currently work **independently** — there's no navigation wiring between them:
- The login page's "Create account" / "Log in" buttons don't go anywhere
- The landing page's "Start free" button doesn't open the login or app page
- There's no real authentication, so the app is open to anyone who opens the file

## Extending the project

When asking for new features, it helps to say which file/view you mean (e.g. "in the Tasks view..." or "on the login page..."). Common next steps people ask for:
- Wire the pages together (landing → login → app)
- Add due-date reminders or a calendar view
- Add comments/notes on individual tasks
- Add priority- or quarter-based grouping (seen in the monday.com-style references)
- Move from localStorage to a real backend so data syncs across devices/users
