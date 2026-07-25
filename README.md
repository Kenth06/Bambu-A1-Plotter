# Bambu A1 Plotter

Turn a **Bambu Lab A1** into a pen plotter. A single HTML file that converts images
and SVG files into pen-plotter G-code, entirely in your browser.

No install, no account, no build step, and no server: open `index.html` and it works
offline. Nothing you load is ever uploaded anywhere.

---

## ⚠️ Read this first

**Use at your own risk.** This generates motion commands for a 3D printer being used
outside its intended purpose. A misconfiguration can crash the toolhead, drag it
across the bed, or damage the printer, the pen mount, or the build surface.

- **Bambu Lab A1 only.** Not verified on the A1 Mini, P1P, P1S, X1 Carbon, or any
  other machine. Bed sizes and motion systems differ (some move the toolhead, others
  move the bed) and they behave differently when homing or pausing. Do not assume
  this G-code is safe on anything but an A1.
- **Never skip the setup hold or the boundary check.** They exist so you can catch a
  mistake before the pen touches the paper. Do not walk away until both have passed.
- **Nozzle must be set to 0.4 mm** on the printer *and* in the active Bambu Studio
  profile. A mismatch can make Bambu Studio or the printer reject the file.
- **No filament, no heat.** The tool commands the heaters off. There is no reason to
  preheat at any point; if anything is loaded, unload it before plotting.
- No liability is accepted for damage to your printer, your workspace, or for any
  injury arising from use of this tool. Supervise the machine while it runs.

---

## Quick start

1. Download or clone this repository.
2. Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge).
3. Drop in an image (JPG / PNG / WEBP) or an SVG.
4. Set your **Pen Y Offset** and **Pen Down Z** (see *First-time setup* below).
5. Check the result in **Vector preview** and **Draw simulation**.
6. Open the **G-code** tab and click **Download**.
7. Send the `.gcode` file to the A1 with Bambu Studio, and follow the on-machine
   sequence in *How a plot runs*.

## Features

- **Image tracing** in three modes: Sketch (edge linework), Halftone (tone as grain
  or dots) and Lo-fi (reduced-resolution, chunky steps).
- **SVG import** — paths are sampled directly and plotted as a single colour.
- Pre-trace image adjustments: brightness, contrast, light-image boost
  (auto-contrast) and invert for light-on-dark sources.
- Full machine control: bed size, placement (centered / anchored / custom), margins,
  pen-down Z, pen-up safe Z, setup hold time and pen Y offset.
- **Vector preview** — true-to-scale view of the plate with the artwork in position.
- **Draw simulation** — animated stroke-order playback with scrub bar and speed
  control.
- **G-code output** with download and copy, plus SVG export of the traced vector.
- Runs fully client-side. No network requests, no telemetry.

Not included: multi-colour layer separation with hatch fill, plotted text, QR codes,
frame guides and ASCII art.

## What you need

| Item | Notes |
| --- | --- |
| Bambu Lab A1 | This machine only — see the warning above. |
| Pen mount | Fitted to the **front** of the toolhead. The nozzle stays in place. |
| Bambu Studio | For sending the generated G-code to the printer. |
| A modern browser | Chrome, Firefox, Safari or Edge. |
| 0.4 mm nozzle profile | Set on the printer *and* in the active Bambu Studio profile. |
| Paper | Taped down firmly on all four sides. |

## First-time setup

Do this once, before your first plot.

The key idea: **you do not fit the pen before starting a plot.** The printer homes
first — which touches the nozzle down to the plate — and only then pauses with the
toolhead in position for you to fit and calibrate the pen. This keeps the pen well
clear of the homing routine.

1. Fit the pen **holder** to the front of the toolhead, empty (or with the pen
   retracted), so that an installed pen tip would sit forward of the nozzle. The
   tool's coordinate math is built around this forward position.
2. Measure the nozzle tip to pen tip distance **along the Y axis only**
   (front-to-back). That is your **Pen Y Offset**. The default of 48 mm is very
   likely wrong for your mount and pen — measure your own.
