# Skill: Bill of Materials Generation

Generate a complete Bill of Materials (BOM) from a KiCad schematic or PCB file. Includes reference designators, values, packages, quantities, and Digikey/Mouser part numbers.

---

## What It Does

Extracts the component list from a KiCad schematic (`*.kicad_sch`) or PCB file (`*.kicad_pcb`) and formats it as a structured BOM with distributor part numbers.

Use this skill when Nicolas asks to:
- Generate a BOM for a project
- Find Digikey part numbers for a schematic
- Find Mouser part numbers for a project
- Compare distributors for the same components
- Export BOM as CSV or PDF

---

## How It Works

1. **Confirm source file** — schematic or PCB path
2. **Extract component list** — from KiCad netlist or direct PCB parse
3. **Group by value/package** — consolidate identical components into quantity counts
4. **Look up distributor PNs** — Digikey and Mouser search (via web or API if configured)
5. **Format BOM** — CSV with all required fields
6. **Present to Nicolas** — review and approval before use for procurement

---

## BOM Fields

| Field | Description | Required |
|-------|-------------|----------|
| Ref | Reference designators (e.g. C1, C2, R1) | Yes |
| Qty | Quantity of identical parts | Yes |
| Value | Component value (e.g. 100nF, 10k) | Yes |
| Package | Footprint package (e.g. 0402, QFP-64) | Yes |
| Part | Generic part description | Yes |
| Digikey PN | Digikey part number | Preferred |
| Mouser PN | Mouser part number | Preferred |
| Unit Cost | Estimated unit cost (USD) | Optional |
| Total Cost | Qty × Unit Cost | Optional |
| Notes | Any special sourcing notes | Optional |

---

## Workflows

### Generate BOM from Schematic

```
1. Confirm schematic path
2. Export netlist from KiCad:
   kicad-cli sch export nets <schematic>.kicad_sch --format xml
   Or use kicad-sch-api:
   ksa.export_netlist(sch_path, format='xml', output=netlist_path)

3. Write bom_generator.py:

import xml.etree.ElementTree as ET
import csv
import os

SCH_PATH = r"C:\Users\<username>\Desktop\<project>\<project>.kicad_sch"
NETLIST_PATH = SCH_PATH.replace('.kicad_sch', '_netlist.xml')
BOM_OUT = SCH_PATH.replace('.kicad_sch', '_BOM.csv')

# Parse KiCad netlist XML
tree = ET.parse(NETLIST_PATH)
components = {}

for comp in tree.findall(".//comp"):
    ref = comp.get("ref")
    value = comp.findtext("value")
    footprint = comp.findtext("footprint")

    key = (value, footprint)
    if key not in components:
        components[key] = {"refs": [], "value": value, "footprint": footprint}
    components[key]["refs"].append(ref)

# Write CSV
with open(BOM_OUT, 'w', newline='') as f:
    writer = csv.writer(f)
    writer.writerow(['Ref', 'Qty', 'Value', 'Package', 'Digikey PN', 'Mouser PN', 'Unit Cost', 'Total Cost', 'Notes'])
    for (value, footprint), data in sorted(components.items()):
        refs = ', '.join(data['refs'])
        qty = len(data['refs'])
        writer.writerow([refs, qty, value, footprint, '', '', '', '', ''])

print(f"BOM written: {BOM_OUT}")
print(f"Total unique parts: {len(components)}")
print(f"Total components: {sum(d['qty'] for d in components.values())}")
```

4. Execute:
   "C:\Program Files\KiCad\10.0\bin\python.exe" bom_generator.py

5. Research distributor PNs:
   - For each unique value/package row, search Digikey and Mouser
   - Prefer: in-stock, reel/tray packaging for SMT, shortest lead time
   - Fill in Digikey PN and Mouser PN columns

6. Present BOM to Nicolas for review
```

### Research Part Numbers

```
For each unique component (value + package):
1. Search Digikey: https://www.digikey.com — search "<value> <package>" (e.g. "100nF 0402")
2. Search Mouser: https://www.mouser.com — same search
3. Apply filters:
   - In stock preferred
   - Ceramic capacitors: prefer X7R/X5R dielectric for decoupling
   - Resistors: 1% tolerance for precision, 5% for general
   - ICs: confirm exact package and temperature grade
4. Record: Digikey PN, Mouser PN, unit cost, stock status
```

### Export Final BOM

```
1. After Nicolas reviews and approves:
2. Save as CSV: <project>_BOM_v<rev>.csv
3. Save as PDF (optional): format as table for CM submission
4. Archive previous revision
```

---

## BOM Template (CSV)

```csv
Ref,Qty,Value,Package,Digikey PN,Mouser PN,Unit Cost,Total Cost,Notes
C1 C2 C3 C4,4,100nF,0402,490-8046-1-ND,80-C0402C104K4R,0.01,0.04,X7R
R1 R2,2,10k,0402,RMCF0402FT10K0,0.004,0.008,1%
U1,1,STM32F405RG,QFP-64,497-STM32F405RGT6-ND,,3.50,3.50,LQFP-64
```

---

## Distributor Search Notes

### Capacitors (Decoupling)
- **100nF / 0402**: Most common. Always in stock everywhere.
- **10µF / 0805**: Bulk decoupling near power entry. Check voltage rating (≥16V).
- **1µF / 0402**: Additional decoupling for high-speed lines.

### Resistors
- **10k / 0402**: Pull-ups/pull-downs. Always in stock.
- **33R / 0402**: Series termination for high-speed signals. Check power rating.

### ICs
- Always verify exact part number suffix:
  - STM32F405RGT6 vs STM32F405RGT6TR (tray vs reel)
  - Temperature grade: commercial (0–70°C) vs industrial (–40–85°C)
  - Confirm pin 1 orientation and package type

---

## Quick Commands

| Command | Action |
|---------|--------|
| `generate BOM for <project>` | Full BOM from schematic |
| `BOM for <schematic>` | BOM with ref/Qty/Value/Package |
| `find Digikey PN for <value> <package>` | Single component lookup |
| `export BOM as CSV` | Generate CSV format |
| `check stock on <PN>` | Check distributor stock |

---

## Notes

- **BOM is procurement, not manufacturing** — BOM is for ordering parts; assembly drawings are for the CM
- **Verify PN before procurement** — wrong PN means wrong part. Always confirm package, voltage rating, and temperature grade
- **Quantity grouping** — group identical parts (same value + package) into single rows with consolidated refs and qty
- **Nickel-plated components** — avoid pure tin (RoHS risk: tin whiskers). Specify Matte Tin or Ni/Au finish
- **Date codes** — for long-running production, specify date code restrictions with CM
