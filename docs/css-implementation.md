# CSS Implementation Reference — Guild Attendance Register

> **Status**: All CSS/UI styles documented here are **already implemented** in `index.html`. This document serves as a reference for future maintenance and JavaScript integration. **No additional CSS implementation is needed.**

---

## 1. CSS Custom Properties (Design Tokens)

### 1.1 Light Theme (Default) — `:root`

| Category | Token | Value | Purpose |
|----------|-------|-------|---------|
| **Brand** | `--color-primary` | `#4450b7` | Primary action color |
| | `--color-primary-container` | `#5e6ad2` | Primary hover/active |
| | `--color-on-primary` | `#ffffff` | Text on primary |
| | `--color-on-primary-container` | `#fdfaff` | Text on primary container |
| | `--color-accent` | `#5e6ad2` | Accent (focus, active nav) |
| **Neutral (Light)** | `--bg-canvas` | `#ffffff` | Page background |
| | `--bg-surface` | `#fbfbfa` | Cards, nav, inputs |
| | `--bg-elevated` | `#f7f8f8` | Hover states |
| | `--border-default` | `#e8e8e8` | Hairline borders |
| | `--text-primary` | `#101112` | Primary text |
| | `--text-secondary` | `#6b6b6b` | Secondary text |
| | `--color-surface` | `#f8f9f9` | Surface variant |
| | `--color-surface-dim` | `#d9dada` | Dim surface |
| | `--color-on-surface` | `#191c1c` | Text on surface |
| | `--color-on-surface-variant` | `#454652` | Muted text |
| | `--color-outline` | `#767684` | Outline color |
| | `--color-outline-variant` | `#c6c5d5` | Outline variant |
| **Status** | `--color-success` | `#1f8a57` | Success state |
| | `--color-success-bg` | `rgba(31,138,87,0.1)` | Success background |
| | `--color-danger` | `#dc2b2b` | Danger/error state |
| | `--color-danger-bg` | `rgba(220,43,43,0.1)` | Danger background |
| | `--color-warning` | `#b27a00` | Warning/late state |
| | `--color-warning-bg` | `rgba(178,122,0,0.1)` | Warning background |
| **Layout** | `--container-max-width` | `640px` | Content max width |
| | `--header-height` | `56px` | Top header height |
| | `--row-height` | `48px` | List row height |
| | `--input-height` | `36px` | Input height |
| | `--gutter` | `16px` | Horizontal padding |
| | `--margin-page` | `24px` | Page margins |
| **Spacing** | `--space-xs` | `4px` | Micro spacing |
| | `--space-sm` | `8px` | Small spacing |
| | `--space-md` | `16px` | Medium spacing |
| | `--space-lg` | `24px` | Large spacing |
| **Radii** | `--radius-sm` | `0.125rem` (2px) | Small elements |
| | `--radius-default` | `0.25rem` (4px) | Default |
| | `--radius-md` | `0.375rem` (6px) | Inputs, buttons |
| | `--radius-lg` | `0.5rem` (8px) | Cards |
| | `--radius-xl` | `0.75rem` (12px) | Large cards, modals |
| | `--radius-full` | `9999px` | Pills, avatars |
| **Transitions** | `--transition-fast` | `120ms ease-out` | Standard transition |
| **Typography** | `--font-primary` | System stack + Inter | UI text |
| | `--font-mono` | SF Mono stack | Tabular numbers |

### 1.2 Dark Theme — `@media (prefers-color-scheme: dark)` + `.dark-theme`

| Token | Dark Value |
|-------|------------|
| `--bg-canvas` | `#0f1011` |
| `--bg-surface` | `#1a1b1d` |
| `--bg-elevated` | `#222325` |
| `--border-default` | `#2a2b2d` |
| `--text-primary` | `#f8f9f9` |
| `--text-secondary` | `#a1a1a5` |
| `--color-surface` | `#1e1e21` |
| `--color-surface-dim` | `#121214` |
| `--color-on-surface` | `#f0f1f1` |
| `--color-success` | `#22a06b` |
| `--color-success-bg` | `rgba(34,160,107,0.15)` |
| `--color-danger` | `#e5484d` |
| `--color-danger-bg` | `rgba(229,72,77,0.15)` |
| `--color-warning` | `#dca022` |
| `--color-warning-bg` | `rgba(220,160,34,0.15)` |

