---
title: "Cleaning and Preparing Data in Python"
teaching: 40
exercises: 35
---

:::::::::::::::::::::::::::::::::::::: questions

- What goes wrong when raw data is used as it was downloaded?
- How do I find and fix identifiers, types, special codes, duplicates and geometry problems in pandas and
  GeoPandas?
- How reliable is an American Community Survey estimate?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Recognize the common problems of raw tables and layers, and choose a fix for each
- Keep identifiers as text, convert types, and turn special codes into missing values
- Judge the reliability of ACS estimates with the coefficient of variation
- Create points from coordinates, recode categories, and decide what counts as a duplicate
- Check and repair geometries, and confirm each cleaning step with a count, a join or a map

::::::::::::::::::::::::::::::::::::::::::::::::

## Why cleaning comes first

Every downloaded dataset carries the habits of the system that produced it: the Census Bureau's codes for "no
estimate", a federal directory's codes for "not applicable", a volunteer map's unnamed and split features. None of
these are errors in the source; they are conventions you have to know. But if you analyze the data without
handling them, the software treats a code as a number, a split park as two parks, and a missing identifier as a
mismatch, and it does so **without any warning**.

This episode cleans three raw files you will meet again later in the curriculum:

| File | Raw problems you will fix |
|---|---|
| ACS 2024 table for the 43 census tracts of Tippecanoe County, exactly as the Census API returned it | Identifiers split in three parts; numbers or text depending on the tool; codes such as -666666666; margins of error; a tract with no households |
| IPEDS directory of the 105 colleges and schools in Indiana (2023) | Coordinates in two columns; numeric category codes; `-2` for "not applicable"; ZIP and ZIP+4 mixed; two pairs of colleges at the same address |
| OpenStreetMap parks in Tippecanoe County, one feature per OSM object | Unnamed features; parks split into two features; possible invalid geometries |

## The common problems

| Problem | What it looks like | Why it matters | Fix |
|---|---|---|---|
| **Identifiers read as numbers** | Tract `000100` becomes `100`; ZIP `02138` becomes `2138` | Joins fail or match the wrong rows | Read the column as text; rebuild with padding (`zfill`, `lpad`) |
| **Numbers read as text** | `"53488"` sorts before `"9000"`; sums are impossible | Sorting, statistics and maps go wrong | Convert the type (`to_numeric`, `to_int`) |
| **Codes that mean "missing"** | `-666666666`, `-2`, `9999`, `NA`, an empty string | Averages and maps include impossible values | Turn them into real missing values (`NaN`, `NULL`) and count them |
| **Categories stored as codes** | `SECTOR = 2` | Unreadable tables and legends | Recode with the codebook (data dictionary) |
| **Messy text** | Trailing spaces, `Purdue Univ.` and `Purdue University` | Groups and joins split one thing into two | `strip`/`trim`, consistent case, a lookup table |
| **Duplicates** | The same record twice, or two features for one object | Counts and totals inflated | Decide what makes two records the same (an ID), then remove or merge |
| **Invalid geometry** | A polygon whose edges cross itself | Overlays fail or give wrong areas | Check validity; repair with `make_valid` or Fix Geometries |
| **Wrong or missing CRS** | Points near (0, 0), or a layer that lands in the ocean | Everything spatial is wrong | *Define* the CRS the coordinates are really in, then *reproject* |

Two fixes in that table are easy to confuse. **Defining** a CRS tells the software what the coordinates already
are (it changes no numbers); **reprojecting** converts the coordinates to another CRS (every number changes). If a
layer has no CRS, define it; if it has the right CRS but you need meters, reproject it. Defining the wrong CRS
moves the layer to the wrong place on Earth.

## Census annotation codes

The Census Bureau's API returns numbers in every estimate column, so where no estimate exists it writes a large
negative **annotation code** instead:

