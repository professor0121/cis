# SQL Interview Preparation — Questions & Answers

> Reference stack: **MySQL 8+** & **PostgreSQL** (differences noted where syntax varies).

---

## 1. Basics

### 1.1 SQL aur NoSQL me kya difference hai?
- **SQL (Relational DB — e.g., MySQL, PostgreSQL):**
  - Data is stored in **tables (rows & columns)** with a **fixed/predefined schema**.
  - Strong support for **JOINs**, complex ad-hoc queries, and **ACID** transactions.
  - Typically scales **vertically** (bigger CPU/RAM), though read replicas/sharding are also used.
  - Best for structured data, financial/e-commerce systems, and strict consistency.
- **NoSQL (Non-Relational DB — e.g., MongoDB, Redis, Cassandra):**
  - Data is stored as **documents (JSON/BSON)**, **key-value pairs**, **wide-columns**, or **graphs** with a **flexible/dynamic schema**.
  - Designed for **horizontal scaling** (sharding across nodes), high write throughput, and unstructured/semi-structured data.
  - Often favors **BASE** (eventual consistency), though modern NoSQL DBs like MongoDB support multi-document ACID transactions.
  - Best for catalogs with varying attributes, real-time analytics, caching, and massive scale.

---

### 1.2 DDL, DML, DCL, TCL, DQL kya hote hain? Examples do.
- **DDL (Data Definition Language):** Defines or modifies database structure/schema (auto-committed in MySQL; transactional in PostgreSQL).
  - Commands: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`
  - Example: `CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100));`
- **DML (Data Manipulation Language):** Modifies data stored inside tables.
  - Commands: `INSERT`, `UPDATE`, `DELETE`, `MERGE`
  - Example: `UPDATE users SET name = 'Amit' WHERE id = 1;`
- **DQL (Data Query Language):** Retrieves data from tables.
  - Command: `SELECT`
  - Example: `SELECT id, name FROM users WHERE id = 1;`
- **DCL (Data Control Language):** Controls access and permissions.
  - Commands: `GRANT`, `REVOKE`
  - Example: `GRANT SELECT ON users TO analyst_role;`
- **TCL (Transaction Control Language):** Manages transactions and data consistency.
  - Commands: `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `SET TRANSACTION`
  - Example: `BEGIN; DELETE FROM orders WHERE id = 5; ROLLBACK;`

---

### 1.3 DELETE, TRUNCATE, aur DROP me difference batao.
| Feature | `DELETE` | `TRUNCATE` | `DROP` |
|---|---|---|---|
| **Type** | DML | DDL | DDL |
| **Removes** | Specific rows (with `WHERE`) or all rows | All rows only (no `WHERE`) | Entire table structure + data + indexes |
| **Rollback** | Possible (until `COMMIT`) | Cannot be rolled back in MySQL; **can** be rolled back inside a transaction in PostgreSQL | Cannot be rolled back in MySQL; transactional in PostgreSQL |
| **Speed** | Slower (logs every row deletion) | Faster (deallocates data pages) | Instant (removes metadata & data) |
| **Auto-increment / Identity** | Preserved | Resets in MySQL (`RESTART IDENTITY` optional in Postgres) | Table itself is removed |
| **Triggers** | Fires `ON DELETE` triggers | Does **not** fire row-level delete triggers | N/A |

---

### 1.4 WHERE aur HAVING me kya fark hai?
- **`WHERE`** filters **individual rows BEFORE grouping (`GROUP BY`)** and **cannot** use aggregate functions (`SUM`, `COUNT`, `AVG`).
- **`HAVING`** filters **groups AFTER `GROUP BY`** and **can** use aggregate functions.
```sql
SELECT department_id, COUNT(*) AS emp_count
FROM employees
WHERE status = 'active'          -- filters rows before grouping
GROUP BY department_id
HAVING COUNT(*) > 5;             -- filters groups after aggregation
```
*(Performance tip: Always put non-aggregate filters in `WHERE` rather than `HAVING` so fewer rows enter the grouping step.)*

---

### 1.5 CHAR vs VARCHAR vs TEXT?
- **`CHAR(n)`:** Fixed-length string. Pads shorter strings with spaces up to `n`. Faster for fixed-size codes (e.g., `CHAR(2)` for country code `'IN'`, `CHAR(3)` for currency `'INR'`).
- **`VARCHAR(n)`:** Variable-length string up to `n` characters. Stores only actual length + 1–2 bytes length prefix. Best for names, emails, URLs.
- **`TEXT`:** For large strings (articles, descriptions, JSON logs). Cannot have a full-column B-Tree index without a prefix length in MySQL (and in MySQL `TEXT` cannot have a `DEFAULT` value in older versions). Note: in PostgreSQL, `VARCHAR(n)` and `TEXT` have identical underlying storage performance; `VARCHAR(n)` just adds a length check.

---

### 1.6 UNION vs UNION ALL?
- **`UNION`:** Combines result sets of two or more `SELECT` queries and **removes duplicate rows** (requires sorting/hashing → slower).
- **`UNION ALL`:** Combines result sets and **keeps all rows including duplicates** (no deduplication overhead → much faster).
- **Rules for both:** Same number of columns, compatible data types in corresponding positions, and `ORDER BY` goes at the very end.
- **Tip:** Prefer `UNION ALL` unless you specifically need deduplication.

---

### 1.7 LIKE, IN, BETWEEN, EXISTS kab use karte hain?
- **`LIKE`:** Pattern matching on strings (`%` = 0 or more chars, `_` = exactly 1 char). Note: Postgres also has `ILIKE` for case-insensitive matching.
  - Example: `WHERE email LIKE '%@gmail.com'`
- **`IN`:** Checks if a value matches any value in a list or small subquery.
  - Example: `WHERE status IN ('pending', 'processing')`
- **`BETWEEN`:** Inclusive range check (`>= low AND <= high`) for numbers or dates. *(Caution with `DATETIME`/`TIMESTAMP`: `BETWEEN '2026-01-01' AND '2026-01-31'` misses `'2026-01-31 14:00:00'` because the end defaults to `00:00:00`! Prefer `>= '2026-01-01' AND < '2026-02-01'`.)*
- **`EXISTS`:** Checks whether a correlated subquery returns at least one row (returns boolean `TRUE`/`FALSE`, short-circuits on first match, safe with `NULL`s).
  - Example: `WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id)`

---

### 1.8 COUNT(*), COUNT(col), COUNT(DISTINCT col) me difference?
- **`COUNT(*)`:** Counts **all rows**, including rows with `NULL` values.
- **`COUNT(col)`:** Counts rows where `col IS NOT NULL` (skips `NULL`s).
- **`COUNT(DISTINCT col)`:** Counts **unique non-NULL** values in `col`.

---

### 1.9 ORDER BY ke saath LIMIT/OFFSET kaise kaam karta hai?
- Logical execution: First the DB filters/joins/groups rows, then **sorts** using `ORDER BY`, then skips `OFFSET` rows and returns at most `LIMIT` rows.
```sql
SELECT id, name, created_at
FROM users
ORDER BY created_at DESC, id DESC
LIMIT 10 OFFSET 20; -- skips first 20 rows, returns rows 21–30
```
- **Important:** Without `ORDER BY`, row order is not guaranteed. Also add a unique tie-breaker (like `id`) in `ORDER BY` so pages don't shuffle/duplicate rows across requests.

---

### 1.10 Alias kya hota hai aur kab zaroori hai?
An **alias** (`AS`) gives a temporary name to a column or table for readability or disambiguation.
- **When it is mandatory:**
  1. **Derived tables / subqueries in `FROM`:** `SELECT * FROM (SELECT ...) AS sub;` (required in MySQL & PostgreSQL).
  2. **Self-joins:** Joining a table to itself requires aliases (`employees e JOIN employees m`) so columns can be distinguished.
  3. **Ambiguous columns in JOINs:** When two tables share a column name (e.g., `u.id`, `o.id`).
  4. **Referencing calculated columns in outer queries:** e.g., `SELECT total_amt` from a CTE/subquery.

---

## 2. NULL Handling

### 2.1 NULL = NULL false kyun hota hai?
In SQL, `NULL` does **not** mean zero or empty string—it means **unknown / missing value**.
SQL uses **Three-Valued Logic (3VL)**: `TRUE`, `FALSE`, and `UNKNOWN`.
Comparing two unknown values (`NULL = NULL`) evaluates to **`UNKNOWN`** (which `WHERE` treats as falsy, so the row is excluded).
Always use **`IS NULL`** or **`IS NOT NULL`** (or `IS NOT DISTINCT FROM` in PostgreSQL / `<=>` null-safe equal in MySQL).

---

### 2.2 COALESCE, IFNULL, NULLIF ka use batao.
- **`COALESCE(a, b, c, ...)`:** ANSI standard. Returns the **first non-NULL** argument.
  ```sql
  SELECT COALESCE(phone, mobile, 'N/A') AS contact FROM users;
  ```
- **`IFNULL(a, b)` (MySQL):** Returns `a` if not NULL, else `b` (only takes 2 arguments; Postgres equivalent is `COALESCE`).
- **`NULLIF(a, b)`:** Returns `NULL` if `a = b`, otherwise returns `a`.
  - Classic use case: **Prevent division-by-zero** & treat empty strings as `NULL`:
  ```sql
  SELECT revenue / NULLIF(orders_count, 0) AS aov FROM daily_stats;
  SELECT NULLIF(TRIM(email), '') FROM users;
  ```

---

### 2.3 NOT IN me NULL aaye toh kya problem hoti hai?
If the list or subquery in `NOT IN` contains **even a single `NULL`**, the entire `NOT IN` condition evaluates to **`UNKNOWN` for every row** and returns **0 rows**!
- Why: `x NOT IN (1, 2, NULL)` means `x <> 1 AND x <> 2 AND x <> NULL`. Since `x <> NULL` is `UNKNOWN`, `TRUE AND TRUE AND UNKNOWN` = `UNKNOWN`.
- **Fix:**
  1. Use **`NOT EXISTS`** (safe with `NULL`s), OR
  2. Filter out NULLs inside the subquery: `WHERE id NOT IN (SELECT user_id FROM orders WHERE user_id IS NOT NULL)`.

---

### 2.4 Aggregate functions NULL ko kaise treat karte hain?
- `SUM`, `AVG`, `MIN`, `MAX`, `COUNT(col)` **ignore `NULL` values** completely.
  - Note: For `[10, 20, NULL]`, `AVG(col)` is `(10 + 20) / 2 = 15`, **not** `30 / 3 = 10`. Use `AVG(COALESCE(col, 0))` if you want `NULL` treated as `0`.
- `COUNT(*)` is the exception: it counts **all rows**, even if every column is `NULL`.
- If all rows in a group are `NULL`, `SUM/AVG/MIN/MAX` return `NULL`, while `COUNT(col)` returns `0`.

---

### 2.5 Aise users nikalo jinka email NULL ya empty string dono ho sakta hai.
```sql
SELECT *
FROM users
WHERE email IS NULL OR TRIM(email) = '';

-- Shortcut using NULLIF (handles NULL, '', and whitespace-only strings):
SELECT *
FROM users
WHERE NULLIF(TRIM(email), '') IS NULL;
```

---

## 3. Joins

### 3.1 INNER, LEFT, RIGHT, FULL OUTER, CROSS, SELF join explain karo.
- **`INNER JOIN`:** Returns only rows with matching keys in **both** tables.
- **`LEFT JOIN` (`LEFT OUTER JOIN`):** Returns **all rows from left table** + matching rows from right table (`NULL` if no match).
- **`RIGHT JOIN` (`RIGHT OUTER JOIN`):** Returns **all rows from right table** + matching rows from left table (`NULL` if no match).
- **`FULL OUTER JOIN`:** Returns all rows from **both** tables, matching where possible and filling `NULL` where there is no match. *(Supported natively in PostgreSQL; in MySQL, emulate with `LEFT JOIN ... UNION ALL RIGHT JOIN ... WHERE left.id IS NULL`.)*
- **`CROSS JOIN`:** Cartesian product — every row from table A paired with every row from table B (`M × N` rows).
- **`SELF JOIN`:** Joining a table with itself using two aliases (e.g., employee–manager, finding pairs).

---

### 3.2 Aise users nikalo jinhone kabhi order nahi kiya.
```sql
-- Method 1: LEFT JOIN + IS NULL
SELECT u.id, u.name
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;

-- Method 2: NOT EXISTS (often very fast and NULL-safe)
SELECT u.id, u.name
FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);
```

---

