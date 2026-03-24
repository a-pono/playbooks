# PostgreSQL - ecommerce

Transactional database for a B2C e-commerce platform. Stores the live product catalog, user accounts, orders, and reference data. This is the source of truth for current state - active prices, open orders, real-time inventory.

## Connection

```bash
source .env && psql -h $PG_HOST -p $PG_PORT -U $PG_USER -d $PG_DATABASE
```

## Tables

| Schema | Table | ~Rows | Description |
|--------|-------|------:|-------------|
| public | products | 50K | Product catalog - names, prices, active/inactive status |
| public | users | 500K | User accounts - email, name, country, registration date |
| public | orders | 2M | Order headers - totals, status (pending/completed/cancelled) |
| public | order_items | 8M | Line items per order - product, quantity, price at purchase time |
| public | categories | 200 | Product category hierarchy - self-referencing parent_id |

## DDL Files

`references/ddl/postgres_ecommerce/table/*.md`
