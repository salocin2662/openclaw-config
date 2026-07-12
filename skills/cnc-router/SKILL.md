---
name: "cnc-router"
description: "\"Design parts in OpenSCAD for CNC routing. DXF export, feeds and speeds, bit selection, BobCAD workflow, file types, cut types, job steps, tool heads and spindles.\""
---

# CNC Router — OpenSCAD to G-Code Skill

## Trigger
Use whenever Nicolas works on CNC router projects: designing parts, choosing bits, setting feeds/speeds, exporting for BobCAD, planning toolpaths, selecting cutting operations, organizing job workflow, or choosing spindle/collet/cutting head configurations.

---

## OpenSCAD → BobCAD Workflow

### 1. Design in OpenSCAD
Build parts parametric. Use 2D for cutting profiles, 3D for carving/relief.

```scad
// 2D profile for DXF export — simple box with pocket
difference() {
    square([100, 60]);
    circle(r=10);
    translate([20, 20]) square([30, 20]);
}
```

### 2. Export to DXF
- **Design → 2D outline** — use `projection(cut=true)` on 3D model, or build 2D directly
- **File → Export → DXF** — save alongside the `.scad`
- **For double-side milling** — export separate DXF files for top and bottom surfaces

### 3. Open in BobCAD
- File → Import → DXF
- Verify units (mm vs inches) match OpenSCAD settings
- Set stock dimensions
- Generate toolpath:
  - **Profile** — through-cut outlines
  - **Pocket** — clearing pockets with roughing/finishing passes
  - **Drill** — drilling holes
  - **Engrave** — text and inlays

### 4. Post-Process to G-Code
- Select post-processor for your CNC controller (Grbl, Mach3, etc.)
- Verify safe Z clearances
- Save `.nc` or `.gcode` file to USB or transfer directly

---

## Key OpenSCAD Techniques for CNC

### Rounding Internal Corners
Use the `Round-Anything` library to prevent sharp inside corners that stress end mills:
```
https://github.com/Irev-Dev/Round-Anything
```
Internal corner radius = end mill diameter ÷ 2 for a perfect fit.

### Kerf Compensation
When designing joints (box joints, finger joints):
- If your end mill is ¼" (6.35mm), cut at 6.35mm
- Kerf compensation in BobCAD adds/subtracts half the tool diameter
- Design with actual finished dimensions; compensate in CAM

### OpenSCAD-CNC Library
Simulate cutting operations inside OpenSCAD before exporting:
```
https://github.com/roberchi/OpenSCAD-CNC
```
```scad
use <cnc_path.scad>;
cut(path,
    name="cylinder",
    h=tool_height,
    d0=tool_diameter,
    speed=100);
```

---

## Feed & Speed Reference

### Core Formulas
```
RPM = (Surface Speed × 3.82) ÷ Tool Diameter (inches)
Feed Rate (in/min) = RPM × # Flutes × Chip Load (inches)
Plunge Rate = Feed Rate × 0.5 (or less)
Stepover = Tool Diameter × 0.3–0.5 (finishing vs roughing)
Stepdown = Tool Diameter × 0.5–1.0 (hard materials use less)
```

### Surface Speeds
| Material | Surface Speed |
|---|---|
| MDF / OSB | 650 ft/min |
| Aluminium | 600 ft/min |
| Acrylic | 500 ft/min |
| HDPE | 450 ft/min |
| Delrin / Acetal | 375 ft/min |
| Steel | 200 ft/min |
| Wax (prototyping) | 200 ft/min |
| Insulation foam | 1000 ft/min |

### Chip Loads — ¼" Bit
| Material | Chip Load |
|---|---|
| MDF | 0.013" – 0.016" |
| Aluminium | 0.005" – 0.007" |
| Acrylic | 0.008" – 0.010" |
| Delrin | 0.006" – 0.009" |
| HDPE | 0.007" – 0.010" |
| Steel | 0.0008" – 0.001" |

### Quick Online Calculators
- `pub.pages.cba.mit.edu/feed_speeds` — MIT Fablab (recommended)
- `zero-divide.net/fswizard` — full calculator with tool database
- `cncrouterstore.ca/pages/feed-rate-calculator` — router-specific

---

## Bit Selection Guide

### Upcut vs Downcut vs Compression

| Bit Type | Best For | Avoid When |
|---|---|---|
| **Upcut** | Hardwoods, plastics, 3D pockets | Sheet goods (tear-out on bottom) |
| **Downcut** | Plywood/MDF face milling | Hardwoods (burn risk, chip buildup) |
| **Compression** | **Best all-around for plywood/MDF** — clean top and bottom | Thin materials (< ¼") |
| **Straight flute** | Aluminium, soft metals | Wood (poor chip clearing) |

### Number of Flutes
- **2-flute** — best chip clearing for wood, general purpose
- **3-flute** — smoother finish, acceptable chip clearing
- **4+ flute** — finish cuts only, clogs easily in wood

### Bit Types by Job
| Job | Recommended Bit |
|---|---|
| Roughing pocketing | 2-flute upcut, ¼"–½" |
| Finishing passes | 3-flute, smaller stepover |
| Through-cut plywood | Compression bit, full depth pass |
| 3D relief carving | Ball nose ¼"–½" |
| Engraving text | 30° or 60° V-bit |
| Spoilboard cleanup | 3-flute ½"–¾" surfacing bit |
| PCB isolation | 1/32"–1/16" ball nose |

---

## Key GitHub Resources

| Repo | URL | Use |
|---|---|---|
| openscad-cnc | `github.com/ruliana/openscad-cnc` | Real Longmill projects with flat-pack exports |
| OpenSCAD-CNC | `github.com/roberchi/OpenSCAD-CNC` | Cutting simulation inside OpenSCAD |
| Design_Into_3D | `github.com/WillAdams/Design_Into_3D` | Parametric box joints, tool databases |
| Round-Anything | `github.com/Irev-Dev/Round-Anything` | Corner rounding library |
| awesome-openscad | `github.com/elasticdotventures/awesome-openscad` | Full curated list of OpenSCAD resources |

---

## Workflow Checklist

- [ ] Design part in OpenSCAD (2D profile or 3D model)
- [ ] Round internal corners (≥ end mill radius)
- [ ] Export DXF (or STL for 3D relief)
- [ ] Open DXF in BobCAD — verify units
- [ ] Set stock dimensions and origin
- [ ] Select bit type and diameter
- [ ] Calculate RPM and feed rate from reference tables
- [ ] Generate roughing toolpath (roughing pass first)
- [ ] Generate finishing toolpath
- [ ] Set safe Z clearance and rapid heights
- [ ] Post-process to G-code for your controller
- [ ] Transfer to CNC and run air-test first
- [ ] Run with material — monitor first pass
