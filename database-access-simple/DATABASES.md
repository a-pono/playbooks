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

**Columns:**

`products`:
| Column | Type | Description |
|--------|------|-------------|
| id | serial (PK) | Product ID |
| name | varchar(255) | Product display name |
| price | numeric(10,2) | Current price in USD |
| category_id | int (FK) | References categories.id |
| is_active | boolean | Whether product is listed |
| created_at | timestamptz | When product was added |

`users`:
| Column | Type | Description |
|--------|------|-------------|
| id | serial (PK) | User ID |
| email | varchar(255) | Unique, login identifier |
| name | varchar(255) | Display name |
| country | char(2) | ISO country code |
| created_at | timestamptz | Registration date |

`orders`:
| Column | Type | Description |
|--------|------|-------------|
| id | serial (PK) | Order ID |
| user_id | int (FK) | References users.id |
| total_amount | numeric(10,2) | Order total in USD |
| status | varchar(20) | pending, completed, cancelled |
| created_at | timestamptz | When order was placed |

`order_items`:
| Column | Type | Description |
|--------|------|-------------|
| id | serial (PK) | Line item ID |
| order_id | int (FK) | References orders.id |
| product_id | int (FK) | References products.id |
| quantity | int | Units ordered |
| unit_price | numeric(10,2) | Price at time of purchase |

`categories`:
| Column | Type | Description |
|--------|------|-------------|
| id | serial (PK) | Category ID |
| name | varchar(100) | Category name |
| parent_id | int (FK, nullable) | Self-referencing for hierarchy |

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

**Columns:**

`events`:
| Column | Type | Description |
|--------|------|-------------|
| event_date | Date | Partition key, date of event |
| event_time | DateTime | Exact timestamp (UTC) |
| user_id | UInt64 | User identifier |
| event_type | String | click, purchase, view, signup |
| product_id | UInt32 | Product involved (0 if N/A) |
| amount | Decimal(10,2) | Transaction amount (USD, 0 if non-purchase) |
| device | LowCardinality(String) | desktop, mobile, tablet |
| country | LowCardinality(String) | ISO country code |

`page_views`:
| Column | Type | Description |
|--------|------|-------------|
| view_date | Date | Partition key |
| view_time | DateTime | Exact timestamp (UTC) |
| user_id | UInt64 | User identifier (0 if anonymous) |
| url_path | String | Page path (e.g. /products/123) |
| referrer | String | Referring URL (empty if direct) |
| duration_ms | UInt32 | Time on page in milliseconds |

`daily_metrics`:
| Column | Type | Description |
|--------|------|-------------|
| date | Date | Partition key |
| metric_name | LowCardinality(String) | dau, revenue, signups, page_views |
| dimension | String | Breakdown dimension value |
| value | Float64 | Metric value (auto-summed by engine) |

`sessions`:
| Column | Type | Description |
|--------|------|-------------|
| session_id | String | Unique session identifier |
| user_id | UInt64 | User identifier |
| started_at | DateTime | Session start (UTC) |
| ended_at | DateTime | Session end (UTC) |
| page_count | UInt16 | Pages viewed in session |
| device | LowCardinality(String) | desktop, mobile, tablet |

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

## Cross-Source Queries (ClickHouse + PostgreSQL + Local Files)

ClickHouse can query external sources directly - no ETL, no data copying:
- **PostgreSQL** via `postgresql()` table function
- **Local files** (CSV, Parquet, JSON) via `file()` or `clickhouse-local`

You can join all three in a single query:

```sql
SELECT
    e.event_date,
    p.name AS product_name,
    c.name AS category,
    f.segment AS user_segment,
    count() AS purchases
FROM events e
JOIN postgresql('pg-host:5432', 'ecommerce', 'products', 'app_user', 'pass') AS p
    ON e.product_id = p.id
JOIN postgresql('pg-host:5432', 'ecommerce', 'categories', 'app_user', 'pass') AS c
    ON p.category_id = c.id
JOIN file('user_segments.csv', CSVWithNames) AS f
    ON e.user_id = f.user_id
WHERE e.event_date >= today() - 7
  AND e.event_type = 'purchase'
GROUP BY e.event_date, p.name, c.name, f.segment
ORDER BY purchases DESC
```

One query. Three sources. No middleware.

**Simpler examples:**

```sql
-- Just Postgres lookup
SELECT e.event_type, p.name AS product_name
FROM events e
JOIN postgresql('pg-host:5432', 'ecommerce', 'products', 'app_user', 'pass') AS p
    ON e.product_id = p.id
WHERE e.event_date = today();

-- Just local file query (no server needed)
clickhouse-local -q "
    SELECT product_id, sum(amount) AS total
    FROM file('exports/sales_2024.csv', CSVWithNames)
    GROUP BY product_id
    ORDER BY total DESC
    LIMIT 10
"
```

**Supported file formats:** CSV, CSVWithNames, TSV, Parquet, JSONEachRow, and many more.

**When to use:**
- Ad-hoc analytics enriched with reference data from Postgres
- One-off queries combining server data with local CSV/Parquet exports
- Exploratory analysis where you need data from multiple sources fast

**Production note:**
The `postgresql()` function pulls data over the network on every query. For small lookup tables (categories, products) this is fine. For large tables or repeated/dashboard queries, materialize the data into a ClickHouse [Dictionary](https://clickhouse.com/docs/en/sql-reference/dictionaries) or schedule a periodic sync instead. Similarly, `file()` on the server requires files to be in the `user_files` directory - use `clickhouse-local` for arbitrary file paths.

---

## Quick Reference

| Need | Tool | Example |
|------|------|---------|
| Query Postgres | `psql` | `psql -h pg-host -d ecommerce -c "SELECT ..."` |
| Query ClickHouse | `clickhouse-client` | `clickhouse-client -q "SELECT ..."` |
| Query local files | `clickhouse-local` | `clickhouse-local -q "SELECT * FROM file('data.csv', CSVWithNames)"` |
| Join CH + Postgres | `clickhouse-client` | Use `postgresql()` table function inside CH query |
| Join CH + files | `clickhouse-client` | Use `file()` function inside CH query |
