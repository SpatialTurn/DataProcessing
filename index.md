---
site: sandpaper::sandpaper_site
---

Data rarely arrives ready to analyze. This module teaches the everyday work between collecting data and
analyzing it: **cleaning** raw tables and layers, **geoprocessing** (buffers, overlays, distances and areal
weighting), **raster processing** (map algebra, reclassification and zonal statistics) and **automating** the
steps you repeat. It is written for social scientists and other beginners who have finished the
[Foundations](https://spatialturn.github.io/Foundations/) module.

Each topic is taught twice, once in **QGIS** (point and click) and once in **Python** (Google Colab), with the
same data and the same results. Follow either one, or both. All examples use Tippecanoe County, Indiana
(Lafayette and West Lafayette) and Indiana as a whole.

| Episode | Question it answers | Tools |
|---|---|---|
| Introduction to Data Processing | What happens between download and analysis, and how do I keep it repeatable? | Concepts |
| Cleaning and Preparing Data (Python / QGIS) | How do I fix identifiers, types, special codes, duplicates and geometry problems? | `pandas`, `geopandas` / Refactor Fields, Check Validity |
| Geoprocessing (Python / QGIS) | What lies within a distance, inside a boundary, or nearest? | `geopandas` / QGIS Vector menu |
| Raster Processing (Python / QGIS) | How do rasters combine, and what do they say about each tract? | `rasterio`, `rasterstats` / Raster Calculator, Zonal statistics |
| Automating Repeated Steps (Python / QGIS) | How do I run the same workflow for many inputs without mistakes? | Functions and loops / Model Designer, batch processing |

Start with the Introduction, then take the episodes in order: each one uses results or ideas from the one
before. The advanced methods (network analysis, spatial autocorrelation, clustering and NDVI) follow in the
[Data Analysis](https://spatialturn.github.io/DataAnalysis/) module.

::::::::::::::::::::::::::::::::::::::::::: prereq

## Prerequisites

- For the Python episodes: a Google account, and basic Python, `pandas` and GeoPandas (Foundations notebooks 2a
  to 4c)
- For the QGIS episodes: QGIS installed, and the Foundations QGIS episodes (Getting Started with QGIS; Tables,
  Joins and Selections)
- No Census API key is needed: the Census tables are provided as downloaded

New to Python, Colab or QGIS? Start with the [Foundations](https://spatialturn.github.io/Foundations/) module,
then read the [Setup](learners/setup.md) page.

::::::::::::::::::::::::::::::::::::::::::::::::::

This module is part of [Enabling the Geospatial Turn in the Social Sciences](https://spatialturn.github.io/),
a set of training modules developed at Purdue University with support from the National Science Foundation.
