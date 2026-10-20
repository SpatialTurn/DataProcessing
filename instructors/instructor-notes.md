---
title: 'Instructor Notes'
---

## Overall approach

- Every topic after the Introduction comes as a **pair** of episodes, one in Python (Colab) and one in QGIS, with
  the same theory sections, the same data and the same numbers. A mixed group can learn the theory together, split
  by tool for the hands-on part, and compare results at the end.
- Take the episodes in order: Cleaning produces the kind of clean files the later episodes read; Geoprocessing is
  the workflow that Automating packages as a function and a model; Raster Processing reuses Geoprocessing's areal
  weighting.
- The Python episodes link to short Colab notebooks (20 to 35 minutes each). Ask everyone to **Save a copy in
  Drive** before running anything.
- In QGIS, every tool writes a temporary layer. Learners who close QGIS lose their work; remind them to use
  **Make Permanent** or to save outputs to a GeoPackage before a break.
- The notebooks were last tested in October 2026 with pandas 2.3 and GeoPandas 1.2; the QGIS steps with QGIS 4.0.

## Episode by episode

### Introduction to Data Processing

- Theory only (25 minutes plus two challenges). The three rules (never edit raw data, record every step, check
  after every step) come back in every later episode; point them out when they do.
- Challenge 2 (a join that leaves 9 empty rows) is a good place to ask who has had a join fail silently.

### Cleaning and Preparing Data in Python and in QGIS

- The two tools fail in **opposite** directions on the raw ACS CSV: pandas reads the tract code `000100` as the
  number 100; QGIS (through the web, with no `.csvt` file) reads every column as text. Showing both side by side
  makes the point that types must always be checked.
- Shared numbers: 43 tracts; 8 annotation codes (median income and its margin in tracts 104 and 105, median rent
  and its margin in 16.01 and 104); income reliability 21 high and 20 medium; households without a car: CV 40% or
  more in 25 of 42 tracts; `pct_no_vehicle` from 0.2 to 40.7, NULL for tract 104; 43 of 43 joined. Colleges: 105
  institutions, 70 degree-granting, two pairs at shared coordinates (Trine University, Indiana Tech), all in the
  right county. Parks: 80 features, 0 invalid, 10 unnamed, 70 named, 68 after dissolving by name; Ross Hills Park
  143.5 ha.
- Tract 104 (the Purdue residence halls: 8,248 residents, no households) is the best example in the module of a
  value that is missing for a reason. Ask learners what it means for any map of "households".
- The statewide table used in Automating also contains `-333333333` (Hamilton County tract 1110.09, median income
  `250001`, meaning "$250,000 or more").
- QGIS: the Refactor Fields table is fiddly. Learners can also keep the 14 original rows and edit them, deleting
  the ones they do not need. Expressions are easier to type in the expression builder (the ε button).
- QGIS: do not let anyone run **Delete duplicate geometries** on the colleges; it removes two real institutions.
  That is the point of Part 6.

### Geoprocessing in Python and in QGIS

- The two episodes share their theory sections and use the same data, so a mixed group can split by tool
  after the theory and compare numbers at the end (17 supermarkets; tracts selected by intersect / within /
  centroid: 35 / 6 / 24; about 114,000 city residents by areal weighting, about 77,500 or 68% within 1 mile).
- The commonest failure is buffering a layer still in EPSG:4326. In QGIS the Buffer dialog then shows
  "degrees" and a warning sign; in Python GeoPandas prints a geographic CRS warning. Show both on purpose.
- QGIS Buffer defaults to 5 segments; the episode uses 16 to match GeoPandas. With 5, sums differ by well
  under 1%.
- Challenge 3 (areal weighting on the Purdue tract) works well as a pair discussion and sets up the Raster
  Processing episode (land cover as ancillary data).

### Raster Processing in Python and in QGIS

- Shared numbers: 86.9 km2 below 170 m; groups 10.8 / 196.8 / 167.1 / 904.0 / 23.4 km2; developed below 170 m
  10.7 km2 (5%); tract 1 mean elevation 165.6 m; percent developed min 4.8% (tract 110), median 82%;
  correlation with log density 0.89; dasymetric city population 122,098 vs 114,345 areal, share within 1 mile
  68.3% vs 67.8%.
- QGIS Reclassify by table defaults to `min < value <= max`, which silently drops codes equal to the minimum
  (11, 21, 31...). Make learners set `min <= value <= max` and check with the unique values report.
- The Raster Calculator OK button stays gray until an output file is set (or on-the-fly is ticked).
- Colab needs `pip install rasterstats` (in notebook 3b).
- "Below 170 m" is a teaching stand-in, not a flood map; say so.

### Automating Repeated Steps in Python and in QGIS

- Shared numbers (Python function and QGIS model): residents within 400 m / 800 m / 1 mile of a supermarket
  6.9% / 23.2% / 67.8% (7,913 / 26,579 / 77,473 of 114,345); within 2 miles 99.3% (Python only); within 800 m of a
  park 71.5%; children within 1 mile of a supermarket 14,797 of 19,540 (75.7%).
- Insist on the test against the known answer (77,473) before anything new is run. It is the habit the episode
  exists to teach.
- QGIS Model Designer: inputs become variables named after their description without spaces or punctuation
  (`Population field` is `@populationfield`). If the formula gives NULL everywhere, the name is the first thing
  to check. The finished model is in `data/population_within_distance.model3` for anyone who falls behind.
- QGIS batch runs need a name in every output cell, then **Create Temporary Layer** from the cell's **...** menu
  (or files; when saving to files, **Fill with parameter values** builds names from the Distance column).
- Python notebook 4b: Indiana has 1,696 ACS tracts but 1,693 tract boundaries; the three without boundaries are
  the Lake Michigan water tracts (codes ending in `990000`). The top three counties for households without a car
  are LaGrange, Daviess and Adams (large Amish communities); Tippecanoe is fifth and Marion tenth. Expect
  questions.
- Challenge 2 (mean of tract percentages, 10.3%, versus share of the county's households, 9.7%) is worth doing as
  a class.

## Common problems

| Symptom | Likely cause | Fix |
|---|---|---|
| `NameError: name 'x' is not defined` | A cell was skipped | **Runtime > Run before** |
| A join matches 0 rows | One key is text and the other a number, or zeros were lost | Check `dtypes`; read IDs with `dtype=str` |
| `HTTPError` or a timeout while reading data | Network hiccup reading from GitHub | Rerun the cell |
| `ModuleNotFoundError: rasterstats` | The `!pip install` cell did not run | Run it, then rerun the imports |
| QGIS: an expression gives NULL in every row | A field name in double quotes is misspelled, or a text field needs `to_int()` | Check the field list in the expression builder |
| QGIS asks to choose a transformation | Layers in EPSG:4326 drawn in an EPSG:26916 project | Keep the first option and click **OK** |
| QGIS model output is empty | A step's input is set to the wrong source (a fixed value instead of an algorithm output) | Open the step in the Model Designer and check each input's source |
