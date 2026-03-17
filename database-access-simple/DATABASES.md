# Database Access Guide

This file describes all available databases, how to connect, and how to query them.
Read this before running any database queries.

---

## PostgreSQL (Transactional)

**Purpose:** Product catalog, user accounts, orders, reference/dictionary data.

**CLI tool:** `psql`

**Connection:**
```bash
psql -h pg-host -p 5432 -U app_user -d ecommerce
```

**Key tables:**

| Table | Description | Row count |
|-------|------------|-----------|
| `products` | Product catalog with pricing | ~50K |
| `users` | User accounts and profiles | ~500K |
| `orders` | Order headers | ~2M |
| `order_items` | Line items per order | ~8M |
| `categories` | Product categories (dictionary) | ~200 |

**Relationships:**
- `orders.user_id` -> `users.id`
- `order_items.order_id` -> `orders.id`
- `order_items.product_id` -> `products.id`
- `products.category_id` -> `categories.id`

**Common queries:**
```sql
-- Active products with category name
SELECT p.id, p.name, p.price, c.name AS category
FROM products p
JOIN categories c ON p.category_id = c.id
WHERE p.is_active = true;

-- Recent orders for a user
SELECT o.id, o.created_at, o.total_amount
FROM orders o
WHERE o.user_id = 12345
ORDER BY o.created_at DESC
LIMIT 10;
```

**Notes:**
- All timestamps are UTC
- Use `LIMIT` on large tables - `order_items` has 8M rows
- Read-only access - no INSERT/UPDATE/DELETE

---

## ClickHouse (Analytics)

**Purpose:** Event analytics, aggregations, time-series data. Billions of rows.

**CLI tool:** `clickhouse-client`

**Connection:**
```bash
clickhouse-client -h ch-host --port 9000 -u analyst -d analytics --password
```

**Key tables:**

| Table | Description | Row count | Engine |
|-------|------------|-----------|--------|
| `events` | User activity events | ~5B | MergeTree |
| `page_views` | Page view tracking | ~12B | MergeTree |
| `daily_metrics` | Pre-aggregated daily stats | ~50M | SummingMergeTree |
| `sessions` | User sessions | ~800M | MergeTree |

**Partitioning:**
- `events` - partitioned by `toYYYYMM(event_date)`, ordered by `(user_id, event_date, event_type)`
- `page_views` - partitioned by `toYYYYMM(view_date)`, ordered by `(view_date, url_path)`
- `daily_metrics` - partitioned by `toYYYYMM(date)`, ordered by `(metric_name, date)`

**Common queries:**
```sql
-- Daily active users for the last 7 days
SELECT event_date, uniqExact(user_id) AS dau
FROM events
WHERE event_date >= today() - 7
GROUP BY event_date
ORDER BY event_date;

-- Top pages by views this month
SELECT url_path, count() AS views
FROM page_views
WHERE view_date >= toStartOfMonth(today())
GROUP BY url_path
ORDER BY views DESC
LIMIT 20;

-- Hourly event breakdown
SELECT toStartOfHour(event_time) AS hour, event_type, count() AS cnt
FROM events
WHERE event_date = today()
GROUP BY hour, event_type
ORDER BY hour, cnt DESC;
```

**Performance tips:**
- Always filter by partition key (`event_date`, `view_date`) first - this avoids full table scans
- Use `uniqExact()` for precise counts, `uniq()` for approximations (faster on billions)
- Prefer `count()` over `count(*)` in ClickHouse
- For large result sets, add `FORMAT TabSeparated` or `FORMAT JSON` to control output

**Notes:**
- Read-only access
- Queries on `events` or `page_views` without a date filter WILL be slow (billions of rows)

---

## Cross-Source Joins (ClickHouse + PostgreSQL)

ClickHouse can query PostgreSQL tables directly using the `postgresql()` table function.
Use this when you need reference/dictionary data from Postgres inside an analytical query.

**Pattern:**
```sql
SELECT
    e.event_type,
    e.event_date,
    p.name AS product_name,
    p.price,
    c.name AS category
FROM events e
JOIN postgresql('pg-host:5432', 'ecommerce', 'products', 'app_user', 'password') AS p
    ON e.product_id = p.id
JOIN postgresql('pg-host:5432', 'ecommerce', 'categories', 'app_user', 'password') AS c
    ON p.category_id = c.id
WHERE e.event_date = today()
  AND e.event_type = 'purchase';
```

**When to use:**
- Enriching analytics with product names, user details, category labels
- Ad-hoc queries where you need the latest reference data
- Avoid for high-frequency queries - each call hits Postgres in real time

**When NOT to use:**
- If the Postgres table has millions of rows (will be slow over network)
- For dashboards or repeated queries - materialize the data into a ClickHouse dictionary instead

---

## Local File Queries (ClickHouse)

ClickHouse can query CSV, Parquet, and JSON files directly.

**Using `clickhouse-local` (no server needed):**
```bash
clickhouse-local -q "
    SELECT product_id, sum(amount) AS total
    FROM file('exports/sales_2024.csv', CSVWithNames)
    GROUP BY product_id
    ORDER BY total DESC
    LIMIT 10
"
```

**Using `clickhouse-client` with server (files must be in `user_files` directory):**
```sql
SELECT *
FROM file('enrichment.csv', CSVWithNames)
WHERE region = 'US';
```

**Joining server data with local files:**
```sql
SELECT
    e.event_date,
    e.user_id,
    f.segment
FROM events e
JOIN file('user_segments.csv', CSVWithNames) f
    ON e.user_id = f.user_id
WHERE e.event_date >= today() - 30;
```

**Supported formats:** CSV, CSVWithNames, TSV, Parquet, JSONEachRow, and many more.

---

## Quick Reference

| Need | Tool | Example |
|------|------|---------|
| Query Postgres | `psql` | `psql -h pg-host -d ecommerce -c "SELECT ..."` |
| Query ClickHouse | `clickhouse-client` | `clickhouse-client -q "SELECT ..."` |
| Query local files | `clickhouse-local` | `clickhouse-local -q "SELECT * FROM file('data.csv', CSVWithNames)"` |
| Join CH + Postgres | `clickhouse-client` | Use `postgresql()` table function inside CH query |
| Join CH + files | `clickhouse-client` | Use `file()` function inside CH query |
