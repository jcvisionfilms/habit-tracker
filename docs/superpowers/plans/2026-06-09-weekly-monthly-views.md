# Weekly & Monthly Views Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Weekly view (7-day grid + completion % per habit) and a Monthly view (calendar grid colored by completion) to the existing single-file habit tracker, navigated by a three-tab bar.

**Architecture:** All changes are confined to `index.html`. New CSS classes are added to the `<style>` block. A tab bar is inserted between the `.divider` and `#habit-list`. State variables (`currentView`, `currentMonth`, `currentYear`) and new pure/DOM functions are added to the `<script>` block following existing patterns. No data model changes required.

**Tech Stack:** Vanilla HTML, CSS, JavaScript. localStorage. No dependencies.

---

## File Map

| File | Changes |
|------|---------|
| `index.html` | Add CSS, tab bar HTML, state vars, 6 new JS functions |
| `test.html` | Add `getWeekDates()` and `getDayColor()` with tests |

---

### Task 1: Add CSS for tabs, weekly view, and monthly view

**Files:**
- Modify: `index.html` (inside `<style>` block, before `</style>`)

- [ ] **Step 1: Add the following CSS at the very end of the `<style>` block, just before `</style>` on line 295**

```css
    /* ── Tabs ─────────────────────────────────────────────────── */
    .tab-bar {
      display: flex;
      gap: 4px;
      background: #1e293b;
      border-radius: 10px;
      padding: 4px;
      margin-bottom: 16px;
    }

    .tab {
      flex: 1;
      background: none;
      border: none;
      color: #64748b;
      padding: 8px;
      border-radius: 7px;
      font-size: 13px;
      cursor: pointer;
      font-family: inherit;
    }

    .tab.active {
      background: #334155;
      color: #f1f5f9;
      font-weight: 700;
    }

    /* ── Weekly view ──────────────────────────────────────────── */
    .week-header {
      display: flex;
      align-items: center;
      gap: 4px;
      padding: 0 4px;
      margin-bottom: 4px;
    }

    .week-row {
      display: flex;
      align-items: center;
      gap: 4px;
      background: #1e293b;
      border-radius: 10px;
      padding: 10px 12px;
      border-left: 3px solid #334155;
      margin-bottom: 8px;
    }

    .week-name-col {
      width: 110px;
      flex-shrink: 0;
      overflow: hidden;
    }

    .habit-name-week {
      font-size: 13px;
      color: #f1f5f9;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      display: block;
    }

    .week-day-label {
      flex: 1;
      text-align: center;
      font-size: 10px;
      color: #64748b;
    }

    .week-pct-col {
      width: 44px;
      flex-shrink: 0;
      text-align: center;
    }

    .week-pct-val {
      font-size: 13px;
      font-weight: 700;
    }

    .week-pct-sub {
      font-size: 10px;
      color: #64748b;
    }

    .day-sq {
      flex: 1;
      aspect-ratio: 1;
      background: #374151;
      border-radius: 4px;
    }

    .day-sq.done {
      background: #4ade80;
    }

    .day-sq.today-sq {
      outline: 2px solid #60a5fa;
      outline-offset: 1px;
    }

    /* ── Monthly view ─────────────────────────────────────────── */
    .month-nav {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 12px;
    }

    .month-title {
      font-size: 15px;
      font-weight: 700;
      color: #f1f5f9;
    }

    .month-arrow {
      background: #1e293b;
      border: none;
      color: #94a3b8;
      padding: 6px 12px;
      border-radius: 6px;
      cursor: pointer;
      font-size: 20px;
      line-height: 1;
      font-family: inherit;
    }

    .month-arrow:disabled {
      opacity: 0.3;
      cursor: default;
    }

    .cal-grid-labels,
    .cal-grid {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 4px;
    }

    .cal-grid-labels {
      margin-bottom: 4px;
    }

    .cal-day-label {
      text-align: center;
      font-size: 10px;
      color: #64748b;
    }

    .cal-grid {
      margin-bottom: 12px;
    }

    .cal-day {
      background: #374151;
      border-radius: 6px;
      padding: 6px 0;
      text-align: center;
      font-size: 11px;
      color: #64748b;
    }

    .cal-day.all {
      background: #4ade80;
      color: #000;
      font-weight: 700;
    }

    .cal-day.some {
      background: #facc15;
      color: #000;
      font-weight: 700;
    }

    .cal-day.future {
      background: #1e293b;
      color: #334155;
    }

    .cal-day.empty {
      background: transparent;
    }

    .cal-day.cal-today {
      outline: 2px solid #60a5fa;
      outline-offset: 1px;
    }

    .cal-legend {
      display: flex;
      align-items: center;
      gap: 12px;
      justify-content: center;
      flex-wrap: wrap;
      margin-top: 4px;
    }

    .legend-item {
      display: flex;
      align-items: center;
      gap: 4px;
      font-size: 10px;
      color: #64748b;
    }

    .legend-sq {
      width: 12px;
      height: 12px;
      border-radius: 3px;
      background: #374151;
      flex-shrink: 0;
    }

    .legend-sq.all  { background: #4ade80; }
    .legend-sq.some { background: #facc15; }
    .legend-sq.none { background: #374151; }
    .legend-sq.today-legend {
      background: transparent;
      outline: 2px solid #60a5fa;
      outline-offset: 1px;
    }
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add CSS for tab bar, weekly view, and monthly view"
```

