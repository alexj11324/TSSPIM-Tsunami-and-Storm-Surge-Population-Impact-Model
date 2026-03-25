# CLAUDE.md — ARC Capstone

## Project Overview

CMU Heinz MSPPM 2026 Capstone for American Red Cross. Property-level storm surge/tsunami impact modeling using FEMA's FAST (Flood Assessment Structure Tool) with NSI building inventory (30M+ structures) and SLOSH surge rasters. Goal: estimate building damage, displaced population, and high-need populations for Red Cross shelter/casework planning.

## Architecture

```
NSI Parquet → DuckDB: clean/filter/dedup/map → FAST CSV → FAST engine → damage predictions
NHC P-Surge GeoTIFF (FAST-main/rasters/) ──────────────────────────────↗
```

- Primary pipeline: `scripts/duckdb_fast_pipeline.py`
- FAST headless engine: `FAST-main/Python_env/run_fast.py` (no GUI for production)
- `hazus_notinuse.py` is NOT obsolete — it is the active FAST execution engine
- `manage.py` is Windows-only (`ctypes.windll`); do not import on macOS/Linux

## Key Scripts

| Script | Purpose |
|--------|---------|
| `scripts/duckdb_fast_pipeline.py` | **Primary pipeline**: NSI Parquet → FAST CSV via DuckDB SQL |
| `scripts/download_nsi_by_state.py` | Download NSI from USACE API → Parquet |
| `scripts/nsi_raw_to_parquet.py` | Raw NSI GPKG/GeoJSON → Parquet conversion |
| `scripts/h3_spatial_index.py` | H3 hex spatial pre-filtering for raster-aware building selection |
| `scripts/slosh_to_raster.py` | SLOSH Parquet → GeoTIFF converter |
| `scripts/validate_pipeline.py` | Post-run validation: schema checks + aggregate stats |

## Data Contracts

### NSI → FAST CSV Column Mapping (see AGENTS.md §3-4)

| NSI Field | FAST Column | Notes |
|-----------|-------------|-------|
| `bid` | `FltyId` | Deduplicate before writing |
| `occtype` | `Occ` | e.g. RES1, COM1 |
| `val_struct` | `Cost` | Replacement cost ($) |
| `sqft` | `Area` | Floor area (sqft) |
| `num_story` | `NumStories` | Stories above ground |
| `found_type` | `FoundationType` | Numeric via `found_type_map`: Pier=2, Basement=4, Crawl=5, Slab=7 |
| `found_ht` | `FirstFloorHt` | Feet above grade |
| `latitude` / `longitude` | `Latitude` / `Longitude` | WGS84 |
| `val_cont` | `ContentCost` | Optional |

### SLOSH → Raster

- Geometry: `geometry_wkt` | Surge: `cN_mean`/`cN_high` (N=0..5) | Terrain: `topography`
- Inundation depth = surge elevation - topography; output GeoTIFF in feet

### FAST Runtime Parameters

- `flC`: `CoastalA` (default) | `CoastalV` (high-risk) | `Riverine` (inland)
- `raster`: path to `.tif` flood depth raster

## Configuration

- `configs/event_state_map.yaml` — hurricane → affected state mapping

## Common Commands

```bash
# Primary pipeline (DuckDB)
python scripts/duckdb_fast_pipeline.py --state Florida

# Download NSI data by state
python scripts/download_nsi_by_state.py --state Florida --engine duckdb --output-dir data

# SLOSH → raster
python scripts/slosh_to_raster.py --basin ny3mom --category 3 --tide high

# H3 spatial pre-indexing
python scripts/h3_spatial_index.py --raster path/to/raster.tif --resolution 7

# Validate pipeline output
python scripts/validate_pipeline.py --predictions path/to/output.csv
```

## Known Issues

### 99.7% Zero-Loss Spatial Mismatch (RESOLVED)

Was caused by legacy pipeline (`fast_e2e_from_oracle.py`) allowing buildings with valid FIRM zones to bypass bbox filter. The DuckDB pipeline applies raster-bbox spatial filtering to ALL buildings via SQL, resolving this issue.

### FltyId Deduplication (RESOLVED)

DuckDB pipeline handles dedup via `ROW_NUMBER() OVER (PARTITION BY bid)` in a single SQL pass.

### Partial FAST Output (MEDIUM)

`run_fast_job` checks returncode + file existence but not row count. Partial writes on crash pass the success check.

## Spatial Filtering Rules

1. `impact-only` mode: drop ALL buildings outside `raster_bbox_wgs84(raster_path)`
2. BBox is coarse; for irregular footprints consider raster valid-pixel convex hull
3. Do NOT use FIRM zones as proxy for event footprint (FIRM = long-term risk; raster = event)

## What NOT to Do

- Do not run GUI mode for production
- Do not import `manage.py` on macOS/Linux
- Do not use FIRM zone as spatial filter in `impact-only` mode
- Do not skip FltyId deduplication
- Do not expand scope beyond what is requested
- Do not ask questions answered by AGENTS.md or this file
