# Decision Ledger — Volume-Booster

Durable record of the significant decisions made in this repository and the reasoning behind them.

- **Confirmed** decisions are human-reviewed and binding. This section is maintained by the repository owner; the automated decision-ledger pass never edits it.
- **Inferred** decisions are hypotheses proposed automatically from the code, commit history, and any agent instructions (CLAUDE.md / AGENTS.md). They are **not binding** until the owner moves them into Confirmed.

## Confirmed

_None yet. Merge a proposal from Inferred to confirm it._

## Inferred (proposed — awaiting confirmation)

> Every item below is a hypothesis generated automatically on 2026-06-21. Where the rationale could not be recovered from the available evidence it is marked "rationale unknown — please supply".

### [hypothesis] Build the product as a browser extension (Chrome MV3 + Safari Web Extension)
- **Decision:** Ship Volume Booster as a browser extension. The Chrome build uses Manifest V3 (`"manifest_version": 3`) with a service-worker background script; the same extension is also distributed as a Safari Web Extension for macOS and iOS.
- **Rationale (hypothesis):** A browser extension is the delivery mechanism that can inject into arbitrary web pages to amplify their media; MV3 is the only manifest version Chrome currently accepts for new extensions, and the Safari Web Extension model lets the same WebExtensions codebase reach Apple platforms.
- **Evidence:** `Chrome_Extension.zip` → `Chrome/manifest.json` (`"manifest_version": 3`, `"background": { "service_worker": "background.js" }`); `Privacy Policy` ("available as a Chrome extension and as a Safari Web Extension for macOS and iOS"); README.md.
- **First observed:** 535a241 (Add files via upload, 2026-04-06)

### [hypothesis] Amplify media using the Web Audio API gain node, capped at 600%
- **Decision:** Boost audio/video by routing each media element through a Web Audio `MediaElementSourceNode` → `GainNode` → destination graph, and clamp the gain to a 100%–600% range.
- **Rationale (hypothesis):** Standard HTML media volume is capped at 100%, so a `GainNode` is the supported way to exceed it; the 600% upper bound is the product's advertised maximum (README and manifest description).
- **Evidence:** `Chrome/content.js` (`getAudioContext`, `createMediaElementSource`, `createGain`, and `setVolume` → `Math.max(100, Math.min(600, ...))`); `Privacy Policy` ("uses the Web Audio API to amplify video and audio"); README.md ("up to 600%"); `Chrome/manifest.json` description.
- **First observed:** 535a241 (Add files via upload, 2026-04-06)

### [hypothesis] Privacy-first design: no data collection, no network, no persistence
- **Decision:** Collect nothing, make no external network requests, use no analytics/ads/trackers, and persist nothing — volume settings live in memory only and reset on page close/refresh.
- **Rationale (hypothesis):** Stated as a core product/privacy commitment; keeping all processing local and in-memory removes data-handling obligations and supports the "no data leaves your device" promise.
- **Evidence:** `Privacy Policy` (entire document — "No data leaves your device", "Nothing is saved to disk, not even your volume settings", "makes no external network requests"); `Chrome/content.js` (state held in `currentGain` / `WeakMap` / `WeakSet`, no storage API use).
- **First observed:** 659efbb (Privacy Policy, 2026-04-06)

### [hypothesis] Request only `<all_urls>` host access; no other extension permissions
- **Decision:** Declare an empty `"permissions"` array and request only `"host_permissions": ["<all_urls>"]` (mirrored by content-script `"matches": ["<all_urls>"]`).
- **Rationale (hypothesis):** Boosting media on "any webpage" requires injecting a content script across all sites, but requesting no additional API permissions keeps the permission surface minimal — consistent with the privacy-first stance.
- **Evidence:** `Chrome/manifest.json` (`"permissions": []`, `"host_permissions": ["<all_urls>"]`, content_scripts `"matches": ["<all_urls>"]`); `Privacy Policy` (Permissions section).
- **First observed:** 535a241 (Add files via upload, 2026-04-06)

### [hypothesis] Inject content script early and into every frame, with dynamic-media tracking
- **Decision:** Run the content script at `document_start` across `all_frames`, and use a `MutationObserver` plus `play`/`loadedmetadata` listeners to detect and boost media added after load, including media inside shadow DOM.
- **Rationale (hypothesis):** rationale unknown — please supply
- **Evidence:** `Chrome/manifest.json` (`"run_at": "document_start"`, `"all_frames": true`); `Chrome/content.js` (`installObserver` / `MutationObserver`, `collectMedia` recursing into `el.shadowRoot`, listeners on `play` and `loadedmetadata`).
- **First observed:** 535a241 (Add files via upload, 2026-04-06)

### [hypothesis] Popup ↔ content messaging brokered through the background service worker
- **Decision:** The popup sends `ACTIVE_TAB_REQUEST` messages to the background service worker, which resolves the active tab (`chrome.tabs.query`) and relays `GET_STATUS` / `SET_VOLUME` messages to that tab's content script, returning structured `{ ok, reason }` results on failure.
- **Rationale (hypothesis):** rationale unknown — please supply
- **Evidence:** `Chrome/background.js` (`MESSAGE_ACTIVE_TAB`, `getActiveTabId`, `sendToActiveTab`, relays `GET_STATUS`/`SET_VOLUME`); `Chrome/content.js` and `Chrome/popup.js` (matching `GET_STATUS` / `SET_VOLUME` message constants).
- **First observed:** 535a241 (Add files via upload, 2026-04-06)

---
*Decision-ledger automated pass. Operation: Bootstrap. Last reflection: commit `e2458ef` (2026-06-21). Decisions above are AI-inferred hypotheses; nothing is binding until merged into Confirmed.*
