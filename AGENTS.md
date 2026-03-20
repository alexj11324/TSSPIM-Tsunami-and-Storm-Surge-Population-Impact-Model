# ARC Capstone Agent Execution Contract

This file defines hard execution rules for agents working in this repository.
Follow these rules by default unless the user explicitly overrides them.

## 1. Project Goal (Do Not Drift)

Primary production path:

1. Use NSI processed data to build FAST-ready building inventory CSV.
2. Use NHC P-Surge GeoTIFF rasters (downloaded directly from NHC) as flood depth input.
3. Run FAST headless (no GUI) and generate FAST output CSV artifacts.

Do not introduce unrelated architecture changes unless requested.

## 2. Default Runtime Path (Authoritative)

Use headless entrypoint:

- `FAST-main/Python_env/run_fast.py`

Do not require GUI for production runs.

## 3. FAST Input Contract (Production)

### 3.1 Required building columns in FAST CSV

Keep and populate these columns:

1. `FltyId` (or mapped to `UserDefinedFltyId`)
2. `Occ`
3. `Cost`
4. `Area`
5. `NumStories`
6. `FoundationType`
7. `FirstFloorHt`
8. `Latitude`
9. `Longitude`

### 3.2 Optional columns (can be omitted)

1. `ContentCost`
2. `BDDF_ID`
3. `CDDF_ID`
4. `IDDF_ID`
5. `InvCost`
6. `SOID`

### 3.3 Non-column runtime parameters (still required at run time)

1. `flC` (`Riverine` / `CoastalA` / `CoastalV`)
2. `raster` (`.tif` path/name)

## 4. NSI/SLOSH Source Mapping Rules

### 4.1 NSI -> FAST CSV (canonical mapping)

1. `bid` -> `FltyId`
2. `occtype` -> `Occ`
3. `val_struct` -> `Cost`
4. `sqft` -> `Area`
5. `num_story` -> `NumStories`
6. `found_type` -> `FoundationType` (must be normalized to FAST-expected numeric code)
7. `found_ht` -> `FirstFloorHt`
8. `latitude` -> `Latitude`
9. `longitude` -> `Longitude`
10. `val_cont` -> `ContentCost` (optional)

### 4.2 P-Surge Rasters

Rasters are downloaded directly from NHC as GeoTIFF (`.tif`) in feet. No SLOSH-to-raster conversion needed.

## 5. Default Hazard Choice Policy

For coastal storm-surge workflows in this repo:

1. Default baseline: `CoastalA`
2. Sensitivity/high-risk run: `CoastalV`
3. Use `Riverine` only for inland/riverine tasks

If user does not specify and the task is SLOSH-driven, use `CoastalA`.

## 6. Execution Behavior Rules (No Low-Value Questions)

Do not ask obvious or repetitive questions when the repository context already answers them.
Instead, proceed using defaults and document assumptions briefly.

### 6.1 Questions that should NOT be asked

1. Asking whether FAST needs raster (it does).
2. Asking whether `flC` is required at run time (it is).
3. Asking for fields already defined in this contract.
4. Asking whether to use GUI for production runs (do not).

### 6.2 When questions ARE allowed

Ask only when blocked by missing irrecoverable inputs, such as:

1. Missing target region or time/event scope needed to pick source partitions.
2. Missing raster scenario selection when multiple outputs are explicitly required.
3. Missing credentials/access needed to read data sources.

When asking, provide exactly what is missing and a recommended default.

### 6.3 Self-Serve Before Asking (Global)

Before asking any question, you must first search for relevant tools, CLI commands, and files in this repo.
Do not ask for information that is discoverable from tools, the CLI, or the filesystem.

## 7. Output Standards

1. Prefer deterministic, reproducible scripts and commands.
2. Keep production data schemas explicit.
3. Avoid introducing optional complexity unless requested.
4. Summarize results with concrete artifacts and paths.

## 8. Guardrails

1. Do not silently alter business assumptions.
2. Do not switch data model without explicit request.
3. Do not expand scope to OCI/DB refactors unless user asks.
4. Keep changes focused on NSI -> FAST CSV, P-Surge rasters, and FAST execution.

## 9. Skill Routing for Repository Organization

When user intent is repository organization, default to the `repo-ia-reorg` skill without requiring explicit skill mention.

### 9.1 Auto-trigger keywords

Trigger on organization intents including:

1. "organize", "organization", "tidy", "clean up", "declutter"
2. "reorg", "reorganize", "restructure"
3. Chinese intents like "组织", "整理", "重整", "信息架构重排", "代码库太乱"

### 9.2 `rebase` disambiguation rule

`rebase` alone is not a trigger. Use `repo-ia-reorg` only when `rebase` appears with structure-cleanup context, such as:

1. root/docs/path/layout reorganization
2. file moves, archive consolidation, or IA cleanup

If `rebase` is requested as a pure git-history operation, do not trigger this skill.
