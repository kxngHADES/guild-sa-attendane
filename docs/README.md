# GUILD SA Student Attendance Register Tool

## Summary

A single-file `index.html` browser-based attendance register for Buildathon 01, built with vanilla HTML/CSS/JS and localStorage persistence. No backend, frameworks, or build steps permitted.

**Core Features:** Add students (duplicate prevention via student number), check-in/out with timestamps, searchable register, real-time dashboard (Total Registered, Present, Absent, Attendance %), edit/remove/reset records.

**Extended Features:** QR check-in concept, CSV export, responsive mobile layout with bottom navigation, dark/light theme toggle, audit trail.

**Data Schema:** Student objects `{id, studentNumber (PK), fullName, isPresent, checkInTime, lastModified}` stored in `guild_sa_attendance_list`; theme in `guild_sa_app_settings`.

**UI:** Four-view SPA (Dashboard, Register, Attendance, Settings) switched via `currentView` state. Mobile-first fixed bottom nav. Student cards show status badge, timestamp, toggle/edit/remove actions.

**Logic:** Registration validates + duplicate checks → localStorage. Attendance toggles presence + ISO timestamp → localStorage. Stats computed via array.filter(). Management uses filter/find/map.

**Deadline:** Wednesday, 22 July 2026, 12:00 PM. Deliverables: `index.html`, ≤200-word technical summary, AI disclosure statement.