# Life App V3: AI Development Context & Rules

> **PURPOSE:** This app is built on a proven iOS PWA template. Every pattern here eliminates trial-and-error. Follow them exactly.

---

## 🎯 ALWAYS-ON CONTEXT

### Project Identity
- **App Type:** Progressive Web App (PWA)
- **Target Platform:** iOS Safari (primary), Android Chrome (secondary)
- **Architecture:** Single-file React app (Babel in-browser), offline-first, localStorage + Firebase/Firestore cloud sync
- **Storage Key:** `life_app_v3_data` — IMMUTABLE, never change
- **Firestore Path:** `apps/life_app_v3/[collection]`
- **Critical Constraint:** Must survive instant iOS app termination (swipe-up kill)

### Source of Truth Hierarchy
1. **PRD.md Section 7** — iOS Best Practices & Template Standards (MANDATORY)
2. **PRD.md Sections 1–6** — Product requirements, data structure, UI rules
3. **TODO.md** — Feature roadmap and task tracking
4. **This file** — Patterns, workflow, lessons learned

---

## 🔒 LOCKED DECISIONS (FILL AS YOU BUILD)

These are explicitly requested by the user. Never reverse them, even when refactoring.

| Area | Decision | What NOT to do |
|------|----------|----------------|
| _(add decisions here as they are made)_ | | |

> **Rule:** Before changing anything, check if it is locked. If user requests a change to a locked decision, UPDATE THIS TABLE FIRST, then touch the code.

---

## ⚠️ MANDATORY PATTERNS — NEVER VIOLATE

### 1. IMMEDIATE SAVE PATTERN (iOS Safety) — #1 MOST IMPORTANT

**Why:** iOS kills PWAs instantly on swipe-up. React setState is async and will NOT complete in time.

```javascript
// Helper (add once near top of App component)
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

// EVERY data mutation uses this pattern:
const updateSomething = (newValue) => {
    const updated = { ...appData, field: newValue };
    immediatelySaveToLocalStorage(updated);  // 1. SYNC save
    setAppData(updated);                      // 2. THEN async state
};
```

Applies to:
- ✅ All form inputs (text, checkboxes, dropdowns, toggles)
- ✅ Photo/video uploads
- ✅ Status changes
- ✅ Notes, timestamps, selections
- ✅ Timer states, custom fields
- ❌ NEVER skip — there are no "simple" updates on iOS

---

### 2. MEDIA HANDLING (Local-First)

```javascript
// Pattern: IndexedDB for files, localStorage for metadata
await savePendingUpload(itemId, file, thumbnail);
immediatelySaveToLocalStorage({
    ...appData,
    photos: [...photos, { id, thumbnail, mediaType, _pendingSync: true }]
});
setAppData(updated);

// On app load — restore blob URLs (they die when app is killed)
useEffect(() => {
    const restorePendingMedia = async () => {
        const pending = await getPendingUploads();
        // Recreate blob URLs for videos
        // Restore thumbnails for photos
    };
    setTimeout(restorePendingMedia, 1000);
}, []);
```

---

### 3. DATA STRUCTURE PRESERVATION

- ❌ NEVER rename fields in saved data objects (breaks existing data)
- ❌ NEVER remove fields (causes data loss on load)
- ❌ NEVER change the structure of your primary data without a migration plan
- ✅ Add new fields with defaults (backwards compatible)
- ✅ Old saved data must always load correctly

---

### 4. REACT HOOKS RULES

- ❌ Never call `useState` / `useEffect` inside `.map()`, loops, or conditionals
- ✅ All hooks must be at the top level of a component
- ✅ Components that need per-item state must be extracted into standalone named components

---

### 5. CLOUD SYNC ARCHITECTURE

```
Local changes → immediatelySaveToLocalStorage() → setAppData()
                        ↓ (explicit user action only)
                 syncToCloud()
                        ↓
            Conflict detection (field-level timestamps)
                        ↓
            User resolves conflicts (local vs cloud per field)
                        ↓
            Push resolved data to Firestore
```

- ❌ Disable real-time `onSnapshot` listeners — they cause duplicate record bugs
- ✅ Manual sync only (explicit button press)
- ✅ `isUpdatingFromSync` ref prevents sync pulls from re-triggering the save `useEffect`

---

### 6. NOTIFICATION POLICY

- ❌ No `alert()` — interrupts workflow
- ❌ No `confirm()` — same
- ❌ No `prompt()` — same
- ❌ No lengthy help text boxes or tutorial overlays in the UI
- ✅ `console.log()` for all status and debug messages
- ✅ In-app status badges (⏱ pending, ☁ synced, ✓ saved)
- ✅ Brief inline status text (1–2 words max)
- ✅ Visual indicators (colors, icons, progress bars)

---

### 7. FEATURE AUTHORIZATION

- ❌ No features, text, labels, or UI elements without explicit user request
- ❌ No "helpful extras" or interpretations beyond what was asked
- ✅ When in doubt, ask before adding anything

---

## 📋 DEVELOPMENT WORKFLOW

### Starting Every Session
1. Read this file + PRD.md Section 7
2. Check TODO.md for current priorities
3. Review `git log --oneline -10` if resuming after a break

