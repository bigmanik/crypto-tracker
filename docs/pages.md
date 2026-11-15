# Pages

Reference for the two route-level components. For the shared card and helpers they use, see [Components](components.md).

---

## `Home` — `src/pages/Home.jsx`

**Route:** `/`
**Purpose:** fetch the top-100 market list, poll it, and let the user search, sort, and switch between grid and list layouts.

### State

| State | Initial | Type | Purpose |
| --- | --- | --- | --- |
| `cryptoList` | `[]` | `MarketCoin[]` | Raw API result. |
| `filteredList` | `[]` | `MarketCoin[]` | Filtered + sorted result rendered in the UI. |
| `isLoading` | `true` | `boolean` | Shows the spinner. Set to `false` after the first request settles. |
| `viewMode` | `"grid"` | `"grid" \| "list"` | Container layout class. |
| `sortBy` | `"market_cap_rank"` | `string` | Active sort key. |
| `searchQuery` | `""` | `string` | Search text. |

### Effects

| Effect | Dependencies | Behavior |
| --- | --- | --- |
| Polling | `[]` | `setInterval(fetchCryptoData, 30000)`; cleanup clears the interval. **Does not call the fetch immediately.** |
| Filter/sort | `[sortBy, cryptoList, searchQuery]` | Runs `filterAndSort()` after mount and whenever an input changes. |

### Functions

#### `fetchCryptoData()`

```ts
fetchCryptoData(): Promise<void>
```

- Calls `fetchCryptos()`.
- On success: `setCryptoList(data)`.
- On failure: `console.error("Error fetching crypto: ", err)`; existing list is kept.
- Always: `setIsLoading(false)`.

#### `filterAndSort()`

```ts
filterAndSort(): void
```

1. Filters `cryptoList` where `name` **or** `symbol` contains `searchQuery` (case-insensitive).
2. Sorts the filtered array **in place** using `sortBy`.
3. Stores the result in `filteredList`.

Filtering and sorting are client-side and apply only to the 100 coins currently loaded.

### Sort options

| `sortBy` | UI label | Comparator | Order |
| --- | --- | --- | --- |
| `market_cap_rank` (default) | Rank | `a.market_cap_rank - b.market_cap_rank` | Best rank first |
| `name` | Name | `a.name.localeCompare(b.name)` | A → Z |
| `price` | Price (low to high) | `a.current_price - b.current_price` | Cheapest first |
| `price_desc` | Price (high to low) | `b.current_price - a.current_price` | Most expensive first |
| `change` | 24hr Change | `a.price_change_percentage_24h - b.price_change_percentage_24h` | **Worst first** (ascending) |
| `market_cap` | Market cap | `a.market_cap - b.market_cap` | Smallest first |

Notes:

- `change` and `market_cap` are ascending only; there is no descending variant.
- Ties are left in whatever order the filter produced.
- `null` numeric fields coerce to `0` in subtraction, so they sort as zero.

### UI structure

```text
.app
├── header.header
│   └── .header-content
│       ├── .logo-section (title + subtitle)
│       └── .search-section > input.search-input
├── .controls
│   ├── .filter-group (label + <select>)
│   └── .view-toggle (Grid / List buttons)
├── .loading (spinner)  OR  .crypto-container.grid|.list
│   └── CryptoCard × N
└── footer.footer (CoinGecko attribution)
```

### Behavior notes and caveats

