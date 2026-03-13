
# SQL + Pandas Comparison Exercise (Employees Database)

Assumed schema:

employees(emp_id, name, department_id, salary, hire_date)
departments(department_id, department_name, budget)

Example employees data used for outputs:

| emp_id | name    | department_id | salary | hire_date  |
|-------|---------|---------------|--------|------------|
| 1 | Alice | 1 | 90000 | 2021-01-10 |
| 2 | Bob | 1 | 75000 | 2022-03-15 |
| 3 | Charlie | 2 | 82000 | 2020-06-20 |
| 4 | David | 2 | 67000 | 2023-02-11 |
| 5 | Eva | 3 | 72000 | 2021-09-01 |

Departments:

| department_id | department_name | budget |
|---|---|---|
|1|Engineering|250000|
|2|Data Science|220000|
|3|IT Operations|90000|

---
# SECTION 1 — BASIC SELECT / WHERE (5)

## Q1 Employees with salary > 80000

SQL
```sql
SELECT name, salary
FROM employees
WHERE salary > 80000;
```

Pandas
```python
employees[employees["salary"] > 80000][["name","salary"]]
```

Output

| name | salary |
|---|---|
| Alice | 90000 |
| Charlie | 82000 |

---

## Q2 Employees hired after 2021

SQL
```sql
SELECT name, hire_date
FROM employees
WHERE hire_date > '2021-01-01';
```

Pandas

```python
employees[employees["hire_date"] > "2021-01-01"][["name","hire_date"]]
```

Output

| name | hire_date |
|---|---|
| Bob | 2022‑03‑15 |
| David | 2023‑02‑11 |

---

## Q3 Employees in department 1

SQL
```sql
SELECT *
FROM employees
WHERE department_id = 1;
```

Pandas

```python
employees[employees["department_id"] == 1]
```

Output

| emp_id | name | department_id | salary |
|---|---|---|---|
|1|Alice|1|90000|
|2|Bob|1|75000|

---

## Q4 Employees salary between 70000 and 85000

SQL
```sql
SELECT name,salary
FROM employees
WHERE salary BETWEEN 70000 AND 85000;
```

Pandas

```python
employees[(employees.salary>=70000) & (employees.salary<=85000)][["name","salary"]]
```

Output

| name | salary |
|---|---|
| Bob | 75000 |
| Charlie | 82000 |
| Eva | 72000 |

---

## Q5 Sort employees by salary desc

SQL

```sql
SELECT name,salary
FROM employees
ORDER BY salary DESC;
```

Pandas

```python
employees.sort_values("salary",ascending=False)[["name","salary"]]
```

Output

| name | salary |
|---|---|
| Alice | 90000 |
| Charlie | 82000 |
| Bob | 75000 |
| Eva | 72000 |
| David | 67000 |

---

# SECTION 2 — JOINS (5)

## Q6 Employee + Department name

SQL

```sql
SELECT e.name,d.department_name
FROM employees e
JOIN departments d
ON e.department_id = d.department_id;
```

Pandas

```python
employees.merge(departments,on="department_id")[["name","department_name"]]
```

Output

| name | department |
|---|---|
|Alice|Engineering|
|Bob|Engineering|
|Charlie|Data Science|
|David|Data Science|
|Eva|IT Operations|

---

## Q7 Employees with department budget

SQL

```sql
SELECT e.name,e.salary,d.budget
FROM employees e
JOIN departments d
ON e.department_id=d.department_id;
```

Pandas

```python
employees.merge(departments,on="department_id")[["name","salary","budget"]]
```

---

## Q8 Count employees per department

SQL

```sql
SELECT d.department_name,COUNT(e.emp_id)
FROM departments d
LEFT JOIN employees e
ON d.department_id=e.department_id
GROUP BY d.department_name;
```

Pandas

```python
employees.merge(departments,on="department_id") .groupby("department_name").emp_id.count()
```

---

## Q9 Employees with department name and salary >70000

SQL

```sql
SELECT e.name,d.department_name
FROM employees e
JOIN departments d
ON e.department_id=d.department_id
WHERE salary>70000;
```

Pandas

```python
employees.merge(departments,on="department_id") .query("salary>70000")[["name","department_name"]]
```

---

## Q10 Department budgets with employee names

SQL

```sql
SELECT d.department_name,e.name
FROM departments d
LEFT JOIN employees e
ON d.department_id=e.department_id;
```

Pandas

```python
departments.merge(employees,on="department_id",how="left")
```

---

# SECTION 3 — AGGREGATIONS (5)

## Q11 Average salary

SQL

```sql
SELECT AVG(salary) FROM employees;
```

Pandas

```python
employees.salary.mean()
```

Output ≈ 77200

---

## Q12 Total salary per department

SQL

```sql
SELECT department_id,SUM(salary)
FROM employees
GROUP BY department_id;
```

Pandas

```python
employees.groupby("department_id").salary.sum()
```

---

## Q13 Max salary

SQL

```sql
SELECT MAX(salary) FROM employees;
```

Pandas

```python
employees.salary.max()
```

---

## Q14 Avg salary per department

SQL

```sql
SELECT department_id,AVG(salary)
FROM employees
GROUP BY department_id;
```

Pandas

```python
employees.groupby("department_id").salary.mean()
```

---

## Q15 Departments with avg salary >75000

SQL

```sql
SELECT department_id,AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary)>75000;
```

Pandas

```python
employees.groupby("department_id").salary.mean().query(">75000")
```

---

# ANALYSIS

### Salary filter

```sql
EXPLAIN SELECT * FROM employees WHERE salary>80000;
```

Insight: PostgreSQL performs **Seq Scan** on small tables.

---

### Join execution

```sql
EXPLAIN SELECT *
FROM employees e
JOIN departments d
ON e.department_id=d.department_id;
```

Insight: Planner uses **Hash Join**.

---

### Aggregation

```sql
EXPLAIN SELECT department_id,AVG(salary)
FROM employees
GROUP BY department_id;
```

Insight: PostgreSQL uses **HashAggregate**.

---

# ADVANCED SQL PROBLEMS

## Running total

```sql
SELECT name,
salary,
SUM(salary) OVER(ORDER BY emp_id) running_total
FROM employees;
```

## Top 3 per department

```sql
SELECT *
FROM(
SELECT *,
RANK() OVER(PARTITION BY department_id ORDER BY salary DESC) r
FROM employees
) t
WHERE r<=3;
```

## Month‑over‑month growth

```sql
SELECT month,
revenue,
revenue-LAG(revenue) OVER(ORDER BY month) growth
FROM monthly_sales;
```

## CTE

```sql
WITH dept_avg AS (
SELECT department_id,AVG(salary) avg_sal
FROM employees
GROUP BY department_id
)
SELECT *
FROM employees e
JOIN dept_avg d
ON e.department_id=d.department_id;
```

## Correlated subquery

```sql
SELECT name,salary
FROM employees e
WHERE salary >
(
SELECT AVG(salary)
FROM employees
WHERE department_id=e.department_id
);
```

