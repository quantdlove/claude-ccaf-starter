# Weekly Report Generator

Generate a weekly performance report from the analytics database.

## When to Use
Run every Monday morning, or when someone says "weekly report" or "last week's numbers."

## Inputs
- Date range (defaults to previous Monday-Sunday)
- Report type: "executive" (summary) or "detailed" (full breakdown)

## Steps

### Step 1: Pull Data
Query the analytics database for the date range:
- Total users (new, returning, churned)
- Revenue (MRR, new sales, churn)
- Product metrics (DAU, feature adoption, support tickets)

Use `search_datasets` first to verify available endpoints. Do not use cached tool knowledge from prior sessions.

### Step 2: Compare to Benchmarks
- Compare each metric to the 4-week rolling average
- Flag any metric that moved more than 10% in either direction
- Note the trend direction (up/down/flat) for each category

### Step 3: Generate Report
- Use the dashboard CSS system (scrollable, max-width container)
- Executive summary at top (3-5 bullet points, biggest movers only)
- Detailed sections below with charts
- Data freshness footer: "Data as of [last date in range]"

### Step 4: Validate
- Verify key numbers match the source database (spot-check 3 metrics)
- Confirm all charts render correctly
- Check that no placeholder data leaked from the template

## Output
- File: `reports/weekly-YYYY-MM-DD.html` (Track 2: Archive, date-stamped)
- Present file link to user after generation

## Constraints
- Do NOT fabricate metrics. Every number traces to the database query.
- Do NOT editorialize in the data sections. "Revenue grew 12%" is data. "Revenue had an impressive 12% surge" is editorial.
- If a data source is unavailable, write `[DATA UNAVAILABLE]` and flag for review. Do not estimate.
