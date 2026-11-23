# Findings report

Three things your data shows, each backed by one of your published tables.
Next week you'll ask your AI app these questions and check whether it gets the same answers.

## Finding 1

**What I found** (one or two sentences, with a number):
The cleaned Manhattan snapshot contains 42,504 parcels, all from PLUTO version 26v2. This is a single-version snapshot, not a historical trend.

**From table:** `course.main._manhattan_parcels_clean`

**The query or row that shows it:**

```sql
SELECT version, COUNT(*) AS parcels
FROM course.main.manhattan_parcels_clean
GROUP BY version;
```
**A question a user could ask the app to discover this:**
How many Manhattan parcels are in the dataset, and which PLUTO version does it use?

## Finding 2

**What I found:**
The community facility scenario has 33,467 parcels with a positive FAR gap, compared with 28,525 for residential and 7,622 for commercial. These are separate comparisons against existing built FAR, not additive development capacities.

**From table:** `course.main.manhattan_far_summary`

**The query or row that shows it:**

```sql
SELECT use_type, version,
       SUM(comparable_parcels) AS comparable_parcels,
       SUM(not_comparable_parcels) AS not_comparable_parcels,
       SUM(positive_gap_parcels) AS positive_gap_parcels
FROM course.main.manhattan_far_summary
GROUP BY use_type, version
ORDER BY positive_gap_parcels DESC;
```

**A question a user could ask the app to discover this:**
Which use scenario has the most parcels with a reference FAR above their existing built FAR?

## Finding 3

**What I found:**
The highest-ranked commercial parcel is labeled BODY OF WATER, with an area proxy of 45,544,670 square feet. The highest-ranked residential parcel is RIVERSIDE PARK, showing that a large FAR gap and lot area alone cannot identify realistic redevelopment opportunities.

**From table:** `course.main.manhattan_far_ranking`

**The query or row that shows it:**
Which use scenario has the most parcels with a reference FAR above their existing built FAR?

```sql
SELECT use_type, bbl, address, landuse,
       far_gap, positive_gap_area_proxy_sqft
FROM course.main.manhattan_far_ranking
WHERE screening_rank = 1
ORDER BY use_type;
```

**A question a user could ask the app to discover this:**
Do the highest-ranked parcels reveal any limitations in the current screening method?
---

## What this data can't tell you

One limit a reader should know before trusting these findings (missing places, short time span, how values are collected):
The FAR gap is a preliminary screening indicator, not a verified amount of buildable floor area. The current ranking includes parcels labeled as water or parks and does not account for ownership, all zoning restrictions, or development applications. Further checks are needed before using these results to identify redevelopment opportunities.