# public.categories

Product category hierarchy. Self-referencing tree structure via parent_id. Top-level categories have parent_id = NULL. Small lookup table - safe to SELECT * without LIMIT.

**Relationships:**
- `parent_id` -> `categories.id` (self-referencing)
- Referenced by `products.category_id`

## DDL

```sql
CREATE TABLE public.categories (
    id serial PRIMARY KEY,
    name varchar(100) NOT NULL,
    parent_id integer REFERENCES categories(id)
);
```

## Statistics

Collected on: 2026-03-20

**Row count:** 187

| Column | Nulls | Min | Max | Distinct | All Unique |
|--------|------:|-----|-----|:--------:|:----------:|
| id | 0 | 1 | 187 | 187 | yes |
| name | 0 | - | - | 187 | yes |
| parent_id | 12 (6%) | 1 | 180 | 45 | no |

**parent_id:** 12 rows with NULL = top-level categories (Electronics, Clothing, Home & Garden, Sports, Books, Toys, Health, Automotive, Food, Office, Pet Supplies, Jewelry).
