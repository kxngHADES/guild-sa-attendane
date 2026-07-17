# Data Persistence & State Management — Guild Attendance Register

> **Location**: `index.html` lines 810–1039 (single IIFE module)

## Overview
- In-memory `students` array + `localStorage` sync
- Keys: `guild_sa_attendance_list`, `guild_sa_app_settings`
- Schema: id, studentNumber, fullName, isPresent, checkInTime, lastModified
- Auto-load on `DOMContentLoaded`, auto-save on every mutation

## Storage Schema
| Key | Value |
|-----|-------|
| `guild_sa_attendance_list` | `Student[]` |
| `guild_sa_app_settings` | `{ darkMode: boolean }` |

### Student Object
```js
{
  id: "timestamp_random",
  studentNumber: "STU-001",      // Primary key (unique)
  fullName: "Alice Johnson",
  isPresent: false,
  checkInTime: null,             // ISO string when present
  lastModified: "2026-07-17T..."
}
```

## API Reference (`window.App`)

| Method | Signature | Returns |
|--------|-----------|---------|
| **Read** |
| `getAll()` | `()` | `Student[]` |
| `getById(id)` | `(id: string)` | `Student \| undefined` |
| `getByNumber(num)` | `(num: string)` | `Student \| undefined` |
| **Write** |
| `add(number, name)` | `(string, string)` | `{ok, student} \| {ok: false, error}` |
| `togglePresence(id)` | `(id: string)` | `{ok, student} \| {ok: false}` |
| `update(id, patches)` | `(id, Partial<Student>)` | `{ok, student} \| {ok: false}` |
| `remove(id)` | `(id: string)` | `{ok: boolean}` |
| `clearAll()` | `()` | `{ok: true}` |
| **Stats** |
| `getStats()` | `()` | `{total, present, absent, percent}` |
| **Theme** |
| `setTheme(dark)` | `(boolean)` | `void` |
| `getTheme()` | `()` | `boolean` |

## Lifecycle

```mermaid
flowchart TD
  A[DOMContentLoaded] --> B[loadData()]
  B --> C[Parse localStorage]
  C --> D[normalizeStudent() each]
  D --> E[students[] in memory]
  E --> F[renderDashboardStats()]
  F --> G[UI Ready]
  
  H[Any Mutation] --> I[saveData()]
  I --> J[localStorage.setItem]
```

## Initialization
```js
document.addEventListener('DOMContentLoaded', init);
function init() { loadData(); }
```

## Error Handling
- `try/catch` on all `localStorage` ops
- Failures logged to console, empty array fallback
- No UI toast (UI layer responsibility)

## Dev Helper
```js
App.__devTestPersist()  // Adds test student, reloads, verifies persistence
```

## Integration Points
| UI Event | Data Call |
|----------|-----------|
| Registration submit | `App.add()` |
| Check-in toggle | `App.togglePresence()` |
| Dashboard render | `App.getStats()` |
| Attendance list | `App.getAll()` |
| Quick check-in | `App.getByNumber()` + `togglePresence()` |
| Settings theme | `App.setTheme()` / `getTheme()` |
| Reset All | `App.clearAll()` |

## Testing Checklist
- [ ] `App.add()` persists to localStorage
- [ ] Refresh → `App.getAll()` returns saved data
- [ ] `App.togglePresence()` updates `checkInTime`
- [ ] `App.getStats()` matches array state
- [ ] Duplicate studentNumber rejected
- [ ] Theme persists across reloads