# API Reference

All network access is centralized in [`src/api/coinGecko.js`](../src/api/coinGecko.js). It exports three async functions and talks to the public CoinGecko v3 API. There is no SDK, client instance, caching, retry, or authentication.

## Module overview

| Export | Purpose | HTTP request |
| --- | --- | --- |
| `fetchCryptos()` | Top-100 market list | `GET /coins/markets` |
| `fetchCoinData(id)` | Full detail record for one coin | `GET /coins/{id}` |
| `fetchChartData(id)` | 7-day price series for one coin | `GET /coins/{id}/market_chart` |

**Base URL (hard-coded):** `https://api.coingecko.com/api/v3`

```js
const BASE_URL = "https://api.coingecko.com/api/v3";
```

## `fetchCryptos()`

Fetches the first page of coins ordered by market cap, in USD, without sparkline data.

```js
export const fetchCryptos = async () => { … }
```

### Signature

```ts
fetchCryptos(): Promise<MarketCoin[]>
```

### Request

```http
GET https://api.coingecko.com/api/v3/coins/markets
  ?vs_currency=usd
  &order=market_cap_desc
  &per_page=100
  &page=1
  &sparkline=false
```

### Parameters

None. Currency, ordering, page, page size, and sparkline are fixed in the query string.

### Returns

A `Promise` resolving to an array of up to 100 market objects. Fields consumed by the app:

| Field | Type | Used by |
| --- | --- | --- |
| `id` | `string` | Detail link (`/coin/:id`) |
| `name` | `string` | Card title, search |
| `symbol` | `string` | Card ticker, search |
| `image` | `string` (URL) | Card logo |
| `market_cap_rank` | `number \| null` | Card rank badge, default sort |
| `current_price` | `number` | Price display, price sort |
| `price_change_percentage_24h` | `number \| null` | Change badge, change sort |
| `market_cap` | `number` | Market-cap stat, market-cap sort |
| `total_volume` | `number` | Volume stat |

The response contains many more fields; they are ignored.

### Errors

- Non-2xx response → throws `Error("Failed to fetch cryptos")`.
- Network failure / CORS block → the underlying `fetch` rejects with `TypeError`.
- Malformed JSON → `response.json()` rejects with a `SyntaxError`.

No timeout, abort signal, retry, or fallback is applied.

### Example

```js
import { fetchCryptos } from "./api/coinGecko";

const coins = await fetchCryptos();
console.log(coins[0].name); // e.g. "Bitcoin"
```

### Suggested inline documentation

The source currently has no comments. This JSDoc block describes the existing behavior and can be pasted above the function when inline docs are desired:

```js
/**
 * Fetch the top 100 cryptocurrencies by market capitalization in USD.
 *
 * Calls `GET /coins/markets` with `vs_currency=usd`, `order=market_cap_desc`,
 * `per_page=100`, `page=1`, and `sparkline=false`.
 *
 * @async
 * @returns {Promise<Array<Object>>} Array of CoinGecko market objects. Each item
 *   includes `id`, `name`, `symbol`, `image`, `market_cap_rank`,
 *   `current_price`, `price_change_percentage_24h`, `market_cap`, and
 *   `total_volume`.
 * @throws {Error} "Failed to fetch cryptos" when the response status is not OK.
 */
```

## `fetchCoinData(id)`

Fetches the full record for one coin, with localization, tickers, community data, developer data, and sparkline disabled.

```js
export const fetchCoinData = async (id) => { … }
```

### Signature

```ts
fetchCoinData(id: string): Promise<CoinDetail>
```

### Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | Yes | CoinGecko coin id, for example `"bitcoin"`, `"ethereum"`, `"solana"`. This is the `:id` route param, not the ticker symbol. |

The id is interpolated into the URL without `encodeURIComponent`. Normal coin ids are URL-safe, but unusual values could break the request.

### Request

```http
GET https://api.coingecko.com/api/v3/coins/{id}
  ?localization=false
  &tickers=false
  &market_data=true
  &community_data=false
  &developer_data=false
  &sparkline=false
```

### Returns

A `Promise` resolving to a coin object. Fields consumed by `CoinDetail`:

| Field | Type | Used for |
| --- | --- | --- |
| `name` | `string` | Page title |
| `symbol` | `string` | Ticker |
| `image.large` | `string` (URL) | Coin logo |
| `market_data.market_cap_rank` | `number \| null` | Rank badge |
| `market_data.current_price.usd` | `number` | Current price |
| `market_data.high_24h.usd` | `number` | 24-hour high |
| `market_data.low_24h.usd` | `number` | 24-hour low |
| `market_data.market_cap.usd` | `number` | Market-cap card |
| `market_data.total_volume.usd` | `number` | Volume card |
| `market_data.circulating_supply` | `number \| null` | Supply card |
| `market_data.total_supply` | `number \| null` | Supply card |