### Implementing a Feature
1. Understand the goal
2. Identify which mandatory pattern applies
3. Implement iOS-safe (immediate save, IndexedDB if media)
4. Think: will this survive an instant app kill?
5. If new pattern discovered, document it in PRD Section 7

### Bug Fixing
1. What does the user observe?
2. Which mandatory pattern was skipped?
3. Apply the correct pattern
4. Verify with kill test

---

## 🧪 TESTING PROTOCOL

### Kill Test (CRITICAL — run for every data-related change)
1. Make a change (edit text, tap checkbox, upload photo)
2. Immediately swipe up to kill the app
3. Reopen the app
4. Data must still be there
5. Repeat 3 times

### Standard Checklist
- [ ] Works in Chrome (desktop)
- [ ] Works in Safari PWA (iPhone)
- [ ] No console errors
- [ ] Data persists on page reload
- [ ] Offline mode functional
- [ ] Kill test passes (x3)
- [ ] Storage quota exceeded handled gracefully

---

## 📝 CODE STYLE

### Storage Keys — IMMUTABLE once data is in production
```javascript
const STORAGE_KEY = 'life_app_v3_data'; // NEVER change this
```

### Console Log Conventions
```javascript
console.log('🔥 App init')
console.log('💾 Saved to localStorage')
console.log('📤 Pushing to cloud')
console.log('📥 Pulling from cloud')
console.log('✅ Success')
console.log('⚠️ Warning')
console.log('❌ Error')
console.log('📍 View changed to:', view)
console.log('🔄 Sync started')
console.log('📴 Offline / sync disabled')
```

### Function Naming Conventions
```javascript
immediatelySaveToLocalStorage()  // iOS safety helper
syncToCloud()                    // Cloud push
loadFromCloud()                  // Cloud pull
syncBidirectional()              // Push then pull
updateItem()                     // Data mutation
handleExport()                   // User-triggered action
saveAndNavigate()                // Navigate with guaranteed local save
```

---

## 🗂️ DATA PATTERNS

### Primary Data Shape
```javascript
// { [year]: Record[] }

// Each Record:
{
    id: string,          // uuid — never changes
    name: string,
    lastSynced: ISO8601, // conflict detection timestamp
    year: number,
    data: { [sectionId]: Item[] },
    fieldModifications: { [fieldPath]: { value, timestamp, user } }
}
```

### Conflict Detection (Field-Level)
```javascript
// On sync: check each local modification
if (cloudMod.timestamp > localRecord.lastSynced) {
    if (JSON.stringify(localValue) !== JSON.stringify(cloudValue)) {
        conflicts.push({ fieldPath, localValue, cloudValue, ... });
    }
}
// Show resolution UI — user picks local or cloud per field
```

### Protected Records
- Some records should always exist (like default entries)
- Guard them: restore on first load, never delete during sync/merge
- Use id-based lookup, not name-based

---

## 🎓 LESSONS LEARNED — DO NOT REPEAT

### ❌ NEVER
- Rely on React state for data persistence (iOS kills too fast)
- Upload large files before saving locally (causes silent data loss)
- Use blob URLs without IndexedDB backup (become invalid on app kill)
- Skip immediate save for "simple" changes (they all matter)
- Assume iOS gives time for async operations (it doesn't)
- Enable real-time Firestore `onSnapshot` without careful dedup logic (causes duplicate records)
- Change saved data field names (breaks existing user data)
- Remove fields from data structures (data loss)
- Add unauthorized UI text, buttons, or features
- Use `alert()`, `confirm()`, or `prompt()` anywhere

### ✅ ALWAYS
- Save synchronously to localStorage before setState
- Test kill scenario: change something → immediately kill → reopen → verify
- Provide visual feedback for async operations (badges, progress)
- Handle `QuotaExceededError` from localStorage gracefully
- Restore pending IndexedDB media on app load
- Preserve backwards compatibility — old saved data must always load
- Keep UX clean and minimal (no instruction boxes, no hand-holding)
- Ask user before adding any feature or content not explicitly requested

---

## 🔄 QUICK REFERENCE PATTERNS

### Data Mutation
```javascript
const updateField = (newValue) => {
    const updated = { ...appData, field: newValue };
    immediatelySaveToLocalStorage(updated);
    setAppData(updated);
};
```

### Cloud Push (after local save)
```javascript
const syncToCloud = async () => {
    // 1. Pull cloud to detect conflicts
    // 2. Compare field timestamps
    // 3. If conflicts: show resolution UI, halt
    // 4. If no conflicts: merge + push all records
    // 5. Update lastSynced on all records
    // 6. isUpdatingFromSync.current = true
    // 7. setAppData(mergedData)
};
```

### Navigation with Save
```javascript
const saveAndNavigate = (targetView) => {
    // Guarantee local save before navigation (race condition prevention)
    localStorage.setItem('life_app_v3_data', JSON.stringify(appData));
    setView(targetView);
};
```

### Sync Guard (prevent save loop)
```javascript
// In useEffect that watches appData:
if (isUpdatingFromSync.current) {
    isUpdatingFromSync.current = false;
    return; // Don't mark cloud pulls as local changes
}
```

---

## 📞 WHEN IN DOUBT
1. Check: does a pattern for this already exist in this file or PRD Section 7?
2. Apply iOS-safe pattern (immediate save first, state second)
3. Verify: will this survive instant app kill?
4. If structural change (data shape, field names): ask user before proceeding
