# Skill: Manufacturing Documentation

Package a complete manufacturing deliverable for a KiCad PCB: Gerbers, drill files, assembly drawings, manufacturing notes, and revision history. Generates a CM-ready ZIP file.

**Prerequisite: DRC must pass (0 violations or documented exceptions) before running this skill.**

---

## What It Does

Takes a DRC-cleared board and produces the full manufacturing package: Gerber files for each layer, EXCELLON drill file, IPC-356 netlist, assembly drawing with component orientation, stackup and tolerance notes, and revision history. Packages everything into a ZIP for delivery to the contract manufacturer (CM).

Use this skill when Nicolas asks to:
- Generate manufacturing files for a board
- Create Gerber package
- Prepare files for a CM (JLCPCB, SeeedFusion, PCBWay, etc.)
- Build the assembly drawing
- Write manufacturing notes

---

## What Must Exist Before Running

1. **Schematic**: passing ERC (0 errors)
2. **PCB Layout**: complete placement and routing
3. **DRC**: 0 violations OR documented exceptions approved by Nicolas
4. **BOM**: approved by Nicolas
5. **Board revision**: ECN number or rev letter assigned

---

## Manufacturing Package Contents

| File | Format | Purpose |
|------|--------|---------|
| F.Cu.gbr | Gerber | Top copper layer |
| B.Cu.gbr | Gerber | Bottom copper layer |
| In1.Cu.gbr | Gerber | Internal layer 1 (4-layer) |
| In2.Cu.gbr | Gerber | Internal layer 2 (4-layer) |
| F.Mask.gbr | Gerber | Top solder mask |
| B.Mask.gbr | Gerber | Bottom solder mask |
| F.SilkS.gbr | Gerber | Top silkscreen |
| B.SilkS.gbr | Gerber | Bottom silkscreen |
| Edge.Cuts.gbr | Gerber | Board outline |
| PTH.drl | EXCELLON | Plated through-hole drills |
| NPTH.drl | EXCELLON | Non-plated drills |
| *.drl.map | Drill map | Drill size to filename mapping |
| *.gbr-job | Gerber job | Manufacturing metadata |
| BOM.csv | CSV | Bill of materials |
| assembly.pdf | PDF | Assembly drawing |
| manufacturing_notes.txt | Text | Stackup, tolerances, surface finish |
| revision_history.txt | Text | ECN/changelog |

---

## How It Works

1. **Verify DRC pass** — check drc-validate skill output
2. **Confirm board details** — layers, dimensions, impedance requirements
3. **Generate Gerbers** — via KiCad plot dialog or pcbnew Python export
4. **Generate drill files** — EXCELLON format, plated and non-plated
5. **Generate assembly drawing** — board outline, component refs, polarity/orientation markers
6. **Write manufacturing notes** — stackup, tolerances, surface finish, IPC compliance
7. **Write revision history** — ECN number, date, changes from previous rev
8. **Package ZIP** — compress all files for CM delivery

---

## Workflows

### Generate Full Manufacturing Package