3. Open the tool and enter the measured offset under **Bed & Placement**.
4. Under **Pen (Z-lift)**, set **Pen Down Z** to roughly where the pen tip should sit
   to touch the paper. Ballpark is fine; real contact is set by sliding the pen in
   its mount during the setup hold, not by perfecting this number.
5. Confirm the 0.4 mm nozzle setting in both places.
6. Run one test plot on scrap paper. Use the setup hold to fit and calibrate the pen,
   then watch the boundary check confirm the plot stays on the plate. Only move to a
   real piece once both look right.

## How a plot runs

Every plot follows the same four stages.

**1. Homing (G28).** Mandatory and not skippable — it is what keeps the origin
accurate. The A1 homes Z by touching the nozzle position down to the plate. Because
the pen is fitted *after* homing, it is nowhere near the plate during that touch-off.
Keep the pen out of the mount, or retracted, until the hold begins.

**2. Setup hold.** The toolhead travels to the **front-left corner** of your artwork,
lowers to pen-down height and holds still (30 s by default). This is when you fit and
calibrate the pen:

- Slide the pen into the mount until the tip just touches the paper at that corner.
- Nip up the clamp so the pen cannot move.
- That is the calibration — the tip is now set to the paper surface at a known,
  repeatable point.

The hold is always at the same corner at the same height, so every plot calibrates
from an identical reference. If 30 s is too short, raise it: **Pen (Z-lift) → Setup /
swap hold time**, range 5–120 s.

**3. Boundary check.** The pen lifts and traces the outline of the artwork's bounding
box **pen up**, making no marks. Watch this closely. If the head heads for an edge or
off the visible plate, **stop the print immediately** — something is misconfigured,
usually the Pen Y Offset.

**4. Plotting.** The pen lowers and the artwork draws. A single colour is plotted, so
there is no colour-swap stage: once plotting starts it runs straight through.

## Interface reference

One page: a settings sidebar on the left, preview and output on the right. Every
sidebar section starts collapsed — click a section title to open it.

### Source
Drop an image (JPG / PNG / WEBP) or an SVG, or click to browse. Images can be rotated
90° either way before tracing. SVG paths are sampled directly and plotted as a single
colour. *Remove image* clears the source.

### Trace
Converts a raster image into plotted linework.

- **Sketch** — traces edges into clean lines.
- **Halftone** — converts tone into a grain or dot pattern.
- **Lo-fi** — traces at reduced resolution for a chunky, stepped look.

Each mode exposes its own sliders, mirrored as a two-slider quick strip under the
vector preview for fast tweaking without opening the sidebar. Brightness, Contrast
and Light Image Boost (auto-contrast) adjust the source before tracing; Invert is for
light-on-dark sources.

### Bed & Placement
Bed size, how the artwork is placed on it (Centered, Anchored to a corner, or
Custom), margins, and the Pen Y Offset.

### Pen (Z-lift)
Pen Down Z, Pen Up (safe) Z, and the setup / swap hold time.

### Preview tabs
- **Vector preview** — fixed, true-to-scale view of the plate with the artwork at its
  real position.
- **Draw simulation** — the same view animated in stroke order, with play/pause, a
  scrub bar and a speed slider.
- **G-code** — the generated file, with Download and Copy.

Both preview tabs have zoom controls (−, percentage, +, Reset) above the panel. If
the zoomed view is larger than the window, scroll to reach the rest of it.

## Thicker paper, card and other surfaces

The tool is not limited to standard paper. Card, thick watercolour stock, or anything
raised above the plate sits closer to the pen, so the pen needs to travel less far
down to reach it.

The control for this is **Pen Down Z**: how far the pen lowers on a pen-down move,
measured as a height in mm above the build plate (the default of 1 mm stops the
toolhead 1 mm above the plate). Higher values keep the pen higher — the thicker the
surface, the higher the pen should stop.

- **Thin paper** — Pen Down Z near your normal calibrated value.
- **Thicker card / stock** — increase it, since the surface top is closer to the pen.
- **Very thick or uneven surfaces** — increase further, and lean on a spring mount to
  absorb the variation across the sheet.

