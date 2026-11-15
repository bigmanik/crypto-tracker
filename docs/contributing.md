# Contributing

Thanks for improving Crypto Tracker. This guide covers setup, conventions, and the documentation expectations for changes.

## Before you start

1. Read [Getting started](getting-started.md) and [Architecture](architecture.md).
2. Check [Known issues](known-issues.md) so you do not re-report or accidentally reintroduce a documented defect.
3. Install dependencies with `npm install`.

## Workflow

```bash
# 1. Create a branch
git checkout -b feat/short-description

# 2. Make changes
npm run dev        # verify in the browser
npm run lint       # must pass

# 3. Build check (recommended for routing/CSS changes)
npm run build

# 4. Commit and open a pull request
```

There is no test suite, so manual verification in the browser is required. Describe what you clicked and what you observed in the pull request.

## Code conventions

These reflect the existing codebase. Follow them for consistency.

### General

- JavaScript + JSX only; the project is not typed.
- ES modules (`import` / `export`), `"type": "module"` in `package.json`.
- 2-space indentation in most files; preserve the surrounding style when editing.
- Semicolons are used in `src/api`, `src/pages`, and `src/utils`; the app shell omits them. Match the file you touch.
- Keep comments meaningful; the codebase currently has almost none.

### Components and pages

- Function components with hooks.
- Pages and `CryptoCard` use **named exports** (`export const Home = …`); `App` uses a **default export**.
- Keep API access in `src/api`, formatting in `src/utils`, and presentational pieces in `src/components`.
- Page files own their own state; do not introduce a global store without documenting the decision in [Architecture](architecture.md#state-management).

### API layer

- One exported async function per endpoint.
- Throw an `Error` with a clear message on non-2xx responses.
- Keep query parameters explicit in the template string.
- Document any new function in [API reference](api.md) with signature, request, returns, errors, and an example.

### Styling

- Add selectors to `src/index.css`, grouped under the existing section comments.
- Reuse the palette documented in [Styling](styling.md#color-palette); do not introduce near-duplicate colors.
- Respect the single `768px` breakpoint, or document a new one.
- Do not add styles to the unused `src/App.css`.

### Routing

- Register new routes in `src/App.jsx`.
- Update the route tables in [README](../README.md#routes) and [Architecture](architecture.md#routing).
- Remember production hosts need an `index.html` fallback for `BrowserRouter`.

## Documentation standards

Documentation lives in `docs/` and the root `README.md`. Update docs **in the same change** as the behavior they describe.

| If you change… | Update |
| --- | --- |
| Features, scripts, tech stack, routes | `README.md` |
| Install steps, configuration, troubleshooting | `docs/getting-started.md` |
| Folder structure, routing, data flow, state, error handling | `docs/architecture.md` |
| An exported API function or endpoint usage | `docs/api.md` |
| `CryptoCard`, `App`, `main`, or formatter helpers | `docs/components.md` |
| `Home` or `CoinDetail` behavior/state | `docs/pages.md` |
| CSS classes, palette, layout, breakpoints | `docs/styling.md` |
| Fix or introduce a limitation | `docs/known-issues.md` (remove fixed items) |
| Add a new documentation file | `docs/README.md` index and the docs list in `README.md` |

### Writing style

- Use short sections and tables for reference material.
- Write paths relative to the repository root.
- Prefer present tense: "Fetches the top 100 coins", not "will fetch".
- Describe what the code **does**, including quirks; do not claim behavior that is not implemented.
- Include a minimal code example for every public function.
- When documenting a defect, state the location, the observed behavior, the impact, and a suggested fix — but do not silently "fix it in the docs".

### Inline documentation

Most functions lack inline JSDoc. When you touch a function, add a short JSDoc block:

```js
/**
 * One-sentence summary.
 *
 * Optional details about side effects, endpoints, or edge cases.
 *
 * @param {Type} name - Description.
 * @returns {Type} Description.
 * @throws {Error} When and why.
 */
```

Proposed JSDoc for the current API, components, and utilities is already written in [API reference](api.md#suggested-inline-documentation), [Components](components.md), and [Pages](pages.md); copy it into source when you are already editing those files.

## Adding a feature

### New page

1. Create `src/pages/YourPage.jsx` with a named export.
2. Add a `<Route>` in `src/App.jsx`.
3. Add the route to the tables in `README.md` and `docs/architecture.md`.
4. Document behavior and state in `docs/pages.md`.
5. Add any new CSS to `src/index.css` and document classes in `docs/styling.md`.

### New API call

1. Add the function to `src/api/coinGecko.js` following the existing error pattern.
2. Add a section to `docs/api.md` (signature, request, params, returns, errors, example, suggested JSDoc).
3. Note the endpoint and call frequency in the "External API notes" table.

### New formatter

1. Add the function to `src/utils/formatter.js`.
2. Document its table of behaviors and examples in `docs/components.md`.

## Pull request checklist

- [ ] `npm run lint` passes.
- [ ] `npm run build` succeeds for routing, asset, or CSS changes.
- [ ] Manually verified the affected screens (including mobile width if styling changed).
- [ ] No new console errors under normal conditions.
- [ ] Documentation updated per the table above.
- [ ] `docs/known-issues.md` updated if a limitation was fixed or introduced.
- [ ] No changes to `src/App.css` (it is unused).

## Reporting bugs

Use [Known issues](known-issues.md) first. When reporting a new bug, include:

- Route and steps to reproduce.
- Expected vs. actual behavior.
- Browser and Node version.
- Console output, especially the fixed error messages thrown by `src/api/coinGecko.js`.

## Related documents

- [Documentation index](README.md)
- [Architecture](architecture.md)
- [Known issues](known-issues.md)
