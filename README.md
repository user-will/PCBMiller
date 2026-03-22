# PCBMiller

![PCBMiller](pcbmiller.png)

A tool for generating CNC G-code from PCB Gerber files that doesn't completely suck. Open `index.html` and go — no install, no server, no dependencies.

---

## The Goal

Most PCB milling workflows involve expensive software, complicated CAM setups, or cloud tools that require accounts. PCBMiller is the opposite: clone the repo, open one HTML file in your browser, drop in your Gerber, and get G-code you can send straight to your CNC with Universal Gcode Sender.

The aim is to make this as accessible as possible — if you can design a PCB in KiCad, you should be able to mill it without fighting your toolchain.

---

## Usage

```
git clone https://github.com/user/PCBMiller
cd PCBMiller
open index.html   # or just double-click it
```

1. Export your **Edge Cuts** layer from KiCad as a Gerber (`Edge_Cuts.gbr`, `.gko`, or `.gm1`)
2. Drop the file onto the drop zone
3. Adjust the milling parameters for your tool and material
4. Optionally click **Place Tabs** to add holding tabs so the board doesn't shift on the final pass
5. Click **Generate G-code**, then copy or download the `.nc` file
6. Send it to your CNC with [Universal Gcode Sender](https://winder.github.io/ugs_website/)

---

## Features

- **Visual preview** — board outline and toolpath shown on a 2D canvas with pan and zoom
- **Multi-pass depth** — set a pass depth and the tool steps down incrementally rather than plunging to full depth in one go
- **Tabs** — click directly on the toolpath to place holding tabs; set how many passes thick each tab is
- **Live parameter feedback** — pass count and tab thickness update as you type
- **No install** — pure HTML + JavaScript, runs entirely in the browser

---

## Tested On

- Genmitsu 3018 Pro (GRBL 1.1f)
- Universal Gcode Sender

---

## Roadmap

- Isolation routing (engraving trace outlines)
- Drill file support
- More Gerber layer types (copper, silkscreen preview)
