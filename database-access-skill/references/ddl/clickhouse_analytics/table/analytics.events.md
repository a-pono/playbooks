# analytics.events

User activity events. Every click, purchase, view, and signup is recorded here. This is the primary fact table for behavioral analytics. **5 billion rows** - always filter by `event_date` to hit the partition and avoid scanning the entire table.

**Logical joins:**
- `user_id` maps to PostgreSQL `users.id`
- `product_id` maps to PostgreSQL `products.id` (0 = no product involved)

## DDL

```sql
CREATE TABLE analytics.events (
    event_date Date,
    event_time DateTime,
    user_id UInt64,
    event_type String,
    product_id UInt32,
    amount Decimal(10,2),
    device LowCardinality(String),
    country LowCardinality(String)
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_date)
ORDER BY (user_id, event_date, event_type)
SETTINGS index_granularity = 8192;
```

## Statistics

Collected on: 2026-03-20

**Row count:** ~5,000,000,000
**Partitions:** 72 (2020-01 through 2026-03)

| Column | Nulls | Min | Max | Distinct | All Unique |
|--------|------:|-----|-----|:--------:|:----------:|
| event_date | 0 | 2020-01-01 | 2026-03-20 | 2,271 | no |
| event_time | 0 | 2020-01-01 00:00:01 | 2026-03-20 23:59:58 | ~5B | ~yes |
| user_id | 0 | 1 | 500,000 | 498,000 | no |
| event_type | 0 | - | - | 4 | no |
| product_id | 0 | 0 | 50,000 | 50,001 | no |
| amount | 0 | 0.00 | 4,999.99 | ~8,000 | no |
| device | 0 | - | - | 3 | no |
| country | 0 | - | - | 89 | no |

**event_type:** 4 distinct values
- `view`: user viewed a product page (~3.5B rows, 70%)
- `click`: user clicked a UI element (~1B rows, 20%)
- `purchase`: completed transaction (~400M rows, 8%)
- `signup`: new account registration (~100M rows, 2%)

**device:** 3 distinct values
- `desktop`: ~2.25B rows (45%)
- `mobile`: ~2B rows (40%)
- `tablet`: ~750M rows (15%)

**product_id:** 0 means no product involved (signups, non-product clicks). ~600M rows have product_id = 0.
