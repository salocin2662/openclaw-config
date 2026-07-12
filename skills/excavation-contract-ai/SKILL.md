---
name: "excavation-contract-ai"
description: "AI-powered contract review, job estimation, and cost analysis for excavation & grading contractors in Kentucky."
---

# excavation-contract-ai Skill

## Purpose

Enable AI contract review, job estimation, and cost analysis for excavation and grading contractors — with a focus on Kentucky law and South Central Kentucky market conditions.

## When to Use

- User provides contract text for review
- User describes a job and wants an estimate built
- User provides actual vs. estimated job costs for profitability analysis
- User asks about excavation/grading contract terms or Kentucky contractor law

## How to Use

### Files to Load

Load these files in order for full context:

1. `https://github.com/salocin2662/excavation-contract-ai/blob/main/MASTER_PROMPT.md`
2. `https://github.com/salocin2662/excavation-contract-ai/blob/main/contracts/CONTRACT_REVIEW_PROMPT.md`
3. `https://github.com/salocin2662/excavation-contract-ai/blob/main/reference-docs/KENTUCKY_FAIRNESS_IN_CONSTRUCTION_ACT.md`
4. `https://github.com/salocin2662/excavation-contract-ai/blob/main/templates/MATERIAL_COST_REFERENCE.md`

### Workflow

#### For Contract Review:
1. Load `MASTER_PROMPT.md` and `CONTRACT_REVIEW_PROMPT.md`
2. Apply the Kentucky law flagging steps FIRST (KRS 371.400–371.415)
3. Then apply general red-flag analysis
4. Rate each clause: 🔴 UNACCEPTABLE / 🟠 HIGH RISK / 🟡 CAUTION / 🟢 STANDARD
5. End with risk score and recommended action

#### For Job Estimates:
1. Load `MASTER_PROMPT.md` and `JOB_ESTIMATE_TEMPLATE.md`
2. Request missing job details from user using the template as a guide
3. Build line-item estimate using material cost reference
4. Flag risk factors for unknown conditions
5. Recommend payment terms and contract language

#### For Cost Analysis:
1. Load `MASTER_PROMPT.md` and `JOB_COSTING_SHEET.md`
2. Calculate profitability by category
3. Flag variances > ±15%
4. Identify patterns across similar jobs
5. Recommend estimate improvements

### Kentucky Law Hard Stops (always flag)

- ❌ Any waiver of litigation rights (KRS 371.405(2)(a))
- ❌ Any full waiver of mechanic's lien rights (KRS 371.405(2)(b))
- ❌ Any "no damages for delay" clause (KRS 371.405(2)(c))
- ❌ Retainage exceeding 10%/5%/200% limits (KRS 371.410)
- 🟡 Missing 30-day payment timeline (owner→contractor) (KRS 371.405(5))
- 🟡 Missing 15-day payment timeline (GC→sub) (KRS 371.405(8))
- 🟡 No interest rate on late payments (KY default 12%/yr applies)

### Reference Business

**Southern Bluegrass Excavation & Contracting LLC**
- Glasgow, KY | Barren, Warren, Hart, Adair, Metcalfe Counties
- Services: Septic, grading, driveways, drains, foundation prep, lot clearing
- License: Excavation — Bowling Green License Board (Active, Expires Aug 29, 2026)
- USDOT: 3833423 | 1 power unit | 1 driver | 0 OOS violations
- Rating: 5.0⭐ Google (23 reviews)

### Material Cost Reference (South Central KY — 2026)

| Material | Unit | Cost |
|---|---|---|
| Crusher run ABC | Per ton | $28–$38 |
| Class II base | Per ton | $30–$42 |
| 57/68 stone | Per ton | $32–$45 |
| Conventional septic tank (1000 gal) | Each | $550–$900 |
| PVC SDR-35 (4") | Per LF | $1.50–$3.00 |
| Operator labor | Per hour | $35–$55 |
| Excavator rental (standard) | Per day | $250–$450 |
| Bobcat/skid steer | Per day | $150–$275 |

### Output Format Standards

- Use emoji ratings: 🔴 🟠 🟡 🟢
- Include KRS citations for all KY law flags
- End every contract review with: RISK CLASSIFICATION + MUST FIX list
- End every estimate with: ASSUMPTIONS & EXCLUSIONS + PAYMENT TERMS
- End every cost analysis with: PROFITABILITY SCORE + PATTERN WARNINGS + RECOMMENDATIONS

## Skill Version

v1.0 — 2026-07-07 — Initial release
