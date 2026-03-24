# analytics.sessions

User sessions. One row per session, defined by a 30-minute inactivity timeout. Tracks session duration and pages visited. Useful for engagement analysis and funnel metrics.

**Logical joins:**
- `user_id` maps to PostgreSQL `users.id`

## DDL

```sql
CREATE TABLE analytics.sessions (
    session_id String,
    user_id UInt64,
    started_at DateTime,
    ended_at DateTime,
    page_count UInt16,
    device LowCardinality(String)
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(toDate(started_at))
ORDER BY (user_id, started_at)
SETTINGS index_granularity = 8192;
```

## Statistics

Collected on: 2026-03-20

**Row count:** ~800,000,000
**Partitions:** 72 (2020-01 through 2026-03)

| Column | Nulls | Min | Max | Distinct | All Unique |
|--------|------:|-----|-----|:--------:|:----------:|
| session_id | 0 | - | - | ~800M | yes |
| user_id | 0 | 1 | 500,000 | 497,000 | no |
| started_at | 0 | 2020-01-01 00:00:12 | 2026-03-20 23:45:00 | ~800M | ~yes |
| ended_at | 0 | 2020-01-01 00:00:12 | 2026-03-20 23:59:58 | ~800M | ~yes |
| page_count | 0 | 1 | 312 | 290 | no |
| device | 0 | - | - | 3 | no |

**page_count:** median = 4. ~180M sessions (22%) are single-page (page_count = 1).

**device:** 3 distinct values
- `desktop`: ~360M rows (45%)
- `mobile`: ~320M rows (40%)
- `tablet`: ~120M rows (15%)
