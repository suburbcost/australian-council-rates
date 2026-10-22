# Australian council rates and household running costs

Residential council rates for **227 local government areas** across Queensland,
New South Wales and Victoria, plus modelled household running costs for the 148
councils where suburb-level records support them. 2026-27. CC BY 4.0.

| File | Councils | States | Contents |
| --- | --- | --- | --- |
| `suburbcost-council-running-costs.csv` | 148 | QLD, NSW | Council rate, plus median water, modelled commute, building insurance and body corporate, with data-quality flags |
| `suburbcost-council-rates.csv` | 227 | QLD, NSW, VIC | Residential council rates only, each row naming its own basis |

## Read this before you rank anything

The three states do not publish the same statistic, and this is the whole
methodological point of the dataset:

- **Queensland** publishes the *minimum* residential general rate for an
  owner-occupied principal place of residence. It is a floor. Most owner
  occupiers pay more, scaled to land value.
- **New South Wales** publishes an *average* residential rate, via the Office of
  Local Government, uplifted here by each council's IPART rate peg. It sits at a
  typical bill, not below it.
- **Victoria** publishes the Local Government Performance Reporting Framework
  indicator FIM-E4, the *average rate per property assessment* across all
  rateable property types, excluding service rates and charges. It is diluted by
  commercial, industrial and farm assessments and is not a residential bill.

So sorting the file by rate and reading off the top produces a wrong answer, and
a mean of the rate column is not a number that means anything. **Group by state
first.** Every row names its own basis in `rate_statistic`, and
`comparable_across_states` is `no` on all 227 rows.

```python
import pandas as pd

df = pd.read_csv("suburbcost-council-rates.csv")

# Right: rank inside one state.
qld = df[df.state == "QLD"].sort_values("rate_aud", ascending=False)

# Wrong: this mixes a Queensland minimum, a NSW average and a Victorian
# per-assessment average, and the answer means nothing.
# df.sort_values("rate_aud", ascending=False).head()
```

## Reproducing the running-cost medians

Every dollar figure is rounded **half up**. Python's `round()` and R's
`round()` both round half to **even**, so recomputing the medians in either
language returns a figure one dollar lower on the cells that land on a half
dollar. 61 of the 888 median cells do. That is a convention
difference, not a data disagreement.

## Limits

- Building insurance and body corporate are **estimated, not sourced**, and are
  marked with an `_est` suffix.
- The commute figure is **modelled** from straight-line distance and is not a
  measured journey. For 52 councils it is a flat regional driving allowance
  rather than a commute at all.
- Victoria is a year behind. The most recent full LGPRF release is 2024-25. It is
  reported as published rather than uplifted, because an uplift would be an
  estimate sitting in a column of measured figures.
- This is a **council-level aggregate**. Suburb-level rows are not published.
- The dataset contains **no property prices, no land valuations and no capital
  growth**, and SuburbCost does not publish those anywhere.

## Provenance

Queensland rates come from each council's own budget or revenue statement. New
South Wales rates come from the NSW Office of Local Government "Your Council"
release uplifted by the relevant IPART determination. Victorian rates come from
the Local Government Performance Reporting Framework full council data set.
Water comes from the named retailer where one exists.

Column-by-column documentation is in [`data-dictionary.md`](data-dictionary.md).

## Updates

Council rates are set annually. This file tracks the 2026-27 year and is
versioned `2026-27+5afccafe`, which is the data vintage plus a hash of the file
contents, so the version changes when a figure changes and not merely when the
generator runs.

## Licence and citation

CC BY 4.0, https://creativecommons.org/licenses/by/4.0/

> SuburbCost (2026). Australian council-level household running cost dataset, 2026-27. Version 2026-27+5afccafe. https://doi.org/10.5281/zenodo.22951042 https://www.suburbcost.com.au/dataset/ Licensed CC BY 4.0.

Canonical page: https://www.suburbcost.com.au/dataset/

## Disclosure

Published by SuburbCost, which runs a consumer site built on the same model.
The limits above are the ones that matter for reuse.
