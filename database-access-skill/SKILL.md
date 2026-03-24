---
name: database-access-skill
description: Query databases safely and accurately using CLI tools and structured documentation. Use when the user asks data questions, needs to explore database schemas, wants to run SQL queries, check table structures, find column types, analyze data, build reports, or debug data issues. Also use when the user mentions specific table names, database names, or asks questions like "how many rows", "what columns does X have", or "show me the schema."
---

# Database Access Skill

A structured approach to giving AI coding agents safe, accurate access to your databases - using markdown files and CLI tools instead of custom integrations.

## Security

Credentials in `.env` must never be read directly (`cat`, `read`, `print`, or logged) because they would leak into the LLM context window and potentially into logs or conversation history. Instead, load them into the shell environment so they're used by the CLI tool but never visible in the output:

```bash
source .env && <cli-command>
```

Use environment variable names (`$PG_HOST`, `$CH_PASSWORD`, etc.) in all commands - never hardcoded values. See `assets/.env.example` for all required variable names.

## .env Setup

1. Copy the template to your project root:
   ```bash
   cp ${CLAUDE_SKILL_DIR}/assets/.env.example .env
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

1. **Read before you query.** Before writing any SQL, read the DDL file (`references/ddl/...`) for every table you plan to use. DDL files contain column names, types, relationships, and statistics that prevent wrong assumptions and wasted queries.
2. **Use partition keys.** Large tables (events, page_views) are partitioned by date. Without a date filter, the query scans the entire table - billions of rows, minutes of wall time, and potentially an expensive mistake on a production cluster.
3. **LIMIT results.** Start with `LIMIT 100` for exploration. Only remove the limit when you specifically need the full result set.
4. **Read-only.** This skill is for querying only - no INSERT, UPDATE, DELETE, DROP, or ALTER. If the user needs to modify data, surface that as a separate conversation.
5. **Use statistics for planning.** DDL files include row counts, null rates, distinct values, and value ranges. Use these to plan joins and estimate result sizes before running queries - this is what makes the skill better than raw schema access.

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