| Code | Meaning |
|---|---|
| `-666666666` | The estimate could not be computed (too few sample cases, or nothing to compute, such as median household income where there are no households) |
| `-999999999` | The estimate or margin of error cannot be shown (too few sample cases) |
| `-888888888` | Not applicable |
| `-222222222` | The margin of error could not be computed (too few sample observations) |
| `-333333333` | The margin of error could not be computed, because the median is in an open-ended top or bottom range (median income `250001` means "$250,000 or more") |
| `-555555555` | The estimate is controlled (fixed by design), so its margin of error is zero |

In Tippecanoe County, Census Tract 104 has 8,248 residents and **no households**: it is the Purdue residence
halls, where everyone lives in group quarters. Its median household income and median rent are both
`-666666666`, and any rate with households as the denominator is undefined (a division by zero). Two other tracts
have too few sample cases for one of the medians.

## Margins of error and reliability

The American Community Survey asks a sample of households, so every number is an **estimate** with a **margin of
error (MOE)**: with 90% confidence the true value lies between estimate minus MOE and estimate plus MOE. To
compare how reliable different estimates are, divide the standard error (MOE / 1.645) by the estimate:

$$CV = \frac{MOE / 1.645}{\text{estimate}} \times 100$$

The **coefficient of variation (CV)** is the error as a percentage of the estimate. A widely used rule of thumb:
under 12% is high reliability, 12 to 40% medium, over 40% low.

Small counts are the usual problem. In Tippecanoe County, median household income has medium or high
reliability in every tract, but the number of **households without a car** has a CV over 40% in 25 of the 42
tracts that have households. That does not make the variable useless, but it limits what you can say: map it in
a few broad classes, add up tracts into larger areas, report the margins, and do not rank single tracts.

## Rows you remove are decisions

Cleaning includes choices that are not fixes: keeping only degree-granting colleges, dropping unnamed park
features, leaving out the residence-hall tract from a rate. Each is reasonable, and each changes the result.
Write them down, with the number of rows affected, in your processing log (see the
[Introduction](introduction.md)).

## Check after every step

| Step | Check |
|---|---|
| Reading the file | Row count matches the source (43 tracts, 105 institutions, 80 park features) |
| Fixing identifiers | GEOIDs have 11 characters; every row joins to a boundary |
| Turning codes into missing | Count of codes before (8) and after (0); count of missing values |
| Filtering and merging | Rows before and after (105 to 70 colleges; 80 to 70 to 68 parks) |
| Making points | The points land in Indiana, in the county their record names |
| Comparing with a trusted copy | The cleaned files match those the Foundations module uses |

::::::::::::::::::::::::::::::::::::: callout

## The same cleaning in QGIS

The [Cleaning and Preparing Data in QGIS](cleaning-qgis.md) episode cleans the same three files with the
Refactor Fields, Create Points Layer from Table, Check Validity and Dissolve tools, and reaches the same numbers.

::::::::::::::::::::::::::::::::::::::::::::::::

## The workflow in code

The two notebooks go step by step. The core of the Census table cleaning:

```python
import pandas as pd

raw = pd.read_csv(url, dtype={"state": str, "county": str, "tract": str})   # identifiers as text
raw["GEOID"] = raw["state"] + raw["county"] + raw["tract"]                   # 18 + 157 + 000100

acs = raw.rename(columns={"B19013_001E": "median_income", "B19013_001M": "median_income_moe"})
num = acs.select_dtypes("number").columns
print("codes:", acs[num].lt(-100_000_000).sum().sum())                       # count before
acs[num] = acs[num].mask(acs[num] < -100_000_000)                           # codes -> NaN

acs["income_cv"] = (acs["median_income_moe"] / 1.645) / acs["median_income"] * 100
acs["pct_no_vehicle"] = acs["hh_no_vehicle"] / acs["households"].where(acs["households"] > 0) * 100
```

And of the points and polygons:

