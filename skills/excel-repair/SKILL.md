---
name: "excel-repair"
description: "Diagnose and repair broken Excel xlsx files using raw XML inspection and Python zipfile/xml.etree reconstruction"
---

# excel-repair Skill

## Purpose

Diagnose and repair broken Excel `.xlsx` files by inspecting raw XML inside the ZIP archive and rebuilding damaged sheets programmatically using Python's `zipfile` + `xml.etree.ElementTree`. No third-party libraries (openpyxl/lxml) required — works in bare Python environments.

## When to Use

- An Excel file reports XML errors on open (styles.xml, workbook.xml, worksheet XML)
- Formulas return blank/zero when cells contain valid data
- Column data is misaligned (strings appearing in numeric columns)
- Data validation, named ranges, or sheet properties are removed by Excel's repair
- You need to add/remove/rename columns across an entire workbook programmatically
- The file is locked by Excel (`~$` temp file) and needs a parallel fixed copy

## How It Works

### Step 1 — Extract and Inspect the ZIP

```python
import zipfile, os, shutil
from xml.etree import ElementTree as ET

src = '/path/to/file.xlsx'
work = '/tmp/xlsx_inspect'
shutil.rmtree(work, ignore_errors=True)
os.makedirs(work)

with zipfile.ZipFile(src, 'r') as z:
    z.extractall(work)
```

### Step 2 — Read Shared Strings

```python
NS = 'http://schemas.openxmlformats.org/spreadsheetml/2006/main'
ss_tree = ET.parse(open(f'{work}/xl/sharedStrings.xml', 'rb'))
ss_root = ss_tree.getroot()
strings = []
for si in ss_root.findall(f'{{{NS}}}si'):
    text = ''.join(t.text or '' for t in si.iter(f'{{{NS}}}t'))
    strings.append(text)
```

### Step 3 — Inspect Sheet XML

```python
def sheet_data(path):
    tree = ET.parse(open(path, 'rb'))
    root = tree.getroot()
    rows = {}
    for row in root.findall(f'.//{{{NS}}}row'):
        rn = int(row.get('r'))
        cells = {}
        for c in row.findall(f'{{{NS}}}c'):
            ref = c.get('r')
            col = ''.join(filter(str.isalpha, ref))
            t = c.get('t', 'n')  # n=number, s=shared string, str=inline string
            v_el = c.find(f'{{{NS}}}v')
            f_el = c.find(f'{{{NS}}}f')
            v = v_el.text if v_el is not None else ''
            f = f_el.text if f_el is not None else ''
            val = strings[int(v)] if t == 's' and v else v
            cells[col] = {'v': val, 'f': f}
        rows[rn] = cells
    return rows
```

### Step 4 — Diagnose the Problem

Common failure patterns:
- **QTY column has strings**: Column E has header labels instead of numbers → `AND(E<>"", I<>"")` evaluates to FALSE → J returns blank. Fix: restore numeric values to E.
- **styles.xml line N error**: Malformed XML in style definitions. Fix: strip the damaged style definitions or rebuild from a valid template.
- **Data validation removed**: `<dataValidation>` elements present in sheet XML were stripped by Excel's repair. Fix: re-add validation elements manually.
- **Shared strings mismatch**: Cell type `t="s"` references a string index that doesn't exist → `#REF!` or blank. Fix: rebuild cell values.

### Step 5 — Rebuild the Sheet

When rebuilding a sheet from scratch:
- Use `ET.Element` and `ET.SubElement` to construct XML
- Register namespaces before writing:
  ```python
  ET.register_namespace('', NS)
  ET.register_namespace('r', NS_R)
  ```
- Serialize with declaration: `ET.tostring(root, encoding='unicode')`
- Write as UTF-8 bytes with `<?xml version="1.0" encoding="UTF-8" standalone="yes"?>\n` prefix

### Step 6 — Build Cell Elements

```python
def make_cell(ref, val=None, formula=None, s=None, t=None):
    c = ET.Element('c')
    c.set('r', ref)
    if s is not None: c.set('s', str(s))  # style index
    if t is not None: c.set('t', t)        # type: 'n','s','str'
    if formula is not None:
        f = ET.SubElement(c, 'f')
        f.text = formula
    if val is not None:
        v = ET.SubElement(c, 'v')
        v.text = str(val)
    return c
```

