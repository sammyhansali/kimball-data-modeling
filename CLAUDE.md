# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Chapter-by-chapter implementation of dimensional models from *The Data Warehouse Toolkit* (Kimball & Ross, 3rd ed.). Each chapter gets its own isolated dbt project (`ch03_retail/`, `ch04_.../`, etc.) backed by DuckDB. The goal is to design and build each star schema following the Kimball 4-step process.

## Stack

- **dbt-core + dbt-duckdb** — transformation logic with DuckDB as a zero-infrastructure local warehouse
- **Python 3.12** managed via `pyproject.toml` (uv)
- One dbt project per chapter, each with its own DuckDB file

## Commands

All dbt commands must be run from within the chapter's directory (e.g., `cd ch03_retail`):

```bash
dbt run          # build all models
dbt test         # run data tests
dbt seed         # load seed CSVs
dbt build        # seed + run + test in one pass
dbt run --select models/staging/stg_products   # run a single model
dbt clean        # remove target/ and dbt_packages/
```

Each chapter project references a dbt profile by name (e.g. `ch03_retail`) — profiles are defined in `~/.dbt/profiles.yml` and must be set up locally.

## Project Structure

Each chapter directory is a standalone dbt project:

```
ch<NN>_<name>/
  seeds/          CSV source data loaded via `dbt seed`
  models/
    staging/      stg_* models — light cleanup only, no business logic
    marts/        dim_* and fct_* models — the final star schema
  tests/          custom data tests
  snapshots/      SCD type-2 snapshots (where applicable)
  dbt_project.yml
```

New chapters are added as new top-level directories following the same layout.

## Schema Conventions

- **Staging (`stg_*`)**: one-to-one with source tables; rename columns, cast types, no joins
- **Dimensions (`dim_*`)**: surrogate integer PK (e.g. `product_key`); natural/operational key preserved as `_id`; denormalized (no snowflaking)
- **Facts (`fct_*`)**: FK columns reference dimension surrogate keys; degenerate dimensions (e.g. `transaction_number`) stored directly on the fact; measures should be fully additive
- **Degenerate dimensions**: operational keys with no standalone dimension table live as bare columns on the fact table

## Kimball Design Process

Each chapter model is designed following these four steps in order:

1. **Select the business process** — the operational process being measured
2. **Declare the grain** — exactly what one fact row represents (most critical decision; finest available grain preferred)
3. **Identify the dimensions** — descriptive context (who, what, where, when, how)
4. **Identify the facts** — numeric, additive measures at the declared grain
