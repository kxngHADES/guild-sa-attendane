# CRUD Operations — Guild Attendance Register

> **Location**: `index.html` lines 960–1375 (single IIFE module)

## Overview
Complete CRUD implementation wired to the data persistence layer. All operations use the `dataAPI` (exposed as `window.App`) and automatically sync to `localStorage`.

---

## Create — Student Registration

### UI Entry Point
- **View**: Registration (`#view-registration`)
- **Form**: `#registration-form` (student number + full name)
- **Notice**: `#registration-notice` (success/error feedback)

### Logic (`regForm.addEventListener('submit')`)
```javascript
// Validation
if (!studentNumber || !fullName) → showNotice('Both fields required', 'error')

// Edit mode detection
const isEditMode = regForm.dataset.editMode === 'true'

if (isEditMode) {
  // UPDATE: duplicate check only if number changed
  const existing = dataAPI.getByNumber(studentNumber)
  if (existing && existing.id !== editingStudentId) → error
  dataAPI.update(editingStudentId, { studentNumber, fullName })
} else {
  // CREATE: duplicate prevention
  const result = dataAPI.add(studentNumber, fullName)
  // Returns { ok: false, error: 'Student number already exists' } if duplicate
}
```

### Data API: `add(studentNumber, fullName)`
```javascript
add: (studentNumber, fullName) => {
  // Duplicate prevention: O(n) scan
  if (students.some(s => s.studentNumber === studentNumber)) {
    return { ok: false, error: 'Student number already exists' }
  }
  const now = new Date().toISOString()
  const student = {
    id: `${now}_${Math.random().toString(36).slice(2, 9)}`,
    studentNumber, fullName,
    isPresent: false, checkInTime: null,
    lastModified: now
  }
  students.push(student)
  saveData()  // ← auto-persist
  return { ok: true, student }
}
```

### Duplicate Prevention Strategy
- **Primary key**: `studentNumber` (user-provided, unique)
- **Check**: `Array.some()` before insert
- **Edit mode**: Allows same number for same student, blocks if different student owns it

---

## Read — Dynamic List Rendering

### Dashboard Stats (`renderDashboardStats()`)
```javascript
const stats = dataAPI.getStats()
// { total, present, absent, percent }
document.getElementById('stat-registered').textContent = stats.total
document.getElementById('stat-present').textContent = stats.present
document.getElementById('stat-absent').textContent = stats.absent
document.getElementById('stat-percent').textContent = stats.percent + '%'
```

### Attendance List (`renderAttendeeList()`)
**Input sources:**
- `dataAPI.getAll()` → full student array
- `#attendance-search` value → filter
- `sortAscending` flag → sort order

**Pipeline:**
```javascript
let studentsToRender = dataAPI.getAll()

// 1. Filter (case-insensitive, name + number)
if (searchTerm) {
  studentsToRender = studentsToRender.filter(s =>
    s.fullName.toLowerCase().includes(searchTerm) ||
    s.studentNumber.toLowerCase().includes(searchTerm)
  )
}

// 2. Sort (alphabetical by name)
studentsToRender.sort((a, b) => 
  sortAscending 
    ? a.fullName.localeCompare(b.fullName)
    : b.fullName.localeCompare(a.fullName)
)

// 3. Render → HTML string with template literals
list.innerHTML = studentsToRender.map(student => `
  <div class="attendee-item" data-id="${escapeHtml(student.id)}">
    <div class="attendee-avatar">${escapeHtml(getInitials(student.fullName))}</div>
    <div class="attendee-info">
      <div class="attendee-name">${escapeHtml(student.fullName)}</div>
      <div class="attendee-number">${escapeHtml(student.studentNumber)}</div>
    </div>
    <span class="attendee-status ${student.isPresent ? 'present' : 'absent'}">
      ${student.isPresent ? 'Present' : 'Absent'}
    </span>
    <span class="attendee-time">${formatTime(student.checkInTime)}</span>
    <div class="attendee-actions">
      <button class="btn-toggle" data-id="${student.id}">...</button>
      <button class="btn-edit" data-id="${student.id}">Edit</button>
      <button class="btn-delete danger" data-id="${student.id}">Delete</button>
    </div>
  </div>