---

### Task 2: Add tab bar HTML + state variables + switchView/renderCurrentView

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add the tab bar HTML**

Find this block in `index.html` (around line 307):
```html
    <div class="divider"></div>

    <div id="habit-list"></div>
```

Replace it with:
```html
    <div class="divider"></div>

    <div class="tab-bar">
      <button class="tab active" data-view="today" onclick="switchView('today')">Today</button>
      <button class="tab" data-view="week" onclick="switchView('week')">Week</button>
      <button class="tab" data-view="month" onclick="switchView('month')">Month</button>
    </div>

    <div id="habit-list"></div>
```

- [ ] **Step 2: Add state variables at the very top of the `<script>` block**

Find the opening `<script>` tag (around line 331). Add these three lines immediately after it (before `function getToday`):

```javascript
    let currentView = 'today';
    let currentMonth = new Date().getMonth();
    let currentYear = new Date().getFullYear();
```

- [ ] **Step 3: Add switchView() and renderCurrentView() after the escapeHtml() function**

Find `function escapeHtml(str)` (around line 383). Add these two functions immediately after the closing `}` of `escapeHtml`, before `function render()`:

```javascript
    function switchView(view) {
      currentView = view;
      document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
      document.querySelector(`.tab[data-view="${view}"]`).classList.add('active');
      renderCurrentView();
    }

    function renderCurrentView() {
      if (currentView === 'today') render();
      else if (currentView === 'week') renderWeekView();
      else renderMonthView();
    }
```

- [ ] **Step 4: Replace the init call at the bottom of the script**

Find `render();` at the very bottom of the `<script>` block (the last line before `</script>`). Replace it with:

```javascript
    renderCurrentView();
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add tab bar and view dispatcher"
```

---

### Task 3: getWeekDates() — pure function (TDD)

**Files:**
- Modify: `test.html` (add function + tests)
- Modify: `index.html` (add function after calcOverall)

- [ ] **Step 1: Add getWeekDates() to test.html in the pure functions section**

In `test.html`, add after `calcOverall`:

```javascript
    function getWeekDates(today = getToday()) {
      const cursor = new Date(today + 'T00:00:00');
      const dow = cursor.getDay();
      const daysFromMonday = (dow === 0) ? 6 : dow - 1;
      cursor.setDate(cursor.getDate() - daysFromMonday);
      const dates = [];
      for (let i = 0; i < 7; i++) {
        dates.push(cursor.toLocaleDateString('en-CA'));
        cursor.setDate(cursor.getDate() + 1);
      }
      return dates;
    }
```

