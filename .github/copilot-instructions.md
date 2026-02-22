# Life App V3 — Auto-loaded Copilot Instructions

This is a single-file React PWA (`index.html`). Firebase + Firestore backend. Primary target: iOS Safari.

---

## 🔒 LOCKED DECISIONS — DO NOT REVERT

| Area | Decision | Never do this |
|------|----------|---------------|
| _(add as you make explicit decisions)_ | | |

> **Rule:** When the user makes an explicit UI/UX/behavior decision, add it to this table BEFORE touching the code. Never revert a locked decision even during refactors.

---

## 📖 READ THESE DOCS FIRST — EVERY SESSION

Before making any code changes, read:

1. **[docs/COPILOT-INSTRUCTIONS.md](../docs/COPILOT-INSTRUCTIONS.md)** — full patterns, iOS safety rules, lessons learned
2. **[docs/PRD.md](../docs/PRD.md)** — product requirements (Section 7: iOS Best Practices is critical)
3. **[docs/TODO.md](../docs/TODO.md)** — current priorities

If context is summarized and these files haven't been read yet this session, read them before proceeding.

---

## 🎯 PLATFORM TARGET

- **Primary:** iOS Safari PWA (installed to home screen)
- **Secondary:** Android Chrome
- This is NOT a native app and NOT a desktop web app
- All UI, notifications, storage, and API decisions must be validated against iOS Safari PWA behavior
- Never assume desktop browser behavior applies

---

## 👤 USER PREFERENCES — DO NOT CHANGE WITHOUT BEING ASKED

| Preference | Current default | Change when |
|------------|----------------|-------------|
| Notifications / alerts | In-app status messages (toast, header ✓). No `alert()`, `confirm()`, or `prompt()` | User explicitly asks for a different pattern |
| Navigation after save | User controls navigation. No auto-navigate after save | User explicitly asks |
| Button layout | Separate labeled buttons. Not combined | User explicitly asks |
| Sticky UI elements | Inline/non-sticky by default | User explicitly asks for sticky on specific element |
| Data mutations | Save to localStorage synchronously BEFORE calling setState | Never — this is an iOS safety requirement, not a preference |

---

## ❓ ASK BEFORE ASSUMING

**Always ask the user when:**
- A platform API behaves differently on iOS Safari vs. desktop (notifications, file access, storage, etc.)
- There are two valid implementation approaches with different UX tradeoffs
- The existing code doesn't show a clear pattern for what's being added
- A change would affect navigation, data persistence, or notifications

**Never silently assume and implement** platform-specific behavior.

---

## ⚠️ MANDATORY PATTERNS

### iOS Safety — Immediate Save
ALL data mutations must save to localStorage synchronously BEFORE calling setState:
```js
const updated = { ...appData, field: newValue };
localStorage.setItem('life_app_v3_data', JSON.stringify(updated)); // sync first
setAppData(updated); // then react state
```

Applies to:
- ✅ All form inputs (text, checkboxes, dropdowns, toggles)
- ✅ Photo/video uploads
- ✅ Status changes
- ✅ Notes, timestamps, selections
- ✅ Timer states, custom fields
- ❌ NEVER skip — there are no "simple" updates on iOS

### React Hooks
Never call `useState` / `useEffect` inside `.map()`, loops, or conditionals. Must be at component top level.

### Data Structure
Never rename or remove fields from saved data objects — breaks existing saved data. Add new fields with defaults only.

### Feature Authorization
No features, text, UI elements, or functionality may be added unless explicitly requested by the user. When in doubt, ask.

---

## 🗝️ Key Architecture
- Primary data shape: `{ [key]: items[] }` — adapt to your app's domain
- Cloud sync: Firestore collection `apps/life_app_v3/[collection]` — one doc per record
- `lastSynced` on each record is the conflict detection timestamp
- Photos stored as base64 thumbnails in `item.photos[]`, full files in IndexedDB with `_pendingSync: true`
- `isUpdatingFromSync` ref prevents the data `useEffect` from marking cloud pulls as local changes
- Real-time sync **DISABLED** by default — manual sync only (prevents duplicate record bugs)

---

## 📚 Full Docs
- [docs/COPILOT-INSTRUCTIONS.md](../docs/COPILOT-INSTRUCTIONS.md)
- [docs/PRD.md](../docs/PRD.md)
- [docs/TODO.md](../docs/TODO.md)
