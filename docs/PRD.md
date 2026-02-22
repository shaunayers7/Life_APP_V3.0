# Life App V3 — Product Requirements Document

---

## Section 1: Overview

**App Name:** Life App V3  
**Type:** Progressive Web App (PWA)  
**Primary Platform:** iOS Safari (installed to home screen)  
**Secondary Platform:** Android Chrome  
**Architecture:** Single-file React app (`index.html`), Babel in-browser, offline-first  
**Backend:** Firebase + Firestore  
**Storage Key:** `life_app_v3_data` — IMMUTABLE  

---

## Section 2: Goals

> _(Fill in as the product takes shape)_

- [ ] Goal 1
- [ ] Goal 2

---

## Section 3: Features

| Feature | Status | Notes |
|---------|--------|-------|
| Zen Focus Mode | ✅ Built | Shows top N scored tasks, tap to select, others fade out |
| Task scoring algorithm | ✅ Built | Priority × weight + due date urgency + neglect + age |
| Pass / skip task | ✅ Built | Increments skippedCount, boosts future score |
| Dashboard / task list | ✅ Built | Filter by category, sorted by score, toggle complete |
| Add / edit task | ✅ Built | Title, category, priority (1–5), due date, notes |
| Stopwatch timer | ✅ Built | Tracks time spent per task, survives app kill |
| Countdown timer | ✅ Built | Counts down from configurable duration |
| Pomodoro timer | ✅ Built | Configurable work/break cycles with dot tracker |
| localStorage persistence | ✅ Built | Immediate sync before setState — iOS safe |
| Toast notifications | ✅ Built | No alert()/confirm()/prompt() used anywhere |
| Settings view | ✅ Built | Zen offer count, Pomodoro durations, stats, clear completed |
| Stats dashboard | ✅ Built | Active, done, total, time tracked |
| Default categories | ✅ Built | Work, Health, Home, Finance, Personal, Projects |

---

## Section 4: Data Structure

```javascript
// Primary storage key: life_app_v3_data
{
  tasks: Task[],
  categories: Category[],
  settings: {
    zenOfferCount: number,   // 1–5, default 3
    pomodoroWork:  number,   // minutes, default 25
    pomodoroBreak: number,   // minutes, default 5
  }
}

// Task shape (DO NOT rename/remove fields)
{
  id:                 string,   // uuid — never changes
  title:              string,
  categoryId:         string,   // references Category.id
  priority:           1|2|3|4|5, // 1=Low … 5=Critical
  dueDate:            ISO8601 | null,
  notes:              string,
  status:             'active' | 'completed',
  createdAt:          ISO8601,
  updatedAt:          ISO8601,
  completedAt:        ISO8601 | null,
  skippedCount:       number,   // times passed in Zen mode
  timeSpent:          number,   // total seconds tracked
  timerType:          'stopwatch' | 'countdown' | 'pomodoro',
  timerRunning:       boolean,
  timerStartedAt:     timestamp | null,  // Date.now() value
  timerCountdownFrom: number,   // seconds
  pomodoroCount:      number,   // completed pomodoros
  pomodoroPhase:      'work' | 'break',
  lastSynced:         ISO8601 | null,
  fieldModifications: object,   // reserved for cloud sync
}

// Category shape
{
  id:    string,
  name:  string,
  color: string,  // hex
  icon:  string,  // emoji
}
```

---

## Section 5: UI / UX Rules

- No `alert()`, `confirm()`, or `prompt()`
- In-app status messages only (toast or header badges)
- No auto-navigation after save — user controls navigation
- Separate labeled buttons — not combined
- No sticky headers/footers unless explicitly requested
- No instruction boxes, tutorial overlays, or hand-holding text
- Clean, minimal UI

---

## Section 6: Firestore / Sync Rules

- Collection path: `apps/life_app_v3/[collection]`
- Manual sync only — no real-time `onSnapshot` listeners
- Field-level conflict detection using `lastSynced` timestamps
- User resolves conflicts manually via UI
- `isUpdatingFromSync` ref guards against save loops

---

## Section 7: iOS Best Practices & Template Standards

> **This section is MANDATORY reading before any code change.**

### 7.1 Immediate Save Pattern
ALL data mutations must run `localStorage.setItem()` BEFORE `setState()`. No exceptions.

```javascript
const immediatelySaveToLocalStorage = (updatedData) => {
    try {
        localStorage.setItem('life_app_v3_data', JSON.stringify(updatedData));
        console.log('💾 IMMEDIATE save to localStorage');
        return true;
    } catch (error) {
        if (error.name === 'QuotaExceededError' || error.code === 22) {
            console.error('⚠️ Storage full! Cannot save.');
        }
        return false;
    }
};
```

### 7.2 Media Handling
- Store files in IndexedDB, thumbnails/metadata in localStorage
- Mark pending uploads with `_pendingSync: true`
- Restore blob URLs on app load (they die on kill)

### 7.3 React Hooks
- Never call hooks inside loops, conditionals, or `.map()`
- Per-item stateful components must be extracted as named components

### 7.4 Data Safety
- Never rename or remove fields — add new fields with defaults only
- Old saved data must always load without errors

### 7.5 Kill Test
Every data-related change must pass:
1. Make change
2. Immediately kill app (swipe up)
3. Reopen
4. Data persists
5. Repeat x3

### 7.6 Patterns Discovered During Build
> _(Add new iOS-specific patterns here as they are discovered)_

---

## Section 8: Out of Scope

> _(Document explicitly rejected features here)_

| Feature | Why excluded |
|---------|-------------|
| _(add as needed)_ | |
