# Zurich Infrastructure Reports - A Spatial Analysis of ZueriWieNeu

A reproducible spatial analysis of reported infrastructure issues in Zurich, investigating where issues are reported, which types of issues dominate, and how long the city takes to resolve them across neighborhoods (Quartiere).

Course: SDS210
Author: M. Rausch
Date: May 2026

## Research question

What spatial, temporal, and categorical patterns characterise reports of infrastructure problems in Zurich?

The notebook breaks this down into four sub-questions:

- Q1 - Which infrastructure issues are reported the most?
- Q2 - Which neighborhood reports the most (per 1000 residents)?
- Q3 - Are certain issue types concentrated in particular neighborhoods?
- Q4 - How quickly are issues resolved, by category (Q4.1, Q4.2) and by neighborhood (Q4.3)?

## Data sources

All three datasets come from the Stadt Zuerich Open Data portal (https://data.stadt-zuerich.ch/).

| File (expected in `data/`) | Description | Source |
|---|---|---|
| `data_zwn.gpkg` | ZueriWieNeu citizen reports (point geometries, EPSG:2056) | Stadt Zuerich - ZueriWieNeu |
| `data_qnr.gpkg` (layer: `stzh.adm_statistische_quartiere_v`) | Statistical neighborhood polygons (EPSG:2056) | Stadt Zuerich - Statistische Quartiere |
| `data_pop.csv` | Resident population by neighborhood and year | Stadt Zuerich - Bevoelkerung nach Quartier |

Both spatial datasets use the Swiss CRS EPSG:2056 (LV95). The folium heatmap reprojects to WGS84 on the fly.

Data files are not tracked in Git (see `.gitignore`). To rerun the notebook, download the three files above and place them in a local `data/` folder using the filenames in the table.

## Key assumptions

These analytical choices shape every figure in the notebook and should be understood before interpreting the results:

- Working CRS: all spatial operations use EPSG:2056 (CH1903+ / LV95). Only the folium heatmap reprojects to WGS84 (EPSG:4326), as required by the basemap.
- "Resolved" definition: a report is counted as resolved only when `status == "fixed - council"`. Other terminal statuses are excluded from resolution-time analysis.
- Resolution timestamp: `resolved_datetime` is derived from `updated_datetime`, which records administrative closure of a case, not the moment of physical repair.
- Population snapshot: reports are normalised against the 2025 resident population (`StichtagDatJahr == 2025`) from the city's population register.
- Minimum group size: the `resolution_table` helper drops any group (category or neighborhood) with fewer than 30 resolved cases, since smaller samples produce unstable medians and means.
- Spatial join predicate: reports are assigned to neighborhoods using `gpd.sjoin(..., predicate="within")`.

## Repository structure

```text
.
├── README.md
├── .gitignore
├── environment.yml
├── data/                   # not tracked - download manually (see above)
│   ├── data_zwn.gpkg
│   ├── data_qnr.gpkg
│   └── data_pop.csv
├── notebooks/
    └── analysis.ipynb      # the full analysis notebook
```

## Setup

Requires Python 3.11+ with `pandas`, `geopandas`, `matplotlib`, `folium`, `mapclassify`, `jupyterlab` (see `environment.yml`).

Download the three datasets listed above into `./data/`, then run the notebook top to bottom.

## How to run

The project is a single notebook. From the top, run Kernel -> Restart & Run All in Jupyter.

Cells are ordered so the workflow runs sequentially: function definitions, libraries, load data, clean and join, analysis (Q1 to Q4.3). All file paths are relative to the repository root, so the notebook will work on any machine as long as the structure above is preserved.

## Method in brief

1. Load the reports (point geometries), neighborhood polygons, and the 2025 population table.
2. Clean: drop unused columns, rename a handful for clarity (`service_name` to `CATEGORY`, `e/n` to `easting/northing`, etc.), compute neighborhood area.
3. Spatial join reports to neighborhoods (`gpd.sjoin`, predicate `within`), then merge population by `qnr`.
4. Parse the three datetime columns and derive `resolved_datetime` from `updated_datetime` where `status == "fixed - council"`.
5. Analyse with a small reusable helper (`resolution_table`) that computes median, mean, count, and resolution rate for any grouping column, dropping groups with n < 30 to avoid unstable estimates.
6. Visualise with a mix of static matplotlib choropleths/bar charts and one interactive folium heatmap for spatial EDA.

## Outputs

- Figure A - Interactive folium heatmap of report density across Zurich.
- Figure B - Monthly report volume, 2013 to 2026 (temporal EDA).
- Figure 1 - Reports per category (horizontal bar chart).
- Figure 2 - Side-by-side choropleths: raw report counts vs reports per 1000 residents.
- Figure 3 - 3x3 small-multiples choropleth showing each category's spatial signature.
- Figure 4.1 / 4.2 - Resolution-time summary table and bar chart (median bars, mean dots to expose right-skew).
- Figure 4.3 - Choropleth of median resolution days by neighborhood.

## Known limitations

- `updated_datetime` records administrative closure of a case, not the moment of physical repair. Median resolution times under one day therefore describe back-office processing speed, not how fast issues are actually fixed.
- "Reports per 1000 residents" inflates central neighborhoods like City, which have low residential populations but heavy daytime footfall.
- Q3 panels use row-percentages, so neighborhoods with very few total reports can still appear dark on a category's map.
- Only reports with `status == "fixed - council"` are counted as resolved; other terminal statuses are excluded from resolution-time analysis.

## License

Course project. Code shared for educational and grading purposes. Data remains under the original Stadt Zuerich Open Data terms of use.
