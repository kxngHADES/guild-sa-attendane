# UI Style Guide

This style guide defines the system design, visual hierarchy, and CSS token standards for the GUILD SA Student Attendance Register Tool. Built on a Minimalist and Linear-esque design system, this interface is engineered for sub-$100\text{ ms}$ interaction speeds, zero-friction mobile execution, and high layout scanability.

---

## 1. CSS Custom Properties (Theme Token Map)

The style values from DESIGN.md are mapped directly below. To comply with the "Single-File" constraint, place these variables inside the `:root` pseudo-class within the `<style>` tag of your `index.html` file.

```css
:root {
  /* Brand & Core Accents */
  --color-primary: #4450b7;
  --color-primary-container: #5e6ad2;
  --color-on-primary: #ffffff;
  --color-on-primary-container: #fdfaff;
  --color-accent: #5e6ad2;

  /* Neutral Scale (Light Mode Default) */
  --bg-canvas: #ffffff;
  --bg-surface: #fbfbfa;
  --bg-elevated: #f7f8f8;
  --border-default: #e8e8e8;
  
  --text-primary: #101112;
  --text-secondary: #6b6b6b;

  --color-surface: #f8f9f9;
  --color-surface-dim: #d9dada;
  --color-on-surface: #191c1c;
  --color-on-surface-variant: #454652;
  --color-outline: #767684;
  --color-outline-variant: #c6c5d5;

  /* Status Colors */
  --color-success: #1f8a57;
  --color-success-bg: rgba(31, 138, 87, 0.1);
  --color-danger: #dc2b2b;
  --color-danger-bg: rgba(220, 43, 43, 0.1);
  --color-warning: #b27a00;
  --color-warning-bg: rgba(178, 122, 0, 0.1);

  /* Layout Spacing */
  --container-max-width: 640px;
  --header-height: 56px;
  --row-height: 48px;
  --input-height: 36px;
  --gutter: 16px;
  --margin-page: 24px;
  
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;

  /* Radii */
  --radius-sm: 0.125rem;   /* 2px */
  --radius-default: 0.25rem; /* 4px */
  --radius-md: 0.375rem;   /* 6px - Core UI components */
  --radius-lg: 0.5rem;     /* 8px */
  --radius-xl: 0.75rem;    /* 12px - Cards & View blocks */
  --radius-full: 9999px;   /* Avatars & Pill Badges */

  /* Transitions */
  --transition-fast: 120ms ease-out;
}

/* Dark Theme Overrides */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-canvas: #0f1011;
    --bg-surface: #1a1b1d;
    --bg-elevated: #222325;
    --border-default: #2a2b2d;
    
    --text-primary: #f8f9f9;
    --text-secondary: #a1a1a5;

    --color-surface: #1e1e21;
    --color-surface-dim: #121214;
    --color-on-surface: #f0f1f1;
    
    --color-success: #22a06b;
    --color-success-bg: rgba(34, 160, 107, 0.15);
    --color-danger: #e5484d;
    --color-danger-bg: rgba(229, 72, 77, 0.15);
    --color-warning: #dca022;
    --color-warning-bg: rgba(220, 160, 34, 0.15);
  }
}

/* Explicit class-based Dark Theme Selector */
body.dark-theme {
  --bg-canvas: #0f1011;
  --bg-surface: #1a1b1d;
  --bg-elevated: #222325;
  --border-default: #2a2b2d;
  
  --text-primary: #f8f9f9;
  --text-secondary: #a1a1a5;

  --color-surface: #1e1e21;
  --color-surface-dim: #121214;
  --color-on-surface: #f0f1f1;
  
  --color-success: #22a06b;
  --color-success-bg: rgba(34, 160, 107, 0.15);
  --color-danger: #e5484d;
  --color-danger-bg: rgba(229, 72, 77, 0.15);
  --color-warning: #dca022;
  --color-warning-bg: rgba(220, 160, 34, 0.15);
}
```

---

## 2. Typography Hierarchy

The typography layout relies purely on platform-native modern system fonts to limit external layout render delays.

