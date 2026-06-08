# Habit Tracker — Design Spec
Date: 2026-06-08

## Overview

A single-file web app (`index.html`) for tracking daily habits with streak counting and overall completion stats. No server, no dependencies — opens directly in a browser and stores all data in `localStorage`.

---

## Architecture

**Single file:** `index.html`

Everything lives in one file:
- HTML structure at the top
- `<style>` block for all CSS
- `<script>` block at the bottom for all JS logic

No build tools, no npm, no server required. The user opens `index.html` directly in their browser.

---

## Data Model

All data is stored under the key `"habits"` in `localStorage` as a JSON array.

```json
[
  {
    "id": "abc123",
    "name": "Drink Water",
    "createdDate": "2026-05-18",
    "completedDates": ["2026-05-18", "2026-05-19", "2026-05-20"]
  }
]
```

- `id` — unique identifier (random string), used for safe rename and delete
- `name` — habit label entered by the user
- `createdDate` — ISO date string (YYYY-MM-DD) set when the habit is first added; used to calculate "days since started"
- `completedDates` — array of ISO date strings, one entry per day the habit was completed

Streak and overall counts are computed from `completedDates` at render time — they are not stored.

---

## UI Components

### Header
- App title ("Habit Tracker") and today's date
- "+ Add Habit" button (top right)

### Habit List
Each habit is a row showing:
- Checkbox (left) — click to toggle today's completion
- Habit name
- 🔥 Streak counter — current days-in-a-row count (orange)
- Overall count — `completedDates.length / daysSinceCreated` (blue, e.g. `17/21`)
- ⋯ Menu button — opens rename / delete options
- Green left border when completed today; grey when not

### Add Habit Modal
- Text input for habit name
- Save button — adds habit and closes modal
- Cancel button — closes modal without saving

### Footer
- Summary line: "X of Y habits done today"

---

## Logic

### Streak Calculation
Walk backwards from today:
1. If today's date is in `completedDates`, count = 1, move to yesterday
2. For each prior day, if it's in `completedDates` continue counting; if not, stop
3. If today is not completed, start from yesterday using the same logic (streak is not broken until the day resets at midnight)

### Overall Calculation
- **Completed:** `completedDates.length`
- **Possible:** number of days from `createdDate` up to and including today

### Toggle Completion
- If today's date is already in `completedDates`, remove it (uncheck)
- If not, add it (check)
- Save updated habits array to `localStorage`
- Re-render the list

### Add Habit
- Generate a random `id` (e.g. `Date.now().toString(36)`)
- Set `createdDate` to today
- Push to habits array, save to `localStorage`, re-render

### Delete Habit
- Filter habit with matching `id` out of the array
- Save to `localStorage`, re-render

### Rename Habit
- Find habit by `id`, update `name`
- Save to `localStorage`, re-render

---

## Edge Cases

- **Uncheck same day:** removing today's date from `completedDates` correctly drops the streak
- **Timezone:** uses the device's local date via `new Date().toLocaleDateString('en-CA')` (returns YYYY-MM-DD) — no server timezone issues
- **Empty state:** if no habits exist, show a prompt encouraging the user to add their first habit
- **Duplicate names:** allowed — `id` is the unique key, not `name`

---

## Error Handling

- `localStorage` parse errors: wrap in try/catch, fall back to empty array
- No other external dependencies or network calls — failure surface is minimal

---

## Out of Scope

- Reminders / notifications
- Cloud sync or multi-device support
- Charts or calendar heatmap views
- Authentication
