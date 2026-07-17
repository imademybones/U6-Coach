# U6 Coach App — Claude Code Brief

## What this project is

A single-file PWA (Progressive Web App) called **U6 Coach** — a match day tool for a U6 soccer coach. It lives at:

**GitHub repo:** `https://github.com/imademybones/U6-Coach`
**Live URL:** `https://imademybones.github.io/U6-Coach/`

The app is a single HTML file (`u6-rotation-card-vN.html`) with all CSS and JS inline. There are no build tools, no npm, no frameworks. Everything is vanilla HTML/CSS/JS.

---

## Repo structure

```
U6-Coach/
├── index.html                      ← redirect to latest version (update this on every release)
├── u6-rotation-card-v11.html       ← current latest version of the app
├── manifest.json                   ← PWA manifest (start_url points to index.html)
├── sw.js                           ← service worker (cache name + asset list must match current version)
├── offline.html                    ← offline fallback page
├── android-chrome-192x192.png      ← app icon
└── android-chrome-512x512.png      ← app icon
```

---

## How versioning works

Every change to the app creates a new version file. The version number increments (v11 → v12 etc).

**On every new version, three files must be updated:**

### 1. `u6-rotation-card-vN.html` (new file)
- Copy the previous version and make changes
- Update the `<title>` tag to the new version number

### 2. `index.html` (update one line)
Change the meta refresh URL:
```html
<meta http-equiv="refresh" content="0; url=./u6-rotation-card-vNEW.html">
```
Also update the fallback anchor href to match.

### 3. `sw.js` (update two things)
```js
const CACHE = 'u6-coach-vNEW';   // bump version number
const ASSETS = [
  './index.html',
  './u6-rotation-card-vNEW.html', // update filename
  './manifest.json',
  './offline.html',
  'https://fonts.googleapis.com/css2?family=Bebas+Neue&family=DM+Sans:wght@400;500;600;700&display=swap'
];
```

`manifest.json` does NOT need to change — it already points to `./index.html` permanently.

---

## App architecture

The entire app runs in one HTML file. Key sections:

### Three screens (CSS class `.screen`, active state `.screen.on`)
1. **`#screen-squad`** — Squad Manager: add/delete players, persisted to `localStorage` under key `u6v8-squad`
2. **`#screen-matchday`** — Match Day: pick players for pitch slots + bench, choose half length
3. **`#screen-card`** — Rotation Card: generated rotation blocks, timer, playing time tracker

### Key state variables
```js
let squad = [];            // [{id, name, tag}] — loaded from localStorage
let pitchSlots = [null,null,null,null];  // 4 on-field player ids
let benchIds = [];         // bench player ids
let liveBlocks = [];       // [{on:[...playerIndices], off:[...playerIndices]}] × 6 blocks
let blockFormations = [];  // formation key per block: '2-2'|'1-2-1'|'1-3'|'3-1'
let halfLengthMins = 15;   // set by picker on match day screen
let cardNames = [];        // player names in card order
let cardTags = [];         // player tags (not actively used for display)
let cardRoles = [];        // 'F'|'B'|null per player
let playingMins = [];      // playing time tracker values
let timerSecs;             // countdown timer
let currentBlockIdx = 0;  // which rotation block we're in
```

### Rotation templates (`const R`)
Fixed rotation patterns for 4–7 players, each with 6 blocks (3 per half). Each block has `on` and `off` arrays of player indices (not IDs). These are cloned into `liveBlocks` when the card is generated.

### Formations
```js
const FORMATIONS = {
  '2-2':   { label:'2–2',   rows:[2,2] },
  '1-2-1': { label:'1–2–1', rows:[1,2,1] },
  '1-3':   { label:'1–3',   rows:[1,3] },
  '3-1':   { label:'3–1',   rows:[3,1] },
};
const FORMATION_ROLES = {
  '2-2':   ['F','F','B','B'],
  '1-2-1': ['F','M','M','B'],
  '1-3':   ['F','B','B','B'],
  '3-1':   ['F','F','F','B'],
};
const ROLE_COLORS = { F:'#e63b2e', M:'#e6a817', B:'#1a6b3c' };
```

### Timer
- Counts down from `halfLengthMins × 60` seconds
- Triggers sub alert pop-up at each block boundary (at 1/3 and 2/3 of half elapsed)
- Sub alert (`#sub-alert`) shows who comes off and who comes on for the next block
- Timer bar colour: default blue → amber (first alert) → red (second alert) → dark (half time)

### Sub alert modal (`#sub-alert`)
Triggered by `showSubAlert(fromBlockIdx, toBlockIdx)` inside `timerTick()`.
Shows:
- Who's coming off (red column)
- Who's coming on (green column)
- Next block full lineup preview
- "Done ✓" button to dismiss

### Swap modal (`#swap-modal`)
Allows manual player swaps between on-field and bench within a specific block.
Triggered by tapping the pitch area or bench pills on any rotation block.

### Playing time tracker
- Calculates scheduled minutes: `blockMins × blocks_on_field` per player
- `blockMins = halfLengthMins / 3`
- Total game time = `halfLengthMins × 2`
- +/− buttons adjust in `blockMins` increments
- Bar colours: green = on schedule, amber = under, red = over

---

## Design system

**Fonts:** Bebas Neue (headings/numbers), DM Sans (body) — loaded from Google Fonts

**Colours:**
- Navy: `#1a3a6b` (primary, headers)
- Green: `#1a6b3c` (on-field, defence)
- Red: `#e63b2e` (attack, coming off)
- Amber: `#e6a817` (midfield, warning)
- Background: `#eef2f7`
- Card white: `#fff`

**Conventions:**
- All buttons wired via `addEventListener` in `DOMContentLoaded` — no inline `onclick` anywhere
- All DOM queries use `$(id)` helper: `function $(id){ return document.getElementById(id); }`
- Screen switching via `showScreen(id)` — adds/removes `.on` class
- Squad persisted to `localStorage` key `u6v8-squad` (keep this key consistent across versions)

---

## Deployment

Changes go live automatically ~60 seconds after pushing to the `main` branch on GitHub.

The app is accessed via:
- Direct URL: `https://imademybones.github.io/U6-Coach/u6-rotation-card-vN.html`
- Redirect: `https://imademybones.github.io/U6-Coach/` (via index.html)
- APK: wraps the GitHub Pages URL — works via index.html redirect, never needs rebuilding

---

## Common tasks

### Add a new feature
1. Read the current latest HTML file in full before making changes
2. Make changes inline — keep all CSS and JS in the single file
3. Follow the versioning steps above (new file, update index.html, update sw.js)
4. Push all three files to the repo

### Fix a bug
Same as above — always create a new version file rather than editing the existing one in place.

### Change the squad localStorage key
Do NOT change `u6v8-squad` — this will wipe existing users' squad data. If a migration is needed, read the old key, write to the new key, then delete the old one on load.

### Add a new formation
Add to `FORMATIONS`, `FORMATION_KEYS`, and `FORMATION_ROLES`. The `makeBlock()` function handles rendering automatically.

### Change half length options
Edit the buttons in `#half-length-picker` in the HTML and the `DOMContentLoaded` default selection.

---

## What NOT to do
- Do not split the app into multiple files — it must remain a single HTML file
- Do not add npm, build tools, or external JS libraries
- Do not change the `manifest.json` `start_url` — it permanently points to `./index.html`
- Do not use inline `onclick` attributes — use `addEventListener` only
- Do not change the localStorage key `u6v8-squad`
- Do not edit the existing version file — always create a new one
