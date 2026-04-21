# Blessed International Ministries — Homepage

Static marketing site for Blessed International Ministries. Two pages: a main scrolling
homepage (`index.html`) and a standalone authors page (`about.html`). No build step,
no framework — pure HTML/CSS/JS with Three.js and GSAP loaded from local `node_modules`.

---

## File Structure

```
blessed-homepage/
├── index.html          # All sections, all JS — single file
├── about.html          # Standalone About the Authors page
├── book-covers/
│   ├── README.txt
│   ├── clean-code.jpg          # Book 1 cover (Daily Supernatural)
│   ├── clean-architecture.jpg  # Book 2 cover (Companion Activation Guide)
│   └── ddia.jpg                # Book 3 cover (Guarding the Fire) — may be missing, fallback exists
├── authors/            # Drop author photos here (referenced in HTML comments)
├── node_modules/
│   ├── three/build/three.min.js       # Three.js r128
│   ├── gsap/dist/gsap.min.js          # GSAP 3.12
│   └── gsap/dist/ScrollTrigger.min.js
├── package.json
└── README.md
```

---

## Tech Stack

| Library | Version | Why local, not CDN |
|---|---|---|
| Three.js | r128 | CDN was blocked in dev sandbox; local install was the fix |
| GSAP | 3.12 | Same reason — installed alongside Three.js |
| ScrollTrigger | 3.12 | GSAP plugin — ships in the same `gsap` npm package |

> **For GitHub Pages deployment**: swap the 3 `<script>` tags in both HTML files to CDN
> URLs — `node_modules/` is not committed to Git:
> ```html
> <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
> <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
> <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/ScrollTrigger.min.js"></script>
> ```

---

## Page Sections (index.html, in order)

### 1. Hero (`#hero`)
Canvas particle field (`#hero-canvas`) behind a centered title block. GSAP timeline
animates eyebrow → title → subtitle → CTA on load. `gsap.set` establishes the initial
`y: 40` offset **before** the timeline that tweens to `y: 0`.

### 2. 3D Book Carousel (`#book-section`)
Three.js scene in `#book-canvas`. Books sit on an upright circle (radius `R = 5.2`),
spaced evenly by angle. Scroll via `ScrollTrigger` scrub drives `targetY`; a secondary
`lerp` (`+= delta * 0.052`) inside `requestAnimationFrame` removes micro-jitter.

**Book covers**: `ExtrudeGeometry` UVs are in shape-coordinate space, not 0–1, so
applying a photo texture to the extrusion material causes distortion. Fix: on image
load, a `PlaneGeometry(BW, BH)` overlay with `MeshBasicMaterial({ map: tex })` is
placed just in front of the extrusion (`face.position.z + 0.06`) and the canvas
texture material is hidden. If the image file is missing, the canvas-generated
gradient cover shows instead — this is the intentional fallback.

**Click-to-URL**: `THREE.Raycaster` on `#book-sticky` click. Walks up the hit object's
parent chain to find the book group in `carousel.children`, maps index to `BOOKS[idx].url`.

**Text panel**: `#book-headline` and `#book-body` update when the front-facing book
changes. Change is detected by comparing `frontIdx` to `lastFront` each frame.

### 3. Orbit / Foundation (`#orbit-section`)
Three.js scene in `#orbit-canvas`. Word pills orbit in two rings at different tilts
and speeds. Pills are `PlaneGeometry` meshes with canvas-drawn textures. Each frame,
`mesh.lookAt(camera.position)` keeps pills facing the camera (billboard effect).
No scroll interaction — rings rotate on a pure time clock.

### 4. Metrics (`#metrics`)
Four stat cards. Each `.metric-number` has a `data-target` attribute. `ScrollTrigger`
fires once on enter and uses `gsap.to({}, { onUpdate })` to count up. Float targets
(e.g. `100.00`) use `.toFixed(2)`; integer targets use `.toLocaleString()`.

### 5. Declarations / Principles (`#principles`)
Six declaration items. Each gets a `ScrollTrigger` that fires once and runs a GSAP
timeline: border line sweeps left→right, number shoots in from left with overshoot,
name slides up with skew snap, description fades up.