```
1. Verify DRC:
   - Confirm drc-validate skill shows 0 violations OR documented exceptions
   - If DRC not clean: stop and report violations

2. Confirm with Nicolas:
   - Board revision (e.g. rev A, ECN-2024-001)
   - Layer count (2 or 4)
   - Surface finish (HASL, ENIG, OSP)
   - Stackup (if custom, provide layer thicknesses)
   - Impedance requirements (if applicable)
   - CM (which manufacturer)

3. Write manufacturing_script.py:

import pcbnew
import zipfile
import os
import datetime

BOARD_PATH = r"C:\Users\<username>\Desktop\<project>\board.kicad_pcb"
OUT_DIR = r"C:\Users\<username>\Desktop\<project>\manufacturing"
REVISION = "A"
DATE = datetime.date.today().isoformat()

# Convert WSL path if needed
if BOARD_PATH.startswith('/mnt'):
    BOARD_PATH = BOARD_PATH.replace('/mnt/c/', 'C:/')

board = pcbnew.LoadBoard(BOARD_PATH)

# Create output directory
os.makedirs(OUT_DIR, exist_ok=True)

# Plot (Gerber) settings
plot_opts = pcbnew.PLOT_CONTROLLER(board)
plot_params = plot_opts.GetPlotOptions()

plot_params.SetOutputDirectory(OUT_DIR)
plot_params.SetFormat(1)  # Gerber

layers = {
    pcbnew.F_Cu: "F.Cu",
    pcbnew.B_Cu: "B.Cu",
    pcbnew.F_Mask: "F.Mask",
    pcbnew.B_Mask: "B.Mask",
    pcbnew.F_SilkS: "F.SilkS",
    pcbnew.B_SilkS: "B.SilkS",
    pcbnew.Edge_Cuts: "Edge.Cuts",
}

for layer, name in layers.items():
    plot_opts.SetLayer(layer)
    plot_opts.Plot()

# Drill file
excellon_writer = pcbnew.EXCELLON_WRITER(board)
excellon_writer.SetOptions(False, False, None, True, True)
excellon_writer.CreateDrillandMapFilesSet(OUT_DIR, True, True)

# Assembly drawing
# (Generate via plot of F.Fab or create custom PDF)

# Manufacturing notes
notes = f"""MANUFACTURING NOTES — <project> rev {REVISION}
Generated: {DATE}
Tool: nic-pcb-agent manufacturing-docs skill

BOARD SPECIFICATIONS
-------------------
Board size: {board.GetBoardEdgesBoundingBox().GetWidth():.2f}mm x {board.GetBoardEdgesBoundingBox().GetHeight():.2f}mm
Layers: <N>
Thickness: 1.6mm standard
Material: FR-4, Tg >= 150°C (lead-free assembly compatible)

STACKUP (4-layer)
-----------------
Layer 1 (F.Cu):  1 oz copper
Prepreg:         1080 x 2
Layer 2 (In.Cu): 1 oz copper
Core:            0.021" (0.533mm)
Layer 3 (In.Cu): 1 oz copper
Prepreg:         1080 x 2
Layer 4 (B.Cu):   1 oz copper

DESIGN RULES
------------
Min trace/space:  6 mil preferred, 5 mil minimum
Min via drill:    10 mil preferred, 8 mil minimum
Min annular ring: 6 mil preferred, 5 mil minimum
Pad (8 mil drill): 24 mil preferred, 20 mil minimum
Solder mask dam:  5 mil preferred, 4 mil minimum
Silkscreen text:  40 mil preferred, 32 mil minimum

SURFACE FINISH
--------------
<HASL / ENIG / OSP — specify>

IMPEDANCE CONTROLLED LINES (if applicable)
------------------------------------------
Target: 50Ω (differential or single-ended — specify)
Impedance: 52Ω nominal (centers FR-4 tolerance)
Controlled impedance required on: <net names or "N/A">

IPC COMPLIANCE
--------------
IPC-2221A: Generic PCB design standard — COMPLIANT
IPC-6012:   Rigid PCB qualification — COMPLIANT
IPC-7351:   Surface mount land pattern — COMPLIANT

FAB NOTES
--------
- Peelable mask (optional): not required
- Countersink/countersink: specify locations on assembly drawing
- Gold fingers (if applicable): 30 microinches gold minimum

ASSEMBLY NOTES
--------------
- Lead-free assembly profile: 260°C peak, Tg >= 150°C board
- Reflow: no more than 3 passes
- Initial FMD: included in assembly drawing

REVISION HISTORY
----------------
Rev {REVISION} — {DATE} — Initial release
"""

with open(os.path.join(OUT_DIR, "manufacturing_notes.txt"), 'w') as f:
    f.write(notes)

# Create ZIP
zip_path = os.path.join(os.path.dirname(OUT_DIR), f"<project>_rev{REVISION}_manufacturing.zip")
with zipfile.ZipFile(zip_path, 'w', zipfile.ZIP_DEFLATED) as zf:
    for root, dirs, files in os.walk(OUT_DIR):
        for file in files:
            zf.write(os.path.join(root, file), file)

print(f"Manufacturing package: {zip_path}")
print(f"Files: {len(zf.namelist())}")
```

