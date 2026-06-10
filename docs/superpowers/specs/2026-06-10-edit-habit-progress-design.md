# Edit Habit Progress — Design Spec
Date: 2026-06-10

## Overview

Allow users to manually set a habit's start date and initial progress (total completions + current streak). This covers the use case where someone was already tracking a habit before they started using the app and wants to enter where they currently are.

The feature is accessible in two places:
1. **When adding a new habit** — an optional toggle in the Add Habit modal reveals progress fields
2. **Any time after** — via ⋯ → Edit Progress on any existing habit

---

## UI

### Add Habit Modal (expanded)

The existing modal gains a toggle below the name field. The toggle is off by default so the normal add-habit flow is unchanged.

**Toggle off (default):**
```
[ Habit name input                    ]

☐  Already tracking this habit?

[ Cancel ]  [ Save ]
```

**Toggle on:**
```
[ Habit name input                    ]

☑  Already tracking this habit?

  Start date:         [ date input   ]
  Total days done:    [ number input ]
  Current streak:     [ number input ]

[ Cancel ]  [ Save ]
```

- Start date input defaults to today when the toggle is first turned on
- Total days done and current streak default to 0
- The extra fields are hidden (CSS `display:none`) when toggle is off

### ⋯ Menu

The per-habit dropdown gains a third option between Rename and Delete:

```
Rename
Edit Progress
Delete
```

### Edit Progress Modal (same modal, different mode)

Clicking "Edit Progress" opens the same modal HTML with:
- Name field hidden
- Toggle hidden (progress fields always visible)
- Fields pre-filled with the habit's current values:
  - Start date: `habit.createdDate`
  - Total days done: `habit.completedDates.length`
  - Current streak: calculated via `calcStreak(habit)`
- Save button labeled "Save Progress"

---

## Data Logic

### generateCompletedDates(startDate, totalDone, streak, today)

Pure function. Returns an array of YYYY-MM-DD date strings.

**Algorithm:**
1. Build the streak dates: the last `streak` days ending on `today` (inclusive)
2. Build a list of all available dates from `startDate` to `(today - streak - 1)` exclusive of the streak window
3. Take the first `(totalDone - streak)` dates from the available list
4. Return streak dates + remaining dates, sorted ascending

**Example:** `startDate='2026-05-20', totalDone=17, streak=4, today='2026-06-09'`
- Streak dates: `['2026-06-06', '2026-06-07', '2026-06-08', '2026-06-09']`
- Available pool: May 20 – June 5 (17 days available)
- Take first 13: May 20–June 1 (13 dates)
- Result: 13 early dates + 4 streak dates = 17 total → shows `17/21` overall, `🔥 4` streak

### Validation (checked on Save)

Errors shown inline in the modal (below the fields, in red):

| Condition | Error message |
|-----------|---------------|
| Start date is in the future | "Start date can't be in the future" |
| Streak > total days done | "Streak can't be more than total days done" |
| Total days done > days since start | "Total days done can't exceed days since start date" |
| Total days done < 0 or streak < 0 | "Values must be 0 or greater" |

If there are no errors, save proceeds.

### Saving

**Add Habit (toggle on):**
- `createdDate` = start date input value
- `completedDates` = `generateCompletedDates(startDate, totalDone, streak, today)`

**Add Habit (toggle off, default):**
- `createdDate` = today (unchanged from current behavior)
- `completedDates` = `[]` (unchanged)

**Edit Progress:**
- `createdDate` = start date input value
- `completedDates` = `generateCompletedDates(startDate, totalDone, streak, today)`
- `id` and `name` are not changed

---

## Architecture

All changes in `index.html` only. No new files.

### HTML changes
- Modal: add toggle checkbox + hidden progress fields (`#progress-toggle`, `#progress-fields`, `#start-date-input`, `#total-done-input`, `#streak-input`, `#progress-error`)
- Modal: add hidden name-field wrapper (`#name-field`) so it can be hidden in edit-progress mode
- Modal: save button gets id `#modal-save-btn` so its label can be changed
- `render()`: add "Edit Progress" button to each habit's ⋯ menu

### New pure function
- `generateCompletedDates(startDate, totalDone, streak, today)` — added after `getDayColor` in the pure utils section, tested in `test.html`

### Updated functions
- `showModal()` — reset toggle to off, hide progress fields, reset name field, restore save button label
- `addHabit()` — if toggle is on, validate and call `generateCompletedDates`; if off, behave as today

### New functions
- `toggleProgressFields()` — shows/hides `#progress-fields` based on toggle state, sets start date default to today
- `showEditProgress(id)` — hides name field, shows progress fields pre-filled, sets save button label to "Save Progress", stores editing id in a variable
- `saveEditProgress()` — reads fields, validates, calls `generateCompletedDates`, updates habit, saves, re-renders
- `saveModal()` — new dispatcher called by the Save button: calls `addHabit()` or `saveEditProgress()` depending on modal mode

### State
- `let modalMode = 'add'` — tracks whether modal is in 'add' or 'edit-progress' mode
- `let editingHabitId = null` — stores the id of the habit being edited

---

## Out of Scope
- Editing individual past completion dates (checking off specific days)
- Importing data from other habit tracking apps
- Bulk editing multiple habits at once
