# Edit Habit Progress Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Let users set a habit's start date and initial progress (total completions + current streak) when adding a habit or at any time via ⋯ → Edit Progress.

**Architecture:** All changes in `index.html`. A new pure function `generateCompletedDates` builds the completedDates array from three inputs. The existing Add Habit modal is expanded with an optional toggle + progress fields. A new `showEditProgress` / `saveEditProgress` flow reuses the same modal in a second mode controlled by a `modalMode` state variable.

**Tech Stack:** Vanilla HTML, CSS, JavaScript. localStorage. No dependencies.

---

## File Map

| File | Changes |
|------|---------|
| `index.html` | New CSS, expanded modal HTML, 4 new JS functions, 2 updated JS functions, 2 new state vars |
| `test.html` | Add `generateCompletedDates` and `validateProgress` with tests |

---

### Task 1: generateCompletedDates() + validateProgress() — TDD

**Files:**
- Modify: `test.html` (add functions + tests)
- Modify: `index.html` (add functions after `getDayColor`)

- [ ] **Step 1: Add generateCompletedDates() and validateProgress() to test.html**

In `test.html`, in the `// ── Pure functions under test ──` section, add after `getDayColor`:

```javascript
    function generateCompletedDates(startDate, totalDone, streak, today = getToday()) {
      if (totalDone === 0) return [];

      const streakDates = [];
      const cur = new Date(today + 'T00:00:00');
      for (let i = 0; i < streak; i++) {
        streakDates.unshift(cur.toLocaleDateString('en-CA'));
        cur.setDate(cur.getDate() - 1);
      }
      if (streak === 0) {
        cur.setDate(cur.getDate() - 1);
      }

      const pool = [];
      const poolCursor = new Date(startDate + 'T00:00:00');
      while (poolCursor <= cur) {
        pool.push(poolCursor.toLocaleDateString('en-CA'));
        poolCursor.setDate(poolCursor.getDate() + 1);
      }

      const extra = pool.slice(0, totalDone - streak);
      return [...extra, ...streakDates].sort();
    }

    function validateProgress(startDate, totalDone, streak, today = getToday()) {
      if (!startDate) return 'Please enter a start date';
      if (startDate > today) return "Start date can't be in the future";
      if (totalDone < 0 || streak < 0) return 'Values must be 0 or greater';
      if (streak > totalDone) return "Streak can't be more than total days done";
      const [sy, sm, sd] = startDate.split('-').map(Number);
      const [ey, em, ed] = today.split('-').map(Number);
      const daysSinceStart = Math.floor(
        (Date.UTC(ey, em - 1, ed) - Date.UTC(sy, sm - 1, sd)) / (1000 * 60 * 60 * 24)
      ) + 1;
      if (totalDone > daysSinceStart) return `Total days done can't exceed days since start (${daysSinceStart})`;
      return null;
    }
```

- [ ] **Step 2: Add tests for both functions to test.html**

In the `// ── Tests ──` section, add after the existing getDayColor tests:

```javascript
    section('generateCompletedDates');

    assert('returns empty array when totalDone=0',
      generateCompletedDates('2026-06-01', 0, 0, '2026-06-10'), []);

    assert('returns streak dates only when totalDone equals streak',
      generateCompletedDates('2026-06-01', 3, 3, '2026-06-10'),
      ['2026-06-08', '2026-06-09', '2026-06-10']);

    assert('result length equals totalDone',
      generateCompletedDates('2026-05-20', 17, 4, '2026-06-10').length, 17);

    assert('last date in result is today when streak > 0',
      generateCompletedDates('2026-05-20', 5, 2, '2026-06-10').slice(-1)[0], '2026-06-10');

    assert('second-to-last date is yesterday when streak >= 2',
      generateCompletedDates('2026-05-20', 5, 2, '2026-06-10').slice(-2)[0], '2026-06-09');

    assert('today not in result when streak = 0',
      generateCompletedDates('2026-06-01', 3, 0, '2026-06-10').includes('2026-06-10'), false);

    assert('result is sorted ascending', (() => {
      const dates = generateCompletedDates('2026-05-20', 10, 3, '2026-06-10');
      return dates.join(',') === [...dates].sort().join(',');
    })(), true);

    section('validateProgress');

    assert('returns null for valid inputs',
      validateProgress('2026-05-20', 17, 4, '2026-06-10'), null);

    assert('returns null when totalDone=0 and streak=0',
      validateProgress('2026-06-01', 0, 0, '2026-06-10'), null);

    assert('returns error for future start date',
      validateProgress('2026-12-31', 0, 0, '2026-06-10'), "Start date can't be in the future");

    assert('returns error when streak > totalDone',
      validateProgress('2026-06-01', 3, 5, '2026-06-10'), "Streak can't be more than total days done");

    assert('returns error when totalDone > days since start',
      validateProgress('2026-06-09', 100, 0, '2026-06-10'), "Total days done can't exceed days since start (2)");

    assert('returns error for negative values',
      validateProgress('2026-06-01', -1, 0, '2026-06-10'), 'Values must be 0 or greater');

    assert('returns error for empty startDate',
      validateProgress('', 0, 0, '2026-06-10'), 'Please enter a start date');
```

