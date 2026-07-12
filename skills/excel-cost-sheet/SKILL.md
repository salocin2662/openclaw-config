---
name: "excel-cost-sheet"
description: "Build Excel cost-out sheets with formulas, VBA macros, and structured cost breakdowns for excavation and construction."
---

# excel-cost-sheet Skill

## Purpose

Build professional Excel cost-out sheets, job estimates, and bids for excavation and field service work. Covers Excel formula construction, VBA macro building, and structured cost sheet architecture.

## When to Use

- Nicolas wants to build or improve a cost estimate / bid sheet in Excel
- Needs Excel formulas written (SUMPRODUCT, INDEX/MATCH, nested IF, etc.)
- Needs a VBA macro written for Excel automation
- Wants a structured cost-out sheet template for a specific job type
- Needs help with markup, overhead, profit margin calculations

## Files to Load

1. `https://github.com/salocin2662/excel-cost-sheet/blob/main/MASTER_PROMPT.md`
2. `https://github.com/salocin2662/excel-cost-sheet/blob/main/reference-docs/EXCEL_FORMULAS.md`
3. `https://github.com/salocin2662/excel-cost-sheet/blob/main/reference-docs/VBA_MACROS.md`
4. `https://github.com/salocin2662/excel-cost-sheet/blob/main/macros/EXCAVATION_COSTOUT.vba`
5. `https://github.com/salocin2662/excel-cost-sheet/blob/main/templates/COSTOUT_BUILDER.md`

## Quick Decision Tree

```
What does Nicolas need?
├── Write an Excel formula → Load EXCEL_FORMULAS.md
├── Write a VBA macro → Load VBA_MACROS.md
├── Build a cost-out sheet → Use COSTOUT_BUILDER.md
├── Generate Excel template → Run: python3 scripts/build_costout_template.py
└── Markup/overhead/profit math → Sell = Cost / (1 − OH% − Profit%)
```

## Cost-Out Architecture

```
Cover → Recap → Labor → Equipment → Materials → Dump Fees → Subcontractors → Permits
```

## Standard Contractor Math

```
Loaded Labor Rate = Base × (1 + Fringe%) × (1 + FICA% + WorkersComp%)
Direct Costs = Labor + Equipment + Materials + Dump + Subs + Permits
Grand Total = Direct Costs × (1 + Markup%)
```

## Key GitHub Repos

| Repo | What to Study |
|---|---|
| `OsSyLab/Excel-VBA-Macros` | 20 ready-to-use macros with before/after docs |
| `Lawrence-Gong/VBA-Automation-Toolkit` | Enterprise-scale VBA patterns |
| `Rohanborse0253/inventory-monitoring-excel-vba` | Full inventory/PO system with alerts |
| `1102tools/federal-contracting-skills` | IGCE skill architecture (best practice) |

## Hard Rules

1. **Never hardcode totals** — every total cell must be a formula
2. **Always separate direct costs from markup**
3. **Always include Assumptions & Exclusions**
4. **Use named ranges** for key inputs (labor rate, markup %, etc.)

## Skill Version

v1.0 — 2026-07-10 — Initial release
