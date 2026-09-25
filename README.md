# Streamliner — Framer case study assets

Self-contained HTML embeds for the Dell Streamliner case study on Framer
(`/projects/streamliner`). Each file in `assets/` is named after the Framer node
that hosts it, so the mapping is unambiguous.

**Live index:** https://souvikb93.github.io/streamliner-final/

## Why this repo exists

The assets used to live *inside* Framer as inline `html` attributes on Embed
nodes. Every edit meant rewriting the entire file into the node — for the
system diagram that is ~27,000 characters per change, regardless of how small
the change was.

Hosting them here inverts that: the Framer node points at a URL once and never
changes again. Edits happen in git, Pages redeploys, the embed picks it up.

## How to change an asset

```bash
# edit the file
vim assets/OX6dWZqL5.html

# check it renders and measure its height
open assets/OX6dWZqL5.html

git commit -am "system diagram: <what changed>"
git push
```

Pages redeploys in under a minute. Hard-refresh the Framer preview to bust the
CDN cache.

## Wiring an asset into Framer

On the Embed node, set the URL to the raw Pages address:

```
https://souvikb93.github.io/streamliner-final/assets/<nodeId>.html
```

Keep the node's height lock as it is — these files set their own desktop height
through a `@media (min-width:900px){body{min-height:Npx}}` rule, because Framer
measures the embed document and writes the result back to the node.

## Assets

| Node | Asset | Desktop height |
|---|---|---|
| `OX6dWZqL5` | To-be system diagram | 700 |
| `iVhhBwFBm` | To-be service blueprint | 678 |
| `cJPS6EXdE` | Journey map | 700 |
| `irYUR150B` | Decisions and trade-offs | 660 |
| `k567uewrD` | Synthesis — findings to opportunities | 644 |
| `jDPWO0srd` | Discovery method | 605 |
| `k3rxHcFvx` | Challenge and goal | 580 |
| `riEAs3sap` | As-is service blueprint | 560 |
| `i0e_jTpCf` | Co-creation | 560 |
| `TKuq5vJQK` | Research world map | 488 |
| `CN6IB8Tax` | Rules engine | 460 |
| `LhtrzwMXl` | Design response | 430 |
| `rDjTt_Ep8` | Core + rule layer | 420 |
| `CLTQi8f4o` | Overview stats | 400 |
| `gdBqnnNCX` | 17 → 7 → 3 → 1 funnel | 400 |
| `KhZOFyrxi` | Five modules | 384 |
| `zX_dFoAE6` | Impact | 300 |
| `hCEOXL0l5` | Five-step process | 220 |

## Constraints every file follows

- **No build step.** One file, inline CSS and JS. Only external request is the
  Uncut Sans variable font from jsDelivr.
- **Fixed desktop height.** The height must not change when a tab or region is
  selected — Framer writes the measured height back to the node, so a shifting
  document makes the page jump.
- **Responsive inside the file.** Framer's M (810) and S (390) breakpoints are
  zero-override replicas of L (1200), so they share one node height. All
  responsive behaviour lives in the HTML's own media queries.
- **Reduced motion respected.** Animations collapse but content stays visible.
- **4.5:1 contrast floor** on every piece of text.

Full rules: [`docs/FRAMER_ASSET_DESIGN_SYSTEM.md`](docs/FRAMER_ASSET_DESIGN_SYSTEM.md)
Project specifics: [`docs/STREAMLINER_ASSET_GUIDELINES.md`](docs/STREAMLINER_ASSET_GUIDELINES.md)

## Verifying a change

Headless Chrome, measuring document height and overflow at the three widths that
matter (1072 is 1200 minus Framer's 64px section padding):

```bash
for w in 1072 682 350; do
  # see docs for the iframe harness — Chrome floors window width at 500
done
```
