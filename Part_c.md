````markdown
# Q1 — Logical Execution Order of a SQL SELECT Statement

Although a SQL query is **written** in the order:

SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY

the **logical execution order** used by the SQL engine is:

1. **FROM** – Determine the source tables and perform joins  
2. **WHERE** – Filter rows before grouping  
3. **GROUP BY** – Group rows into aggregates  
4. **HAVING** – Filter groups after aggregation  
5. **SELECT** – Compute and return selected columns  
6. **ORDER BY** – Sort the final result set  
7. **LIMIT / OFFSET** (if present) – Restrict the number of rows returned

### Why this matters for aliases

Aliases defined in the **SELECT** clause are created **after** the `WHERE` clause is processed.

Therefore:

- Aliases **cannot be used in WHERE**
- Aliases **can be used in ORDER BY**
- Some databases allow them in **HAVING**, but not in `WHERE`

Example:

```sql
SELECT salary * 12 AS annual_salary
FROM employees
WHERE annual_salary > 100000;  -- ❌ invalid
````

Correct approach:

```sql
SELECT salary * 12 AS annual_salary
FROM employees
WHERE salary * 12 > 100000;
```

Because `WHERE` is executed **before SELECT**, the alias does not yet exist.

---

# Q2 — Single SQL Query (No Subqueries or CTEs)

Goal:
Show

* employee name
* salary
* department average salary

for employees whose salary is **above the company-wide average**.

This can be solved using **window functions**:

```sql
SELECT 
    name,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary
FROM employees
WHERE salary > AVG(salary) OVER ();
```

Explanation:

* `AVG(salary) OVER ()` → calculates the **company-wide average salary**
* `AVG(salary) OVER (PARTITION BY department)` → calculates **average salary per department**
* Window functions allow these calculations **without subqueries or CTEs**

---

# Q3 — Debug the Query

### Given Query

```sql
SELECT department, AVG(salary) as avg_sal
FROM employees
WHERE AVG(salary) > 70000
GROUP BY department;
```

### Bug

The problem is:

```
WHERE AVG(salary) > 70000
```

Aggregate functions such as `AVG()` **cannot be used in the WHERE clause** because `WHERE` executes **before aggregation**.

---

### Correct Query

```sql
SELECT department, AVG(salary) AS avg_sal
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000;
```

### Explanation

* `GROUP BY department` creates salary groups per department
* `AVG(salary)` calculates the average salary for each group
* `HAVING` filters groups **after aggregation**

So the corrected logic is:

```
GROUP BY → calculate AVG → HAVING filter
```
