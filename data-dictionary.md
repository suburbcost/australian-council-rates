# Data dictionary

**Dataset** Australian council-level household running cost dataset 2026-27
**Version** 2026-27+8fc219d8
**Rows** 148, one per council
**Licence** CC BY 4.0, https://creativecommons.org/licenses/by/4.0/
**Canonical page** https://www.suburbcost.com.au/dataset/
**Files** two, described below

| File | Councils | Covers | Columns |
| --- | --- | --- | --- |
| [`suburbcost-council-running-costs.csv`](https://www.suburbcost.com.au/dataset/suburbcost-council-running-costs.csv) | 148 | QLD, NSW | rates, water, commute, insurance, body corporate, quality flags |
| [`suburbcost-council-rates.csv`](https://www.suburbcost.com.au/dataset/suburbcost-council-rates.csv) | 227 | QLD, NSW, VIC | rates only |

Most of this dictionary describes the first file. The second is documented in
its own section near the end, and there is one rule that governs both: the
states publish different statistics, so the rate column is not comparable
across them.

Attribution, copy this line:

> SuburbCost (2026). Australian council-level household running cost dataset, 2026-27. Version 2026-27+8fc219d8. https://doi.org/10.5281/zenodo.22951042 https://www.suburbcost.com.au/dataset/ Licensed CC BY 4.0.

## What one row is

One local government area, aggregated from the suburbs SuburbCost holds inside
it: 6,158 suburbs across 148 councils. A further
64 suburbs, all in New South Wales, carry no council assignment and
appear nowhere in this file.

**Suburb-level rows are not published.** This is a derived aggregate.

## Columns

| Column | Type | Meaning | Units |
| --- | --- | --- | --- |
| `council_name` | text | Council name as SuburbCost publishes it. Derived from councilSource. | n/a |
| `state` | text | QLD or NSW. Suburb coverage is these two states only. | n/a |
| `suburbcost_slug` | text | Resolves to https://www.suburbcost.com.au/council-rates/{slug}. | n/a |
| `suburb_count` | integer | Suburbs SuburbCost assigns to this council. Every median below is computed over exactly these suburbs. | count |
| `council_rate_aud` | integer | The modal stored rate for the council. Read council_rate_basis before using it. | AUD per year |
| `council_rate_basis` | enum | Which statistic council_rate_aud is. Four values, see Bases below. Averaging across them produces a meaningless number. | n/a |
| `council_rate_year` | text | Financial year the rate applies to. Derived, not read from the source string, which names two years for NSW. | n/a |
| `council_rate_source` | text | The source string verbatim, including the approved SRV percentage where one applies. | n/a |
| `council_rate_variant_count` | integer | Distinct rates stored inside this council. 1 for 144 councils. Logan sets 2 and Western Downs sets 3 by locality, so their single published figure is the modal one. | count |
| `median_total_house_aud_incl_est` | integer | Median of council + water + modelled commute + building insurance across the council's suburbs. Contains an estimated component, see Estimated fields. | AUD per year |
| `mean_total_house_aud_incl_est` | integer | Mean of the same. Published alongside the median because the two diverge sharply where a council mixes tapered and flat commute suburbs. | AUD per year |
| `median_total_unit_aud_incl_est` | integer | Median of councilUnit + water + strata + modelled commute + contents insurance. A different set of components from the house total, not a like-for-like comparison. | AUD per year |
| `median_water_aud` | integer | Median water and sewerage. Assumes 200 kL a year. | AUD per year |
| `median_modelled_commute_drive_aud` | integer | Median modelled driving commute. Read commute_basis before using it, see Commute. | AUD per year |
| `median_insurance_house_aud_est` | integer | Median building insurance. Estimated for every row. | AUD per year |
| `median_strata_aud_est` | integer | Median body corporate, units only, 0 for houses. Estimated for every row. | AUD per year |
| `suburbs_rate_standin` | integer | Suburbs in this row whose own council rate is unknown and which carry this council's rate instead. See Substituted rates. | count |
| `suburbs_water_estimated` | integer | Suburbs whose water figure is a neighbouring utility's rather than their own. | count |
| `suburbs_flat_commute_allowance` | integer | Suburbs carrying the flat $18,200 regional driving allowance rather than a tapered commute. | count |
| `commute_basis` | enum | tapered, mixed or flat_allowance_only. See Commute. | n/a |
| `completeness` | enum | complete when all four flag columns are zero, the council stores one rate, and it has at least 3 suburbs. flagged otherwise. Nothing is excluded from this dataset, so filter on the flag columns rather than assuming a clean file. | n/a |

## Rounding

Every dollar figure is a whole number of Australian dollars, rounded **half
up**. This matters when reproducing the medians. 70 of the 148 councils hold an
even number of suburbs, so their median is the mean of two integers and can land
on a half dollar. 60 of the 888 median cells in this file do.

Python's `round()` and R's `round()` both use half-to-**even**, so
recomputing these medians in either language returns a figure one dollar lower
on those cells unless you round half up explicitly. That is a convention
difference, not a data disagreement.

## Bases, read this before computing anything across states

`council_rate_aud` is not one statistic. It is four, and they are not
interchangeable. A mean across the whole column is meaningless.

| `council_rate_basis` | Councils | What the figure is |
| --- | --- | --- |
| `qld_minimum_general_rate_ppr` | 21 | The **minimum** residential general rate for an owner-occupied principal place of residence. A floor. Most owner-occupiers pay more, scaled to land value. |
| `qld_median_property_rate` | 1 | Redland City only, which publishes a median-valued-property figure rather than a minimum. |
| `nsw_olg_average_residential_uplifted_by_ipart_peg` | 119 | The Office of Local Government average residential rate, uplifted by that council's IPART rate peg. An **average**, not a floor. |
| `nsw_olg_average_residential_uplifted_by_approved_srv` | 7 | The same, uplifted by an IPART-approved special rate variation above the peg. Separated so rate-capping research can isolate them. |

A Queensland minimum sits **below** a typical Queensland bill. A New South
Wales average sits **at** a typical New South Wales bill. Comparing the two
directly understates the real gap between the states rather than overstating
it. Waste, levies and separate charges are excluded throughout.

## Commute

`median_modelled_commute_drive_aud` is **modelled, not measured**, and the
model has two regimes.

Under 100 km from the relevant centre, the annual cost is
`distance x 2 x trips x 48 x $0.91`, where `trips = max(0.5, 4 - distance/20)`,
with a minimum costing distance of 1.5 km. At or beyond 100 km the figure is a
flat **$18,200**, which is 20,000 km a year of **all household driving**. That
is a different quantity from a commute and is deliberately not expected to line
up with the taper.

| `commute_basis` | Councils | Meaning |
| --- | --- | --- |
| `tapered` | 53 | No suburb uses the flat allowance. |
| `mixed` | 43 | Some do. The median may sit either side of the boundary. |
| `flat_allowance_only` | 52 | Every suburb uses it. The figure is **not a commute** for these councils. |

**Distance is straight-line, so the tapered figures understate real driving.**
Measured against 6,212 pinned road routes, median road distance is
**1.294x** straight-line. That factor is valid only for the 4,452 suburbs
measured to a state capital. For suburbs measured to a regional centre such as
Newcastle, Wollongong, Cairns or Townsville **no road routes exist and the
understatement is unmeasured**. No correction has been applied to this file.

The understatement runs in opposite directions. Metropolitan suburbs are
understated. Suburbs on the flat allowance are not understated at all, they are
measuring something else.

## Estimated fields

Columns ending `_est` are AI-estimated and not sourced from a provider.
Building insurance and body corporate are estimated for **every** row, so the
marker is in the column name rather than a per-row flag that would be constant.

The three `_incl_est` totals contain an estimated component. Every component
is published separately in this file, so an estimate-free subtotal can be
reconstructed by summing the columns you accept.

One caution on insurance. It is dominated by **cyclone exposure**, not flood or
fire. Its correlation with the published flood and fire ratings is r = 0.053.
Means run from $2,522 in New South Wales to about $7,500 north of 16 degrees
south. Do not read a high insurance figure as a flood finding.

## Substituted rates

Six Queensland rows cover more councils than they name. 52 suburbs belong to
**18 Queensland councils that have no row in this file** and carry a
neighbouring council's rate instead. Aurukun, Cook, Croydon, Etheridge, Kowanyama, Mornington, Pormpuraaw and Yarrabah sit inside Cairns Regional Council. Carpentaria, Charters Towers, Flinders, Hinchinbrook, McKinlay and Richmond sit inside Townsville City Council. Isaac and Livingstone sit inside Mackay Regional Council. Gympie sits inside Noosa Council and Sunshine Coast Council. Goondiwindi sits inside Toowoomba Regional Council.

`suburbs_rate_standin` gives the count per row. Cairns Regional Council is the
extreme case at 28 of 86 suburbs. **Those councils are absent from this
dataset, not merely incomplete within it.**

| Absent council | Suburbs | Carries the rate of |
| --- | --- | --- |
| Etheridge Shire Council | 10 | Cairns Regional Council |
| Charters Towers Regional Council | 7 | Townsville City Council |
| Croydon Shire Council | 7 | Cairns Regional Council |
| Goondiwindi Regional Council | 5 | Toowoomba Regional Council |
| Cook Shire Council | 4 | Cairns Regional Council |
| Isaac Regional Council | 3 | Mackay Regional Council |
| Mornington Shire Council | 3 | Cairns Regional Council |
| Flinders Shire Council | 2 | Townsville City Council |
| Gympie Regional Council | 2 | Noosa Council, Sunshine Coast Council |
| Aurukun Shire Council | 1 | Cairns Regional Council |
| Carpentaria Shire Council | 1 | Townsville City Council |
| Hinchinbrook Shire Council | 1 | Townsville City Council |
| Kowanyama Aboriginal Shire Council | 1 | Cairns Regional Council |
| Livingstone Shire Council | 1 | Mackay Regional Council |
| McKinlay Shire Council | 1 | Townsville City Council |
| Pormpuraaw Aboriginal Shire Council | 1 | Cairns Regional Council |
| Richmond Shire Council | 1 | Townsville City Council |
| Yarrabah Aboriginal Shire Council | 1 | Cairns Regional Council |

## Water

181 suburbs across five Queensland councils carry a neighbouring utility's
water figure rather than their own, and in all five cases it is **every** suburb
in the council: Cassowary Coast, Mareeba, Southern Downs, Tablelands and Western Downs. `suburbs_water_estimated` equals `suburb_count` for those rows.

## Known limits, stated rather than buried

- **Nothing is excluded.** Only 49 of 148 rows carry no flag at all. A file
  restricted to clean rows would be a third of this one and would hide which
  councils were dropped, so every row is published and every gap is a column.
- **Medians are computed over all of a council's suburbs**, including flagged
  ones. The flag columns give the denominator so you can judge or refilter.
- Mosman Council holds one suburb, so its median is a single observation.
- 39 councils span more than one commute centre, so their median commute mixes
  trips to different destinations.
- Property prices, land valuations and capital growth are **not** in this
  dataset and are not published by SuburbCost anywhere.

## The second file, `suburbcost-council-rates.csv`

227 councils, one row each: 22 in Queensland, 126 in New South Wales
and 79 in Victoria. Rates only, no water, commute, insurance or body
corporate.

Victoria is here and not in the first file because the first file is an
aggregate over suburb records and SuburbCost holds no Victorian suburb records.
There is no honest way to fill a `median_water_aud` for Frankston, so rather
than publish 79 rows of blanks the rate figure gets its own file.

| Column | Type | Meaning |
| --- | --- | --- |
| `council_name` | text | Council name as SuburbCost publishes it. |
| `state` | text | `QLD`, `NSW` or `VIC`. |
| `suburbcost_slug` | text | Slug of that council's page on SuburbCost. |
| `suburbcost_url` | url | The council's page. Every one of the 227 resolves. |
| `rate_aud` | integer | The residential rate figure, whole AUD. **Read the next column before using it.** |
| `rate_statistic` | text | What `rate_aud` actually measures. Three values, one per state. |
| `rate_year` | text | Financial year. QLD and NSW are 2026-27. Victoria is 2024-25. |
| `rate_source` | text | The document the figure came from. |
| `comparable_across_states` | text | Always `no`. Not a column to filter on, a warning. |

### The three statistics

| `state` | `rate_statistic` | What it is |
| --- | --- | --- |
| `QLD` | minimum residential general rate, owner-occupied PPR | A **floor**. Most owner-occupiers pay more, scaled to land value. |
| `NSW` | average residential rate (OLG) uplifted by the IPART peg | An **average**. Close to a typical bill. |
| `VIC` | LGPRF FIM-E4, average rate per property assessment | An average across **all rateable property types**, not just residential, and it excludes service rates and charges. |

These measure three different things. A Queensland minimum sits below a typical
Queensland bill, a New South Wales average sits at one, and a Victorian
per-assessment average is diluted by commercial, industrial and farm
assessments and is not a residential bill at all.

So: sorting the whole file by `rate_aud` and reading off the top gives a wrong
answer, and a mean of the column is not a number. **Group by `state` first.**
Within one state the figures are built the same way and rank cleanly, which is
what the per-state rankings on SuburbCost do.

Victoria also sits a year behind. The Local Government Performance Reporting
Framework's most recent full release is 2024-25, published November 2025. It is
reported as published rather than uplifted to 2026-27, because the uplift would
be an estimate presented in a column of measured figures.

## Provenance

Generated by `generate-dataset.js` from `data/suburbs.json`, plus
`data/vic-council-rates.json` for the second file. The version string is the
data vintage plus a short hash of both files' contents, so it changes when a
figure changes and not merely because the generator ran.

Rate figures come from each council's own budget or revenue statement (QLD) and
from the NSW Office of Local Government "Your Council" release uplifted by the
relevant IPART determination (NSW). Water is from the named retailer where one
exists and is estimated for other regional NSW. Insurance and body corporate are
estimated. The commute model is described above.
