# Components & Utilities

Reference for the shared UI component, the app shell, and the pure formatting helpers. Page-level components are covered in [Pages](pages.md).

---

## `App` — `src/App.jsx`

**Purpose:** application shell. Declares the router and route table only; it holds no state and renders no chrome of its own.

```jsx
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/coin/:id" element={<CoinDetail />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### Behavior

| Concern | Detail |
| --- | --- |
| Router | `BrowserRouter` from `react-router` v7 (HTML5 history). Requires server fallback to `index.html` in production. |
| Routes | `/` → `Home`; `/coin/:id` → `CoinDetail`. |
| Fallback | None. Unmatched URLs render nothing. |
| Props | None. |
| Export | Default export. |

### Notes for maintainers

- Add new routes here and keep them in sync with the [Architecture route table](architecture.md#routing).
- `BrowserRouter` needs a rewrite rule on static hosts (e.g. serve `index.html` for unknown paths), otherwise a hard refresh on `/coin/bitcoin` returns 404 from the host.

### Suggested inline documentation

```jsx
/**
 * Root application component.
 *
 * Sets up client-side routing:
 * - "/" renders the market list (Home).
 * - "/coin/:id" renders a single coin's detail page (CoinDetail).
 *
 * @returns {JSX.Element} Router-wrapped route tree.
 */
```

---

## `main` — `src/main.jsx`

**Purpose:** browser entry point. Creates the React root, enables `StrictMode`, and imports global styles.

```jsx
createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

### Behavior

| Concern | Detail |
| --- | --- |
| Mount target | `#root` in `index.html`. |
| Strict mode | Enabled. In development this double-invokes effects, so polling effects mount/unmount twice; the cleanup prevents duplicate intervals. |
| Styles | Imports `./index.css` globally. |
| `App.css` | **Not imported** anywhere. |

### Suggested inline documentation

```jsx
/**
 * Application entry point.
 *
 * Mounts <App /> into #root using React 19's createRoot API and enables
 * StrictMode. Imports the global stylesheet (index.css).
 */
```

---

## `CryptoCard` — `src/components/CryptoCard.jsx`

**Purpose:** presentational card summarizing one cryptocurrency. Wrapped in a `Link` so the whole card navigates to the detail page.

### Signature

```ts
CryptoCard(props: { crypto: MarketCoin }): JSX.Element
```

### Props

| Prop | Type | Required | Description |
| --- | --- | --- | --- |
| `crypto` | `MarketCoin` | Yes | One element of the `fetchCryptos()` response. |

### Fields read from `crypto`

| Field | Render location |
| --- | --- |
| `id` | Link target: `/coin/{id}` |
| `image` | Logo `<img src>` |
| `name` | Card heading |
| `symbol` | Uppercased ticker |
| `market_cap_rank` | `#rank` badge |
| `current_price` | `formatPrice(...)` |
| `price_change_percentage_24h` | Direction icon + absolute percentage |
| `market_cap` | `formatMarketCap(...)` prefixed with `$` |
| `total_volume` | `formatMarketCap(...)` prefixed with `$` |

### Behavior

- **Direction styling:** `price_change_percentage_24h >= 0` adds `positive`, otherwise `negative`.
- **Icons:** `/arrow-up.png` or `/arrow-down.png` from `public/`.
- **Value:** always shows `Math.abs(change).toFixed(2)` followed by `%`, so the sign is conveyed only by color/icon.
- **Link styling:** inline `textDecoration: "none"` removes the underline.
- **Missing data:** the comparison `null >= 0` is `true`, so a null change renders as positive `0.00%`. A `null` price would throw in `formatPrice` (see below).
- **Accessibility:** the coin logo has `alt={crypto.name}`; the arrow images have **no `alt`**, so they are announced by filename or ignored depending on the screen reader.

### Suggested inline documentation

