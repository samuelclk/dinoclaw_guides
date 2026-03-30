---
name: accounting-workpapers
description: Build, clean, reconcile, and deliver accounting workpapers and bookkeeping support files from messy source material such as screenshots, receipts, booking summaries, Google Drive files, spreadsheets, bank exports, PDFs, and ad hoc user notes. ABSOLUTE TRIGGER: if the user explicitly mentions `accounting-workpapers`, says "use accounting-workpapers", or asks for the accounting-workpapers skill, use this skill. Also use when preparing annual filings, expense schedules, travel expense workpapers, supporting schedules, reconciliation sheets, or any accounting/admin workbook where the output must be clean, numerically correct, and reviewable. Especially use when extracting line items from screenshots into spreadsheets, rolling up totals, linking evidence files, reconciling source documents against draft statements, or producing a tidy Google Sheet/XLSX for the user.
---

# Accounting Workpapers

## Core rule

Verify the actual artifact before saying it is done.

If you created or edited a workbook, sheet, formula, export, Drive link, or uploaded file, inspect the real result directly before replying.

After every task or edit on a workbook/sheet, always include the working file URL in the user-facing response.

Do not trust:
- your intent
- your local intermediate file
- your memory of what should have happened
- Drive preview behavior for uploaded Office files

Check the actual thing the user will open.

## Workflow

### 1. Identify the real source set first

Before extracting or reconciling, enumerate all candidate source files if there may be duplicates or similar versions.

For Google Drive work:
- search for alternate file names and year variants
- verify the exact file ID and title
- do not assume the first matching file is the right one
- if multiple similar files exist, inspect metadata and latest modified items before speaking confidently

When the user says a file exists, check harder before saying it cannot be found.

### 2. Prefer raw source data over polished summaries

Best source hierarchy:
1. transaction exports / bank CSVs / workbook tabs
2. booking confirmations / receipts / source screenshots
3. user-written summaries
4. draft statements

If the task is to build a workpaper, prefer one row per supported source item.

### 3. Decide the output schema before writing data

Before building a workbook, define the exact columns required.

If the user specifies a schema, follow it exactly.
Do not add “helpful” extra columns, summaries, notes, formulas, or metadata unless explicitly asked.

For screenshot-derived expense schedules, default to the minimum reviewable schema:
- line item number
- item type
- booking date
- travel or check-in date
- amount in SGD
- flight or hotel name
- screenshot URL

Keep summaries outside the main data table unless explicitly requested.

### 4. Never mix summary formulas into the data table

Do not insert totals, subtotals, or summary labels inside the row region meant for line items.

If a total is requested:
- put it in a clearly separate summary area, or
- add one final TOTAL row only after the data table is complete

If the user asks for a total and the table is the main deliverable, prefer:
- one final `TOTAL` row after all data rows
- make every numeric total cell in that row a live `=SUM(...)` formula, not a hardcoded number
- totals only in the explicitly requested summary columns
- no subtotals unless the user asked for them
- make the final `TOTAL` row bold by default

If the user is extracting expense-style rows into a review sheet and does **not** explicitly say otherwise, include a final total row by default.

### 5. When data is incomplete, choose one of two modes explicitly

#### Mode A: strict supported rows only
Use when the user wants a clean schedule fast.
- include only rows with supported amount/date/name values
- omit unsupported partial rows
- do not leave mystery blanks if they make the sheet look unfinished

#### Mode B: partial capture
Use only when the user explicitly wants incomplete rows preserved.
- leave blanks intentionally
- mark unsupported fields clearly outside the main requested schema if allowed

Default: use Mode A.

### 6. For booking screenshots, avoid double counting

If a screenshot shows a booking-level total covering multiple legs/items, do not duplicate that amount onto every leg.

Prefer one of these patterns:
- one booking-level row with the total, or
- separate rows only if the screenshot truly provides separate line-item amounts

When in doubt, use one row per booking total.

### 7. For screenshot evidence, upload and link systematically

When the user wants screenshot URLs:
1. create/find a public evidence folder
2. upload every relevant screenshot in the batch, not just the already-processed ones
3. capture public URLs in a mapping table
4. populate the sheet only after that mapping exists

Do not claim all screenshots are uploaded until you have counted them.

### 8. For Google Sheets deliverables, prefer native Google Sheets

If the user will review/edit in Google Sheets, prefer a native Google Sheet over an uploaded XLSX preview.

Reason:
- uploaded Excel files can render weirdly in Drive preview
- native Sheets are easier to verify with `gog sheets get/update`

Recommended flow:
- build clean local XLSX if helpful
- convert/upload as native Google Sheet
- verify live cell values through Sheets API

### 9. Verify the live artifact after every important change

