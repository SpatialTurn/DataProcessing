---
title: "Automating Repeated Steps in Python"
teaching: 35
exercises: 35
---

:::::::::::::::::::::::::::::::::::::: questions

- When is it worth automating a processing workflow?
- How do I turn a sequence of steps into a function, and run it for many inputs?
- How do I make an automated workflow trustworthy?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Write a function with parameters and defaults that packages a geoprocessing workflow
- Test the function against a known result before using it on new inputs
- Use loops and `groupby` to run a workflow for many distances, layers or counties, and collect the results
- Check inputs, report what each run did, and keep one failure from stopping a long loop

::::::::::::::::::::::::::::::::::::::::::::::::

## When to automate

The Geoprocessing episode answered one question: how many people in Lafayette and West Lafayette live within
1 mile of a supermarket? Real projects rarely stop at one answer. A reviewer asks about half a mile; a partner
asks about parks; next year's ACS data arrives; a colleague wants the same numbers for another county. Each
request means running the same steps again with one input changed.

Doing that by hand is slow, but the bigger problem is **mistakes**: a step skipped, an old layer selected in a
drop-down list, a distance typed as 1690 instead of 1609. Automation means writing the steps down once, in a form
the computer can repeat exactly, with the parts that change made into **parameters**.

A useful rule of thumb: the first time, do it by hand and understand it; the second time, notice you are
repeating yourself; the third time, automate. A one-off task is often not worth automating, but it is still
worth recording (see the [Introduction](introduction.md)).

## Levels of automation

| Level | Python | QGIS | Good for |
|---|---|---|---|
| **Record** | The notebook itself | **Processing > History**; a tool's **Advanced > Copy as Python Command** | Seeing exactly what was run, with which settings |
| **Repeat one tool** | A `for` loop around one function call | **Run as Batch Process** (one row per run) | The same tool on many inputs or settings |
| **Package several steps** | A **function** with parameters | A **model** in the Model Designer | A workflow you will run again with different inputs |
| **Run for every feature** | A loop over `groupby` groups or rows | The **Iterate** button (green arrows) next to an input layer | One output per county, per city, per study area |
| **Run without you** | A script run on a schedule or a server | `qgis_process` on the command line | Regular updates, very large jobs |

This episode covers the first four. The two tools mirror each other closely: a Python function and a QGIS model are
both "these steps, with these inputs"; a loop and a batch run are both "again, for each of these".

## Writing steps you can trust

Automated steps run without anyone watching, so they need habits that manual work can do without:

- **Test on a known answer.** Before running a function or model on new inputs, run it on a case you have
  already worked out by hand. Here: 77,473 people within 1 mile of a supermarket, from the Geoprocessing episode.
  If it does not match, find out why before going further.
- **Make the parameters explicit.** Every value that might change (a distance, a layer, a column) becomes a
  parameter with a clear name and, if there is a usual value, a default. Values that should *not* change (the
  projected CRS, the number of buffer segments) stay inside.
- **Check inputs and fail loudly.** A workflow that silently buffers in degrees is worse than one that stops with
  a message. Check the things that commonly go wrong (CRS, expected columns, row counts) at the start, or make the
  workflow fix them itself (the QGIS model reprojects every input).
- **Report what happened.** Print or log the number of rows read, codes replaced, features joined and features
  written. These counts are your evidence that each run worked.
- **Never overwrite inputs.** Write results to new layers or files with systematic names (`near_400.gpkg`,
  `near_800.gpkg`), so a rerun cannot destroy the raw data or an earlier result.
- **Keep one failure from stopping everything, but do not hide it.** When looping over 92 counties, catch an error
  for one county, record it and move on, then read the list of failures at the end.

## Summaries over many units

Running the same summary for every county raises a classic question: how do you combine tract values into a
county value? Add up **counts** first, then compute rates from the sums (households without a car divided by all
households). Averaging tract percentages gives a small rural tract the same weight as a large urban one and
answers a different question. Medians cannot be added up at all: a county median income must come from the
county-level ACS table, not from the tract medians.

::::::::::::::::::::::::::::::::::::: callout

## The same automation in QGIS

The [Automating Repeated Steps in QGIS](automating-qgis.md) episode builds the same workflow as a Model Designer
model and runs it as a batch, with the same results: 6.9%, 23.2% and 67.8% of residents within 400 m, 800 m and
1 mile of a supermarket.

