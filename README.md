# AI-assisted credit analysis: Kernel Holding S.A. (FY2026)

A demo of an AI-assisted credit underwriting workflow, built on the public FY2026 annual report of Kernel Holding S.A. (Ukrainian agribusiness, IFRS, year to 30 June 2026). The goal is to show how classical credit analysis can be partly automated with AI tools while every AI output stays verified against the source document.

**Author:** Kyrylo Iovenko, Economics and Big Data student, Kyiv School of Economics (November 2026)

## Workflow

| Step | What was done | Output |
|---|---|---|
| 1. Pseudonymize | A Python script reads the ESEF/XHTML report, keeps only the selected pages, removes headers, footers and signatures, replaces company and personal names from a local mapping table, and scales every currency value tagged `ix:nonFraction` by a private coefficient. A final leak check searches for any remaining original name. | Text that is safer to send to an external LLM; the key stays local |
| 2. Extract | An LLM converted the pseudonymized balance sheet, P&L and cash flow statement into tables, which were moved to Excel and de-scaled locally with `ROUND(x/k, 0)`. | `model/` sheets BS, PL, CF |
| 3. Validate | 25 reconciliation checks per year (subtotals, statement links, opening and closing cash, equity roll-forward), tested by deliberate error injection in each statement. Result: 47 OK, 0 errors, 2 explained INFO items. | `model/` sheet Checks |
| 4. Analyse | Credit ratios with RED/AMBER flags, reconciliation to company-reported KPIs (all within rounding), driver analysis using the notes, one-page credit memo. | `model/` sheet Ratios, `reports/` |

## Key findings

Leverage is moderate (Net debt / EBITDA 1.3x), but net debt grew 5.1x to USD 736m, mainly because of a USD 348m cash acquisition of Enselco from an entity controlled by the Chairman. The USD 300m Eurobond due October 2027 is shown as current because of cross-acceleration with a waived EIB loan covenant, and no refinancing plan is disclosed. After year-end, Russian attacks halted the main grain terminal, and the auditor's report contains a material uncertainty paragraph on going concern. See [reports/credit_memo_FY2026.pdf](reports/credit_memo_FY2026.pdf).

## How AI was used and checked

LLM assistants (Claude) were used to draft the scripts, the Excel formulas, the table extraction and parts of the analysis. Every step was verified: arithmetic checks with error injection, reconciliation to the company's own KPIs, and checks of every causal claim against the notes. Eight AI errors or omissions were found and corrected; they are listed in [reports/ai_errors_log.md](reports/ai_errors_log.md). The prompts are in [prompts/prompts.md](prompts/prompts.md).

## Repository

```
model/kernel_fy2026_credit_model.xlsx   BS, PL, CF, Checks, Ratios (USD thousands, real figures)
reports/credit_memo_FY2026.pdf          credit memo (English)
reports/detailed_analysis_FY2026.pdf    detailed analysis (Ukrainian, working paper)
reports/ai_errors_log.md                where the AI was wrong and how it was found
prompts/prompts.md                      prompts used at each step
scripts/                                pseudonymization script
templates/mapping_template.csv          format of the local mapping table (fictional rows)
LIMITATIONS.md                          what this demo does not cover
```

The mapping table, the scaling key and the pseudonymized text are not in the repository by design (see `.gitignore`).

## Reproduce

1. Download the FY2026 annual report (XHTML) from [Kernel Investor Relations](https://www.kernel.ua/investor-relations/financial-reports/) into `data/`.
2. `pip install -r requirements.txt`
3. Copy `templates/mapping_template.csv` to `mapping.csv`, fill it in, and run the script in `scripts/`.

## Limitations

Read [LIMITATIONS.md](LIMITATIONS.md) before relying on any number. In short: only the primary statements were scaled, Kernel stays identifiable from public facts, and this is a demo based on public data, not a credit decision or investment advice.
