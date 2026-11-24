# Prompts

The main prompts used at each step, in order. The code chat and the analysis chat were kept separate, and neither the report file, the mapping table nor the scaling key was ever sent to the chat that wrote the code.

## 1. Pseudonymization script (code chat, no files attached)

### 1.1 First version

> Write a Python script (Jupyter-friendly) that anonymizes a company annual report in XHTML format (ESEF, converted by pdf2htmlEX). Do not ask me for the file content — I will run the script locally.
>
> **Input and page selection**
> - Ask me for page numbers in decimal (e.g. 3, 114, 115). Each page is a `<div id="pfXX">` where XX is the page number in hexadecimal (114 → pf72). Keep only those pages.
> - Before extracting text, remove `ix:header`, `<title>`, `<meta>`, `<script>`, `<style>` and images. Never modify the original file.
>
> **Step 1 — find candidates (no replacing yet)**
> Print a list with counts and short context of: capitalized words that occur 3+ times; names next to titles (Mr, Ms, Mrs, Sir, Dr, CEO, CFO, Chairman, Director, Board); URLs; emails; long alphanumeric codes (e.g. LEI). Save it to `candidates.csv`.
>
> **Step 2 — replace names**
> Read `mapping.csv` with columns `original, replacement, type`, which I fill in myself. Case-sensitive, whole words only, longest strings first (so "Kernel Holding S.A." is replaced before "Kernel"). Also replace possessives ("Kernel's").
>
> **Step 3 — scale numbers**
> Ask me for a coefficient. Multiply ONLY numbers inside `ix:nonFraction` tags whose `unitRef` is a currency (e.g. USD), respecting the `scale` and `sign` attributes and formats like "1,194,387", "(1,133,738)", "—". Do not change years, percentages, ratios, share counts, note numbers or untagged numbers.
>
> **Step 4 — check and save**
> - Search the result case-insensitively and without word boundaries for every original string in `mapping.csv`. Print each remaining match with 60 characters of context. Expected: 0 matches.
> - Save `output/anonymized.txt` and `output/key.json` (mapping and coefficient) so I can reverse the process later. Add a small function `deanonymize()` that reverses names and divides numbers by the coefficient.
>
> Explain each block with short comments.

### 1.2 Fix: words and numbers split by `<span>` tags

> The text extraction inserts spaces inside words and numbers ("Ker nel", "369,47 0") because pdf2htmlEX splits them with `<span>` tags. Fix:
> 1. Remove `<span>` tags with an empty string; insert a newline only between `<div>` elements.
> 2. For scaling, read the full text of each `ix:nonFraction` tag (all nested spans joined) before parsing the number.
> 3. Treat units like USD per share as currency too (EPS must be scaled).
> 4. After replacement, also search for surnames alone and for the company name without word boundaries, and print every match with context.

### 1.3 Fix: signatures, headers and footers

> 1. Remove the signature block on every page, from "On behalf of the Board of Directors" up to and including the line with the job titles. The names in this block are split across several `<div>` elements, so replacing them does not work.
> 2. Remove the repeated page header and footer (report title with page number, fragments of the website address, the side menu, "The accompanying notes are an integral part of these financial statements"). A line counts as a service line if it occurs on every selected page. Print the list of removed lines.
> 3. Warn if a replacement value in mapping.csv is shorter than 3 characters.
> 4. Print remaining matches for every original value, each surname alone and the first letters of the company name, case-insensitive and without word boundaries.

### 1.4 Tried and rolled back: masking amounts on untagged pages

> On note pages (no ix:nonFraction tags), replace monetary amounts with [AMOUNT] … keep years, dates, percentages and note references unchanged.

Rolled back because the driver analysis needed the amounts in the notes (see LIMITATIONS.md).

### 1.5 Fix: line-break hyphens and pseudonyms

> 1. Before any replacement, join words split across lines: if a line ends with "-" and the next line starts with a letter, remove the hyphen and the line break.
> 2. Warn if a replacement contains the first letter of the original as a separate token or initials (e.g. "company E" for "Enselco").
> 3. After all replacements, search a copy of the text with all hyphens and line breaks removed, case-insensitive, for every original value and its first 5 letters. Print each match with 60 characters of context.
> 4. Re-run the candidate search on all selected pages and print names not yet in mapping.csv.

## 2. Extraction (new chat, only the pseudonymized text attached)

> You are a credit analyst. The attached text contains anonymized financial statements of "Company A" (USD thousands). Extract three tables: Statement of Financial Position, Statement of Profit or Loss, Statement of Cash Flows.
> Columns: Line item | Note | FY2026 | FY2025.
> Rules: keep original line names and numbers exactly as written; negative numbers in brackets become negative values; "—" becomes 0; do not calculate, round or add any lines. If something is unclear, write "CHECK".
> Output each table as CSV in a separate code block.

## 3. Checks

> I have an Excel file with sheets BS, PL and CF. Column A = line item, C = FY2026, D = FY2025; negative values are stored with a minus sign. Create a sheet "Checks" with columns: Check | Calculated | Reported | Difference | Status. Status = "OK" if |Difference| ≤ 3, otherwise "ERROR"; check 24 shows "INFO". Use XLOOKUP for reported totals. Here is the list of checks: [25 checks: subtotals of each statement section, total assets = total liabilities and equity, PL subtotals, CF subtotals, cash roll-forward, PBT and finance costs between PL and CF, cash in CF vs BS, equity roll-forward]

The formulas were then tested by deliberate errors: +1,000 to Inventories, a changed Total current assets, Finance costs and Interest paid. Each time only the expected checks turned red.

## 4. Ratios

> Add base calculations (EBITDA, Total debt, of which current debt, Net debt, CapEx, Free cash flow) and ratios for FY2026 and FY2025: Net debt / EBITDA, Total debt / EBITDA, Total debt / Total equity, Equity / Total assets, EBITDA / Finance costs, operating profit / Finance costs, current, quick and cash ratios, short-term share of debt, gross, EBITDA and net margins, OCF / profit, free cash flow, OCF / total debt, growth of revenue, EBITDA and net debt. Add RED/AMBER flags with thresholds in separate input cells.

## 5. Driver analysis (analysis chat, pseudonymized notes attached)

> Explain why net debt, lease liabilities, the short-term share of debt and the gap between OCF and profit changed. For every claim, cite the line item or note it comes from.

The answers were first produced from the primary statements only, then again with the notes, and both were checked against the report (see reports/ai_errors_log.md).
