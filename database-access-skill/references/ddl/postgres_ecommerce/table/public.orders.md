# public.orders

Order headers. Each row is one order placed by a user. Contains the total amount and current status. Line-level detail is in order_items.

**Relationships:**
- `user_id` -> `users.id`
- Referenced by `order_items.order_id`

## DDL

```sql
CREATE TABLE public.orders (
    id serial PRIMARY KEY,
    user_id integer NOT NULL REFERENCES users(id),
    total_amount numeric(10,2) NOT NULL,
    status varchar(20) NOT NULL DEFAULT 'pending',
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX idx_orders_user ON public.orders(user_id);
CREATE INDEX idx_orders_status ON public.orders(status);
CREATE INDEX idx_orders_created ON public.orders(created_at);
```

## Statistics

Collected on: 2026-03-20

**Row count:** ~2,000,000

| Column | Nulls | Min | Max | Distinct | All Unique |
|--------|------:|-----|-----|:--------:|:----------:|
| id | 0 | 1 | 2000000 | 2,000,000 | yes |
| user_id | 0 | 1 | 499,998 | 385,000 | no |
| total_amount | 0 | 0.99 | 12,499.95 | ~1,800,000 | no |
| status | 0 | - | - | 3 | no |
| created_at | 0 | 2020-06-15 | 2026-03-20 | ~2,000,000 | ~yes |

**status:** 3 distinct values
- `pending`: order placed, not yet processed (~120,000 rows, 6%)
- `completed`: payment received, items shipped (~1,740,000 rows, 87%)
- `cancelled`: cancelled by user or system (~140,000 rows, 7%)