### 3.3 Aise products nikalo jo kabhi sell nahi hue.
```sql
SELECT p.id, p.name
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
WHERE oi.id IS NULL;
```

---

### 3.4 Har employee ke saath uske manager ka naam dikhao.
```sql
SELECT e.id,
       e.name AS employee_name,
       m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
-- LEFT JOIN ensures top-level employees (like CEO with manager_id IS NULL) are still shown.
```

---

### 3.5 Un departments ki list do jisme koi employee nahi hai.
```sql
SELECT d.id, d.name
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
WHERE e.id IS NULL;
```

---

### 3.6 JOIN vs WHERE me condition rakhne ka LEFT JOIN me kya effect padta hai?
- **Condition in `ON` clause:** Filters the **right table BEFORE joining**. All left-table rows are still preserved (with `NULL`s on the right if the condition fails).
- **Condition on right-table column in `WHERE` clause:** Filters **AFTER the join**, which drops the `NULL`-padded rows and silently converts the `LEFT JOIN` into an **`INNER JOIN`**!
```sql
-- Keeps all users, showing only their 2026 orders (NULL if none in 2026):
SELECT u.id, o.id
FROM users u
LEFT JOIN orders o ON u.id = o.user_id AND o.order_year = 2026;

-- Drops users who don't have a 2026 order (acts like INNER JOIN!):
SELECT u.id, o.id
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.order_year = 2026;
```

---

### 3.7 Natural join kyun avoid karna chahiye?
`NATURAL JOIN` automatically joins tables on **all columns with the same name**.
- **Risks:**
  1. Hidden bugs if both tables have common audit columns like `created_at`, `updated_at`, or `status` — it will join on those too!
  2. **Schema fragility:** Adding a new column with a matching name later silently breaks existing queries.
  3. Poor readability — someone reading the query cannot tell the join condition without checking both schemas.
Always use explicit `JOIN ... ON ...`.

---

### 3.8 3 se zyada tables join karte waqt performance kaise sochoge?
1. **Filter early:** Ensure `WHERE` conditions reduce rows as early as possible (and check `EXPLAIN` to verify join order / driving table).
2. **Index foreign keys & join columns:** Every `ON a.col = b.col` column should have an index on at least the probed side.
3. **Avoid many-to-many join explosion:** Joining two 1-to-many tables (`orders → order_items` AND `orders → payments`) multiplies rows (`M × N`). Pre-aggregate in CTEs/subqueries before joining!
4. **Select only needed columns:** Avoid `SELECT *`, which bloats memory/sort buffers and prevents covering index scans.
5. **Check data types:** Mismatched column types or collations on join keys prevent index usage.

---

## 4. Aggregation & Grouping

### 4.1 Har city ke total users aur total revenue nikalo.
*(Careful: joining `users` and `orders` directly works if we use `COUNT(DISTINCT u.id)` because one user can have multiple orders.)*
```sql
SELECT u.city,
       COUNT(DISTINCT u.id) AS total_users,
       COALESCE(SUM(o.total_amount), 0) AS total_revenue
FROM users u
LEFT JOIN orders o ON u.id = o.user_id AND o.status = 'delivered'
GROUP BY u.city;
```

---

### 4.2 Un categories ko dikhao jinka average price 500 se zyada hai.
```sql
SELECT category_id, AVG(price) AS avg_price
FROM products
GROUP BY category_id
HAVING AVG(price) > 500;
```

---

### 4.3 Month-wise total sales nikalo.
```sql
-- PostgreSQL:
SELECT DATE_TRUNC('month', order_date)::DATE AS sales_month,
       SUM(total_amount) AS total_sales
FROM orders
WHERE status = 'delivered'
GROUP BY 1
ORDER BY 1;

-- MySQL:
SELECT DATE_FORMAT(order_date, '%Y-%m-01') AS sales_month,
       SUM(total_amount) AS total_sales
FROM orders
WHERE status = 'delivered'
GROUP BY 1
ORDER BY 1;
```

---

### 4.4 Top 5 customers by total spending.
```sql
SELECT u.id, u.name, SUM(o.total_amount) AS total_spent
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'delivered'
GROUP BY u.id, u.name
ORDER BY total_spent DESC
LIMIT 5;
```

---

### 4.5 Un users ko nikalo jinhone 3 se zyada orders diye.
```sql
SELECT user_id, COUNT(*) AS order_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 3;
```

---

### 4.6 GROUP BY me non-aggregated column select karne se kya hota hai?
- In **PostgreSQL** and **MySQL 8+ (with `ONLY_FULL_GROUP_BY` enabled, which is default)**: It throws a **syntax/grouping error** (`must appear in the GROUP BY clause or be used in an aggregate function`), because the DB doesn't know which row's value to pick for that column.
- *(Exception: If the column is functionally dependent on the `GROUP BY` primary key, e.g., grouping by `users.id` and selecting `users.name`, both PostgreSQL and MySQL allow it.)*
- If `ONLY_FULL_GROUP_BY` is disabled in older MySQL, it picks an **arbitrary/indeterminate** value, leading to silent bugs.

---

### 4.7 ROLLUP, CUBE, GROUPING SETS kya hain?
They are `GROUP BY` extensions for generating multi-level subtotals in a single query:
- **`ROLLUP(a, b)`:** Hierarchical subtotals from right to left: `(a, b)`, `(a)`, and `()` (grand total).
- **`CUBE(a, b)`:** All possible combinations: `(a, b)`, `(a)`, `(b)`, and `()`. *(Supported natively in PostgreSQL; not in MySQL.)*
- **`GROUPING SETS((a, b), (a), ())`:** Explicitly specify exact grouping combinations you want. *(Supported in PostgreSQL; not in MySQL.)*
```sql
-- PostgreSQL / MySQL (MySQL syntax: GROUP BY region, category WITH ROLLUP)
SELECT region, category, SUM(amount) AS total_sales
FROM sales
GROUP BY ROLLUP(region, category);
```
Use the `GROUPING(col)` function (returns `1` for subtotal `NULL`, `0` for real `NULL`) to label subtotal rows cleanly.

---

### 4.8 Conditional aggregation: ek query me delivered, cancelled, pending orders ka count alag columns me.
```sql
SELECT
    COUNT(CASE WHEN status = 'delivered' THEN 1 END) AS delivered_count,
    COUNT(CASE WHEN status = 'cancelled' THEN 1 END) AS cancelled_count,
    COUNT(CASE WHEN status = 'pending'   THEN 1 END) AS pending_count
FROM orders;

-- PostgreSQL cleaner syntax using FILTER:
SELECT
    COUNT(*) FILTER (WHERE status = 'delivered') AS delivered_count,
    COUNT(*) FILTER (WHERE status = 'cancelled') AS cancelled_count,
    COUNT(*) FILTER (WHERE status = 'pending')   AS pending_count
FROM orders;
```

---

## 5. Subqueries

### 5.1 Correlated vs non-correlated subquery?
- **Non-correlated subquery:** Independent of the outer query. Executes **once**, and its result is passed to the outer query.
  ```sql
  SELECT * FROM products WHERE price > (SELECT AVG(price) FROM products);
  ```
- **Correlated subquery:** References a column from the outer query. Conceptually runs **once per outer row** (though optimizers may rewrite it to a join/hash).
  ```sql
  SELECT e.name, e.salary
  FROM employees e
  WHERE e.salary > (SELECT AVG(e2.salary) FROM employees e2 WHERE e2.dept_id = e.dept_id);
  ```

---

### 5.2 Second highest salary kitne tareekon se nikal sakte ho?
1. **`MAX` with `< MAX`:**
   ```sql
   SELECT MAX(salary) AS second_highest
   FROM employees
   WHERE salary < (SELECT MAX(salary) FROM employees);
   ```
2. **`DENSE_RANK()` (handles ties cleanly and returns NULL if <2 distinct salaries when wrapped):**
   ```sql
   SELECT MAX(salary) AS second_highest
   FROM (
       SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
       FROM employees
   ) t
   WHERE rnk = 2;
   ```
3. **`DISTINCT` + `ORDER BY` + `LIMIT/OFFSET`:**
   ```sql
   SELECT DISTINCT salary AS second_highest
   FROM employees
   ORDER BY salary DESC
   LIMIT 1 OFFSET 1;
   ```

---

### 5.3 Nth highest salary nikalo.
```sql
-- Best & standard approach (handles duplicates properly):
WITH ranked AS (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
)
SELECT DISTINCT salary
FROM ranked
WHERE rnk = :N;

-- Alternative using correlated subquery (classic interview question):
SELECT DISTINCT e1.salary
FROM employees e1
WHERE :N - 1 = (
    SELECT COUNT(DISTINCT e2.salary)
    FROM employees e2
    WHERE e2.salary > e1.salary
);
```

---

### 5.4 Un employees ko nikalo jinki salary apne department ki average se zyada hai.
```sql
-- Using Window Function (single table scan, fast):
SELECT id, name, dept_id, salary
FROM (
    SELECT id, name, dept_id, salary,
           AVG(salary) OVER (PARTITION BY dept_id) AS dept_avg
    FROM employees
) t
WHERE salary > dept_avg;
```

---

### 5.5 IN vs EXISTS me performance difference?
- **`IN`:** Evaluates the inner subquery first, builds a list/hash, then checks outer rows. Good when the **inner subquery result is small** and the outer table is large. Fails with `NULL`s if using `NOT IN`.
- **`EXISTS`:** Evaluates outer row and checks if **any matching row exists** in inner table, **short-circuiting on the first match**. Usually better when the **outer table is small and the inner table is large & indexed** on the join column.
- *(Note: Modern MySQL 8+ and PostgreSQL optimizers often rewrite `IN` and `EXISTS` into the exact same semi-join execution plan, EXCEPT `NOT IN` vs `NOT EXISTS` because of `NULL` semantics—always prefer `NOT EXISTS`!)*

---

### 5.6 Subquery vs JOIN kab kya choose karoge?
- **Use `JOIN`** when you need **columns from both tables** or when combining datasets cleanly (generally easier for query optimizers to plan).
- **Use `EXISTS` / Subquery** when you only want to **filter rows based on existence** in another table without risking **row multiplication** from a 1-to-many relationship.
- **Use CTEs** instead of deeply nested subqueries for readability and maintainability.

---

### 5.7 Un products ko nikalo jinka price overall average se kam hai.
```sql
SELECT id, name, price
FROM products
WHERE price < (SELECT AVG(price) FROM products);
```

---

## 6. Window Functions

### 6.1 ROW_NUMBER, RANK, DENSE_RANK me difference?
For values `[100, 100, 90, 80]` ordered `DESC`:
- **`ROW_NUMBER()`:** Always assigns unique sequential integers → `1, 2, 3, 4` (arbitrary tie-breaking unless deterministic tie-breaker is added in `ORDER BY`).
- **`RANK()`:** Ties get the same rank, **leaves gaps** → `1, 1, 3, 4`.
- **`DENSE_RANK()`:** Ties get the same rank, **no gaps** → `1, 1, 2, 3`.

---

### 6.2 Har department me top 3 salary wale employees.
```sql
WITH ranked AS (
    SELECT id, name, dept_id, salary,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM employees
)
SELECT id, name, dept_id, salary
FROM ranked
WHERE rnk <= 3;
```
*(Use `ROW_NUMBER()` if you want strictly 3 employees per dept; `DENSE_RANK()` if ties should all be included.)*

---

### 6.3 Running total of daily sales.
```sql
WITH daily AS (
    SELECT DATE(order_date) AS sale_date, SUM(total_amount) AS daily_sales
    FROM orders
    GROUP BY DATE(order_date)
)
SELECT sale_date,
       daily_sales,
       SUM(daily_sales) OVER (
           ORDER BY sale_date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM daily
ORDER BY sale_date;
```

---

### 6.4 Month-over-month growth % nikalo (LAG).
```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', order_date)::DATE AS mth, -- MySQL: DATE_FORMAT(order_date, '%Y-%m-01')
           SUM(total_amount) AS revenue
    FROM orders
    GROUP BY 1
)
SELECT mth,
       revenue,
       LAG(revenue) OVER (ORDER BY mth) AS prev_revenue,
       ROUND(
           (revenue - LAG(revenue) OVER (ORDER BY mth)) * 100.0
           / NULLIF(LAG(revenue) OVER (ORDER BY mth), 0),
           2
       ) AS mom_growth_pct
FROM monthly
ORDER BY mth;
```

---

