复制下面全部内容，替换 `docs/data_dictionary.md`：

```markdown
# Data dictionary

## Source and refresh

All four tables use NYC PLUTO data from the Socrata API, filtered to `borough = 'MN'`.
API documentation: https://dev.socrata.com/foundry/data.cityofnewyork.us/64uk-42ks
The successful snapshot uses version `26v2`. GitHub Actions is configured to run daily at 11:17 UTC and supports manual runs.
Each successful run downloads a full Manhattan snapshot and replaces the published tables. Failed validation stops publication. Historical snapshots are not retained.
Column types below describe the transformation SQL. Examples illustrate individual values, not one complete parcel record.

## 1. manhattan_parcels_clean

**Table name on MotherDuck:** `course.main.manhattan_parcels_clean`
**One row is:** One Manhattan tax lot identified by its normalized BBL.
**Source:** The filtered PLUTO API snapshot described above.
**Removed on purpose:** Other boroughs are excluded. No downloaded parcels are removed after validation; parks, water, small lots, and high-built-FAR parcels remain. Blank text and unsuccessful numeric conversions become NULL.

| Column | Type | What it means | Units or allowed values | Example |
|---|---|---|---|---|
| bbl | VARCHAR | Normalized borough-block-lot identifier | Ten-digit text identifier | 1011870003 |
| borough | VARCHAR | Borough abbreviation | MN | MN |
| address | VARCHAR | Parcel address or descriptive label | Text or NULL | RIVERSIDE PARK |
| landuse | INTEGER | PLUTO land-use category | Source code or NULL | 9 |
| zonedist1 | VARCHAR | First recorded zoning district | Uppercase text or NULL | R10A |
| lotarea | DOUBLE | Source tax-lot area | Square feet or NULL | 10000 |
| bldgarea | DOUBLE | Source total building floor area | Square feet or NULL | 20000 |
| builtfar | DOUBLE | Existing total built floor area ratio | Ratio or NULL | 2.0 |
| residfar | DOUBLE | PLUTO residential reference FAR | Ratio or NULL | 3.44 |
| commfar | DOUBLE | PLUTO commercial reference FAR | Ratio or NULL | 10.0 |
| facilfar | DOUBLE | PLUTO community-facility reference FAR | Ratio or NULL | 6.5 |
| landmark | VARCHAR | Landmark designation information | Text or NULL | NULL |
| histdist | VARCHAR | Historic-district designation information | Text or NULL | NULL |
| version | VARCHAR | PLUTO release identifier | One nonempty version per snapshot | 26v2 |
| retrieved_at | TIMESTAMP WITH TIME ZONE | Clean-table transformation time | Timestamp, not the source update date | 2026-11-23 11:17:00+00 |

**Limits:** Manhattan only; one snapshot cannot show historical changes. Reference FAR values do not establish complete parcel-specific development rights.
**Example questions:** How many parcels are included? What zoning and FAR values are recorded for a given BBL?

## 2. manhattan_far_comparison

**Table name on MotherDuck:** `course.main.manhattan_far_comparison`
**One row is:** One parcel under one of three use scenarios.
**Source:** `manhattan_parcels_clean`. All 15 clean-table columns above are retained unchanged, with four additional columns below.
**Removed on purpose:** No parcel-scenario rows are removed. Invalid comparisons receive NULL calculated values.

| Column | Type | What it means | Units or allowed values | Example |
|---|---|---|---|---|
| use_type | VARCHAR | Use scenario | residential, commercial, community_facility | residential |
| reference_far | DOUBLE | residfar, commfar, or facilfar for the scenario | Ratio or NULL | 3.44 |
| far_gap | DOUBLE | reference_far minus builtfar | Positive, zero, negative, or NULL | 2.31 |
| positive_gap_area_proxy_sqft | DOUBLE | Positive far_gap multiplied by valid positive lotarea | Square feet or NULL | 23100 |

**Calculation rules:** far_gap requires finite reference_far > 0 and finite builtfar >= 0. The area proxy additionally requires far_gap > 0 and finite lotarea > 0. Otherwise the calculated value is NULL.
**Limits:** Each scenario uses the same total built FAR, not existing floor area for that individual use. Scenarios overlap and cannot be added as independent capacity. The proxy is not verified buildable floor area.
**Example questions:** How do the three reference FAR values compare with existing built FAR for a parcel? Which comparisons have positive gaps?

## 3. manhattan_far_summary

**Table name on MotherDuck:** `course.main.manhattan_far_summary`
**One row is:** One combination of zonedist1, use_type, and version.
**Source:** `manhattan_far_comparison`. No comparison rows are removed from group counts; missing zoning remains a NULL group.

| Column | Type | What it means | Units or allowed values | Example |
|---|---|---|---|---|
| zonedist1 | VARCHAR | First zoning district defining the group | Zoning label or NULL | R10A |
| use_type | VARCHAR | Use scenario defining the group | Same three scenarios as above | residential |
| version | VARCHAR | Source release defining the group | Version text | 26v2 |
| parcel_count | BIGINT | Number of parcel-scenario rows in the group | Count | 100 |
| comparable_parcels | BIGINT | Number with non-NULL far_gap | Count | 90 |
| not_comparable_parcels | BIGINT | parcel_count minus comparable_parcels | Count | 10 |
| positive_gap_parcels | BIGINT | Number with far_gap > 0 | Count | 60 |
| avg_far_gap | DOUBLE | Mean of non-NULL gaps, including zero and negative gaps; rounded to 3 decimals | Ratio or NULL | 1.25 |
| parcels_with_positive_area_proxy | BIGINT | Number with non-NULL area proxy | Count | 55 |
| positive_gap_area_proxy_sqft | DOUBLE | Sum of non-NULL area proxies; rounded to 2 decimals | Square feet or NULL | 500000 |

**Limits:** Counts can be summed across zoning groups within one scenario and version, but summing across scenarios counts parcels repeatedly. Groups without valid area proxies have NULL sums. Proxy totals are not confirmed development capacity.
**Example questions:** Which scenario has the most positive-gap parcels? How do average gaps vary across zoning districts?

## 4. manhattan_far_ranking

**Table name on MotherDuck:** `course.main.manhattan_far_ranking`
**One row is:** One parcel-scenario combination with far_gap > 0.
**Source:** `manhattan_far_comparison`. Zero, negative, and NULL gaps are excluded. Positive-gap rows with NULL area proxies remain.
**Columns:** bbl, address, landuse, zonedist1, use_type, lotarea, builtfar, reference_far, far_gap, positive_gap_area_proxy_sqft, landmark, histdist, version, and retrieved_at retain the types, meanings, and units defined above.
**Additional column:** screening_rank (BIGINT): row number starting at 1 within each use_type and version. Sort order is area proxy descending with NULL last, then far_gap descending, then bbl ascending.
**Limits:** All qualifying rows are retained, not just the top three. Parks and water are not excluded. Rankings do not establish ownership, availability, legal buildability, or financial feasibility. ZAP is not integrated.
**Example questions:** Which three parcels rank highest in each scenario? Which leading records need further investigation because of their land use or address label?
```