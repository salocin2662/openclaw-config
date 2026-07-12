# VBA Macro Reference

## Running Macros

1. **Open VBA Editor:** `Alt + F11`
2. **Run a macro:** `Alt + F8` → Select macro → Run
3. **Run from button:** Developer → Insert → Button → Assign macro
4. **Edit a macro:** `Alt + F11` → Find macro in Project Explorer → Edit

## Macro Structure

```vba
Sub MacroName()
    ' Declarations
    Dim ws As Worksheet
    Dim lastRow As Long
    
    ' Turn off screen updating (speed)
    Application.ScreenUpdating = False
    Application.Calculation = xlCalculationManual
    
    ' Main code here
    
    ' Restore settings
    Application.ScreenUpdating = True
    Application.Calculation = xlCalculationAutomatic
End Sub
```

---

## Common Patterns

### Find Last Row
```vba
lastRow = Cells(Rows.Count, "A").End(xlUp).Row
lastRow = ws.Cells(ws.Rows.Count, "A").End(xlUp).Row
```

### Loop Through Rows
```vba
For i = 5 To lastRow
    If Cells(i, 4).Value > 0 Then
        Cells(i, 6).Value = Cells(i, 3).Value * Cells(i, 4).Value
    End If
Next i
```

### Loop with Condition
```vba
For i = 5 To lastRow
    If Cells(i, 4).Value <> "" Then
        Cells(i, 6).Value = Cells(i, 3).Value * Cells(i, 4).Value
    End If
Next i
```

### Copy / Paste
```vba
' Copy range
Range("A1:D10").Copy Destination:=Sheets("Sheet2").Range("A1")

' Copy values only (faster)
ws.Range("A1:D10").Value = ws.Range("B1:E10").Value
```

### Clear Cells
```vba
Range("A5:C25").ClearContents   ' Clear values
Range("A5:C25").Clear           ' Clear everything
```

---

## Macro Library

### Macro 1: Auto-Calculate Line Totals
**Use:** Calculates Total = Qty × Unit Cost for all rows
```vba
Sub CalculateLineTotals()
    Dim ws As Worksheet
    Dim lastRow As Long
    Dim i As Long
    
    Set ws = ThisWorkbook.Sheets("Labor")
    lastRow = ws.Cells(ws.Rows.Count, "A").End(xlUp).Row
    
    Application.ScreenUpdating = False
    
    For i = 5 To lastRow
        If ws.Cells(i, 3).Value <> "" And ws.Cells(i, 4).Value <> "" Then
            ws.Cells(i, 5).Value = ws.Cells(i, 3).Value * ws.Cells(i, 4).Value
        End If
    Next i
    
    Application.ScreenUpdating = True
    MsgBox "Line totals calculated!", vbInformation
End Sub
```
**Module:** Standard Module | **Shortcut:** `Ctrl+Shift+T`

---

### Macro 2: Export Sheet to PDF
**Use:** Exports the active sheet to PDF
```vba
Sub ExportToPDF()
    Dim ws As Worksheet
    Dim filePath As String
    Dim fileName As String
    
    Set ws = ActiveSheet
    
    ' Build filename from cells
    fileName = "Estimate_" & Range("B2").Value & "_" & Format(Date, "YYYYMMDD") & ".pdf"
    filePath = Application.GetSaveAsFilename( _
        InitialFileName:=fileName, _
        fileFilter:="PDF Files (*.pdf), *.pdf")
    
    If filePath <> "False" Then
        ws.ExportAsFixedFormat Type:=xlTypePDF, _
            Filename:=filePath, _
            Quality:=xlQualityStandard, _
            IncludeDocProperties:=True, _
            IgnorePrintAreas:=False, _
            OpenAfterPublish:=True
        MsgBox "PDF saved: " & filePath, vbInformation
    End If
End Sub
```
**Module:** Standard Module | **Trigger:** Button on sheet

---