### 6. About the Authors (`#about-section`)
Static HTML — two author cards. Photo boxes show initials as placeholder; replace with
`<img src="authors/authorN.jpg" alt="...">` when photos are available.

### 7. Contact Form (`#contact`)
Submits to Google Sheets via Google Apps Script. Uses `fetch` with `mode: 'no-cors'`
and `URLSearchParams` body (`application/x-www-form-urlencoded` — the only content
type that reliably passes through `no-cors`). On the Apps Script side, read fields via
`e.parameter.name`, `e.parameter.email`, etc. — **not** `e.postData.contents`.

---

## Updating Content

### Books (title, author, colors, links)
Find the `BOOKS` array near the top of the `bookScene` IIFE (~line 950 in index.html):

```javascript
const BOOKS = [
  {
    title: 'Daily\nSupernatural',   // \n = line break on canvas texture
    author: 'Ryan C. Lee',
    tags:  ['...'],                 // shown at bottom of canvas cover
    grad:  ['#hex1', '#hex2'],      // gradient background when no image
    accent: '#hex', accentHex: 0xhex,
    spine: 0xhex,
    imageSrc: 'book-covers/clean-code.jpg',  // relative path; omit or leave blank for canvas fallback
    url: '',                        // paste Amazon/store link here; empty = click does nothing
  },
  ...
];
```

Also update `bookTexts` (just below `BOOKS`) — the headline and body shown in the
bottom-left metadata panel when each book is front-facing.

### Orbit Words
Find the `items` array inside the `orbitScene` IIFE:
```javascript
const items = [
  { label: 'Jesus Christ', color: 0x2997ff },
  ...
];
```
Add, remove, or rename entries freely. Colors are Three.js hex integers.

### Metrics
Edit the `data-target` attributes and label text directly in the HTML `#metrics` section.
Floats (e.g. `100.00`) auto-format with two decimal places; integers use locale formatting.

### Declarations
Edit the `.principle-item` blocks in `#principles` — number, name, description.
The animation is class-driven so no JS changes needed.

### Authors
Edit the `.author-card-main` blocks in `#about-section` (index.html) and in `about.html`.
Both files have the same two author cards — keep them in sync.

---

## Theme

Two complete `:root` blocks live at the top of the `<style>` section in both HTML files.
The dark theme is active; the light theme is commented out directly above it.

To switch to light: comment out the dark `:root` block and uncomment the light one.
The comment markers are labeled clearly: `── DARK THEME (active) ──` / `── LIGHT THEME (inactive) ──`.

---

## Google Sheets Form Setup

