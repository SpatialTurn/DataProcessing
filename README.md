# GIS Data Processing for Social Scientists

Source for the **Data Processing** module of
[Enabling the Geospatial Turn in the Social Sciences](https://spatialturn.github.io/),
published at <https://spatialturn.github.io/DataProcessing/>.

The lesson is built with [The Carpentries Workbench](https://carpentries.github.io/sandpaper-docs/).
The module was created in October 2026 (Step 9 of the rework): the Geoprocessing and Raster Analysis (now Raster
Processing) episodes moved here from the [Data Analysis module](https://github.com/SpatialTurn/DataAnalysis), and
the Introduction, Cleaning and Automating episodes are new.

## Repository layout

| Folder | Contents |
|---|---|
| `episodes/` | The nine lesson pages shown on the website, with their figures |
| `notebooks/` | The nine Google Colab notebooks linked from the Python episodes (`01a` to `04b`) |
| `data/` | Raw data read directly from GitHub into memory, and the QGIS model |
| `learners/`, `instructors/` | Setup, glossary and instructor notes |

Notebooks open in Colab from
`https://colab.research.google.com/github/SpatialTurn/DataProcessing/blob/main/notebooks/<name>.ipynb`
and read data from `https://raw.githubusercontent.com/SpatialTurn/DataProcessing/main/data/<file>` (and from the Foundations and CaseStudies `data/` folders).

| Episode file | Topic | Notebooks |
|---|---|---|
| `introduction.md` | Introduction to Data Processing (theory) | none |
| `cleaning-python.md`, `cleaning-qgis.md` | Cleaning and Preparing Data | `01a`, `01b` |
| `geoprocessing-python.md`, `geoprocessing-qgis.md` | Geoprocessing | `02a` to `02c` |
| `raster-python.md`, `raster-qgis.md` | Raster Processing | `03a`, `03b` |
| `automating-python.md`, `automating-qgis.md` | Automating Repeated Steps | `04a`, `04b` |

## Data

| File | Contents | Source |
|---|---|---|
| `acs_2024_tippecanoe_tracts_raw.csv` | ACS 2020-2024 5-year estimates for the 43 tracts of Tippecanoe County, exactly as the Census API returned them (`outputFormat=csv`) | U.S. Census Bureau API |
| `acs_2024_indiana_tracts_raw.csv` | The same request for all 1,696 Indiana tracts | U.S. Census Bureau API |
| `indiana_tracts_2024.geojson` | 1,693 Indiana census tracts, 2024 cartographic boundary file (1:500,000), EPSG:4269 | U.S. Census Bureau |
| `ipeds_hd2023_indiana_raw.csv` | The 105 Indiana institutions in the IPEDS directory (HD2023), 19 of its 73 columns, values unchanged | U.S. Department of Education, NCES |
| `osm_parks_tippecanoe_raw.geojson` | 80 OpenStreetMap features tagged `leisure=park` in Tippecanoe County, one per OSM object, geometry not repaired | OpenStreetMap contributors (ODbL), Overpass API, October 2026 |
| `population_within_distance.model3` | The QGIS model built in the Automating episode | This module |

The raw files were written by QGIS (the only route to these services from the build computer), as recorded in
`SpatialTurn_build_scripts/utils/step9/`. The episodes also read clean files from the Foundations module
(`tippecanoe_tracts.geojson`, `tippecanoe_tracts_acs.geojson`, `tippecanoe_acs_2024.csv`, `tippecanoe_parks.geojson`,
`indiana_counties.geojson`, the elevation and land cover rasters) and the Case Studies module
(`grocery_stores.geojson`, `study_area.geojson`).

## Figures and build scripts

All in `SpatialTurn_build_scripts/`:

- Notebooks: `notebook_builders/dp_cleaning.py` (`01a`, `01b`), `dp_geoprocessing.py` (`02a` to `02c`),
  `dp_raster.py` (`03a`, `03b`), `dp_automating.py` (`04a`, `04b`).
- `dp_pipeline.png`: `figure_builders/dp_figures.py`. `da_geoprocessing_ops.png` and `da_raster_ops.png`:
  `da_geoprocessing_figures.py` and `da_raster_figures.py` (named before the move).
- The `cl_qgis_*`, `au_qgis_*`, `gp_qgis_*` and `rs_qgis_*` screenshots were taken in QGIS 4.0 (light and dark UI)
  through the QGIS MCP plugin; `gp_qgis_result.png` and `rs_qgis_result.png` are QGIS renders composed by
  `gp_result_fig.py` and `rs_result_fig.py`.

Every figure has a `-dark` copy for the site's dark mode.

Contact: ilindsay@purdue.edu
