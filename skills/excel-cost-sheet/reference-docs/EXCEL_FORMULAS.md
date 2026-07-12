# Excel Formulas Reference

## By Category

---

### 📊 SUM & AGGREGATION

| Formula | What it does | Example |
|---|---|---|
| `=SUM(range)` | Sum a range | `=SUM(B5:B25)` |
| `=SUMIF(criteria_range,criteria,sum_range)` | Sum with one condition | `=SUMIF(D5:D25,"Labor",E5:E25)` |
| `=SUMIFS(sum_range,criteria1_range,criteria1,...)` | Sum with multiple conditions | `=SUMIFS(E:E,D:D,"Labor",F:F,">0")` |
| `=SUBTOTAL(function_num,range)` | Sum filtered data (ignores hidden rows) | `=SUBTOTAL(9,B5:B25)` |
| `=AGGREGATE(function,options,array,k)` | Advanced ignore errors/hidden | `=AGGREGATE(9,6,B5:B25)` |
| `=SUMPRODUCT(array1,array2)` | Sum of products — weighted averages | `=SUMPRODUCT(B5:B25,C5:C25)` |
| `=SUMPRODUCT((range1=crit1)*(range2=crit2)*sum_range)` | Multi-condition sum | `=SUMPRODUCT((D5:D25="Labor")*(F5:F25>0)*(E5:E25))` |

---

### 🔍 LOOKUPS

| Formula | What it does | Example |
|---|---|---|
| `=VLOOKUP(lookup_value,table,col_index,FALSE)` | Lookup in table (leftmost column) | `=VLOOKUP(A5,RateTable!A:C,3,FALSE)` |
| `=INDEX(return_range,MATCH(lookup,lookup_range,0))` | INDEX/MATCH — better than VLOOKUP | `=INDEX(RateTable!C:C,MATCH(A5,RateTable!A:A,0))` |
| `=XLOOKUP(lookup,lookup_array,return_array)` | Modern lookup (Excel 365) | `=XLOOKUP(A5,RateTable!A:A,RateTable!C:C)` |
| `=INDIRECT(address)` | Get value from text address | `=INDIRECT("B"&ROW())` |
| `=CHOOSE(index,val1,val2,...)` | Pick from list | `=CHOOSE(B5,"Labor","Equipment","Material")` |

**VLOOKUP vs INDEX/MATCH:**
```
VLOOKUP — must lookup from leftmost column
INDEX/MATCH — can lookup from any column in either direction
```

---

### 🔢 MATH & ROUNDING

| Formula | What it does | Example |
|---|---|---|
| `=ROUND(number,decimals)` | Round to N decimals | `=ROUND(B5*C5,2)` |
| `=ROUNDUP(number,significance)` | Always round up | `=ROUNDUP(B5/2,0)` |
| `=ROUNDDOWN(number,significance)` | Always round down | `=ROUNDDOWN(B5,0)` |
| `=CEILING(number,significance)` | Round to nearest increment up | `=CEILING(B5,5)` → rounds to nearest $5 |
| `=FLOOR(number,significance)` | Round to nearest increment down | `=FLOOR(B5,0.25)` |
| `=MOD(number,divisor)` | Remainder after division | `=MOD(B5,7)` — days of week |
| `=ABS(number)` | Absolute value | `=ABS(B5-C5)` |
| `=POWER(base,exponent)` | Power | `=POWER(B5,2)` |
| `=SQRT(number)` | Square root | `=SQRT(B5)` |

---

### ❓ CONDITIONAL LOGIC

| Formula | What it does | Example |
|---|---|---|
| `=IF(condition,value_if_true,value_if_false)` | Basic IF | `=IF(B5>100,"OK","LOW")` |
| `=IFERROR(value,value_if_error)` | Handle errors | `=IFERROR(VLOOKUP(...),"NOT FOUND")` |
| `=IFNA(value,value_if_na)` | Handle #N/A only | `=IFERROR(INDEX(...MATCH(...)),0)` |
| `=IFS(condition1,val1,condition2,val2,...)` | Multiple conditions | `=IFS(B5>=100,"High",B5>=50,"Med","Low")` |
| `=SWITCH(expression,val1,result1,val2,result2,...,default)` | Switch cases | `=SWITCH(B5,1,"Low",2,"Med","High")` |

**Nested IF (avoid if possible — use IFS instead):**
```
=IF(B5>=100,"Excellent",IF(B5>=75,"Good",IF(B5>=50,"Fair","Poor")))
```

**Tiered rate example (equipment pricing):**
```
=IF(B5>=500, B5*0.80, IF(B5>=200, B5*0.90, B5*0.95))
  → 500+ units: 20% discount
  → 200-499: 10% discount
  → <200: 5% discount
```

---

### 📅 DATE & TIME

| Formula | What it does | Example |
|---|---|---|
| `=TODAY()` | Today's date | `=TODAY()` |
| `=NOW()` | Current date + time | `=NOW()` |
| `=DATE(year,month,day)` | Build a date | `=DATE(2026,7,10)` |
| `=DATEDIF(start,end,"unit")` | Date difference | `=DATEDIF(A5,B5,"D")` — days between |
| `=WORKDAY(start,days,holidays)` | Add workdays | `=WORKDAY(B5,14)` — 2 weeks out |
| `=EOMONTH(start,months)` | End of month N months out | `=EOMONTH(TODAY(),1)` — end of next month |
| `=TEXT(date,"format")` | Format date as text | `=TEXT(B5,"MMM-YYYY")` |

---

### 🔢 STATISTICS