### 6.5 Har user ka pehla aur aakhri order dikhao.
```sql
SELECT DISTINCT
       user_id,
       FIRST_VALUE(id) OVER w AS first_order_id,
       FIRST_VALUE(created_at) OVER w AS first_order_date,
       LAST_VALUE(id) OVER w AS last_order_id,
       LAST_VALUE(created_at) OVER w AS last_order_date
FROM orders
WINDOW w AS (
    PARTITION BY user_id
    ORDER BY created_at, id
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
);
```
*(Or simply aggregate `MIN(created_at)` / `MAX(created_at)` or `ROW_NUMBER()` `ASC` & `DESC`.)*

---

### 6.6 Moving average (last 7 days).
```sql
WITH daily AS (
    SELECT DATE(order_date) AS sale_date, SUM(total_amount) AS daily_sales
    FROM orders
    GROUP BY DATE(order_date)
)
-- If no missing days (last 7 rows):
SELECT sale_date,
       AVG(daily_sales) OVER (
           ORDER BY sale_date
           ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
       ) AS moving_avg_7d
FROM daily;

-- If days can be missing (true calendar 7-day window in PostgreSQL / MySQL 8+):
SELECT sale_date,
       AVG(daily_sales) OVER (
           ORDER BY sale_date::TIMESTAMP
           RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW
       ) AS moving_avg_7d
FROM daily;
```

---

### 6.7 PARTITION BY vs GROUP BY?
- **`GROUP BY`:** Collapses multiple rows into **one summary row per group**. You lose individual row details.
- **`PARTITION BY` (with Window Functions):** Divides rows into logical partitions to compute aggregates/ranks **without collapsing rows** — every original row is preserved alongside the partition result.

---

### 6.8 NTILE se customers ko 4 buckets me divide karo.
```sql
WITH customer_spend AS (
    SELECT user_id, SUM(total_amount) AS total_spent
    FROM orders
    GROUP BY user_id
)
SELECT user_id,
       total_spent,
       NTILE(4) OVER (ORDER BY total_spent DESC) AS spend_quartile
       -- 1 = top 25% spenders, 4 = bottom 25%
FROM customer_spend;
```

---

### 6.9 FIRST_VALUE, LAST_VALUE ka frame clause issue kya hai?
When you write `OVER (PARTITION BY ... ORDER BY ...)`, the **default window frame** in SQL is:
`RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.
- Because the frame ends at `CURRENT ROW`, **`LAST_VALUE()` returns the current row's value** instead of the true last value of the partition!
- **Fix:** Explicitly define the frame up to the end of the partition:
  ```sql
  LAST_VALUE(amount) OVER (
      PARTITION BY user_id
      ORDER BY created_at
      ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
  )
  ```

---

## 7. CTE & Recursive Queries

### 7.1 CTE vs subquery vs temp table?
- **Subquery:** Inline query inside `SELECT`/`FROM`/`WHERE`. Hard to read when nested deeply; cannot be referenced multiple times in the same query.
- **CTE (`WITH ...`):** Named temporary result set scoped to **a single SQL statement**. Much cleaner, supports recursion, and can be referenced multiple times. Usually inlined by the optimizer (Postgres 12+ inlines unless `MATERIALIZED` is specified).
- **Temp Table (`CREATE TEMP TABLE`):** Physically materialized in temp storage for the **duration of the session/transaction**. You can **create indexes** on it and reuse it across **multiple statements** — great for complex multi-step ETL or huge intermediate datasets.

---

### 7.2 Recursive CTE se employee hierarchy (manager → reportees) nikalo.
```sql
WITH RECURSIVE org_tree AS (
    -- Anchor member: top-level manager(s)
    SELECT id, name, manager_id, 1 AS level, CAST(name AS VARCHAR(500)) AS path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive member: join direct reportees
    SELECT e.id, e.name, e.manager_id, ot.level + 1,
           CONCAT(ot.path, ' -> ', e.name)
    FROM employees e
    JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT id, name, manager_id, level, path
FROM org_tree
ORDER BY level, path;
```

---

### 7.3 1 se 100 tak numbers generate karo bina table ke.
```sql
WITH RECURSIVE nums AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM nums WHERE n < 100
)
SELECT n FROM nums;

-- PostgreSQL shortcut:
SELECT generate_series(1, 100) AS n;
```
*(Note: In MySQL, `cte_max_recursion_depth` defaults to 1000.)*

---

### 7.4 Category tree (parent-child) flatten karo.
```sql
WITH RECURSIVE cat_tree AS (
    SELECT id, name, parent_id,
           id AS root_id,
           name AS root_name,
           CAST(name AS VARCHAR(1000)) AS full_path,
           0 AS depth
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    SELECT c.id, c.name, c.parent_id,
           ct.root_id,
           ct.root_name,
           CONCAT(ct.full_path, ' > ', c.name),
           ct.depth + 1
    FROM categories c
    JOIN cat_tree ct ON c.parent_id = ct.id
)
SELECT id, name, root_name, depth, full_path
FROM cat_tree;
```

---

### 7.5 Multiple CTEs chain karke monthly cohort report banao.
```sql
WITH user_cohort AS (
    SELECT user_id,
           DATE_TRUNC('month', MIN(order_date))::DATE AS cohort_month
    FROM orders
    GROUP BY user_id
),
user_activities AS (
    SELECT o.user_id,
           uc.cohort_month,
           DATE_TRUNC('month', o.order_date)::DATE AS order_month,
           (EXTRACT(YEAR FROM o.order_date) - EXTRACT(YEAR FROM uc.cohort_month)) * 12
           + (EXTRACT(MONTH FROM o.order_date) - EXTRACT(MONTH FROM uc.cohort_month)) AS month_number
    FROM orders o
    JOIN user_cohort uc ON o.user_id = uc.user_id
),
cohort_size AS (
    SELECT cohort_month, COUNT(*) AS total_users
    FROM user_cohort
    GROUP BY cohort_month
)
SELECT ua.cohort_month,
       ua.month_number,
       COUNT(DISTINCT ua.user_id) AS active_users,
       cs.total_users,
       ROUND(COUNT(DISTINCT ua.user_id) * 100.0 / cs.total_users, 2) AS retention_pct
FROM user_activities ua
JOIN cohort_size cs ON ua.cohort_month = cs.cohort_month
GROUP BY ua.cohort_month, ua.month_number, cs.total_users
ORDER BY ua.cohort_month, ua.month_number;
```

---

## 8. Date & String Functions

### 8.1 Pichle 30 din ke orders nikalo.
```sql
-- PostgreSQL:
SELECT * FROM orders
WHERE order_date >= CURRENT_DATE - INTERVAL '30 days';

-- MySQL:
SELECT * FROM orders
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY);
```
*(Prefer comparing `order_date >= ...` rather than `DATEDIFF(NOW(), order_date) <= 30` so indexes on `order_date` remain usable.)*

---

### 8.2 Har month ka pehla aur aakhri din nikalo.
```sql
-- PostgreSQL:
SELECT DATE_TRUNC('month', order_date)::DATE AS month_start,
       (DATE_TRUNC('month', order_date) + INTERVAL '1 month - 1 day')::DATE AS month_end
FROM orders;

-- MySQL:
SELECT DATE_FORMAT(order_date, '%Y-%m-01') AS month_start,
       LAST_DAY(order_date) AS month_end
FROM orders;
```

---

### 8.3 Age calculate karo DOB se.
```sql
-- PostgreSQL:
SELECT name, dob, EXTRACT(YEAR FROM AGE(CURRENT_DATE, dob))::INT AS age
FROM users;

-- MySQL:
SELECT name, dob, TIMESTAMPDIFF(YEAR, dob, CURDATE()) AS age
FROM users;
```

---

### 8.4 Un users ko nikalo jinka signup weekend pe hua.
```sql
-- PostgreSQL (DOW: 0 = Sunday, 6 = Saturday):
SELECT * FROM users
WHERE EXTRACT(DOW FROM created_at) IN (0, 6);

-- MySQL (DAYOFWEEK: 1 = Sunday, 7 = Saturday):
SELECT * FROM users
WHERE DAYOFWEEK(created_at) IN (1, 7);
```

---

### 8.5 Email se domain extract karo.
```sql
-- PostgreSQL:
SELECT email, SPLIT_PART(email, '@', 2) AS domain FROM users;

-- MySQL:
SELECT email, SUBSTRING_INDEX(email, '@', -1) AS domain FROM users;
```

---

### 8.6 Name ka first letter capital karo.
```sql
-- PostgreSQL (INITCAP capitalizes first letter of each word):
SELECT INITCAP(name) FROM users;

-- Capitalize ONLY the first letter of the string (MySQL & PostgreSQL):
SELECT CONCAT(UPPER(LEFT(name, 1)), LOWER(SUBSTRING(name, 2))) AS formatted_name
FROM users;
```

---

### 8.7 Comma-separated string ko rows me todo.
```sql
-- PostgreSQL:
SELECT id, TRIM(tag) AS tag
FROM posts,
     LATERAL UNNEST(STRING_TO_ARRAY(tags, ',')) AS tag;

-- MySQL 8+ (using JSON_TABLE trick):
SELECT p.id, jt.tag
FROM posts p
JOIN JSON_TABLE(
    CONCAT('["', REPLACE(p.tags, ',', '","'), '"]'),
    '$[*]' COLUMNS (tag VARCHAR(100) PATH '$')
) jt;
```

---

### 8.8 Duplicate spaces hatao string se.
```sql
-- PostgreSQL & MySQL 8+:
SELECT REGEXP_REPLACE(TRIM(full_name), ' +', ' ', 'g') FROM users; -- Postgres ('g' = global)
SELECT REGEXP_REPLACE(TRIM(full_name), ' +', ' ')      FROM users; -- MySQL 8+ (replaces all by default)
```

---

## 9. Duplicates & Data Cleaning

### 9.1 Duplicate emails dhundo.
```sql
SELECT email, COUNT(*) AS cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