**Activation**: JS toggles `.dark-theme` on `<html>` (prevents flash, respects system + manual override).

---

## 2. Global Reset & Base Styles

```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html { font-size: 16px; -webkit-text-size-adjust: 100%; }
body {
  font-family: var(--font-primary);
  background: var(--bg-canvas);
  color: var(--text-primary);
  line-height: 1.5;
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden; /* SPA shell - no page scroll */
}
input, button, select, textarea { font: inherit; }
button { background: none; border: none; cursor: pointer; }
a { color: inherit; text-decoration: none; }
```

---

## 3. Typography Scale

| Class | Size | Weight | Line Height | Tracking | Use Case |
|-------|------|--------|-------------|----------|----------|
| `.text-h1` / `#dashboard-heading` | 24px | 600 | 32px | -0.5px | Page titles |
| `.text-h2` / `h2` view headings | 18px | 600 | 24px | 0 | Section headers |
| `.text-body` / default | 14px | 400 | 20px | 0 | Body text, inputs |
| `.text-label` / `label` | 12px | 500 | 16px | +0.5px | Labels, metadata (uppercase) |
| `.text-mono` / `.stat-value` | 14px | 400/600 | — | — | **Tabular nums** (counters, IDs, times) |
| `.app-title` | 16px | 600 | — | — | Top header brand |

**Critical**: All numeric displays use `font-variant-numeric: tabular-nums` via `.text-mono` to prevent layout jitter.

---

## 4. SPA Shell Layout (Fixed Viewport)

```
┌─────────────────────────────────────┐
│ .global-header (fixed, 56px)        │  ← z-index: 100
├─────────────────────────────────────┤
│                                     │
│ .view-container (flex:1, scroll)    │  ← max-width: 640px, centered
│   ├── #view-dashboard               │     padding: 16px 16px
│   ├── #view-registration            │     padding-bottom: 72px (nav buffer)
│   ├── #view-attendance              │
│   └── #view-settings                │
│                                     │
├─────────────────────────────────────┤
│ .bottom-nav (fixed, 56px)           │  ← z-index: 100
└─────────────────────────────────────┘
```

### Key Classes

| Class | Purpose |
|-------|---------|
| `.global-header` | Fixed top bar, hairline bottom border |
| `.view-container` | Scrollable content area, centered, max 640px |
| `.view-hidden` / `.view-visible` | View switching (`display: none/block`) |
| `.bottom-nav` | Fixed bottom nav, 4 equal-width items |
| `.nav-item` | Nav button, inactive = secondary text |
| `.nav-item.active` | Accent color + 2px top inset shadow |

---

## 5. Component Styles Reference

### 5.1 Dashboard View

| Component | Class | Key Styles |
|-----------|-------|------------|
| Stats Grid | `.stats-grid` | `grid-template-columns: repeat(2,1fr)` → `repeat(4,1fr)` @ ≥480px |
| Stat Card | `.stat-card` | `bg-surface`, `border`, `radius-xl`, `padding-md`, centered |
| Stat Label | `.stat-label` | 10px, uppercase, `text-secondary` |
| Stat Value | `.stat-value` / `.text-mono` | 24px, 600, tabular-nums |
| Quick Check-in | `.quick-actions` | Card wrapper, label + full-width input |

### 5.2 Registration View

| Component | Class/Selector | Key Styles |
|-----------|----------------|------------|
| Form Group | `.form-group` | `margin-bottom: var(--space-md)` |
| Label | `.form-group label` | 12px, 500, uppercase, `text-secondary` |
| Input | `.form-group input` | 36px, `bg-canvas`, `border-default`, `radius-md` |
| Input Focus | `:focus` | `border-color: accent` + `box-shadow: 0 0 0 3px rgba(94,106,210,0.15)` |
| Submit Button | `#registration-form button[type="submit"]` | Full width, 36px, `bg-primary`, `radius-md` |
| Notice Area | `#registration-notice` | Dynamic `.success` / `.error` classes |

### 5.3 Attendance View

