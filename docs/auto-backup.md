# Auto-Backup System — Guild Attendance Register

> **Location**: `index.html` lines 2287–2337 (inside IIFE module)  
> **Settings UI**: `view-settings` section → "Auto Backup" fieldset  
> **Last Updated**: 2026-07-19

---

## Table of Contents

### For Developers
1. [Architecture & Data Flow](#architecture--data-flow)
2. [Storage Keys](#storage-keys)
3. [Backup Schema](#backup-schema)
4. [Trigger Points](#trigger-points)
5. [Restore Flow](#restore-flow)
6. [API Reference (`window.App`)](#api-reference-windowapp)
7. [Integration Points](#integration-points)
8. [Testing Checklist](#testing-checklist)
9. [Limitations & Known Issues](#limitations--known-issues)

### For End Users
1. [What Is Auto-Backup?](#what-is-auto-backup)
2. [How to Enable/Disable](#how-to-enabledisable)
3. [What Gets Backed Up](#what-gets-backed-up)
4. [How Restore Works](#how-restore-works)
5. [Manual Backup vs Auto-Backup](#manual-backup-vs-auto-backup)
6. [Troubleshooting](#troubleshooting)

---

## For Developers

### Architecture & Data Flow

```mermaid
flowchart TD
  A[User toggles Auto-Backup ON] --> B[localStorage.setItem guild_sa_auto_backup=true]
  C[Any data mutation] --> D[saveData() / saveActivityLog()]
  D --> E{Auto-backup enabled?}
  E -->|Yes| F[createAutoBackup()]
  F --> G[localStorage.setItem guild_sa_auto_backup_data]
  E -->|No| H[Skip backup]
  I[App init: students.length === 0] --> J{Auto-backup exists?}
  J -->|Yes| K[prompt 'Restore auto-backup?']
  K -->|Yes| L[Restore students + activityLog]
  L --> M[saveData() + saveActivityLog()]
  K -->|No| N[Delete auto-backup?]
```

### Storage Keys

| Key | Type | Description |
|-----|------|-------------|
| `guild_sa_auto_backup` | `string` ("true"/"false") | Toggle preference, persists 365 days via cookie-like storage |
| `guild_sa_auto_backup_data` | `string` (JSON) | Full backup payload, overwritten on each trigger |

> **Note**: These are separate from main data keys (`guild_sa_attendance_list`, `guild_sa_activity_log`, `guild_sa_app_settings`).

### Backup Schema

```json
{
  "version": 1,
  "exportedAt": "2026-07-19T10:56:25.072Z",
  "students": [
    {
      "id": "2026-07-17T20:30:56.984Z_fz7iad4",
      "studentNumber": "STU-002",
      "fullName": "Bra",
      "isPresent": true,
      "checkInTime": "2026-07-17T20:31:16.480Z",
      "lastModified": "2026-07-17T20:31:16.480Z"
    }
  ],
  "activityLog": [
    {
      "type": "register",
      "studentName": "Brad",
      "studentNumber": "PTA2022",
      "timestamp": "2026-07-19T10:56:25.072Z"
    }
  ],
  "settings": {
    "darkMode": true,
    "autoBackup": true
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `version` | Yes | Schema version (currently 1) |
| `exportedAt` | Yes | ISO timestamp of backup creation |
| `students` | Yes | Array of student objects (full schema) |
| `activityLog` | Yes | Array of activity entries (max 50) |
| `settings.darkMode` | No | Theme preference at backup time |
| `settings.autoBackup` | No | Auto-backup toggle state at backup time |

### Trigger Points

Auto-backup runs **synchronously** after these functions when toggle is ON:

| Function | Called From |
|----------|-------------|
| `saveData()` | `dataAPI.add()`, `dataAPI.update()`, `dataAPI.remove()`, `dataAPI.clearAll()`, `dataAPI.togglePresence()` |
| `saveActivityLog()` | `logActivity()` (register, checkin, checkout, edit, remove) |

**Implementation** (lines 2271–2285):
```javascript
const originalSaveData = saveData;
saveData = function() {
  originalSaveData();
  if (autoBackupToggle?.checked) createAutoBackup();
};
// Same pattern for saveActivityLog
```

### Restore Flow

Runs **once** on app initialization, **before** `loadData()` (line 2416):

```javascript
function init() {
  checkAutoBackupRestore();  // ← Runs FIRST
  loadData();                // Then loads main data
  renderDashboardStats();
  renderAttendeeList();
}
```

**Logic** (`checkAutoBackupRestore`, lines 2306–2337):
1. Check if `guild_sa_auto_backup_data` exists
2. Parse JSON, validate `backup.students?.length > 0`
3. Check if main data store is empty: `localStorage.getItem(STORAGE_KEYS.STUDENTS)` is empty/null
4. If both true → show `confirm()` dialog with backup timestamp
5. If user confirms → push students to array, merge activityLog, call `saveData()` + `saveActivityLog()`

### API Reference (`window.App`)

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `getAutoBackup()` | `()` | `boolean` | Read toggle state from cookie |
| `setAutoBackup(enabled)` | `(boolean)` | `void` | Write toggle state to cookie |
| `createAutoBackup()` | `()` | `void` | Force immediate backup (exposed for testing) |

> **Note**: `getAutoBackup`/`setAutoBackup` use cookie storage (not localStorage) for consistency with theme persistence. The toggle UI reads/writes `localStorage.setItem('guild_sa_auto_backup', ...)`.

### Integration Points

| UI Element | Action |
|------------|--------|
| Settings → "Auto Backup" toggle | Writes `guild_sa_auto_backup` to localStorage |
| Any data change (add/edit/delete/check-in) | Triggers `createAutoBackup()` if enabled |
| App load with empty data | Prompts restore via `checkAutoBackupRestore()` |
| Manual JSON Backup/Restore modal | Independent; does NOT affect auto-backup |

### Testing Checklist

- [ ] Toggle ON → `localStorage.getItem('guild_sa_auto_backup') === 'true'`
- [ ] Add student → `guild_sa_auto_backup_data` updated with new student
- [ ] Check-in student → `activityLog` in backup includes `checkin` entry
- [ ] Toggle OFF → No new backups created on data changes
- [ ] Clear main data (`localStorage.removeItem('guild_sa_attendance_list')`) → Reload → Restore prompt appears
- [ ] Restore prompt → Click OK → Students + activity log restored
- [ ] Restore prompt → Click Cancel → Auto-backup preserved (can retry on next reload)
- [ ] `App.createAutoBackup()` in console → Backup updates immediately
- [ ] `App.getAutoBackup()` returns correct boolean

### Limitations & Known Issues

| Limitation | Impact |
|------------|--------|
| Single backup slot | Overwrites previous; no history/rotation |
| No size limit check | Large datasets may hit `localStorage` quota (~5MB) |
| Synchronous backup | Blocks UI briefly on large datasets |
| Restore only on empty data | Won't prompt if any students exist in main store |
| No conflict resolution | Restore blindly pushes; duplicates possible if data not truly empty |
| Cookie vs localStorage mismatch | Toggle UI uses localStorage; API uses cookies |

---

## For End Users

### What Is Auto-Backup?

Auto-Backup automatically saves a complete copy of your attendance data (students, check-in history, and settings) to your browser's local storage **every time you make a change**. If you accidentally clear your data or switch devices, you can restore everything with one click.

### How to Enable/Disable

1. Go to **Settings** (bottom navigation)
2. Find the **"Auto Backup"** section
3. Toggle **"Automatically save backup to localStorage on changes"**
   - **ON** (green): Creates backup after every change
   - **OFF** (gray): No automatic backups

> Your preference is remembered across browser sessions.

### What Gets Backed Up

| Data | Included? |
|------|-----------|
| Student list (numbers, names) | ✅ Yes |
| Check-in status & times | ✅ Yes |
| Recent activity log (last 50 actions) | ✅ Yes |
| Dark/Light theme setting | ✅ Yes |
| Auto-backup toggle state | ✅ Yes |
| CSV exports / manual JSON backups | ❌ No (separate features) |

### How Restore Works

**Scenario**: You open the app and see **zero students** (fresh browser, cleared storage, or new device).

1. App detects auto-backup exists
2. Shows dialog: *"An auto-backup was found (Jul 19, 2026, 10:56 AM). Restore it?"*
3. Click **OK** → All students, check-ins, and history restored instantly
4. Click **Cancel** → Backup kept; prompt appears again on next visit

> **Important**: Restore only offers if your current student list is completely empty. If you have even 1 student, it assumes you're working with current data.

### Manual Backup vs Auto-Backup

| Feature | Auto-Backup | Manual JSON Backup |
|---------|-------------|-------------------|
| **Trigger** | Automatic on every change | Manual: Settings → "Backup Data (JSON)" |
| **Format** | localStorage (browser only) | `.json` file (downloadable) |
| **Portability** | Same browser only | Transferable (email, USB, cloud) |
| **Restore** | Auto-prompt on empty data | Manual: Settings → "Restore Data (JSON)" |
| **History** | Single latest only | Unlimited files you manage |

**Recommendation**: Use **both**. Auto-backup for daily safety; manual JSON backup weekly or before major changes.

### Troubleshooting

| Problem | Solution |
|---------|----------|
| "Auto-backup was found" prompt doesn't appear | You already have students in the list. Restore only triggers when list is empty. |
| Restored data looks old | Auto-backup updates on every change. Check `exportedAt` timestamp in backup. |
| Backup not creating | Check Settings → Auto Backup is ON. Check browser storage not full. |
| "Failed to parse backup" error | Backup file corrupted. Try manual JSON restore instead. |
| Storage full / quota exceeded | Clear old data or disable auto-backup. Browser limit ~5MB. |

---

## Quick Console Reference (Dev)

```javascript
// Check toggle state
document.getElementById('auto-backup-toggle').checked
// or
App.getAutoBackup()

// Force backup now
App.createAutoBackup()

// View backup content
JSON.parse(localStorage.getItem('guild_sa_auto_backup_data'))

// Simulate data loss (KEEPS auto-backup)
localStorage.removeItem('guild_sa_attendance_list');
localStorage.removeItem('guild_sa_activity_log');
location.reload();
// → Should see restore prompt

// Clear everything including auto-backup
localStorage.removeItem('guild_sa_auto_backup_data');
localStorage.removeItem('guild_sa_auto_backup');
```