### Macro 3: Auto-Fill Formulas Down a Column
**Use:** Fills formula down to last row of data
```vba
Sub FillFormulasDown()
    Dim ws As Worksheet
    Dim lastRow As Long
    Dim formulaCell As String
    Dim colLetter As String
    
    Set ws = ThisWorkbook.Sheets("Labor")
    colLetter = "E"  ' Column to fill
    lastRow = ws.Cells(ws.Rows.Count, "A").End(xlUp).Row
    
    ' Copy formula from row 5 down to last row
    ws.Range(colLetter & "5").Copy
    ws.Range(colLetter & "5:" & colLetter & lastRow).PasteSpecial xlPasteFormulas
    Application.CutCopyMode = False
    
    MsgBox "Formulas filled to row " & lastRow, vbInformation
End Sub
```
**Module:** Standard Module

---

### Macro 4: Populate Summary from Line Items
**Use:** Pulls subtotals from each sheet into the Recap sheet
```vba
Sub PopulateSummary()
    Dim wsSummary As Worksheet
    Dim wsLabor As Worksheet, wsEquip As Worksheet
    Dim wsMaterials As Worksheet, wsDump As Worksheet
    
    Set wsSummary = ThisWorkbook.Sheets("Recap")
    Set wsLabor = ThisWorkbook.Sheets("Labor")
    Set wsEquip = ThisWorkbook.Sheets("Equipment")
    Set wsMaterials = ThisWorkbook.Sheets("Materials")
    Set wsDump = ThisWorkbook.Sheets("DumpFees")
    
    Application.ScreenUpdating = False
    
    ' Find last rows
    Dim lastLabor As Long, lastEquip As Long
    lastLabor = wsLabor.Cells(wsLabor.Rows.Count, "E").End(xlUp).Row
    lastEquip = wsEquip.Cells(wsEquip.Rows.Count, "E").End(xlUp).Row
    
    ' Write subtotals to Recap
    wsSummary.Range("C5").Value = Application.WorksheetFunction.Sum(wsLabor.Range("E5:E" & lastLabor))
    wsSummary.Range("C6").Value = Application.WorksheetFunction.Sum(wsEquip.Range("E5:E" & lastEquip))
    wsSummary.Range("C7").Value = Application.WorksheetFunction.Sum(wsMaterials.Range("E5:E" & lastEquip))
    wsSummary.Range("C8").Value = Application.WorksheetFunction.Sum(wsDump.Range("E5:E" & lastEquip))
    
    ' Calculate Grand Total
    wsSummary.Range("C10").Value = Application.WorksheetFunction.Sum(wsSummary.Range("C5:C8"))
    
    Application.ScreenUpdating = True
    MsgBox "Summary updated!", vbInformation
End Sub
```
**Module:** Standard Module | **Trigger:** Button on Recap sheet

---

### Macro 5: Protect Sheet with Password
**Use:** Locks formulas but allows data entry in specific cells
```vba
Sub ProtectSheetWithInput()
    Dim ws As Worksheet
    Set ws = ThisWorkbook.Sheets("Labor")
    
    ' Unlock only input cells (B, C, D)
    ws.Cells.Locked = True
    ws.Range("B5:D25").Locked = False  ' Input cells
    ws.Range("E5:E25").Locked = True   ' Formula cells (protected)
    
    ws.Protect Password:="Estimator2026", _
        UserInterfaceOnly:=True, _
        AllowFormattingCells:=True, _
        AllowFormattingColumns:=True
End Sub
```
**Module:** Worksheet Module (for auto-run on activate)

---

### Macro 6: UserForm Data Entry
**Use:** Professional data entry popup for adding line items
```vba
Sub ShowLineItemForm()
    LineItemForm.Show
End Sub
```
*(Requires a UserForm with: Role/Desc TextBox, Hours/Qty TextBox, Rate TextBox, Add Button)*

---

### Macro 7: Auto-Populate Date
**Use:** Auto-fills date when a row is edited
```vba
Private Sub Worksheet_Change(ByVal Target As Range)
    If Not Intersect(Target, Range("B5:B25")) Is Nothing Then
        If Target.Value <> "" Then
            Cells(Target.Row, 6).Value = Format(Date, "MM/DD/YYYY")
        End If
    End If
End Sub
```
**Module:** Worksheet Module (not Standard Module) — paste into sheet's code module

---