- [ ] **Step 3: Open test.html and confirm all new tests pass**

Open `test.html` in your browser. You should see 14 new green ✓ lines (7 + 7). Total should now read "43 passed, 0 failed".

- [ ] **Step 4: Copy both functions into index.html after getDayColor**

In `index.html`, find `function getDayColor` and add both functions immediately after its closing `}`:

```javascript
    function generateCompletedDates(startDate, totalDone, streak, today = getToday()) {
      if (totalDone === 0) return [];

      const streakDates = [];
      const cur = new Date(today + 'T00:00:00');
      for (let i = 0; i < streak; i++) {
        streakDates.unshift(cur.toLocaleDateString('en-CA'));
        cur.setDate(cur.getDate() - 1);
      }
      if (streak === 0) {
        cur.setDate(cur.getDate() - 1);
      }

      const pool = [];
      const poolCursor = new Date(startDate + 'T00:00:00');
      while (poolCursor <= cur) {
        pool.push(poolCursor.toLocaleDateString('en-CA'));
        poolCursor.setDate(poolCursor.getDate() + 1);
      }

      const extra = pool.slice(0, totalDone - streak);
      return [...extra, ...streakDates].sort();
    }

    function validateProgress(startDate, totalDone, streak, today = getToday()) {
      if (!startDate) return 'Please enter a start date';
      if (startDate > today) return "Start date can't be in the future";
      if (totalDone < 0 || streak < 0) return 'Values must be 0 or greater';
      if (streak > totalDone) return "Streak can't be more than total days done";
      const [sy, sm, sd] = startDate.split('-').map(Number);
      const [ey, em, ed] = today.split('-').map(Number);
      const daysSinceStart = Math.floor(
        (Date.UTC(ey, em - 1, ed) - Date.UTC(sy, sm - 1, sd)) / (1000 * 60 * 60 * 24)
      ) + 1;
      if (totalDone > daysSinceStart) return `Total days done can't exceed days since start (${daysSinceStart})`;
      return null;
    }
```

- [ ] **Step 5: Commit**

```bash
git add test.html index.html
git commit -m "feat: add generateCompletedDates and validateProgress"
```

---

### Task 2: CSS for new modal fields

**Files:**
- Modify: `index.html` (inside `<style>` block, before `</style>`)

- [ ] **Step 1: Add CSS at the end of the `<style>` block, before `</style>`**

```css
    /* ── Progress modal fields ─────────────────────────────────── */
    .progress-toggle-label {
      display: flex;
      align-items: center;
      gap: 8px;
      color: #94a3b8;
      font-size: 13px;
      cursor: pointer;
      margin-bottom: 16px;
    }

    .progress-toggle-label input[type="checkbox"] {
      width: 16px;
      height: 16px;
      accent-color: #4ade80;
      cursor: pointer;
      flex-shrink: 0;
    }

    .field-label {
      display: block;
      font-size: 11px;
      color: #64748b;
      text-transform: uppercase;
      letter-spacing: 0.5px;
      margin-bottom: 4px;
    }

    #progress-fields .modal-input {
      margin-bottom: 12px;
    }

    #progress-error {
      color: #f87171;
      font-size: 13px;
      margin-bottom: 12px;
    }
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add CSS for progress modal fields"
```

---

### Task 3: Expand modal HTML + state vars + modal JS functions

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace the entire modal HTML block**

Find and replace everything from `<!-- Add Habit Modal -->` through the closing `</div>` of the modal (lines ~542–558):

```html
  <!-- Add Habit / Edit Progress Modal -->
  <div id="modal" onclick="if(event.target===this) hideModal()">
    <div class="modal-box">
      <h2 id="modal-title">New Habit</h2>
      <div id="name-field">
        <input
          class="modal-input"
          id="habit-input"
          type="text"
          placeholder="e.g. Drink Water"
          onkeydown="if(event.key==='Enter') saveModal()"
        >
      </div>
      <div id="toggle-field">
        <label class="progress-toggle-label">
          <input type="checkbox" id="progress-toggle" onchange="toggleProgressFields()">
          <span>Already tracking this habit?</span>
        </label>
      </div>
      <div id="progress-fields" style="display:none">
        <label class="field-label">Start date</label>
        <input class="modal-input" id="start-date-input" type="date">
        <label class="field-label">Total days done</label>
        <input class="modal-input" id="total-done-input" type="number" min="0" placeholder="0">
        <label class="field-label">Current streak</label>
        <input class="modal-input" id="streak-input" type="number" min="0" placeholder="0">
      </div>
      <div id="progress-error" style="display:none"></div>
      <div class="modal-actions">
        <button class="btn-secondary" onclick="hideModal()">Cancel</button>
        <button class="btn-primary" id="modal-save-btn" onclick="saveModal()">Save</button>
      </div>
    </div>
  </div>
