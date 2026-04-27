> **Stacks on top of the deploy PR** for #3. Please merge the deploy PR (#<deploy-PR-number>) first; this PR's diff against `main` will then auto-shrink to just the PWA changes once the deploy commits land in upstream.

## Summary

Adds PWA polish so OpenQuacks behaves and feels like a native app when installed to the home screen, plus a guided in-app install flow that explains *Add to Home Screen* on iOS (where Apple doesn't fire `beforeinstallprompt`).

The work follows the standard PWA "decompose and distill" playbook: lock down the viewport, take control of touch events, fix the offline cache, and offer a clear install affordance. One squashed commit, ~16 files touched (+~310 lines), no source-code changes outside the PWA scope.

## What's added

**Standalone & home-screen identity**
- `manifest.json` rewrite: `scope`, `id`, `lang`, `dir`, `categories`, `display_override: ["fullscreen","standalone"]`, maskable icon entries, `start_url: "./?source=pwa"` so installed launches are distinguishable in analytics
- HTML head: `theme-color`, `color-scheme: dark`, `format-detection: telephone=no`, `mobile-web-app-capable`, additional `apple-touch-icon` sizes, `apple-mobile-web-app-title: "Quacks"`

**Notch & viewport**
- `viewport-fit=cover` on the viewport meta
- Safe-area-inset CSS variables (`--sat`, `--sar`, `--sab`, `--sal`) threaded into `body`, `.modal-overlay`, and `#achievement-toast` padding, so content sits below notches and home indicators on iOS
- `100vh → 100dvh` fallback chain everywhere — content no longer jumps when the iOS URL bar collapses

**Touch / gesture control**
- `body { touch-action: pan-y }` blocks pinch-zoom and double-tap-zoom at the CSS level, in BOTH regular Safari and standalone (the viewport `user-scalable=no` hack only works in standalone — Apple ignores it in regular Safari for accessibility)
- `canvas { touch-action: none }` keeps the game's custom touch handlers in control of chip drawing
- `overscroll-behavior: none` on `html, body` kills pull-to-refresh and rubber-band overscroll
- `-webkit-touch-callout: none` and `user-select: none` scoped to `body` and media elements (canvas/video/img) only — NOT on `*`, which on iOS Safari can interfere with synthesized click events on plain divs and break the tutorial highlight taps

**Mobile responsive layout**
- Body locked at `100dvh; overflow: hidden`; `#game-container` scrolls internally with `overscroll-behavior: contain` rather than the whole page scrolling
- (Note: a load-bearing comment in this rule explicitly warns *not* to add `-webkit-overflow-scrolling: touch` here — it creates a stacking context on iOS Safari that traps `#ui-sack`'s `z-index` inside `#game-container` and causes the tutorial highlight to end up below the overlay. Found this the hard way.)

**Install UX**
- New `#install-section` inside the existing Settings modal:
  - Android / Chromium: "Install Now" button, hooked up to the captured `beforeinstallprompt` event, auto-hides on `appinstalled`
  - iOS: "How to Install on iPhone" button → opens new `#ios-install-modal` with a 3-step *Add to Home Screen* walkthrough
  - Whole section auto-hides if the page is already running standalone

**Offline / Service Worker**
- `sw.js` cache name `v4 → v9` so prior caches purge on next visit
- Adds `lang/ja.json` and `lang/pt.json` to the cache list — these were silently broken offline before

**i18n**
- 10 new install-related keys, translated across all 12 lang files: en, es, de, he, ar, fa, pl, uk, el, ga, pt, ja

**Docs**
- `README.md`: replaces the old "save as Web App" iPhone instruction with a pointer to the new in-app Settings → "How to Install on iPhone" flow

## Tested end-to-end

Live preview on my fork (per-branch preview from the deploy PR's workflow):
**https://eyalfrish.github.io/OpenQuacks/preview/feature-pwa-polish/**

- iPhone Safari (Private mode): tutorial highlight + tap on the SACK works; pinch-zoom blocked; notch handled; install instructions modal renders correctly
- Pre-PWA prod URL on the same iPhone (control): also works (regression check)
- Desktop Chrome DevTools → Application → Manifest renders all fields including maskable icons with no warnings; Lighthouse PWA audit: green
- Offline test: Japanese (ja) and Portuguese (pt) language switches work offline (they didn't before)

## Future work (deliberately out of scope of this PR)

Three improvements I considered and chose not to bundle here:

- **Smart loading** — render panel scaffolding immediately while sprite sheets preload, so the game shows *something* during the unavoidable canvas-asset load instead of a full-screen spinner. `<link rel="preload" as="image">` for the largest sprites in `<head>` would also help. Polish, not a regression.
- **IndexedDB migration** — replace `setCookie` / `getCookie` for sound, character, language, ingredient-rules-set, `hasSeenHelp`, and achievement records with IndexedDB. Significant — touches every persistence call site and needs a one-time read-cookies-and-migrate path. Deserves its own dedicated PR, with care for existing users.
- **True maskable icon variant** — generate `icon-maskable-192.png` / `icon-maskable-512.png` with the duck centered in the inner ~80% safe zone. Currently the regular PNGs are reused with `purpose: "maskable"`, which may get edge-cropped on round Android launchers.

Happy to follow up with any of these in separate PRs if you'd like.

## Notes

- Zero source-code changes outside the PWA scope (no game-logic touches, no upstream-incompatible refactors)
- No new secrets; uses default `GITHUB_TOKEN` only via the deploy workflows from the other PR
- This branch is stacked on top of `feature/github-pages-deploy`; please merge the deploy PR first so this one's diff cleans up