- [ ] **Step 2: Add tests for getWeekDates() to test.html**

In `test.html`, add after the calcOverall tests:

```javascript
    section('getWeekDates');

    assert('returns 7 dates', getWeekDates('2026-06-03').length, 7);

    assert('Wednesday → week starts on Monday', getWeekDates('2026-06-03')[0], '2026-06-02');

    assert('Wednesday → week ends on Sunday', getWeekDates('2026-06-03')[6], '2026-06-08');

    assert('Monday → first element is itself', getWeekDates('2026-06-01')[0], '2026-06-01');

    assert('Monday → last element is Sunday', getWeekDates('2026-06-01')[6], '2026-06-07');

    assert('Sunday → week starts 6 days prior', getWeekDates('2026-06-07')[0], '2026-06-01');

    assert('Sunday → last element is itself', getWeekDates('2026-06-07')[6], '2026-06-07');
```

- [ ] **Step 3: Open test.html and confirm the new tests pass**

Open `test.html` in your browser. You should see 7 new green ✓ lines. Total should now read "23 passed, 0 failed".

- [ ] **Step 4: Copy getWeekDates() into index.html**

In `index.html`, add `getWeekDates` immediately after `calcOverall` (after its closing `}`):

```javascript
    function getWeekDates(today = getToday()) {
      const cursor = new Date(today + 'T00:00:00');
      const dow = cursor.getDay();
      const daysFromMonday = (dow === 0) ? 6 : dow - 1;
      cursor.setDate(cursor.getDate() - daysFromMonday);
      const dates = [];
      for (let i = 0; i < 7; i++) {
        dates.push(cursor.toLocaleDateString('en-CA'));
        cursor.setDate(cursor.getDate() + 1);
      }
      return dates;
    }
```

- [ ] **Step 5: Commit**

```bash
git add test.html index.html
git commit -m "feat: add getWeekDates utility"
```

---

### Task 4: renderWeekView()

**Files:**
- Modify: `index.html` (add after render())

- [ ] **Step 1: Add renderWeekView() after the closing } of render()**

In `index.html`, add immediately after `render()` (after its closing `}`):

```javascript
    function renderWeekView() {
      const habits = loadHabits();
      const today = getToday();
      const weekDates = getWeekDates(today);
      const dayLabels = ['M', 'T', 'W', 'T', 'F', 'S', 'S'];
      const list = document.getElementById('habit-list');
      const footer = document.getElementById('footer');

      const start = weekDates[0];
      const end = weekDates[6];
      const fmt = d => new Date(d + 'T00:00:00').toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
      const year = new Date(today + 'T00:00:00').getFullYear();
      document.getElementById('today-label').textContent =
        `Week of ${fmt(start)} – ${fmt(end)}, ${year}`;

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

      const headerHtml = `
        <div class="week-header">
          <div class="week-name-col"></div>
          ${dayLabels.map(l => `<div class="week-day-label">${l}</div>`).join('')}
          <div class="week-pct-col" style="font-size:10px;color:#64748b;">Week</div>
        </div>
      `;

      const rowsHtml = habits.map(habit => {
        const doneCount = weekDates.filter(d => habit.completedDates.includes(d)).length;
        const pct = Math.round((doneCount / 7) * 100);
        const pctColor = pct >= 50 ? '#4ade80' : '#f97316';
        const squares = weekDates.map(d => {
          const done = habit.completedDates.includes(d);
          const isToday = d === today;
          return `<div class="day-sq${done ? ' done' : ''}${isToday ? ' today-sq' : ''}"></div>`;
        }).join('');
        return `
          <div class="week-row">
            <div class="week-name-col">
              <span class="habit-name-week">${escapeHtml(habit.name)}</span>
            </div>
            ${squares}
            <div class="week-pct-col">
              <div class="week-pct-val" style="color:${pctColor}">${pct}%</div>
              <div class="week-pct-sub">${doneCount}/7</div>
            </div>
          </div>
        `;
      }).join('');

      list.innerHTML = headerHtml + rowsHtml;

      const avg = Math.round(
        habits.map(h => weekDates.filter(d => h.completedDates.includes(d)).length / 7 * 100)
              .reduce((a, b) => a + b, 0) / habits.length
      );
      footer.textContent = `Week average: ${avg}% across all habits`;
    }
```