For Google Sheets, verify with direct cell reads from the live sheet.
Check:
- headers
- a sample of data rows
- amount column values as strings/numbers
- URL column presence
- total row if requested
- visible sort order if the user asked for chronological ordering
- column widths / layout if you changed schema or added columns
- the working file URL, so it can be included in the reply every time

Formatting preference for review sheets:
- make columns fit to the smallest width that still keeps all text visible
- do not wrap cells unless the user explicitly asks for wrapping
- prefer overflow/no-wrap presentation instead of taller wrapped rows

For local XLSX, open/read with openpyxl and inspect cell values.

### 10. Default sort convention for reviewable workpapers

Unless the user asks otherwise:
- sort transaction-style tabs and supporting schedules by the relevant business date column
- for travel / expense review tabs, default to `newest on top`
- if multiple tabs are part of one deliverable, apply the same direction consistently across them
- verify the live order from the real sheet after sorting; do not assume the sort worked

When a tab has a dedicated date column such as `Booking Date`, sort by that actual column, not by row number or a line-item id column.

If the user later says “latest date on top”, treat that as a sticky preference for this skill and apply it by default to future expense-style extraction tabs unless they override it.

If the user asks to split a category into its own worksheet, make that routing preference stick for the current skill workflow. Example: move all USD expenses except descriptions containing `Airalo` into a separate worksheet titled `Subscriptions`, then remove those rows from the main expense tab and maintain totals/sort/formatting on both tabs.

### 10. When the user already asked and the task is obviously unfinished, continue

Do not ask permission again for the same unfinished deliverable.
Just continue until the requested outcome is complete, unless a destructive or external-risk action newly appears.

## Travel/workpaper pattern

For travel expense extraction from screenshots:

1. collect all relevant screenshots in the batch
2. upload screenshots to a public evidence folder
3. extract item-level bookings carefully
4. normalize to the requested schema
5. avoid double counting round-trip totals
6. populate screenshot URLs for every retained row
7. add a final TOTAL row only if asked or clearly useful
8. verify the total directly from the live native Google Sheet

### Email attachment → spreadsheet workflow

Use this when the source set arrives as emails with attachments, especially `.eml`, PDF receipts, or forwarded booking emails.

1. fetch the parent email and enumerate **all** attachments first
2. determine the reporting period explicitly before filtering rows
   - do not silently assume calendar-year logic
   - if the user gives a fiscal-year window, apply that exact window
3. if attachments are nested emails (`.eml` / `message/rfc822`):
   - inspect each attachment as its own source document
   - extract the source email date, booking identifiers, and visible total/currency fields
4. create or find a public Drive evidence folder and upload one evidence artifact per retained source item
5. build the spreadsheet only after you know:
   - total source attachment count
   - retained attachment count
   - exact schema
   - evidence-link mapping
6. if multiple currencies appear, do **not** force them into one or two columns
   - create one column per encountered reporting currency when the user expects explicit native-currency visibility
   - if SGD normalization is required, add a separate `Converted to SGD Cost` column
7. if the user asks for normalized SGD values:
   - use explicit FX assumptions stated in the sheet/task notes or user instruction
   - prefer quoted pair logic such as `SGD/MYR` and `SGD/THB` instead of vague prose rates
   - calculate the converted value in a dedicated column, not by overwriting native-currency columns
8. if the user asks for a total:
   - add one final `TOTAL` row after the data region
   - make the `TOTAL` row bold by default
   - total the normalized column unless the user asked for native-currency subtotals
9. after populating the sheet:
   - verify headers, row count, sort order, links, converted values, and total row formatting from the live sheet

## Annual filing / reconciliation pattern

When reconciling statements against source workbooks:

1. verify the exact source file IDs first
2. extract top-line statement values from the real workbook tabs
3. compare them with the draft UFS/financial statements
4. identify template artifacts, mislabeled periods, broken notes, and internal inconsistencies
5. do not call the document ready if dates/labels/logic are inconsistent even when top-line values match

## Anti-patterns to avoid

Do not:
- tell the user a file cannot be found before checking alternate candidates
- publish a workbook before checking the actual live artifact
- leave broken formulas in the data table
- store amounts as dates/times by accident
- present partial completion as complete
- ask for permission again when the same unfinished task obviously still needs completion

## Completion checklist

Before replying that a workpaper is done, verify:
- correct source files used
- correct schema used
- no garbage columns
- no broken formulas in data rows
- no accidental date/time formatting in amount cells
- no unsupported partial rows unless explicitly requested
- screenshot links populated where expected
- total checked from the live artifact if a total matters
- total cells use live `=SUM(...)` formulas when a TOTAL row exists
- final user-facing URL tested/read back from the live file metadata
