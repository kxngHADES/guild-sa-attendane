# View Switching Logic — Guild Attendance Register

> **Location**: Embedded in `index.html` before `</body>` (lines 806–867)

---

## Overview

Implements state-based view switching for the 4-view SPA (Dashboard, Registration, Attendance, Settings) using vanilla JavaScript. No frameworks, no page reloads, single-file constraint maintained.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        index.html                           │
├─────────────────────────────────────────────────────────────┤
│  <header class="global-header">           (fixed, 56px)    │
│  <main class="view-container">             (flex:1, scroll) │
│    ├── #view-dashboard       ← views[dashboard]            │
│    ├── #view-registration    ← views[registration]         │
│    ├── #view-attendance      ← views[attendance]           │
│    └── #view-settings        ← views[settings]             │
│  </main>                                                    │
│  <nav class="bottom-nav">                 (fixed, 56px)    │
│    ├── .nav-item[data-view="dashboard"]   → showView()    │
│    ├── .nav-item[data-view="registration"]                 │
│    ├── .nav-item[data-view="attendance"]                   │
│    └── .nav-item[data-view="settings"]                     │
│  </nav>                                                     │
│  <script>...view switching logic...</script>                │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Module (IIFE)

```js
(function () {
  'use strict';

  // 1. DOM REFERENCES (cached once)
  const views = {
    dashboard: document.getElementById('view-dashboard'),
    registration: document.getElementById('view-registration'),
    attendance: document.getElementById('view-attendance'),
    settings: document.getElementById('view-settings'),
  };

  const navButtons = document.querySelectorAll('.nav-item[data-view]');

  // 2. VIEW SWITCHING FUNCTION
  function showView(viewName) {
    // Hide ALL views
    Object.values(views).forEach(view => {
      view.classList.remove('view-visible');
      view.classList.add('view-hidden');
    });

    // Show TARGET view
    const targetView = views[viewName];
    if (targetView) {
      targetView.classList.remove('view-hidden');
      targetView.classList.add('view-visible');
    }

    // Update NAV active state + ARIA
    navButtons.forEach(btn => {
      const isActive = btn.dataset.view === viewName;
      btn.classList.toggle('active', isActive);
      btn.setAttribute('aria-current', isActive ? 'page' : 'false');
    });
  }

  // 3. EVENT DELEGATION (click)
  document.querySelector('.bottom-nav')
    .addEventListener('click', (e) => {
      const btn = e.target.closest('.nav-item[data-view]');
      if (btn) showView(btn.dataset.view);
    });

  // 4. KEYBOARD SUPPORT (Enter/Space)
  document.querySelector('.bottom-nav')
    .addEventListener('keydown', (e) => {
      if (e.key === 'Enter' || e.key === ' ') {
        const btn = e.target.closest('.nav-item[data-view]');
        if (btn) { e.preventDefault(); showView(btn.dataset.view); }
      }
    });

  // 5. PUBLIC API for future data logic
  window.App = window.App || {};
  window.App.showView = showView;
})();
```

---

## CSS Integration

| Class | Purpose | Applied To |
|-------|---------|------------|
| `.view-visible` | `display: block` | Active view |
| `.view-hidden` | `display: none` | Inactive views |
| `.nav-item.active` | Accent color + 2px top inset shadow | Active nav button |

**Initial State (HTML)**:
```html
<section id="view-dashboard" class="view-visible">...</section>
<section id="view-registration" class="view-hidden">...</section>
<section id="view-attendance" class="view-hidden">...</section>
<section id="view-settings" class="view-hidden">...</section>
```

---

## Data Attributes (HTML ↔ JS Contract)

| Element | Attribute | Values |
|---------|-----------|--------|
| `.nav-item` | `data-view` | `"dashboard" \| "registration" \| "attendance" \| "settings"` |

**Adding a new view:**
1. Add `<section id="view-newview" class="view-hidden">...</section>` in `.view-container`
2. Add `<button class="nav-item" data-view="newview">New View</button>` in `.bottom-nav`
3. Add `newview: document.getElementById('view-newview')` to `views` object
4. No other JS changes needed

---

## Public API

```js
// Programmatic view switch (e.g., from form submit, QR scan, deep link)
window.App.showView('attendance');  // → switches to Attendance view
```

**Use cases for future data logic:**
- Registration success → `window.App.showView('attendance')`
- QR check-in → `window.App.showView('dashboard')` (to see stats update)
- Deep link handling → `window.App.showView(viewFromHash)`

---

## Accessibility

| Feature | Implementation |
|---------|----------------|
| **Keyboard nav** | `Tab` focuses buttons, `Enter`/`Space` activates |
| **ARIA current** | `aria-current="page"` on active, `"false"` on others |
| **Focus visible** | Inherits global `:focus-visible` (3px accent ring) |
| **Screen readers** | `aria-label` on each button, `role="navigation"` on nav |

---

## Performance

- **Single DOM query** per element (cached in `views` object)
- **Event delegation**: 1 listener on `.bottom-nav` vs 4 per button
- **No reflows**: Only class toggles, no inline styles
- **~30 lines** — negligible bundle impact

---

## Testing Checklist

- [ ] Click each nav button → correct view shows, others hidden
- [ ] Active button has accent color + top border
- [ ] `Tab` through nav → `Enter`/`Space` switches view
- [ ] `aria-current` updates correctly
- [ ] Dashboard visible on fresh load
- [ ] Rapid clicking doesn't break state
- [ ] Mobile (≤480px) and desktop layouts intact

---

## Related Files

| File | Purpose |
|------|---------|
| `index.html` | Contains views, nav, and this script |
| `docs/css-implementation.md` | CSS classes `.view-hidden`, `.view-visible`, `.nav-item.active` |
| `docs/data-logic.md` | Future: will call `window.App.showView()` after data mutations |

---

## Future Extensions (Not Implemented)

| Feature | Approach |
|---------|----------|
| **Hash routing** | `window.addEventListener('hashchange', () => showView(hash.slice(1)))` |
| **View transitions** | Add `.view-transitioning` class, use `opacity` + `transform` with `--transition-fast` |
| **Lifecycle hooks** | `window.App.onViewEnter(viewName)`, `onViewExit(viewName)` callbacks |
| **Deep linking** | Parse `location.hash` on load, call `showView()` |

---