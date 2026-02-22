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

> _(List features as they are explicitly requested and built)_

| Feature | Status | Notes |
|---------|--------|-------|
| _(add features here)_ | | |

---

## Section 4: Data Structure

> _(Document the finalized data shape here as it is decided)_

```javascript
// Primary storage key: life_app_v3_data
// Shape TBD — fill in as data structure is designed
{}
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