4. Execute:
   "C:\Program Files\KiCad\10.0\bin\python.exe" manufacturing_script.py

5. Verify ZIP contents:
   - List all files, confirm each required type is present
   - Spot-check layer count matches board

6. Report to Nicolas with package location
```

### Assembly Drawing

```
The assembly drawing (assembly.pdf) shows:
- Board outline with dimensions
- All component reference designators
- Pin 1 polarity markers (IC packages: QFP, QFN, SOIC, etc.)
- Connector orientation (USB, JTAG, power barrel)
- Component height restrictions (if any)
- Assembly date, revision, project name

Create via:
1. Export F.Fab (fabrication layer) as Gerber or PDF
2. Overlay component ref designators from silkscreen
3. Add pin 1 markers for all polarized packages
4. Add title block: project name, revision, date, drawn by
5. Save as PDF
```

---

## Common CMs and Their Requirements

| CM | Format | Notes |
|----|--------|-------|
| JLCPCB | Gerber + BOM + centroid CSV | Use JLCPCB plugin for KiCad |
| SeeedFusion | Gerber + BOM | Accepts KiCad native Gerber |
| PCBWay | Gerber + BOM | Has KiCad plugin |
| OSH Park | Gerber (purple) | 2-layer only, no assembly |
| MacroFab | Gerber + BOM + pick-and-place | Full assembly service |

---

## Manufacturing Notes Template

```
MANUFACTURING NOTES — <project> rev <rev>
=========================================

BOARD
-----
Dimensions:    <W>mm × <H>mm
Layers:        <N>
Thickness:      1.6mm
Material:       FR-4, Tg ≥ 150°C
Surface finish: <HASL-LF / ENIG / OSP>

STACKUP (4-layer)
-----------------
F.Cu:   1 oz copper
Prepreg: 1080 × 2
L2:     1 oz copper
Core:   0.021"
L3:     1 oz copper
Prepreg: 1080 × 2
B.Cu:   1 oz copper

DESIGN RULES
------------
Trace/space:    6/6 mil preferred, 5/5 mil minimum
Via drill:      10 mil preferred, 8 mil minimum
Annular ring:   6 mil preferred, 5 mil minimum
Pad (8ml drill): 24 mil preferred, 20 mil minimum
Solder mask:    5 mil dam preferred
Silkscreen:     40 mil text preferred

IMPEDANCE (if applicable)
--------------------------
Target: 50Ω ± 10%
Nets:   <list>
Profiled: Yes/No

IPC STANDARDS
-------------
IPC-2221A: COMPLIANT
IPC-6012:   COMPLIANT
IPC-7351:   COMPLIANT

REVISION HISTORY
----------------
<rev> — <date> — <description>
```

---

## Quick Commands

| Command | Action |
|---------|--------|
| `generate manufacturing package for <project>` | Full CM package |
| `make Gerbers for <board>` | Gerber layers only |
| `assembly drawing for <board>` | Assembly PDF |
| `manufacturing notes for <project>` | Stackup + tolerance notes |
| `package for JLCPCB` | JLCPCB-format package |

---

## Notes

- **DRC pass first** — never generate manufacturing files from a board with unresolved DRC violations
- **Gerber format** — use RS-274X (extended Gerber) for all copper and mask layers; EXCELLON for drills
- **Drill hit count** — confirm plated vs non-plated drill count matches board footprint count
- **ENIG surface finish** — adds ~2µm nickel + 0.05µm gold per pad; affects hole size for press-fit connectors
- **IPC-6012 class** — specify Class 2 (general电子产品) or Class 3 (high-reliability) based on application
- **Gerber job file** — KiCad generates a `.gbrjob` file with manufacturing metadata; include this in the ZIP
