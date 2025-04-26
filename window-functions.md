```sql
SELECT
  product_id,
  DATE_TRUNC(sale_date, DAY) sale_day,
  revenue,

  SUM(revenue) OVER (
    PARTITION BY product_id, DATE_TRUNC(sale_date, MONTH)
    ORDER BY sale_date
    RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_monthly_total,
  SUM(revenue) OVER (
    PARTITION BY product_id, DATE_TRUNC(sale_date, MONTH)
  ) AS monthly_total,
  -- Notice the difference for the default RANGE

  DATE_DIFF(DATE_TRUNC(sale_date, DAY), DATE '1990-01-01', DAY) AS day_difference,
  SUM(revenue) OVER (
    PARTITION BY product_id
    ORDER BY DATE_DIFF(DATE_TRUNC(sale_date, DAY), DATE '1990-01-01', DAY)
    RANGE BETWEEN 14 PRECEDING AND 1 PRECEDING
  ) AS PREV_14D_REVENUE,
  -- Very neat way to calculate previous 14 days revenue without densifying input

  FIRST_VALUE(revenue) OVER (
    PARTITION BY product_id, DATE_TRUNC(sale_date, MONTH)
    ORDER BY sale_date
  ) AS first_sale,
  LAST_VALUE(revenue) OVER (
    PARTITION BY product_id, DATE_TRUNC(sale_date, MONTH)
    ORDER BY sale_date
  ) AS last_sale,
  LAST_VALUE(revenue) OVER (
    PARTITION BY product_id, DATE_TRUNC(sale_date, MONTH)
    ORDER BY sale_date
    RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
  ) AS last_sale_of_product_in_a_month,
  -- RANGE/ROWS matter for LAST_VALUE

  COUNT(*) OVER(
    ORDER BY revenue
    RANGE BETWEEN 100 PRECEDING AND 100 FOLLOWING
  ) AS count_of_sales_plus_minus_100,
  --count of sales between 100 less and 100 more

  DATE_DIFF(DATE_TRUNC(sale_date, MONTH), DATE '1990-01-01', MONTH) AS month_difference,
  MAX(revenue) OVER (
    ORDER BY DATE_DIFF(DATE_TRUNC(sale_date, MONTH), DATE '1990-01-01', MONTH)
    RANGE BETWEEN 2 PRECEDING AND 1 FOLLOWING
  ) AS max_revenue_p2m_n1m --max revenue between previous 2 month and next 1 mnth

FROM interview.sales
ORDER BY product_id, sale_day
;
```

| product_id | sale_day   | revenue | running_monthly_total | monthly_total | day_difference | PREV_14D_REVENUE | first_sale | last_sale | last_sale_of_product_in_a_month | count_of_sales_plus_minus_100 | month_difference | max_revenue_p2m_n1m |
|------------|------------|---------|-----------------------|---------------|----------------|-----------------|------------|-----------|-------------------------------|-------------------------------|------------------|---------------------|
| 101        | 2024-01-20 | 1700    | 1700                  | 3250          | 12437          |                 | 1700       | 1700      | 1550                          | 2                             | 408              | 1900                |
| 101        | 2024-01-25 | 1550    | 3250                  | 3250          | 12442          | 1700            | 1700       | 1550      | 1550                          | 1                             | 408              | 1900                |
| 101        | 2024-02-10 | 1900    | 1900                  | 3650          | 12458          |                 | 1900       | 1900      | 1750                          | 1                             | 409              | 1900                |
| 101        | 2024-02-22 | 1750    | 3650                  | 3650          | 12470          | 1900            | 1900       | 1750      | 1750                          | 2                             | 409              | 1900                |
| 102        | 2024-01-15 | 900     | 900                   | 2620          | 12432          |                 | 900        | 900       | 870                           | 6                             | 408              | 1900                |
| 102        | 2024-01-18 | 850     | 1750                  | 2620          | 12435          | 900             | 900        | 850       | 870                           | 5                             | 408              | 1900                |
| 102        | 2024-01-28 | 870     | 2620                  | 2620          | 12445          | 1750            | 900        | 870       | 870                           | 5                             | 408              | 1900                |
| 102        | 2024-02-14 | 950     | 950                   | 2880          | 12462          |                 | 950        | 950       | 980                           | 6                             | 409              | 1900                |
| 102        | 2024-02-15 | 950     | 1900                  | 2880          | 12463          | 950             | 950        | 950       | 980                           | 6                             | 409              | 1900                |
| 102        | 2024-02-26 | 980     | 2880                  | 2880          | 12474          | 1900            | 950        | 980       | 980                           | 4                             | 409              | 1900                |