`).join('')

// 4. Attach event handlers (delegation not used — list re-renders on every change)
```

**Empty state handling:**
```javascript
if (studentsToRender.length === 0) {
  list.innerHTML = `<div class="attendee-item" style="...">No students found...</div>`
  return
}
```

---

## Update — Attendance Toggle & Edit

### Toggle Presence (Check-in/Out)
**Trigger**: `.btn-toggle` click in attendee list
```javascript
list.querySelectorAll('.btn-toggle').forEach(btn => {
  btn.addEventListener('click', () => {
    dataAPI.togglePresence(btn.dataset.id)
    renderAttendeeList()
    renderDashboardStats()
  })
})
```

**Data API: `togglePresence(id)`**
```javascript
togglePresence: (id) => {
  const student = students.find(s => s.id === id)
  if (!student) return { ok: false }
  student.isPresent = !student.isPresent
  student.checkInTime = student.isPresent ? new Date().toISOString() : null
  student.lastModified = new Date().toISOString()
  saveData()
  return { ok: true, student }
}
```
- **Optimistic UI**: Immediate re-render after mutation
- **Timestamp**: ISO string recorded on check-in, cleared on check-out

### Edit Student
**Flow**: Click "Edit" → populate registration form → switch to Registration view → "Save Changes"
```javascript
function openEditModal(studentId) {
  const student = dataAPI.getById(studentId)
  editingStudentId = studentId
  document.getElementById('student-number').value = student.studentNumber
  document.getElementById('full-name').value = student.fullName
  document.getElementById('reg-heading').textContent = 'Edit Student'
  document.getElementById('btn-add-student').textContent = 'Save Changes'
  document.getElementById('registration-form').dataset.editMode = 'true'
  dataAPI.showView('registration')
}

// On submit (edit mode):
const existing = dataAPI.getByNumber(studentNumber)
if (existing && existing.id !== editingStudentId) → error
dataAPI.update(editingStudentId, { studentNumber, fullName })
closeEditModal()
```

**Data API: `update(id, patches)`**
```javascript
update: (id, patches) => {
  const idx = students.findIndex(s => s.id === id)
  if (idx === -1) return { ok: false }
  students[idx] = { ...students[idx], ...patches, lastModified: new Date().toISOString() }
  saveData()
  return { ok: true, student: students[idx] }
}
```

---

## Delete — Remove Student & Clear All

### Single Delete
**Trigger**: `.btn-delete` click → confirm dialog
```javascript
list.querySelectorAll('.btn-delete').forEach(btn => {
  btn.addEventListener('click', () => {
    if (confirm('Delete this student? This cannot be undone.')) {
      dataAPI.remove(btn.dataset.id)
      renderAttendeeList()
      renderDashboardStats()
    }
  })
})
```

**Data API: `remove(id)`**
```javascript
remove: (id) => {
  const idx = students.findIndex(s => s.id === id)
  if (idx === -1) return { ok: false }
  students.splice(idx, 1)
  saveData()
  return { ok: true }
}
```

### Clear All (Settings)
```javascript
document.getElementById('btn-reset-all').addEventListener('click', () => {
  if (confirm('Delete ALL student records? This cannot be undone.')) {
    dataAPI.clearAll()
    renderAttendeeList()
    renderDashboardStats()
  }
})
```

**Data API: `clearAll()`**
```javascript
clearAll: () => {
  students.length = 0  // mutate in-place (preserves reference)
  saveData()
  return { ok: true }
}
```

---

## Bonus Features

### Quick Check-in (Dashboard)
```javascript
quickCheckin.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') {
    const student = dataAPI.getByNumber(num)
    if (student) {
      dataAPI.togglePresence(student.id)
      quickCheckin.value = ''
      renderDashboardStats()
      renderAttendeeList()
    }
  }
})
```

