# Seasonal Biomarker Atlas

An interactive globe for visualising seasonal variation in biomarker levels across countries. Designed to be data-driven and future-proof: swap in any seasonal biomarker by editing CSV files and a JSON configuration — no code changes required.

The default dataset shows **serum 25-hydroxyvitamin D [25(OH)D]** across 39 countries, compiled from three systematic literature reviews covering national health surveys and published cohort studies. Data span healthy adults, paediatrics, and elderly populations.

## Live demo

Deploy to GitHub Pages (see below) or open `index.html` directly in a browser.

## How it works

The tool reads two things at startup:

1. **`data/config.json`** — defines the biomarker name, units, color scale, status thresholds, dataset file paths, and all display text. Everything the user sees on screen comes from this file.

2. **CSV data files** in `data/` — one file per population group. Each row is a geographic location with a measured value in June–August and a measured value in December–February. The globe interpolates sinusoidally between these two points as the month slider moves.

When a user drags the month slider, the app calculates the expected value at each location for that calendar month using a sinusoidal model. This naturally handles hemisphere differences: a location at 34°S whose June–August value is low (local winter) will show that low value when the slider is on July, while a location at 51°N whose June–August value is high (local summer) will peak at the same time.

## CSV format

Each data CSV must have these columns (header names are mapped in `config.json` under `csv_columns`):

| Column | Description |
|--------|-------------|
| `country` | Country name |
| `region` | Sub-national region or study name |
| `lat` | Latitude (decimal degrees, negative for south) |
| `lng` | Longitude (decimal degrees, negative for west) |
| `jun_aug_value` | Measured biomarker value during June–August (in standardised units) |
| `dec_feb_value` | Measured biomarker value during December–February (in standardised units) |
| `n` | Sample size |
| `short_citation` | Abbreviated citation for on-screen display (e.g., "Looker et al. 2002") |
| `full_citation` | Complete bibliographic reference with authors, title, journal, volume, pages |
| `url` | Link to PubMed, DOI, or source website |

For tropical or equatorial locations with minimal seasonal variation, set `jun_aug_value` and `dec_feb_value` to similar values.

For locations where the biomarker is paradoxically *higher* in winter (e.g., Saudi Arabian vitamin D due to summer sun-avoidance), simply enter the actual measured values for each period — the sinusoidal model will reflect the inverted pattern automatically.

## Swapping in a different biomarker

To display a different seasonal biomarker (cortisol, melatonin, testosterone, TSH, etc.):

1. **Edit `data/config.json`**: change `title`, `subtitle`, `biomarker_name`, `unit`, `description`, `about`, and the `color_scale` thresholds to reflect the new biomarker's clinical interpretation.

2. **Replace the CSV files** in `data/` with new data in the same column format. Update the `datasets` array in `config.json` if you change filenames or add/remove population groups.

3. **Adjust `interpolation_peak_month`** if the biomarker's annual peak is not in August (month 8). For example, melatonin peaks in winter, so you might set this to 1 (January) for Northern Hemisphere populations.

No changes to `index.html` are needed.

## Deploying to GitHub Pages

1. Push this repository to GitHub.
2. Go to **Settings → Pages**.
3. Under "Source", select **Deploy from a branch** and choose `main` (or `master`) and `/ (root)`.
4. The site will be live at `https://<username>.github.io/<repo-name>/` within a few minutes.

## Dependencies

All loaded from CDNs at runtime — no build step, no `npm install`:

- [D3.js v7](https://d3js.org/) — globe projection, rendering, and interaction
- [TopoJSON Client v3](https://github.com/topojson/topojson-client) — country boundary decoding
- [World Atlas v2](https://github.com/topojson/world-atlas) — Natural Earth 110m country boundaries
- [PapaParse v5](https://www.papaparse.com/) — CSV parsing

## File structure

```
├── index.html                  Main application (HTML + CSS + JS, no build step)
├── data/
│   ├── config.json             Display configuration (swap for any biomarker)
│   ├── healthy_adults.csv      Adult population data
│   ├── paediatrics.csv         Paediatric population data
│   └── elderly.csv             Elderly population data
├── archive/                    Detailed research-archive CSVs (optional)
│   ├── vitamin_d_healthy_adults.csv
│   ├── vitamin_d_paediatrics.csv
│   └── vitamin_d_elderly.csv
└── README.md
```

The `archive/` directory contains the full research-database CSVs with multiple rows per location (one per seasonal measurement), detailed assay information, and 16 columns. These are the primary data provenance records. The simplified CSVs in `data/` are derived from these for the globe display.

## Data provenance

Every data point traces to a published source via the `full_citation` and `url` fields. The current vitamin D dataset draws on national surveys (NHANES, NDNS, CHMS, DEGS1, KiGGS, KNHANES, ENSANUT, NHES, TILDA, StatusD, Tromsø, LASA, Clalit) and peer-reviewed cohort studies. Assay heterogeneity (RIA, CLIA, ELISA, HPLC, LC-MS/MS) and VDSP standardisation status are documented in the archive CSVs. Cross-country comparisons of absolute values carry an estimated ±10–20 nmol/L systematic uncertainty from assay differences; within-country seasonal amplitudes are more reliable.

## Licence

Data are compiled from published sources cited in each CSV row. Code is provided as-is for research and educational use.