::::::::::::::::::::::::::::::::::::::::::::::::

## The workflow in code

The function, the test and the loop at the heart of notebook 4a:

```python
def pop_within(facilities, distance, tracts=tracts, boundary=city, column="total_pop"):
    """People inside `boundary` within `distance` meters of `facilities` (areal weighting)."""
    t = tracts.copy()
    t["tract_m2"] = t.area
    in_boundary = gpd.clip(t, boundary)
    zone = gpd.GeoDataFrame(geometry=[facilities.buffer(distance).union_all()], crs=t.crs)
    near = gpd.overlay(in_boundary, zone, how="intersection")
    people_near = (near.area / near["tract_m2"] * near[column]).sum()
    people_all = (in_boundary.area / in_boundary["tract_m2"] * in_boundary[column]).sum()
    return round(people_near), round(people_all), round(people_near / people_all * 100, 1)

assert pop_within(supermarkets, 1609)[0] == 77473          # the Geoprocessing episode's answer

rows = []
for d in [400, 800, 1609, 3218]:
    near, total, share = pop_within(supermarkets, d)
    rows.append({"distance_m": d, "people_within": near, "share_pct": share})
pd.DataFrame(rows)
```

Notebook 4b wraps the Census cleaning of the Cleaning episode in a `clean_acs()` function, runs it on all 1,696
Indiana tracts, and summarizes all 92 counties with a loop (and with `groupby().apply()`).

:::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Parameter or not?

You are turning the park access analysis into a function. Which of these should be parameters, which should have
defaults, and which should stay fixed inside the function?

1. The parks layer
2. The walking distance (800 m)
3. The projected CRS (EPSG:26916)
4. The population column (`total_pop`)
5. The number of segments used to draw buffer circles

:::::::::::::::: solution

1. A **parameter** without a default: the function is about *some* set of parks or facilities.
2. A **parameter** with a **default** of 800: the usual case, easy to change for a sensitivity test.
3. **Fixed** (or checked): it is a technical requirement, not a choice the user should make each time. For
   another state you would change it once, or make the function choose a suitable CRS.
4. A **parameter** with a **default** of `total_pop`, so the same function can count children or households.
5. **Fixed**: it affects the result only slightly and should be the same in every run so results are comparable.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: The average of percentages

A loop over Indiana's counties computes each county's share of households without a car as the **mean** of its
tracts' `pct_no_vehicle`. For Tippecanoe County this gives 10.3%, but notebook 4b reports 9.7%. Which is right,
and why do they differ?

:::::::::::::::: solution

9.7% is the share of the county's households: the sum of households without a car divided by the sum of
households. The mean of 42 tract percentages (10.3%) gives every tract the same weight, so small tracts near campus
with very high shares pull it up. It answers "what is the typical tract like?", not "what share of the county's
households have no car?". Add counts first, then divide.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::: challenge

## Challenge 3: A silent failure

A colleague's loop over 92 counties uses `try: ... except: pass` so that "nothing stops the loop". The output
table has 88 rows and no error appeared. What went wrong with the design, and how would you change it?

:::::::::::::::: solution

`except: pass` hides every error, so four counties failed without a trace. Catch the error, but **record** it:
print the county and the message, or append it to a list of failures, and check the count at the end
(`assert len(table) == 92`, or report how many failed). Then fix the cause and rerun.

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

- Automate work you will repeat; record work you will not
- A function packages steps; parameters are the parts that change, defaults the usual values
- Test on a known answer, check inputs, and report counts so every run leaves evidence
- Loops (or `groupby`) run the same workflow for many inputs and collect the results in one table
- Combine units by adding counts first, then computing rates
- Catch errors in long loops, but record them and check the totals at the end

::::::::::::::::::::::::::::::::::::::::::::::::

## Notebooks

| Notebook | What you will do |
|---|---|
| <a href="https://colab.research.google.com/github/SpatialTurn/DataProcessing/blob/main/notebooks/04a_Functions_and_Loops.ipynb" target="_blank">4a. Functions and Loops</a> | Turn the Geoprocessing episode's areal weighting into a function, test it, and loop over distances and facility layers |
| <a href="https://colab.research.google.com/github/SpatialTurn/DataProcessing/blob/main/notebooks/04b_Many_Counties.ipynb" target="_blank">4b. Many Counties</a> | Clean the ACS table for all of Indiana with one function, explain the rows that do not join, summarize 92 counties and save a map per county |
