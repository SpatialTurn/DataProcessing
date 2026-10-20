---
title: 'Reference'
---

## Glossary

Annotation code
:   A special value such as `-666666666` that the Census API writes instead of an estimate (or margin of error)
    that does not exist. It must be turned into a missing value before any calculation.

Areal weighting
:   Splitting a polygon's count (such as population) among the pieces it is cut into, in proportion to area.
    It assumes the count is spread evenly across the polygon.

Batch processing
:   Running one QGIS tool or model several times with different settings, one row per run.

Buffer
:   A zone at a fixed distance around features. Distances need a CRS in meters or feet.

Clip
:   Cutting a layer to the shape of another layer, keeping only the first layer's attributes.

Codebook (data dictionary)
:   The document that says what each column and each code in a dataset means.

Coefficient of variation (CV)
:   The standard error of an estimate as a percentage of the estimate: (MOE / 1.645) / estimate x 100. A common
    rule of thumb: under 12% high reliability, 12 to 40% medium, over 40% low.

Coordinate Reference System (CRS)
:   The system that links coordinates to places on Earth. EPSG:4326 is latitude and longitude; EPSG:26916 (UTM
    zone 16N) uses meters and suits Indiana.

Dasymetric estimation
:   Areal weighting refined with other data, such as land cover, so that counts go where people actually live.

Define versus reproject
:   Defining a CRS states what the coordinates already are (no numbers change); reprojecting converts them to
    another CRS (every number changes).

Dissolve
:   Merging features into one, or into one per value of a column, and combining their attributes.

Function
:   Named, reusable code with parameters (inputs) and a return value (output).

GEOID
:   The Census Bureau's identifier for a geographic area. A tract GEOID has 11 characters: state (2), county (3)
    and tract (6). It must be stored as text so leading zeros are kept.

Intersect
:   Cutting two layers against each other and keeping the overlapping pieces, with the attributes of both.

Invalid geometry
:   A shape that breaks the rules of its type, such as a polygon whose edges cross itself. Overlays can fail or
    give wrong areas.

Map algebra
:   Calculations on rasters cell by cell, such as `elevation < 170`.

Margin of error (MOE)
:   For ACS estimates, the range around the estimate that contains the true value with 90% confidence.

Model (QGIS)
:   A chain of Processing tools saved as a new tool, with inputs and outputs, built in the Model Designer.

Raster
:   A grid of cells, each holding a value, such as elevation or a land cover code.

Reclassification
:   Mapping old raster values to new ones with a table, for example 16 land cover codes into 5 groups.

Refactor Fields
:   A QGIS tool that creates a new layer with fields renamed, retyped or computed from expressions.

Zonal statistics
:   Summaries of the raster cells inside each polygon, such as the mean elevation of each tract.

## Library and tool documentation

- [pandas: working with missing data](https://pandas.pydata.org/docs/user_guide/missing_data.html)
- [GeoPandas](https://geopandas.org/)
- [rasterio](https://rasterio.readthedocs.io/) and [rasterstats](https://pythonhosted.org/rasterstats/)
- [QGIS User Manual: the Model Designer](https://docs.qgis.org/latest/en/docs/user_manual/processing/modeler.html)
- [QGIS User Manual: batch processing](https://docs.qgis.org/latest/en/docs/user_manual/processing/batch.html)
- [Census Bureau: ACS notes on annotation values](https://www.census.gov/data/developers/data-sets/acs-1year/notes-on-acs-estimate-and-annotation-values.html)
- [Census Bureau: Understanding and Using ACS Data (margins of error)](https://www.census.gov/programs-surveys/acs/library/handbooks/general.html)
