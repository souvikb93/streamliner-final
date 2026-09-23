# Framer embed asset — design system

Project-agnostic rules for every interactive HTML asset embedded in a Framer case-study page.
Colour, copy and data change per project. Everything below does not.

Companion file: `STREAMLINER_ASSET_GUIDELINES.md` holds the Dell/Streamliner-specific values
(palette, node ids, per-asset notes). Start a new project by copying that file, not this one.

---

## 1. Delivery contract

One asset = one self-contained `.html` file = one Framer `<Embed>` node.

    <Embed width="1fr" height="fit-content" maxWidth="1200px" type="html" radius="0px" zoom="1" html="…"/>

- **No build step, no external CSS/JS** except a pinned CDN (fonts, d3, topojson).
- **Transparent background** on `html, body` so the Framer section colour shows through.
- The pipeline is: local file → `s.replace('\n',' ')` → `html.escape(s, quote=True)` → the `html` attribute.
  Because newlines are collapsed, **never use `//` comments in the JS** — use `/* */` or none.
- Push with `updateXmlForNode`, always `zoomIntoView: false`.

### Height: Framer measures and writes back
Framer measures the embed document and writes the measured height into the node. Two consequences:

- `height="fit-content"` is the default — it lets each breakpoint measure itself.
- Pin the desktop height with a lock so the node never drifts:
  `@media (min-width:900px){body{min-height:<H>px}}`
- For a frame that must never change size at all (a card whose content swaps), pin the document too:
  `html,body{height:<H>px;overflow:hidden}` and release it in the tablet query.

**Measure at 1072px, not 1200px.** `maxWidth:1200` minus the section's 64px side padding is the real
render width on desktop. A lock derived from a 1200px measurement will clip.

### Size budget
| | max |
|---|---|
| width | 1200 (renders at 1072) |
| height | 700 |

Anything taller gets scrolled past. If content does not fit, do not shrink the type —
scroll it horizontally or stage it over time (§5).

---

## 2. Breakpoints

Framer M (810) and S (390) are **replicas** of L (1200): they inherit L's children with zero
overrides, so one `<Embed>` height is shared across all three widths, and per-breakpoint overrides
on replica children are not reachable through the MCP. All responsive behaviour therefore lives
**inside the HTML**.

Three tiers, always in this order:

    @media (min-width:900px){ body{min-height:<desktop lock>px} }   /* desktop lock   */
    @media (max-width:820px){ … }                                   /* tablet reflow  */
    @media (max-width:440px){ … }                                   /* phone density  */

Rules:
- Multi-column grids collapse to `1fr` at 820.
- Use `minmax(0,1fr)`, never `1fr`, and `min-width:0` on grid/flex children — otherwise long
  unbreakable strings force horizontal overflow.
- Wide diagrams (blueprints, journey maps, swimlanes) **scroll horizontally** below 760px
  (`overflow-x:auto` on the wrapper + `min-width:<Npx>` on the grid). Never squeeze them.
- Padding steps down: 24 → 16 → 12. Gaps 16 → 12 → 10.
- Verify at **1072 / 682 / 350** (the three real inner widths).

---

## 3. Type

