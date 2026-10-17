# Window function notes

## 1. Row numbering

| Function | Ties | Gaps |
|---|---|---|
| ROW_NUMBER() | none  | none |
| RANK()       | shared | yes  |
| DENSE_RANK() | shared | no   |

```sql
-- top-3 products by revenue per region, no gaps after ties
SELECT region, product, revenue,
       DENSE_RANK() OVER (PARTITION BY region ORDER BY revenue DESC) AS rnk
FROM sales;
```

## 2. Lag / lead — change detection

```sql
-- daily change; the first day reports NULL
SELECT date, close,
       close - LAG(close, 1) OVER (ORDER BY date) AS chg,
       close > LAG(close, 1) OVER (ORDER BY date) AS up
FROM prices;
```

## 3. Running totals (the one that surprises people)

```sql
SELECT date, amount,
       SUM(amount) OVER (ORDER BY date
                          ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cum
FROM orders;
```

Without the ROWS BETWEEN frame, two rows on the same date both get the same
cumulative value. The frame clause pins the total to "up to this row".

## 4. N-th occurrence per group

```sql
-- each customer's second order
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id
                               ORDER BY ordered_at) AS n
  FROM orders
) t WHERE n = 2;
```

## 5. Percentiles

```sql
-- p50 / p95 of latency per service
SELECT service,
       PERCENTILE_CONT(0.50) OVER (PARTITION BY service) AS p50,
       PERCENTILE_CONT(0.95) OVER (PARTITION BY service) AS p95
FROM requests;
```

PostgreSQL only — MySQL 8 has no percentile window; emulate with a CTE over
ranked rows if you need it there.
