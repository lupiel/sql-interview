1. Retrieve the Nth highest salary within each department.
```sql
WITH cte_ranked_salaries AS(
  SELECT 
    department_id,
    salary,
    row_number() over (partition by department_id order by salary) as rnk
  FROM interview.employees
)
SELECT department_id, salary, rnk
FROM cte_ranked_salaries
WHERE rnk = 2
;
```
| department_id | salary | rnk |
|---------------|--------|-----|
| 2             | 90000  | 2   |
| 1             | 90000  | 2   |
| 3             | 75000  | 2   |

2. Calculate Year-over-Year (YoY) and Month-over-Month (MoM) growth percentages using SQL.
```sql
WITH cte_mths AS (
  SELECT mth
  FROM UNNEST(GENERATE_DATE_ARRAY('2023-01-01',
    '2024-12-01', INTERVAL 1 MONTH)) AS mth 
),
cte_products AS (
  SELECT DISTINCT product_id
  FROM interview.sales
),
cte_month_sales AS (
  SELECT
    product_id,
    DATE_TRUNC(sale_date, MONTH) sale_month,
    SUM(revenue) as revenue
  FROM
    interview.sales
  GROUP BY 1, 2
),
cte_month_ago_sales AS (
  SELECT
    p.product_id,
    m.mth as sale_month,
    revenue,
    LAG(revenue) OVER(
      PARTITION BY p.product_id
      ORDER BY m.mth
    ) revenue_month_ago
  FROM cte_mths m
  CROSS JOIN cte_products p
  LEFT JOIN cte_month_sales s
  ON m.mth = s.sale_month
  AND p.product_id = s.product_id
)
SELECT
  product_id,
  sale_month,
  revenue,
  revenue_month_ago,
  ROUND(100 * (revenue - revenue_month_ago)/revenue_month_ago) AS pct_growth_mom
FROM cte_month_ago_sales
WHERE revenue IS NOT NULL
;
```
| product_id | sale_month | revenue | revenue_month_ago | pct_growth_mom |
|------------|------------|---------|-------------------|----------------|
| 101        | 2024-01-01 | 3250    |                   |                |
| 101        | 2024-02-01 | 3650    | 3250              | 12.0           |
| 102        | 2024-01-01 | 2620    |                   |                |
| 102        | 2024-02-01 | 2880    | 2620              | 10.0           |


3. Pivot rows into columns based on specific field values.
```sql
SELECT *
FROM (
  SELECT 
    product_id,
    FORMAT_DATE('%Y-%m', sale_date) AS sale_month,
    revenue
  FROM interview.sales
)
PIVOT(
  SUM(revenue) FOR sale_month IN ('2023-01', '2023-02', '2023-03', '2024-01', '2024-02')
)
ORDER BY product_id;
```
| product_id | 2023-01 | 2023-02 | 2023-03 | 2024-01 | 2024-02 |
|------------|---------|---------|---------|---------|---------|
| 101        |         |         |         | 3250    | 3650    |
| 102        |         |         |         | 2620    | 2880    |

4. Assign employee levels (e.g., L1, L2, etc.) based on salary ranges.
```sql
SELECT 
  emp_id,
  emp_name,
  salary,
  NTILE(7) OVER(PARTITION BY department_id ORDER BY salary) as n7tile,
  "L" || (3 + NTILE(7) OVER(PARTITION BY department_id ORDER BY salary)) AS level
FROM interview.employees
ORDER BY department_id, salary
;
```
| emp_id | emp_name | salary | n7tile | level |
|--------|----------|--------|--------|-------|
| 3      | Charlie  | 85000  | 1      | L4    |
| 2      | Bob      | 90000  | 2      | L5    |
| 1      | Alice    | 120000 | 3      | L6    |
| 6      | Frank    | 85000  | 1      | L4    |
| 5      | Eve      | 90000  | 2      | L5    |
| 4      | Dave     | 95000  | 3      | L6    |
| 9      | Ivan     | 70000  | 1      | L4    |
| 8      | Heidi    | 75000  | 2      | L5    |
| 7      | Grace    | 100000 | 3      | L6    |

5. Retrieve each manager’s name along with the number of employees in their department, including managers with zero employees.
```sql
SELECT
  m.emp_id AS manager_id,
  m.emp_name,
  COUNT(e.emp_id) AS direct_subordinates_cnt
FROM interview.employees m
LEFT JOIN interview.employees e
ON m.emp_id = e.manager_id
GROUP BY
  m.emp_id,
  m.emp_name
ORDER BY 3 desc
;
```
| manager_id | emp_name | direct_subordinates_cnt |
|------------|----------|--------------------------|
| 1          | Alice    | 2                        |
| 2          | Bob      | 2                        |
| 7          | Grace    | 1                        |
| 4          | Dave     | 1                        |
| 8          | Heidi    | 1                        |
| 3          | Charlie  | 0                        |
| 5          | Eve      | 0                        |
| 6          | Frank    | 0                        |
| 9          | Ivan     | 0                        |


```sql
WITH RECURSIVE employee_hierarchy AS (
  -- Base case: Direct reports
  SELECT
    emp_id AS manager_id,
    emp_id AS employee_id
  FROM interview.employees

  UNION ALL

  -- Recursive step: Find employees reporting to current employee
  SELECT
    h.manager_id,
    e.emp_id
  FROM employee_hierarchy h
  JOIN interview.employees e
  ON h.employee_id = e.manager_id
)
SELECT
  m.emp_id AS manager_id,
  m.emp_name AS manager_name,
  COUNT(h.employee_id) - 1 AS total_subordinates_cnt
FROM interview.employees m
JOIN employee_hierarchy h
ON m.emp_id = h.manager_id
GROUP BY m.emp_id, m.emp_name
ORDER BY total_subordinates_cnt DESC;
```
| manager_id | manager_name | total_subordinates_cnt |
|------------|--------------|-------------------------|
| 1          | Alice        | 5                       |
| 2          | Bob          | 3                       |
| 7          | Grace        | 2                       |
| 8          | Heidi        | 1                       |
| 4          | Dave         | 1                       |
| 6          | Frank        | 0                       |
| 9          | Ivan         | 0                       |
| 5          | Eve          | 0                       |
| 3          | Charlie      | 0                       |