```

- [ ] **Step 2: Add two new state variables after the existing three at the top of the script block**

Find:
```javascript
    let currentView = 'today';
    let currentMonth = new Date().getMonth();
    let currentYear = new Date().getFullYear();
```

Replace with:
```javascript
    let currentView = 'today';
    let currentMonth = new Date().getMonth();
    let currentYear = new Date().getFullYear();
    let modalMode = 'add';
    let editingHabitId = null;
```

- [ ] **Step 3: Replace showModal() and hideModal() with updated versions, and add toggleProgressFields() and saveModal()**

Find and replace the existing `showModal()` and `hideModal()` functions:

```javascript
    function showModal() {
      modalMode = 'add';
      editingHabitId = null;
      document.getElementById('modal-title').textContent = 'New Habit';
      document.getElementById('name-field').style.display = 'block';
      document.getElementById('toggle-field').style.display = 'block';
      document.getElementById('progress-toggle').checked = false;
      document.getElementById('progress-fields').style.display = 'none';
      document.getElementById('progress-error').style.display = 'none';
      document.getElementById('habit-input').value = '';
      document.getElementById('start-date-input').value = '';
      document.getElementById('total-done-input').value = '';
      document.getElementById('streak-input').value = '';
      document.getElementById('modal-save-btn').textContent = 'Save';
      document.getElementById('modal').style.display = 'flex';
      setTimeout(() => document.getElementById('habit-input').focus(), 50);
    }

    function hideModal() {
      document.getElementById('modal').style.display = 'none';
    }

    function toggleProgressFields() {
      const checked = document.getElementById('progress-toggle').checked;
      document.getElementById('progress-fields').style.display = checked ? 'block' : 'none';
      if (checked && !document.getElementById('start-date-input').value) {
        document.getElementById('start-date-input').value = getToday();
      }
    }

    function saveModal() {
      if (modalMode === 'add') {
        addHabit();
      } else {
        saveEditProgress();
      }
    }
```

- [ ] **Step 4: Replace addHabit() with the updated version**

Find and replace the existing `addHabit()` function:

```javascript
    function addHabit() {
      const input = document.getElementById('habit-input');
      const name = input.value.trim();
      if (!name) return;

      const habits = loadHabits();
      const today = getToday();
      let createdDate = today;
      let completedDates = [];

      if (document.getElementById('progress-toggle').checked) {
        const startDate = document.getElementById('start-date-input').value;
        const totalDone = parseInt(document.getElementById('total-done-input').value) || 0;
        const streak = parseInt(document.getElementById('streak-input').value) || 0;

        const error = validateProgress(startDate, totalDone, streak, today);
        if (error) {
          const errEl = document.getElementById('progress-error');
          errEl.textContent = error;
          errEl.style.display = 'block';
          return;
        }

        createdDate = startDate;
        completedDates = generateCompletedDates(startDate, totalDone, streak, today);
      }

      habits.push({
        id: Date.now().toString(36),
        name,
        createdDate,
        completedDates
      });

      saveHabits(habits);
      hideModal();
      renderCurrentView();
    }
