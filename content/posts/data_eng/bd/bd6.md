---
title: "Big Data 6. Slowly Changing Dimensions (SCD)"
date: 2025-06-05
draft: false
ShowToc: true
tags: ["Big Data", "dbt", "basics", "modern data stack", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

## Slowly Changing Dimensions (SCD)

**Slowly Changing Dimensions (SCDs)** refer to attributes in a data warehouse dimension table that change infrequently—yet unpredictably—over time. When a source system updates a dimension attribute, we need a strategy to decide how that change will be reflected (or not) in the data warehouse. In some cases, storing historical values isn’t necessary; in others, retaining a full history is critical for reporting, auditing, or trend analysis.

There are several recognized **SCD Types**, each with its own pros and cons. Below, we cover Types 0–3 (the most common) and discuss how they manage history, along with examples and trade-offs.

---

## 1. SCD Types

### 1.1 SCD Type 0

- **Behavior**: No change is propagated to the data warehouse dimension. Once a row is inserted, it remains “locked” in time, even if the source data changes.
- **Use Case**: When historical values must never be altered in the warehouse—e.g., a fax number or legacy identifier that should remain “frozen” after initial load.
- **Example**: Since fax became completely obsolete from 2010, Airbnb’s property‐owner fax number may be recorded once but is not mission-critical if it changes. We choose not to overwrite or track fax number updates in the DW.  

---

### 1.2 SCD Type 1

- **Behavior**: Overwrite the old value in the dimension row with the new value. No history is kept.
- **Use Case**: When only the **current** value matters and historical changes are irrelevant. This is the simplest approach.
- **Columns in DW**: Same as the source—no extra “history” columns.
- **Example**:  
  - Airbnb’s “air conditioning status” (e.g., “Yes” or “No”) on a property: if a host installs or removes AC, the DW just overwrites the previous flag.  
  - Only the latest status (“has AC” / “no AC”) is preserved; past states are discarded.

> **Pros**:  
> - Simplicity & minimal storage overhead (no extra columns or rows).  
> - Fast updates (just an `UPDATE` statement).  
>
> **Cons**:  
> - You lose all historical context: no way to see what the value was yesterday or a year ago.

---

### 1.3 SCD Type 2

- **Behavior**: When a dimension attribute changes, we **insert a new row** for the new attribute value and mark the old row as “expired.” Each row carries metadata about its validity period.
- **Common Columns** (added to the DW dimension table):
  - `surrogate_key` (integer, unique for every row)
  - `natural_key` (the business key that ties back to the source, e.g., `property_id`)
  - `attribute` (e.g., `rental_price`)
  - `effective_date` / `start_date` (timestamp when this row became active)
  - `end_date` (timestamp when this row was superseded; often `NULL` for the current row)
  - `is_current` (boolean or flag indicating if this row is the “active” one)
- **Use Case**: When you must preserve a **full history** of changes for trend analysis, auditing, or regulatory reasons.
- **Example**:  
  - Airbnb’s rental price:  
    1. Host sets price \$100—DW inserts row A with `start_date = 2025-01-01`, `end_date = NULL`, `is_current = TRUE`.  
    2. Later, host changes price to \$120—DW updates row A: `end_date = 2025-06-01`, `is_current = FALSE`; then inserts row B with `start_date = 2025-06-01`, `end_date = NULL`, `is_current = TRUE`.  
    3. If they change price again, repeat: expire row B, insert a new current row C, etc.

> **Pros**:  
> - Persists a complete, time-trackable history (every price change is preserved).  
> - Enables “point-in-time” queries: “What was the price of property X on 2025-04-15?”  
>
> **Cons**:  
> - More storage (multiple rows per business key).  
> - Additional processing (INSERT + UPDATE or MERGE logic).  
> - Queries must filter on `is_current = TRUE` (for “current” data) or on date ranges (to find historical records).

---

### 1.4 SCD Type 3

- **Behavior**: Keep only a limited history—usually just the **previous** value alongside the current one. Instead of inserting new rows, we add **extra columns** to the same row to capture “old” values.
- **Common Columns** (in addition to the business attributes):
  - `attribute_current` (e.g., `room_type_current`)
  - `attribute_previous` (e.g., `room_type_previous`)
  - `effective_date` (when the current value took effect)
  - (Optionally) `previous_date` (when the old value was active, if desired)
- **Use Case**: When you need to retain a short window of history (for example, the last known value), but you don’t want to store a full audit trail.
- **Example**:  
  - Airbnb’s room type (e.g., `PRIVATE_ROOM`, `ENTIRE_PLACE`, `SHARED_ROOM`):  
    1. Initially: `room_type_current = PRIVATE_ROOM`, `room_type_previous = NULL`.  
    2. Host moves out and lists entire place: update row to `room_type_previous = PRIVATE_ROOM`, `room_type_current = ENTIRE_PLACE`, `effective_date = <today>`.  
    3. Host moves back in and switches to shared room: update to `room_type_previous = ENTIRE_PLACE`, `room_type_current = SHARED_ROOM`, etc.  
    4. **No extra rows** are created—just column updates.

> **Pros**:  
> - Lower storage overhead compared to SCD 2 (only one row per key).  
> - Simpler queries (no need to union multiple rows).  
>
> **Cons**:  
> - Only a fixed number of past values are stored (commonly only one “previous” value).  
> - Lost deep history—only current and (possibly) immediate prior state remain.

---

## 2. (Optional) Advanced SCD Types

While Types 0–3 are the most widely used, there are other specialized patterns:

- **SCD Type 4 (Historical Table)**  
  - Maintain a **separate history table** (archive) keyed by the natural/business key. The main dimension table always holds just the **current** record.  
  - Queries for historical data must join/match to the history table.  

- **SCD Type 6 (Hybrid / “Mini‐SCD”)**  
  - Combines features of Types 1, 2, and 3, e.g.:  
    - Keep one “previous” column (Type 3)  
    - Insert a new row with `is_current` flag (Type 2)  
    - Overwrite certain attributes in the main row (Type 1)  
  - Useful when you need a partial audit trail plus a “soft overwrite” for attributes that aren’t critical to track history on.

> 🔍 If you only need “current” data, use Type 1.  
> If you need full history, use Type 2.  
> If you need just a snapshot of “previous” + “current,” use Type 3.  

---

## 3. Choosing the Right SCD Type

1. **Business Requirements**  
   - Do analysts need to run **point-in-time** queries or audit past changes? → Type 2 (or 6).  
   - Is only the **current** value relevant for all reports? → Type 1 (or 0, if you never want to update DW).  
   - Do you need a short, fixed history (e.g., last value)? → Type 3.  

2. **Storage & Performance Constraints**  
   - SCD 2 requires more storage and slower writes (multiple rows, extra indexes on `business_key + effective_date`).  
   - SCD 3 has minimal overhead but very limited history.  
   - SCD 0 & 1 are cheapest in terms of storage, but lose history entirely.

3. **ETL/ELT Complexity**  
   - SCD 2 logic is more complex (MERGE or INSERT/UPDATE step, plus date/flag maintenance).  
   - SCD 1 is a simple `UPDATE` statement when source data changes.  
   - SCD 3 needs an `UPDATE` that shifts “current” → “previous” columns and then writes the new “current.”

---

## 4. Example Schema Snippets

### 4.1 SCD Type 2 Table Example (PostgreSQL)
```sql
CREATE TABLE property_price_history (
  surrogate_key      BIGSERIAL PRIMARY KEY,
  property_id        INT      NOT NULL,
  rental_price       DECIMAL(10,2) NOT NULL,
  effective_date     DATE     NOT NULL DEFAULT CURRENT_DATE,
  end_date           DATE,
  is_current         BOOLEAN  NOT NULL DEFAULT TRUE,
  -- (other dimension attributes)
  UNIQUE(property_id, effective_date)
);
```

- When rental_price changes:

``` sql
-- 1) Expire the old row
UPDATE property_price_history
SET end_date = CURRENT_DATE - INTERVAL '1 day',
    is_current = FALSE
WHERE property_id = 123
AND is_current = TRUE;

-- 2) Insert a new “current” row
INSERT INTO property_price_history (property_id, rental_price)
VALUES (123, 150.00);
```

### 4.2 SCD Type 3 Table Example

``` sql
CREATE TABLE property_room_type (
  property_id          INT      PRIMARY KEY,
  room_type_current    TEXT     NOT NULL,
  room_type_previous   TEXT,
  effective_date       DATE     NOT NULL DEFAULT CURRENT_DATE
);
```

- When `room_type` changes:
``` sql
UPDATE property_room_type
SET room_type_previous  = room_type_current,
    room_type_current   = 'ENTIRE_PLACE',
    effective_date      = CURRENT_DATE
WHERE property_id = 456;
```