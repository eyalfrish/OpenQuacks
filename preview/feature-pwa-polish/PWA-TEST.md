# PWA test checklist for `feature/pwa-polish`

## URLs

Always test in **Private Browsing** mode to avoid stale cache fights.

- **PWA build (this branch):** https://eyalfrish.github.io/OpenQuacks/preview/feature-pwa-polish/
- **Pre-PWA control (upstream behavior):** https://eyalfrish.github.io/OpenQuacks/

## Wait for the deploy

Every push to the branch triggers the preview workflow. Give it ~30 seconds before testing the latest version. The SW cache name (`open-quacks-v11` currently) bumps every time we ship a fix, so the new version is forced fresh.

---

## 1 · Just-fixed regressions (priority)

These broke during PWA development and got fixed. Confirm they're stable.

| # | Test | Expected |
|---|---|---|
| 1 | Pick a character on first load → tutorial starts → SACK at bottom-right of cauldron | Pulsing gold border around SACK; tooltip text "Tap the sack…" visible |
| 2 | Tap the highlighted SACK | Chip drawn into cauldron; tutorial advances to the next step |
| 3 | Double-tap anywhere on UI | No zoom |
| 4 | Pinch anywhere | No zoom |
| 5 | Drag your finger across the cauldron canvas | Page may scroll vertically; no zoom; no side effects (canvas has no touch handlers, it's a render surface) |
| 6 | Scroll the page during the tutorial | Body scrolls so the SACK is reachable even if it starts off-screen on a small phone |

---

## 2 · Standalone / Install — Android Chrome

| # | Test | Expected |
|---|---|---|
| 7 | Open URL in Chrome on Android. Wait ~2s, open Settings (⚙️) | "📲 Install as App" section visible, with **Install Now** button |
| 8 | Tap **Install Now** | Native Chrome install prompt appears; after install, app icon on home screen |
| 9 | Launch from the home-screen icon | No Chrome chrome (no URL bar). Address-bar area dark (matches `theme-color: #1a1a2e`) |
| 10 | Inside the installed app, open Settings | Install section is hidden (already installed) |

---

## 3 · Standalone / Install — iOS Safari

| # | Test | Expected |
|---|---|---|
| 11 | Open URL on iPhone Safari. Open Settings (⚙️) | "📲 Install as App" with **How to Install on iPhone** button |
| 12 | Tap that button | New modal with 3 numbered steps for *Add to Home Screen* |
| 13 | Follow the steps: Share → Add to Home Screen → Add | App icon "Quacks" appears on home screen |
| 14 | Launch from the home-screen icon | No Safari URL bar, no bottom bar. Status bar overlays the game (black-translucent) |
| 15 | On a notched iPhone, look at the top of the screen | Game content sits *below* the notch / dynamic island, not under it. Same for the home indicator at the bottom |
| 16 | Inside the installed app, open Settings | Install section is hidden |

---

## 4 · Visual / Layout

| # | Test | Expected |
|---|---|---|
| 17 | iOS Safari (regular, not installed): scroll up to make URL bar visible, then back down | Layout doesn't jump — uses `100dvh` |
| 18 | Open any modal (Settings, Help ?, Book 📖, Language) | Modal content padded away from notch and home indicator |
| 19 | Trigger an explosion (when you have ≥7 boomberries in the cauldron) | Achievement toast / explosion modal also padded for safe areas |
| 20 | Switch language to Hebrew (he), Arabic (ar), or Persian (fa) | RTL layout still respects safe areas; install instructions ordered list flows right-to-left |

---

## 5 · Touch / gesture

| # | Test | Expected |
|---|---|---|
| 21 | Long-press the cauldron canvas / a sprite token | Nothing happens (no "Save Image" menu) |
| 22 | Long-press a button or a text label | Nothing (no iOS text-select magnifying glass) |
| 23 | Pull-to-refresh (drag down from top of viewport) | Blocked, no Chrome/Safari refresh fires |
| 24 | Try to overscroll past top/bottom of content | No rubber-band bounce |

---

## 6 · Offline / Service Worker

| # | Test | Expected |
|---|---|---|
| 25 | Visit the URL once, then turn airplane mode on, reload | Game loads (SW serves cache) |
| 26 | While offline, ⚙️ → Language → 日本語 (Japanese) | UI switches to Japanese — works |
| 27 | While offline, ⚙️ → Language → Português | UI switches to Portuguese — works |
| 28 | While offline, switch through all other languages | All work |

---

## 7 · Desktop DevTools (any modern browser; Chrome/Edge easiest)

| # | Test | Expected |
|---|---|---|
| 29 | DevTools → Application → Manifest | All fields populate, no warnings; maskable icon preview shows |
| 30 | DevTools → Application → Service Workers | One active, shows `open-quacks-v11` |
| 31 | DevTools → Application → Cache Storage → `open-quacks-v11` | Lists all assets including `lang/ja.json` and `lang/pt.json` |
| 32 | Lighthouse → PWA audit | Pass / 100 |

---

## 8 · Pre-PWA vs With-PWA — these should DIFFER

Open both URLs side by side; these should behave differently.

| # | Test | Pre-PWA (`/`) | With-PWA (`/preview/feature-pwa-polish/`) |
|---|---|---|---|
| 33 | Pinch on UI | Zooms | No zoom |
| 34 | Double-tap | Zooms | No zoom |
| 35 | Pull-to-refresh | Triggers Safari/Chrome refresh | Doesn't |
| 36 | iOS notch overlap | Content under notch | Content below notch |
| 37 | Settings modal contents | Sound + Rules only | Sound + Rules + 📲 Install section |
| 38 | Status bar in installed iOS | Default | Black-translucent, content extends behind |
| 39 | Switch to Japanese offline | May fail | Works |

---

## 9 · Pre-PWA vs With-PWA — these should be SAME (regression check)

| # | Test | Both should |
|---|---|---|
| 40 | Brewing, exploding, selling potions | Identical game logic |
| 41 | Tutorial flow on first visit | Walks through all 6 steps |
| 42 | All 12 languages render | Work the same |
| 43 | Achievements unlock | Same triggers |
| 44 | All character powers | Same |

---

## What to report when something fails

- Which test number(s).
- Which device + browser + iOS/Android version.
- Whether the SAME test passes on the pre-PWA control URL.

If all of 1–6 pass on iPhone Safari, the original tutorial regression is dead and the rest is "feature work to verify". Tests 7–32 are mobile-platform-specific verification of the actual PWA feature set. 33–44 are A/B against upstream.
