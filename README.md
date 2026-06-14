# Replication package — Horn & Sanctuary (2026)

This folder is a self-contained replication package for the bilateral investment
treaty (BIT) analysis dataset. Two Jupyter notebooks turn the raw input files in
this directory into the final analysis dataset written to [`output/`](output/).

```
raw inputs  ──►  Jupyter notebooks  ──►  output/iia_dataset_for_paper_20260323.csv
```

---

## Contents

| Path | Role |
|------|------|
| `build_tax_revenue_datasets.ipynb` | **Notebook 1** — builds the two tax-revenue tables from public APIs |
| `build_iia_dataset_20260323.ipynb` | **Notebook 2** — builds the final BIT analysis dataset |
| `output/` | Generated outputs (created/overwritten by the notebooks) |
| *raw input files* | See [Input files](#input-files) below |

---

## Requirements

- **Python ≥ 3.10** (tested on 3.13)
- **Jupyter** (`jupyter notebook` or `jupyter lab`, or run in VS Code)
- Python packages: `pandas`, `numpy`, `requests`, `openpyxl`

Install everything with:

```bash
pip install jupyter pandas numpy requests openpyxl
```

> If you use Anaconda, all of these ship with the base environment.

**Internet access** is required for Notebook 1 (it downloads live data from the
OECD, UNU-WIDER, and World Bank). Notebook 2 runs fully offline.

---

## Quickstart

```bash
# 1. clone / download this folder, then launch Jupyter from inside it
cd replication_data
jupyter lab          # or: jupyter notebook

# 2. (optional) run Notebook 1 to refresh the tax-revenue inputs
#    open build_tax_revenue_datasets.ipynb → Run All

# 3. open build_iia_dataset_20260323.ipynb → Run All
#    (no path edits needed when Jupyter is launched from this folder)

# 4. find the result in output/iia_dataset_for_paper_20260323.csv
```

The two tax-revenue CSVs are already included in this folder, so **you can skip
Notebook 1 and run Notebook 2 directly** to reproduce the final dataset. Run
Notebook 1 only when you want to regenerate the tax inputs from the latest
source vintages.

---

## How the notebooks fit together

```
build_tax_revenue_datasets.ipynb
        │  downloads OECD / UNU-WIDER / World Bank data
        ▼
   tax_revenue_2024_worldbank.csv
   tax_revenue_harmonised_2018_2024.csv
        │  (consumed as inputs)
        ▼
build_iia_dataset_20260323.ipynb  +  other raw inputs
        │
        ▼
   output/iia_dataset_for_paper_20260323.csv
```

---

## Step 1 — `build_tax_revenue_datasets.ipynb` (optional refresh)

Regenerates two country-level tax-revenue tables straight from source APIs. **No
manual path editing is needed** — it writes to the directory you launched Jupyter
from, so launch Jupyter from inside `replication_data`.

Run **Kernel → Restart & Run All**. It produces:

| Output | Description |
|--------|-------------|
| `tax_revenue_2024_worldbank.csv` | World Bank only, 2024. Tax revenue (% of GDP), GDP and tax revenue in **current 2024 US dollars**, 217 economies. |
| `tax_revenue_harmonised_2018_2024.csv` | Hierarchical series (OECD Revenue Statistics → UNU-WIDER GRD 2025 → World Bank WDI). Tax revenue **excluding social contributions**, % of GDP, most recent year 2018–2024 per country. |

Notes:
- The OECD download is ~85 MB; this notebook takes 1–2 minutes.
- Re-running picks up the latest published vintages, so a few values can differ
  from the shipped copies as sources release new years. This is expected.
- A short validation report prints at the end (coverage, year distribution,
  source distribution).

---

## Step 2 — `build_iia_dataset_20260323.ipynb` (main pipeline)

Builds the final dataset: **1,837 rows × 192 columns**, one row per in-force BIT
with ISO-resolved party codes.

**No path editing is needed.** By default `DATA_DIR` is set to the folder the
notebook is run from (`os.getcwd()`), so launching Jupyter from inside
`replication_data` makes the notebook find all its inputs automatically.
`OUTPUT_DIR` (= `DATA_DIR/output`) is created automatically. Just run
**Kernel → Restart & Run All**.

> Running from a different working directory? Set `DATA_DIR` to an absolute path
> in the cell headed `CONFIGURATION — self-contained replication package`.

The pipeline (11 steps, printed as it runs):

1. Load and flatten the scraped IIA JSON dictionary
2. Build a DataFrame and apply column renaming
3. Resolve country names to ISO 3166-1 alpha-3 codes for both treaty parties
4. Standardise values and filter to **BITs / In force / both parties ISO-resolved**
5. Classify protective vs. non-protective treaties
6. Merge biodiversity exposure (IUCN species ranges by country)
7. Merge OECD bilateral outward FDI positions
8. Merge tax-revenue and GDP variables onto both parties
9. Write `output/iia_dataset_for_paper_20260323.csv`

**Output:** [`output/iia_dataset_for_paper_20260323.csv`](output/)

---

## Input files

All inputs live directly inside this folder.

| File | Used by | Description |
|------|---------|-------------|
| `iia_dict_26-02-12.json` | Notebook 2 | Scraped UNCTAD IIA mapping database (nested JSON, ~41 MB) |
| `rename_datacols.csv` | Notebook 2 | JSON-path → short column-name mapping |
| `iso_2024-04-09.xlsx` | Notebook 2 | Country name → ISO3 lookup |
| `full_species_list_dec2023_final_240108_renamed.csv` | Notebook 2 | IUCN Red List species assessments |
| `spp_pp_2023_final_240111_renamed.csv` | Notebook 2 | Species range fractions by country |
| `oecd_fdi_outward_raw.csv` | Notebook 2 | OECD bilateral outward FDI positions (~63 MB) |
| `tax_revenue_2024_worldbank.csv` | Notebook 2 | Produced by Notebook 1 (World Bank, 2024, USD) |
| `tax_revenue_harmonised_2018_2024.csv` | Notebook 2 | Produced by Notebook 1 (harmonised, % GDP) |
| `IIABD_t_treaties_20260323.csv` | Notebook 2 *(optional)* | Reference dataset for the built-in validation check; if absent, validation is skipped automatically |

---

## Output

`output/iia_dataset_for_paper_20260323.csv` — the analysis dataset, one row per
in-force BIT (1,837 rows × 192 columns). Re-running Notebook 2 overwrites this
file in place.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `FileNotFoundError` in Notebook 2 | Launch Jupyter from inside `replication_data` (so `DATA_DIR` resolves correctly), and confirm all [input files](#input-files) are present. |
| `ModuleNotFoundError: openpyxl` | `pip install openpyxl` (needed to read `.xlsx`). |
| Notebook 1 connection / HTTP errors | A source API was temporarily unavailable; re-run the cell. The shipped tax CSVs let you proceed without Notebook 1. |
| Tax values differ slightly from shipped CSVs | Expected — Notebook 1 pulls the latest source vintages on each run. |
| `Reference dataset not found … Skipping validation.` | Harmless. `IIABD_t_treaties_20260323.csv` is optional; only the validation cell uses it. |

---

## Reproducing from scratch

```bash
cd replication_data
pip install jupyter pandas numpy requests openpyxl

# refresh tax inputs (optional, needs internet)
jupyter nbconvert --to notebook --execute --inplace build_tax_revenue_datasets.ipynb

# build the final dataset (run from this folder; no path edits needed)
jupyter nbconvert --to notebook --execute --inplace build_iia_dataset_20260323.ipynb

# result: output/iia_dataset_for_paper_20260323.csv
```
