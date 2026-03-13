
# SQL Project Table Design and Queries

This document demonstrates:
1. Creating a projects table
2. Inserting sample data
3. Running queries involving 3-table JOINs
4. Comparing department budget vs total project budget

Assumed existing tables:
- employees(emp_id, name, department_id)
- departments(department_id, department_name, budget)

---

# 1. Create Projects Table

```sql
CREATE TABLE projects (
    project_id INT PRIMARY KEY,
    project_name VARCHAR(100),
    lead_emp_id INT,
    budget DECIMAL(12,2),
    start_date DATE,
    end_date DATE
);
```

---

# 2. Insert Sample Data

```sql
INSERT INTO projects VALUES
(1, 'AI Analytics Platform', 101, 150000, '2025-01-10', '2025-06-30'),
(2, 'Cloud Migration', 102, 120000, '2025-02-01', '2025-09-01'),
(3, 'Customer Insights Engine', 103, 80000, '2025-03-15', '2025-07-30'),
(4, 'Fraud Detection System', 104, 200000, '2025-04-01', '2025-12-15'),
(5, 'Mobile Data Pipeline', 105, 60000, '2025-05-10', '2025-10-20');
```

---

# Sample Reference Data

## Employees

| emp_id | name | department_id |
|------|------|------|
| 101 | Alice | 1 |
| 102 | Bob | 1 |
| 103 | Charlie | 2 |
| 104 | David | 2 |
| 105 | Eva | 3 |

## Departments

| department_id | department_name | budget |
|---|---|---|
| 1 | Engineering | 250000 |
| 2 | Data Science | 220000 |
| 3 | IT Operations | 90000 |

---

# Query 1 — 3 Table JOIN

Show employee name, department budget, and project budget.

```sql
SELECT 
    e.name AS employee_name,
    d.department_name,
    d.budget AS department_budget,
    p.project_name,
    p.budget AS project_budget
FROM projects p
JOIN employees e
    ON p.lead_emp_id = e.emp_id
JOIN departments d
    ON e.department_id = d.department_id;
```

## Output

| employee_name | department_name | department_budget | project_name | project_budget |
|---|---|---|---|---|
| Alice | Engineering | 250000 | AI Analytics Platform | 150000 |
| Bob | Engineering | 250000 | Cloud Migration | 120000 |
| Charlie | Data Science | 220000 | Customer Insights Engine | 80000 |
| David | Data Science | 220000 | Fraud Detection System | 200000 |
| Eva | IT Operations | 90000 | Mobile Data Pipeline | 60000 |

---

# Query 2 — Departments Where Total Project Budget Exceeds Department Budget

```sql
SELECT 
    d.department_name,
    d.budget AS department_budget,
    SUM(p.budget) AS total_project_budget
FROM departments d
JOIN employees e
    ON d.department_id = e.department_id
JOIN projects p
    ON e.emp_id = p.lead_emp_id
GROUP BY d.department_name, d.budget
HAVING SUM(p.budget) > d.budget;
```

## Output

| department_name | department_budget | total_project_budget |
|---|---|---|
| Engineering | 250000 | 270000 |
| Data Science | 220000 | 280000 |

Explanation:
Engineering: 150000 + 120000 = 270000 > 250000  
Data Science: 80000 + 200000 = 280000 > 220000  
IT Operations: 60000 ≤ 90000 (not included)

---

# SQL Concepts Demonstrated

- Table creation
- INSERT statements
- Multi-table JOIN
- Aggregation (SUM)
- GROUP BY
- HAVING filtering