### Macro 8: Clear and Reset Sheet
**Use:** Clears all data entry cells for a fresh estimate
```vba
Sub ResetEstimate()
    Dim ws As Worksheet
    Dim response As VbMsgBoxResult
    
    response = MsgBox("Clear all line items? This cannot be undone.", _
        vbYesNo + vbExclamation, "Reset Estimate")
    
    If response = vbYes Then
        Set ws = ThisWorkbook.Sheets("Labor")
        
        Application.ScreenUpdating = False
        
        ' Clear data rows (5-25) but keep headers (1-4)
        ws.Range("A5:E25").ClearContents
        
        ' Reset summary too
        ThisWorkbook.Sheets("Recap").Range("C5:C10").ClearContents
        
        Application.ScreenUpdating = True
        MsgBox "Estimate reset!", vbInformation
    End If
End Sub
```
**Module:** Standard Module

---

### Macro 9: Email Active Sheet as Attachment
**Use:** Emails the estimate sheet to client
```vba
Sub EmailEstimate()
    Dim ws As Worksheet
    Dim subj As String
    Dim recipient As String
    
    Set ws = ActiveSheet
    recipient = InputBox("Client email address:", "Email Estimate")
    
    If recipient <> "" Then
        subj = "Estimate for " & Range("B2").Value & " - " & Range("B4").Value
        
        ws.ExportAsFixedFormat Type:=xlTypePDF, _
            Filename:=Environ("TEMP") & "\Estimate.pdf"
        
        ThisWorkbook.SendMail _
            Recipients:=recipient, _
            Subject:=subj, _
            Attachments:=Environ("TEMP") & "\Estimate.pdf"
        
        MsgBox "Estimate sent to " & recipient, vbInformation
    End If
End Sub
```
**Module:** Standard Module | **Note:** Requires Outlook configured

---

### Macro 10: Conditional Formatting Highlighter
**Use:** Highlights rows where total exceeds a threshold
```vba
Sub HighlightHighItems()
    Dim ws As Worksheet
    Dim lastRow As Long
    Dim i As Long
    Dim threshold As Double
    
    Set ws = ThisWorkbook.Sheets("Labor")
    threshold = InputBox("Highlight items over $:", "Threshold", 500)
    
    lastRow = ws.Cells(ws.Rows.Count, "E").End(xlUp).Row
    
    ' Clear existing conditional formatting
    ws.Range("A5:E25").FormatConditions.Delete
    
    ' Add highlight for high values
    For i = 5 To lastRow
        If IsNumeric(ws.Cells(i, 5).Value) Then
            If ws.Cells(i, 5).Value > threshold Then
                ws.Range("A" & i & ":E" & i).Interior.Color = RGB(255, 230, 153)
            End If
        End If
    Next i
    
    MsgBox "High-value items highlighted!", vbInformation
End Sub
```
**Module:** Standard Module

---

## Error Handling

```vba
Sub SafeMacro()
    On Error GoTo ErrorHandler
    
    ' Your code here
    Dim ws As Worksheet
    Set ws = ThisWorkbook.Sheets("Labor")
    ' ... code that might fail
    
    Exit Sub
    
ErrorHandler:
    Application.ScreenUpdating = True
    Application.Calculation = xlCalculationAutomatic
    MsgBox "Error " & Err.Number & ": " & Err.Description, vbCritical
End Sub
```

---

## Useful Constants

```vba
xlUp = -4162
xlDown = -4121
xlToLeft = -4159
xlToRight = -4161
xlCalculationManual = 2
xlCalculationAutomatic = 1
xlTypePDF = 0
xlQualityStandard = 0
```

---

## GitHub Topic Pages
- `github.com/topics/excel-macros` — VBA macros by stars
- `github.com/topics/excel-automation` — Business automation VBA
- `github.com/topics/vba-excel` — General VBA projects

---

## Top Reference Repos
| Repo | What to Study |
|---|---|
| `OsSyLab/Excel-VBA-Macros` | 20 ready-to-use macros with before/after docs |
| `Lawrence-Gong/VBA-Automation-Toolkit` | Enterprise-scale VBA, professional patterns |
| `sergey-frolov-pets/excel-helpers` | Focused business tool snippets |
| `HarshaaNandakumar/Freight-Quote-Automation` | Quote generator with PDF export, audit log |

---

*v1.0 — 2026-07-10*
