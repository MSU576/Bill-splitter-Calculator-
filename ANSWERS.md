# ANSWERS.md

---

## 1. How to Run

Zero dependencies — no install required.

**Quickest:**
```
open index.html
```
Drag `index.html` into any modern browser.

**With a local server (avoids any `file://` quirks):**
```bash
python3 -m http.server 3000
# visit http://localhost:3000
```
or
```bash
npx serve .
```

No Node, no npm, no bundler. It's one HTML file.

_Deployed URL: (add after deploying to Vercel/Netlify/GH Pages)_

---

## 2. Stack & Design Choices

**Why vanilla HTML/CSS/JS?**  
For a single-screen calculator with no persistence and no complex state tree, a framework adds ceremony without benefit. Zero build step means a reviewer can open the file instantly. The DOM surface area is small enough that manual element references (`getElementById`) are faster to read and debug than virtual DOM reconciliation.

**Visual / interaction decision 1 — Two-column layout with output pinned on the right**  
The inputs and results panel sit side-by-side at desktop widths (> 620 px), both visible simultaneously. The user sees the per-person number change in real time without any scrolling. This matters most during the moment of negotiation at a table — people are watching the result react. Putting the output below the inputs (the more common layout) forces the eye to travel and breaks the feedback loop. The breakpoint collapses to single-column at ≤ 620 px (covers the 360 px target), where the output panel naturally sits just below the fold — reachable with one short scroll after finishing input, which is fine because on mobile you've already set all three fields.

**Visual / interaction decision 2 — Ceiling rounding surfaced as a badge, not hidden**  
The per-person value is rounded up to the nearest paisa (Rs 0.01) using `Math.ceil(n * 100) / 100`. The rounding policy is stated directly on the output panel: *"Per-person amounts rounded up to nearest paisa (ceiling)"*. I made that label part of the output section — not a footnote in a modal — because the rounding affects the number the user reads. Hiding the policy in docs means users will distrust a Rs 42.67 result that "should" be Rs 42.65. Surfacing it right under the reset button ties cause and effect together.

---

## 3. Responsive & Accessibility

**Responsive behaviour**  
- **360 px phone:** single column, full-width inputs and output panel stacked vertically. Tip preset buttons wrap into two rows at very narrow widths (`flex-wrap: wrap`). `inputmode="decimal"` on bill/tip and `inputmode="numeric"` on people surfaces the right native keyboard. The output panel sits below the inputs — one short scroll to see results, which is acceptable given the interaction model (fill three fields, glance at result).  
- **1440 px laptop:** two-column side-by-side layout. Both panels are visible together; nothing requires scrolling. Max-width is capped at 860 px so the layout doesn't stretch into an unusable wide strip.

**Accessibility consideration handled — ARIA live region on output**  
The results `<section>` has `aria-live="polite"`, so screen readers announce updated values after every keystroke without interrupting mid-word. Individual error `<div>`s also have `role="alert"` and `aria-live="polite"` so validation messages are read immediately when they appear. Preset buttons use `aria-pressed` (true/false) so a screen reader communicates toggle state, not just a generic button label. Focus order follows the natural visual flow: bill → tip (presets skip-tabbed in group) → custom tip → people stepper (dec → input → inc) → reset.

**Accessibility consideration knowingly skipped — high-contrast mode / forced-colors**  
The design uses CSS custom properties (`--accent: #c8f55a` etc.) which work well in standard and dark contexts, but I haven't added a `@media (forced-colors: active)` override. In Windows High Contrast mode the accent yellow on dark background may collapse. Fixing this properly requires testing in that mode and adding `forced-color-adjust` rules — worth doing for a shipped product but out of scope for this assessment.

---

## 4. AI Usage

| Where | Tool | What I asked | What it gave me |
|---|---|---|---|
| Initial scaffold | Claude | "Give me a tip calculator layout in vanilla HTML with a two-column grid, dark theme, monospace output values, and inline validation" | A working two-column layout with CSS Grid and basic JS validation |
| Rounding logic | Claude | "How do I ceiling-round to 2 decimal places in JS without floating-point drift?" | `Math.ceil(n * 100) / 100` with explanation of why `toFixed` alone drifts |
| ARIA for preset buttons | Claude | "What ARIA attributes should toggle buttons in a group use?" | Suggested `role="group"` + `aria-pressed` on each button |
| Flash animation | Claude | "Animate a number value changing without re-rendering the element" | Gave a CSS keyframe on a class toggled with `offsetWidth` reflow trick |

**Change I made to AI output:**  
For the layout, the AI gave me a CSS Grid with `grid-template-columns: 1fr 1fr` and a fixed `max-width: 700px`. I changed the max-width to `860px` and added the `@media (max-width: 620px)` breakpoint to collapse to single column — the AI defaulted to `768px` as the breakpoint, but 768 px is a tablet landscape width where the two-column layout still works fine. Collapsing at 768 px would have given phone users a single narrow column even on a standard tablet, so I moved the breakpoint down to 620 px where the panels would actually start crowding each other.

---

## 5. Honest Gap

**What isn't polished enough: the stepper button UX on fast taps**  
Tapping the `+` / `−` buttons rapidly fires individual `click` events and recalculates on each one, which works correctly but doesn't feel snappy — there's a perceptible per-event DOM update cadence. With another day I'd add `pointerdown` long-press acceleration: hold the button and the value increments at increasing speed (100 ms → 50 ms → 20 ms intervals), the same pattern used by iOS number pickers. I'd implement it with a `setTimeout`/`setInterval` chain on `pointerdown`/`pointerup`, and debounce the final `calculate()` call to the end of the press rather than each tick.
