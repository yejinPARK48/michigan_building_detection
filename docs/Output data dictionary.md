# Output Data Dictionary

Building detection pipeline — PUMA estimation outputs. Two CSV files are produced
per PUMA, stored in `predictions_puma/`.

## 1. Tile level — `tile_predictions_puma_<code>.csv`

One row per 256 m × 256 m tile within the PUMA. Tiles are generated from a single
regular grid over the PUMA polygon and retained when their centroid falls inside it.

| Column | Description |
|---|---|
| `tile_id` | Unique tile identifier (`<code>_000001`, …) |
| `puma` | PUMA code (7-digit GEOID) |
| `bg_geoid` | Census block group (12-digit GEOID) containing the tile centroid |
| `county_fips` | County FIPS (5-digit), derived from `bg_geoid` |
| `pred_count` | Predicted building count (connected components ≥ 50 px) |
| `mask_area_m2` | Predicted building footprint area in m² (same ≥ 50 px components as `pred_count`) |
| `mask_ratio` | Fraction of building-class pixels in the tile (thresholded mask only) |
| `tier` | Built-up class — Dense / Sparse / Empty, based on `mask_ratio` |

## 2. Block-group level — `bg_estimates_puma_<code>.csv`

Tile predictions aggregated to the census block group. This is the analytic file
used for downstream linkage (e.g., joining to ACS records).

| Column | Description |
|---|---|
| `puma`, `bg_geoid`, `county_fips` | Identifiers |
| `area_km2` | Total block-group land area (km²), from the block-group polygon in UTM |
| `tiled_area_km2` | Area actually covered by tiles (km²); coverage check against `area_km2` |
| `pred_buildings` | Total predicted buildings in the block group |
| `building_density_km2` | Building count density — `pred_buildings` / `area_km2` |
| `mask_area_km2` | Total predicted building footprint area (km²) |
| `built_fraction` / `built_pct` | Built-up coverage — footprint area / total area (ratio 0–1 / percent) |
| `n_tiles` | Number of tiles in the block group |
| `mean_mask_ratio` | Mean tile mask ratio (raw built-up proxy) |
| `dense_tiles` / `sparse_tiles` / `empty_tiles` | Tile counts by tier |

## Notes

**Two distinct density measures.** `building_density_km2` is a count density
(buildings per km²); `built_pct` is an area share (percent of land covered by
building footprint).

**Internal consistency.** Footprint area (`mask_area_m2` / `mask_area_km2`) and
building counts (`pred_count` / `pred_buildings`) are both computed from the same
connected components (≥ 50 px ≈ 12.5 m²).
