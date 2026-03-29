# PCBMiller

![PCBMiller](pcbmiller.png)

A tool for generating CNC G-code from PCB Gerber files that doesn't completely suck. Open `index.html` and go — no install, no server, no dependencies.

> **⚠ ACTIVE DEVELOPMENT — USE AT YOUR OWN RISK**
> This tool is under active development. G-code output may be incorrect, incomplete, or unsafe for your specific machine and setup. **Always simulate or dry-run before cutting.** Check feeds, depths, and toolpaths carefully before sending anything to your CNC. You are responsible for verifying the output. Broken bits, damaged boards, and ruined spoilboards are on you.

---

## The Goal

Most PCB milling workflows involve expensive software, complicated CAM setups, or cloud tools that require accounts. PCBMiller is the opposite: clone the repo, open one HTML file in your browser, drop in your Gerbers, and get G-code you can send straight to your CNC.

If you can design a PCB in KiCad, you should be able to mill it without fighting your toolchain.

---

## Usage

```
git clone https://github.com/user/PCBMiller
cd PCBMiller
open index.html   # or just double-click it
```

### Single-Layer Board

1. Drop your **Edge Cuts** Gerber (`Edge_Cuts.gbr`, `.gko`, `.gm1`) to set the board outline
2. Drop your **F.Cu** Gerber or SVG for isolation routing
3. Drop your **drill file** (`.drl`, `.exc`) for through-holes
4. Adjust tool diameter, pass depth, feed rate, isolation width and passes
5. Optionally place **holding tabs** by clicking on the outline toolpath
6. Hit **Generate G-code** → copy or download the `.nc` file

### Two-Layer Board

1. Load edge cuts and front copper as above
2. Check **Two-Layer Board** in the Copper Layer section
3. Drop your **B.Cu** Gerber into the back copper drop zone
4. In the **Alignment Pins** card, set your pin diameter and click **Place Pins on Canvas** to click 2+ pin locations onto the board (use asymmetric placement to prevent loading the board backwards)
5. Use the **Front / Both / Back** toggle in the preview to check both sides
6. Generate → download **Side1.nc** and **Side2.nc**

**Two-layer workflow:**
- Run Side1.nc: mills front copper, drills alignment holes + through-holes, cuts board outline
- Insert pins into the drilled holes (also drill matching holes in your spoilboard)
- Flip the board face-down onto the pins, re-zero Z
- Run Side2.nc: mills back copper with automatically mirrored coordinates

---

## Features

### File Inputs

| File | Purpose |
|---|---|
| Edge Cuts Gerber (`.gbr`, `.gko`, `.gm1`) | Board outline + edge milling |
| Front copper Gerber (`F_Cu.gbr`) or SVG | Front isolation routing |
| Back copper Gerber (`B_Cu.gbr`) or SVG | Back isolation routing (two-layer) |
| Drill file (`.drl`, `.exc`) | Through-hole and via drilling |
| V-cut Gerber | V-score line scoring |

### Isolation Routing
- Clipper-based union+offset algorithm correctly handles ground fills, overlapping pads, and complex copper geometries
- Multi-pass with configurable stepover — each pass clears one more tool-width of copper
- V-bit aware: set your bit angle and tip diameter, depth is auto-calculated from isolation width
- Click paths to select/deselect individual isolation passes before generating

### Drilling
- Parses Excellon drill files (KiCad PTH + NPTH)
- Groups holes by diameter, generates M0 tool-change pauses between bit sizes
- Nearest-neighbor ordering to minimize rapid travel

### Edge Cutting
- Multi-pass depth ramping with configurable pass depth
- Manual tab placement — click directly on the toolpath preview
- Inner contour selection for boards with cutouts
- **Depanelization mode** — cuts around all sub-boards in a panel with auto-spaced tabs

### V-Cut Scoring
- Partial-depth V-bit scoring for panel separation
- Click to select individual score lines

### Two-Layer PCBs
- Load front and back copper separately
- Back copper automatically mirrored in G-code output (`x = board_width − x`)
- Canvas preview toggle: Front / Both / Back — back side shown with correct horizontal flip
- Alignment pin placement tool: click to place pins on canvas, generates drill operations in Side 1 G-code
- Combined mode: single file with embedded flip instructions and M0 pause between sides
- Separate mode: Side1.nc and Side2.nc as standalone files

### Preview & UX
- Live 2D canvas preview with pan and zoom
- Color-coded layers: copper (amber), isolation (red), back copper (blue), back isolation (cyan), drills (purple), V-cuts (teal), outline (blue), tabs (yellow), alignment pins (yellow)
- Interactive path/contour selection via click or drag-select rectangle
- Runtime estimate updates as you change parameters
- G-code optimization: collinear move merging, RDP simplification, nearest-neighbor sorting
- No install — pure HTML + JavaScript, runs entirely in the browser, no build step

---

## Export Modes

**Combined** — single `.nc` file with M0 tool-change pauses between operations

**Separate** — individual files per operation: `Traces.nc`, `Drill.nc`, `Outline.nc`, `VCuts.nc`, `Depanel.nc`

**Two-layer** — `Side1.nc` (front) and `Side2.nc` (back), each a complete standalone file

---

## Tested On

- Genmitsu 3018 Pro (GRBL 1.1f)
- Universal Gcode Sender
