# Immediate Tsunami and Storm Surge Population Impact Modeling

CMU Heinz MSPPM 2026 Capstone Project for the American Red Cross.

Property-level storm surge/tsunami impact modeling using FEMA's FAST tool, USACE National Structure Inventory (30M+ buildings), and NOAA SLOSH surge models. Estimates building damage, displaced population, and high-need populations to inform Red Cross shelter and casework planning.

## Architecture

```
NSI Parquet → DuckDB: clean/filter/dedup → FAST CSV ─┐
NHC P-Surge GeoTIFF (.tif) ──────────────────────────┤→ FAST engine → damage predictions
```

See `docs/pipeline_flowchart.md` for the full Mermaid diagram.

## Prerequisites

- Python 3.10+
- FAST engine (`FAST-main/Python_env/run_fast.py`)

```bash
pip install pyarrow rasterio pyyaml h3 duckdb geopandas
```

## Quick Start

```bash
# Run the primary pipeline for a single state
python scripts/duckdb_fast_pipeline.py --state Florida

# Download NSI data by state
python scripts/download_nsi_by_state.py --state Florida --engine duckdb --output-dir data

# Convert SLOSH to raster
python scripts/slosh_to_raster.py --basin ny3mom --category 3 --tide high
```

## Project Structure

```
scripts/
  duckdb_fast_pipeline.py   # Primary pipeline: NSI Parquet → FAST CSV → FAST
  download_nsi_by_state.py  # NSI API download → Parquet
  nsi_raw_to_parquet.py     # Raw NSI → processed Parquet conversion
  h3_spatial_index.py       # H3 hex spatial pre-filtering
  slosh_to_raster.py        # SLOSH Parquet → GeoTIFF converter
  validate_pipeline.py      # Post-run validation
configs/
  event_state_map.yaml      # Hurricane → state mapping
docs/
  pipeline_flowchart.md     # Architecture diagram
notebooks/
  shelter_demand.ipynb      # Colab: tract-level shelter demand (Pipeline 3)
FAST-main/
  Python_env/run_fast.py    # FAST headless engine
```

## Data Sources

| Source | Description | Format |
|--------|-------------|--------|
| NSI | USACE National Structure Inventory 2022 | Parquet, partitioned by state |
| SLOSH | NOAA MOM surge grids | Parquet, partitioned by basin |
| SVI | CDC Social Vulnerability Index | Census tract level |

## Key Documentation

- `CLAUDE.md` — Agent instructions, data contracts, known issues
- `AGENTS.md` — Execution contract and column mapping rules
- `NSI_DATA_DICTIONARY_EN.md` / `SLOSH_DATA_DICTIONARY_EN.md` — Field definitions

## Output

Per-building: `BldgDmgPct` (% damaged), `BldgLossUSD` ($ loss), `Depth_in_Struc` (ft). These feed into population disruption and Red Cross service demand estimates.

---

## Prediction Results

Results for 9 hurricane events × 3 advisories (27 runs, ~3.9M building predictions):

**Coverage**

| Event | Advisories | Buildings | Notes |
|-------|-----------|-----------|-------|
| BERYL_2024 | 39, 40, 41 | ~107K each | TX/LA Gulf Coast |
| DEBBY_2024 | 18, 19, 20 | ~103K each | FL/GA/NC/SC/VA |
| FLORENCE_2018 | 63, 64, 65 | 17K–32K | NC/SC/VA Atlantic |
| HELENE_2024 | 14, 15, 16 | 240K–475K | FL/GA/NC/SC |
| IAN_2022 | 31, 32, 33 | ~119K–122K | FL/NC/SC |
| IDALIA_2023 | 18, 19, 20 | 62K–124K | FL/GA/SC |
| IDA_2021 | 16, 17, 18 | ~412K each | AL/LA/MS |
| MICHAEL_2018 | 20, 21, 22 | ~900 each | Coastal GA (small raster footprint) |
| MILTON_2024 | 20, 21, 22 | 70K–208K | FL |

### Output Column Reference

**Building Attributes**

| Column | Description |
|--------|-------------|
| `FltyId` | NSI unique building ID |
| `Occ` | Occupancy type (RES1=single-family, RES3=multi-family, COM1=commercial, …) |
| `Cost` | Replacement cost ($) |
| `Area` | Floor area (sqft) |
| `NumStories` | Stories above ground |
| `FoundationType` | 2=Pier, 4=Basement, 5=Crawlspace, 7=Slab |
| `FirstFloorHt` | First floor height above grade (ft) |
| `Latitude` / `Longitude` | WGS84 coordinates |
| `state` | State name |

**Flood Depth**

| Column | Description |
|--------|-------------|
| `Depth_Grid` | Surge depth from SLOSH raster at building location (ft) |
| `Depth_in_Struc` | Effective depth inside structure = `Depth_Grid` − `FirstFloorHt` (ft) |

**Damage & Loss**

| Column | Description |
|--------|-------------|
| `BldgDmgPct` | Structural damage percentage (%) |
| `BldgLossUSD` | Structural loss ($) |
| `ContentCost` | Contents replacement value ($) |
| `ContDmgPct` | Contents damage percentage (%) |
| `ContentLossUSD` | Contents loss ($) |
| `InventoryLossUSD` | Inventory loss ($ — commercial buildings) |

**Debris & Recovery**

| Column | Description |
|--------|-------------|
| `Debris_Fin` | Finish debris (tons) |
| `Debris_Struc` | Structural debris (tons) |
| `Debris_Found` | Foundation debris (tons) |
| `Debris_Tot` | Total debris (tons) |
| `Restor_Days_Min` / `Restor_Days_Max` | Estimated restoration days (range) |

**Partition & Provenance**

| Column | Description |
|--------|-------------|
| `event` | Hurricane event slug |
| `adv` | Advisory number |
| `raster_name` | Source SLOSH raster filename |
| `run_id` | Pipeline run ID (timestamp-based) |
| `flc` | Flood class: CoastalA / CoastalV / Riverine |

---

## Team

CMU Heinz College — Master of Science in Public Policy and Management, 2026