| Name | CSS Target | Size | Weight | Line Height | Tracking | Purpose |
|---|---|---|---|---|---|---|
| Header Title | `.app-title` | $16\text{ px}$ | $600$ (Semi-Bold) | $24\text{ px}$ | $-0.2\text{ px}$ | Top Global Persistent Header |
| Headline LG | `h1`, `.text-h1` | $24\text{ px}$ | $600$ (Semi-Bold) | $32\text{ px}$ | $-0.5\text{ px}$ | Page/View headings |
| Headline MD | `h2`, `.text-h2` | $18\text{ px}$ | $600$ (Semi-Bold) | $24\text{ px}$ | $0\text{ px}$ | Card & form section labels |
| Body MD | `p`, `input`, `.text-body` | $14\text{ px}$ | $400$ (Regular) | $20\text{ px}$ | $0\text{ px}$ | Names, form field entries |
| Label SM | `label`, `.text-label` | $12\text{ px}$ | $500$ (Medium) | $16\text{ px}$ | $+0.5\text{ px}$ (Caps) | Upper metadata labels |
| Mono Tabular | `code`, `.text-mono` | $14\text{ px}$ | $400$ (Regular) | — | — | Counts, IDs, timestamps |

### Prevent Layout Shift (Jitter-Free Counters)

To eliminate structural shifts during real-time score updates in your Dashboard Stat cards, all numerical values and statistics indicators must utilize:

```css
.text-mono {
  font-family: var(--font-mono);
  font-variant-numeric: tabular-nums;
}
```

---

## 3. Persistent Layout Architecture & SPA Shell

The SPA framework operates as a vertical column with absolute viewport bounding. This guarantees zero mobile page-scrolling bounce.

```
+------------------------------------+
|   HEADER: "Guild Attendance"       | -> Fixed top (56px)
+------------------------------------+
|                                    |
|         SCROLLABLE CONTENT         | -> Main View container
|         (Max Width: 640px)         |
|                                    |
+------------------------------------+
|   BOTTOM TAB NAVIGATION BAR        | -> Fixed base (56px)
+------------------------------------+
```

### Global Frame Controls

```css
body {
  margin: 0;
  padding: 0;
  font-family: var(--font-primary);
  background-color: var(--bg-canvas);
  color: var(--text-primary);
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.view-container {
  flex: 1;
  overflow-y: auto;
  max-width: var(--container-max-width);
  width: 100%;
  margin: 0 auto;
  padding: var(--space-md) var(--gutter);
  padding-bottom: calc(var(--space-lg) * 3); /* Bottom nav buffer */
  box-sizing: border-box;
}
```

---

## 4. Component-Level Visual Styles

### A. Persistent Top Header

Every view displays a minimalist top header styled directly from the wireframe reference (`dashboard_2.jpg`).

- **Dimensions:** $56\text{ px}$ height, centering content horizontally or holding it to the left margins.
- **Aesthetics:** High-contrast text, transparent background, thin bottom hairline border (`1px solid var(--border-default)`).

```css
.global-header {
  height: var(--header-height);
  border-bottom: 1px solid var(--border-default);
  display: flex;
  align-items: center;
  padding: 0 var(--gutter);
  background-color: var(--bg-canvas);
  z-index: 100;
}
.global-header .app-title {
  margin: 0;
  font-size: 16px;
  font-weight: 600;
  color: var(--text-primary);
}
```

### B. Dashboard View Components

#### 1. Quick Check-In Layout

As seen in `dashboard_2.jpg`, the check-in is displayed as a clean card containing an inline text field and action button:

- **Desktop Layout:** Side-by-side flex block.
- **Mobile Layout:** Stacks vertical.

```css
.quick-checkin-card {
  background: var(--bg-surface);
  border: 1px solid var(--border-default);
  border-radius: var(--radius-xl);
  padding: var(--space-md);
  margin-bottom: var(--space-md);
}
.quick-checkin-form {
  display: flex;
  gap: var(--space-sm);
  margin-top: var(--space-sm);
}
@media (max-width: 480px) {
  .quick-checkin-form {
    flex-direction: column;
  }
}
```

#### 2. Real-Time Stat Cards Grid

Displays real-time computed attendance indicators.

- **Layout:** 2-column grid on mobile platforms, modern 4-column inline grid on widths above $480\text{ px}$.

```css
.stats-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: var(--space-sm);
  margin-bottom: var(--space-lg);
}
@media (min-width: 480px) {
  .stats-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}
.stat-card {
  background: var(--bg-surface);
  border: 1px solid var(--border-default);
  border-radius: var(--radius-xl);
  padding: var(--space-md);
  text-align: center;
}
.stat-card .stat-label {
  display: block;
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  color: var(--text-secondary);
  margin-bottom: var(--space-xs);
}
.stat-card .stat-value {
  font-size: 24px;
  font-weight: 600;
  color: var(--text-primary);
}
```

#### 3. Recent Activity Block

Holds log entries of the latest check-ins.

