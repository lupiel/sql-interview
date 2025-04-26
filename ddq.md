```sql
CREATE OR REPLACE TABLE interview.employees (
  emp_id INT64,
  emp_name STRING,
  salary INT64,
  hire_date DATE,
  department_id INT64,
  manager_id INT64
);

INSERT INTO interview.employees VALUES
  (1, 'Alice', 120000, DATE '2020-01-15', 1, NULL),    -- Top manager
  (2, 'Bob', 90000, DATE '2021-03-22', 1, 1),          -- Reports to Alice
  (3, 'Charlie', 85000, DATE '2021-07-01', 1, 1),      -- Reports to Alice
  (4, 'Dave', 95000, DATE '2022-02-10', 2, 2),         -- Reports to Bob
  (5, 'Eve', 90000, DATE '2022-05-18', 2, 2),          -- Reports to Bob
  (6, 'Frank', 85000, DATE '2023-01-20', 2, 4),        -- Reports to Dave
  (7, 'Grace', 100000, DATE '2020-09-15', 3, NULL),    -- Another top manager
  (8, 'Heidi', 75000, DATE '2022-11-03', 3, 7),        -- Reports to Grace
  (9, 'Ivan', 70000, DATE '2023-06-12', 3, 8);         -- Reports to Heidi

CREATE OR REPLACE TABLE interview.sales (
  sale_id INT64,
  product_id INT64,
  sale_date DATE,
  revenue INT64
);

INSERT INTO interview.sales VALUES
  (9, 102, DATE '2024-01-15', 900),
  (10, 102, DATE '2024-02-15', 950),
  (11, 101, DATE '2024-01-20', 1700),
  (12, 101, DATE '2024-01-25', 1550),
  (13, 102, DATE '2024-01-18', 850),
  (14, 102, DATE '2024-01-28', 870),
  (15, 101, DATE '2024-02-10', 1900),
  (16, 101, DATE '2024-02-22', 1750),
  (17, 102, DATE '2024-02-14', 950),
  (18, 102, DATE '2024-02-26', 980);
```
