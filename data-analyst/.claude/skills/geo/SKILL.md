---
name: geo
description: >
  Use for spatial operations: spatial joins, geocoding, boundary crosswalks,
  and mapping. Examples: "Which county is each address in?",
  "Map turnout by county", "Convert ZIP codes to counties".
---

# Geospatial Analysis

Spatial operations and mapping.

## Capabilities

### Spatial joins
Point-in-polygon, polygon-to-polygon.
- Tools: geopandas
- Example: assign addresses to counties, overlay census tracts on school districts

### Geocoding
Address → coordinates.
- Preferred: Census geocoder for US bulk geocoding (free, no API key)
- Alternative: geopy with Nominatim for small batches

### Boundary crosswalks
Translate between geographic units (ZIP ↔ county, tract ↔ county).
- Sources: Census TIGER files, HUD crosswalks
- Tools: pygris for boundary files

### Mapping
Choropleth and point maps.
- Interactive: folium → HTML in `notebooks/`
- Static: matplotlib + geopandas → PNG in `notebooks/`

## Workflow

1. Identify the task (spatial join, geocoding, crosswalk, mapping)
2. Get boundary files via pygris if needed
3. Verify CRS alignment (reproject if necessary)
4. Execute the operation
5. Open outputs: `open <file>` (use `run_in_background: true`)
