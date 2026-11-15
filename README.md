# Crypto Tracker

A single-page cryptocurrency market dashboard built with **React 19**, **Vite 7**, **React Router 7**, and **Recharts**. It shows the top 100 cryptocurrencies by market cap, lets you search and sort them, and opens a detail view with market statistics and a 7-day price chart. All market data comes from the public [CoinGecko API](https://www.coingecko.com/en/api).

The UI brands itself **"BigmanIk Crypto Tracker"**.

---

## Features

- **Top 100 market list** — fetched from CoinGecko, ordered by market cap.
- **Live search** — filters by coin name or ticker symbol (case-insensitive).
- **Sorting** — by rank, name, price (ascending/descending), 24-hour change, or market cap.
- **Grid and list layouts** — toggle without reloading.
- **Coin detail page** — current price, 24-hour high/low, market cap, volume, circulating supply, total supply, and a 7-day line chart.
- **Loading states** — animated spinner while data is in flight.
- **Responsive dark UI** — glassmorphism styling with a single mobile breakpoint.

---

## Tech stack

| Concern | Technology |
| --- | --- |
| UI library | React 19 (`react`, `react-dom`) |
| Build tool | Vite 7 |
| Routing | React Router 7 (`react-router`) |
| Charts | Recharts 3 |
| Styling | Plain CSS (`src/index.css`) |
| Linting | ESLint 9 + `eslint-plugin-react-hooks` + `eslint-plugin-react-refresh` |
| Language | JavaScript / JSX (no TypeScript) |
| Tests | None configured |

---

## Quick start

### Prerequisites

- **Node.js 20.19+ or 22.12+** (required by Vite 7)
- npm (a `package-lock.json` is committed)

### Install and run

```bash
npm install
npm run dev
```

Vite prints a local URL (typically `http://localhost:5173`). Open it in a browser.

> **First-load note:** the home page currently waits ~30 seconds before its first data request because the polling interval is never invoked immediately. See [Known Issues](docs/known-issues.md#1-home-page-waits-30-seconds-before-the-first-request).

### Available scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the Vite development server with HMR. |
| `npm run build` | Produce a production build in `dist/`. |
| `npm run preview` | Serve the production build locally. |
| `npm run lint` | Run ESLint over the project. |

---

## Project structure

```text
crypto-tracker/
├── public/                     # Static assets served from /
│   ├── arrow-down.png
│   ├── arrow-up.png
│   └── vite.svg
├── src/
│   ├── api/
│   │   └── coinGecko.js        # CoinGecko API client (3 exported functions)
│   ├── components/
│   │   └── CryptoCard.jsx      # Summary card for one coin
│   ├── pages/
│   │   ├── Home.jsx            # "/" — market list, search, sort, view toggle
│   │   └── CoinDetail.jsx      # "/coin/:id" — stats + 7-day chart
│   ├── utils/
│   │   └── formatter.js        # formatPrice / formatMarketCap helpers
│   ├── App.css                 # Unused Vite template styles (see docs/styling.md)
│   ├── App.jsx                 # BrowserRouter + route table
│   ├── index.css               # Global app styles and design tokens
│   └── main.jsx                # React entry point
├── docs/                       # Project documentation (start at docs/README.md)
├── eslint.config.js
├── index.html
├── package.json
└── vite.config.js
```

---

## Routes

| Path | Component | Description |
| --- | --- | --- |
| `/` | `Home` | Searchable, sortable list of the top 100 coins. |
| `/coin/:id` | `CoinDetail` | Detail view for one CoinGecko coin id (e.g. `/coin/bitcoin`). |

Unknown paths are not handled and render an empty route.

---

## Data source

- **API:** CoinGecko public API v3 (`https://api.coingecko.com/api/v3`)
- **Authentication:** none (no API key is used)
- **Network calls:**
  - `GET /coins/markets` — once per 30-second poll on the home page (100 coins).
  - `GET /coins/{id}` — once per detail-page visit.
  - `GET /coins/{id}/market_chart` — once per detail-page visit (7 days).
- **Rate limits:** CoinGecko throttles the free public API. There is no caching, retry, or backoff logic. If requests fail, errors are logged to the console and the UI keeps its last state.

See [docs/api.md](docs/api.md) for the full client reference and response shapes.

---

## Documentation

Full documentation lives in [`docs/`](docs/README.md):

- [Documentation index](docs/README.md)
- [Getting started](docs/getting-started.md) — install, scripts, configuration, troubleshooting
- [Architecture](docs/architecture.md) — structure, routing, data flow, state
- [API reference](docs/api.md) — `src/api/coinGecko.js` and CoinGecko endpoints
- [Components](docs/components.md) — `CryptoCard`, `App`, `main`, formatter utilities
- [Pages](docs/pages.md) — `Home` and `CoinDetail` behavior and state
- [Styling](docs/styling.md) — design tokens, class map, responsive rules
- [Contributing](docs/contributing.md) — conventions, workflows, documentation standards
- [Known issues](docs/known-issues.md) — documented defects and limitations

---

## Known limitations

These are documented in detail in [docs/known-issues.md](docs/known-issues.md). Highlights:

- The home page does not fetch data until 30 seconds after mount.
- The detail page's 24-hour change badge always shows `0.00%` (it reads a field the API does not return at that level).
- The detail page can briefly show "coin not found" while data is still loading.
- Errors are only logged to the console; there is no user-facing error state.
- No tests, no TypeScript, and the API base URL is hard-coded.

---

## Contributing

See [docs/contributing.md](docs/contributing.md) for setup, coding conventions, and the documentation checklist.

---

## License

No license file is present in this repository. Add one before distributing or open-sourcing the project.
