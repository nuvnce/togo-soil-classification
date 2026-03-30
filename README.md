# Togo Soil Classification

> Supervised classification of agricultural soil types across the 599 administrative cantons of Togo,
> combining SoilGrids v2.0 (ISRIC/FAO), Google Earth Engine Sentinel-2 imagery,
> Random Forest machine learning, and an interactive Leaflet web map.

**Author:** Daniel ESSONANI  
**Supervisor:** [M. Leri TCHANTCHO](https://www.linkedin.com/in/leri-damigouri-tchantcho-28503873/)

**Date:** February 2026

---

## 🗺️ Live Map

**[View the interactive map →](...)**

Each of the 599 cantons is clickable and shows:
- Dominant WRB soil type
- Fertility level
- Recommended crops
- Main agricultural constraints

---

## Project Overview

Togo's agricultural sector employs ~65% of the active population but suffers from a
critical lack of up-to-date soil spatial data — existing pedological maps date from the
1970s–80s and cover only part of the territory.

This project addresses that gap by building a **national-scale soil classification pipeline**
using freely available satellite data and open-source tools, producing results directly
usable by agricultural planners and decision-makers.

The work also serves as the **complementary validation component** of a larger academic
study on ERDAS IMAGINE supervised classification of soil types in Togo
(*"La classification supervisée des types de sols agricoles au Togo grâce à ERDAS IMAGINE"*,
Daniel ESSONANI, NONCHALENT INC., February 2026).

---

## Technical Stack

| Tool | Role |
|------|------|
| Python 3 + Jupyter | Data processing pipeline |
| SoilGrids v2.0 REST API (ISRIC/FAO) | WRB soil type reference labels |
| Google Earth Engine Python API | Sentinel-2 spectral feature extraction |
| scikit-learn | Random Forest classifier |
| GeoPandas | Shapefile processing and spatial joins |
| QGIS + qgis2web | Cartographic styling and Leaflet export |
| Leaflet.js | Interactive web map |

---

## Repository Structure

```
togo-soil-classification/
│
├── README.md
├── rapport_complet_togo_sols.docx      ← Full academic report (87 pages, French)
│
├── map/                                ← Interactive Leaflet map (full qgis2web export)
│   ├── index.html
│   ├── css/
│   ├── data/                           ← GeoJSON canton data
│   ├── js/
│   ├── legend/
│   └── ...
│
└── notebooks/
    ├── 01_soilgrids_extraction.ipynb
    ├── 02_agronomic_reference_table.ipynb
    ├── 03_shapefile_join_enrichment.ipynb
    ├── 04_gee_sentinel2_extraction.ipynb
    └── 05_random_forest_classification.ipynb
```

---

## Notebooks

### `01` — SoilGrids API Extraction
Queries the SoilGrids v2.0 REST API for each of the 599 canton centroids to retrieve
the dominant WRB soil type and top-3 class probabilities. Includes a **resume mechanism**
that picks up from any interruption point without losing previous results.

### `02` — Agronomic Reference Table
Builds a lookup table mapping each WRB soil class to fertility level, recommended crops,
and main agricultural constraints, based on FAO guidelines and West African soil literature.

### `03` — Shapefile Join & Enrichment
Merges the cantons shapefile with SoilGrids results and the agronomic reference table
into a single enriched GeoDataFrame. Handles ESRI Shapefile column name constraints
(≤10 characters) and exports the final `cantons_togo_sols.shp`.

### `04` — Sentinel-2 Extraction via GEE
Builds a dry-season median composite (Nov 2023 – Mar 2024) and extracts 13 spectral
features per canton centroid using `reduceRegions()` in batches of 50 — the approach
that solved GEE computation timeouts encountered with individual point queries.

### `05` — Random Forest Classification
Trains a 200-tree Random Forest on the 13 Sentinel-2 features with SoilGrids labels
as ground truth. Reports accuracy, cross-validation, confusion matrix, feature importance,
and the concordance metric between RF predictions and SoilGrids reference.

---

## Results

| Metric | Value |
|--------|-------|
| Cantons covered | 599 |
| Valid cantons for RF training | 586 |
| WRB soil types identified | 9 |
| RF test accuracy | 43.2% |
| 5-fold cross-validation | 31.4% ± 6.4% |
| **RF ↔ SoilGrids concordance** | **86.6%** |
| Mean prediction confidence | 68.5% |
| Discordant cantons | 80 (13.4%) |

The 43.2% accuracy is consistent with spectral soil classification literature for
tropical Africa (typical range: 35–55% without auxiliary topographic or geological data).
The **86.6% concordance** is the key result: the model trained purely on satellite
spectral data successfully reproduces the broad pedogeographic patterns of the
FAO/ISRIC reference database.

The most discriminant features were **B11 (SWIR1)** and **NDWI** — consistent with
the SWIR/NIR/Red false-color composite chosen in the parallel ERDAS IMAGINE study
for visual soil discrimination.

### WRB Soil Types Found in Togo (599 cantons)

| Soil Type | Cantons | % |
|-----------|---------|---|
| Lixisols | 119 | 20.2% |
| Acrisols | 115 | 19.5% |
| Luvisols | 102 | 17.3% |
| Cambisols | 98 | 16.6% |
| Gleysols | 74 | 12.5% |
| Vertisols | 65 | 11.0% |
| Fluvisols | 13 | 2.2% |
| Arenosols | 1 | 0.2% |

---

## Known Limitations

- **No field validation data** — SoilGrids predictions are used as proxy ground truth,
  not actual field observations. The 80 discordant cantons are priority targets for
  future field campaigns.
- **Single-date imagery** — the dry-season composite captures bare soil well but misses
  seasonal soil dynamics (waterlogging, cracking vertisols, etc.)
- **Canton-level resolution** — results are averaged per administrative unit; fine-scale
  within-canton variability is not captured.

---

## How to Run

### Prerequisites
```bash
pip install geopandas pandas requests scikit-learn matplotlib seaborn nbformat
pip install earthengine-api
earthengine authenticate   # run once — opens browser for GEE login
```

### Run the pipeline
Open each notebook in order in VSCode or JupyterLab and update the file paths
in the first code cell to match your local data location.

```
notebooks/
  01 → produces  sols_togo_cantons.csv
  02 → produces  table_reference_sols.csv
  03 → produces  cantons_togo_sols.shp
  04 → produces  sentinel2_599_cantons.csv
  05 → produces  resultats_classification.csv + figures
```

---

## License

This project is for academic purposes.  
Data sources: SoilGrids (ISRIC, CC BY 4.0), Sentinel-2 (ESA Copernicus, free & open).
