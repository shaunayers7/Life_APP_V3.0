# Life App V3 — TODO

> Update this file at the start and end of every session.  
> Current date: 2026-02-23

---

## 🔴 In Progress

_(nothing active)_

---

## 🟡 Up Next

- [ ] Category management — delete category (with task reassignment prompt)
- [ ] Firebase Auth + Firestore sync (when ready)
- [ ] OpenAI task suggestion layer on top of rule scoring
- [ ] PWA manifest + service worker for install prompt

---

## 🟢 Backlog

- [ ] Field-level conflict resolution UI
- [ ] IndexedDB media handling
- [ ] Offline indicator
- [ ] Export / backup feature
- [ ] PWA manifest + service worker

---

## ✅ Completed

- [x] Set up docs/ folder + copilot-instructions files — 2026-02-22
- [x] Scaffold index.html with React + Babel (single file) — 2026-02-22
- [x] Implement `immediatelySave` helper (iOS safe) — 2026-02-22
- [x] localStorage load on app init — 2026-02-22
- [x] Zen Focus Mode with fade transition + Pass — 2026-02-22
- [x] Task scoring algorithm (priority + due date + neglect + age) — 2026-02-22
- [x] Dashboard with category filter + completion toggle — 2026-02-22
- [x] Add/Edit task form (title, category, priority, due date, notes) — 2026-02-22
- [x] WorkView with stopwatch / countdown / Pomodoro timers — 2026-02-22
- [x] Settings view (Zen count, Pomodoro durations, stats) — 2026-02-22
- [x] Toast notification system (no alert/confirm/prompt) — 2026-02-22
- [x] Default categories: Work, Health, Home, Finance, Personal, Projects — 2026-02-22
- [x] Edit button in WorkView header → opens AddTask in edit mode, returns to WorkView — 2026-02-23
- [x] Task delete via long-press on TaskCard in Dashboard — 2026-02-23
- [x] Category management in Settings (rename inline, reorder up/down, add new) — 2026-02-23

---

## 🚫 Decided Against

_(features explicitly rejected — include reason)_

| Feature | Reason | Date |
|---------|--------|------|
| Real-time Firestore sync | Causes duplicate record bugs | — |
| `alert()` / `confirm()` / `prompt()` | Interrupts iOS PWA workflow | — |
