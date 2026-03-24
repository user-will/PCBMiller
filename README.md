# PCBMiller

![PCBMiller](pcbmiller.png)

A tool for generating CNC G-code from PCB Gerber files that doesn't completely suck. Open `index.html` and go — no install, no server, no dependencies.

> **⚠ ACTIVE DEVELOPMENT — USE AT YOUR OWN RISK**
> This tool is under active development. G-code output may be incorrect, incomplete, or unsafe for your specific machine and setup. **Always simulate or dry-run before cutting.** Check feeds, depths, and toolpaths carefully before sending anything to your CNC. You are responsible for verifying the output. Broken bits, damaged boards, and ruined spoilboards are on you.

---

## The Goal

Most PCB milling workflows involve expensive software, complicated CAM setups, or cloud tools that require accounts. PCBMiller is the opposite: clone the repo, open one HTML file in your browser, drop in your Gerber, and get G-code you can send straight to your CNC with Universal Gcode Sender.

If you can design a PCB in KiCad, you should be able to mill it without fighting your toolchain.

---

## Usage

```
git clone https://github.com/user/PCBMiller
cd PCBMiller
open index.html   # or just double-click it
```

### Board Outline (Edge Cuts)

1. Export your **Edge Cuts** layer from KiCad as a Gerber (`Edge_Cuts.gbr`, `.gko`, or `.gm1`)
2. Drop the file onto the **Board Outline** drop zone
3. Adjust the milling parameters (tool diameter, pass depth, feed rate)
4. Optionally click the toolpath to place **holding tabs** so the board doesn't pop free on the final pass
5. Click **Generate G-code**, then copy or download the `.nc` file

### Isolation Routing (Copper Traces)

1. Export your **copper layer** from KiCad as a Gerber (`F_Cu.gbr`) or SVG
2. Drop it onto the **Copper Layer** drop zone — traces and pads appear as an overlay
3. Set your isolation tool diameter, clearance width, number of passes, and overlap
4. Isolation toolpaths are computed automatically; increase passes to clear more copper between features
5. For boards with a **ground fill**, load the Edge Cuts file too — the tool uses the board boundary to correctly compute the gaps between traces and the ground plane

---

## Features

- **Visual preview** — copper layer, isolation paths, and board outline shown on a 2D canvas with pan and zoom
- **Isolation routing** — mills grooves around copper traces to electrically isolate them; handles ground-fill boards, pads, and SVG exports from KiCad
- **Ground-fill aware** — uses a Clipper-based union+difference algorithm so isolation passes correctly fill the gaps between traces and a ground plane rather than stopping at overlap points
- **Multi-pass depth** — set a pass depth and the tool steps down incrementally rather than plunging to full depth in one go
- **Tabs** — click directly on the toolpath to place holding tabs; set how many passes thick each tab is
- **Live parameter feedback** — path count, pass depth, and tab thickness update as you type
- **No install** — pure HTML + JavaScript, runs entirely in the browser

---

## Supported Inputs

| File type | What it's used for |
|---|---|
| Gerber Edge Cuts (`.gbr`, `.gko`, `.gm1`) | Board outline + edge milling toolpath |
| Gerber copper layer (`F_Cu.gbr`) | Isolation routing |
| SVG copper export | Isolation routing (KiCad SVG export) |

---

## Tested On

- Genmitsu 3018 Pro (GRBL 1.1f)
- Universal Gcode Sender

---

## Roadmap

- Drill file support (Excellon `.drl`)
- Gerber arc support (G02/G03 curved traces)
- Preview: colour paths by pass number
- Warning when traces are too close to isolate with the current tool
