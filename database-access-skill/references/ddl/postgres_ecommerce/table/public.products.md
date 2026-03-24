# public.products

Product catalog. Each row is a single product with its current price and category assignment. Products can be deactivated (is_active = false) but are never deleted - historical order_items reference them.

**Relationships:**
- `category_id` -> `categories.id`
- Referenced by `order_items.product_id`

## DDL

```sql
CREATE TABLE public.products (
    id serial PRIMARY KEY,
    name varchar(255) NOT NULL,
    price numeric(10,2) NOT NULL,
    category_id integer NOT NULL REFERENCES categories(id),
    is_active boolean NOT NULL DEFAULT true,
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX idx_products_category ON public.products(category_id);
CREATE INDEX idx_products_active ON public.products(is_active) WHERE is_active = true;
```

## Statistics

Collected on: 2026-03-20

**Row count:** ~50,000

| Column | Nulls | Min | Max | Distinct | All Unique |
|--------|------:|-----|-----|:--------:|:----------:|
| id | 0 | 1 | 50000 | 50000 | yes |
| name | 0 | - | - | 49,850 | ~yes |
| price | 0 | 0.99 | 4,999.99 | 8,200 | no |
| category_id | 0 | 1 | 187 | 187 | no |
| is_active | 0 | false | true | 2 | no |
| created_at | 0 | 2019-01-03 | 2026-03-19 | ~50,000 | ~yes |

**is_active:** 2 distinct values
- `true`: product listed and available for purchase (~42,000 rows)
- `false`: delisted, not shown in catalog but preserved for order history (~8,000 rows)
