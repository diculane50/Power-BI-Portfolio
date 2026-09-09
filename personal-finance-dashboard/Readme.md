# Personal Finance Dashboard — Power BI

An interactive Power BI report built from a (synthetic) monthly bank statement — spending by category, a running balance trend, and total spending at a glance.

> **Note on the data:** The dataset used here is fictional. I built it myself to practice on realistic, intentionally messy data (inconsistent text formatting, a duplicate row, blank category values) without using any real personal financial information.

## What's in the report

| Visual | Shows |
|---|---|
| Card | Total Spending for the month |
| Column chart | Spending by Category (Income excluded via a visual-level filter) |
| Line chart | Account balance over time, drilled to daily granularity |
| Slicer | Filters the whole page by date range |

## Tools used

Power BI Desktop — Power Query (M), DAX, and native visuals only (no custom/AppSource visuals).

## Process

**1. Connect**
Loaded the CSV via Get Data > Text/CSV.

**2. Clean (Power Query)**
- Set correct data types — `Transaction Date` to Date, `Amount`/`Balance` to Decimal Number (auto-detect isn't trustworthy on either of these)
- Trimmed and cleaned the `Description` column — the raw export had inconsistent spacing and casing on repeated merchants (e.g. `  Trader Joe's #114 ` vs `TRADER JOE'S #114`)
- Replaced blank `Category` values with "Unknown" — a couple of transactions came in with no category at all, and leaving them blank would have silently dropped them from any category-level total
- Removed a duplicate transaction row — matched specifically on `Transaction Date`, `Description`, and `Amount` (not every column), since the running `Balance` column is legitimately different between the two rows even though the charge itself was duplicated

**3. Model**
Added one DAX measure to isolate spending from income:

```dax
Total Spending = -1 * CALCULATE(SUM(practice_bank_statement[Amount]), practice_bank_statement[Amount] < 0)
```

Summing `Amount` directly would have netted expenses against income and produced a meaningless number — this measure filters to debits only and flips the sign so it reads as a positive spend figure.

**4. Visualize**
- Card and column chart were straightforward once the measure existed
- The line chart needed extra attention: Power BI's auto date hierarchy defaults to Year, which collapsed an entire month of transactions into a single point. Drilled down to Day level to actually see the trend
- Changed `Balance`'s aggregation from the default Sum to Maximum on the line chart, since several dates have more than one transaction and summing a running balance across them would double-count it

## Challenges worth calling out

- **Aggregation defaults are not always right.** Power BI silently sums or counts fields based on data type, not based on what makes sense — this showed up twice: a text field defaulting to Count, and a running-balance column needing Maximum instead of the default Sum.
- **A duplicate row isn't always safe to just delete.** Decided how to define "duplicate" deliberately (which columns count as identifying the transaction) rather than matching on every column, which would have missed it.
- **Blank values need an explicit decision**, not just "let Power BI figure it out" — null category values would have quietly disappeared from any grouping instead of being counted as their own bucket.

## Screenshots

![Full dashboard view](screenshots/Screenshot%201.png)
 
![Dashboard filtered by date range](screenshots/Screenshot%202.png)

## Files

- `practice_bank_statement.csv` — the source data
- `Practice Total Spending Dash.pbix` — the full Power BI report