### 9.2 Duplicate rows delete karo, par ek rakho.
```sql
-- PostgreSQL (using ctid or id):
DELETE FROM users
WHERE id IN (
    SELECT id FROM (
        SELECT id,
               ROW_NUMBER() OVER (PARTITION BY email ORDER BY id ASC) AS rn
        FROM users
    ) t
    WHERE rn > 1
);

-- MySQL (self-join DELETE keeps lowest id):
DELETE u1
FROM users u1
JOIN users u2
  ON u1.email = u2.email AND u1.id > u2.id;
```
*(In MySQL/Postgres, after cleaning, add a `UNIQUE` constraint/index on `email` so duplicates can't enter again.)*

---

### 9.3 Table me kisi bhi column ka duplicate count nikalo.
```sql
-- Total duplicate rows beyond unique values:
SELECT COUNT(*) - COUNT(DISTINCT email) AS extra_duplicate_rows
FROM users;

-- Per-value duplicate count:
SELECT col, COUNT(*) AS occurrences
FROM my_table
GROUP BY col
HAVING COUNT(*) > 1;
```

---

### 9.4 Consecutive duplicate values detect karo.
```sql
-- Example: find rows where status is identical to the immediately preceding row (ordered by id/time):
WITH flagged AS (
    SELECT id, status,
           LAG(status) OVER (ORDER BY id) AS prev_status
    FROM status_logs
)
SELECT id, status
FROM flagged
WHERE status = prev_status;
```

---

### 9.5 Missing IDs (gaps in sequence) dhundo.
```sql
-- Method 1: Find gap start and gap end using LEAD():
WITH gaps AS (
    SELECT id AS current_id,
           LEAD(id) OVER (ORDER BY id) AS next_id
    FROM orders
)
SELECT current_id + 1 AS gap_start,
       next_id - 1    AS gap_end
FROM gaps
WHERE next_id - current_id > 1;

-- Method 2: List every exact missing ID (PostgreSQL):
SELECT s.id AS missing_id
FROM generate_series(
    (SELECT MIN(id) FROM orders),
    (SELECT MAX(id) FROM orders)
) AS s(id)
LEFT JOIN orders o ON s.id = o.id
WHERE o.id IS NULL;
```

---

## 10. DML & DDL

### 10.1 INSERT ... SELECT kab use karte ho?
Used to **copy or transform rows** from one table (or query) directly into another table in a single statement—useful for archiving old data, populating summary/reporting tables, or data migration.
```sql
INSERT INTO orders_archive (id, user_id, total_amount, created_at)
SELECT id, user_id, total_amount, created_at
FROM orders
WHERE created_at < '2025-01-01';
```

---

### 10.2 UPDATE with JOIN kaise karte hain?
Syntax differs between MySQL and PostgreSQL:
```sql
-- MySQL:
UPDATE orders o
JOIN users u ON o.user_id = u.id
SET o.discount = 10
WHERE u.tier = 'gold';

-- PostgreSQL (uses UPDATE ... FROM):
UPDATE orders o
SET discount = 10
FROM users u
WHERE o.user_id = u.id
  AND u.tier = 'gold';
```

---

### 10.3 UPSERT (ON DUPLICATE KEY / ON CONFLICT) explain karo.
**UPSERT** = **UP**date if row exists (based on Primary Key / Unique constraint), else in**SERT** atomically.
```sql
-- MySQL:
INSERT INTO inventory (product_id, qty)
VALUES (101, 5)
ON DUPLICATE KEY UPDATE qty = qty + VALUES(qty); -- MySQL 8.0.20+: alias syntax `AS new`

-- PostgreSQL:
INSERT INTO inventory (product_id, qty)
VALUES (101, 5)
ON CONFLICT (product_id)
DO UPDATE SET qty = inventory.qty + EXCLUDED.qty;
```
*(Postgres also supports `ON CONFLICT DO NOTHING` to silently skip duplicates.)*

---

### 10.4 ALTER TABLE se column add, rename, drop, type change.
```sql
-- Add column:
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Rename column:
ALTER TABLE users RENAME COLUMN phone TO mobile;

-- Drop column:
ALTER TABLE users DROP COLUMN mobile;

-- Change column type:
-- PostgreSQL:
ALTER TABLE users ALTER COLUMN age TYPE BIGINT;
-- MySQL:
ALTER TABLE users MODIFY COLUMN age BIGINT;
```

---

### 10.5 MERGE statement kya hai?
`MERGE` is an ANSI SQL statement (supported in **PostgreSQL 15+**, SQL Server, Oracle; not native in MySQL) that synchronizes a target table with a source table in one atomic query—handling **`WHEN MATCHED`** (`UPDATE`/`DELETE`) and **`WHEN NOT MATCHED`** (`INSERT`) together.
```sql
-- PostgreSQL 15+:
MERGE INTO target_products t
USING staging_products s ON t.id = s.id
WHEN MATCHED AND s.is_deleted THEN DELETE
WHEN MATCHED THEN UPDATE SET price = s.price, name = s.name
WHEN NOT MATCHED THEN INSERT (id, name, price) VALUES (s.id, s.name, s.price);
```

---

### 10.6 Bulk insert ke best practices?
1. **Batch inserts:** Insert 1,000–5,000 rows per multi-row `INSERT` or transaction instead of 1 row per network round-trip.
2. **Use native bulk loaders:** `COPY` / `\copy` in PostgreSQL, `LOAD DATA INFILE` in MySQL (10–50x faster than `INSERT`).
3. **Wrap in a single transaction** (or large chunks) to avoid flushing WAL/redo logs on every single row.
4. **Temporarily drop/defer non-unique secondary indexes & triggers** before massive initial loads, then rebuild indexes afterward.
5. **Use prepared statements** if inserting via application code.

---

### 10.7 Large table me ADD COLUMN production me safely kaise karoge?
- **Adding a nullable column or a column with a non-volatile `DEFAULT`:**
  - **PostgreSQL 11+:** Instant metadata-only operation (`O(1)`), does **not** rewrite the table. Still acquires a brief `ACCESS EXCLUSIVE` lock, so set a **`lock_timeout`** (e.g., `SET lock_timeout = '2s';`) so it doesn't queue behind a long-running query and block all traffic!
  - **MySQL 8+:** `ALGORITHM=INSTANT` adds the column instantly without table copy (for trailing columns without full rebuild).
- **Adding `NOT NULL` with backfill / `CHECK` constraint safely:**
  1. Add column as nullable first.
  2. Backfill in small batches (`LIMIT 10000` with brief sleep) to avoid replication lag and lock contention.
  3. Add `CHECK (col IS NOT NULL) NOT VALID` (Postgres), then `VALIDATE CONSTRAINT` (takes weak lock, doesn't block reads/writes).
  4. For MySQL online DDL on heavy schema changes, use **`gh-ost`** or **`pt-online-schema-change`**.

---

## 11. Keys, Constraints, Normalization

### 11.1 Primary, Foreign, Unique, Composite, Candidate, Super key?
- **Super Key:** Any set of 1+ columns that uniquely identifies a row (e.g., `{id}`, `{id, email}`).
- **Candidate Key:** Minimal super key (no redundant columns) — e.g., `{id}` or `{email}`.
- **Primary Key (PK):** The chosen candidate key for the table; **unique + `NOT NULL`**, one per table.
- **Unique Key:** Enforces uniqueness; **allows `NULL`** (multiple `NULL`s allowed in SQL standard/Postgres/MySQL unless `NULLS NOT DISTINCT` in Postgres 15+).
- **Composite Key:** A key made of **2 or more columns** together (e.g., `(student_id, course_id)`).
- **Foreign Key (FK):** Column(s) referencing a Primary/Unique key in another (or same) table to enforce **referential integrity**.

---

### 11.2 ON DELETE CASCADE, SET NULL, RESTRICT?
Controls what happens to child rows when a parent row is deleted:
- **`CASCADE`:** Automatically deletes matching child rows (e.g., deleting an `order` deletes its `order_items`).
- **`SET NULL`:** Sets child's FK column to `NULL` (column must be nullable; e.g., deleting a `manager` sets `employees.manager_id = NULL`).
- **`RESTRICT` / `NO ACTION`:** Blocks deleting the parent row if child rows still reference it (default & safest for financial/audit data).

---

### 11.3 CHECK, DEFAULT, NOT NULL constraints?
- **`NOT NULL`:** Prevents `NULL` values in a column.
- **`DEFAULT`:** Assigns a fallback value when omitted on `INSERT` (e.g., `status VARCHAR(20) DEFAULT 'pending'`).
- **`CHECK`:** Enforces a custom boolean condition on row values (e.g., `CHECK (price >= 0 AND stock >= 0)`). *(Enforced in PostgreSQL and MySQL 8.0.16+; ignored in MySQL < 8.0.16.)*

---

### 11.4 1NF, 2NF, 3NF, BCNF with example.
- **1NF (First Normal Form):** Atomic values only (no comma-separated lists/arrays in a cell), no repeating column groups, each row uniquely identifiable.
  - *Violation:* `orders(id, items)` where `items = 'Pen, Book'`. *Fix:* Move items into `order_items`.
- **2NF (Second Normal Form):** Must be in 1NF + **no partial dependency** (non-key column must depend on the *entire* composite primary key, not just part of it).
  - *Violation:* `enrollments(student_id, course_id, course_name)`. `course_name` depends only on `course_id`. *Fix:* Move `course_name` to `courses`.
- **3NF (Third Normal Form):** Must be in 2NF + **no transitive dependency** (non-key column should not depend on another non-key column).
  - *Violation:* `employees(emp_id, dept_id, dept_name)`. `dept_name` depends on `dept_id`, not `emp_id`. *Fix:* Move `dept_name` to `departments`.
- **BCNF (Boyce-Codd Normal Form):** Stricter 3NF — for every functional dependency `X → Y`, `X` must be a **super key** (handles edge cases where overlapping composite candidate keys exist).

---

### 11.5 Denormalization kab sahi hai?
When **read performance matters more than write simplicity/storage**, and joins are causing bottlenecks:
- **Good use cases:**
  - Storing pre-computed aggregates (`post.like_count`, `user.order_count`) instead of running `COUNT(*)` on millions of rows per page load.
  - Analytics/reporting tables & data warehouses (star schema).
  - Snapshotting historical data (storing `unit_price` inside `order_items` even though `products.price` exists).
- **Trade-off:** Faster reads, fewer joins, but requires careful sync on writes (triggers/transactions) to prevent stale data.

---

### 11.6 Surrogate key vs natural key?
- **Natural Key:** Derived from real-world business data (e.g., `email`, `SSN`, `country_code`).
  - *Pros:* Meaningful, saves a column/index.
  - *Cons:* Can change (user changes email → expensive cascading FK updates), often larger strings, privacy issues in URLs/logs.
- **Surrogate Key:** Artificial system-generated ID (e.g., `BIGSERIAL`, `AUTO_INCREMENT`, `UUID`).
  - *Pros:* Immutable, compact, decoupled from business changes.
  - *Cons:* No business meaning; still need a separate `UNIQUE` constraint on natural attributes (like `email`).
- **Rule of thumb:** Use **surrogate keys** for almost all application tables (`id`), and natural keys mainly for small static lookup tables (e.g., `currencies(code)`).

---

### 11.7 UUID vs auto-increment ID: pros/cons?
| Feature | Auto-Increment (`BIGINT`) | UUID (`UUIDv4` / `UUIDv7`) |
|---|---|---|
| **Size** | 8 bytes (`BIGINT`) | 16 bytes (2x larger indexes) |
| **Index locality / Insert speed** | Sequential → high B-Tree cache locality, very fast inserts | **UUIDv4 is random** → B-Tree page splits, fragmentation, slower inserts. *(UUIDv7 is time-ordered and solves this!)* |
| **Predictability / Security** | Sequential → easy to guess/scrape (`/users/101`, `/102`) | Unpredictable (v4) → hides record counts & prevents enumeration |
| **Distributed generation** | Needs central DB sequence (harder across shards) | Can be generated safely on client/microservices offline |

---

## 12. Indexes & Performance

### 12.1 Index kya hai aur kaise kaam karta hai (B-Tree)?
An **index** is a separate data structure (default **B+Tree**) that keeps column values sorted with pointers to table rows, turning an `O(N)` full table scan into an **`O(log N)`** tree lookup.
- A **B+Tree** has a root node, internal branch nodes, and linked **leaf nodes** containing sorted keys + row pointers (so range queries `BETWEEN`, `>`, `ORDER BY` can scan leaf nodes sequentially without going back up the tree).

---

### 12.2 Clustered vs non-clustered index?
- **Clustered Index:** The **table rows themselves** are physically sorted and stored in the B-Tree leaf nodes in the order of the index key. A table can have **only ONE** clustered index.
  - **MySQL (InnoDB):** The Primary Key **is** the clustered index (secondary indexes store the PK value as the pointer).
  - **PostgreSQL:** Heap tables have no automatic clustered index (`CLUSTER` is a one-time physical reorder, not maintained on new writes).
- **Non-Clustered (Secondary) Index:** Separate B-Tree storing indexed column(s) + a pointer (PK in InnoDB, physical `ctid` in Postgres) to look up the actual row. A table can have many non-clustered indexes.

---

### 12.3 Composite index me column order kyun matter karta hai?
B-Tree composite indexes follow the **Leftmost Prefix Rule**.
An index on `(a, b, c)` is sorted by `a`, then `b`, then `c`. It can speed up queries on:
- `(a)`
- `(a, b)`
- `(a, b, c)`
It **cannot** efficiently use the index for `(b)` or `(b, c)` alone (without scanning the whole index).
- **Golden ordering rule (ESR — Equality, Sort, Range):**
  Put **Equality (`=`)** columns first, then **Sort (`ORDER BY`)**, then **Range (`>`, `<`, `BETWEEN`)** columns last!

---

### 12.4 Covering index kya hota hai?
A **covering index** contains **all columns needed by a query** (in `SELECT`, `WHERE`, `JOIN`, `ORDER BY`), so the DB answers the entire query directly from the index (**Index-Only Scan**) without ever reading the main table heap/pages!
- **PostgreSQL:** Supports `INCLUDE` for non-search payload columns:
  ```sql
  CREATE INDEX idx_orders_user_cov ON orders(user_id) INCLUDE (status, total_amount);
  ```

---

### 12.5 Index hone ke baad bhi query slow kyun ho sakti hai?
1. **Low selectivity:** E.g., `WHERE is_active = true` matches 80% of rows → optimizer chooses Sequential/Full Table Scan because random I/O index lookups cost more than a sequential scan.
2. **Non-sargable query:** Applying functions/math on the indexed column (`WHERE LOWER(email) = ...`, `WHERE id + 1 = 5`).
3. **Leading wildcard:** `LIKE '%abc'`.
4. **Implicit type conversion:** Comparing string column with integer (`WHERE phone = 9876543210`).
5. **OR across different columns** without composite/bitmap index.
6. **Outdated statistics / index bloat:** Run `ANALYZE` (Postgres) or `ANALYZE TABLE` (MySQL).
7. **Fetching too many columns (`SELECT *`):** Extra heap lookups instead of index-only scan.

---

### 12.6 EXPLAIN / EXPLAIN ANALYZE kaise padhte ho?
- **`EXPLAIN`:** Shows the **estimated** execution plan (does not run the query).
- **`EXPLAIN ANALYZE`:** Actually **executes** the query and shows **actual** row counts, loops, and execution time! *(Be careful running on `UPDATE`/`DELETE`—wrap in `BEGIN; ... ROLLBACK;`!)*
- **What to look for:**
  1. **Scan type:** `Seq Scan` / `ALL` (full scan — check if unexpected on large tables) vs `Index Scan` / `Index Only Scan` / `ref` / `const`.
  2. **Estimated vs Actual rows:** Huge mismatch means stale table statistics (`ANALYZE`).
  3. **Join strategy:** `Nested Loop` (good for small outer + indexed inner), `Hash Join` (good for large unsorted joins), `Merge Join` (sorted inputs).
  4. **Expensive operations:** `Sort` (using disk/external merge), `HashAggregate` spilling to disk, high `actual time` or `loops`.

---

### 12.7 LIKE '%abc' pe index kyun nahi lagta?
A standard B-Tree sorts strings **from left to right** (like a dictionary).
- `LIKE 'abc%'` can jump directly to `'abc'` in the tree.
- `LIKE '%abc'` has an unknown prefix, so matching values can start with `'a'`, `'b'`, ..., `'z'` anywhere in the tree — a B-Tree range lookup is impossible.
- **Solutions:**
  - **PostgreSQL:** Use a **GIN trigram index (`pg_trgm`)** (`CREATE INDEX ... USING gin (name gin_trgm_ops)`), which indexes 3-char chunks and accelerates `%abc%`!
  - Or use **Full-Text Search** (`tsvector` / `FULLTEXT`), or reverse-string index for suffix-only searches.

---

### 12.8 Function lagane se (WHERE YEAR(date)=2025) index kyun skip hota hai?
The B-Tree index stores raw `date` values, **not** `YEAR(date)` results. Wrapping the column in a function makes the predicate **non-sargable** (Search ARGument-able), forcing the DB to compute `YEAR(date)` on every row!
- **Fix (rewrite as range):**
  ```sql
  -- Fast, uses index on `date`:
  WHERE date >= '2025-01-01' AND date < '2026-01-01'
  ```
- *(Alternative in PostgreSQL/MySQL 8+: create an **expression/functional index** on `(EXTRACT(YEAR FROM date))`.)*

---

### 12.9 N+1 query problem kya hai?
Common ORM/backend bug:
1. Run **1 query** to fetch `N` parent records (`SELECT * FROM users LIMIT 50`).
2. Loop in application code and run **1 query per parent** (`SELECT * FROM orders WHERE user_id = ?`) → `1 + 50 = 51` DB round-trips!
- **Fix:** Fetch related data in **1 `JOIN` query** or **2 queries using `WHERE user_id IN (...)`** (eager loading / DataLoader batching).

---

### 12.10 Pagination: OFFSET vs keyset pagination?
- **`OFFSET` pagination (`LIMIT 20 OFFSET 100000`):**
  - DB must scan and discard the first 100,000 rows → gets linearly slower on deep pages (`O(OFFSET + LIMIT)`).
  - Also causes **duplicate/skipped rows** if new items are inserted while the user paginates.
- **Keyset (Cursor) pagination:**
  - Uses the last seen row's key with an indexed `WHERE` clause:
    ```sql
    SELECT id, created_at, title
    FROM posts
    WHERE (created_at, id) < ('2026-10-01 10:00:00', 5432)
    ORDER BY created_at DESC, id DESC
    LIMIT 20;
    ```
  - **Constant `O(log N)` performance** even at page 10,000, and no shifted/duplicate rows. *(Trade-off: cannot jump directly to "page 47".)*

---

### 12.11 Slow query debug karne ka step-by-step approach batao.
1. **Reproduce & measure:** Run `EXPLAIN (ANALYZE, BUFFERS)` (Postgres) or `EXPLAIN ANALYZE` (MySQL).
2. **Find the bottleneck node:** Look for full table scans (`Seq Scan`/`ALL`), huge row estimates mismatch, disk sorts, or nested loops with high `loops`.
3. **Check query & predicates:** Fix non-sargable functions, `SELECT *`, `OR` conditions, `OFFSET`, or `NOT IN` / correlated subqueries.
4. **Check indexes:** Add missing composite/covering indexes (following ESR rule) or remove redundant ones.
5. **Check table health & stats:** Run `ANALYZE`, check table/index bloat or fragmented pages.
6. **Check locks & system load:** Inspect `pg_stat_activity` / `SHOW PROCESSLIST` to see if the query is actually waiting on locks or I/O rather than CPU.

---

### 12.12 Too many indexes ka downside?
1. **Slower writes (`INSERT`, `UPDATE`, `DELETE`):** Every write must update the table AND every index B-Tree on that table.
2. **More disk & RAM usage:** Large indexes compete with table pages in the buffer pool/cache, reducing cache hit rate.
3. **Longer query planning time** and higher vacuum/maintenance overhead.

---

## 13. Transactions & Concurrency

### 13.1 ACID properties example ke saath.
*(Example: Bank transfer of ₹1,000 from Account A to Account B)*
- **Atomicity (All or Nothing):** Either both debit from A and credit to B succeed, or neither happens (rolled back on crash/error).
- **Consistency (Valid state to valid state):** All constraints (`balance >= 0`, FK, triggers) remain satisfied; total money across A + B stays conserved.
- **Isolation (Concurrent safety):** Concurrent transactions don't see intermediate/half-finished states of this transfer.
- **Durability (Permanent once committed):** Once `COMMIT` returns success, changes survive crashes/power loss via Write-Ahead Logging (WAL / redo log).

---

### 13.2 Isolation levels aur unke problems (dirty read, non-repeatable read, phantom read).
- **Concurrency anomalies:**
  - **Dirty Read:** Reading uncommitted data from another transaction that might later roll back.
  - **Non-Repeatable Read:** Reading the same row twice inside one transaction yields different values because another transaction updated/deleted and committed it.
  - **Phantom Read:** Re-running a range query (`WHERE age > 20`) inside one transaction returns new/missing rows inserted/deleted and committed by another transaction.

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| **READ UNCOMMITTED** | Possible | Possible | Possible |
| **READ COMMITTED** *(Postgres default)* | Prevented | Possible | Possible |
| **REPEATABLE READ** *(MySQL InnoDB default)* | Prevented | Prevented | Prevented in Postgres (snapshot); prevented for locking reads in InnoDB via Gap Locks |
| **SERIALIZABLE** | Prevented | Prevented | Prevented |

---

### 13.3 Deadlock kya hai aur kaise avoid karoge?
A **deadlock** occurs when Transaction 1 holds Lock A and waits for Lock B, while Transaction 2 holds Lock B and waits for Lock A — neither can proceed. The DB detects the cycle and aborts one transaction (victim) with a deadlock error.
- **How to avoid / handle:**
  1. **Consistent lock ordering:** Always lock/update rows or tables in the **same deterministic order** (e.g., `ORDER BY id ASC` before locking multiple rows).
  2. **Keep transactions short and fast:** Do external API calls, emails, or heavy computation *outside* the DB transaction.
  3. **Use proper indexes:** Missing indexes cause InnoDB to lock many scanned rows (or full table) instead of just the target row.
  4. **Implement retry logic** in application code for transient deadlock errors.

---

### 13.4 Optimistic vs pessimistic locking?
- **Pessimistic Locking:** Assumes conflicts are frequent → **locks the row immediately** (`SELECT ... FOR UPDATE`) until commit. Prevents lost updates, safe for high-contention inventory/banking, but reduces concurrency.
- **Optimistic Locking:** Assumes conflicts are rare → **no DB lock held during read**. Uses a `version` column (or `updated_at`) and checks it on update:
  ```sql
  UPDATE products
  SET stock = stock - 1, version = version + 1
  WHERE id = 101 AND version = 5;
  ```
  If `0 rows affected`, someone else updated it first → app retries or shows conflict error. Great for low-contention UI edits.

---

### 13.5 SELECT ... FOR UPDATE kab use karte ho?
Use inside a transaction when you need to **read a row, make a business decision in application code, and then update it**, preventing another concurrent transaction from reading/modifying the same row in between (e.g., wallet balance deduction, seat booking, stock decrement).
- Variants:
  - `FOR UPDATE NOWAIT`: Fail immediately instead of waiting if already locked.
  - `FOR UPDATE SKIP LOCKED`: Skip locked rows — ideal for **job queues** with multiple workers!

---

### 13.6 Savepoint, commit, rollback?
- **`COMMIT`:** Permanently saves all changes made in the current transaction.
- **`ROLLBACK`:** Reverts **all** changes since `BEGIN`.
- **`SAVEPOINT name`:** Sets a partial checkpoint inside a transaction so you can `ROLLBACK TO SAVEPOINT name` without aborting earlier valid work in the same transaction.
```sql
BEGIN;
INSERT INTO orders ...;
SAVEPOINT sp1;
INSERT INTO gift_cards ...; -- fails or unwanted
ROLLBACK TO SAVEPOINT sp1;  -- keeps the order insert, undoes gift_card
COMMIT;
```

---

### 13.7 Do users ek saath last stock item kharidne ki koshish karein, kya hoga?
- **Without locking (naive read-then-write):** Both read `stock = 1`, both pass `if (stock > 0)`, both update `stock = stock - 1` → **overselling (`stock = -1`)** (Race Condition / Lost Update)!
- **Proper solutions:**
  1. **Atomic conditional `UPDATE` (simplest & fastest):**
     ```sql
     UPDATE products SET stock = stock - 1 WHERE id = 101 AND stock >= 1;
     -- Check rows_affected == 1; if 0, return "Out of Stock".
     ```
  2. **Pessimistic lock (`SELECT stock FROM products WHERE id = 101 FOR UPDATE`)** inside a transaction + `CHECK (stock >= 0)` constraint as a DB-level safety net.

---

## 14. Views, Procedures, Triggers, Functions

### 14.1 View vs Materialized View?
- **View:** A saved virtual query (`SELECT`). Stores **no data** — re-executes the underlying query every time you read it. Good for hiding complexity and restricting column/row access.
- **Materialized View:** Physically **stores the query result on disk** like a cached table. Reads are instant, and you can **index** it! Needs manual/scheduled refresh (`REFRESH MATERIALIZED VIEW`) so data can be slightly stale. *(Supported natively in PostgreSQL; in MySQL, simulated via summary tables + scheduled events/cron.)*

---

### 14.2 Stored procedure vs function?
| Feature | Function | Stored Procedure |
|---|---|---|
| **Return value** | Must return a single value (or table in Postgres) | Optional (`OUT` parameters / result sets) |
| **Callable inside `SELECT`/`WHERE`** | Yes | No (invoked via `CALL proc()`) |
| **Transaction control (`COMMIT`/`ROLLBACK`)** | Not allowed | Allowed (Postgres 11+, MySQL) |
| **Side effects (DML)** | Restricted (MySQL disallows modifying tables in functions) | Fully allowed |

---

### 14.3 Trigger ka use case aur downside?
- **Use cases:** Automatic audit logging (`INSERT`/`UPDATE`/`DELETE` history), maintaining `updated_at` timestamps, enforcing complex cross-table business rules, syncing denormalized counters.
- **Downsides:**
  1. **Hidden magic:** Hard to debug because side effects happen silently inside the DB rather than application code.
  2. **Performance overhead:** Runs synchronously on every row/statement, slowing down bulk writes.
  3. **Cascading loops:** Trigger A updates Table B which fires Trigger B on Table A.

---

### 14.4 Updatable views ke rules?
A simple view can accept `INSERT`/`UPDATE`/`DELETE` if it maps 1-to-1 to base table rows:
- Derived from a **single base table** (no `JOIN`s, `UNION`, `DISTINCT`, `GROUP BY`, `HAVING`, window functions, or aggregates).
- Includes all `NOT NULL` columns without `DEFAULT` values (for `INSERT`).
- No computed/expression columns in the updated fields.
*(Use `WITH CHECK OPTION` on the view so updates cannot push rows outside the view's `WHERE` filter.)*

---

### 14.5 Audit log trigger se kaise banaoge?
```sql
-- PostgreSQL example:
CREATE TABLE employee_audit (
    audit_id BIGSERIAL PRIMARY KEY,
    emp_id INT,
    action VARCHAR(10),
    old_salary NUMERIC,
    new_salary NUMERIC,
    changed_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION log_salary_change()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.salary IS DISTINCT FROM NEW.salary THEN
        INSERT INTO employee_audit(emp_id, action, old_salary, new_salary)
        VALUES (NEW.id, TG_OP, OLD.salary, NEW.salary);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_emp_salary_audit
AFTER UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION log_salary_change();
```

---

## 15. Scenario-Based (E-commerce)

### 15.1 Last 7 din me jin users ne cart me add kiya par order nahi kiya, unki list.
```sql
SELECT DISTINCT c.user_id
FROM cart_items c
WHERE c.added_at >= CURRENT_DATE - INTERVAL '7 days'
  AND NOT EXISTS (
      SELECT 1
      FROM orders o
      WHERE o.user_id = c.user_id
        AND o.created_at >= c.added_at
        AND o.status <> 'cancelled'
  );
```

---

### 15.2 Har product category ka top-selling product.
```sql
WITH product_sales AS (
    SELECT p.category_id,
           p.id AS product_id,
           p.name,
           SUM(oi.quantity) AS total_units_sold
    FROM products p
    JOIN order_items oi ON p.id = oi.product_id
    JOIN orders o ON oi.order_id = o.id
    WHERE o.status = 'delivered'
    GROUP BY p.category_id, p.id, p.name
),
ranked AS (
    SELECT *,
           DENSE_RANK() OVER (PARTITION BY category_id ORDER BY total_units_sold DESC) AS rnk
    FROM product_sales
)
SELECT category_id, product_id, name, total_units_sold
FROM ranked
WHERE rnk = 1;
```

---

### 15.3 Repeat customers (2+ orders) ka percentage.
```sql
WITH customer_orders AS (
    SELECT user_id, COUNT(*) AS order_cnt
    FROM orders
    WHERE status <> 'cancelled'
    GROUP BY user_id
)
SELECT ROUND(
    COUNT(CASE WHEN order_cnt >= 2 THEN 1 END) * 100.0 / NULLIF(COUNT(*), 0),
    2
) AS repeat_customer_pct
FROM customer_orders;
```

---

### 15.4 Har user ka average order value aur lifetime value.
```sql
SELECT u.id AS user_id,
       u.name,
       COUNT(o.id) AS total_orders,
       COALESCE(ROUND(AVG(o.total_amount), 2), 0) AS aov,
       COALESCE(SUM(o.total_amount), 0) AS ltv
FROM users u
LEFT JOIN orders o
  ON u.id = o.user_id AND o.status = 'delivered'
GROUP BY u.id, u.name;
```

---

### 15.5 Un orders ko nikalo jinka payment fail hua par order status “confirmed” hai.
```sql
-- Care for multiple payment attempts per order: check if order has NO 'success' payment, or latest payment failed:
SELECT o.id AS order_id, o.user_id, o.status, p.status AS payment_status
FROM orders o
JOIN payments p ON o.id = p.order_id
WHERE o.status = 'confirmed'
  AND p.status = 'failed'
  AND NOT EXISTS (
      SELECT 1 FROM payments p2
      WHERE p2.order_id = o.id AND p2.status = 'success'
  );
```

---

### 15.6 Month-wise new vs returning customers.
```sql
WITH first_orders AS (
    SELECT user_id, MIN(DATE_TRUNC('month', order_date)::DATE) AS first_month
    FROM orders
    GROUP BY user_id
),
monthly_buyers AS (
    SELECT DISTINCT o.user_id,
           DATE_TRUNC('month', o.order_date)::DATE AS order_month,
           f.first_month
    FROM orders o
    JOIN first_orders f ON o.user_id = f.user_id
)
SELECT order_month,
       COUNT(CASE WHEN order_month = first_month THEN 1 END) AS new_customers,
       COUNT(CASE WHEN order_month > first_month THEN 1 END) AS returning_customers
FROM monthly_buyers
GROUP BY order_month
ORDER BY order_month;
```

---

### 15.7 Revenue by city, jisme sirf delivered orders count hon.
```sql
SELECT u.city,
       COALESCE(SUM(o.total_amount), 0) AS total_revenue
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'delivered'
GROUP BY u.city
ORDER BY total_revenue DESC;
```
*(Use `LEFT JOIN` with `AND o.status = 'delivered'` in the `ON` clause if you also want cities with `0` revenue included.)*

---

### 15.8 Aise users jinhone pehle order ke 30 din ke andar dusra order kiya.
```sql
WITH numbered_orders AS (
    SELECT user_id,
           order_date,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date, id) AS rn
    FROM orders
    WHERE status <> 'cancelled'
),
first_two AS (
    SELECT user_id,
           MAX(CASE WHEN rn = 1 THEN order_date END) AS first_order,
           MAX(CASE WHEN rn = 2 THEN order_date END) AS second_order
    FROM numbered_orders
    WHERE rn <= 2
    GROUP BY user_id
)
SELECT user_id, first_order, second_order
FROM first_two
WHERE second_order IS NOT NULL
  AND second_order <= first_order + INTERVAL '30 days';
```

---

### 15.9 Product ke saath aksar kaunse products saath kharide jaate hain (market basket)?
```sql
SELECT a.product_id AS product_a,
       b.product_id AS product_b,
       COUNT(*) AS bought_together_count
FROM order_items a
JOIN order_items b
  ON a.order_id = b.order_id
 AND a.product_id < b.product_id   -- avoids self-pairs & duplicate (A,B)/(B,A) pairs
GROUP BY a.product_id, b.product_id
ORDER BY bought_together_count DESC
LIMIT 20;
```

---

### 15.10 Stock negative ho gaya, kaise detect aur prevent karoge?
- **Detect:**
  ```sql
  SELECT id, name, stock FROM products WHERE stock < 0;
  ```
- **Prevent:**
  1. Add a **DB-level constraint:** `ALTER TABLE products ADD CONSTRAINT chk_stock_nonneg CHECK (stock >= 0);`
  2. Perform **atomic conditional updates** inside transactions:
     ```sql
     UPDATE products SET stock = stock - :qty WHERE id = :pid AND stock >= :qty;
     ```
     Check that `affected_rows = 1`; otherwise roll back and return "Insufficient stock".

---

### 15.11 Refund hone par revenue report me kaise adjust karoge?
Never overwrite the original `order.total_amount` (breaks accounting/audit history). Instead, keep a separate `refunds` (or `transactions` ledger) table and compute **Net Revenue = Gross Sales − Refunds**:
```sql
SELECT DATE_TRUNC('month', o.order_date)::DATE AS mth,
       SUM(o.total_amount) AS gross_revenue,
       COALESCE(SUM(r.refund_amount), 0) AS total_refunds,
       SUM(o.total_amount) - COALESCE(SUM(r.refund_amount), 0) AS net_revenue
FROM orders o
LEFT JOIN (
    SELECT order_id, SUM(amount) AS refund_amount
    FROM refunds
    WHERE status = 'completed'
    GROUP BY order_id
) r ON o.id = r.order_id
WHERE o.status IN ('delivered', 'refunded')
GROUP BY 1;
```
*(Notice pre-aggregating `refunds` before joining prevents duplicate multiplication if one order has multiple partial refunds!)*

---

### 15.12 Coupon abuse detect karo: ek user ne same coupon 3+ baar use kiya.
```sql
SELECT user_id, coupon_code, COUNT(*) AS usage_count
FROM orders
WHERE coupon_code IS NOT NULL
  AND status <> 'cancelled'
GROUP BY user_id, coupon_code
HAVING COUNT(*) >= 3;
```

---

## 16. Scenario-Based (HR / Company)

### 16.1 Un managers ko nikalo jinke under 5 se zyada reportees hain.
```sql
SELECT m.id, m.name, COUNT(e.id) AS reportee_count
FROM employees e
JOIN employees m ON e.manager_id = m.id
GROUP BY m.id, m.name
HAVING COUNT(e.id) > 5;
```

---

### 16.2 Har department me highest paid employee (ties handle karte hue).
```sql
WITH ranked AS (
    SELECT id, name, dept_id, salary,
           DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
    FROM employees
)
SELECT id, name, dept_id, salary
FROM ranked
WHERE rnk = 1;
```

---

### 16.3 Employees jo apne manager se zyada kama rahe hain.
```sql
SELECT e.id, e.name AS employee_name, e.salary AS emp_salary,
       m.name AS manager_name, m.salary AS mgr_salary
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

---

### 16.4 Pichle 6 mahine me hire hue employees ka dept-wise count.
```sql
-- Include departments with 0 recent hires using LEFT JOIN + condition in ON:
SELECT d.id, d.name AS dept_name, COUNT(e.id) AS recent_hires
FROM departments d
LEFT JOIN employees e
  ON d.id = e.dept_id
 AND e.hire_date >= CURRENT_DATE - INTERVAL '6 months' -- MySQL: DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
GROUP BY d.id, d.name;
```

---

### 16.5 Salary ka median nikalo.
```sql
-- PostgreSQL (built-in):
SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees;

-- MySQL / Standard SQL (works for both odd & even row counts):
WITH ordered AS (
    SELECT salary,
           ROW_NUMBER() OVER (ORDER BY salary) AS rn,
           COUNT(*) OVER () AS total_rows
    FROM employees
    WHERE salary IS NOT NULL
)
SELECT AVG(salary) AS median_salary
FROM ordered
WHERE rn IN (FLOOR((total_rows + 1) / 2.0), CEIL((total_rows + 1) / 2.0));
```

---

### 16.6 Company hierarchy me kisi employee ke upar ke saare managers (CEO tak).
```sql
WITH RECURSIVE managers_above AS (
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE id = :target_emp_id

    UNION ALL

    SELECT m.id, m.name, m.manager_id, ma.level + 1
    FROM employees m
    JOIN managers_above ma ON ma.manager_id = m.id
)
SELECT id, name, level
FROM managers_above
WHERE level > 0
ORDER BY level;
```

---

## 17. Scenario-Based (Analytics / Product)

### 17.1 DAU, WAU, MAU nikalo login table se.
```sql
-- For a reference date (e.g., CURRENT_DATE):
SELECT
    COUNT(DISTINCT CASE WHEN login_at >= CURRENT_DATE THEN user_id END) AS dau,
    COUNT(DISTINCT CASE WHEN login_at >= CURRENT_DATE - INTERVAL '6 days' THEN user_id END) AS wau,
    COUNT(DISTINCT CASE WHEN login_at >= CURRENT_DATE - INTERVAL '29 days' THEN user_id END) AS mau
FROM user_logins
WHERE login_at >= CURRENT_DATE - INTERVAL '29 days';
```

---

### 17.2 Retention: day-1, day-7, day-30 cohort analysis.
```sql
WITH user_signup AS (
    SELECT id AS user_id, DATE(created_at) AS cohort_date
    FROM users
),
user_activity AS (
    SELECT DISTINCT s.user_id, s.cohort_date,
           (DATE(e.event_time) - s.cohort_date) AS day_diff -- MySQL: DATEDIFF(DATE(e.event_time), s.cohort_date)
    FROM user_signup s
    JOIN events e ON s.user_id = e.user_id
)
SELECT s.cohort_date,
       COUNT(DISTINCT s.user_id) AS cohort_size,
       ROUND(COUNT(DISTINCT CASE WHEN a.day_diff = 1  THEN a.user_id END) * 100.0 / COUNT(DISTINCT s.user_id), 2) AS d1_pct,
       ROUND(COUNT(DISTINCT CASE WHEN a.day_diff = 7  THEN a.user_id END) * 100.0 / COUNT(DISTINCT s.user_id), 2) AS d7_pct,
       ROUND(COUNT(DISTINCT CASE WHEN a.day_diff = 30 THEN a.user_id END) * 100.0 / COUNT(DISTINCT s.user_id), 2) AS d30_pct
FROM user_signup s
LEFT JOIN user_activity a ON s.user_id = a.user_id
GROUP BY s.cohort_date
ORDER BY s.cohort_date;
```

---

### 17.3 Funnel: signup → add to cart → checkout → payment, har step ka drop-off.
```sql
WITH funnel AS (
    SELECT user_id,
           MAX(CASE WHEN event_name = 'signup'      THEN 1 ELSE 0 END) AS s1,
           MAX(CASE WHEN event_name = 'add_to_cart' THEN 1 ELSE 0 END) AS s2,
           MAX(CASE WHEN event_name = 'checkout'    THEN 1 ELSE 0 END) AS s3,
           MAX(CASE WHEN event_name = 'payment'     THEN 1 ELSE 0 END) AS s4
    FROM events
    GROUP BY user_id
),
counts AS (
    SELECT SUM(s1) AS signup_users,
           SUM(CASE WHEN s1=1 AND s2=1 THEN 1 ELSE 0 END) AS cart_users,
           SUM(CASE WHEN s1=1 AND s2=1 AND s3=1 THEN 1 ELSE 0 END) AS checkout_users,
           SUM(CASE WHEN s1=1 AND s2=1 AND s3=1 AND s4=1 THEN 1 ELSE 0 END) AS paid_users
    FROM funnel
)
SELECT signup_users,
       cart_users,
       ROUND((signup_users - cart_users) * 100.0 / NULLIF(signup_users, 0), 2) AS drop_signup_to_cart_pct,
       checkout_users,
       ROUND((cart_users - checkout_users) * 100.0 / NULLIF(cart_users, 0), 2) AS drop_cart_to_checkout_pct,
       paid_users,
       ROUND((checkout_users - paid_users) * 100.0 / NULLIF(checkout_users, 0), 2) AS drop_checkout_to_paid_pct
FROM counts;
```

---

### 17.4 Session nikalo: 30 min inactivity ke baad naya session.
```sql
WITH flagged AS (
    SELECT user_id, event_time,
           CASE
               WHEN event_time - LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time)
                    > INTERVAL '30 minutes'
                 OR LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time) IS NULL
               THEN 1 ELSE 0
           END AS is_new_session
    FROM events
)
SELECT user_id, event_time,
       SUM(is_new_session) OVER (PARTITION BY user_id ORDER BY event_time) AS session_num
FROM flagged;
```

---

### 17.5 Consecutive 3 din login karne wale users.
```sql
WITH distinct_logins AS (
    SELECT DISTINCT user_id, DATE(login_at) AS login_date
    FROM user_logins
),
grouped AS (
    SELECT user_id, login_date,
           login_date - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) * INTERVAL '1 day') AS grp
           -- MySQL: DATE_SUB(login_date, INTERVAL ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) DAY)
    FROM distinct_logins
)
SELECT DISTINCT user_id
FROM grouped
GROUP BY user_id, grp
HAVING COUNT(*) >= 3;
```

---

### 17.6 Longest login streak per user.
```sql
WITH distinct_logins AS (
    SELECT DISTINCT user_id, DATE(login_at) AS login_date
    FROM user_logins
),
streaks AS (
    SELECT user_id,
           login_date - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) * INTERVAL '1 day') AS grp
    FROM distinct_logins
),
streak_lengths AS (
    SELECT user_id, grp, COUNT(*) AS streak_days
    FROM streaks
    GROUP BY user_id, grp
)
SELECT user_id, MAX(streak_days) AS longest_streak
FROM streak_lengths
GROUP BY user_id;
```

---

### 17.7 Peak hour of orders.
```sql
SELECT EXTRACT(HOUR FROM created_at) AS order_hour, -- MySQL: HOUR(created_at)
       COUNT(*) AS total_orders
