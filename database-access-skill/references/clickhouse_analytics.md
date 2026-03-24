# ClickHouse - analytics

Analytical database for historical analysis, dashboards, and aggregations. Ingests event streams from the e-commerce platform. Billions of rows - always filter by partition key (date columns) to avoid full table scans.

## Connection

```bash
source .env && clickhouse-client -h $CH_HOST --port $CH_PORT -u $CH_USER -d $CH_DATABASE --password $CH_PASSWORD
```

## Tables

| Schema | Table | ~Rows | Description |
|--------|-------|------:|-------------|
| analytics | events | 5B | User activity - clicks, purchases, views, signups. Partitioned by month on event_date. |
| analytics | page_views | 12B | Page view tracking - URL paths, referrers, time on page. Partitioned by month on view_date. |
| analytics | daily_metrics | 50M | Pre-aggregated daily stats (DAU, revenue, signups). SummingMergeTree - values auto-sum on merge. |
| analytics | sessions | 800M | User sessions with page counts and device info. |

## DDL Files

`references/ddl/clickhouse_analytics/table/*.md`
