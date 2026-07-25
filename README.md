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