- [ ] **Step 2: Test the weekly view**

Open `index.html` in your browser. Click the **Week** tab. If you have habits, you should see a row per habit with 7 squares and a weekly %. Today's square should have a blue outline. The footer should show a week average. Click **Today** — confirms it switches back. No console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add weekly view"
```

---

### Task 5: getDayColor() — pure function (TDD)

**Files:**
- Modify: `test.html`
- Modify: `index.html` (add after getWeekDates)

- [ ] **Step 1: Add getDayColor() to test.html in the pure functions section**

In `test.html`, add after `getWeekDates`:

```javascript
    function getDayColor(habits, dateStr, today = getToday()) {
      if (dateStr > today) return 'future';
      if (habits.length === 0) return 'none';
      const done = habits.filter(h => h.completedDates.includes(dateStr)).length;
      if (done === 0) return 'none';
      if (done === habits.length) return 'all';
      return 'some';
    }
```

- [ ] **Step 2: Add tests for getDayColor() in test.html**

In `test.html`, add after the getWeekDates tests:

```javascript
    section('getDayColor');

    const habitsForColor = [
      { completedDates: ['2026-06-01', '2026-06-02'] },
      { completedDates: ['2026-06-01'] }
    ];

    assert('future date returns future', getDayColor(habitsForColor, '2026-12-31', '2026-06-09'), 'future');

    assert('no habits returns none', getDayColor([], '2026-06-01', '2026-06-09'), 'none');

    assert('all habits done returns all', getDayColor(habitsForColor, '2026-06-01', '2026-06-09'), 'all');

    assert('some habits done returns some', getDayColor(habitsForColor, '2026-06-02', '2026-06-09'), 'some');

    assert('no habits done returns none', getDayColor(habitsForColor, '2026-06-03', '2026-06-09'), 'none');

    assert('today is not treated as future even when not completed', getDayColor(habitsForColor, '2026-06-09', '2026-06-09'), 'none');
```

- [ ] **Step 3: Open test.html and confirm the new tests pass**

Open `test.html`. You should see 6 new green ✓ lines. Total: "29 passed, 0 failed".

- [ ] **Step 4: Copy getDayColor() into index.html**

In `index.html`, add `getDayColor` immediately after `getWeekDates`:

```javascript
    function getDayColor(habits, dateStr, today = getToday()) {
      if (dateStr > today) return 'future';
      if (habits.length === 0) return 'none';
      const done = habits.filter(h => h.completedDates.includes(dateStr)).length;
      if (done === 0) return 'none';
      if (done === habits.length) return 'all';
      return 'some';
    }