**Aesthetics:** Rounded container with an interactive "View All Records" button at the bottom that switches state to the Attendance view.

### C. Registration Form Fields & Notices

A modern, single-column alignment interface for registration.

**Aesthetics:** Flat border-based entry text inputs. Input fields glow with standard focus-ring shadows.

#### Form Field Focus Ring

```css
input[type="text"], input[type="search"] {
  width: 100%;
  height: var(--input-height);
  padding: 0 var(--space-sm);
  background: var(--bg-canvas);
  border: 1px solid var(--border-default);
  border-radius: var(--radius-md);
  color: var(--text-primary);
  box-sizing: border-box;
  transition: var(--transition-fast);
}
input[type="text"]:focus, input[type="search"]:focus {
  outline: none;
  border-color: var(--color-accent);
  box-shadow: 0 0 0 3px rgba(94, 106, 210, 0.15);
}
```

#### Notification Banners

Flat notification banners appear directly under the registration form (`#registration-notice`) with immediate state layout updates. Use `--color-success` or `--color-danger` colored borders.

### D. Attendance List Card Design

Styled according to the detailed mockup in `attendance.png`, each row card represents a student element:

```
+-----------------------------------------------------------------+
|  [ AJ ]  Alice Johnson               [ PRESENT ]   <->  [Edit]  [Delete]
|          #STU-10492                   08:14 AM                  |
+-----------------------------------------------------------------+
```

#### Avatar Badges

A $32\text{ px} \times 32\text{ px}$ circle element containing the attendee's initials (e.g. "AJ" for Alice Johnson). Built using `background: var(--bg-elevated); color: var(--text-secondary);`.

#### Attendance Badges

Fully flat pill-type element showing current check-in state:

- **PRESENT:** Light green container (`var(--color-success-bg)`), green text (`var(--color-success)`).
- **ABSENT:** Light grey container (`var(--bg-elevated)`), muted grey text (`var(--text-secondary)`).
- **LATE:** Light yellow container (`var(--color-warning-bg)`), yellow text (`var(--color-warning)`).

#### Audit Timestamps

Appears below status badges using `.text-mono` tabular layout to track chronological registration times.

#### Pagination Bar

Minimalist row displaying "Previous Page X of Y Next" with flat, simple hover action buttons at the bottom.

### E. Settings View & Toggles

#### 1. Theme Slider Toggle (Switch)

Follows the structural design in `settings.png`:

```css
.toggle-switch {
  display: inline-flex;
  align-items: center;
  gap: var(--space-sm);
  cursor: pointer;
}
.toggle-slider {
  position: relative;
  width: 36px;
  height: 20px;
  background-color: var(--border-default);
  border-radius: var(--radius-full);
  transition: var(--transition-fast);
}
.toggle-slider::after {
  content: "";
  position: absolute;
  top: 2px;
  left: 2px;
  width: 16px;
  height: 16px;
  background-color: #ffffff;
  border-radius: var(--radius-full);
  transition: var(--transition-fast);
}
input[type="checkbox"]:checked + .toggle-slider {
  background-color: var(--color-accent);
}
input[type="checkbox"]:checked + .toggle-slider::after {
  transform: translateX(16px);
}
```

#### 2. Primary & Destructive Buttons

- **Standard / Primary Buttons:** `background: var(--color-primary); color: var(--color-on-primary);`
- **Danger Actions (Reset All):**

```css
.btn-danger {
  background: transparent;
  border: 1px solid var(--color-danger);
  color: var(--color-danger);
  border-radius: var(--radius-md);
  padding: var(--space-sm) var(--space-md);
  cursor: pointer;
  transition: var(--transition-fast);
}
.btn-danger:hover {
  background: var(--color-danger);
  color: #ffffff;
}
```

### F. Fixed Bottom Navigation Bar

The bottom system shell anchors to the browser bottom viewport layout.

- **Structure:** Absolute bottom positioning, $56\text{ px}$ row boundary.
- **Active Accent line:** The active tab is indicated with an inline $2\text{ px}$ top border highlight colored by `var(--color-accent)`.

```css
.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 56px;
  background: var(--bg-surface);
  border-top: 1px solid var(--border-default);
  display: flex;
  justify-content: space-around;
  align-items: center;
  z-index: 100;
}
.nav-item {
  flex: 1;
  height: 100%;
  border: none;
  background: transparent;
  color: var(--text-secondary);
  font-size: 11px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}
.nav-item.active {
  color: var(--color-accent);
  box-shadow: inset 0 2px 0 0 var(--color-accent);
}
```