```jsx
/**
 * Card summarizing one cryptocurrency.
 *
 * Renders logo, name, ticker, market-cap rank, current USD price, the
 * 24-hour percentage change (absolute value with a direction arrow), market
 * cap, and 24-hour volume. The entire card links to `/coin/{crypto.id}`.
 *
 * @param {Object} props
 * @param {Object} props.crypto - CoinGecko market object from fetchCryptos().
 * @param {string} props.crypto.id - CoinGecko coin id used in the detail URL.
 * @param {string} props.crypto.name - Display name.
 * @param {string} props.crypto.symbol - Ticker symbol (uppercased for display).
 * @param {string} props.crypto.image - Logo URL.
 * @param {number|null} props.crypto.market_cap_rank - Rank badge value.
 * @param {number} props.crypto.current_price - Current price in USD.
 * @param {number|null} props.crypto.price_change_percentage_24h - 24h change.
 * @param {number} props.crypto.market_cap - Market capitalization in USD.
 * @param {number} props.crypto.total_volume - 24h volume in USD.
 * @returns {JSX.Element} Linked card element.
 */
```

---

## Formatter utilities — `src/utils/formatter.js`

Two pure functions with no dependencies, no state, and no error guards.

### `formatPrice(price)`

Formats a USD price for display.

```ts
formatPrice(price: number): string
```

| Input condition | Output | Example |
| --- | --- | --- |
| `price < 0.01` | `price.toFixed(8)` — **no `$` symbol** | `0.00012345` → `"0.00012345"` |
| `price >= 0.01` | `Intl.NumberFormat("en-US", { style: "currency", currency: "USD", min/maxFractionDigits: 2 })` | `1234.5` → `"$1,234.50"` |

Behavior notes:

- Values between `0.01` and `1` are rounded to **2 decimals**, so `0.0123` renders as `"$0.01"`.
- Negative sub-cent values also skip the currency symbol.
- `null`, `undefined`, or a non-number throws (`toFixed` on `null` / `Intl` on a non-number).
- `NaN` falls into the `>= 0.01` branch and renders `"$NaN"`.
- No localization parameter is exposed; currency and locale are fixed.

```js
formatPrice(0.00000012); // "0.00000012"
formatPrice(0.5);        // "$0.50"
formatPrice(64000);      // "$64,000.00"
```

**Suggested inline documentation:**

```js
/**
 * Format a USD price for display.
 *
 * Sub-cent values (< 0.01) are shown with 8 decimal places and no currency
 * symbol. All other values use en-US USD currency formatting with exactly
 * two fraction digits.
 *
 * @param {number} price - Price in USD.
 * @returns {string} Formatted price.
 */
```

### `formatMarketCap(marketCap)`

Abbreviates a large USD amount with a `T`/`B`/`M` suffix.

```ts
formatMarketCap(marketCap: number): string
```

| Input condition | Output | Example |
| --- | --- | --- |
| `>= 1e12` | `${(n / 1e12).toFixed(2)}T` | `2.5e12` → `"2.50T"` |
| `>= 1e9` | `${(n / 1e9).toFixed(2)}B` | `1.234e9` → `"1.23B"` |
| `>= 1e6` | `${(n / 1e6).toFixed(2)}M` | `5.5e6` → `"5.50M"` |
| otherwise | `n.toLocaleString()` | `12345` → `"12,345"` |

Behavior notes:

- The function returns **no `$` prefix**; callers add it (`CryptoCard` and `CoinDetail` render `$` separately).
- Boundary values use `>=`, so exactly `1e6` renders `"1.00M"`.
- Negative values fall through to `toLocaleString()`.
- `null` / `undefined` throw on `toLocaleString`.
- Abbreviations use binary-ish thresholds (1e12/1e9/1e6), which match common finance usage.

```js
formatMarketCap(1_234_567_890_000); // "1.23T"
formatMarketCap(987_654_321);       // "987.65M"
formatMarketCap(450_000);           // "450,000"
```

**Suggested inline documentation:**

```js
/**
 * Abbreviate a large USD amount using T (trillion), B (billion), or M
 * (million) suffixes with two decimals.
 *
 * Returns a plain number string (with thousands separators) below 1e6.
 * The currency symbol is not included; callers prepend "$".
 *
 * @param {number} marketCap - Amount in USD.
 * @returns {string} Abbreviated amount, e.g. "1.23T", "987.65M", "450,000".
 */
```

---

## Documentation status

| Item | Inline docs | This reference |
| --- | --- | --- |
| `App` | Missing | ✅ |
| `main` | Missing | ✅ |
| `CryptoCard` | Missing | ✅ |
| `formatPrice` | Missing | ✅ |
| `formatMarketCap` | Missing | ✅ |

## Related documents

- [Pages](pages.md)
- [Styling](styling.md)
- [Architecture](architecture.md)
