# Documentation Rules

Standards for creating and maintaining database object documentation.

## File Naming

Pattern: `<schema>.<ObjectName>.md`

The filename must uniquely identify the object within its database instance.

Examples:
- `public.orders.md` (PostgreSQL table in public schema)
- `analytics.events.md` (ClickHouse table in analytics database)
- `reporting.monthly_revenue.md` (a view)

## File Location

Place files by object type:

```
references/ddl/<database>/
├── table/
│   └── schema.TableName.md
├── view/
│   └── schema.ViewName.md
├── procedure/
│   └── schema.ProcName.md
└── function/
    └── schema.FunctionName.md
```

Only create type folders that the database actually uses. ClickHouse has no stored procedures - don't create an empty `procedure/` folder.

## Required Sections

Every DDL file must contain exactly three sections:

### 1. Description

What this object is, what data it holds, how it relates to other objects. Include:
- Purpose in plain language
- Key relationships (foreign keys, logical joins)
- Important constraints or business rules
- Anything a person querying this table for the first time needs to know

### 2. DDL

The exact `CREATE` statement from the database. Copy it directly - do not modify or simplify.

For ClickHouse, include the `ENGINE`, `PARTITION BY`, and `ORDER BY` clauses - these are critical for query planning.

### 3. Statistics

Collected from the actual database. Always record:

- **Collected on:** date when statistics were gathered (so readers know how fresh they are)
- **Row count:** approximate total rows

For each column:
- **Nulls:** count or percentage of NULL values
- **Min / Max:** value range
- **Distinct count:** number of unique values
- **All unique:** yes/no - whether every value is unique (helps identify natural keys)

**Distinct values (if < 10 unique):** List all values with a short description of what each means. This turns implicit codes into documented knowledge.

Example:
```
status: 3 distinct values
- pending: order placed, not yet processed
- completed: payment received, items shipped
- cancelled: order cancelled by user or system
```

## Rules

- **Never guess.** If you can't verify something from the database, mark it as `[ASSUMPTION - verify]`.
- **Update the registry.** After creating a new DDL file, add the object to the table/view list in the parent database reference file (`references/<database>.md`).
- **Keep statistics fresh.** If statistics are older than 30 days, consider re-collecting them before relying on them for query planning.
