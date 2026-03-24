# public.order_items

Line items for each order. One row per product in an order. Stores the price at the time of purchase (which may differ from the current product price). The largest table in PostgreSQL - always join with orders for date filtering.

**Relationships:**
- `order_id` -> `orders.id`
- `product_id` -> `products.id`

## DDL

```sql
CREATE TABLE public.order_items (
    id serial PRIMARY KEY,
    order_id integer NOT NULL REFERENCES orders(id),
    product_id integer NOT NULL REFERENCES products(id),
    quantity integer NOT NULL CHECK (quantity > 0),
    unit_price numeric(10,2) NOT NULL
);

CREATE INDEX idx_order_items_order ON public.order_items(order_id);
CREATE INDEX idx_order_items_product ON public.order_items(product_id);
```

## Statistics

Collected on: 2026-03-20

**Row count:** ~8,000,000

| Column | Nulls | Min | Max | Distinct | All Unique |
|--------|------:|-----|-----|:--------:|:----------:|
| id | 0 | 1 | 8000000 | 8,000,000 | yes |
| order_id | 0 | 1 | 2,000,000 | 2,000,000 | no |
| product_id | 0 | 1 | 50,000 | 48,500 | no |
| quantity | 0 | 1 | 20 | 20 | no |
| unit_price | 0 | 0.99 | 4,999.99 | 8,150 | no |

**quantity:** 20 distinct values (1-20). ~65% of rows have quantity = 1.
