# Taiwan Battery Supply Chain Trade Dashboard
## Taiwan Battery Supply Chain Trade Dashboard

Tracking monthly data on Taiwan's imports and exports of finished batteries and raw materials, for use by DSET colleagues.
Static HTML dashboard, deployed on GitHub Pages, no server required.

---

## Quick Start (3 Steps)

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Download data (see instructions below) and process into standardized format
python scripts/process_data.py

# 3. Generate static HTML
python scripts/generate_dashboard.py
# → Open docs/index.html, or push to GitHub Pages
```

---

## Architecture

```
TWbattery/
├── config.py                       # HS Code definitions and classifications
├── requirements.txt
├── scripts/
│   ├── fetch_data.py               # Automated CSV download from Customs Administration (Playwright)
│   ├── process_data.py             # Process raw CSV → standardized format
│   ├── generate_dashboard.py       # Generate static HTML dashboard
│   └── setup_reminder.sh           # Configure macOS monthly reminder
├── .github/workflows/
│   └── monthly_update.yml          # GitHub Actions: automated monthly rebuild on the 10th
├── data/
│   ├── MANUAL_DOWNLOAD.md          # Manual download instructions
│   ├── raw/                        # Raw CSVs downloaded from Customs Administration
│   └── processed/                  # Processed data (parquet + csv)
└── docs/
    └── index.html                  # Static dashboard (served by GitHub Pages)
```

---

## Tracked HS Codes

| Category | HS Code | Description |
|----------|---------|-------------|
| Finished Batteries | 850760 | Lithium-ion accumulators |
| Finished Batteries | 850790 | Accumulator parts |
| Cathode Materials | 283329 | Other sulphates (cobalt / nickel / manganese) |
| Cathode Materials | 282200 | Cobalt oxides and hydroxides |
| Cathode Materials | 282520 | Lithium oxide and hydroxide |
| Cathode Materials | 282540 | Nickel oxides and hydroxides |
| Cathode Materials | 283691 | Lithium carbonates |
| Anode Materials | 250410 | Natural graphite in powder or flakes |
| Anode Materials | 380110 | Artificial graphite |
| Anode Materials | 280300 | Carbon black |
| Electrolytes | 382499 | Other chemical products not elsewhere specified |
| Metal Raw Materials | 750210 | Unwrought nickel, not alloyed |
| Metal Raw Materials | 281820 | Aluminum oxide (excl. artificial corundum) |
| Metal Raw Materials | 810520 | Cobalt powders |
| Packaging Materials | 390120 | High-density polyethylene (HDPE) |
| Packaging Materials | 390210 | Polypropylene (PP) |

---

## Data Download

### Method A: Automated (Recommended, requires Playwright)

```bash
playwright install chromium   # Only needs to be run once
python scripts/fetch_data.py --start 202001 --end 202503
```

If a CAPTCHA appears, keep the browser window open, solve it manually, and press Enter to continue.

### Method B: Manual

Refer to [`data/MANUAL_DOWNLOAD.md`](data/MANUAL_DOWNLOAD.md) for detailed step-by-step instructions.

---

## Monthly Update Workflow

### Automated (GitHub Actions)

`.github/workflows/monthly_update.yml` is scheduled to run automatically on the 10th of every month:
1. Reprocess CSV files under `data/raw/`
2. Regenerate `docs/index.html`
3. Commit & push changes back to the repository

**Prerequisite**: The latest monthly CSV must be placed in `data/raw/` and pushed manually before the Action can process new data.

### Manual Update

```bash
python scripts/fetch_data.py          # Download new monthly data
python scripts/process_data.py        # Reprocess data
python scripts/generate_dashboard.py  # Regenerate HTML
git add data/ docs/ && git commit -m "data: update to YYYY-MM" && git push
```

### macOS Calendar Reminder (10th of every month)

```bash
bash scripts/setup_reminder.sh
```

---

## Setting up GitHub Pages

1. Push the repository to GitHub
2. Settings → Pages → Source: **Deploy from a branch**
3. Branch: `main` / Folder: `/docs`
4. Save → dashboard URL: `https://deeper747.github.io/TWbattery/`

---

## Ukraine HS 8507 Import Analysis

### Research Questions

Has Taiwan ever exported electric vehicle batteries (HS 8507) to Ukraine? How has the volume changed before and after the Russia-Ukraine war? Where does Taiwan rank among Ukraine's import source countries?

### Data Sources

Annual import and export details released by the State Customs Service of Ukraine (Державна митна служба України), available in Excel format, broken down by country of origin × 6-digit HS code for import value (thousand USD) and net weight (metric tons).

Data storage path: `data/raw/country_goods/`, file naming pattern: `12 month_YYYY_country_goods.xlsx`

> **Note**: Ukrainian Customs registers Taiwan as "Тайвань, провінція Китаю" (Taiwan, Province of China).

### Analysis Scripts

| Script | Function |
|--------|----------|
| `scripts/fetch_ukraine_taiwan_8507.py` | Filter historical data for Ukrainian imports of HS 8507 from Taiwan, exporting detailed and annual summary CSVs |
| `scripts/fetch_ukraine_hs8507_all_countries.py` | Expand scope to all origin countries, calculate Taiwan's ranking, and generate stacked bar charts and line charts |

### How to Run

```bash
# Step 1: Examine Taiwan → Ukraine figures only
python scripts/fetch_ukraine_taiwan_8507.py
# → data/ukraine_taiwan_8507.csv
# → data/ukraine_taiwan_8507_summary.csv

# Step 2: Compare all partner countries + Taiwan ranking + charts
python scripts/fetch_ukraine_hs8507_all_countries.py
# → data/processed/ukraine_hs8507_all_countries.csv
# → data/processed/ukraine_hs8507_by_country_year.csv
# → data/processed/ukraine_hs8507_ranking_value.csv
# → data/processed/ukraine_hs8507_ranking_weight.csv
# → data/processed/ukraine_hs8507_stacked_value.png
# → data/processed/ukraine_hs8507_stacked_weight.png
# → data/processed/ukraine_hs8507_line_value.png
# → data/processed/ukraine_hs8507_line_weight.png
```

### Key Deliverables

- **Ranking Tables**: Top 3 supplying countries per year + Taiwan's rank and value, provided in both import value and net weight versions.
- **Stacked Bar Charts**: Share of top 10 source countries by year (`*` denotes incomplete year data).
- **Line Charts**: Historical trends of top 10 source countries (logarithmic scale).

---

## Data Sources

- **Taiwan Trade Data**: Ministry of Finance Customs Administration Statistical Database Query Portal (portal.sw.nat.gov.tw/APGA/GA30)
  - Released after the 2nd of each month for previous month's data; minor adjustments may occur before month-end.
- **Ukraine Import Data**: State Customs Service of Ukraine annual Excel reports.
- **HS Code Classification**: Adapted from methodology used in the U.S. Energy Trade Dashboard and Council on Strategic Risks studies.