This does not need to be exact. Pen Down Z only needs to be in the right ballpark;
real contact is still set by sliding the pen in its mount during the setup hold.

Whatever the surface, tape it down flat and firmly on all sides. Card can bow or lift
as the pen drags across it, and any movement ruins registration.

## Choosing a pen mount

The physical mount matters as much as the software settings. Four qualities to look
for or build toward:

**Front-mounted.** The holder attaches ahead of the nozzle rather than replacing it —
the nozzle stays, the pen sits forward of it. The tool's whole coordinate model
assumes this (that is what Pen Y Offset measures). A clean front position keeps the
offset simple, consistent and small.

**A spring mechanism.** A spring-loaded holder lets the tip float slightly instead of
being locked rigidly to the toolhead. It compensates for small inconsistencies in
paper thickness and bed flatness, keeping line pressure consistent without a perfect
Z number, and gives you forgiveness when setting the tip against the paper during the
hold — the spring absorbs minor over-travel instead of digging in or lifting off.

**A friction-tolerant tip.** The tip is dragged sideways across paper repeatedly at
speed. Technical fineliners, ballpoints, gel pens and hard plastic or metal nibs hold
up well. Fine felt-tips and soft brush pens fray or wear down quickly and are the
least suited to plotting.

**An adjustable clamp.** Pen barrels vary a lot in diameter. An adjustable clamp,
grub screw or tunable friction collar means you are not tied to one pen model, and
you can dial in enough grip to stop the pen twisting during fast direction changes
without over-tightening and cracking a thin barrel.

## Troubleshooting

**Plot is offset from where the simulation showed it.**
The Pen Y Offset is probably wrong. Re-measure the physical distance from the nozzle
position to the pen tip.

**Pen will not reach the paper at the hold.**
Slide the pen further down in the mount until the tip meets the paper, then tighten.
That is the intended fix — contact is set by moving the pen, not by chasing the Pen
Down Z number. If the pen bottoms out in the mount and still cannot reach, lower Pen
Down Z slightly and re-run.

**Pen presses too hard, drags or digs in at the hold.**
Back the pen up slightly in the mount before tightening. A spring mount gives you
some forgiveness here.

**Pen lifts off the paper in some areas of the bed but not others.**
The plate is not perfectly flat and the tool plots as though it were. Every bed
deviates a little, usually most toward the back, and a pen has none of the give
molten filament does — a few tenths of a millimetre is the difference between a solid
line and no line at all. No setting fixes this. What helps, in order of impact:

1. A spring-loaded pen mount. This is the real answer, and what plotters have always
   used — the spring absorbs the variation mechanically.
2. Plot nearer the centre and front of the plate, where the surface is usually
   flattest. Shrinking the artwork slightly is often enough.
3. Lower Pen Down Z slightly so the pen presses harder everywhere. This trades one
   problem for another: it fixes the light areas but presses harder where things were
   already fine, which can broaden lines or splay a soft nib.
4. Run a bed calibration from the printer's screen. Worth doing, but not a fix on its
   own.

**Boundary check goes off the visible plate.**
Stop the print. Check Pen Y Offset and the placement setting (Centered / Anchored /
Custom) before trying again.

**Nozzle heats up during a park or pause.**
Normal and expected. The A1's firmware park routine heats the nozzle; it is hardcoded
into the printer and cannot be switched off from G-code. It is harmless for plotting
— no filament is loaded, so nothing extrudes — and the tool re-issues heaters-off
commands as soon as the plot resumes. Do not manually load filament or leave any in
the hotend.

**Bambu Studio rejects the file or reports a nozzle mismatch.**
Confirm the nozzle is set to 0.4 mm in both the printer's own settings and the
currently active Bambu Studio profile.

**A sidebar section shows no controls.**
Sections start collapsed. Click the section title to open it.

**The preview or simulation is too small to work with.**
Use the −, + and Reset buttons above the panel to zoom.