| Component | Class | Key Styles |
|-----------|-------|------------|
| Controls Bar | `.controls` | Flex wrap, gap `space-sm` |
| Search Input | `.controls input[type="search"]` | Flex:1, min-width 200px |
| Sort Button | `.controls button` | 36px, `bg-surface`, `border-default` |
| Attendee Row | `.attendee-item` | Flex row, `bg-surface`, `border`, `radius-lg`, hover → `bg-elevated` |
| Avatar | `.attendee-avatar` | 32px circle, `bg-elevated`, initials |
| Name | `.attendee-name` | 14px, 500, ellipsis |
| Student # | `.attendee-number` | Mono, tabular, 12px, `text-secondary` |
| Status Badge | `.attendee-status` | Pill (`radius-full`), 11px, uppercase |
| &nbsp;&nbsp;• Present | `.attendee-status.present` | `success-bg` + `success` text |
| &nbsp;&nbsp;• Absent | `.attendee-status.absent` | `bg-elevated` + `text-secondary` |
| &nbsp;&nbsp;• Late | `.attendee-status.late` | `warning-bg` + `warning` text |
| Timestamp | `.attendee-time` | Mono, tabular, 12px |
| Actions | `.attendee-actions button` | Small, transparent, hover → `bg-elevated` |
| &nbsp;&nbsp;• Danger | `.attendee-actions button.danger:hover` | `danger-bg` + `danger` text + `danger` border |
| Pagination | `.pagination` | Centered, disabled state (opacity 0.4) |

### 5.4 Settings View

| Component | Class | Key Styles |
|-----------|-------|------------|
| Toggle Switch | `.toggle-switch` | Inline-flex, hidden checkbox |
| Slider Track | `.toggle-slider` | 36×20px, `border-default`, `radius-full` |
| Slider Knob | `.toggle-slider::after` | 16×16px, white, shadow |
| Checked | `input:checked + .toggle-slider` | `bg-accent`, knob `translateX(16px)` |
| Focus | `input:focus-visible + .toggle-slider` | 3px accent ring |
| Fieldset | `fieldset` | No border, `margin-bottom: space-lg` |
| Legend | `fieldset legend` | 12px, 600, uppercase, `text-secondary` |
| Action Buttons | `fieldset button` | Full width, 36px, left-aligned, `bg-surface` |
| Danger Button | `.btn-danger` | Transparent, `danger` border/text, hover → fill `danger` + white text |

### 5.5 Modal (Bulk Import)

| Component | Class | Key Styles |
|-----------|-------|------------|
| Overlay | `.modal-overlay` | Fixed inset, `rgba(0,0,0,0.5)`, `opacity/visibility` transition |
| Open State | `.modal-overlay.open` | `opacity:1`, `visibility:visible` |
| Modal | `.modal` | `bg-canvas`, `border`, `radius-xl`, `max-width:480px`, `max-height:80vh` |
| Enter Animation | `.modal-overlay.open .modal` | `transform: translateY(0)` (from 10px) |
| Textarea | `.modal textarea` | Mono, 13px, min-height 200px |
| Actions | `.modal-actions` | Flex-end, gap `space-sm` |
| Primary | `.btn-primary` | `bg-primary`, `on-primary` text |
| Secondary | `.btn-secondary` | `bg-surface`, `border-default` |

---

## 6. Responsive Breakpoints

| Breakpoint | Changes |
|------------|---------|
| **< 480px** (mobile) | Stats grid: 2 columns; Quick check-in stacks (via media query on `.quick-checkin-form` if added); Bottom nav labels at 11px |
| **≥ 480px** (tablet/desktop) | Stats grid: 4 columns; Container max-width 640px centers content |

No other breakpoints needed — fluid layout handles intermediate sizes.

---

## 7. View Switching (JS Integration Points)

### HTML Structure
```html
<section id="view-dashboard" class="view-visible" aria-labelledby="dashboard-heading">...</section>
<section id="view-registration" class="view-hidden" aria-labelledby="reg-heading">...</section>
<section id="view-attendance" class="view-hidden" aria-labelledby="att-heading">...</section>
<section id="view-settings" class="view-hidden" aria-labelledby="settings-heading">...</section>
```

