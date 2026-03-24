---
name: database-access-skill
description: Query databases safely and accurately using CLI tools and structured documentation. Use when the user asks data questions, needs to explore database schemas, or wants to run SQL queries against PostgreSQL or ClickHouse.
---

# Database Access Skill

Query databases safely and accurately using CLI tools and structured documentation.

## Security

- **NEVER read `.env` directly.** Do not `cat`, `read`, `print`, or log `.env` contents. Credentials must never appear in context.
- Before running any CLI command, load credentials into the shell environment:
  ```bash
  source .env && <cli-command>
  ```
- Use environment variable names (`$PG_HOST`, `$CH_PASSWORD`, etc.) in all commands - never hardcoded values.
- See `assets/.env.example` for all required variable names.

## .env Setup

1. Copy the template to your project root:
   ```bash
   cp assets/.env.example .env
   ```
2. Fill in real values. Use **single quotes** for passwords (handles special characters safely):
   ```bash
   PG_PASSWORD='my$ecret!pass'
   ```
3. Never commit `.env` to version control.

## Available Databases

### PostgreSQL - ecommerce

Transactional database. Live product catalog, user accounts, orders. This is where real-time data lives - current prices, active orders, latest user activity.

**Reference:** `references/postgres_ecommerce.md`

### ClickHouse - analytics

Analytical database. Event streams, page views, pre-aggregated metrics. Billions of rows. Use this for historical analysis, trends, dashboards, and any question involving aggregations over time.

**Reference:** `references/clickhouse_analytics.md`

## Query Rules

1. **Read before you query.** Before writing any SQL, read the DDL file (`references/ddl/...`) for every table you plan to use. No exceptions. Do not guess column names, types, or relationships.
2. **Use partition keys.** Large tables (events, page_views) are partitioned by date. Always include a date filter to avoid full table scans.
3. **LIMIT results.** Always add `LIMIT` unless you specifically need all rows. Start with `LIMIT 100` for exploration.
4. **Read-only.** No INSERT, UPDATE, DELETE, DROP, or ALTER. Queries only.
5. **Use statistics for planning.** DDL files include row counts, null rates, distinct values, and value ranges. Use these to plan joins and estimate result sizes before running queries.

## Workflow

When a user asks a data question:

1. Read this file (you're here)
2. Determine which database has the data based on descriptions above
3. Read the database reference file (`references/<database>.md`) for connection details and table registry
4. Read DDL files (`references/ddl/<database>/<type>/<schema.table>.md`) for every table you'll query
5. Write and execute the query using the CLI pattern from the reference file
6. If the question spans multiple databases, query each separately and combine results

## Documenting New Objects

To create documentation for a new database object:

1. Read `references/doc-rules.md` for formatting standards
2. Collect DDL and statistics from the database
3. Create the `.md` file in the correct path: `references/ddl/<database>/<type>/<schema.ObjectName>.md`
4. Add the object to the table registry in the database reference file (`references/<database>.md`)
