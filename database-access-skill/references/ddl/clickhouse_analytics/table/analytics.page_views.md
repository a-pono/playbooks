# analytics.page_views

Page view tracking. Every page load generates one row. Includes anonymous visitors (user_id = 0). The largest table in the system at **12 billion rows** - always filter by `view_date`.

**Logical joins:**
- `user_id` maps to PostgreSQL `users.id` (0 = anonymous visitor)

## DDL

```sql
CREATE TABLE analytics.page_views (
    view_date Date,
    view_time DateTime,
    user_id UInt64,
    url_path String,
    referrer String,
    duration_ms UInt32
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(view_date)
ORDER BY (view_date, url_path)
SETTINGS index_granularity = 8192;
```

## Statistics

Collected on: 2026-03-20

**Row count:** ~12,000,000,000
**Partitions:** 72 (2020-01 through 2026-03)

| Column | Nulls | Min | Max | Distinct | All Unique |
|--------|------:|-----|-----|:--------:|:----------:|
| view_date | 0 | 2020-01-01 | 2026-03-20 | 2,271 | no |
| view_time | 0 | 2020-01-01 00:00:00 | 2026-03-20 23:59:59 | ~12B | ~yes |
| user_id | 0 | 0 | 500,000 | 498,500 | no |
| url_path | 0 | / | - | ~85,000 | no |
| referrer | 0 | - | - | ~2,100,000 | no |
| duration_ms | 0 | 0 | 1,800,000 | ~450,000 | no |

**user_id:** 0 = anonymous visitor. ~4.8B rows (40%) are anonymous.

**url_path:** top 5 by frequency
- `/`: ~1.2B rows (10%)
- `/products/*`: ~4.8B rows (40%)
- `/cart`: ~1.1B rows (9%)
- `/checkout`: ~600M rows (5%)
- `/account/*`: ~480M rows (4%)

**duration_ms:** median ~35,000 (35 seconds). Values of 0 = bounce (page loaded but user left immediately), ~1.4B rows (12%).
