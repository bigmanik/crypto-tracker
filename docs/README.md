# Crypto Tracker — Documentation

Welcome to the project documentation. The app is a React + Vite dashboard that displays cryptocurrency market data from the public CoinGecko API.

## Start here

| I want to… | Read |
| --- | --- |
| Understand what the app does | [Root README](../README.md) |
| Install and run the app | [Getting started](getting-started.md) |
| Understand how the pieces fit together | [Architecture](architecture.md) |
| Call or extend the CoinGecko client | [API reference](api.md) |
| Work on the shared component or helpers | [Components](components.md) |
| Work on the list or detail screen | [Pages](pages.md) |
| Change the look and feel | [Styling](styling.md) |
| Contribute code or docs | [Contributing](contributing.md) |
| Check known bugs before reporting one | [Known issues](known-issues.md) |

## Documentation map

```text
docs/
├── README.md          # This index
├── getting-started.md # Install, scripts, configuration, troubleshooting
├── architecture.md    # Folder layout, routing, data flow, state management
├── api.md             # src/api/coinGecko.js reference + CoinGecko endpoints
├── components.md      # App shell, CryptoCard, formatter utilities
├── pages.md           # Home and CoinDetail behavior
├── styling.md         # Design tokens, CSS class map, responsive rules
├── contributing.md    # Conventions, workflows, docs standards
└── known-issues.md    # Documented defects and limitations
```

## Documentation conventions

- **Paths** are written relative to the repository root, for example `src/api/coinGecko.js`.
- **Code identifiers** use backticks: `fetchCryptos`, `CryptoCard`.
- **Line references** use the form `src/pages/Home.jsx:13-17` and refer to the state of the code when these docs were written.
- Documentation describes **current behavior**, including defects. Defects are collected in [Known issues](known-issues.md) rather than hidden.

## Keeping docs current

When you change behavior, update the matching page here in the same pull request. The [contributing guide](contributing.md#documentation-standards) lists which file to touch for each kind of change.
