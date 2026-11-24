# Pseudonymization script

`anonymize_esef.ipynb` runs locally and never sends data anywhere. Leave `INPUT_FILE` and `COEFFICIENT` as `None`; the notebook asks for them.

1. Asks for page numbers (decimal) and keeps only those pages (`<div id="pfXX">`, XX in hexadecimal).
2. Removes `ix:header`, metadata, images, repeated headers and footers, and signature blocks; joins words split by `<span>` tags and line-break hyphens.
3. Lists candidate identifiers (`candidates.csv`) and writes `mapping_template.csv`; fill it in by hand and save it as `mapping.csv`.
4. Replaces names from `mapping.csv`: case-sensitive, whole words, longest first, possessives included.
5. Asks for a coefficient and scales only currency values tagged `ix:nonFraction`.
6. Runs a leak check and saves `output/anonymized.txt` and `output/key.json`; `deanonymize()` reverses both steps.

`mapping.csv`, `key.json` and `output/` are excluded by `.gitignore`. Use `templates/mapping_template.csv` for the format.