### JS Pattern
```js
function showView(viewId) {
  document.querySelectorAll('[id^="view-"]').forEach(el => el.classList.add('view-hidden'));
  document.getElementById(viewId).classList.remove('view-hidden');
  document.getElementById(viewId).classList.add('view-visible');
  
  // Update nav
  document.querySelectorAll('.nav-item').forEach(btn => {
    btn.classList.toggle('active', btn.dataset.view === viewId.replace('view-', ''));
    btn.setAttribute('aria-current', btn.dataset.view === viewId.replace('view-', '') ? 'page' : 'false');
  });
}
```

### Nav Button Data Attributes
```html
<button class="nav-item active" data-view="dashboard" aria-label="Dashboard" aria-current="page">Dashboard</button>
<button class="nav-item" data-view="registration" aria-label="Register">Register</button>
<button class="nav-item" data-view="attendance" aria-label="Attendance">Attendance</button>
<button class="nav-item" data-view="settings" aria-label="Settings">Settings</button>
```

---

## 8. Theme Toggle (JS Integration)

### CSS Hooks
- System preference: `@media (prefers-color-scheme: dark)` on `:root`
- Manual override: `.dark-theme` on `<html>` (or `<body>`)

### JS Implementation
```js
const html = document.documentElement;
const toggle = document.getElementById('theme-toggle');

// Initialize from localStorage or system preference
const saved = localStorage.getItem('theme');
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
const isDark = saved ? saved === 'dark' : prefersDark;

if (isDark) html.classList.add('dark-theme');
toggle.checked = isDark;

toggle.addEventListener('change', () => {
  const dark = toggle.checked;
  html.classList.toggle('dark-theme', dark);
  localStorage.setItem('theme', dark ? 'dark' : 'light');
});

// Optional: listen for system changes
window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', e => {
  if (!localStorage.getItem('theme')) {
    html.classList.toggle('dark-theme', e.matches);
    toggle.checked = e.matches;
  }
});
```

---

## 9. State Classes for Dynamic Updates

| Element | State Classes | Applied By JS |
|---------|---------------|---------------|
| `#registration-notice` | `.success` / `.error` | Form submit result |
| `.attendee-status` | `.present` / `.absent` / `.late` | Attendance toggle |
| `.attendee-item` | (hover handled by CSS) | — |
| `.nav-item` | `.active` | View switch |
| `.modal-overlay` | `.open` | Bulk import modal |
| `.pagination button` | `:disabled` | Page boundaries |
| `#btn-sort-toggle` | `[aria-pressed="true/false"]` | Sort toggle |

---

## 10. Accessibility Notes

- **Focus visible**: All interactive elements have `:focus-visible` styles (3px accent ring)
- **Color contrast**: All text meets WCAG AA on both themes
- **Reduced motion**: Consider adding `@media (prefers-reduced-motion: reduce) { * { transition: none !important; } }`
- **ARIA**: Views have `aria-labelledby`, nav has `aria-label`, live region for notices
- **Tabular nums**: Prevents layout shift on counter updates

---

## 11. Maintenance Checklist

When adding new features, ensure:

- [ ] Use existing CSS variables — **no hardcoded colors/spacing**
- [ ] Follow spacing scale (`space-xs` through `space-lg`)
- [ ] Use radius scale (`radius-sm` through `radius-xl`)
- [ ] Add dark theme values if new colors introduced
- [ ] Test both light/dark modes
- [ ] Test mobile (≤480px) and desktop (≥480px)
- [ ] Verify focus states for new interactive elements
- [ ] Use `.text-mono` for any numeric display

---

## 12. File Structure Reference

```
index.html
├── <head>
│   └── <style> (ALL CSS - single file constraint)
│       ├── :root (tokens)
│       ├── @media dark (system dark)
│       ├── .dark-theme (manual dark)
│       ├── Reset & Base
│       ├── Typography
│       ├── SPA Shell (.global-header, .view-container, .bottom-nav)
│       ├── View Visibility (.view-hidden/.view-visible)
│       └── Component Styles (Dashboard, Registration, Attendance, Settings, Modal)
└── <body>
    ├── .global-header
    ├── .view-container
    │   ├── #view-dashboard.view-visible
    │   ├── #view-registration.view-hidden
    │   ├── #view-attendance.view-hidden
    │   └── #view-settings.view-hidden
    └── .bottom-nav
```

---

> **Bottom Line**: The entire design system is implemented in `index.html`. This document is the authoritative reference for JavaScript integration and future maintenance. No additional CSS work is required.