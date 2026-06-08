# Habit Tracker Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single `index.html` habit tracker that stores data in localStorage, supports adding/renaming/deleting habits, and displays a streak counter and overall completion count per habit.

**Architecture:** Everything lives in one file (`index.html`) — HTML structure, a `<style>` block for CSS, and a `<script>` block for all JS logic. Pure functions (date utils, streak calc, overall calc) are written and tested in `test.html` first, then copied into `index.html`. DOM functions are implemented directly in `index.html`.

**Tech Stack:** Vanilla HTML, CSS, JavaScript (ES modules not used — single file with inline script). localStorage for persistence. No npm, no build tools.

---

## File Map

| File | Purpose |
|------|---------|
| `index.html` | The entire app — HTML, CSS, JS |
| `test.html` | Browser-based test runner for pure functions |

---

### Task 1: Scaffold index.html

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create index.html with the base skeleton**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Habit Tracker</title>
  <style>
    /* styles go here in Task 11 */
  </style>
</head>
<body>
  <div class="container">
    <header>
      <div>
        <h1>Habit Tracker</h1>
        <div class="date" id="today-label"></div>
      </div>
      <button class="add-btn" onclick="showModal()">+ Add Habit</button>
    </header>

    <div class="divider"></div>

    <div id="habit-list"></div>
    <div class="footer" id="footer"></div>
  </div>

  <!-- Add Habit Modal -->
  <div id="modal">
    <div class="modal-box">
      <h2>New Habit</h2>
      <input
        class="modal-input"
        id="habit-input"
        type="text"
        placeholder="e.g. Drink Water"
        onkeydown="if(event.key==='Enter') addHabit()"
      >
      <div class="modal-actions">
        <button class="btn-secondary" onclick="hideModal()">Cancel</button>
        <button class="btn-primary" onclick="addHabit()">Save</button>
      </div>
    </div>
  </div>

  <script>
    // pure utils — added in Tasks 3–6
    // DOM functions — added in Tasks 7–10
  </script>
</body>
</html>
```

- [ ] **Step 2: Open index.html in your browser and confirm it renders without errors**

Open `index.html` by double-clicking it in Finder. You should see a dark (unstyled) page with "Habit Tracker" text and a "+ Add Habit" button.
Open the browser console (Cmd+Option+J) — there should be no red errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: scaffold index.html structure"
```

---

### Task 2: Create test.html with assertion framework

**Files:**
- Create: `test.html`

