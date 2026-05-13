# ch03_retail — Design Spec

**Date:** 2026-05-12
**Scope:** Core star schema from *The Data Warehouse Toolkit* Chapter 3 (Retail Sales), with complexity added after the core is working.

---

## Stack

- dbt-core + dbt-duckdb (added to root `pyproject.toml`)
- DuckDB file at `ch03_retail/dev.duckdb` (local, zero infrastructure)
- dbt profile `ch03_retail` in `~/.dbt/profiles.yml`

---

## Project Structure

```
ch03_retail/
  seeds/
    raw_transactions.csv     # POS line items — one row per product per transaction
    raw_products.csv         # product catalog
    raw_stores.csv           # store master
    raw_promotions.csv       # promotion definitions

  models/
    staging/
      stg_transactions.sql
      stg_products.sql
      stg_stores.sql
      stg_promotions.sql

    marts/
      dim_date.sql            # generated via SQL (no seed)
      dim_product.sql
      dim_store.sql
      dim_promotion.sql
      fct_retail_sales.sql

  dbt_project.yml
```

The boilerplate `models/example/` directory is deleted.

---

## Target Star Schema

### Dimensions

**dim_date**
- `date_key` (PK, integer surrogate)
- `full_date`, `day_of_week`, `day_of_week_name`, `month`, `month_name`, `quarter`, `year`
- `is_weekend`, `is_holiday`
- ~20 total attributes; generated from a SQL date spine, no seed

**dim_product**
- `product_key` (PK), `product_id` (NK)
- `product_name`, `brand`, `category`, `subcategory`, `package_type`

**dim_store**
- `store_key` (PK), `store_id` (NK)
- `store_name`, `city`, `state`, `district`, `region`, `store_type`

**dim_promotion**
- `promotion_key` (PK), `promotion_id` (NK)
- `promotion_name`, `price_reduction_type`, `ad_type`, `display_type`

### Fact Table

**fct_retail_sales** — grain: one row per product line item per transaction

| Column | Type | Notes |
|---|---|---|
| `date_key` | FK → dim_date | |
| `product_key` | FK → dim_product | |
| `store_key` | FK → dim_store | |
| `promotion_key` | FK → dim_promotion | NULL when no promotion |
| `transaction_number` | string | Degenerate dimension — no separate dim table |
| `quantity_sold` | integer | Additive |
| `unit_price` | decimal | Additive |
| `extended_sales_amount` | decimal | Additive |
| `extended_discount_amount` | decimal | Additive |
| `extended_cost_amount` | decimal | Additive |
| `extended_gross_profit` | decimal | Additive |

---

## Staging Layer Rules

- One model per source table, one-to-one with seed columns
- Allowed: rename to snake_case, cast types, coalesce nulls
- Not allowed: joins, business logic, surrogate key assignment

---

## Learning Flow

Mode: **you design, I guide.** The implementation plan walks through the Kimball 4-step process explicitly:

1. **Select the business process** — user identifies what operational process is being measured
2. **Declare the grain** — user declares what one fact row represents; guided feedback on why this is the critical decision
3. **Identify the dimensions** — user chooses descriptive context; feedback on conformance and surrogate key rationale
4. **Identify the facts** — user selects measures; feedback on additivity

Each step: user answers → validation against Kimball → write the model.

---

## Phasing

1. **Phase 1 — Core star schema:** `dim_date`, `dim_product`, `dim_store`, `dim_promotion`, `fct_retail_sales`
2. **Phase 2 — Full Chapter 3 extensions:** additional dimensions (Customer, Cashier/Employee), factless fact table for promotions coverage — added after Phase 1 is working

---

## dbt Configuration

`dbt_project.yml` needs:
```yaml
models:
  ch03_retail:
    staging:
      +materialized: view
    marts:
      +materialized: table
```

`~/.dbt/profiles.yml` entry:
```yaml
ch03_retail:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: dev.duckdb
```
