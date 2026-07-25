# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

A single-file browser tool that converts raster images and SVG files into pen-plotter
G-code for a **Bambu Lab A1** with a pen mount fitted to the front of the toolhead.

Everything lives in `index.html` — markup, CSS and JS in one file, no dependencies,
no build step, no server, no network requests at runtime. Opening the file in a
browser *is* running the app. Do not introduce a bundler, a package manager, a
framework, or any external asset (CDN script, webfont, remote image); the offline,
single-file property is the point.

```
index.html        the entire application (~3.9k lines)
docs/MANUAL.txt   plain-text manual, mirrors the MANUAL_TEXT constant in index.html
README.md         user-facing documentation
```

## Working in index.html

Rough layout of the file:

| Lines (approx.) | Contents |
| --- | --- |
| `<head>` … ~715 | CSS. Custom properties are `--pf-*`; component classes are `pf-*`. |
| ~716–1150 | Markup: header, sidebar sections, preview panels, help panel, footer. |
| ~1155–1525 | `MANUAL_TEXT` — the embedded manual template literal. |
| ~1525–1560 | App state and element refs. |
| ~1560–1940 | UI wiring: file loading, sliders, quick sliders, tabs, zoom, theme. |
| ~1940–2550 | Tracing: grayscale, contrast, blur, Sobel, edge chains, halftone modes, hatch, spiral, Douglas-Peucker simplify, nearest-neighbour path ordering, SVG sampling. |
| ~2550–3300 | Paper-space transform, vector preview, draw simulation. |
| ~3300–3600 | G-code generation and exports. |

Line numbers drift — grep for the `// ---------------- section ----------------`
banners rather than trusting them.

### Coordinate model — the thing most likely to break

Documented in the comment block above `makeMmTransform`. The invariants:

- Tracing and layout work entirely in **paper space**. Conversion to toolhead
  coordinates happens once, at G-code emit time, in `toToolheadXY`.
- **Y0 is the FRONT of the plate**, Y increases toward the back. Source-image top maps
  to high paper Y, so the plot reads upright viewed from the front of the printer.
- The pen sits `penOffsetY` mm ahead of the toolhead, so placing the pen at paper Y
  means the toolhead sits at `paperY - penOffsetY`. Toolhead Y cannot go negative,
  which makes the rear `penOffsetY` strip of the plate permanently unreachable.
- The source→paper transform is computed **once** from all included paths, so every
  pass shares one registration. Do not recompute it per path or per pass.

Any change here can drive the toolhead off the bed on real hardware. Re-derive the
math rather than patching symptoms.

### Generated G-code

`buildGcode` emits a fixed safety sequence that must be preserved in order:

1. `G28` home all axes — mandatory, never make this conditional or skippable.
2. **Setup hold** — travel to the front-left corner of the artwork, pen down, dwell
   (5–120 s, default 30). The user fits and calibrates the pen during this hold.
3. **Boundary check** — pen-up trace of the artwork bounding box.
4. Plotting.

Heaters are commanded off; the tool never extrudes and never loads filament. The A1's
firmware park routine heats the nozzle on its own — that is expected, and the code
re-issues heaters-off on resume. Do not add heating, extrusion, or a nozzle-based
levelling step.

## Scope

This build ships single-colour output only. Multi-colour layer separation with hatch
fill, plotted text, QR codes, frame guides and ASCII art are deliberately absent. Some
dead scaffolding remains (`textActive`, `applyTextOverlay`, hidden SVG-hatch
fallbacks, colour-pass filename logic); leave it alone unless a change actually needs
it, and do not document absent features as if they were present.

## Verifying a change

There is no test suite. After editing:

```sh
# extract the inline script and syntax-check it
awk '/^<script>$/{f=1;next} /^<\/script>$/{f=0} f' index.html > /tmp/tool.js
node --check /tmp/tool.js
```

Then open `index.html` in a browser, load a source image, and confirm the vector
preview, the draw simulation and the G-code tab all still render. For anything that
touches geometry or G-code, read the emitted output and check the boundary-check
block before considering it done.

Keep `docs/MANUAL.txt` and the `MANUAL_TEXT` constant in sync — they are the same
document in two places, and the in-app "Download manual" button serves the constant.

## Conventions

- Commits are Conventional Commits (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`),
  atomic, one concern each, with a body explaining *why*.
- British spelling in user-facing copy (`colour`, `centred`), matching the existing
  text.
- Safety wording in the README, the manual and the app footer is deliberate. Do not
  soften the A1-only warning, the setup-hold/boundary-check instruction, or the
  liability disclaimer.

## Known drift

The in-app help panel (around line 1022) still refers to a pen-swap prompt on
multi-colour plots and suggests fitting the pen before homing. Both contradict the
current single-colour, fit-the-pen-during-the-hold workflow described everywhere else.
Worth fixing next time that panel is touched.
