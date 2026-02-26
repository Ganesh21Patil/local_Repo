
# Interview-Ready Answers for All SQL Questions

---

## Category 1: Basic Queries

**1. What is the difference between WHERE and HAVING?**

WHERE filters rows before grouping happens. HAVING filters after grouping. So WHERE works on individual rows and HAVING works on aggregated results. For example if you want departments where average salary is more than 50,000 you cannot use WHERE because average is calculated after grouping — you must use HAVING. Simple rule: if your condition has an aggregate function like SUM, COUNT, AVG — use HAVING, otherwise use WHERE.

---

**2. What is the difference between DELETE, DROP, and TRUNCATE?**

DELETE removes specific rows from a table based on a condition and can be rolled back because it is a DML operation. It is slow on large tables because it logs each row deletion. TRUNCATE removes all rows from a table at once, is much faster, and cannot be rolled back in most databases because it is a DDL operation. DROP completely removes the table structure itself along with all data — the table no longer exists after DROP. In interviews remember: DELETE is selective, TRUNCATE is full wipe but table stays, DROP removes everything including the table.

---

**3. What are different types of JOINs? Explain each.**

INNER JOIN returns only the rows that have matching values in both tables. LEFT JOIN returns all rows from the left table and matching rows from the right — if no match, right side shows NULL. RIGHT JOIN is opposite of LEFT — all rows from right table and matching from left. FULL OUTER JOIN returns all rows from both tables — where there is no match it shows NULL on the missing side. SELF JOIN is when a table joins with itself, commonly used for hierarchical data like finding employee and their manager from same table. CROSS JOIN returns cartesian product — every row from first table combined with every row from second table.

---

**4. What is the difference between UNION and UNION ALL?**

Both combine results of two SELECT queries but UNION removes duplicate rows from the final result while UNION ALL keeps all rows including duplicates. UNION ALL is faster because it does not spend time checking and removing duplicates. Use UNION when you want unique records, use UNION ALL when duplicates are acceptable or when you know data is already unique and want better performance.

---

**5. What is PRIMARY KEY vs FOREIGN KEY?**

PRIMARY KEY uniquely identifies each row in a table. It cannot have NULL values and a table can have only one primary key. FOREIGN KEY is a column in one table that refers to the PRIMARY KEY of another table. It creates a relationship between two tables and ensures referential integrity — meaning you cannot insert a value in the foreign key column that does not exist in the referenced table. For example employee table has department_id as foreign key which references department table's primary key.

---

**6. What is the difference between DISTINCT and GROUP BY?**

Both can be used to get unique values but they work differently. DISTINCT simply removes duplicate rows from the result. GROUP BY groups rows together so you can apply aggregate functions like COUNT, SUM, AVG on each group. If you just want unique values use DISTINCT. If you want unique values along with some calculation on each group use GROUP BY. In terms of performance GROUP BY is sometimes faster because databases optimize it differently.

---

**7. What are aggregate functions? Name them.**

Aggregate functions perform a calculation on a set of rows and return a single value. Main ones are COUNT which counts number of rows, SUM which adds up values, AVG which calculates average, MIN which returns minimum value, and MAX which returns maximum value. These are always used with GROUP BY when you want results per group, or without GROUP BY when you want a single result for the whole table. They ignore NULL values except COUNT(*) which counts all rows including NULLs.

---

**8. What is a NULL value? How do you handle it?**

NULL means absence of a value — it is not zero, not empty string, it is simply unknown or missing. You cannot compare NULL using equal sign — WHERE column = NULL will never work. You must use IS NULL or IS NOT NULL. To handle NULLs in calculations use COALESCE function which returns the first non-null value from a list — for example COALESCE(salary, 0) will return 0 if salary is NULL. You can also use ISNULL or NULLIF depending on the database. In joins NULL values never match each other which is important to understand.

---

## Category 2: Joins

