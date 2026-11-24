# Limitations

This is a learning demo built on one public annual report. It shows a workflow, not a complete credit assessment.

## 1. Not every step went through the pseudonymizer

| Part of the report | Names replaced | Numbers scaled |
|---|---|---|
| Primary statements, pp. 114–118 | Yes | Yes, every currency value tagged `ix:nonFraction` |
| Notes and management report (pp. 4–30, 101–103, 135–161) | Yes | **No** |

ESEF tags only the primary statements: the note pages contain no `ix:nonFraction` tags, so the script cannot tell amounts from years, percentages or note references there. Masking those numbers was tried and rolled back, because the driver analysis needed them. As a result, real amounts from the notes sat next to scaled amounts from the statements, and the scaling coefficient could be derived by dividing one by the other.

Early runs also leaked identifiers that were later fixed: names split by line-break hyphens ("Ker-nel", "Enselco Hold-ing"), short forms missing from the mapping ("Enselco"), pseudonyms that kept initials ("company K", "company E", "Mr AVE"), and asset and brand names (Avere, TBK). The detailed analysis was written during these runs and still mixes pseudonyms with some real names.

## 2. A public company cannot be hidden

Pseudonymization reduces what an external AI tool receives, but Kernel remains identifiable:

- the share count (293,129,230) is not a currency value and was not scaled;
- ratios do not change with scaling, so margins and leverage work as a fingerprint;
- narrative facts identify the company directly: Ukraine's largest grain exporter, 530 thousand hectares of leased land, the Chornomorsk terminal, the delisting from the Warsaw Stock Exchange.

For a real, non-public client the same pipeline protects much more, because there is no public report to compare against. Pseudonymization is reversible by design (the key is kept locally) and is not anonymization. Real client data would still require a corporate AI tool whose terms exclude training on and retention of the data.

## 3. Data scope

- One document: the FY2026 annual report with FY2025 comparatives. Growth rates cover one year only; no interim reports and no market data were used.
- The bond prospectus was not read, so covenant thresholds and the fixed charges cover ratio are unknown.
- The auditor's report was read only for the going concern paragraph.
- Events are covered up to the report's publication date (October 2026).

## 4. Extraction and checks

- The LLM extraction was validated by arithmetic: 25 checks per year catch numbers that do not add up, but not a correct number placed under a wrong but consistent label.
- Values were checked for the three primary statements only. Other comprehensive loss (check 25) is a manual input from p. 116.
- Scaling and rounding created differences of up to ±2 thousand; de-scaling with `ROUND(x/k, 0)` restores the published integers.

## 5. Analysis

- RED/AMBER thresholds are typical market benchmarks, not any bank's credit policy.
- Some definitions differ from the company's: EBITDA / finance costs is 6.4x on a gross basis and 10.2x on the company's net basis; net debt includes lease liabilities.
- Three conclusions in the detailed analysis are inferences without direct confirmation in the report: Enselco's pre-acquisition loss (derived from pro forma figures), its post-acquisition profit being mainly IAS 41 revaluation, and the commission-agreement liability being owed to a related party.
- The detailed analysis is in Ukrainian. The English credit memo was drafted with Claude from that analysis and checked against the model.

## 6. Use

Not investment advice, a credit rating or a lending recommendation. All information comes from public sources; statements about related-party transactions only restate what the company disclosed (Notes 8 and 30).