FROM orders
GROUP BY 1
ORDER BY total_orders DESC
LIMIT 1;
```

---

### 17.8 Un users ko dhundo jinki activity pichle 30 din me zero hai (churned).
```sql
SELECT u.id, u.email
FROM users u
WHERE u.created_at < CURRENT_DATE - INTERVAL '30 days'
  AND NOT EXISTS (
      SELECT 1 FROM events e
      WHERE e.user_id = u.id
        AND e.event_time >= CURRENT_DATE - INTERVAL '30 days'
  );
```

---

## 18. Debugging & Optimization Scenarios

### 18.1 Ek dashboard query 30 sec le rahi hai, kya-kya check karoge?
1. Run **`EXPLAIN (ANALYZE, BUFFERS)`** (Postgres) / **`EXPLAIN ANALYZE`** (MySQL) to find the slowest node (`Seq Scan`, disk `Sort`, bad `Nested Loop`).
2. Check missing/unused indexes on `WHERE`, `JOIN`, and `ORDER BY` columns (fix non-sargable functions like `DATE(created_at) = ...`).
3. Check if the dashboard is scanning **months/years of raw rows** on every refresh → pre-aggregate into a **Materialized View** or daily summary table!
4. Check for **join row explosion** or unnecessary `DISTINCT` / `ORDER BY` on huge intermediate results.
5. Verify table statistics (`ANALYZE`) and lock waits (`pg_stat_activity` / `SHOW PROCESSLIST`), and route heavy BI queries to a **read replica**.

---

### 18.2 Production me table lock ho gayi, kaise investigate karoge?
- **PostgreSQL:**
  Query `pg_stat_activity` and `pg_locks` (using `pg_blocking_pids(pid)`):
  ```sql
  SELECT a.pid, a.state, a.wait_event_type, a.query,
         pg_blocking_pids(a.pid) AS blocked_by,
         NOW() - a.query_start AS duration
  FROM pg_stat_activity a
  WHERE cardinality(pg_blocking_pids(a.pid)) > 0
     OR a.pid IN (SELECT unnest(pg_blocking_pids(pid)) FROM pg_stat_activity);
  ```
- **MySQL:**
  Check `SHOW PROCESSLIST;` and `SELECT * FROM performance_schema.data_lock_waits;` / `information_schema.innodb_trx`.
- **Resolution:** Identify the root blocker (often an idle-in-transaction session or unindexed `UPDATE`/DDL migration). Kill only after confirming impact (`SELECT pg_terminate_backend(pid);` / `KILL <id>;`).

---

### 18.3 Report query me wrong totals aa rahe hain, join se rows duplicate toh nahi ho rahe? Kaise verify karoge?
Yes—joining two 1-to-many child tables (e.g., `orders → order_items` AND `orders → payments`) creates a Cartesian product per order, inflating `SUM()`!
- **How to verify:**
  1. Compare `COUNT(*)` vs `COUNT(DISTINCT primary_key)` after the joins.
  2. Check row counts before and after adding each `JOIN` step-by-step.
- **How to fix:** **Pre-aggregate** each child table in its own CTE/subquery grouped by `order_id` *before* joining them together.

---

### 18.4 10 crore rows ki table me data archive kaise karoge?
Never run a single giant `DELETE FROM logs WHERE created_at < '2025-01-01'` (causes massive table/row locks, transaction log bloat, and replication lag!).
- **Best approaches:**
  1. **Chunked/Batched archiving:** Copy & delete in small batches (e.g., `5,000–10,000` rows per transaction ordered by PK, with brief sleep between batches) during off-peak hours.
  2. **Table Partitioning (best long-term fix):** Partition by month/quarter (`RANGE` partitioning). Archiving an old month becomes an instant **`ALTER TABLE ... DETACH PARTITION`** (Postgres) or **`DROP PARTITION`** (MySQL) with zero bloat!

---

### 18.5 Ek query ko bina index add kiye kaise fast karoge?
1. **Rewrite non-sargable predicates:** Replace `YEAR(dt) = 2025` with `dt >= '2025-01-01' AND dt < '2026-01-01'` so existing indexes work.
2. **Replace `SELECT *` with needed columns** (reduces I/O, enables existing covering index scans).
3. **Replace `UNION` with `UNION ALL`** if duplicates aren't possible/needed.
4. **Replace `NOT IN` / correlated subqueries** with `NOT EXISTS` or a `JOIN`.
5. **Pre-aggregate before joining**, and filter early inside CTEs.
6. **Switch `OFFSET` pagination to Keyset pagination**.
7. **Run `ANALYZE`** so the planner picks a better join order/plan, or use a **materialized/summary table** + **read replica**.

---

### 18.6 Read-heavy vs write-heavy system ke liye DB design me kya change karoge?
- **Read-Heavy System:**
  - Add targeted **covering/composite indexes** & materialized views.
  - Strategic **denormalization** (pre-computed counts/totals).
  - Add **Read Replicas** + caching layer (**Redis**).
- **Write-Heavy System:**
  - Keep **indexes minimal** (every index slows writes).
  - Use **append-only / event-log** tables instead of frequent `UPDATE`s; **partition** tables to keep hot indexes small in RAM.
  - Use **batch inserts**, async queues (Kafka/RabbitMQ) to smooth spikes, and sequential PKs (`BIGINT` / `UUIDv7`) to avoid random B-Tree page splits.

---

### 18.7 Ek table me 500 columns hain, kya problem hai aur kaise fix karoge?
- **Problems:**
  1. **Row size & I/O bloat:** Fewer rows fit per 8KB/16KB DB page → `SELECT *` or full scans read huge amounts of unused data.
  2. **Sparse NULL waste** and hitting DB row-size / column-count limits (e.g., InnoDB 65,535-byte row limit).
  3. **Lock contention:** Updating one rare field locks/rewrites the whole wide row.
- **How to fix (Vertical Partitioning + Normalization):**
  1. Split into a lean **core table** (frequently queried columns) + **1-to-1 extension tables** (e.g., `user_profile`, `user_billing`, `user_preferences`).
  2. Move repeating numbered columns (`phone1, phone2, phone3...`) into a proper **1-to-many child table**.
  3. Store truly dynamic/sparse optional attributes in a **`JSONB` column** (Postgres).

---

## 19. Database Design Scenarios

### 19.1 Instagram-style follow/follower system ka schema.
```sql
CREATE TABLE follows (
    follower_id BIGINT NOT NULL REFERENCES users(id),
    followee_id BIGINT NOT NULL REFERENCES users(id),
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (follower_id, followee_id),        -- fast lookup: "who does X follow?"
    CHECK (follower_id <> followee_id)             -- prevent self-follow
);

