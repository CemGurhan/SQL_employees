# Task - Employee Records

The major corporation BusinessCorp&#8482; wants to do some analysis of varioius metrics around its employees and have contracted the job out to you. They have provided you with two SQL files, you will need to write the queries which will extract the data they are looking for.

## Setup

- Create a database to run the files against
- Use the `psql -d database -f file` command to run the SQL files
- Before starting to write the queries take some time to look at the data and figure out the relationships between the tables - maybe even draw an ERD to help.

## Tasks

### CTEs

1) Calculate the average salary of all employees

```sql
SELECT 
	AVG(salary) AS employees_average_salary 
FROM 
	employees; 
```

2) Calculate the average salary of the employees in each team (hint: you'll need to `JOIN` and `GROUP BY` here)

```sql
SELECT 
	departments.name AS department_name,
	AVG(employees.salary) AS team_average_salary
FROM 
	employees
INNER JOIN
	departments
ON 
	departments.id = employees.department_id
GROUP BY
	departments.name
;
```

3) Using a CTE find the ratio of each employees salary to their team average, eg. an employee earning £55000 in a team where the average is £50000 has a ratio of 1.1

```sql
WITH team_average_salary (department_id,department_name,team_average_salary) AS(
SELECT 
	departments.id AS department_id,
	departments.name AS department_name,
	AVG(employees.salary) AS team_average_salary
FROM 
	employees
INNER JOIN
	departments
ON 
	departments.id = employees.department_id
GROUP BY
	departments.id)

SELECT 
	e.first_name,
	e.last_name,
	t.department_name,
	e.salary/t.team_average_salary AS ratio
	
FROM 
	team_average_salary AS t
INNER JOIN
	employees AS e
ON
	e.department_id = t.department_id
ORDER BY
	ratio desc;
```

4) Find the employee with the highest ratio in Argentina

```sql
WITH team_average_salary (department_id,department_name,team_average_salary) AS(
SELECT 
	departments.id AS department_id,
	departments.name AS department_name,
	AVG(employees.salary) AS team_average_salary
FROM 
	employees
INNER JOIN
	departments
ON 
	departments.id = employees.department_id
GROUP BY
	departments.id)

SELECT 
	e.first_name,
	MAX(e.salary/t.team_average_salary) AS ratio,
	e.country
FROM 
	team_average_salary AS t
INNER JOIN
	employees AS e
ON
	e.department_id = t.department_id
WHERE
	e.country='Argentina'
GROUP BY
	e.first_name,
	
	e.country
ORDER BY
	ratio DESC LIMIT 1;
```

5) **Extension:** Add a second CTE calculating the average salary for each country and add a column showing the difference between each employee's salary and their country average

```sql
WITH average_salary_for_country (id,country,average) AS(
SELECT
	employees.id, 
	employees.country AS country,
	AVG(employees.salary) AS average
FROM
	employees
GROUP BY
	employees.id,
	employees.country)
	
	

SELECT 
	e.first_name,
	e.last_name,
	(a.average - e.salary) AS Difference_from_countries_average
	
FROM 
	average_salary_for_country AS a
INNER JOIN
	employees as e
ON
	e.department_id	= a.id
```

---

### Window Functions

1) Find the running total of salary costs as the business has grown and hired more people

```sql
SELECT 
	employees.start_date AS date,
	SUM(employees.salary) OVER (ORDER BY employees.start_date) AS salary_costs
FROM 
	employees;
```

2) Determine if any employees started on the same day (hint: some sort of ranking may be useful here) !

```sql
<!--Copy solution here-->
```

3) Find how many employees there are from each country

```sql
SELECT
	DISTINCT employees.country, 
	COUNT (*) OVER(PARTITION BY employees.country)
FROM 
	employees;
```

4) Show how the average salary cost for each department has changed as the number of employees has increased 

```sql
SELECT
	departments.name AS department_name,
	AVG(employees.salary) OVER (PARTITION BY departments.name ORDER BY employees.start_date) AS department_average_salary,
	COUNT(*) OVER(PARTITION BY departments.name ORDER BY employees.start_date) AS number_of_employees	
FROM
	employees
INNER JOIN
	departments
ON departments.id = employees.department_id;

```

5) **Extension:** Research the `EXTRACT` function and use it in conjunction with `PARTITION` and `COUNT` to show how many employees started working for BusinessCorp&#8482; each year. If you're feeling adventurous you could further partition by month...

```sql
SELECT 
	DISTINCT (COUNT(*) OVER (PARTITION BY EXTRACT(YEAR FROM employees.start_date))) AS number_of_employees, 
	EXTRACT(YEAR FROM employees.start_date) AS start_year,
	EXTRACT(MONTH FROM employees.start_date) AS start_month	
FROM
	employees
INNER JOIN
	departments
ON 
	departments.id = employees.department_id;

```

---

### Combining the two

1) Find the maximum and minimum salaries

```sql
SELECT
	MAX(employees.salary) as max_salary,
	MIN(employees.salary) as min_salary
FROM
	employees;
```

2) Find the difference between the maximum and minimum salaries and each employee's own salary

```sql
<!--Copy solution here-->
```

3) Order the employees by start date. Research how to calculate the **median** salary value and the **standard deviation** in salary values and show how these change as more employees join the company

```sql
<!--Copy solution here-->
```

4) Limit this query to only Research & Development team members and show a rolling value for only the 5 most recent employees.

```sql
<!--Copy solution here-->
```