Key style indices (from Hancock_Automated):
- `s=1` : normal text
- `s=2` : section header (bold, blue text)
- `s=5` : input cell (white background)
- `s=6` : formula cell (light blue background)
- `s=7` : subtotal row (bold, medium blue)
- `s=13` : label text (bold)
- `s=14` : user input cell
- `s=16` : description cell
- `s=18` : section header
- `s=19` : numeric percentage
- `s=27` : dropdown input cell
- `s=30` : supplier name cell
- `s=33/34` : price cell (right-aligned number)

### Step 7 — Repackage as xlsx

```python
dst = '/path/to/fixed.xlsx'
with zipfile.ZipFile(dst, 'w', zipfile.ZIP_DEFLATED) as zout:
    for dirpath, dirnames, filenames in os.walk(work):
        for filename in filenames:
            filepath = os.path.join(dirpath, filename)
            arcname = os.path.relpath(filepath, work)
            zout.write(filepath, arcname)
```

## Key Formula Patterns

### Selected Cost (auto-supplier lookup)
```excel
=IF(C8="Richardsons",F8,IF(C8="Winnelson",G8,IF(C8="Lowes",H8,0)))
```

### Selected Cost with Manual Override (L overrides K)
```excel
=IF(L8<>"",L8,IF(C8="Richardsons",F8,IF(C8="Winnelson",G8,IF(C8="Lowes",H8,0))))
```

### Total (zero on empty, no errors)
```excel
=IF(OR(E8="",I8=0),0,ROUND(E8*I8,2))
```

### Subtotal
```excel
=SUM(J8:J41)
```

### Grand Total Chain
```excel
J73 = J42+J51+J57+J62+J67+J71   ' Direct costs
J74 = ROUND(J73*I74,2)           ' Markup
J75 = J73+J74                     ' Subtotal
J76 = ROUND(J75*I76,2)           ' Sales tax
J77 = J75+J76                     ' Grand total
```

## Column Architecture Reference

```
A = Line # / label
B = Description
C = Supplier (dropdown)
D = Unit
E = QTY (numeric)
F = Supplier A price
G = Supplier B price
H = Supplier C price
I = Selected Cost  =IF(L<>"",L,K)   ← manual override L, else auto K
J = Total          =IF(OR(E="",I=0),0,ROUND(E*I,2))
K = BEFORE (auto price, pre-manual)
L = AFTER  (manual override — fill this to override K)
```

## Common Repairs

| Problem | Symptom | Fix |
|---------|---------|-----|
| Column misalignment | E col has text strings | Restore numeric QTY values |
| Formulas returning blank | AND(E<>"",I<>"") = FALSE | Fix E column values or I formula |
| Division by zero | `#DIV/0!` | Wrap with `IF(denominator=0,0,...)` |
| Empty-safe totals | Errors when cells blank | `IFERROR(...,0)` or `IF(OR(E="",...),0,...)` |
| Percentage stored as text | `"6.0%"` not `0.06` | Store as decimal number |
| XML entity not escaped | `<>` in formula XML | Replace with `&lt;&gt;` |
| Duplicate row numbers | Two `<row r="N">` elements | Merge into one row element |
| Circular reference | K=K formula | I=K formula; K=supplier lookup (no K reference) |
| Styles missing | Format lost | Copy `<numFmts>` and `<cellXfs>` from valid file |

## Notes

- `openpyxl` not available in bare WSL2 Python — use `zipfile` + `xml.etree.ElementTree` only
- Excel's repair log (`error197240_03.xml`-style) describes the *pre-repair* state — always inspect the *current* file's XML to see what actually remains
- Data validation (`<dataValidation>`) survives Excel's repair if valid — inspect before assuming it's gone
- `~$` lock files appear when Excel has the file open — save as `_fixed.xlsx` instead
- Shared string indices are 0-indexed in XML (`t="s"` with `v="5"` = `strings[5]`)
