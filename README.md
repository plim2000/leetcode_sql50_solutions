# LeetCode SQL50 Solutions
Documenting my [LeetCode SQL50](https://leetcode.com/studyplan/top-sql-50/) solutions.

## Sections
1. [Select](#select)
2. [Basic Joins](#basic-joins)
3. [Basic Aggregate Functions](#basic-aggregate-functions)
4. [Sorting and Grouping](#sorting-and-grouping)
5. [Advanced Select and Joins](#advanced-select-and-joins)
6. [Subqueries](#subqueries)
7. [Advanced String Functions/Regex/Clause](#advanced-string-functions-regex-clause)


## Select
### Recyclable and Low Fat Products
```sql
SELECT product_id
FROM Products
WHERE low_fats = 'Y' AND recyclable = 'Y'
```
### Find Customer Referee
```sql
SELECT name
FROM Customer
WHERE referee_id != 2 OR referee_id is NULL
```
### Big Countries
```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000
```
### Article Views I
```sql
SELECT DISTINCT author_id AS id
FROM Views
WHERE author_id = viewer_id
ORDER BY author_id
```
### Invalid Tweets
```sql
SELECT tweet_id
FROM Tweets
WHERE LENGTH(content) > 15
```

## Basic Joins
### Replace Employee ID With The Unique Identifier
```sql
SELECT EmployeeUNI.unique_id, Employees.name
FROM Employees LEFT JOIN EmployeeUNI ON (Employees.id = EmployeeUNI.id)
```
### Product Sales Analysis I
```sql
SELECT Product.product_name, Sales.year, Sales.price
FROM Sales JOIN Product ON (Sales.product_id = Product.product_id)
```
### Customer Who Visited but Did Not Make Any Transactions
```sql
SELECT v.customer_id, COUNT(*) count_no_trans
FROM Visits v LEFT JOIN Transactions t ON v.visit_id = t.visit_id
WHERE t.transaction_id IS NULL
GROUP BY v.customer_id
```
### Rising Temperature
```sql
SELECT w1.id
FROM Weather w1 JOIN Weather w2 ON w1.recordDate = DATE_ADD(w2.recordDate, Interval 1 DAY)
WHERE w1.temperature > w2.temperature
```
### Average Time of Process per Machine
```sql
SELECT machine_id,
       ROUND(AVG(p.end_time - p.start_time), 3) processing_time
FROM   (SELECT machine_id, 
               process_id,
               MAX(CASE WHEN activity_type = 'start' THEN timestamp END) start_time,
               MAX(CASE WHEN activity_type = 'end' THEN timestamp END) end_time
        FROM Activity
        GROUP BY machine_id, process_id) p
GROUP BY machine_id
```
### Employee Bonus
```sql
SELECT e.name, b.bonus
FROM Employee e LEFT JOIN Bonus b ON (e.empId = b.empId)
WHERE COALESCE(b.bonus, 0) < 1000
```
### Students and Examinations
```sql
WITH 
    StudentSubject AS (
    SELECT st.student_id, st.student_name, su.subject_name
    FROM Students st
    CROSS JOIN Subjects su),

    AttendedExams AS (
    SELECT e.student_id, e.subject_name, count(*) exam_count
    FROM Examinations e
    GROUP BY e.student_id, e.subject_name)

SELECT ss.student_id, ss.student_name, ss.subject_name, COALESCE(ae.exam_count, 0) attended_exams
FROM StudentSubject ss LEFT JOIN AttendedExams ae ON (ss.student_id = ae.student_id)
                                                  AND (ss.subject_name = ae.subject_name)
ORDER BY ss.student_id, ss.subject_name
```
### Managers with at Least 5 Direct Reports
```sql
WITH
    manager_count_table AS (
    SELECT managerid, COUNT(*) manager_count
    FROM employee
    GROUP BY managerid)

SELECT name
FROM employee e LEFT JOIN manager_count_table m ON (e.id = m.managerid)
WHERE m.manager_count >= 5
```
### Confirmation Rate
```sql
WITH
    total_count_table AS (
    SELECT user_id, count(*) total_count
    FROM confirmations
    GROUP BY user_id),

    confirm_count_table AS (
    SELECT user_id, SUM(CASE WHEN action = 'confirmed' THEN 1 ELSE 0 end) confirm_count
    FROM confirmations
    GROUP BY user_id)

SELECT s.user_id, COALESCE(ROUND((c.confirm_count / t.total_count), 2), 0) confirmation_rate
FROM signups s LEFT JOIN total_count_table t USING(user_id)
               LEFT JOIN confirm_count_table c USING(user_id)
```