> **Important:** the 24-hour percentage change on the detail page is read from `coin.price_change_percentage_24h` at the **top level**, but CoinGecko returns it under `market_data.price_change_percentage_24h`. As a result the badge always shows `0.00%`. See [Known issues #2](known-issues.md#2-detail-page-24-hour-change-always-shows-000).

### Errors

- Non-2xx response (including an unknown coin id) → throws `Error("Failed to fetch coin data")`.
- Network failure → `fetch` rejects with `TypeError`.

### Example

```js
import { fetchCoinData } from "./api/coinGecko";

const bitcoin = await fetchCoinData("bitcoin");
console.log(bitcoin.market_data.current_price.usd);
```

### Suggested inline documentation

```js
/**
 * Fetch detailed market data for a single coin.
 *
 * Calls `GET /coins/{id}` with localization, tickers, community data,
 * developer data, and sparkline disabled.
 *
 * @async
 * @param {string} id - CoinGecko coin id (e.g. "bitcoin"). Not the ticker symbol.
 * @returns {Promise<Object>} CoinGecko coin object. Market values are under
 *   `market_data` (e.g. `market_data.current_price.usd`,
 *   `market_data.market_cap.usd`, `market_data.price_change_percentage_24h`).
 * @throws {Error} "Failed to fetch coin data" when the response status is not OK.
 */
```

## `fetchChartData(id)`

Fetches the 7-day market chart series for one coin in USD.

```js
export const fetchChartData = async (id) => { … }
```

### Signature

```ts
fetchChartData(id: string): Promise<MarketChart>
```

### Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | Yes | CoinGecko coin id, same value used by `fetchCoinData`. |

### Request

```http
GET https://api.coingecko.com/api/v3/coins/{id}/market_chart
  ?vs_currency=usd
  &days=7
```

### Returns

A `Promise` resolving to an object with parallel time series:

| Field | Type | Description |
| --- | --- | --- |
| `prices` | `Array<[number, number]>` | `[timestampMs, priceUsd]` pairs. Used by the chart. |
| `market_caps` | `Array<[number, number]>` | Not used by the app. |
| `total_volumes` | `Array<[number, number]>` | Not used by the app. |

`CoinDetail` maps `prices` to `{ time: "Mon D", price: "1234.56" }`. The `price` value is a **string**, not a number.

### Errors

- Non-2xx response → throws `Error("Failed to fetch chart data")`.
- Network failure → `fetch` rejects with `TypeError`.

### Example

```js
import { fetchChartData } from "./api/coinGecko";

const chart = await fetchChartData("bitcoin");
const latest = chart.prices.at(-1); // [timestamp, price]
```

### Suggested inline documentation

```js
/**
 * Fetch the 7-day USD market chart for a single coin.
 *
 * Calls `GET /coins/{id}/market_chart?vs_currency=usd&days=7`.
 *
 * @async
 * @param {string} id - CoinGecko coin id (e.g. "bitcoin").
 * @returns {Promise<{prices: Array<[number, number]>,
 *   market_caps: Array<[number, number]>,
 *   total_volumes: Array<[number, number]>}>}
 *   Time series as `[timestampMs, value]` pairs.
 * @throws {Error} "Failed to fetch chart data" when the response status is not OK.
 */
```

## External API notes

### Endpoints used

| Endpoint | When | Calls per session |
| --- | --- | --- |
| `/coins/markets` | Home page poll | 1 every 30 s (after the initial delay) |
| `/coins/{id}` | Detail page mount | 1 per visit |
| `/coins/{id}/market_chart` | Detail page mount | 1 per visit |

### Rate limits and reliability

- The public API is rate-limited; CoinGecko changes the exact quota over time. Heavy reloading or multiple tabs can trigger `429 Too Many Requests`.
- The app does **not** send an API key, cache responses, deduplicate in-flight requests, or implement exponential backoff.
- A `429` is surfaced as the generic thrown `Error` messages above and logged to the console.
- Consider a dev-server proxy or a server-side proxy if the browser blocks the API (corporate networks, some regions).

### Response type reference

The module does not define TypeScript types. The field tables above describe the subset the UI depends on. Full schemas are published in the [CoinGecko API docs](https://docs.coingecko.com/reference/coins-markets).

## Related documents

- [Architecture](architecture.md)
- [Pages](pages.md)
- [Known issues](known-issues.md)
