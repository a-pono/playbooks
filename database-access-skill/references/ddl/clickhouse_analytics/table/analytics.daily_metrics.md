# analytics.daily_metrics

Pre-aggregated daily metrics. One row per metric-dimension-date combination. Uses **SummingMergeTree** - values auto-sum during background merges, so always wrap reads in a `GROUP BY` to get correct totals.

Use this table for dashboard queries instead of scanning the raw events table.

## DDL

```sql
CREATE TABLE analytics.daily_metrics (
    date Date,
    metric_name LowCardinality(String),
    dimension String,
    value Float64
)
ENGINE = SummingMergeTree(value)
PARTITION BY toYYYYMM(date)
ORDER BY (metric_name, date, dimension)
SETTINGS index_granularity = 8192;
```

**Important:** Because this is a SummingMergeTree, duplicate inserts are summed, not replaced. Always query with:
```sql
SELECT date, metric_name, dimension, sum(value) AS value
FROM analytics.daily_metrics
WHERE ...
GROUP BY date, metric_name, dimension
```

## Statistics

Collected on: 2026-03-20

**Row count:** ~50,000,000
**Partitions:** 72 (2020-01 through 2026-03)

| Column | Nulls | Min | Max | Distinct | All Unique |
|--------|------:|-----|-----|:--------:|:----------:|
| date | 0 | 2020-01-01 | 2026-03-20 | 2,271 | no |
| metric_name | 0 | - | - | 4 | no |
| dimension | 0 | - | - | ~1,200 | no |
| value | 0 | 0.0 | 12,500,000.0 | ~48,000,000 | no |

**metric_name:** 4 distinct values
- `dau`: daily active users (value = count of unique users)
- `revenue`: daily revenue in USD (value = sum of purchase amounts)
- `signups`: new user registrations (value = count)
- `page_views`: total page views (value = count)

**dimension:** varies by metric. Examples: country code (US, GB, DE), device type (desktop, mobile, tablet), product category. Empty string = overall total (no breakdown).