| Formula | What it does | Example |
|---|---|---|
| `=AVERAGE(range)` | Average | `=AVERAGE(B5:B25)` |
| `=AVERAGEIF(range,criteria)` | Conditional average | `=AVERAGEIF(D5:D25,"Labor",E5:E25)` |
| `=MEDIAN(range)` | Middle value | `=MEDIAN(B5:B25)` |
| `=MAX(range)` | Maximum | `=MAX(E5:E25)` |
| `=MIN(range)` | Minimum | `=MIN(E5:E25)` |
| `=COUNT(range)` | Count numeric cells | `=COUNT(B5:B25)` |
| `=COUNTA(range)` | Count non-empty cells | `=COUNTA(B5:B25)` |
| `=COUNTIF(range,criteria)` | Count with condition | `=COUNTIF(D5:D25,"Labor")` |
| `=COUNTIFS(range1,crit1,range2,crit2)` | Count with multiple conditions | `=COUNTIFS(D5:D25,"Equipment",F5:F25,">0")` |
| `=STDEV(range)` | Standard deviation | `=STDEV(E5:E25)` |
| `=LARGE(range,k)` | Kth largest | `=LARGE(E5:E25,1)` — highest value |
| `=SMALL(range,k)` | Kth smallest | `=SMALL(E5:E25,1)` — lowest value |

---

### 📝 TEXT

| Formula | What it does | Example |
|---|---|---|
| `=CONCATENATE(text1,text2)` or `=text1&text2` | Join text | `="Estimate #"&A5` |
| `=TRIM(text)` | Remove extra spaces | `=TRIM(B5)` |
| `=UPPER(text)` / `=LOWER(text)` | Change case | `=UPPER(B5)` |
| `=LEFT(text,num_chars)` | Extract from left | `=LEFT(A5,3)` |
| `=RIGHT(text,num_chars)` | Extract from right | `=RIGHT(A5,4)` |
| `=MID(text,start,num_chars)` | Extract middle | `=MID(A5,5,3)` |
| `=LEN(text)` | Character count | `=LEN(B5)` |
| `=TEXT(value,"format")` | Format as text | `=TEXT(B5,"$#,##0.00")` |
| `=VALUE(text)` | Convert text to number | `=VALUE(B5)` |
| `=SUBSTITUTE(text,old,new)` | Replace text | `=SUBSTITUTE(A5," ","_")` |

---

### 🧮 CONTRACTOR-SPECIFIC FORMULAS

#### Loaded Labor Rate
```
=(BaseRate*(1+FringePct))*(1+FICA_Pct+WorkersCompPct)
Example: =(35*(1+0.12))*(1+0.0765+0.05)
Result: $43.63/hr loaded
```

#### Markup Calculation
```
Total Sell Price = DirectCost / (1 - MarkupPct)
Example: =10000 / (1 - 0.25)  → $13,333.33
```

#### Weighted Average Rate
```
= SUMPRODUCT(Hours, Rates) / SUM(Hours)
=SUMPRODUCT(C5:C25,D5:D25)/SUM(C5:C25)
```

#### Estimate Total with Multiple Categories
```
=SUMIF(Category,"Labor",Total) + SUMIF(Category,"Equipment",Total)
  + SUMIF(Category,"Materials",Total) + SUMIF(Category,"Dump Fees",Total)
  + SUMIF(Category,"Permits",Total) + SUMIF(Category,"Subcontractor",Total)
```

#### Grand Total with Markup
```
=SUM(B5:B10)*(1+$D$1)
  where D1 = markup % (e.g., 0.25 for 25%)
```

#### Bid Validity Days Remaining
```
=IF(ISBLANK(B5),"",D5-B5)
  where B5 = Quote Date, D5 = Expiration Date
```

---

### ⚠️ FORMULA RULES FOR COST SHEETS

1. **Never hardcode numbers in formulas** — reference input cells
2. **Use `$` on cell references** to lock rates/assumptions: `$D$1`
3. **Use IFERROR** on all VLOOKUP/INDEX-MATCH to handle missing rates
4. **Use ROUND** on all currency calculations: `=ROUND(B5*C5,2)`
5. **Use named ranges** for key inputs — makes sheets auditable

---

### 📍 Named Ranges (Critical for Cost Sheets)

Create via: `Formulas → Name Manager → New`

| Name | Refers to | Use |
|---|---|---|
| `MarkupPct` | `=Sheet!$D$1` | Centralized markup % |
| `OH_Pct` | `=Sheet!$D$2` | Overhead % |
| `Profit_Pct` | `=Sheet!$D$3` | Profit % |
| `LaborRate` | `=Sheet!$D$10` | Base labor rate |
| `DumpFeeTon` | `=Sheet!$D$15` | Dump fee per ton |
| `RateTable` | `=Sheet2!$A$5:$D$50` | Equipment rate lookup table |

Then formulas read:
```
= DirectCost * (1 + MarkupPct)
= SUMPRODUCT(Hours, Rates) / SUM(Hours)   ← no cell references!
```

---

### 🧩 ARRAY FORMULAS (Ctrl+Shift+Enter)

Some formulas must be entered as array formulas in older Excel:

```
{=SUM(IF(ISNUMBER(SEARCH("Labor",Category)),Total,0))}
```

In Excel 365, most array operations work without Ctrl+Shift+Enter.

---

## GitHub Topic Pages
- `github.com/topics/excel-formulas`
- `github.com/topics/excel-templates`
- `github.com/topics/spreadsheet`
- `github.com/topics/vba-excel`

---

*v1.0 — 2026-07-10*