-- Reverse index for fast lookup: "who follows X?"
CREATE INDEX idx_follows_followee ON follows(followee_id, follower_id);
```
*(Scale tip: Maintain denormalized `followers_count` & `following_count` on `users` or in Redis instead of running `COUNT(*)` on millions of rows per profile view.)*

---

### 19.2 Many-to-many relationship (students–courses) ka design.
Use a **junction (bridge) table**:
```sql
CREATE TABLE students (id BIGSERIAL PRIMARY KEY, name VARCHAR(100) NOT NULL);
CREATE TABLE courses  (id BIGSERIAL PRIMARY KEY, title VARCHAR(150) NOT NULL);

CREATE TABLE enrollments (
    student_id  BIGINT NOT NULL REFERENCES students(id) ON DELETE CASCADE,
    course_id   BIGINT NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    enrolled_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    grade       VARCHAR(5),
    PRIMARY KEY (student_id, course_id)
);
CREATE INDEX idx_enrollments_course ON enrollments(course_id);
```

---

### 19.3 Multi-tenant SaaS ke liye schema strategies.
1. **Shared DB, Shared Schema (Row-level isolation via `tenant_id`):**
   - *Pros:* Lowest cost, easiest migrations, scales to thousands of small tenants.
   - *Cons:* Must never forget `WHERE tenant_id = ?` (use **Row-Level Security (RLS)** in PostgreSQL for safety!).
2. **Shared DB, Separate Schemas (`tenant_a.orders`, `tenant_b.orders`):**
   - *Pros:* Logical separation, easy per-tenant backup/restore.
   - *Cons:* Schema migrations must loop over all tenants; metadata bloat at >1,000 tenants.
3. **Separate DB per Tenant:**
   - *Pros:* Strongest isolation, compliance (HIPAA/enterprise), zero noisy-neighbor impact.
   - *Cons:* Highest cost, connection pooling & migration complexity.

---

### 19.4 Soft delete vs hard delete?
- **Hard Delete (`DELETE FROM ...`):** Physically removes the row. Keeps tables small & `UNIQUE` constraints simple, but loses audit history and can break FK references.
- **Soft Delete (`deleted_at TIMESTAMPTZ NULL`):** Marks row as deleted without removing it.
  - *Pros:* Easy undo/recovery, preserves audit & order history.
  - *Cons:* Every query must add `WHERE deleted_at IS NULL`; breaks simple `UNIQUE(email)` (fix in Postgres with **partial unique index:** `CREATE UNIQUE INDEX ... ON users(email) WHERE deleted_at IS NULL`).
- **Best practice:** Soft delete (or `status = 'archived'`) for business/user entities; hard delete for ephemeral tokens/logs or GDPR "right to be forgotten" compliance.

---

### 19.5 Order history me price change hone par purana price kaise preserve karoge?
**Snapshot the price at purchase time** inside `order_items` (never rely only on `JOIN products` for historical order totals!):
```sql
CREATE TABLE order_items (
    id                 BIGSERIAL PRIMARY KEY,
    order_id           BIGINT NOT NULL REFERENCES orders(id),
    product_id         BIGINT NOT NULL REFERENCES products(id),
    product_name_snap  VARCHAR(200) NOT NULL, -- optional snapshot
    unit_price         NUMERIC(10, 2) NOT NULL,
    quantity           INT NOT NULL CHECK (quantity > 0)
);
```

---

### 19.6 Hierarchical data (comments, categories) store karne ke tareeke: adjacency list, nested set, materialized path.
1. **Adjacency List (`parent_id` column):**
   - *Pros:* Simple schema, instant `INSERT`/move.
   - *Cons:* Fetching full subtree requires **Recursive CTE**.
2. **Materialized Path (`path = '1/4/12/'`):**
   - *Pros:* Instant subtree fetch via `WHERE path LIKE '1/4/%'` (uses B-Tree index); easy depth count.
   - *Cons:* Moving a parent node requires updating `path` of all descendants.
3. **Nested Set (`lft`, `rgt` integers):**
   - *Pros:* Very fast subtree reads (`WHERE lft BETWEEN parent.lft AND parent.rgt`).
   - *Cons:* Very expensive writes — inserting/moving one node requires recalculating `lft`/`rgt` across half the tree!
- **Pick:** **Adjacency List + Recursive CTE** for most apps (like comments), or **Materialized Path** (`ltree` in Postgres) for read-heavy category trees.

---

### 19.7 Notification system ka schema (read/unread, types).
```sql
CREATE TABLE notifications (
    id           BIGSERIAL PRIMARY KEY,
    user_id      BIGINT NOT NULL REFERENCES users(id),
    type         VARCHAR(50) NOT NULL,   -- e.g. 'order_shipped', 'comment_reply'
    payload      JSONB NOT NULL,         -- flexible metadata (order_id, actor_name, link)
    is_read      BOOLEAN NOT NULL DEFAULT FALSE,
    read_at      TIMESTAMPTZ,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Fast unread badge count & notification feed:
CREATE INDEX idx_notif_user_unread ON notifications(user_id, created_at DESC) WHERE is_read = FALSE;
CREATE INDEX idx_notif_user_feed   ON notifications(user_id, created_at DESC);
```

---

### 19.8 Inventory + warehouse + stock movement ka design.
Never just overwrite a `stock` number without an audit trail—combine a **current balance table** with an **append-only ledger**:
```sql
CREATE TABLE warehouses (id BIGSERIAL PRIMARY KEY, name VARCHAR(100) NOT NULL);

-- Current fast-read inventory per warehouse + SKU:
CREATE TABLE inventory (
    warehouse_id BIGINT NOT NULL REFERENCES warehouses(id),
    product_id   BIGINT NOT NULL REFERENCES products(id),
    qty_on_hand  INT NOT NULL DEFAULT 0 CHECK (qty_on_hand >= 0),
    qty_reserved INT NOT NULL DEFAULT 0 CHECK (qty_reserved >= 0),
    PRIMARY KEY (warehouse_id, product_id)
);

-- Immutable audit ledger of every stock change:
CREATE TABLE stock_movements (
    id            BIGSERIAL PRIMARY KEY,
    warehouse_id  BIGINT NOT NULL,
    product_id    BIGINT NOT NULL,
    delta_qty     INT NOT NULL,               -- +50 restock, -2 sold, etc.
    reason        VARCHAR(50) NOT NULL,       -- 'purchase', 'sale', 'return', 'adjustment'
    reference_id  BIGINT,                     -- order_id / shipment_id
    created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 20. MySQL vs PostgreSQL (tumhare stack ke liye)

### 20.1 AUTO_INCREMENT vs SERIAL / IDENTITY?
- **MySQL:** Uses **`AUTO_INCREMENT`** on an integer PK column.
- **PostgreSQL:**
  - **`SERIAL` / `BIGSERIAL`:** Older shorthand that creates a `SEQUENCE` and sets `DEFAULT nextval(...)`.
  - **`GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY` (Postgres 10+):** **SQL-standard and recommended** over `SERIAL` — cleaner permissions, prevents accidental manual ID insertion if `ALWAYS` is used.

---

### 20.2 JSON/JSONB columns: query aur indexing.
- **MySQL:** Has **`JSON`** (stored in optimized binary format). Query with `->>` (`data->>'$.city'`). Indexing uses **generated/virtual columns** + B-Tree index (or multi-valued indexes in MySQL 8.0.17+ for arrays).
- **PostgreSQL:** Has **`JSON`** (stores raw text) and **`JSONB`** (binary parsed, **preferred** — faster queries, removes whitespace/duplicate keys).
  - Query: `data->>'city'` (text), `data->'addr'->>'zip'`.
  - Indexing: **GIN index** (`CREATE INDEX idx_gin ON events USING gin (data);`) directly supports `@>` containment queries (`WHERE data @> '{"plan":"pro"}'`), or expression B-Tree index on `(data->>'city')`.

---

### 20.3 Postgres me RETURNING clause ka use?
`RETURNING` returns affected column values directly from `INSERT`, `UPDATE`, or `DELETE` in **one DB round-trip** (no extra `SELECT` needed):
```sql
INSERT INTO orders (user_id, total_amount)
VALUES (42, 999.00)
RETURNING id, created_at;

UPDATE inventory SET qty = qty - 1 WHERE product_id = 10 RETURNING qty;
```
*(MySQL lacks `RETURNING` for `INSERT`/`UPDATE` — you have to call `LAST_INSERT_ID()` or another `SELECT`.)*

---

### 20.4 MySQL InnoDB vs MyISAM?
- **InnoDB (Default & recommended):** Supports **ACID transactions**, **row-level locking**, **foreign keys**, and **crash recovery**.
- **MyISAM (Legacy):** No transactions, no foreign keys, **table-level locking** (every write locks the whole table!), no crash safety. Avoid in modern apps.

---

### 20.5 Full-text search dono me kaise hota hai?
- **MySQL:** Create a `FULLTEXT` index on `TEXT`/`VARCHAR` columns and query with `MATCH(title, body) AGAINST('search term' IN NATURAL LANGUAGE MODE)` (or `IN BOOLEAN MODE`).
- **PostgreSQL:** Converts text to **`tsvector`** (tokens/lexemes) and queries with **`tsquery`** using **`@@`** operator, accelerated by a **GIN index**:
  ```sql
  CREATE INDEX idx_articles_fts ON articles USING gin (to_tsvector('english', title || ' ' || body));

  SELECT id, title FROM articles
  WHERE to_tsvector('english', title || ' ' || body) @@ plainto_tsquery('english', 'sql indexing');
  ```

---

### 20.6 Postgres partial index aur expression index?
- **Partial Index:** Indexes only rows matching a `WHERE` condition → tiny index size, faster writes & lookups!
  ```sql
  -- Only index pending orders (say 2% of table), ignoring millions of completed ones:
  CREATE INDEX idx_orders_pending ON orders(created_at) WHERE status = 'pending';
  ```
- **Expression (Functional) Index:** Indexes the result of a function/expression:
  ```sql
  CREATE INDEX idx_users_lower_email ON users(LOWER(email));
  -- Query `WHERE LOWER(email) = 'abc@x.com'` now uses the index!
  ```
*(MySQL 8.0.13+ also supports functional indexes via `((LOWER(email)))`, though MySQL lacks native partial indexes.)*

---

### 20.7 Partitioning (range, list, hash) kab use karoge?
Use partitioning when a single table grows huge (e.g., **tens/hundreds of millions of rows** / >50–100 GB) so queries and data retention stay fast:
- **`RANGE`:** Split by continuous ranges like **date/month** (most common — logs, orders, events). Enables instant `DROP/DETACH PARTITION` for archiving!
- **`LIST`:** Split by discrete categories like **region/country** (`'IN'`, `'US'`, `'EU'`).
- **`HASH`:** Split evenly by hash of `user_id`/`tenant_id` across `N` partitions to spread write I/O when there's no natural date/region key.

---

### 20.8 Connection pooling kyun zaroori hai?
Opening a fresh DB connection (TCP + TLS + auth + DB process/thread allocation — especially heavy in PostgreSQL, which forks a process per connection!) takes **10–50 ms** and consumes **5–10 MB RAM per connection**.
Under high traffic, opening a connection per request crashes the DB (`too many connections`) and causes context-switching overhead.
A **connection pool** (e.g., **HikariCP** in Java, **pg-pool**, or **PgBouncer** for Postgres) keeps a small, reusable set of warm connections open, queuing bursts safely.

---

## 21. Rapid-Fire Theory

### 21.1 Cardinality vs selectivity?
- **Cardinality:** Total number of **unique values** in a column (e.g., `email` = high cardinality; `gender` = low cardinality).
- **Selectivity:** Ratio of rows matching a query/distinct values to total rows (`distinct_values / total_rows`, `0` to `1`). High selectivity (near `1`) = B-Tree index is very effective; low selectivity = full scan is often faster.

---

### 21.2 OLTP vs OLAP?
- **OLTP (Online Transaction Processing):** Day-to-day app traffic (orders, logins, payments). Many short, concurrent reads/writes, normalized schema, low-latency row lookups (MySQL, PostgreSQL).
- **OLAP (Online Analytical Processing):** BI, dashboards, reporting across years of data. Few complex queries scanning millions of rows, denormalized star/snowflake schema, **columnar storage** (ClickHouse, BigQuery, Redshift, Snowflake).

---

### 21.3 Sharding vs replication vs partitioning?
- **Replication:** Copying the **same data** to multiple servers (Primary → Read Replicas) for **read scaling** and **high availability**.
- **Partitioning:** Splitting **one large table** into smaller physical pieces on the **same DB server** for manageability and partition pruning.
- **Sharding:** Splitting data horizontally across **multiple independent DB servers** (each server holds a subset of rows, e.g., by `user_id % 4`) for **write & storage scaling**.

---

### 21.4 CAP theorem?
In a distributed data store during a **Network Partition (`P`)** (which is unavoidable in distributed networks), you must choose between:
- **Consistency (`C`):** Every read receives the most recent write (or an error).
- **Availability (`A`):** Every request receives a non-error response (even if data might be stale).
*(Thus distributed systems are either **CP** — e.g., HBase/MongoDB default strong reads, or **AP** — e.g., Cassandra/DynamoDB eventual consistency.)*

---

### 21.5 SQL injection kya hai aur kaise rokte ho?
- **SQL Injection:** Security vulnerability where untrusted user input is concatenated directly into a SQL string, allowing attackers to alter query logic (e.g., `' OR '1'='1'; DROP TABLE users;--`).
- **How to prevent:**
  1. **Parameterized queries / Prepared statements** (`?` or `$1` placeholders — input is treated strictly as data, never executable SQL).
  2. **ORMs** using parameterized bindings (avoid raw string interpolation!).
  3. **Input validation / allowlisting** for dynamic identifiers (column names in `ORDER BY` cannot be parameterized!).
  4. **Least-privilege DB user** (app account shouldn't have `DROP`/`SUPER` rights).

---

### 21.6 Prepared statements ke fayde?
1. **Prevents SQL Injection** completely by separating SQL code from parameter values.
2. **Performance:** The DB parses, validates, and caches the execution plan once and reuses it across repeated executions with different parameters.
3. **Cleaner handling** of quotes, special characters, and types.

---

### 21.7 Temp table vs table variable?
- **Temp Table (`CREATE TEMP TABLE`):** Physically created in temp DB/storage, supports **indexes**, **table statistics**, and participates in transactions (mostly used in Postgres, MySQL, SQL Server). Best for medium-to-large intermediate datasets.
- **Table Variable (`DECLARE @t TABLE` — primarily SQL Server; Postgres uses arrays/CTEs/temp tables):** Scoped to a batch/function, usually has no column statistics (planner assumes 1 row → bad join plans on big data), and doesn't roll back on `ROLLBACK`. Best only for tiny row sets (<100 rows).

---

### 21.8 DISTINCT vs GROUP BY?
- Both deduplicate rows, and modern optimizers often generate the **exact same execution plan** when no aggregate function is used.
- Use **`DISTINCT`** when you simply want unique combinations of selected columns (clearer intent).
- Use **`GROUP BY`** when you need **aggregates** (`COUNT`, `SUM`, `MAX`) or `HAVING` per group.

---

### 21.9 Self-referencing foreign key?
A foreign key on a table that references the **Primary Key of the same table** — used to model hierarchies/trees (e.g., `employees.manager_id REFERENCES employees(id)` or `categories.parent_id REFERENCES categories(id)`).

---

### 21.10 Orphan records kya hote hain?
Child rows whose Foreign Key value points to a **non-existent parent row** (e.g., an `order_item` with `order_id = 999`, while `order 999` was deleted or never existed).
- Caused by missing Foreign Key constraints (e.g., MyISAM tables or disabled FK checks).
- **Detect:** `SELECT c.* FROM child c LEFT JOIN parent p ON c.parent_id = p.id WHERE p.id IS NULL;`
- **Prevent:** Always enforce `FOREIGN KEY` constraints (`ON DELETE RESTRICT` or `CASCADE`).
