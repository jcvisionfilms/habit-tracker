# Weekly & Monthly Views — Design Spec
Date: 2026-06-09

## Overview

Add a Weekly view and a Monthly view to the existing single-file habit tracker (`index.html`). Navigation between views uses three tabs ("Today | Week | Month") inserted between the header and the habit list. No data model changes required — all views read from the existing `completedDates` arrays in localStorage.

---

## Architecture

All changes are confined to `index.html`. No new files are created.

**New state variables** (added to the top of the script block):
- `let currentView = 'today'` — tracks which tab is active
- `let currentMonth` — the month number (0–11) shown in the monthly view (defaults to current month)
- `let currentYear` — the year shown in the monthly view (defaults to current year)

**New HTML** — a tab bar inserted between `.divider` and `#habit-list`:
```html
<div class="tab-bar">
  <button class="tab active" onclick="switchView('today')">Today</button>
  <button class="tab" onclick="switchView('week')">Week</button>
  <button class="tab" onclick="switchView('month')">Month</button>
</div>
```

**New functions** added to the script block:
- `switchView(view)` — sets `currentView`, updates active tab styling, calls `renderCurrentView()`
- `renderCurrentView()` — dispatcher: calls `render()`, `renderWeekView()`, or `renderMonthView()` based on `currentView`
- `renderWeekView()` — renders the weekly grid
- `renderMonthView()` — renders the monthly calendar
- `prevMonth()` / `nextMonth()` — decrement/increment `currentMonth`/`currentYear`, re-render

The existing `render()` call at the bottom of the script is replaced with `renderCurrentView()`.

---

## Weekly View

Renders into `#habit-list`. Each habit gets one row containing:
- Habit name (truncated with ellipsis if too long)
- 7 day squares (Mon–Sun of the current week): green (`#4ade80`) if completed, grey (`#374151`) if not
- Weekly completion percentage and fraction (e.g. `86% / 6/7`) in green if ≥ 50%, orange if < 50%

**Header label** (`#today-label`) shows: `"Week of [Mon date] – [Sun date], [Year]"`

**Footer** shows: `"Week average: X% across all habits"`

**Week definition:** Monday = start of week. Current week is calculated from today's date by finding the most recent Monday.

**Day squares** — a square is green if the ISO date string for that day (YYYY-MM-DD) is present in `habit.completedDates`.

---

## Monthly View

Renders into `#habit-list`. Displays a full calendar grid for `currentMonth`/`currentYear`.

**Header label** (`#today-label`) shows: `"[Month Name] [Year]"` (e.g. `"June 2026"`)

**Calendar structure:**
- Day-of-week header row: M T W T F S S
- Grid of day cells, starting on the correct weekday (Monday-based), with empty cells for days before the 1st
- Each day cell shows the day number and is colored based on completion ratio:
  - **Grey** (`#374151`, dim text) — 0 habits completed (or no habits exist)
  - **Yellow** (`#facc15`, black text) — 1 or more habits completed, but not all
  - **Green** (`#4ade80`, black text) — all habits completed
  - **Blue border** (`#60a5fa`) — today's date (color fill still applies)
- Future days (after today) are rendered grey with dim text and no completion coloring

**Month navigation:** `‹` and `›` arrow buttons call `prevMonth()` and `nextMonth()`. Cannot navigate to a future month beyond the current month.

**Legend** below the grid:
- Green = All done · Yellow = Some done · Grey = None done · Blue border = Today

---

## Color Reference

| State | Color | Hex |
|-------|-------|-----|
| All habits done | Green | `#4ade80` |
| Some habits done | Yellow | `#facc15` |
| No habits done | Grey | `#374151` |
| Today indicator | Blue border | `#60a5fa` |
| Weekly % (good) | Green | `#4ade80` |
| Weekly % (low) | Orange | `#f97316` |

---

## CSS additions

New classes needed:
- `.tab-bar` — flex container, dark background, rounded, padding
- `.tab` — flex button, transparent background, grey text
- `.tab.active` — highlighted background (`#334155`), white text, bold
- `.week-row` — habit row layout for weekly view (name + squares + %)
- `.day-sq` — individual day square in both views
- `.cal-grid` — CSS grid, 7 columns, gap
- `.cal-day` — individual calendar day cell
- `.cal-day.all` — green background
- `.cal-day.some` — yellow background
- `.cal-day.none` — grey background (default)
- `.cal-day.today` — blue border
- `.cal-day.future` — grey, dimmed
- `.month-nav` — flex row with ‹ month name › layout
- `.week-legend` / `.cal-legend` — legend rows

---

## Out of Scope

- Clicking a calendar day to jump to that day's habits
- Editing or checking off habits from the weekly/monthly views (read-only)
- Exporting data
- Notifications or reminders
