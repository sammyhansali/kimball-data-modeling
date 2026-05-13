# Kimball Data Modelling

A chapter-by-chapter implementation of the dimensional models from *The Data Warehouse Toolkit* (Kimball & Ross, 3rd ed.), built as runnable dbt projects backed by DuckDB.

---

## Overview

This repo tracks hands-on work alongside reading the Kimball book. Each chapter that introduces a new dimensional model gets its own dbt project (`ch03_retail/`, `ch04_.../`, etc.) with seed data, staging models, and mart models. The goal is to go beyond reading — actually designing and building each schema, making the same decisions Kimball walks through in the text.

---

## Methods

**Stack**
- [dbt-core](https://docs.getdbt.com/) + [dbt-duckdb](https://github.com/duckdb/dbt-duckdb) — dbt for transformation logic, DuckDB as a zero-infrastructure local warehouse
- Python (managed via `pyproject.toml`) for the shared environment
- One dbt project per chapter, each with its own isolated DuckDB file

**Process**

Each chapter follows the Kimball 4-step dimensional design process:

1. **Select the business process** — identify the operational process being measured (e.g. a retail sale at the point of sale)
2. **Declare the grain** — define exactly what one row in the fact table represents; this is the most critical decision
3. **Identify the dimensions** — choose the descriptive context that surrounds each fact row (who, what, where, when, how)
4. **Identify the facts** — select the numeric, additive measures that belong at the declared grain

**Schema conventions**
- Surrogate keys (integer PKs) on all dimension tables; natural/operational keys preserved as `_id` columns
- Degenerate dimensions (operational keys with no dimension table of their own) stored directly on the fact table
- Staging layer (`stg_*`) for light cleanup before marts; no business logic in staging

---

## Projects

### Chapter 3 — Retail Sales

> *The Data Warehouse Toolkit*, Chapter 3: "Retail Sales"

#### Goals

- Build the canonical retail sales star schema from the book
- Work through all four Kimball design steps for a point-of-sale (POS) transaction
- Implement the model end-to-end in dbt: seeds → staging → dimension tables → fact table

#### Relevant Kimball Concepts

**Star schema** — a central fact table surrounded by denormalized dimension tables. Joins are simple (fact → dim on surrogate key), query performance is predictable, and business users can navigate the model without knowing SQL internals.

**Grain** — the retail sales model is at *individual line-item grain*: one row per product sold per transaction. This is the most atomic (finest) grain available from POS data. Finer grain = more flexibility; you can always aggregate up, never down.

**Surrogate keys** — dimension tables use integer surrogate keys (e.g. `product_key`) as their PK rather than the operational system's ID. This decouples the warehouse from source system changes and is required to support slowly changing dimensions later.

**Degenerate dimension** — the transaction/ticket number has no attributes of its own (no separate dimension table makes sense), so it lives directly on the fact table as `transaction_number`. It's still useful for grouping all line items from the same transaction.

**Additive facts** — measures like `quantity_sold`, `extended_sales_amount`, and `extended_gross_profit` are fully additive: you can sum them across any combination of dimensions (by store, by date, by product, etc.). This is the most valuable kind of fact.

**Tables in this model**

| Table | Type | Description |
|---|---|---|
| `dim_date` | Dimension | One row per calendar day; ~25 date attributes for slicing by week, month, quarter, holiday, etc. |
| `dim_product` | Dimension | Product catalogue with brand, category, subcategory, package type |
| `dim_store` | Dimension | Store locations with city, state, district, region, store type |
| `dim_promotion` | Dimension | Promotion mechanics — price reduction type, ad type, display type |
| `fct_retail_sales` | Fact | One row per line item sold; FKs to all four dimensions + `transaction_number` (DD) + six additive measures |

#### What I Did

<!-- Write this section after completing the implementation. -->
<!-- Describe the design decisions you made: how you declared the grain, which dimensions you chose and why, -->
<!-- any tradeoffs you hit, and how the dbt project is structured. -->

*[Your notes here]*

#### My Takeaways

<!-- What clicked? What surprised you? What would you do differently? -->
<!-- This is the most important section — the point of the exercise is what you internalize, not just what compiles. -->

*[Your notes here]*

---

<!-- Repeat the above structure for each new chapter -->

---

## Conclusion

<!-- After finishing multiple chapters, reflect here on the broader patterns: -->
<!-- What design principles kept recurring? What's the hardest part of dimensional modelling in practice? -->
<!-- How does the theory in the book hold up when you actually build it? -->

*[Your notes here]*
