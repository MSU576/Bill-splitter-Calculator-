# SplitTab — Tip Calculator

A single-screen tip calculator and bill splitter. Live updates as you type, no "Calculate" button.

## How to Run

This is a **zero-dependency, single-file app** — no build step, no npm install.

### Option 1 — Just open the file
```
open index.html
```
Double-click `index.html` in your file manager, or drag it into any modern browser (Chrome, Firefox, Edge, Safari).

### Option 2 — Local dev server (recommended for consistent behaviour)
```bash
# Python 3 (usually pre-installed)
python3 -m http.server 3000
# then visit http://localhost:3000

# OR with Node.js
npx serve .
# then visit the printed URL
```

### Deployed URL
_Add your Vercel / Netlify / GitHub Pages URL here after deployment._

---

## Stack

Vanilla HTML + CSS + JavaScript. No frameworks, no bundler, no dependencies.  
One file: `index.html` (HTML structure, CSS custom-properties theming, and ES2020 JS — all inline for portability).

---

## Files

```
index.html      ← entire app
README.md       ← this file
ANSWERS.md      ← the 5 submission questions
```