### Search & Sort (Attendance)
```javascript
// Search: real-time filter on input
document.getElementById('attendance-search').addEventListener('input', renderAttendeeList)

// Sort: toggle A-Z / Z-A
document.getElementById('btn-sort-toggle').addEventListener('click', (e) => {
  sortAscending = !sortAscending
  e.target.textContent = sortAscending ? 'Sort: A–Z' : 'Sort: Z–A'
  renderAttendeeList()
})
```

### CSV Export
```javascript
document.getElementById('btn-export-csv').addEventListener('click', () => {
  const students = dataAPI.getAll()
  const csv = [headers.join(','), ...rows.map(...)]
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
  // trigger download via <a download>
})
```

### Bulk Import (Modal)
```javascript
// Modal created dynamically, appended to body
// Parses CSV lines: "STU-001, John Doe"
// Calls dataAPI.add() per line, reports imported/errors
```

### Theme Toggle
```javascript
themeToggle.addEventListener('change', () => {
  document.documentElement.classList.toggle('dark-theme', themeToggle.checked)
  dataAPI.setTheme(themeToggle.checked)
})
themeToggle.checked = dataAPI.getTheme()  // init from storage
```

---

## Event Handler Architecture

| Handler Type | Pattern | Example |
|--------------|---------|---------|
| **Form submit** | Direct listener + `preventDefault()` | `regForm.addEventListener('submit', ...)` |
| **List actions** | Re-attach on every render (simple, no delegation) | `list.querySelectorAll('.btn-toggle').forEach(...)` |
| **Search input** | `input` event → re-render | `search.addEventListener('input', renderAttendeeList)` |
| **View enter** | Hook in `showView()` | `if (viewName === 'attendance') renderAttendeeList()` |
| **Init** | `DOMContentLoaded` → `loadData()` + initial renders | `document.addEventListener('DOMContentLoaded', init)` |

---

## Data Flow Summary

```
USER ACTION
    │
    ▼
EVENT HANDLER (UI)
    │
    ├─► VALIDATION
    │
    ▼
DATA API (window.App)
    │
    ├─► students[] MUTATION
    │
    ▼
saveData() → localStorage.setItem()
    │
    ▼
RE-RENDER (renderDashboardStats / renderAttendeeList)
    │
    ▼
UI UPDATED
```

---

## Testing Checklist

| Operation | Test |
|-----------|------|
| **Create** | Add student → appears in list, stats update, persists after refresh |
| **Duplicate** | Add same student number → error notice, no duplicate in storage |
| **Edit** | Click Edit → form populates → save → updates list, original ID preserved |
| **Toggle** | Click Check In → status "Present", timestamp appears → Check Out → "Absent", timestamp cleared |
| **Delete** | Click Delete → confirm → removed from list, stats decrement |
| **Clear All** | Settings → Reset All → confirm → all data gone, stats zero |
| **Search** | Type in search → list filters in real-time |
| **Sort** | Click Sort → toggles A-Z / Z-A |
| **Quick Check-in** | Enter number on dashboard → toggles presence instantly |
| **Export** | Click Export CSV → downloads file with all data |
| **Import** | Bulk Import → paste CSV → imports valid, reports errors |
| **Theme** | Toggle dark mode → persists after refresh |
| **Persistence** | Refresh page (F5) → all data intact |

---

## Key Implementation Decisions

| Decision | Rationale |
|----------|-----------|
| **Re-render on every mutation** | Simplicity over virtual DOM; list small (<1000), performance fine |
| **No event delegation on list** | Buttons recreated each render; attaching directly is clearer |
| **In-place array mutation** (`splice`, `length=0`) | Preserves `students` reference for any external holders |
| **ISO timestamps** | Sortable, timezone-aware, standard format |
| **`data-id` on buttons** | Direct ID access without DOM traversal |
| **Edit mode flag on form** | Single form serves both create/edit; minimal UI duplication |
| **Modal created in JS** | Keeps HTML clean; modal only exists when needed |
| **Confirm dialogs for destructive actions** | Native, accessible, no custom modal needed |