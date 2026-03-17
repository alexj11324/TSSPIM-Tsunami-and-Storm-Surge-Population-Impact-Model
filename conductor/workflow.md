# Workflow Preferences

## Test Driven Development (TDD)
**Strict**: Tests are required before implementation. 
All data transformation functions and algorithm units must have accompanying unit test suites (e.g., `pytest`). When adding new features, mock the corresponding AWS S3 / Parquet payload to guarantee robust validation without external cloud dependencies.

## Commit Strategy
**Conventional Commits**: 
Always prefix commit messages indicating intent, structured as `<type>: <subject>`.
For example: `feat: add DuckDB join support for SVI data`, `fix: correct CRS projection issue in Rasterio processing`.

## Code Review
**Required for all changes**: 
Due to the critical nature of disaster data, all scripts, schema changes, and model configuration tweaks require PR creation and peer/agent review before merge.

## Verification Checkpoints
**Track completion only**:
Manual verification (where humans explicitly review logs, script outputs or schemas in Athena) is only mandatory upon completing a full track/epic. Atomic units inside a track can be verified via automated TDD suites.