- [ ] **Step 1: Create test.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Habit Tracker Tests</title>
  <style>
    body { font-family: monospace; background: #0f172a; color: #f1f5f9; padding: 24px; }
    h1 { margin-bottom: 16px; }
    pre { line-height: 1.7; font-size: 14px; }
    .pass { color: #4ade80; }
    .fail { color: #f87171; }
    .summary { margin-top: 16px; font-size: 15px; font-weight: bold; }
  </style>
</head>
<body>
  <h1>Habit Tracker Tests</h1>
  <pre id="output"></pre>
  <div class="summary" id="summary"></div>

  <script>
    let passed = 0;
    let failed = 0;
    const out = document.getElementById('output');

    function assert(description, actual, expected) {
      const ok = JSON.stringify(actual) === JSON.stringify(expected);
      const span = document.createElement('span');
      span.className = ok ? 'pass' : 'fail';
      span.textContent = ok
        ? `✓ ${description}\n`
        : `✗ ${description}\n  Expected: ${JSON.stringify(expected)}\n  Got:      ${JSON.stringify(actual)}\n`;
      out.appendChild(span);
      ok ? passed++ : failed++;
    }

    function section(name) {
      const el = document.createElement('span');
      el.textContent = `\n— ${name} —\n`;
      el.style.color = '#94a3b8';
      out.appendChild(el);
    }

    // ── Pure functions under test ────────────────────────────────────────────
    // Paste functions here as you write them in Tasks 3–6


    // ── Tests ────────────────────────────────────────────────────────────────
    // Tests added in Tasks 3–6


    // ── Summary ──────────────────────────────────────────────────────────────
    document.getElementById('summary').textContent =
      `${passed} passed, ${failed} failed`;
    document.getElementById('summary').style.color = failed ? '#f87171' : '#4ade80';
  </script>
</body>
</html>
```

- [ ] **Step 2: Open test.html in your browser**

Double-click `test.html` in Finder. You should see "Habit Tracker Tests" with "0 passed, 0 failed" at the bottom. No errors in the console.

- [ ] **Step 3: Commit**

```bash
git add test.html
git commit -m "feat: add test runner"
```

---

### Task 3: Date utility (TDD)

**Files:**
- Modify: `test.html` (add function + tests)
- Modify: `index.html` (add function to script block)

- [ ] **Step 1: Add the failing test to test.html**

In `test.html`, inside the `// ── Pure functions under test ──` section, add:

```javascript
function getToday() {
  return new Date().toLocaleDateString('en-CA');
}
```

In the `// ── Tests ──` section, add:

```javascript
section('getToday');
const today = getToday();
assert('returns a string', typeof today, 'string');
assert('format is YYYY-MM-DD (length 10)', today.length, 10);
assert('contains dashes at positions 4 and 7', today[4] + today[7], '--');
assert('year is 2026', today.slice(0, 4), '2026');
```

- [ ] **Step 2: Open test.html and confirm all 4 tests pass**

Reload `test.html`. You should see 4 green ✓ lines and "4 passed, 0 failed".

- [ ] **Step 3: Copy getToday() into index.html**

In `index.html`, replace the comment `// pure utils — added in Tasks 3–6` with:

```javascript
function getToday() {
  return new Date().toLocaleDateString('en-CA');
}
```

- [ ] **Step 4: Commit**

```bash
git add test.html index.html
git commit -m "feat: add getToday utility"
```

---

### Task 4: Storage layer (TDD)

**Files:**
- Modify: `test.html`
- Modify: `index.html`

- [ ] **Step 1: Add storage tests to test.html**

In `test.html`, add these functions in the pure functions section:

```javascript
function loadHabits() {
  try {
    return JSON.parse(localStorage.getItem('habits') || '[]');
  } catch {
    return [];
  }
}

function saveHabits(habits) {
  localStorage.setItem('habits', JSON.stringify(habits));
}
```

In the tests section, add:

```javascript
section('loadHabits / saveHabits');
localStorage.removeItem('habits');
assert('returns empty array when nothing saved', loadHabits(), []);

const sample = [{ id: 'a1', name: 'Test', createdDate: '2026-06-01', completedDates: [] }];
saveHabits(sample);
assert('round-trips a habit array', loadHabits(), sample);

localStorage.setItem('habits', 'not valid json{{');
assert('returns empty array on corrupt data', loadHabits(), []);

localStorage.removeItem('habits');
```

- [ ] **Step 2: Open test.html and confirm all tests pass**

Reload `test.html`. You should see 3 new green ✓ lines. Total: "7 passed, 0 failed".

- [ ] **Step 3: Copy storage functions into index.html**

In `index.html`, add after `getToday()`:

```javascript
function loadHabits() {
  try {
    return JSON.parse(localStorage.getItem('habits') || '[]');
  } catch {
    return [];
  }
}

function saveHabits(habits) {
  localStorage.setItem('habits', JSON.stringify(habits));
}
```

- [ ] **Step 4: Commit**

```bash
git add test.html index.html
git commit -m "feat: add localStorage storage layer"
```

---

### Task 5: calcStreak() (TDD)

**Files:**
- Modify: `test.html`
- Modify: `index.html`

- [ ] **Step 1: Add calcStreak to test.html**

In the pure functions section of `test.html`, add:

```javascript
function calcStreak(habit, today = getToday()) {
  const dates = new Set(habit.completedDates);
  let streak = 0;
  const cursor = new Date(today + 'T00:00:00');

  if (!dates.has(today)) {
    cursor.setDate(cursor.getDate() - 1);
  }

  while (true) {
    const d = cursor.toLocaleDateString('en-CA');
    if (dates.has(d)) {
      streak++;
      cursor.setDate(cursor.getDate() - 1);
    } else {
      break;
    }
  }

  return streak;
}
```

- [ ] **Step 2: Add calcStreak tests to test.html**

In the tests section, add:

```javascript
section('calcStreak');

assert('streak is 0 with no completed dates', calcStreak(
  { completedDates: [] },
  '2026-06-08'
), 0);

assert('streak is 1 when only today is done', calcStreak(
  { completedDates: ['2026-06-08'] },
  '2026-06-08'
), 1);

assert('streak is 3 for three consecutive days ending today', calcStreak(
  { completedDates: ['2026-06-06', '2026-06-07', '2026-06-08'] },
  '2026-06-08'
), 3);

assert('streak is 2 when today not done but yesterday and day before are', calcStreak(
  { completedDates: ['2026-06-06', '2026-06-07'] },
  '2026-06-08'
), 2);

assert('streak resets after a gap', calcStreak(
  { completedDates: ['2026-06-01', '2026-06-07', '2026-06-08'] },
  '2026-06-08'
), 2);

assert('streak is 0 when last done was 2 days ago', calcStreak(
  { completedDates: ['2026-06-06'] },
  '2026-06-08'
), 0);
```

- [ ] **Step 3: Open test.html and confirm all tests pass**

Reload `test.html`. You should see 6 new green ✓ lines. Total: "13 passed, 0 failed".

- [ ] **Step 4: Copy calcStreak() into index.html**

In `index.html`, add after `saveHabits()`:

```javascript
function calcStreak(habit, today = getToday()) {
  const dates = new Set(habit.completedDates);
  let streak = 0;
  const cursor = new Date(today + 'T00:00:00');

  if (!dates.has(today)) {
    cursor.setDate(cursor.getDate() - 1);
  }

  while (true) {
    const d = cursor.toLocaleDateString('en-CA');
    if (dates.has(d)) {
      streak++;
      cursor.setDate(cursor.getDate() - 1);
    } else {
      break;
    }
  }

  return streak;
}
```

- [ ] **Step 5: Commit**

```bash
git add test.html index.html
git commit -m "feat: add streak calculation"
```

---

### Task 6: calcOverall() (TDD)

**Files:**
- Modify: `test.html`
- Modify: `index.html`

- [ ] **Step 1: Add calcOverall to test.html**

In the pure functions section of `test.html`, add:

```javascript
function calcOverall(habit, today = getToday()) {
  const start = new Date(habit.createdDate + 'T00:00:00');
  const end = new Date(today + 'T00:00:00');
  const possible = Math.floor((end - start) / (1000 * 60 * 60 * 24)) + 1;
  return {
    completed: habit.completedDates.length,
    possible
  };
}
```

- [ ] **Step 2: Add calcOverall tests to test.html**

In the tests section, add:

```javascript
section('calcOverall');

assert('1 possible on the day habit was created', calcOverall(
  { createdDate: '2026-06-08', completedDates: [] },
  '2026-06-08'
), { completed: 0, possible: 1 });

assert('21 possible after 21 days', calcOverall(
  { createdDate: '2026-05-18', completedDates: [] },
  '2026-06-07'
), { completed: 0, possible: 21 });

assert('17/21 matches example from spec', calcOverall(
  {
    createdDate: '2026-05-18',
    completedDates: ['2026-05-18','2026-05-19','2026-05-20','2026-05-21','2026-05-22',
                     '2026-05-24','2026-05-25','2026-05-26','2026-05-27','2026-05-28',
                     '2026-05-29','2026-05-31','2026-06-01','2026-06-02','2026-06-03',
                     '2026-06-04','2026-06-05']
  },
  '2026-06-07'
), { completed: 17, possible: 21 });
```

- [ ] **Step 3: Open test.html and confirm all tests pass**

Reload `test.html`. You should see 3 new green ✓ lines. Total: "16 passed, 0 failed".

- [ ] **Step 4: Copy calcOverall() into index.html**

In `index.html`, add after `calcStreak()`:

```javascript
function calcOverall(habit, today = getToday()) {
  const start = new Date(habit.createdDate + 'T00:00:00');
  const end = new Date(today + 'T00:00:00');
  const possible = Math.floor((end - start) / (1000 * 60 * 60 * 24)) + 1;
  return {
    completed: habit.completedDates.length,
    possible
  };
}
```

- [ ] **Step 5: Commit**

```bash
git add test.html index.html
git commit -m "feat: add overall count calculation"
```

---

### Task 7: render() function + empty state

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add escapeHtml helper and render() to index.html**

In `index.html`, add after `calcOverall()`:

```javascript
function escapeHtml(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');
}

function render() {
  const habits = loadHabits();
  const today = getToday();
  const list = document.getElementById('habit-list');
  const footer = document.getElementById('footer');

  document.getElementById('today-label').textContent =
    new Date().toLocaleDateString('en-US', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });

  if (habits.length === 0) {
    list.innerHTML = `
      <div class="empty-state">
        <p>No habits yet.</p>
        <p>Click <strong>+ Add Habit</strong> to get started.</p>
      </div>
    `;
    footer.textContent = '';
    return;
  }

  const doneCount = habits.filter(h => h.completedDates.includes(today)).length;
  footer.textContent = `${doneCount} of ${habits.length} habits done today`;

  list.innerHTML = habits.map(habit => {
    const done = habit.completedDates.includes(today);
    const streak = calcStreak(habit);
    const { completed, possible } = calcOverall(habit);
    return `
      <div class="habit-row ${done ? 'done' : ''}" data-id="${habit.id}">
        <div class="habit-left" onclick="toggleComplete('${habit.id}')">
          <div class="checkbox ${done ? 'checked' : ''}">${done ? '✓' : ''}</div>
          <span class="habit-name">${escapeHtml(habit.name)}</span>
        </div>
        <div class="habit-right">
          <div class="stat streak">
            <div class="stat-value">🔥 ${streak}</div>
            <div class="stat-label">day streak</div>
          </div>
          <div class="divider-v"></div>
          <div class="stat overall">
            <div class="stat-value">${completed}/${possible}</div>
            <div class="stat-label">overall</div>
          </div>
          <button class="menu-btn" onclick="toggleMenu(event, '${habit.id}')">⋯</button>
          <div class="menu" id="menu-${habit.id}">
            <button onclick="startRename('${habit.id}')">Rename</button>
            <button onclick="deleteHabit('${habit.id}')">Delete</button>
          </div>
        </div>
      </div>
    `;
  }).join('');
}
```

- [ ] **Step 2: Add init call at the bottom of the script block**

At the very bottom of the `<script>` block (after all functions), add:

```javascript
document.addEventListener('click', (e) => {
  if (!e.target.closest('.menu-btn') && !e.target.closest('.menu')) {
    document.querySelectorAll('.menu').forEach(m => m.style.display = 'none');
  }
});

render();
```

- [ ] **Step 3: Open index.html and confirm the empty state shows**

Reload `index.html`. You should see "No habits yet. Click + Add Habit to get started." No console errors.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add render function and empty state"
```

---

### Task 8: Toggle completion

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add toggleComplete() to index.html**

In `index.html`, add after `render()`:

```javascript
function toggleComplete(id) {
  const habits = loadHabits();
  const today = getToday();
  const habit = habits.find(h => h.id === id);
  if (!habit) return;

  const idx = habit.completedDates.indexOf(today);
  if (idx === -1) {
    habit.completedDates.push(today);
  } else {
    habit.completedDates.splice(idx, 1);
  }

  saveHabits(habits);
  render();
}
```

- [ ] **Step 2: Seed a test habit directly in localStorage to test with**

Open `index.html` in the browser. Open the console (Cmd+Option+J) and paste:

```javascript
localStorage.setItem('habits', JSON.stringify([
  { id: 'test1', name: 'Drink Water', createdDate: '2026-05-18', completedDates: [] }
]));
location.reload();
```

- [ ] **Step 3: Confirm toggling works**

You should now see "Drink Water" in the list. Click the habit row — the checkbox should turn green, the streak should show 🔥 1, and the overall should update. Click again — it should uncheck. No console errors.

- [ ] **Step 4: Clear test data**

In the console, run:

```javascript
localStorage.removeItem('habits');
location.reload();
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add toggle completion"
```

---

### Task 9: Add habit modal

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add modal functions to index.html**

In `index.html`, add after `toggleComplete()`:

```javascript
function showModal() {
  document.getElementById('modal').style.display = 'flex';
  document.getElementById('habit-input').value = '';
  setTimeout(() => document.getElementById('habit-input').focus(), 50);
}

function hideModal() {
  document.getElementById('modal').style.display = 'none';
}

function addHabit() {
  const input = document.getElementById('habit-input');
  const name = input.value.trim();
  if (!name) return;

  const habits = loadHabits();
  habits.push({
    id: Date.now().toString(36),
    name,
    createdDate: getToday(),
    completedDates: []
  });

  saveHabits(habits);
  hideModal();
  render();
}
```

- [ ] **Step 2: Close modal when clicking outside the box**

Find the `<div id="modal">` element in the HTML and add an onclick to it:

```html
<div id="modal" onclick="if(event.target===this) hideModal()">
```

- [ ] **Step 3: Test the modal**

Reload `index.html`. Click "+ Add Habit". A modal should appear. Type "Drink Water" and press Enter (or click Save). The habit should appear in the list with streak 🔥 0 and overall 1/1. Click Cancel — modal should close. No console errors.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add habit modal"
```

---

### Task 10: Rename and delete

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add toggleMenu(), deleteHabit(), and startRename() to index.html**

In `index.html`, add after `addHabit()`:

```javascript
function toggleMenu(event, id) {
  event.stopPropagation();
  const menu = document.getElementById(`menu-${id}`);
  const isOpen = menu.style.display === 'block';
  document.querySelectorAll('.menu').forEach(m => m.style.display = 'none');
  if (!isOpen) menu.style.display = 'block';
}

function deleteHabit(id) {
  if (!confirm('Delete this habit? This cannot be undone.')) return;
  const habits = loadHabits().filter(h => h.id !== id);
  saveHabits(habits);
  render();
}

function startRename(id) {
  const habits = loadHabits();
  const habit = habits.find(h => h.id === id);
  if (!habit) return;
  const newName = prompt('Rename habit:', habit.name);
  if (!newName || !newName.trim()) return;
  habit.name = newName.trim();
  saveHabits(habits);
  render();
}
```

- [ ] **Step 2: Test rename and delete**

Add a couple of habits via the modal. Click ⋯ on one — a small menu should appear with Rename and Delete. Click Rename, type a new name, press OK — the habit updates. Click ⋯ again, click Delete, confirm — the habit is removed. No console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add rename and delete"
```

---

### Task 11: CSS dark theme

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Replace the `/* styles go here */` comment in the `<style>` block with the full CSS**

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  background: #0f172a;
  color: #f1f5f9;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  min-height: 100vh;
}

.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 28px 16px;
}

header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}

h1 {
  font-size: 24px;
  font-weight: 700;
}

.date {
  color: #64748b;
  font-size: 13px;
  margin-top: 3px;
}

.add-btn {
  background: #4ade80;
  color: #000;
  border: none;
  border-radius: 8px;
  padding: 10px 16px;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
  white-space: nowrap;
}

.add-btn:hover {
  background: #22c55e;
}

.divider {
  height: 1px;
  background: #1e293b;
  margin: 16px 0;
}

.habit-row {
  background: #1e293b;
  border-radius: 10px;
  padding: 14px 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-left: 3px solid #374151;
  margin-bottom: 10px;
  position: relative;
  transition: border-left-color 0.2s;
}

.habit-row.done {
  border-left-color: #4ade80;
}

.habit-left {
  display: flex;
  align-items: center;
  gap: 12px;
  flex: 1;
  cursor: pointer;
}

.checkbox {
  width: 26px;
  height: 26px;
  background: #374151;
  border: 2px solid #475569;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  flex-shrink: 0;
  transition: background 0.2s, border-color 0.2s;
}

.checkbox.checked {
  background: #4ade80;
  border-color: #4ade80;
  color: #000;
  font-weight: 700;
}

.habit-name {
  font-size: 15px;
}

.habit-row.done .habit-name {
  color: #f1f5f9;
}

.habit-row:not(.done) .habit-name {
  color: #94a3b8;
}

.habit-right {
  display: flex;
  align-items: center;
  gap: 14px;
  flex-shrink: 0;
}

.stat {
  text-align: center;
}

.stat-value {
  font-size: 17px;
  font-weight: 700;
}

.streak .stat-value {
  color: #f97316;
}

.overall .stat-value {
  color: #60a5fa;
}

.stat-label {
  font-size: 10px;
  color: #64748b;
  margin-top: 2px;
}

.divider-v {
  width: 1px;
  height: 28px;
  background: #334155;
}

.menu-btn {
  background: none;
  border: none;
  color: #475569;
  cursor: pointer;
  font-size: 20px;
  padding: 0 4px;
  line-height: 1;
}

.menu-btn:hover {
  color: #94a3b8;
}

.menu {
  display: none;
  position: absolute;
  right: 12px;
  top: calc(100% + 4px);
  background: #1e3a5f;
  border: 1px solid #334155;
  border-radius: 8px;
  padding: 6px;
  z-index: 10;
  min-width: 110px;
  box-shadow: 0 8px 24px rgba(0,0,0,0.4);
}

.menu button {
  display: block;
  width: 100%;
  background: none;
  border: none;
  color: #f1f5f9;
  cursor: pointer;
  padding: 7px 14px;
  font-size: 13px;
  text-align: left;
  border-radius: 5px;
}

.menu button:hover {
  background: #2d4a6f;
}

.empty-state {
  text-align: center;
  padding: 48px 0;
  color: #475569;
  line-height: 2;
}

.footer {
  text-align: center;
  color: #475569;
  font-size: 13px;
  margin-top: 16px;
}

/* Modal */
#modal {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.modal-box {
  background: #1e293b;
  border-radius: 12px;
  padding: 24px;
  width: 90%;
  max-width: 400px;
  box-shadow: 0 16px 48px rgba(0,0,0,0.5);
}

.modal-box h2 {
  font-size: 18px;
  margin-bottom: 16px;
}

.modal-input {
  width: 100%;
  background: #0f172a;
  border: 1px solid #334155;
  border-radius: 8px;
  padding: 10px 14px;
  color: #f1f5f9;
  font-size: 15px;
  margin-bottom: 16px;
  outline: none;
}

.modal-input:focus {
  border-color: #4ade80;
}

.modal-actions {
  display: flex;
  gap: 8px;
  justify-content: flex-end;
}

.btn-primary {
  background: #4ade80;
  color: #000;
  border: none;
  border-radius: 8px;
  padding: 10px 20px;
  font-size: 14px;
  font-weight: 700;
  cursor: pointer;
}

.btn-primary:hover {
  background: #22c55e;
}

.btn-secondary {
  background: #374151;
  color: #94a3b8;
  border: none;
  border-radius: 8px;
  padding: 10px 20px;
  font-size: 14px;
  cursor: pointer;
}

.btn-secondary:hover {
  background: #4b5563;
}
```

- [ ] **Step 2: Reload index.html and verify the full dark theme**

The app should now look like the mockup: dark navy background, green checkboxes, orange streak numbers, blue overall counts, a clean modal. Add a couple of habits and check them off.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add dark theme CSS"
```

---

### Task 12: Final checks

**Files:**
- None (verification only)

- [ ] **Step 1: Full manual walkthrough**

Open `index.html` and do these steps:
1. Confirm empty state shows
2. Add 3 habits
3. Check 2 of them — verify green border, ✓ checkbox, streak shows 🔥 1, overall shows 1/1
4. Uncheck one — verify it reverts
5. Click ⋯ → Rename one habit → confirm it updates
6. Click ⋯ → Delete one habit → confirm it's gone
7. Close and reopen `index.html` — confirm habits and completions persisted

- [ ] **Step 2: Run all tests one final time**

Open `test.html` — confirm "16 passed, 0 failed".

- [ ] **Step 3: Final commit**

```bash
git add -A
git commit -m "feat: habit tracker complete"
```