```python
import geopandas as gpd

colleges = gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df["LONGITUD"], df["LATITUDE"]), crs="EPSG:4326")
print((~parks.is_valid).sum(), "invalid geometries")
parks = parks[parks["name"].notna()].dissolve(by="name", as_index=False)   # drop unnamed, merge split parks
```

:::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: What went wrong?

A colleague joins a Census table to tract boundaries in pandas and gets zero matches. The table's `GEOID` column
shows values like `18157000100` and the boundaries show `18157000100` too. What is the most likely cause, and
how do you confirm it?

:::::::::::::::: solution

One side holds the GEOID as a **number** and the other as **text**; pandas does not match `18157000100` to
`"18157000100"`. Confirm with `df.dtypes` (or `df["GEOID"].map(type).value_counts()`) on both sides. Fix it by
reading the column as text (`dtype={"GEOID": str}`), so leading zeros are kept for states such as Alabama
(`01`) too.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Rank the tracts?

A news story wants to name "the five tracts in Tippecanoe County with the most households without a car". The
raw table gives these five estimates, with their margins of error (MOE):

| Tract | Households without a car | MOE |
|---|---|---|
| 55 | 909 | 289 |
| 4 | 666 | 291 |
| 112 | 425 | 168 |
| 102.08 | 411 | 211 |
| 111 | 384 | 179 |

Is the ranking sound? What would you suggest?

:::::::::::::::: solution

Not as a strict ranking. The 90% intervals overlap widely: tract 111 could have as many as 384 + 179 = 563
households without a car, and tract 4 as few as 666 - 291 = 375, so even second and fifth place could swap, and
places 3 to 5 are hard to tell apart. The sixth tract (7, with 341 plus or minus 164) could belong in the list too.
Say that these tracts have *among the highest* numbers, report the margins, or test whether two estimates differ
significantly (the Census Bureau publishes a simple formula for that). Tract 55 stands out, but its interval
(620 to 1,198) still overlaps tract 4's (375 to 957).

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::: challenge

## Challenge 3: Define or reproject?

You receive a CSV of community centers with columns `x` and `y` holding values such as `506512` and `4475023`, and
no CRS. The data provider says they are "UTM zone 16N, NAD83". What do you do in GeoPandas?

:::::::::::::::: solution

The values are in meters, not degrees, so the CRS is **EPSG:26916**. **Define** it when you create the points:
`gpd.GeoDataFrame(df, geometry=gpd.points_from_xy(df["x"], df["y"]), crs="EPSG:26916")`. Do not use
`to_crs()` for this: it would try to convert coordinates that have no CRS yet (GeoPandas refuses). Reproject
afterwards only if you need another CRS, for example `to_crs(epsg=4326)` for a web map.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- Inspect before changing anything: shape, types, minimum and maximum reveal most problems
- Read identifiers as text, convert numbers stored as text, and turn special codes into missing values
- ACS estimates have margins of error; the coefficient of variation shows which ones are too uncertain to rank
- Decide what makes two records the same before removing duplicates
- Define a CRS to say what coordinates are; reproject to change them
- Check every step with a count, a join or a map, and write your decisions down

::::::::::::::::::::::::::::::::::::::::::::::::

## Notebooks

| Notebook | What you will do |
|---|---|
| <a href="https://colab.research.google.com/github/SpatialTurn/DataProcessing/blob/main/notebooks/01a_Cleaning_a_Census_Table.ipynb" target="_blank">1a. Cleaning a Census Table</a> | Read the raw ACS table, build GEOIDs, turn codes into missing values, compute reliability and a safe rate, and check with a join and a map |
| <a href="https://colab.research.google.com/github/SpatialTurn/DataProcessing/blob/main/notebooks/01b_Cleaning_Points_and_Polygons.ipynb" target="_blank">1b. Cleaning Points and Polygons</a> | Make college points from coordinates, recode categories, find shared locations, check and merge OpenStreetMap parks |
