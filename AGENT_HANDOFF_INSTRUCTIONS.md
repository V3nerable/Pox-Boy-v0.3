# POX-BOY 3026 OS - Agent Handoff Document

**ATTENTION AI AGENT:** 
You are taking over development of a highly customized, offline-first Progressive Web App (PWA) designed for a live-action roleplay (LARP) event called **Pox Eclipse** (A Fallout / Mad Max crossover).

## 1. The Architecture
*   **No Backend:** This app is 100% serverless. Do NOT attempt to add Node.js, Python, or SQL databases. 
*   **Data Persistence:** All user data (items, quests, factions, stats) is permanently saved to the device using `localStorage`.
*   **File Structure:** The app was previously a single 3,000-line HTML file but has been modularized. You must maintain this structure:
    *   `index.html` (The core UI layout, modals, and templates)
    *   `styles.css` (All styling. Note the heavy use of CSS variables for the CRT phosphor glow).
    *   `app.js` (All logic, caching, saving, and hardware API calls).
    *   `sw.js` (The Service Worker. **CRITICAL:** You must increment the `CACHE_NAME` version in this file every time you make an update, or users' phones will not download the new code).
    *   `manifest.json` (Forces immersive fullscreen on mobile).

## 2. Core Mechanics (Already Implemented)
*   **V.T.A.R.S. Registration:** A robust 6-step onboarding loop. Users pick a Name, Origin, and Trait, take a 5-question G.O.A.T. exam, manually allocate S.P.E.C.I.A.L. points, and pick a Perk. The math engine converts this into their starting HP, Skills, Class Title, and Faction Reputations.
*   **Virtual Keyboard:** To prevent native mobile keyboards from breaking the immersion, all `<input>` fields are set to `readonly`. Clicking them triggers a custom HTML/CSS glowing green virtual keyboard (`vk-modal`).
*   **P2P QR Engine:** The app uses `qrcodejs` and `html5-qrcode`. Players can trade items or copy quests by generating JSON payloads in a QR code and scanning each other's screens.
*   **Geofencing Map:** Uses `Leaflet.js`. The user's GPS is tracked. If they walk within 30m of a hidden map coordinate, a location is "Discovered".
*   **Live Radar (Firebase):** An optional Firebase integration allows users who opt-in to broadcast their GPS coordinates to a shared map.

## 3. The Development Backlog
Please review `IDEAS_LOG.md` in this directory. The user intends to implement the following features next:
*   **The Wastelander Rolodex:** A system where users scan each other's "Profile QR" to save them as a Contact.
*   **Bounty Hunting PVP:** Users can accept bounties against specific players. They must physically hunt them down and scan a "Surrender QR" on the target's phone to claim the reputation reward.
*   **Dynamic Traits:** Allowing "Overseers" (Admins with the `1234` PIN) to grant permanent digital badges/traits to players in the field.

## 4. UI Rules
*   **No Native Popups:** Do NOT use `alert()` or `confirm()`. You must route all warnings through `showNotification(msg)` or `showCustomPrompt(text, buttons)`.
*   **Landscape Lock:** The app is strictly designed for landscape orientation. Do not build portrait-mode UIs.

**Proceed by reading `app.js` and `IDEAS_LOG.md`, then ask the user which feature they want to tackle first.**

## 5. Changelog
*   **v0.22:** Fullscreen rewritten (API-only truth + intent tracking + RESUME state). Fixed missing `cam-placeholder`/`cam-start-btn` IDs (camera crash), duplicate `custom-prompt-modal`, `event.target` tab fragility, Android `new Notification()` crash (now via ServiceWorker). Manifest orientation → landscape.
*   **v0.23:** Fullscreen engine hardened again: FUSED truth (`fullscreenElement` AND browser-chrome height check) kills the permission-popup "wedge" where the API stays non-null after visual exit; `exitFullscreen`/`requestFullscreen` never naked-awaited (raced vs timeout — wedged promises hang); unstick = phantom-exit then re-request inside the same tap; `resize`/`visibilitychange` listeners added. `sw.js` is now NETWORK-FIRST for same-origin files + `controllerchange` auto-reload so devices can never run a stale mixed version again.
*   **v0.24:** `[EXIT FULL]` fixed for installed PWAs: manifest changed `display: fullscreen` -> `standalone` (the old value locked the WebAPK into OS-level immersion the DOM API could not override -- only killing the app escaped it). `exitFullscreen()` now tries all vendor exit variants raced with timeouts. CAM tab restructured to `.cam-split`: 4:3 sensor pane left + all controls right in landscape (mirrors MAP); `[ POWER ON SENSOR ]` no longer wraps mid-label in portrait. NOTE: iOS bakes display-mode at install -- users must delete the home-screen icon and re-add once after this deploy.
*   **v0.25:** Fullscreen AUTOPILOT: installed app = always-fullscreen by design (fsIntent defaults true outside browser tabs); permission-popout recovery now happens on the NEXT real tap anywhere (touchend/pointerdown capture listeners -- popup callbacks carry no user activation, which is why RESUME taps kept getting rejected). FS button hides itself in `display-mode: fullscreen` installs and on iOS home-screen apps (no API). Manifest back to `display: fullscreen` (OS re-immerses after system dialogs automatically). Camera flip rewritten: `enumerateDevices()` + explicit `deviceId` hard-restart (applyConstraints could 'succeed' without switching sensors); chosen camera persists across tab switches. Onboarding name field now survives the GPS opt-in toggle re-render (obNameCache).
*   **v0.26:** Numpad ban completed: `rep-amount`, `fac-rep`, `edit-fac-rep` are now readonly display chips driven by +/- steppers (+-5 per tap via `stepNumberInput()`, min 5 for the auth amount). Pre-boot calibration screen gets `max-height: 94vh; overflow-y: auto` plus landscape compaction rules (@media landscape max-height 600px) so it no longer clips on small phones. Theme tint engine: each entry in `themes[]` now carries `mapFx`/`camFx` filter strings; `cycleTheme()` pushes `--tile-filter` (`.leaflet-tile`) and live-updates `#cam-video`/`#cam-canvas`; `savePhoto()` bakes the current theme tint + watermark colour into saved images. WHITE theme = true grayscale phosphor.
*   **v0.27:** GLOBAL CRT ENCLOSURE: single fixed layer `#crt-global` (z-99999, pointer-events none) over the whole app = curved-edge shadow + vignette + bezel frame + glass sheen + scanlines + rolling refresh bar (theme-tinted via `--pip-rgb`); body lost its `class="crt"` (global layer supersedes it). SENSOR TINT via CSS vars: `:root` now defines `--pip-rgb`/`--cam-filter`/`--tile-filter`; `cycleTheme()` flips all three; `#cam-video`/`#cam-canvas`/Leaflet tiles AND `#qr-reader video` all read `--cam-filter`/`--tile-filter` (QR scanner UI also themed: links/buttons/selects pip-coloured). DATABANK: gallery is now a `.photo-tile-grid` of small 4:3 tiles; `openPhotoViewer(i)` opens `#photo-viewer-modal` (img max 78vh object-fit contain = whole image visible, zero scrolling) with DELETE/CLOSE; `deleteViewerPhoto()` routes through showCustomPrompt.
*   **v0.28:** Global CRT enclosure REMOVED per user request (element + CSS deleted; body regained its original class="crt" scanlines/flicker). Databank tiles fixed: dropped `loading="lazy"` (WebKit lazy-load quirk inside scroll panes = blank tiles) and replaced `aspect-ratio` with fixed 90px tile height (aspect-ratio unsupported on older iOS collapsed tiles to zero height). Theme tint vars (--pip-rgb/--cam-filter/--tile-filter) retained for marker/camera/QR/map tinting.
*   **v0.29:** Radar staleness cap -- Firebase listener skips drawing any other-player beacon whose timestamp is missing or older than 24h (LKL labels for fresh beacons unchanged). G.O.A.T. exam is now the SOLE S.P.E.C.I.A.L. allocator: each answer awards +2 to both listed stats (cap 10; ~27 total points vs old 28), Q5 commits userProfile.special and jumps straight to trait select; manual assignment screen (obStep 5), obPoints and adjustObSpecial() deleted; answer labels now read "(+2 STR, +2 AGI)" style.
*   **v0.30:** Boot-time HARDWARE AUTHORIZATION: new `[3] AUTHORIZE SATELLITE & OPTICS` step on the pre-boot calibration screen runs `primeDevicePermissions()` -- one-shot geolocation fix, a camera getUserMedia whose tracks are immediately stopped (permission primed, hardware released), and Notification.requestPermission (guarded for API-less devices). A live in-universe status block prints OK/DENIED/UNAVAILABLE per system; ends with restoreFullscreenIfDesired() so the Android popup chain can't strand the user out of fullscreen. Rationale: each native prompt can only fire once per origin, so burning all three during boot guarantees zero immersion-breaking popups mid-game. Commence Logon renumbered to [4].
