# Skill: KiCad Board Inspection

Inspect KiCad board files: component counts, layer counts, net lists, connectivity, SVG layer exports, and comparison between two board revisions.

---

## What It Does

Uses `kicad_tools.py` to extract structural information from a KiCad PCB file without opening the GUI. Reads board metadata, generates layer-by-layer SVG exports, and compares two board files to identify what changed between revisions.

Use this skill when Nicolas asks to:
- Inspect a board file
- Check what's on a board without opening KiCad
- Export SVG layers for documentation or review
- Compare two board revisions
- Get a quick summary of a board's state

---

## How It Works

1. **Confirm board path** — Windows path, convert WSL paths if needed
2. **Run inspection** — `kicad_tools.py inspect` for structural data
3. **Export layers** — `kicad_tools.py export` for SVG layer images
4. **Compare if needed** — `kicad_tools.py compare` for revision diff
5. **Present structured summary**

---

## Workflows

### Inspect Board File

```
1. Confirm board path (Windows format: C:\Users\...\board.kicad_pcb)
2. Convert WSL path if needed: path.replace('/mnt/c/', 'C:/')
3. Run:
   python kicad_tools.py inspect <board_path.kicad_pcb>

4. Parse output for:
   - Layer count and names
   - Footprint count (by package type)
   - Net count
   - Track count
   - Zone count
   - Board dimensions
   - DRC last-run date and violation count

5. Present summary:
```

```markdown
## Board Inspection — <board_name>

**File:** `<path>`
**Layers:** F.Cu, B.Cu, F.Silkscreen, B.Silkscreen, F.Mask, B.Mask, Edge.Cuts [+ any additional]
**Dimensions:** XXXmm × YYYmm
**Footprints:** N total
  - QFP/TQFP: N
  - QFN/SON: N
  - SOIC: N
  - Passives (R, C, L): N
  - Connectors: N
  - Mounting holes: N
**Nets:** N
**Tracks:** N
**Zones:** N
**Last DRC:** N violations
```

### Export SVG Layers

```
1. Confirm board path and output directory
2. Run:
   python kicad_tools.py export <board.kicad_pcb> --output ./layers

3. SVG files produced:
   - layers/F_Cu.svg       — Top copper
   - layers/B_Cu.svg       — Bottom copper
   - layers/F_SilkS.svg    — Top silkscreen
   - layers/B_SilkS.svg    — Bottom silkscreen
   - layers/F_Mask.svg     — Top solder mask
   - layers/B_Mask.svg     — Bottom solder mask
   - layers/Edge_Cuts.svg  — Board outline
   - layers/In1_Cu.svg     — Internal layer 1 (4-layer)
   - layers/In2_Cu.svg     — Internal layer 2 (4-layer)

4. Confirm output:
   ls ./layers/*.svg | wc -l
   Report: "N layer SVGs exported to ./layers/"
```

### Compare Two Boards

```
1. Confirm both board paths (older revision, newer revision)
2. Run:
   python kicad_tools.py compare <board_old.kicad_pcb> <board_new.kicad_pcb>

3. Parse output for:
   - Added/removed footprints
   - Added/removed nets
   - Changed track geometries
   - Changed zone boundaries
   - Layer count differences

4. Present:
```

```markdown
## Board Comparison — <project>

**Old:** `<board_v1.kicad_pcb>`
**New:** `<board_v2.kicad_pcb>`

### Changes

| Change Type | Count | Details |
|-------------|-------|---------|
| Footprints added | N | [list refs] |
| Footprints removed | N | [list refs] |
| Nets added | N | [net names] |
| Tracks changed | N | [net names] |
| Zones changed | N | [net names] |
| Layers added/removed | N | [which layers] |

### Verdict
[High-level summary of what changed between revisions]
```
---

## kicad_tools.py Reference

### inspect

```bash
python kicad_tools.py inspect <board.kicad_pcb>
```

Returns JSON or text with:
- `layer_count`
- `footprint_count` (by package type)
- `net_count`
- `track_count`
- `via_count`
- `zone_count`
- `board_dimensions` (width × height in mm)
- `last_drc_violations`

### export

```bash
python kicad_tools.py export <board.kicad_pcb> --output ./layers
```

Exports each layer as a separate SVG file. Uses `pcbnew` Python API with `SetSvgPrecision(4)`.

### compare

```bash
python kicad_tools.py compare <board1.kicad_pcb> <board2.kicad_pcb>
```

Reports structural diff: added/removed/changed footprints, nets, tracks, zones, and layers.

---

## Quick Commands

| Command | Action |
|---------|--------|
| `inspect <board>` | Full structural inspection of a board |
| `export layers from <board>` | SVG layer export |
| `compare <board1> and <board2>` | Diff two board revisions |
| `what's on <board>` | Quick component/net summary |
| `board dimensions <board>` | Just the board size |

---

## Notes

- **Read-only operation** — inspection does not modify any files
- **SVG export is for documentation** — it is not the same as Gerber output for manufacturing
- **SetSvgPrecision(4)** — the argument is a single integer, not a tuple. Common mistake: `SetSvgPrecision(4, 4)` will fail
- **Path convention**: always convert WSL paths to Windows paths before passing to KiCad Python on Windows
