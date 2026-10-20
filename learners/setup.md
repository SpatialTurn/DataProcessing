---
title: Setup
---

The Python episodes run in **Google Colab**, a free online notebook service, so you do not need to install
Python. The QGIS episodes need QGIS on your computer; follow the
[Foundations setup](https://spatialturn.github.io/Foundations/index.html#part-2-install-qgis) to install it.
No Census API key is needed in this module.

## What you need

- A Google account (Python episodes)
- QGIS 3.34 or later; the episodes were checked in QGIS 4.0 (QGIS episodes)
- A web browser (Chrome, Firefox, Edge or Safari)

## Opening a notebook

1. Click a notebook link in the **Notebooks** table at the end of any Python episode. It opens in Colab.
2. Click **File > Save a copy in Drive**. Your own copy keeps your changes; the original stays read only.
3. Run cells from top to bottom. Click a cell and press **Shift + Enter**, or click the play button to its left.
4. The first time you run a cell, Colab may warn that the notebook was not written by Google. Click
   **Run anyway**.

::::::::::::::::::::::::::::::::::::::::::: callout

### Colab tips

- **Run cells in order.** Most errors come from skipping a cell, for example the one that reads the data.
- **Start over cleanly** with **Runtime > Restart session and run all**.
- Data lives in memory and files you write (such as the county maps in notebook 4b) live in the session; download
  them from the Files panel on the left before the session ends.
- Cells that start with `!pip install` add libraries Colab does not include by default (`rasterstats` in the
  Raster Processing notebooks). They take a few seconds and only need to run once per session.

:::::::::::::::::::::::::::::::::::::::::::::::::::

## Loading data in QGIS

The QGIS episodes load every layer from a web address: **Layer > Add Layer > Add Vector Layer** (or **Add Raster
Layer**), **Source Type: Protocol: HTTP(S), cloud, etc.**, **Type: HTTP/HTTPS/FTP** (in QGIS 4 the Type list starts
with cloud services, so check it), and paste the address. Layers loaded this way are read only: tools write their
results to new temporary layers, which disappear when QGIS closes unless you save them (right-click >
**Make Permanent**).

## Data

You never need to download data by hand. Every notebook and QGIS episode reads its data from the web.

| Episode | Data | Source |
|---|---|---|
| Cleaning | Raw ACS 2024 table for Tippecanoe County tracts; raw IPEDS directory of Indiana colleges; raw OpenStreetMap parks | U.S. Census Bureau API; NCES; OpenStreetMap |
| Geoprocessing | Lafayette food stores and city limits; Tippecanoe County tracts with 2024 ACS estimates | OpenStreetMap; U.S. Census Bureau (pre-fetched) |
| Raster Processing | Tippecanoe County elevation (USGS 3DEP, 30 m) and land cover (NLCD 2021), with tracts and 2024 ACS estimates | USGS; Multi-Resolution Land Characteristics Consortium; U.S. Census Bureau |
| Automating | The Geoprocessing layers and parks; raw ACS 2024 table and tract boundaries for all of Indiana | As above; U.S. Census Bureau |

## Working on your own computer (optional)

To run the notebooks locally, install [Anaconda](https://www.anaconda.com/download) and create an environment:

```bash
conda create -n spatialturn -c conda-forge python=3.11 geopandas rasterio rasterstats mapclassify matplotlib jupyterlab
conda activate spatialturn
jupyter lab
```

Then download a notebook from GitHub (**File > Download > Download .ipynb** in Colab) and open it in JupyterLab.
The data is still read from the web, so you need an internet connection.
