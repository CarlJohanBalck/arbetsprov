# CLAUDE.md

This file provides guidance for AI assistants working on the `arbetsprov` codebase.

## Project Overview

A React-based RSS news aggregator that fetches and displays articles from the Swedish newspaper Dagens Nyheter (DN). It is a client-side-only single-page application bootstrapped with Create React App.

Key features:
- Fetches the DN RSS feed via a CORS proxy
- Displays articles in a card-based slider (navigate with Bakåt/Nästa buttons)
- Supports search filtering by article title prefix
- Allows users to "like" (favorite) individual articles
- Toast notifications for feed fetch status

## Repository Structure

```
src/
  App.js              # Root component (wraps News)
  App.scss            # SCSS styles for the slider and layout
  App.css             # Compiled CSS (do not edit directly; edit App.scss)
  index.js            # React entry point; imports Bootstrap and Font Awesome
  config.json         # API configuration (RSS endpoint URL)
  components/
    news.jsx          # Main stateful component; orchestrates feed, search, slider
    Card.jsx          # Presentational card for a single article
  common/
    like.jsx          # Like/favorite toggle button (functional component)
    searchbox.jsx     # Search input (functional component)
public/               # Static HTML template, PWA manifest, icons
```

## Development Workflow

### Setup

```bash
npm install
```

### Run locally

```bash
npm start
# Opens http://localhost:3000
```

### Build for production

```bash
npm build
```

### Run tests

```bash
npm test
```

There is currently only a single smoke test (`src/App.test.js`). No comprehensive test suite exists.

## Key Conventions

### Component style
- Class components are used for stateful logic (`News`, `Card`, `App`).
- Functional components are used for presentational leaves (`Like`, `SearchBox`).
- Do not convert existing class components to hooks unless that is the explicit task.

### Naming
- Component files: PascalCase with `.jsx` extension (`Card.jsx`, `news.jsx`).
- Non-component utilities: camelCase with `.js`.
- State/prop/method names: camelCase (`newsItems`, `handleLike`, `searchQuery`).
- CSS class names: kebab-case (`cards-slider`, `active-slide-N`).

### Imports / exports
- All modules use ES6 `import`/`export`.
- Every component uses `export default`.
- Config is imported as JSON: `import config from "../config"`.
- CSS and SCSS are imported directly inside the component that owns them.

### Styling
- Edit `App.scss` for slider/layout styles; `App.css` is generated from it.
- Bootstrap 4 grid (`container`, `row`, `col-*`) is available globally via `index.js`.
- Font Awesome 4 icon classes (`fa fa-star`, `fa fa-star-o`) are available globally.

### Language
- All user-facing strings are in Swedish. Keep them in Swedish when editing or adding UI text.

### PropTypes
- `Card.jsx` validates its props with `prop-types`. Add PropTypes when creating new components.

## Data Flow

```
componentDidMount (News)
  └─> rss-parser fetches CORS_PROXY + config.apiEndpoint
      └─> newsItems stored in state, each augmented with .index and .liked
          └─> getFilteredArray() filters by searchQuery, reassigns indices
              └─> renders Card list; slider transform is driven by newsItem.index
```

## External Dependencies to Be Aware Of

| Concern | Detail |
|---|---|
| CORS proxy | Hardcoded to `https://cors-anywhere.herokuapp.com/` in `news.jsx:57`. This third-party service may require users to manually unblock the proxy in a browser. |
| RSS endpoint | `https://www.dn.se/rss` in `src/config.json`. Change here to switch sources. |
| SCSS compilation | Requires `node-sass` (v4). If the version causes build errors on newer Node, consider migrating to `sass`. |

## Known Limitations / Technical Debt

- The CORS proxy (`cors-anywhere`) is unreliable in production. A proper backend proxy or server-side rendering is the correct long-term fix.
- `getFilteredArray()` mutates the `newsItems` objects in state (it deletes and reassigns `index`). This is a side-effectful pattern; avoid deepening it.
- `handleLike` contains a stale `console.log` left from development.
- `news.jsx` imports both `App.css` and `App.scss`; in practice only one should be imported (prefer `App.scss`).
- No i18n library is used; Swedish strings are hardcoded throughout.
- No CI/CD pipeline is configured.
- The PWA manifest still shows "Create React App Sample" as the app name.

## What Not to Do

- Do not run `npm run eject` — it is irreversible and not needed for any current work.
- Do not edit `App.css` directly; regenerate it by editing `App.scss`.
- Do not add Redux, Context API, or a global state library without an explicit requirement — the app intentionally uses local component state.
- Do not add dependencies without checking whether an existing library already covers the need (lodash and react-bootstrap are already available).
