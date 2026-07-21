# GUILD SA Student Attendance Register Tool

## Summary

A single-file `index.html` browser-based attendance register for Buildathon 01, built with vanilla HTML/CSS/JS and localStorage persistence. No backend, frameworks, or build steps permitted.

**Core Features:** Add students (duplicate prevention via student number), check-in/out with timestamps, searchable register, real-time dashboard (Total Registered, Present, Absent, Attendance %), edit/remove/reset records.

**Extended Features:** QR check-in concept, CSV export, responsive mobile layout with bottom navigation, dark/light theme toggle, audit trail.

**Data Schema:** Student objects `{id, studentNumber (PK), fullName, isPresent, checkInTime, lastModified}` stored in `guild_sa_attendance_list`; theme in `guild_sa_app_settings`.

**UI:** Four-view SPA (Dashboard, Register, Attendance, Settings) switched via `currentView` state. Mobile-first fixed bottom nav. Student cards show status badge, timestamp, toggle/edit/remove actions.

**Logic:** Registration validates + duplicate checks → localStorage. Attendance toggles presence + ISO timestamp → localStorage. Stats computed via `Array.filter()`. Management uses filter/find/map.

**Deadline:** Wednesday, 22 July 2026, 12:00 PM. Deliverables: `index.html`, ≤200-word technical summary, AI disclosure statement.

---

## Technical Summary

**Guild Attendance Register** is a single-file (`index.html`) browser-based SPA for event attendance tracking, built with vanilla HTML/CSS/JS and localStorage persistence—no backend, frameworks, or build steps.

**Architecture**: Single IIFE module exposes `window.App` API. State lives in-memory (`students[]`, `activityLog[]`, `events[]`) synced to localStorage on every mutation. Multi-event support isolates data per event via namespaced keys (`guild_sa_attendance_{id}`, `guild_sa_settings_{id}`, `guild_sa_activity_{id}`) with a registry (`guild_sa_events`) tracking event metadata and active event ID. Legacy keys auto-migrate on first load.

**Data Model**: Student records `{id, studentNumber (PK), fullName, isPresent, isLate, checkInTime (ISO), lastModified}`. Activity log captures audit trail (register/checkin/checkout/edit/remove) capped at 50 entries. Settings per event: dark mode, late cutoff time, auto-backup toggle.

**Core Flows**: Registration validates required fields, prevents duplicate studentNumbers, normalizes schema. Attendance toggles presence, records ISO timestamp, computes late status against configurable cutoff. Dashboard stats derive via `Array.filter()` in real time. Render-on-mutation re-renders affected views (list ≤1000 rows, performant).

**Extended Features**: CSV export (RFC 4180), bulk import (paste/file drag-drop, custom parser), JSON backup/restore (merge/replace), auto-backup snapshot on change with restore prompt, print-optimized A4 report (`@media print`), responsive mobile-first UI with fixed bottom nav, dark/light theme, full ARIA/keyboard accessibility.

**Compliance**: Single `index.html`, vanilla stack, works offline on open—meets all Buildathon 01 constraints.

---

## AI & External Tool Disclosure

This project was developed with assistance from local coding AI agents in OpenCode connected to Ollama for code generation, and documentation. All code was reviewed, tested, and integrated manually. Architectural planning was not done by AI. No external libraries, frameworks, or build tools were used—implementation is pure vanilla HTML/CSS/JS as required by the Buildathon constraints.