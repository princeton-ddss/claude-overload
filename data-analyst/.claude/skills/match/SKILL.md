---
name: match
description: >
  Use when linking records across datasets. Handles exact key matching
  (FIPS codes, IDs), fuzzy matching (names, addresses), crosswalks
  (different geographies or vintages), and probabilistic record linkage.
  Examples: "Merge census and election data", "Link by county name".
---

# Match Datasets

Link records across datasets.

## Strategies

### Exact key match
Records share a common identifier (FIPS code, ID, year).
- Method: Direct join via DuckDB
- Use when: keys are standardized and complete

### Fuzzy match
Keys are similar but not identical ("St. Louis" vs "Saint Louis").
- Method: String distance matching (`rapidfuzz`, `recordlinkage`)
- Use when: joining on names, addresses, or other free-text fields

### Crosswalk
Different geographic units or vintages need translation (ZIP → county, 2010 → 2020 tracts).
- Method: Official crosswalk files with weights (Census, HUD, NHGIS)
- Use when: datasets use different geographic definitions

### Probabilistic linkage
No shared key; match on multiple attributes.
- Method: Probabilistic record linkage (`dedupe`, `recordlinkage`)
- Use when: linking survey respondents, patient records, etc.

## Workflow

1. Identify key columns in both datasets
2. Diagnose match quality (record counts, expected match rate)
3. Standardize keys if needed (case, punctuation, abbreviations)
4. Execute the match (prefer DuckDB for large datasets)
5. Validate: report matched/unmatched counts, spot-check results
6. Warn if match rate is unexpectedly low (<90% for exact keys)
