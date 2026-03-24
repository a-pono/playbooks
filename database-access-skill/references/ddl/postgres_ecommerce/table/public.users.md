# public.users

User accounts. One row per registered user. Email is the login identifier and is unique. Country is stored as ISO 3166-1 alpha-2 code.

**Relationships:**
- Referenced by `orders.user_id`

## DDL

```sql
CREATE TABLE public.users (
    id serial PRIMARY KEY,
    email varchar(255) NOT NULL UNIQUE,
    name varchar(255) NOT NULL,
    country char(2) NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX idx_users_country ON public.users(country);
CREATE INDEX idx_users_created ON public.users(created_at);
```

## Statistics

Collected on: 2026-03-20

**Row count:** ~500,000

| Column | Nulls | Min | Max | Distinct | All Unique |
|--------|------:|-----|-----|:--------:|:----------:|
| id | 0 | 1 | 500000 | 500,000 | yes |
| email | 0 | - | - | 500,000 | yes |
| name | 0 | - | - | 412,000 | no |
| country | 0 | AE | ZW | 89 | no |
| created_at | 0 | 2020-06-15 | 2026-03-20 | ~500,000 | ~yes |

**country:** 89 distinct values (top 5 by frequency)
- `US`: ~210,000 rows (42%)
- `GB`: ~45,000 rows (9%)
- `DE`: ~30,000 rows (6%)
- `CA`: ~25,000 rows (5%)
- `FR`: ~20,000 rows (4%)
