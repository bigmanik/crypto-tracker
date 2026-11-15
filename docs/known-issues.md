# Known Issues & Limitations

This file records defects and constraints observed in the current code. It is documentation only — nothing here has been changed. Each entry lists the location, observed behavior, impact, and a suggested fix.

## Summary

| # | Issue | Area | Severity |
| --- | --- | --- | --- |
| [1](#1-home-page-waits-30-seconds-before-the-first-request) | Home page waits 30 s before first request | `Home.jsx` | High |
| [2](#2-detail-page-24-hour-change-always-shows-000) | Detail 24h change always `0.00%` | `CoinDetail.jsx` | High |
| [3](#3-detail-page-can-show-coin-not-found-while-loading) | False "coin not found" while loading | `CoinDetail.jsx` | Medium |
| [4](#4-detail-page-title-block-uses-the-wrong-css-class) | Wrong CSS class on title block | `CoinDetail.jsx` | Low |
| [5](#5-detail-footer-claims-30-second-updates-but-there-is-no-polling) | Footer claims 30 s updates; no polling | `CoinDetail.jsx` | Low |
| [6](#6-srcappcss-is-unused-template-code) | `src/App.css` unused | `App.css` | Low |
| [7](#7-default-html-title-and-favicon) | Default title/favicon | `index.html` | Low |
| [8](#8-errors-are-only-logged-to-the-console) | No user-facing error UI | Both pages | Medium |
| [9](#9-detail-requests-have-no-abort-or-stale-response-guard) | No abort / stale response guard | `CoinDetail.jsx` | Medium |
| [10](#10-no-test-suite) | No tests | Project | Medium |
| [11](#11-no-caching-retry-or-backoff) | No caching/retry/backoff | `coinGecko.js` | Medium |
| [12](#12-sub-cent-prices-omit-the-currency-symbol) | Sub-cent prices omit `$` | `formatter.js` | Low |
| [13](#13-api-base-url-and-limits-are-hard-coded) | Hard-coded API config | `coinGecko.js`, `Home.jsx` | Low |
| [14](#14-array-index-used-as-list-key) | Index used as list key | `Home.jsx` | Low |
| [15](#15-list-is-limited-to-the-top-100-coins) | Only top 100 coins searchable | `Home.jsx` | Low |
| [16](#16-accessibility-gaps) | Accessibility gaps | Multiple | Medium |

---

## 1. Home page waits 30 seconds before the first request

**Location:** `src/pages/Home.jsx:13-17`

```jsx
useEffect(() => {
  const interval = setInterval(fetchCryptoData, 30000);
  return () => clearInterval(interval);
}, []);
```

**Observed behavior:** the effect only registers an interval. `fetchCryptoData` is never called on mount, so the first data appears after the first 30-second tick. Until then the spinner is shown.

**Impact:** first-time users see a loading screen for 30 seconds and may assume the app is broken.

**Suggested fix:** call `fetchCryptoData()` once inside the effect before starting the interval (and keep the cleanup). Document the immediate fetch in [Pages](pages.md#home--srcpageshomejsx) afterward.

---

## 2. Detail page 24-hour change always shows `0.00%`

**Location:** `src/pages/CoinDetail.jsx:76`

```jsx
const priceChange = coin.price_change_percentage_24h || 0;
```

**Observed behavior:** the `/coins/{id}` response exposes the percentage under `market_data.price_change_percentage_24h`, not at the top level. The top-level field is undefined, so `priceChange` is always `0`, `isPositive` is `true`, and the badge renders an up arrow with `0.00%`.

**Impact:** the detail page always reports a flat 24-hour change, which is misleading.

**Suggested fix:** read `coin.market_data.price_change_percentage_24h` (with a null fallback) and keep the existing direction logic.

**See also:** [API reference — `fetchCoinData`](api.md#fetchcoindataid).

---

## 3. Detail page can show "coin not found" while loading

**Location:** `src/pages/CoinDetail.jsx:11-13`, `:28-30`, `:50-52`, `:65-74`

**Observed behavior:** `loadCoinData` and `loadChartData` share one `isLoading` flag and both set it to `false` in `finally`. If the chart request resolves before the coin request, `isLoading` becomes `false` while `coin` is still `null`, so the render branch shows "coin not found". It switches to the detail view when the coin request finishes.

**Impact:** a misleading flash of the not-found state, and a false not-found screen if the coin request fails while the chart succeeds.

**Suggested fix:** track loading/error per request (e.g. `coinLoading`, `chartLoading`) or derive the not-found state from an explicit error flag rather than `coin === null`.

---

## 4. Detail page title block uses the wrong CSS class

**Location:** `src/pages/CoinDetail.jsx:92` uses `className="cointitle"`; `src/index.css:424` defines `.coin-title`.

**Observed behavior:** the logo + name/symbol block does not receive the flex layout, image sizing, heading size, or symbol styling defined for `.coin-title`.

**Impact:** the detail header is visually misaligned compared with the intended design.

**Suggested fix:** change the markup class to `coin-title` (or rename the selector), and update the [class map](styling.md#detail-page).

---

## 5. Detail footer claims 30-second updates but there is no polling

**Location:** `src/pages/CoinDetail.jsx:188-190`

```jsx
<p> Data provided by CoinGecko API . updated every 30 seconds</p>
```

**Observed behavior:** the home page polls every 30 seconds; the detail page fetches once per mount and never refreshes.

**Impact:** stale prices are presented as live, and the footer contradicts the implementation.

**Suggested fix:** either add polling on the detail page (with cleanup and abort) or change the copy to state that data is loaded on page open.

---

## 6. `src/App.css` is unused template code

**Location:** `src/App.css`

**Observed behavior:** the file is never imported. It contains Vite starter styles (`#root`, `.logo`, `.card`, `.read-the-docs`, `logo-spin`) that do not correspond to any current markup. `src/main.jsx` imports only `./index.css`.

**Impact:** confusion for contributors and dead bytes in the repository (it is not bundled, since it is never imported).

**Suggested fix:** delete the file, or import and intentionally repurpose it. Until then, do not add styles there.

---

## 7. Default HTML title and favicon

**Location:** `index.html:5-7`

```html
<link rel="icon" type="image/svg+xml" href="/vite.svg" />
<title>crypto-track</title>
```

**Observed behavior:** the browser tab shows the Vite logo and the package name rather than the app brand.

**Impact:** unpolished appearance and poor tab identification.

**Suggested fix:** add a project favicon to `public/` and update the title to "Crypto Tracker" (or the brand name).

---

## 8. Errors are only logged to the console

**Location:** `src/pages/Home.jsx:28-30`, `src/pages/CoinDetail.jsx:26-28`, `:48-50`

**Observed behavior:** failed requests call `console.error` and otherwise keep the last state. The home page silently shows an empty list after the spinner clears; the detail page falls into "coin not found".

**Impact:** users cannot distinguish "no results", "still loading", and "request failed", and have no retry path.

**Suggested fix:** add explicit error state plus a message and retry button. Document the new states in [Pages](pages.md).

---

## 9. Detail requests have no abort or stale-response guard

**Location:** `src/pages/CoinDetail.jsx:16-19`

**Observed behavior:** the effect re-runs when `id` changes, but neither request uses an `AbortController` or an "is current request" check. A slow response for a previous coin can overwrite the current coin, and state is set after unmount if the user navigates away.

**Impact:** wrong data can be displayed briefly; React may warn about state updates after unmount.

**Suggested fix:** create an `AbortController` per effect run, pass its signal to `fetch`, abort on cleanup, and ignore stale results. `src/api/coinGecko.js` would need to accept an optional signal.

---

## 10. No test suite

**Location:** project root (`package.json` has no `test` script; no `*.test.*` / `*.spec.*` files)

**Observed behavior:** there is no automated coverage for formatters, sorting/filtering, API error handling, or render states.

**Impact:** regressions are only caught manually.

**Suggested fix:** add Vitest + React Testing Library. High-value first tests: `formatPrice`, `formatMarketCap`, `filterAndSort` ordering, and the three `CoinDetail` render states.

---

## 11. No caching, retry, or backoff

**Location:** `src/api/coinGecko.js`

**Observed behavior:** every call hits the network. There is no in-memory cache, request deduplication, timeout, retry, or handling of `429`/`5xx` beyond throwing.

**Impact:** rate limiting is likely with multiple tabs or rapid reloads; a transient failure leaves the UI empty until the next 30-second tick.

**Suggested fix:** add a small cache keyed by endpoint/id, an `AbortSignal.timeout`, and bounded retry with backoff for `429`/`5xx`. Document the policy in [API reference](api.md#rate-limits-and-reliability).

---

## 12. Sub-cent prices omit the currency symbol

**Location:** `src/utils/formatter.js:1-9`

```js
if (price < 0.01) return price.toFixed(8);
```

**Observed behavior:** values below one cent are returned as a bare number string with 8 decimals (e.g. `0.00012345`), while other values are currency-formatted with `$`.

**Impact:** inconsistent presentation for low-priced coins.

**Suggested fix:** return `` `$${price.toFixed(8)}` `` (or use `Intl` with `maximumFractionDigits: 8`) and update the examples in [Components](components.md#formatpriceprice).

---

## 13. API base URL and limits are hard-coded

**Location:** `src/api/coinGecko.js:1`, `src/pages/Home.jsx:14`

**Observed behavior:** the base URL, page size (`100`), and poll interval (`30000`) are literals. There is no `.env` support or config module.

**Impact:** switching to a proxy, API key, or different environment requires source edits; limits cannot be tuned per environment.

**Suggested fix:** read from `import.meta.env` with sensible defaults, and centralize constants. Document them in [Getting started — Configuration](getting-started.md#configuration).

---

## 14. Array index used as list key

**Location:** `src/pages/Home.jsx:107-109`

```jsx
{filteredList.map((crypto, key) => (
  <CryptoCard crypto={crypto} key={key} />
))}
```

**Observed behavior:** keys are array indices. When sorting or filtering changes the order, React reuses DOM nodes by position.

**Impact:** minor unnecessary re-renders; can cause stale local UI state if cards ever become stateful.

**Suggested fix:** use `key={crypto.id}`.

---

## 15. List is limited to the top 100 coins

**Location:** `src/api/coinGecko.js:6`, `src/pages/Home.jsx:35-40`

**Observed behavior:** only page 1 (`per_page=100`) is fetched, and search filters that single page. Coins outside the top 100 cannot be found.

**Impact:** search appears broken for smaller-cap coins.

**Suggested fix:** either label the list as "Top 100", add pagination/infinite scroll, or use CoinGecko's `/search` endpoint for global search. Document the chosen behavior.

---

## 16. Accessibility gaps

**Locations:** `src/components/CryptoCard.jsx:25`, `src/pages/CoinDetail.jsx:107`

**Observed behavior:**

- Direction arrow images have no `alt` text (decorative images should use `alt=""`).
- The change percentage is conveyed by color and arrow only; screen readers hear a bare number with no "up/down" context.
- View-toggle buttons do not expose `aria-pressed`.
- The search input has a placeholder but no associated `<label>` or `aria-label`.
- Focus styles are defined for inputs/selects but not for links or toggle buttons beyond hover.

**Impact:** the app is harder to use with assistive technology.

**Suggested fix:** add `alt=""` to decorative arrows, include a visually hidden "up/down" word or `aria-label`, add `aria-pressed` to toggle buttons, label the search input, and add visible `:focus-visible` styles.

---

## Not issues (intentional or acceptable)

- **No backend:** the app is deliberately client-only and calls a public API.
- **No TypeScript:** a valid choice for this project size; documented in the [README](../README.md#tech-stack).
- **No state library:** local state is sufficient for two pages.
- **Client-side sorting:** appropriate for a 100-item list.

## Fixed issues

_None recorded yet. When an issue above is resolved, move it here with the fix commit/PR reference so the history stays visible._

## Related documents

- [Architecture](architecture.md)
- [Pages](pages.md)
- [API reference](api.md)
- [Styling](styling.md)
- [Contributing](contributing.md)
