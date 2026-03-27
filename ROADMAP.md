# PCBMiller Feature Roadmap

Features and improvements identified by researching pcb2gcode and FlatCAM.

---

## Implemented ✓

- **Isolation routing** — multi-pass with configurable overlap %, V-bit angle/tip diameter, live cut-width preview
- **Excellon drill support** — parses .drl/.xln, groups by diameter, G1 plunge loops (no G81), nearest-neighbour sort
- **Circular milling for oversized holes** — set a circular mill bit diameter; holes larger than it are circular-milled with G2 arcs instead of drilled
- **Multi-bit drilling** — tool-change pauses with instructions between each drill size
- **G02/G03 arc support in copper Gerber parser** — curved traces and arcs in region fills now correctly interpolated
- **Bezier curve support in SVG copper parser** — cubic (C/S) and quadratic (Q/T) beziers properly subdivided
- **localStorage settings persistence** — all parameters survive page reload
- **Estimated run time** — shown in Board Info based on path lengths and feed rates
- **Clear/reset per file** — remove a loaded file without reloading the page
- **Machine presets** — Genmitsu 3018, 3018-PRO, Snapmaker, Shapeoko, X-Carve
- **Material presets** — FR-1 (paper phenolic) and FR-4 (fibreglass) with appropriate feed rates + FR-4 dust warning
- **Start point optimisation** — isolation loops start on the straightest segment, avoiding corner plunge marks
- **`; Units: mm` header** — emitted in every generated file
- **Safe Z default 5 mm** — avoids most clamp collisions

---

## High Impact

- **Autoleveller / Z-probing** — Probe a grid of Z heights across the board, dynamically compensate Z during milling. Essential for thin isolation cuts on warped boards (even 0.05 mm warp ruins traces at 0.1 mm depth). pcb2gcode generates a probing GCode file; the sender runs it first, stores the height map, then applies bilinear interpolation during the actual milling file.

- **Bridge/tab placement control** — Set bridge count and place them manually on arbitrary board shapes. Currently PCBMiller auto-places tabs but doesn't expose count or exact position.

- **G-code optimisation** — Remove redundant G1 moves that are collinear within a tolerance. pcb2gcode achieves up to 95% file size reduction. Useful for large boards with dense isolation passes.

- **V-cut scoring** — Generate a series of shallow passes along a line to score the board for snapping, rather than cutting all the way through.

---

## Medium Impact

- **Selective isolation** — Click a specific trace or copper polygon to isolate only that region. Useful for dense boards or iterative debugging.

- **Paint / copper pocket milling** — Fill all non-copper areas with concentric or raster toolpaths to fully clear copper (not just cut isolation channels). Slower but produces clean copper pours. FlatCAM calls this "Paint Area."

- **SVG / DXF import for board outline** — Accept custom board shapes beyond Gerber edge cuts files.

- **G-code flavor / controller compatibility** — Dropdown to target GRBL, LinuxCNC, Mach3, Mach4. Adjusts output syntax (e.g. removes G81, changes spindle commands, units handling).

---

## Double-Sided PCB (Significant Scope)

- **Mirror/flip with alignment holes** — Mirror all layer coordinates around a specified axis. Automatically compute and duplicate alignment hole positions so the board can be flipped and re-registered accurately. FlatCAM handles this as a first-class workflow.

---

## Lower Priority / Niche

- **Panelisation** — Array N copies of the design on one blank with shared edge cuts. FlatCAM has a full panel/array tool with area constraints.

- **Gerber line drawing (silk screen)** — Attach a pen to draw the silk screen layer directly on the PCB surface.

- **Invert Gerbers** — Mill the traces themselves rather than the gaps around them (used in some toner-transfer hybrid techniques).

- **Backtracking optimisation** — Continue milling backward along a completed path instead of retracting, when that is faster than retract + rapid move.

- **Bed flattening** — Generate a surfacing toolpath to flatten the wasteboard/sacrificial layer before milling begins.

---

## Small Polish — The Details That Make It Good

### Warnings & Validation

