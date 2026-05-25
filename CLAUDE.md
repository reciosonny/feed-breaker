# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project scope

A Chrome extension (Manifest V3) that blocks TikTok's addictive feed pages. It injects a content script into `tiktok.com`, hides the feed DOM elements, mutes/pauses videos, and shows a "blocked" overlay. Users can disable blocking temporarily via the extension's options page.

## Commands

```bash
pnpm install          # Install dependencies

pnpm build            # Production build (minified, no sourcemaps)
pnpm build:dev        # Development build (sourcemaps, unminified)
pnpm build:watch      # Development build with watch mode

pnpm check            # Type-check with svelte-check + tsc
```

There are no tests. `pnpm dev` / `pnpm preview` are for standard Vite dev server use, but this extension runs as a Chrome content script — load it via `chrome://extensions` in developer mode, pointing to `browser-extensions/chrome/`.

## Build Output

Vite builds into `browser-extensions/chrome/dist/`. The manifest at `browser-extensions/chrome/manifest.json` references `./dist/index.js` and `./dist/index.css` as content scripts. After build, `options.html` is moved from deep inside `dist/` to `browser-extensions/chrome/options.html` by a custom Vite plugin (`MoveHtmlPlugin` in `vite.config.ts`).

## Coding styles

- Use `arrow function` declaration whenever you create new functions inside the app like so:
   const test = () => {

   }
- Use "named exports" all the time
- DO NOT use `var` when declaring variables. Always use `let` or `const` whenever possible
- Use ES6+ syntax and features at all times (destructuring, spread operator, optional chaining, template literals, etc.)
- Prefer spread syntax over `Object.assign` whenever possible:

   // preferred
   const merged = { ...obj1, ...obj2 }

   // avoid
   const merged = Object.assign({}, obj1, obj2)

## Architecture

### Entry Points

- **`src/main.ts`** — Content script entry. Detects whether running as a content script on tiktok.com or as the extension options page (`chrome-extension://` protocol), and mounts the appropriate Svelte app.
- **`src/browser-widgets/options/main.ts`** — Options page entry (mounted from `src/main.ts` when on `chrome-extension://`).

### Core Content Script Flow

1. `main.ts` reads `DISABLE_DURATION` from Chrome storage. If blocking is currently disabled (user paused it), sets `settingStore.disableUIBlock = true` and skips injection.
2. Otherwise, calls `injectReelsEradicator()` (`src/lib/injectReelsEradicator.ts`), which:
   - Starts a `MutationObserver` on `document.body` to detect DOM changes and URL navigation (TikTok is a SPA).
   - On each mutation, checks if current path is in `PATHS_TO_WATCH`.
   - Hides the TikTok feed container (`#main-content-homepage_hot`, `#main-content-friends_page`, `#main-content-explore_page`, `#main-content-homepage_follow`) via `display: none`.
   - Pauses and mutes any `<video>` elements.
   - Updates `mainDisplayStore.isVisible` to show/hide the blocker overlay.
3. `App.svelte` mounts the blocker overlay UI. It's only visible when `mainDisplayStore.store.isVisible` is true.

### State Management

Uses Svelte 5 runes (`$state`) with hand-rolled stores (not Svelte stores):

- **`mainDisplayStore`** (`src/store/mainDisplayStore.svelte.ts`) — Controls whether the blocking overlay is shown (`isVisible`).
- **`settingStore`** (`src/store/settings.svelte.ts`) — Tracks whether the UI blocker is currently disabled (`disableUIBlock`).

### Chrome Storage Keys

All persistence goes through `src/lib/chromeStorage.ts` using `chrome.storage.local`:

| Key | Type | Purpose |
|-----|------|--------|
| `TIMESPENT_STORE` | `number` | Cumulative seconds saved (not scrolling) |
| `DISABLE_DURATION` | `number \| 'forever'` | Unix timestamp until which blocking is paused |
| `DISABLE_REEL_OPTIONS` | `boolean` | Whether the "disable blocking" toggle is on |
| `DISABLE_DURATION_SELECTION` | `DisableReelOptions` | Which duration option was selected |

### Paths That Trigger Blocking

Defined in `src/lib/utils.ts` as `PATHS_TO_WATCH`:
```ts
["/en", "/foryou", "/", "/friends", "/explore", "/following"]
```
Keep this in sync with any UI that lists these paths.

### URL Alias

`$lib` resolves to `src/lib/` (configured in `vite.config.ts` and `tsconfig.json`).

### Options Page

`src/browser-widgets/options/App.svelte` — Lets users toggle "Disable For You blocking" and select a disable duration (10 min to forever). Uses `svelte-french-toast` for toast notifications and `dayjs` for computing expiry timestamps.
