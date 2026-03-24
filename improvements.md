# PCBMiller — Suggested Improvements

## Isolation Routing

### ~~Nearest-neighbor path ordering~~ ✓ done
~~Currently Clipper returns isolation loops in arbitrary order, causing unnecessary rapid-travel moves between distant loops. A simple nearest-neighbor sort (pick next loop whose start point is closest to current tool position) would reduce air-time significantly on boards with many traces.~~

### Voronoi clipping (pcb2gcode approach)
Build a Voronoi diagram treating each copper shape as a site. Clip each trace's isolation path to its Voronoi cell so the tool never re-mills copper already cleared by an adjacent trace's path. Minimises total copper removal and prevents double-cutting in tight areas.

### Already-milled tracking
After each pass, track the area already cleared (union of all tool-width swaths milled so far). Clip subsequent passes against this region to avoid redundant cuts when two traces share a clearance zone.

### SVG trace strokes
The SVG parser currently only extracts filled shapes. Traces exported from KiCad as stroked `<path>` elements (with a `stroke-width` attribute and no fill) are not captured. Fix: read `stroke-width` and treat open paths as trace segments, converting them to stadium polygons the same way the Gerber D01 handler does.

### Gerber arc (G02/G03) support
The Gerber parser handles D01 linear strokes but ignores curved arcs (G02 clockwise / G03 counter-clockwise). Arcs appear in curved traces and rounded board outlines. Fix: detect the G02/G03 modal and convert I/J arc parameters to a polyline approximation before adding to `segments`.

### Aperture macro shape fidelity
Unknown aperture macros are currently approximated as rectangles using the first two numeric parameters. KiCad's `RoundRect` macro specifically encodes corner radius as its third parameter; using `w × h` with rounded corners (stadium-cap ends) would be more accurate.

## Edge Cuts

### ~~Clipper-based edge-cuts offset~~ ✓ done
~~`offsetPolygon` for the board outline still uses the vertex-bisector method, which misbehaves on non-convex board shapes (L-shaped, tabbed, or notched boards). Replacing it with ClipperOffset (already loaded) would handle all shapes correctly.~~

### Arc support in edge-cuts Gerber
Curved board edges (G02/G03) are silently ignored by the Gerber parser and will produce straight chord approximations. Same fix as the isolation arc support above.

## G-code Quality

### Feed-rate optimisation
Use a higher feed rate for rapid-travel segments between isolation loops (currently all motion uses the same feed). Emit `G0` for rapids with safe-Z retract, which is already done, but also consider reducing plunge rate only on the first plunge per loop.

### ~~GRBL spindle warm-up delay~~ ✓ done
~~The Genmitsu 3018 spindle needs a moment to reach speed after `M3`. Add a configurable dwell (`G4 P<seconds>`) after `M3` before the first cut move. Currently the tool can contact the board before the spindle is at speed.~~

## Drill Files

### ~~Excellon drill file support~~ ✓ done
~~Parse `.drl` / `.xln` Excellon files to generate drill G-code (`G81` canned cycles or explicit plunge loops for GRBL). Display drill hits as crosses on the canvas overlay. This completes the basic PCB-to-CNC workflow.~~

## UX

### ~~Preview: show isolation paths coloured by pass~~ ✓ done
~~Draw pass 1 paths in one colour, pass 2 in another, etc., so the user can see how much copper is being cleared per pass.~~

### Preview: toggle copper / isolation / edge-cuts layers
Checkboxes to show/hide each layer in the canvas preview.

### Warning when adjacent traces are too close
After computing isolation paths, check if any two copper regions are closer than `isoWidth`. Warn the user that those traces cannot be isolated with the current settings.