```

- [ ] **Step 5: Commit**

```bash
git add test.html index.html
git commit -m "feat: add getDayColor utility"
```

---

### Task 6: renderMonthView() + prevMonth() + nextMonth()

**Files:**
- Modify: `index.html` (add after renderWeekView)

- [ ] **Step 1: Add renderMonthView(), prevMonth(), and nextMonth() after renderWeekView()**

In `index.html`, add immediately after the closing `}` of `renderWeekView()`:

```javascript
    function renderMonthView() {
      const habits = loadHabits();
      const today = getToday();
      const list = document.getElementById('habit-list');
      const footer = document.getElementById('footer');

      const monthNames = ['January','February','March','April','May','June',
                          'July','August','September','October','November','December'];
      document.getElementById('today-label').textContent =
        `${monthNames[currentMonth]} ${currentYear}`;

      const firstDow = new Date(currentYear, currentMonth, 1).getDay();
      const offset = (firstDow === 0) ? 6 : firstDow - 1;
      const daysInMonth = new Date(currentYear, currentMonth + 1, 0).getDate();

      let cellsHtml = '';
      for (let i = 0; i < offset; i++) {
        cellsHtml += `<div class="cal-day empty"></div>`;
      }
      for (let d = 1; d <= daysInMonth; d++) {
        const mm = String(currentMonth + 1).padStart(2, '0');
        const dd = String(d).padStart(2, '0');
        const dateStr = `${currentYear}-${mm}-${dd}`;
        const color = getDayColor(habits, dateStr, today);
        const isToday = dateStr === today;
        cellsHtml += `
          <div class="cal-day ${color}${isToday ? ' cal-today' : ''}">
            <span>${d}</span>
          </div>
        `;
      }

      const dayLabels = ['M','T','W','T','F','S','S']
        .map(l => `<div class="cal-day-label">${l}</div>`).join('');

      const now = new Date();
      const isCurrentMonth = currentYear === now.getFullYear() && currentMonth === now.getMonth();

      list.innerHTML = `
        <div class="month-nav">
          <button class="month-arrow" onclick="prevMonth()">&#8249;</button>
          <span class="month-title">${monthNames[currentMonth]} ${currentYear}</span>
          <button class="month-arrow" onclick="nextMonth()" ${isCurrentMonth ? 'disabled' : ''}>&#8250;</button>
        </div>
        <div class="cal-grid-labels">${dayLabels}</div>
        <div class="cal-grid">${cellsHtml}</div>
        <div class="cal-legend">
          <div class="legend-item"><div class="legend-sq all"></div><span>All done</span></div>
          <div class="legend-item"><div class="legend-sq some"></div><span>Some done</span></div>
          <div class="legend-item"><div class="legend-sq none"></div><span>None done</span></div>
          <div class="legend-item"><div class="legend-sq today-legend"></div><span>Today</span></div>
        </div>
      `;

      footer.textContent = '';
    }

    function prevMonth() {
      if (currentMonth === 0) {
        currentMonth = 11;
        currentYear--;
      } else {
        currentMonth--;
      }
      renderMonthView();
    }

    function nextMonth() {
      const now = new Date();
      if (currentYear === now.getFullYear() && currentMonth === now.getMonth()) return;
      if (currentMonth === 11) {
        currentMonth = 0;
        currentYear++;
      } else {
        currentMonth++;
      }
      renderMonthView();
    }
```

- [ ] **Step 2: Test the monthly view**

Open `index.html`. Click the **Month** tab. You should see a calendar grid for the current month. Days with all habits done are green, partial yellow, none grey. Today has a blue outline. The `‹` button goes to the previous month; `›` is greyed out if you're on the current month. Click **Today** and **Week** to confirm tabs still work. No console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add monthly view with navigation"
```

---

### Task 7: Final checks and push

**Files:**
- None (verification + push only)

- [ ] **Step 1: Full walkthrough**

Open `index.html` and verify:
1. **Today tab** — habits list looks the same as before, streaks and overall counts correct
2. **Week tab** — 7 squares per habit, today's square has blue outline, % shown, week average in footer
3. **Month tab** — calendar grid with correct day-of-week alignment, ‹ navigates back, › disabled on current month
4. **Add a habit** — appears in all three views correctly
5. **Close and reopen** — data persists, tab defaults back to Today

- [ ] **Step 2: Run tests**

Open `test.html`. Confirm "29 passed, 0 failed".

- [ ] **Step 3: Push to GitHub**

```bash
git push
```

This updates your live site at `https://jcvisionfilms.github.io/habit-tracker/` within a minute or two.

- [ ] **Step 4: Final commit if any cleanup needed**

```bash
git add -A
git commit -m "feat: weekly and monthly views complete"
git push
```