1. Go to [script.google.com](https://script.google.com) → New project.
2. Paste this as the entire script:
   ```javascript
   function doPost(e) {
     const sheet = SpreadsheetApp.openById('YOUR_SPREADSHEET_ID').getActiveSheet();
     sheet.appendRow([
       e.parameter.date,
       e.parameter.name,
       e.parameter.email,
       e.parameter.age,
       e.parameter.heard,
     ]);
     return ContentService
       .createTextOutput(JSON.stringify({ result: 'ok' }))
       .setMimeType(ContentService.MimeType.JSON);
   }
   ```
3. Replace `YOUR_SPREADSHEET_ID` with the ID from your sheet's URL (`/d/XXXX/edit`).
4. Deploy → New deployment → Web app → Execute as: **Me** → Who has access: **Anyone**.
5. Copy the `/exec` URL.
6. In `index.html`, paste it into: `const SHEET_URL = '...'`

> **Important**: Every time you edit the Apps Script code, you must create a **New Version**
> under Manage Deployments — just saving does not update the live endpoint.

---

## Development

### Local server (required — `file://` blocks ES module features and canvas taint)
```bash
cd blessed-homepage
python3 -m http.server 8080
# open http://localhost:8080
```

### Install dependencies (if node_modules is missing)
```bash
npm install
```

---

## GitHub Pages Deployment

1. Switch the 3 `<script>` tags in **both** `index.html` and `about.html` from
   `node_modules/...` to the CDN URLs shown in the Tech Stack section above.
2. Create a `.gitignore`:
   ```
   node_modules/
   .DS_Store
   ```
3. Push to GitHub:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/USERNAME/REPO.git
   git push -u origin main
   ```
4. In the repo: **Settings → Pages → Branch: main → / (root) → Save**.
5. Live at `https://USERNAME.github.io/REPO/` within ~60 seconds.

---

## Validation Checklist

After any edit, open the browser console and verify zero errors before shipping.

### JS health
- [ ] No `TypeError` in console (most common cause: a removed HTML element whose JS
  reference was not cleaned up — see Gotchas below)
- [ ] Cursor dot follows mouse on `index.html` and `about.html`
- [ ] Click anywhere → fireworks burst

### Book carousel
- [ ] Three books visible on scroll through `#book-section`
- [ ] Bottom-left headline and body update as carousel rotates
- [ ] Top-right counter (`01 / 03`) updates correctly
- [ ] If book cover JPGs are present in `book-covers/`, they appear on the faces
  (no distortion — should be flush, not tiled/squished)

### Orbit
- [ ] Word pills visible and rotating in `#orbit-section`
- [ ] Pills face camera at all times (billboard)
- [ ] Two rings at different tilts visible

### Metrics
- [ ] Numbers count up from 0 when scrolling into `#metrics`

### Declarations
- [ ] Each row animates in (sweep line, number, name, description) on scroll

### Contact form
- [ ] Form submits without page reload
- [ ] Status message appears (`You're on the list.` or error)
- [ ] Row appears in Google Sheet within a few seconds

### Mobile (≤ 768px)
- [ ] Nav links hidden (logo still visible)
- [ ] Principles collapse to 2-column layout
- [ ] Form rows stack to single column
- [ ] Author cards stack vertically
- [ ] No horizontal overflow / scroll on any section

---

## Known Gotchas

### Curly-quote JS crash
macOS and some editors autocorrect straight apostrophes (`'`) and quotes (`"`) to
Unicode curly variants (U+2018/U+2019, U+201C/U+201D). If these appear as JS string
**delimiters** (not inside a template literal), the parser throws a `SyntaxError` and
the entire script dies silently — cursor, animations, fireworks, everything stops.

Diagnosis: `cat -v index.html | grep 'M-^@'` — any hit on a JS string line is the culprit.
Fix: replace the curly character with a straight apostrophe or backtick.

### Removing a feature — always remove JS too
If you delete an HTML element that JS references, add a null guard at the top of its
IIFE (`if (!canvas) return;`) **or** delete the JS block entirely. Forgetting this causes
a `TypeError: Cannot read properties of null` that kills the entire `<script>` tag —
all features on the page go dead, not just the removed one.

The `supernovaScene` IIFE was a historical example of this: the canvas was removed but
the 120-line JS remained for a long time, guarded only by an early return.

### Google Sheets form — no-cors content type
`fetch` with `mode: 'no-cors'` only allows `application/x-www-form-urlencoded`,
`multipart/form-data`, and `text/plain` as Content-Types. Using `application/json`
causes the browser to silently drop the custom header, and Apps Script receives an
empty body. The form uses `URLSearchParams` (which sets `application/x-www-form-urlencoded`)
and reads `e.parameter.*` on the Apps Script side — not `JSON.parse(e.postData.contents)`.

### ExtrudeGeometry UV distortion
`ExtrudeGeometry` UVs use raw shape coordinates, not the normalized 0–1 range that
image textures expect. Applying a photo directly to the extrusion material tiles and
distorts it. Solution: on image load, place a `PlaneGeometry` overlay (`MeshBasicMaterial`)
just in front of the extrusion and hide the underlying canvas material.

### gsap.set must precede the timeline
`gsap.set([...], { y: 40 })` must appear **before** `gsap.timeline(...).to(...)` calls
that animate from that offset. GSAP timelines are not truly lazy — the initial property
capture happens at construction time.