- **Minimum clearance warning** — When isolation offset is larger than the gap between two copper features, they'll merge. pcb2gcode prints a warning when copper areas are too close; PCBMiller should flag this in the UI (e.g. "2 gaps narrower than your tool diameter — they will be bridged").
- **Thin trace warning** — Warn if any copper trace is narrower than the tool diameter. It will be cut through and lost entirely. Recommended design minimums: 0.4 mm trace width, 0.4 mm clearance.
- **No isolation paths generated** — Currently silent if clipper returns nothing. Should say why (e.g. tool too wide for this board's clearances).
- **Board size sanity check** — If parsed board is impossibly large or small (e.g. coordinate unit mismatch), warn before generating multi-metre toolpaths.

### Depth & V-Bit Physics

- **"Shallow is safer" guidance** — Start at −0.03 mm; cutting too shallow means re-run, cutting too deep means lost traces. The penalty is asymmetric. Add a hint to the depth field.
- **Depth-to-width ratio hint** — The deeper you go with a V-bit, the wider the isolation gap. At 90° tip angle, each extra 0.1 mm of depth adds 0.2 mm of width. At 30° it adds only 0.054 mm. This should be visible somewhere in the UI.

### Feed Rate Guidance

- **FR-4 vs FR-1** — FR-4 (fibreglass) is much harder; recommended feed 100–150 mm/min for isolation. FR-1 (paper phenolic) is safer and easier; 200+ mm/min viable. *(Material presets now partially address this.)*
- **Plunge rate should be slower than cut rate** — Most tools default plunge to ~30–50% of XY feed. Currently PCBMiller uses a fixed plunge rate; it should default to something like `min(feedRate, 100)` mm/min.
- **Spindle speed for small bits** — Sub-mm V-bits and drill bits prefer 10,000+ RPM. At lower RPM they rub instead of cut and break. Warn if spindle RPM seems low for the bit diameter.

### Toolpath Quality

- **Nearest-neighbour sort already exists but could show travel distance** — Display estimated total rapid travel distance and total cut distance in the GCode stats. Helps users judge if the sort is working.
- **Climb vs conventional milling toggle** — pcb2gcode defaults to conventional (safer for thin bits, less deflection). Some users prefer climb for cleaner edges. A checkbox would cover both camps.
- **Minimum path length filter** — Very short isolation segments (< 0.5 mm) are artefacts from rounding or near-coincident copper features. They cause the spindle to plunge, move 0.2 mm, retract — wearing the bit for no benefit. Filter them out with a configurable minimum.

### UX / Workflow

- **"Pen plot" dry run mode** — Generate GCode at Z=+1 mm (above the surface) so the user can run it as a pen-on-paper test before cutting copper. pcb2gcode users do this manually; a button would make it obvious.
- **GCode line count and estimated time** — Show approximate run time based on total path length ÷ feed rate + rapid moves. *(Estimated time now shown in Board Info.)*
- **Export settings as config file** — Let users download/upload a JSON of all current parameters. Saves re-entering settings for repeat boards on the same machine.
- **Copy GCode button** — Already present alongside download.
- **File drag-and-drop on the canvas** — Already supported.
- **Clear/reset button per file** — Already implemented.

### GCode Output Quality

- **`G64 P0.01` path blending** — LinuxCNC and some GRBL builds support path blending tolerance. Inserting `G64 P0.01` (blend within 0.01 mm) dramatically speeds up isolation routing by not requiring exact stops at each waypoint.
- **`G4 P0` dwell after spindle on** — Some spindle controllers are slow to reach speed. A configurable dwell (already in PCBMiller) is correct but the default should be noted as "increase if spindle is slow to start."
- **Comment density** — pcb2gcode puts a `;` comment on every operation type change. PCBMiller already does this well for sections but could add per-pass comments.

### Physics / Material Notes Worth Surfacing in the UI

- **Board flatness tolerance** — A V-bit at 0.1 mm depth has zero tolerance for board warp. Even 0.05 mm of warp doubles or halves the cut width. A hint near the depth field: "Ensure board is flat to within ±0.02 mm; use autolevelling or a surfaced wasteboard."
- **Spindle runout** — 0.1 mm of bearing play is already too much for 0.1 mm isolation cuts. Nothing the software can do, but worth calling out in a tooltip.
- **FR-4 health warning** — Fibreglass dust is a serious respiratory hazard. *(Dismissable warning now shown when FR-4 material preset is selected.)*
