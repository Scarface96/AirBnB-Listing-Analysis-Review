# 📊 Airbnb Listings Analysis: Tableau & Power BI kit

This project has no dashboard yet, because the source dataset (`Listings.csv`, about 280,000 listings worldwide) is too large for the repo. Once you have the file, this guide gets you from CSV to a finished Tableau or Power BI dashboard.

## Prepare the data

Filter to Paris and keep the columns the notebook uses. In Python:

```python
import pandas as pd
df = pd.read_csv("Listings.csv", encoding="ISO-8859-1", low_memory=False, parse_dates=["host_since"])
paris = df.loc[df["city"] == "Paris", ["listing_id", "host_id", "host_since", "neighbourhood", "accommodates", "price"]]
paris = paris[(paris["price"] > 0) & (paris["accommodates"] > 0)]   # 54 listings have both at 0
paris.to_csv("bi/data/fact_listing.csv", index=False)
```

That file is small enough (~65,000 rows) to commit.

## Measures

| Name | Power BI (DAX) | Tableau |
|---|---|---|
| Listings | `COUNTROWS ( fact_listing )` | `COUNT([Listing Id])` |
| Avg Nightly Price | `AVERAGE ( fact_listing[price] )` | `AVG([Price])` |
| Price per Guest | `DIVIDE ( SUM ( fact_listing[price] ), SUM ( fact_listing[accommodates] ) )` | `SUM([Price]) / SUM([Accommodates])` |
| New Hosts | `DISTINCTCOUNT ( fact_listing[host_id] )` with year of `host_since` on the axis | `COUNTD([Host Id])` by `YEAR([Host Since])` |
| Price vs Paris | `[Avg Nightly Price] - CALCULATE ( [Avg Nightly Price], ALL ( fact_listing[neighbourhood] ) )` | `AVG([Price]) - {FIXED : AVG([Price])}` |

## Dashboard design

Canvas 1280 × 720, using `powerbi/theme.json` or `tableau/Preferences.tps` from this folder.

- **Where is Paris expensive?** Bar chart of Avg Nightly Price by neighbourhood, sorted, with Price vs Paris in the tooltip. If you add latitude and longitude, use a map.
- **What does extra space cost?** Line chart of Avg Nightly Price by accommodates, with a neighbourhood slicer (default Élysée).
- **What did the 2015 rules change?** Dual-axis chart: New Hosts (columns) and Avg Nightly Price (line) by year of `host_since`, with an annotation at 2015.

## Check your numbers

From the notebook's output. The notebook kept the 54 zero-price rows, so averages after the filter above will be a little higher:

| Measure | Value |
|---|---|
| Paris listings after filtering | 64,690 (before removing the 54 zero rows) |
| Élysée average price | about €211 |
| Louvre / Passy / Palais-Bourbon / Luxembourg | about €175 / €161 / €157 / €156 |
| Élysée, 1-guest vs 4-guest listings | about €80 vs €212 |
| New hosts, 2008 → 2012 | 4 → 4,592 |
