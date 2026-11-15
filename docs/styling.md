# Styling

The app uses a single global stylesheet with plain CSS. There are no CSS modules, preprocessors, utility frameworks, or CSS-in-JS libraries. Component files use `className` strings that map to selectors in `src/index.css`.

> **Note:** [`src/App.css`](../src/App.css) exists but is **not imported anywhere**. It is leftover Vite template CSS (`#root`, `.logo`, `.card`, `.read-the-docs`, `logo-spin`). It has no effect on the rendered app. See [Known issues #6](known-issues.md#6-srcappcss-is-unused-template-code).

---

## Where styles live

| File | Imported by | Status |
| --- | --- | --- |
| `src/index.css` | `src/main.jsx` | Active — all app styling. |
| `src/App.css` | Nobody | Dead code. |

`src/main.jsx:3` is the only stylesheet import:

```js
import './index.css'
```

---

## Design tokens

Colors and spacing are hard-coded rather than exposed as CSS custom properties. These are the values used throughout `index.css`.

### Color palette

| Role | Value | Used in |
| --- | --- | --- |
| Page background | `#010203` | `.app`, active toggle text |
| Surface / card | `rgba(255, 255, 255, 0.03)` | `.crypto-card`, `.coin-price-section`, `.chart-section`, `.stat-card` |
| Surface hover | `rgba(255, 255, 255, 0.05)` | `.crypto-card:hover`, inputs, selects |
| Border | `rgba(255, 255, 255, 0.1)` | Cards, header, footer, controls |
| Accent (light blue) | `#add8e6` | Titles, active controls, rank badges, chart line, focus rings |
| Accent muted | `rgba(173, 216, 230, 0.2–0.5)` | Focus shadows, card hover border, spinner track |
| Primary text | `#e0e0e0` | Headings, labels, values |
| Bright text | `#fff` | Prices, search input |
| Muted text | `#9ca3af` | Symbols, stat labels, footer, axes |
| Subtle text | `#a0a0b0` | Header subtitle |
| Positive | `#2ed573` | `.positive` change states |
| Positive surface | `rgba(46, 213, 115, 0.15)` | Positive badge background |
| Negative | `#ff4757` | `.negative` change states |
| Negative surface | `rgba(255, 71, 87, 0.15)` | Negative badge background |
| Placeholder | `#6b6b7a` | `.search-input::placeholder` |
| Tooltip surface | `rgba(20, 20, 40, 0.95)` | Inline Recharts tooltip style |
| Footer surface | `rgba(20, 20, 40, 0.4)` | `.footer` |

### Typography

- Font stack: `"Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", …`.
- **Inter is not loaded** via `<link>`, `@import`, or `@font-face`, so most users see the system fallback.
- Base line height `1.5`, weight `400`, antialiased smoothing.
- Sizes are defined per selector (e.g. header `2.5rem`, current price `3rem`, card price `1.75rem`, stat label `0.75–0.85rem`).

### Layout

| Token | Value |
| --- | --- |
| Max content width | `1400px` (header, controls, list, detail); `1280px` in unused `App.css` |
| Page gutter | `2rem` |
| Card radius | `20px` (cards, sections); `16px` (stat cards); `8px` (controls) |
| Grid card min width | `320px` |
| Card gap | `1.5rem` grid, `1rem` list |
| Chart height | `400px` |
| Mobile breakpoint | `768px` |

### Effects

- **Glassmorphism:** `backdrop-filter: blur(20px)` on cards and header, `blur(10px)` on controls.
- **Shadows:** layered black shadows (`0 8px 32px rgba(0,0,0,0.3–0.4)`) plus a blue glow on the active toggle and rank badges.
- **Motion:** `0.3s ease` transitions on hover/focus; spinner `spin 1s linear infinite`; card lift `translateY(-8px)`.
- **Card top highlight:** `.crypto-card::before` draws a 1px gradient line revealed on hover.

---

## Class map

### App shell

| Class | File | Purpose |
| --- | --- | --- |
| `.app` | `Home`, `CoinDetail` | Full-height dark page wrapper. |
| `.header` | both pages | Sticky, blurred page header (`z-index: 100`). |
| `.header-content` | both | Flex row, space-between, wraps on mobile. |
| `.logo-section` | both | Title + subtitle block. |
| `.search-section` | Home | Search input wrapper. |
| `.search-input` | Home | Pill-shaped input with focus ring. |
| `.controls` | Home | Sort/view control bar. |
| `.filter-group` | Home | Label + `<select>` group. |
| `.view-toggle` | Home | Grid/List button group; `.active` = accent fill. |
| `.footer` | both | Blurred attribution bar. |

### Market list

| Class | Purpose |
| --- | --- |
| `.crypto-container` | List wrapper; max-width 1400px. |
| `.crypto-container.grid` | Responsive grid, `minmax(320px, 1fr)`. |
| `.crypto-container.list` | Single-column grid. |
| `.crypto-card` | Glass card with hover lift and top highlight. |
| `.crypto-header`, `.crypto-info` | Card header row and logo/text group. |
| `.crypto-price`, `.price`, `.change` | Price block and change pill. |
| `.change.positive`, `.change.negative` | Green/red change states. |
| `.crypto-stats`, `.stat`, `.stat-label`, `.stat-value` | Bottom market-cap/volume row. |

### States

| Class | Purpose |
| --- | --- |
| `.loading` | Centered spinner container. |
| `.spinner` | Rotating ring (`@keyframes spin`). |
| `.no-results` | Centered empty/not-found message. |
| `.back-button` | Defined but **not used** by any component. |

### Detail page

| Class | Purpose |
| --- | --- |
| `.coin-detail` | Detail page content wrapper. |
| `.coin-header` | Header row with title and rank. |
| `.coin-title` | Logo + name/symbol flex group. **Markup uses `.cointitle`, so this selector does not match** (see [Known issues #4](known-issues.md#4-detail-page-title-block-uses-the-wrong-css-class)). |
| `.coin-header .rank` | Rank pill on the detail page. |
| `.coin-price-section` | Price panel. |
| `.current-price` | Large price + change badge. |
| `.change-badge.positive` / `.negative` | Detail change badge states. |
| `.price-ranges`, `.price-range`, `.range-label`, `.range-value` | 24h high/low block. |
| `.chart-section` | Chart panel. |
| `.stats-grid`, `.stat-card` | Four-card statistics grid. |

---

## Responsive behavior

A single breakpoint at **`max-width: 768px`** changes:

| Element | Mobile change |
| --- | --- |
| `.header-content` | Column layout, centered text. |
| `.search-section` | Full width. |
| `.logo-section h1` | `2.5rem` → `2rem`. |
| `.controls` | Column, stretch. |
| `.filter-group` | Column; select full width. |
| `.view-toggle` | Centered. |
| `.crypto-container.grid` | Forced single column. |
| `.coin-header` | Column, left-aligned. |
| `.coin-title h1` | `2rem` → `1.5rem`. |
| `.current-price h2` | `3rem` → `2rem`. |
| `.stats-grid` | Forced single column. |

There are no tablet or large-desktop breakpoints; the grid uses `auto-fill` to adapt between 320px and 1400px.

---

## Static assets

| Asset | Used by |
| --- | --- |
| `public/arrow-up.png` | `CryptoCard`, `CoinDetail` positive change |
| `public/arrow-down.png` | `CryptoCard`, `CoinDetail` negative change |
| `public/vite.svg` | `index.html` favicon (default Vite icon) |

Assets in `public/` are referenced with root-absolute paths (`/arrow-up.png`) and are copied to the build root.

---

## Known styling issues

| Issue | Detail |
| --- | --- |
| Unused stylesheet | `src/App.css` is dead code. |
| Class mismatch | Markup `cointitle` vs CSS `.coin-title`. |
| Unused selector | `.back-button` has no markup usage. |
| Font not loaded | Inter is referenced but never fetched. |
| Commented-out value | `.header` contains `/* background: rgba(20, 20, , 1); */` (invalid CSS, harmless). |
| Default favicon/title | `index.html` still uses `vite.svg` and `<title>crypto-track</title>`. |

---

## How to change the theme

Because there are no design tokens, a theme change requires editing values in `src/index.css`:

1. Background: `.app` and `.header`/`.footer` surfaces.
2. Accent: every `#add8e6` occurrence (titles, buttons, rank badges, spinner, chart line in `CoinDetail.jsx`).
3. Positive/negative: `.positive` / `.negative` blocks in both list and detail sections.
4. Chart colors are inline in `CoinDetail.jsx` (`stroke`, `contentStyle`) and must be changed separately.

If you introduce CSS custom properties, document them here and update the [class map](#class-map).

## Related documents

- [Components](components.md)
- [Pages](pages.md)
- [Known issues](known-issues.md)