Single family, variable weight, self-hosted from a pinned commit:

    @font-face{font-family:'<Family>';
     src:url(https://cdn.jsdelivr.net/gh/<owner>/<repo>@<commit-sha>/<path>.woff2) format('woff2');
     font-weight:300 700;font-style:normal;font-display:swap}

    font-family:'<Family>',-apple-system,sans-serif

Pin the commit — a bare `@main` is not reproducible. Never load Google Fonts in an embed.
SVG text needs the family named explicitly (`font-family="<Family>"`), it does not inherit.

### Scale
| role | size | weight | line-height |
|---|---|---|---|
| card title | 15–17px | 600 | 1.3 |
| body | 12.5–14px | 400 | 1.45–1.5 |
| supporting | 11.5–12px | 400 | 1.45 |
| label / eyebrow | 11–12px | 600 | 1.2 |
| chip, dense diagram cell | 9.5–11px | 500–700 | 1.3 |

- **Never `text-transform:uppercase`.** It reads as machine-generated. Sentence case throughout.
  Acronyms stay capitalised because they are acronyms (CRE, SODS, SFDC), never whole phrases.
- Tracking `.01–.02em` on labels; `-.01em` on headings; default on body.
- 9.5px is the floor, and only inside a dense diagram cell. Nothing below it.
- Numbers in columns, prices and timers: `font-variant-numeric:tabular-nums`.

---

## 4. Colour

Four families, defined once per project:

| token | job |
|---|---|
| **brand** + 4 tints | active state, links, the one accent |
| **positive / caution / negative** + tints | outcome states only |
| **ink / ink-2 / muted** | text, three levels |
| **line-strong / line / track / canvas** | borders, rules, fills |

Rules that do not change per project:
- **Text contrast ≥ 4.5:1.** A `#A1A1AA`-class grey is 2.5:1 on white — it is a *border* colour,
  never a text colour. The lightest usable body/label grey sits around `#696970` (5.4:1).
- Colour never carries meaning alone — pair it with an icon, a label or a position.
- One accent per asset. Outcome colours are reserved for outcomes.
- **Per-entity colour** (a system, a swimlane, an owner) is a separate axis from state colour.
  Give each entity a hue, keep saturation and lightness consistent across the set, and let state
  ride on opacity / ring / shadow instead of hue.

---

## 5. Motion

Every asset that can loop, loops. The loop teaches the diagram.

### The staged-chain pattern
Do not cross-fade between finished states. Build the relationship one beat at a time:

    node A → connector A→B → node B → connector B→C → node C …

- card / node beat **500–600ms**
- connector draw **600–700ms** (`stroke-dasharray` + `stroke-dashoffset` → 0, `pathLength="100"`)
- hold at end of a chain **2000–2600ms**, then advance
- lead-in before the first beat **~360ms**

### Connectors are never idle
Paths start `opacity:0; stroke-dasharray:100 100; stroke-dashoffset:100` and are revealed only
while their step is active. A resting web of grey lines makes the card look noisy and is the single
most common reason a diagram reads as cluttered.

### Connector geometry
- Route horizontals through a **shared channel** inside the gap between rows, offset by a constant
  step (`y = rowBottom + pad + (n-1)*16`) so parallel links stay equidistant and never overlap.
- Leave a node at its centre; enter the target at `width * n/(count+1)` so multiple incoming links
  land at even fractions (⅓, ⅔).
- Same column → plain vertical. Corner radius capped at 12px.
- Keep channels clear of section labels; give the label an opaque background so a line that must
  cross reads as passing behind it.

### Controls, every time
- **hover pauses**, `mouseleave` resumes
- **click holds** that state (15–20s) then releases
- `visibilitychange` stops the timer when the tab is hidden
- state is announced in an `aria-live="polite"` status line, not only by colour

### Reduced motion
Every asset with `@keyframes` needs:

    @media (prefers-reduced-motion:reduce){ … }

and the block must **leave content visible** — an entrance animation that starts at `opacity:0`
must be forced to `opacity:1; transform:none; animation:none`, or reduced-motion users see nothing.
Infinite decorative loops (pulses) stop entirely.

Other limits: 150–300ms for micro-interactions, transform/opacity only (never width/height/top/left),
ease-out in / ease-in out, one or two animated elements per view.

---

## 6. Layout patterns

### Fixed-frame card
A card whose content swaps must not resize. Exactly **one absorber** per card:

    .absorber{flex:1;min-height:0;overflow:hidden}

Everything else is reserved: fixed `height` + `overflow:hidden` on rows, `nowrap` + ellipsis +
`title` on single-line values, `-webkit-line-clamp` on multi-line ones. Pin `html,body` as well —
pinning only the inner wrapper still lets Framer measure a taller document.

### Swimlane / blueprint
- Fixed label rail on the left (86–104px), scrolling canvas on the right.
- Lanes are bands; the standard dividers (line of interaction, of visibility, of internal
  interaction) are dashed rules that span the full canvas width, labelled in the rail.
- Cells are uniform-width cards on a fixed column pitch. One colour per system, header strip
  in the system colour, white body.
- Annotations are **callouts for the active step only**, not a permanent litter of sticky notes.

### Stepper / progress
Long sequences (20+ steps) get a progress bar plus `Step n of N`, and the viewport auto-pans to
keep the active node centred. Offer a **Fit all** toggle that scales the canvas to the frame so the
whole map can be read at once.

---

## 7. Interaction & accessibility floor

Non-negotiable in every asset:

- Real `<button>` for anything clickable — never a `div` with a click handler.
- `:focus-visible{outline:2px solid <brand>;outline-offset:2px}` on every control.
- `aria-label` on any control whose text is not visible (progress dots, icon buttons).
- Tap target ≥ 24px, and ≥ 40px at the phone tier. A control that must stay visually small
  (a 4px progress bar) gets an invisible `::before` overlay that extends the hit area without
  changing layout:

      .dot{position:relative}
      .dot::before{content:"";position:absolute;left:0;right:0;top:-12px;bottom:-12px}

- Truncated text carries a `title` with the full string.
- `cursor:pointer` on clickables; visible hover *and* active states.
- Icons are inline SVG from one family, one stroke width. Never emoji.

---

## 8. Verification before pushing

Headless Chrome, every time — do not eyeball it in the canvas:

    chrome --headless=new --disable-gpu --hide-scrollbars --allow-file-access-from-files \
           --window-size=W,H --virtual-time-budget=T --dump-dom|--screenshot

Chrome floors the window width at 500px, so **test narrow widths through an iframe harness page**
that sizes an `<iframe>` to the exact width, not through `--window-size`.

Checklist:
1. `body.scrollHeight` at 1072 **equals the desktop lock**, and ≤ 700.
2. `body.scrollWidth - W === 0` at 1072, 682 and 350 — no horizontal overflow.
3. Animated assets: dump the DOM at several `--virtual-time-budget` values and assert the step
   sequence (how many connectors are drawn, which card carries the active class).
4. Screenshot mid-sequence and late-sequence; check nothing is clipped at the frame edge.
5. Truncation probe: cycle every data state and compare `scrollWidth`/`scrollHeight` against
   `clientWidth`/`clientHeight` for each text node that could overflow.
6. Re-run 1–2 after *any* edit, including a font or colour change.

---

## 9. Framer MCP gotchas

- `duplicateNode` on a **section** appends the copy to the end of the page root, and the MCP
  **cannot reorder root children**. To place a new block in a specific section, duplicate an
  **embed inside that section** instead — the copy lands as the last child of the same parent.
- `duplicateNode` returns `[object Promise]`. Recover the new node id by re-reading the parent.
- `getNodeXml` on a large embed exceeds the token limit; the result is written to a tool-results
  file — parse it with grep/python rather than reading it back.
- A push returns the full before/after diff. Expect it, and keep payloads as small as the asset allows.
- Writes fail with `Method: setAttributes, is unavailable while the project is read-only` when the
  project is open read-only or another editor holds the lock. Nothing is partially written; re-push
  once edit access is back.

---

## 10. New-asset checklist

- [ ] Copies the project guidelines file's palette and type tokens, adds no new ones
- [ ] Pinned variable font, no Google Fonts link, SVG text names the family
- [ ] Sentence case everywhere, no `text-transform:uppercase`
- [ ] Desktop lock measured at 1072 and ≤ 700 tall
- [ ] Tablet reflow at 820, phone density at 440, horizontal scroll for wide diagrams
- [ ] Loop with staged chain; connectors hidden until their step
- [ ] Hover pause, click hold, `visibilitychange`, `aria-live` status
- [ ] `prefers-reduced-motion` block that leaves content visible
- [ ] `:focus-visible` on every control, `aria-label` where text is not visible, ≥40px targets at phone
- [ ] All text ≥ 4.5:1 contrast
- [ ] Headless checks 1–6 pass
- [ ] Source saved to `kit/<name>.html` **and** `kit/<nodeId>.html`, mirrored to the delivery folder
- [ ] Asset documented in the project guidelines file

---

## Appendix A — verification gotchas found the hard way

**`requestAnimationFrame` does not advance under headless virtual time.**
`--virtual-time-budget` drives `setTimeout` fine, but a headless page is never "visible", so rAF is
throttled to a first frame and stops. A rAF-driven animation therefore screenshots frozen at step 1
no matter how large the budget, and `--disable-background-timer-throttling` does not fix it.

Ship rAF (correct for production — it pauses when the tab is hidden), but verify through a
throwaway copy with the driver swapped:

    probe = src.replace("requestAnimationFrame(frame)}", "setTimeout(()=>frame(performance.now()),16)}")

Screenshot the probe at several budgets, ship the original. Same code path, only the clock differs.

**A scaled-up active node needs frame clearance.**
If the active state scales a node (1.3–1.5×), every node within half a scaled-height of the canvas
edge will bleed past it, and the pulse ring adds more on top. Budget
`rowY + (h/2)*scale + maxRingGrowth` against the viewBox height before choosing row positions, and
raise the `z-index`/paint order of the active node so it lifts over its neighbours instead of
being clipped by them.

**A travelling token must hide when it arrives.**
A courier dot that stays parked on the node it just reached sits on top of that node's label. Show
it during the move phase only; the node's own active state carries the "I am working" signal.

**`duplicateNode` can detach the copy from its parent.**
A duplicate that read back as the last child of its Stack was later found absolutely positioned on
the canvas (`position="absolute"`, its own `top`/`left`, fixed px width) — outside the layout
entirely. Re-read the parent after duplicating *and* after the first content push, and check the
copy still carries the parent's layout attributes (`width="1fr"`, `maxWidth`) rather than
`position="absolute"`. Passing layout attributes in the push does not re-parent it.

---

## 11. Semantic colour groups (system diagrams)

A system or architecture diagram must never colour nodes individually. Group every
node by **what kind of thing it is**, give the group one colour, and reuse that
colour everywhere the node appears — border, label, icon, pulse ring, loader arc,
travelling token, connector on approach, and the readout swatch.

### Group model

| Group | Meaning | Token |
|---|---|---|
| `mod` | The product being designed (primary) | `#0672CB` |
| `crm` | System of record / case system | `#7C3AED` |
| `ppl` | People and channels (human actors) | `#57575C` |
| `data` | Data and catalogue services | `#B02071` |
| `log` | Logistics and physical movement | `#A15C00` |
| `fin` | Finance and payments | `#0B6E4F` |

All six sit in the same blue-violet-teal band as the rest of the kit; none is a
hue the brand does not already use. Every one clears 4.5:1 on white, so the same
token can carry 12px labels — no separate "text version" of a group colour.

Contrast as shipped (on `#FFFFFF`): mod 4.62, crm 6.72, ppl 5.63, data 6.14,
log 5.74, fin 6.19. Two candidates were rejected on this test and darkened:
`#2F8F7F` → `#A15C00`, `#0E7C9B` → `#B02071`.

### Fill vs outline

Hue alone does not create hierarchy — six equally-weighted colours read as a
rainbow. Weight does:

- **Primary group only** is filled (`fill=col`, white label). Everything the
  project actually built is solid.
- **Every other group** is white-filled with a group-coloured border:
  `stroke-opacity` `.45` idle → `1` working → `.3` done.

The result: the five modules are visibly the subject, the surrounding
enterprise systems are legible context.

### Implementation

```js
const PAL={mod:'#0672CB',crm:'#7C3AED',ppl:'#57575C',data:'#B02071',log:'#A15C00',fin:'#0B6E4F'};
const GRP={ce:'mod',vr:'mod',/* … */ arb:'log',om:'fin'};
const CO=id=>PAL[GRP[id]]||DEEP;
```

One lookup, never a per-node hex. Adding a node means adding one `GRP` entry —
if the colour is wrong the group is wrong, which is the useful failure.

### Rules

1. Six groups is the ceiling. Past six, hue stops carrying meaning.
2. Group colour is **state-independent**. State changes opacity, stroke width,
   shadow and scale — never hue. A node that changed colour on activation would
   read as a different system.
3. A legend is optional only while every group is named in the running readout.
   A static export of the same diagram needs a legend.
4. Idle borders never drop below `stroke-opacity:.45` — below that the group
   colour is no longer identifiable.

---

## 12. The toggle

Every asset that switches between two or more views uses the same control. Not a
segmented track, not a row of buttons — one white capsule with a navy pill that
slides behind the active label.

```css
.tabs{position:relative;flex:none;display:flex;gap:3px;background:#fff;
 border:1px solid #D9D9DE;padding:4px;border-radius:999px;
 box-shadow:0 1px 2px rgba(18,18,18,.04)}
.tabs button{font:inherit;position:relative;z-index:1;display:flex;align-items:center;
 height:34px;padding:0 14px;border:0;background:transparent;border-radius:999px;
 color:#454548;font-size:13px;font-weight:600;cursor:pointer;transition:color .25s}
.tabs button:hover{color:#121212}
.tabs button.on{color:#fff}
.tabs button:focus-visible{outline:2px solid #0672CB;outline-offset:3px}
.tabs .pill{position:absolute;top:4px;left:4px;height:34px;border-radius:999px;
 background:#06356E;z-index:0;
 transition:transform .42s cubic-bezier(.34,1.24,.5,1),width .42s cubic-bezier(.34,1.24,.5,1)}
@media (prefers-reduced-motion:reduce){.tabs .pill{transition:none}}
```

The pill is measured from the active button, never hard-coded:

```js
function movePill(){const b=document.querySelector('#tabs button.on');if(!b)return;
 _pill.style.width=b.offsetWidth+'px';_pill.style.height=b.offsetHeight+'px';
 _pill.style.transform='translateX('+(b.offsetLeft-4)+'px) translateY('+(b.offsetTop-4)+'px)'}
```

Reading `offsetHeight` as well as `offsetWidth` matters: mobile media queries raise
the button to a 44px touch target, and a hard-coded 34px pill would sit short of it.

Three things must re-run it or the pill drifts:

1. **after the click** — `setTimeout(movePill,0)` so it lands after the existing handler
2. **`document.fonts.ready`** — the webfont changes label widths on arrival
3. **a `ResizeObserver` on the container** — catches reflow the other two miss

### The status dot

The dot is **optional and rare**. Only an asset whose toggle selects a *running*
process earns one — in this set that is the to-be system diagram alone, where the
dot blinks green (`#3DDC84`, 1.3s, opacity 1 → .4 plus an expanding glow) to say
the simulation is live.

Every other asset switches between static views. Those use the toggle with **no
dot at all** — not a grey dot, not a still dot. A dot that never changes is decoration
pretending to be status.
