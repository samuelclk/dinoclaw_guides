---
name: financial-statements
description: Generate, repair, reconcile, and roll forward linked financial statements in spreadsheets, especially Profit & Loss / Income Statement, Balance Sheet, Cash Flow Statement, and Statement of Changes in Equity. Use when building current-year statements from prior-year closing balances plus current-year schedules, when tracing how statement tabs link together in Google Sheets or Excel, when fixing broken cross-sheet formulas, or when creating a reusable statement model from supporting schedules such as revenue, expenses, depreciation, working-capital, and equity movements.
---

# Financial Statements

Build and edit linked statement workbooks so the four primary statements agree mathematically, not just cosmetically. Use this skill to trace formula flows, roll balances forward from prior year, and repair statement logic when a workbook looks plausible but does not actually reconcile.

## Workflow

### 1. Verify the workbook and read formulas, not just displayed values

For Google Sheets, inspect metadata first to identify the real statement tabs and supporting schedules.

Preferred checks:
- `gog sheets metadata <sheetId> --json`
- `gog sheets get <sheetId> "Tab!A1:Z200" --json --render FORMULA`
- `gog sheets get <sheetId> "Tab!A1:Z200" --json --render UNFORMATTED_VALUE`

Do not rely only on formatted values. Statement work depends on formula lineage.

### 2. Identify the statement roles

Classify tabs into:
- primary statements: P&L / income statement, balance sheet, cash flow statement, SOCE
- supporting schedules: revenue, COGS, subscriptions, payroll, travel, depreciation, loans, tax, fixed assets, working-capital detail
- scratchpads / hidden helper tabs

Write down the role of each tab before editing formulas.

### 3. Map the linkage graph

For each primary statement line, identify whether it is:
- input manually entered
- summed from a supporting schedule
- linked from another primary statement
- derived from prior-year closing balances

Minimum linkage map to confirm:
- P&L net profit/loss flows into retained earnings / accumulated losses
- BS cash agrees to CFS ending cash
- SOCE closing equity agrees to BS equity
- depreciation or other non-cash items are added back in CFS
- capex, loans, and share issuances are reflected in the correct statement sections

If the workbook fails any of these, do not call it balanced.

### 4. Use the standard build order

When creating a new year from prior-year statements, build in this order:

1. prior-year closing balance sheet
2. opening equity buckets for SOCE
3. current-year P&L from detailed schedules
4. non-cash schedules such as depreciation and accruals
5. current-year ending balance sheet
6. cash flow statement derived from P&L + balance-sheet movements
7. SOCE closing balances
8. final cross-checks

This order reduces circular confusion.

## Statement construction rules

### Profit & Loss / Income Statement

Use the P&L as the current-year performance statement.

Typical pattern:
- revenue tabs feed revenue lines
- COGS / direct costs feed gross profit
- opex schedules feed operating expenses
- non-operating items and tax sit below operating result
- total comprehensive income/loss becomes the equity bridge into SOCE / BS accumulated losses

Checks:
- subtotal formulas only sum intended lines
- supporting schedules tie exactly to the statement line
- sign convention is consistent across revenue, expenses, and tax

### Balance Sheet

Build the ending balance sheet from:
- prior-year closing balances
- current-year movements
- current-year profit/loss

Core equation:
- assets = liabilities + equity

Common bridges:
- ending PPE = opening PPE + additions - depreciation - disposals
- ending receivables / payables reflect working-capital movement used in CFS
- ending retained earnings / accumulated losses reflect opening balance + current-year profit/loss - distributions + prior-period adjustments

Never accept a workbook as finished if the balance sheet does not balance exactly.

### Cash Flow Statement

Preferred indirect method:

1. start from profit/loss before tax or after tax depending on the model
2. add back non-cash items
3. adjust for working-capital movements using BS deltas
4. include investing movements such as capex
5. include financing movements such as loans, repayments, share capital changes, and dividends
6. reconcile to ending cash on the BS

Critical rule:
- ending cash in CFS must equal cash and cash equivalents on the balance sheet

### Statement of Changes in Equity

Build SOCE from equity buckets, not from guesswork.

Typical columns:
- share capital
- reserves if any
- retained earnings / accumulated losses
- total equity

Typical rows:
- opening balance
- current-year total comprehensive income/loss
- owner contributions / share issuance
- dividends / distributions
- other reserve movements
- closing balance

Critical rule:
- closing total equity in SOCE must equal total equity on the balance sheet

## Roll-forward method for a new year

When the user wants current-year statements from prior-year statements plus current-year P&L/schedules:

1. copy prior-year closing BS into the new workbook as opening positions
2. copy prior-year closing SOCE equity buckets into the new opening SOCE row
3. build the current-year P&L from detailed current-year schedules
4. update non-cash schedules such as depreciation, amortization, accruals, and provisions
5. compute ending BS line items from opening balances plus movements
6. derive the CFS from P&L plus BS movement analysis
7. post current-year profit/loss and any capital movements into SOCE
8. run all three tie-outs:
   - assets vs liabilities + equity
   - BS cash vs CFS ending cash
   - BS equity vs SOCE closing total

## Editing / repair workflow

When a workbook already exists but is wrong:

1. read formulas on all four primary statements
2. trace each broken or suspicious line to its upstream schedule
3. separate template placeholders from real business logic
4. fix the smallest number of cells that restores the model cleanly
5. verify live outputs after every important edit
6. document remaining modeling assumptions and unresolved gaps

Prefer repairing the source schedule or shared driver cell instead of hardcoding a statement line.

## Red flags

Stop and call out the issue if you see any of these:
- balance sheet does not balance
- CFS ending cash differs from BS cash
- SOCE total equity differs from BS equity
- current-year loss is linked to the wrong equity bucket
- signs flip between statements without explanation
- narrative notes reference a different amount than the main statement line
- helper tabs contain overridden hardcoded values that break the roll-forward

## Output expectations

When reporting back to the user, include:
- the workbook URL
- the statement tabs reviewed
- the key linkage logic found
- whether the model actually balances
- exact inconsistencies if it does not
- what was created or changed in the workbook or skill

## Reference

Use an anonymized real-world linkage example when you need a concrete reference for how a workbook linked P&L, BS, CFS, and SOCE tabs, including where it reconciled correctly and where it failed.