- **First request is delayed ~30 s** — see [Known issues #1](known-issues.md#1-home-page-waits-30-seconds-before-the-first-request).
- **No empty state** — if search matches nothing, the container renders empty with no message.
- **No error state** — failures only appear in the console; the spinner disappears after the first failed attempt.
- **No debounce** — every keystroke triggers a filter/sort pass (fine for 100 items).
- **List keys** use the array index (`key={key}`), not `crypto.id`; reordering can reuse DOM nodes unnecessarily.
- **Polling continues** every 30 s while mounted and stops on unmount.
- **Header is duplicated** in `CoinDetail`; there is no shared layout component.

### Suggested inline documentation

```jsx
/**
 * Home page ("/").
 *
 * Fetches the top 100 coins by market cap, refreshes them every 30 seconds,
 * and renders them as cards. Supports case-insensitive search over name and
 * symbol, six sort keys, and grid/list view modes. All filtering and sorting
 * are client-side over the loaded page of coins.
 *
 * @returns {JSX.Element} Market list screen.
 */
```

---

## `CoinDetail` — `src/pages/CoinDetail.jsx`

**Route:** `/coin/:id`
**Purpose:** show one coin's price summary, 7-day chart, and supply/volume statistics.

### Route params

| Param | Source | Description |
| --- | --- | --- |
| `id` | `useParams()` | CoinGecko coin id, e.g. `bitcoin`. Passed unchanged to both API calls. |

### State

| State | Initial | Type | Purpose |
| --- | --- | --- | --- |
| `coin` | `null` | `CoinDetail \| null` | `/coins/{id}` response. `null` means "not loaded or failed". |
| `chartData` | `[]` | `{ time: string, price: string }[]` | Chart points. |
| `isLoading` | `true` | `boolean` | Shared spinner flag. |

### Effects

| Effect | Dependencies | Behavior |
| --- | --- | --- |
| Load | `[id]` | Calls `loadCoinData()` and `loadChartData()` in parallel. Re-runs when the route param changes. |

There is no cleanup/abort, so responses from a previous `id` can arrive after navigation and overwrite state.

### Functions

#### `loadCoinData()`

```ts
loadCoinData(): Promise<void>
```

- `fetchCoinData(id)` → `setCoin(data)`.
- On error: `console.error("Error fetching crypto: ", err)`.
- Always: `setIsLoading(false)`.

#### `loadChartData()`

```ts
loadChartData(): Promise<void>
```

- `fetchChartData(id)` → transform → `setChartData(...)`.
- Transformation:

  ```js
  data.prices.map(([timestampMs, price]) => ({
    time: new Date(timestampMs).toLocaleDateString("en-US", {
      month: "short",
      day: "numeric",
    }),
    price: price.toFixed(2),
  }));
  ```

- On error: `console.error("Error fetching crypto: ", err)`.
- Always: `setIsLoading(false)`.

Transform caveats:

- `price` is a **string**; Recharts receives string values.
- `time` has no year and repeats for multiple points on the same day, producing duplicate x-axis labels.
- Local timezone is used for date labels.

### Render states

| Condition | UI |
| --- | --- |
| `isLoading === true` | `.loading` with `.spinner` and "Loading coin data…". |
| `isLoading === false && coin === null` | `.no-results` with "coin not found" and a **Go Back** button (`navigate("/")`). |
| `coin` present | Full detail UI. |

Because both requests share `isLoading`, the chart request finishing first can briefly render the "not found" state before the coin request resolves. See [Known issues #3](known-issues.md#3-detail-page-can-show-coin-not-found-while-loading).

### Data displayed

| Section | Source field | Formatter |
| --- | --- | --- |
| Logo | `coin.image.large` | — |
| Name / ticker | `coin.name`, `coin.symbol` | `.toUpperCase()` |
| Rank | `coin.market_data.market_cap_rank` | `#` prefix |
| Current price | `coin.market_data.current_price.usd` | `formatPrice` |
| 24h change | `coin.price_change_percentage_24h` (⚠️ wrong level) | `Math.abs(...).toFixed(2)` + `%` |
| 24h high / low | `coin.market_data.high_24h.usd`, `low_24h.usd` | `formatPrice` |
| Chart | `chartData` | Recharts `LineChart` |
| Market cap | `coin.market_data.market_cap.usd` | `formatMarketCap` |
| Volume | `coin.market_data.total_volume.usd` | `formatMarketCap` |
| Circulating supply | `coin.market_data.circulating_supply` | `toLocaleString()` or `"N/A"` |
| Total supply | `coin.market_data.total_supply` | `toLocaleString()` or `"N/A"` |

### Chart configuration

| Recharts element | Setting |
| --- | --- |
| Container | `ResponsiveContainer` width `100%`, height `400` |
| Chart | `LineChart` with `chartData` |
| Grid | `CartesianGrid` dashed, `rgba(255,255,255,0.1)` |
| X axis | `dataKey="time"`, stroke `#9ca3af`, 12px |
| Y axis | auto domain, stroke `#9ca3af`, 12px |
| Tooltip | dark translucent background, 8px radius |
| Line | `type="monotone"`, `dataKey="price"`, stroke `#ADD8E6`, width 2, no dots |

### UI structure

```text
.app
├── header.header (same markup as Home)
├── .coin-detail
│   ├── .coin-header
│   │   ├── .cointitle  ⚠️ (CSS defines .coin-title)
│   │   └── .rank
│   ├── .coin-price-section
│   │   ├── .current-price (price + .change-badge)
│   │   └── .price-ranges (24h high/low)
│   ├── .chart-section
│   ├── .stats-grid > .stat-card × 4
└── footer.footer ("updated every 30 seconds")
```

### Behavior notes and caveats

- **Change badge is always `0.00%`** — see [Known issues #2](known-issues.md#2-detail-page-24-hour-change-always-shows-000).
- **`cointitle` typo** means the logo/title block does not receive `.coin-title` styles — see [Known issues #4](known-issues.md#4-detail-page-title-block-uses-the-wrong-css-class).
- **Footer claims 30-second updates**, but the detail page never polls; it fetches once per mount.
- **No optional chaining** on `coin.market_data.*`; a malformed response throws during render.
- **No back navigation in the success view**; the only back button is in the not-found state. Users rely on browser back.
- **No `alt` text on the arrow images.**
- **No chart empty state**; if chart data fails, an empty chart renders silently.

### Suggested inline documentation

```jsx
/**
 * Coin detail page ("/coin/:id").
 *
 * Reads the CoinGecko coin id from the route, loads the coin record and its
 * 7-day market chart in parallel, and renders the current price, 24h
 * high/low, market cap, volume, supplies, and a Recharts line chart.
 *
 * Shows a spinner while loading and a "coin not found" screen when the coin
 * record is unavailable.
 *
 * @returns {JSX.Element} Coin detail screen.
 */
```

---

## Page comparison

| Concern | `Home` | `CoinDetail` |
| --- | --- | --- |
| Route | `/` | `/coin/:id` |
| API calls | `fetchCryptos` | `fetchCoinData`, `fetchChartData` |
| Polling | Every 30 s (after initial delay) | None |
| Search | Yes | No |
| Sorting | Yes | No |
| Chart | No | Yes (7-day line) |
| Loading UI | Spinner | Spinner |
| Empty/error UI | None | "coin not found" |

## Related documents

- [Components](components.md)
- [API reference](api.md)
- [Styling](styling.md)
- [Known issues](known-issues.md)