```

- [ ] **Step 5: Test the updated Add Habit modal**

Open `index.html`. Click **+ Add Habit**:
- Modal should show with just the name field and the toggle
- Check the toggle — start date, total days done, and current streak fields should appear
- Start date should auto-fill to today
- Uncheck toggle — fields should disappear
- Type a habit name, leave toggle unchecked, click Save — habit appears with today as start date and 0/1 stats
- Open another habit, check toggle, enter a past date / some completions / a streak, click Save — stats reflect your inputs
- No console errors

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: expand add habit modal with optional progress fields"
```

---

### Task 4: Edit Progress in ⋯ menu + showEditProgress / saveEditProgress

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add "Edit Progress" to the ⋯ menu in render()**

Find this block inside `render()`:
```javascript
              <div class="menu" id="menu-${habit.id}">
                <button onclick="startRename('${habit.id}')">Rename</button>
                <button onclick="deleteHabit('${habit.id}')">Delete</button>
              </div>
```

Replace with:
```javascript
              <div class="menu" id="menu-${habit.id}">
                <button onclick="startRename('${habit.id}')">Rename</button>
                <button onclick="showEditProgress('${habit.id}')">Edit Progress</button>
                <button onclick="deleteHabit('${habit.id}')">Delete</button>
              </div>
```

- [ ] **Step 2: Add showEditProgress() and saveEditProgress() after startRename()**

Find the closing `}` of `startRename()` and add immediately after it:

```javascript
    function showEditProgress(id) {
      const habits = loadHabits();
      const habit = habits.find(h => h.id === id);
      if (!habit) return;

      modalMode = 'edit-progress';
      editingHabitId = id;
      document.getElementById('modal-title').textContent = 'Edit Progress';
      document.getElementById('name-field').style.display = 'none';
      document.getElementById('toggle-field').style.display = 'none';
      document.getElementById('progress-fields').style.display = 'block';
      document.getElementById('progress-error').style.display = 'none';
      document.getElementById('start-date-input').value = habit.createdDate;
      document.getElementById('total-done-input').value = habit.completedDates.length;
      document.getElementById('streak-input').value = calcStreak(habit);
      document.getElementById('modal-save-btn').textContent = 'Save Progress';
      document.getElementById('modal').style.display = 'flex';
      setTimeout(() => document.getElementById('start-date-input').focus(), 50);
    }

    function saveEditProgress() {
      const today = getToday();
      const startDate = document.getElementById('start-date-input').value;
      const totalDone = parseInt(document.getElementById('total-done-input').value) || 0;
      const streak = parseInt(document.getElementById('streak-input').value) || 0;

      const error = validateProgress(startDate, totalDone, streak, today);
      if (error) {
        const errEl = document.getElementById('progress-error');
        errEl.textContent = error;
        errEl.style.display = 'block';
        return;
      }

      const habits = loadHabits();
      const habit = habits.find(h => h.id === editingHabitId);
      if (!habit) return;

      habit.createdDate = startDate;
      habit.completedDates = generateCompletedDates(startDate, totalDone, streak, today);
      saveHabits(habits);
      hideModal();
      renderCurrentView();
    }
```

- [ ] **Step 3: Test Edit Progress from the ⋯ menu**

Open `index.html`. Add a habit if there aren't any. Click ⋯ on a habit — you should see **Rename**, **Edit Progress**, **Delete**. Click Edit Progress:
- Modal should open titled "Edit Progress"
- Name field should be hidden
- Progress fields should be visible, pre-filled with the habit's current start date, total completions, and streak
- Change the start date to a month ago, set total done to 15, streak to 5, click Save Progress
- Habit's stats should now show 15/~30 overall and 🔥 5 streak
- No console errors

- [ ] **Step 4: Test validation errors**

Click ⋯ → Edit Progress. Try:
- Set start date to a future date → should show "Start date can't be in the future"
- Set streak to 10, total to 3 → should show "Streak can't be more than total days done"
- Set start date to yesterday, total to 100 → should show error about days since start

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add edit progress to habit menu"
```

---

### Task 5: Final checks and push

**Files:**
- None (verification + push only)

- [ ] **Step 1: Full walkthrough**

Open `index.html` and verify:
1. Add a new habit without touching the toggle — works exactly as before
2. Add a habit with toggle on — enter past start date, some completions, a streak — stats are correct
3. Edit progress from ⋯ menu — pre-fills correctly, saves correctly
4. Validation errors show and clear when you fix the inputs
5. Today view, Week view, Month view all reflect updated stats correctly after edits

- [ ] **Step 2: Run tests**

Open `test.html` — confirm "43 passed, 0 failed".

- [ ] **Step 3: Push to GitHub**

```bash
git push
```