**9. Write a query to get employees who do NOT have a department**

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;
```
Here we use LEFT JOIN to get all employees including those without a department. Then we filter WHERE department id IS NULL — these are the employees with no matching department. This is a very common pattern called anti-join.

---

**10. What is a SELF JOIN? Give example.**

A SELF JOIN is when a table is joined with itself. This is useful when data has a hierarchical relationship within the same table. Classic example is an employee table where each employee has a manager_id that refers to another employee's id in the same table.

```sql
SELECT e.name as employee, m.name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```
This gives you each employee paired with their manager's name — all from a single table.

---

**11. What is a CROSS JOIN?**

CROSS JOIN produces a cartesian product — it combines every row from the first table with every row from the second table. If table A has 5 rows and table B has 4 rows, CROSS JOIN gives 20 rows. It has no ON condition. It is rarely used in practice but can be useful when you need all possible combinations — like generating all possible combinations of colors and sizes for a product.

---

**12. Write a query to find employees and their manager names using SELF JOIN**
Already covered in question 10. The key point to say in interview is: same table is used twice with different aliases — one alias represents the employee, another represents the manager.

---

**13. Difference between LEFT JOIN and LEFT OUTER JOIN?**

They are exactly the same thing. OUTER is optional keyword. LEFT JOIN and LEFT OUTER JOIN produce identical results. Same applies to RIGHT JOIN and RIGHT OUTER JOIN, and FULL JOIN and FULL OUTER JOIN. This is often asked as a trick question to test if candidates know this — confidently say they are the same.

---

## Category 3: Window Functions

**14. What is ROW_NUMBER vs RANK vs DENSE_RANK?**

All three assign numbers to rows but handle ties differently. ROW_NUMBER assigns a unique number to every row — even if two rows have same value they get different numbers. RANK assigns same number to tied rows but skips the next number — so if two rows are rank 1, next row is rank 3, not 2. DENSE_RANK also assigns same number to tied rows but does NOT skip — so if two rows are rank 1, next row is rank 2. In interviews remember: DENSE_RANK never has gaps, RANK has gaps, ROW_NUMBER never repeats.

---

**15. Write a query to find top 3 salaries in each department**

```sql
SELECT * FROM (
  SELECT name, department, salary,
  DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rnk
  FROM employees
) t
WHERE rnk <= 3;
```
We use DENSE_RANK so that if two people have same salary they both appear and we don't miss the actual 3rd highest salary.

---

**16. What is LAG and LEAD? Give a real use case.**

LAG accesses data from a previous row in the result set without using a self join. LEAD accesses data from a next row. Real use case: in sales data you want to compare each month's revenue with the previous month to calculate growth percentage. You use LAG to get previous month's value right alongside current month's value in the same row. Another use case is finding the difference between consecutive transactions for a customer.

```sql
SELECT month, revenue,
LAG(revenue) OVER (ORDER BY month) as prev_month_revenue,
revenue - LAG(revenue) OVER (ORDER BY month) as growth
FROM sales;
```

---

**17. What is PARTITION BY? How is it different from GROUP BY?**

PARTITION BY is used with window functions to divide data into groups for calculation but it does NOT reduce the number of rows in output — every row is still present. GROUP BY collapses rows into one row per group. So if you GROUP BY department you get one row per department. If you PARTITION BY department in a window function each employee row is still visible but calculations happen within their department group. Use PARTITION BY when you want per-group calculations but still want to see individual row details.

---

**18. Write a query to calculate running total using window function**

```sql
SELECT name, salary,
SUM(salary) OVER (ORDER BY id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM employees;
```
This gives a cumulative sum — each row shows total of all salaries from first row up to current row. This is commonly used in financial reports to show running balance.

---

## Category 4: Classic Tricky Queries

**19. Find second highest salary**

```sql
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```
Inner query finds the highest salary. Outer query finds the maximum salary that is less than the highest — which is the second highest. Always mention you can extend this logic to find Nth highest.

---

**20. Find duplicate records in a table**

```sql
SELECT name, COUNT(*) as count
FROM employees
GROUP BY name
HAVING COUNT(*) > 1;
```
Group by the column you suspect has duplicates, count occurrences, then filter where count is more than 1. These are your duplicates.

---

**21. Delete duplicate rows but keep one**

```sql
DELETE FROM employees WHERE id NOT IN (
  SELECT MIN(id) FROM employees GROUP BY name
);
```
Inner query finds the minimum id for each name — this is the row we want to keep. We delete all rows whose id is NOT in that list — meaning all duplicates are removed while the first occurrence stays.

---

**22. Find employees who joined in the last 30 days**

```sql
SELECT * FROM employees
WHERE joining_date >= DATEADD(DAY, -30, GETDATE());
```
GETDATE() gives today's date. DATEADD subtracts 30 days from today. In MySQL you would use DATE_SUB(NOW(), INTERVAL 30 DAY) instead.

---

**23. Find departments where average salary is more than 50,000**

```sql
SELECT department, AVG(salary) as avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```
Classic GROUP BY with HAVING example. Always use HAVING for conditions on aggregated values.

---

**24. Find employees whose salary is above company average**

```sql
SELECT name, salary FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```
Subquery calculates company-wide average first. Outer query filters employees earning more than that average.

---

**25. Find Nth highest salary**

```sql
SELECT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET (N-1);
```
Replace N with actual number. OFFSET skips N-1 rows. So for 3rd highest, OFFSET 2 skips top 2 and LIMIT 1 takes the next one. In SQL Server you would use FETCH NEXT 1 ROWS ONLY after OFFSET.

---

## Category 5: Subqueries and CTEs

**26. What is a subquery? Types of subqueries?**

A subquery is a query written inside another query. Types are: Single-row subquery which returns one row and is used with operators like =, >, <. Multi-row subquery which returns multiple rows and is used with IN, ANY, ALL. Correlated subquery which references the outer query and runs once for each row of outer query — slower but powerful. Non-correlated subquery runs independently of outer query, executes once, and result is used by outer query. Scalar subquery returns exactly one value and can be used anywhere a single value is expected.

---

**27. What is a CTE? When do you use it?**

CTE stands for Common Table Expression. It is a temporary named result set defined using WITH keyword that exists only for the duration of the query. You use CTE when your query is complex and has multiple steps — CTE makes it readable by breaking it into logical named parts. It is also used for recursive queries like traversing hierarchies. Unlike subqueries, CTEs can be referenced multiple times within the same query which avoids repetition.

---

**28. Difference between correlated and non-correlated subquery?**

Non-correlated subquery is independent — it executes once, returns a result, and that result is used by the outer query. Correlated subquery references a column from the outer query — so it executes once for every row in the outer query making it slower. Example of correlated: finding employees who earn more than the average salary of their own department — inner query needs to know which department each employee belongs to, so it runs row by row.

---

**29. Write the same query using both subquery and CTE**

Find employees earning above department average:

**Subquery version:**
```sql
SELECT name, salary FROM employees e
WHERE salary > (
  SELECT AVG(salary) FROM employees
  WHERE department = e.department
);
```

**CTE version:**
```sql
WITH dept_avg AS (
  SELECT department, AVG(salary) as avg_sal
  FROM employees GROUP BY department
)
SELECT e.name, e.salary
FROM employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_sal;
```
CTE version is more readable, especially when logic is reused.

---

**30. What is a recursive CTE? Give example.**

Recursive CTE is a CTE that references itself. It has two parts — anchor member which is the starting point, and recursive member which keeps joining back to the CTE until a condition stops it. Classic use case is traversing an employee hierarchy from CEO down to all subordinates.

```sql
WITH emp_hierarchy AS (
  SELECT id, name, manager_id, 1 as level
  FROM employees WHERE manager_id IS NULL  -- anchor: top level
  UNION ALL
  SELECT e.id, e.name, e.manager_id, h.level + 1
  FROM employees e
  JOIN emp_hierarchy h ON e.manager_id = h.id  -- recursive part
)
SELECT * FROM emp_hierarchy;
```

---

## Category 6: Performance and Optimization

**31. What is an INDEX? When should you use it?**

An index is a database object that speeds up data retrieval by creating a lookup structure on one or more columns — similar to index at back of a book. Without index, database scans every row. With index it jumps directly to relevant rows. Use index on columns frequently used in WHERE clauses, JOIN conditions, and ORDER BY. Do not index every column — indexes slow down INSERT, UPDATE, DELETE operations because index must also be updated. Best candidates are columns with high cardinality — meaning many unique values.

---

**32. What is the difference between clustered and non-clustered index?**

A table can have only one clustered index because it physically sorts and stores the data rows in the table based on that index key. Primary key is usually the clustered index. Non-clustered index is a separate structure from the data — it contains the index key and a pointer to the actual data row. A table can have multiple non-clustered indexes. Think of clustered index as the actual book pages in order, and non-clustered index as a separate index at back of book pointing to page numbers.

---

**33. How do you optimize a slow running query?**

First I check the execution plan to identify bottlenecks — whether it is doing full table scans or expensive operations. Then I check if appropriate indexes exist on JOIN and WHERE columns. I look for SELECT * and replace with only needed columns. I check if subqueries can be replaced with JOINs or CTEs. I look for functions applied on indexed columns in WHERE clause — this prevents index usage. I check if temporary tables or CTEs can help break down complex logic. I also check statistics are up to date. In my current role I regularly use execution plans to identify missing indexes and it has reduced query times significantly.

---

**34. What is query execution plan? How do you read it?**

Execution plan shows how the database engine executes your query — what operations it performs, in what order, and at what cost. You can see it using EXPLAIN in MySQL or PostgreSQL, or EXPLAIN PLAN in Oracle. Key things to look for are full table scans which are expensive on large tables, nested loop joins which can be slow, and high cost operations. You look at estimated vs actual rows to find where optimizer made wrong assumptions. Reading execution plans regularly helps identify exactly where a query is slow and what fix will help most.

---

**35. What is partitioning in SQL? Why is it used?**

Partitioning divides a large table into smaller manageable pieces called partitions based on a column value — typically date, region, or category. Each partition is stored separately but the table appears as one table to the user. It improves query performance because when you query data for a specific month, database only scans that month's partition instead of entire table. It also helps with data management — you can archive or drop old partitions easily. Very commonly used in data warehouses and large fact tables.

---

**36. What is normalization? Explain 1NF, 2NF, 3NF simply.**

Normalization is the process of organizing database tables to reduce data redundancy and improve data integrity. First Normal Form (1NF) means each column has atomic values — no repeating groups or arrays in a cell. Second Normal Form (2NF) means table is in 1NF and every non-key column is fully dependent on the entire primary key — no partial dependency. Third Normal Form (3NF) means table is in 2NF and no non-key column depends on another non-key column — no transitive dependency. Simply put — each piece of information stored in only one place.

---

**37. When would you denormalize a table?**

Denormalization is intentionally introducing redundancy into a database for performance reasons. We denormalize when read performance is more critical than write performance — typically in data warehouses and reporting systems. Joining many normalized tables for every report query is slow. So we combine them into fewer wider tables even if it means some data is repeated. In OLAP systems and data warehouses denormalization is common practice because analytical queries need to be fast and complex joins are costly at scale.

---

## Category 7: Data Engineering Specific SQL

**38. How do you handle slowly changing dimensions SCD Type 1 and Type 2?**

SCD refers to how you handle changes to dimension data over time. Type 1 simply overwrites the old value with new value — no history is kept. Used when history does not matter, like correcting a typo in a name. Type 2 maintains full history by adding a new row for each change with start and end dates and an is_current flag. Old row gets an end date and is_current set to false. New row gets current date as start date and is_current as true. This way you can see what the value was at any point in time. I have implemented SCD Type 2 in my current role for customer dimension tables.

---

**39. Write a query to do incremental data load**

```sql
INSERT INTO target_table
SELECT * FROM source_table
WHERE updated_at > (SELECT MAX(updated_at) FROM target_table);
```
This loads only records that were created or updated after the last load timestamp. This is much more efficient than full load for large tables. In practice you also need to handle updates to existing records using MERGE or UPSERT logic.

---

**40. How do you identify and handle late arriving data?**

Late arriving data is data that arrives after the expected processing time — for example a transaction that happened yesterday but reached your pipeline today. To identify it you compare event timestamp with processing timestamp. To handle it you can reprocess the affected partition or time window, use watermarking in streaming systems to wait a defined time before finalizing results, or maintain a staging area where late data is held and periodically merged into main tables. In my pipelines I add audit columns like event_time and load_time to track and handle this.

---

**41. What is the difference between OLTP and OLAP?**

OLTP stands for Online Transaction Processing. It handles day-to-day operational transactions — inserts, updates, deletes happening in real time like banking transactions or order placements. Tables are highly normalized, queries are simple, and speed of individual transactions matters. OLAP stands for Online Analytical Processing. It is designed for complex analytical queries on large historical data — like sales reports or trend analysis. Tables are often denormalized like star schema or snowflake schema. Query performance on large aggregations is the priority. As data engineers we typically move data from OLTP systems into OLAP data warehouses for analytics.

---

**42. How do you write upsert logic in SQL?**

Upsert means update if record exists, insert if it does not. In SQL Server and many modern databases you use MERGE statement:

```sql
MERGE target_table AS target
USING source_table AS source
ON target.id = source.id
WHEN MATCHED THEN
  UPDATE SET target.name = source.name, target.salary = source.salary
WHEN NOT MATCHED THEN
  INSERT (id, name, salary) VALUES (source.id, source.name, source.salary);
```

In PostgreSQL you can use INSERT with ON CONFLICT clause:
```sql
INSERT INTO employees (id, name, salary)
VALUES (1, 'John', 60000)
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name, salary = EXCLUDED.salary;
```
Upsert is heavily used in incremental data loads in data engineering pipelines.

---

## Final Tip for Tomorrow

When answering in interview always try to end with **"In my current role I have used this when..."** — connecting theory to real experience makes your answers stand out from other candidates.

**You are well prepared. Go crush it tomorrow! 💪**
