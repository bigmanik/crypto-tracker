# Getting Started

This guide covers installing, running, building, and troubleshooting the Crypto Tracker.

## Prerequisites

| Requirement | Version | Notes |
| --- | --- | --- |
| Node.js | **20.19+ or 22.12+** | Vite 7 requires one of these ranges. Check with `node -v`. |
| npm | Any version bundled with the supported Node release | A `package-lock.json` is committed; npm is the assumed package manager. |
| Network access | — | The app calls the public CoinGecko API at runtime. |

No API key, `.env` file, or other configuration is required.

## Install

```bash
npm install
```

## Run in development

```bash
npm run dev
```

Vite starts a dev server (default `http://localhost:5173`) with hot module replacement.

### What you should see

1. The dark "BigmanIk Crypto Tracker" header and controls render immediately.
2. A spinner labelled "loading crypto data…" appears.
3. **After roughly 30 seconds**, the first batch of coins appears.

The delay is a known defect, not a slow network: the home page sets up a 30-second interval but never performs the initial fetch. See [Known issues #1](known-issues.md#1-home-page-waits-30-seconds-before-the-first-request). To verify your setup without waiting, refresh the page after the interval has fired once, or temporarily call the fetch on mount in a local experiment (do not commit that change as part of documentation work).

## Build for production

```bash
npm run build
```

The static output is written to `dist/`. It can be served by any static host.

## Preview the production build

```bash
npm run preview
```

## Lint

```bash
npm run lint
```

ESLint is configured in `eslint.config.js` with the recommended JS rules, React Hooks rules, and React Refresh rules. It ignores `dist/`.

## Configuration

There is **no runtime configuration system**:

- The API base URL is hard-coded as `https://api.coingecko.com/api/v3` in `src/api/coinGecko.js:1`.
- There are no `import.meta.env` reads and no `.env` files.
- Polling interval (`30000` ms) and page size (`100`) are hard-coded in `src/pages/Home.jsx` and `src/api/coinGecko.js`.

To point the app at a proxy or a different environment, you must change the constant in source (and update [api.md](api.md)).

## Troubleshooting

| Symptom | Likely cause | What to do |
| --- | --- | --- |
| Blank page, console error about `React` or JSX | Dependency install incomplete or wrong Node version | Delete `node_modules` and `package-lock.json`, reinstall with a supported Node version. |
| Spinner for ~30 seconds on first load | Known defect in `Home.jsx` | Wait, or read [Known issues #1](known-issues.md#1-home-page-waits-30-seconds-before-the-first-request). |
| "Failed to fetch cryptos" / "Failed to fetch coin data" in console | CoinGecko returned a non-2xx response (often rate limiting) | Wait a minute and reload. There is no retry or backoff. |
| Network request blocked in the browser | Corporate proxy, ad blocker, or CORS policy | Confirm `api.coingecko.com` is reachable; a dev proxy is not configured. |
| Detail page shows "coin not found" briefly | Known loading race | See [Known issues #3](known-issues.md#3-detail-page-can-show-coin-not-found-while-loading). |
| Detail page change badge shows `0.00%` | Known wrong-field bug | See [Known issues #2](known-issues.md#2-detail-page-24-hour-change-always-shows-000). |
| Styles look plain / no dark theme | `src/index.css` not imported | `src/main.jsx` must import `./index.css`. |

## Related documents

- [Architecture](architecture.md)
- [API reference](api.md)
- [Known issues](known-issues.md)
