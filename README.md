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
    Manager_Count_Table AS (
    SELECT managerid, COUNT(*) manager_count
    FROM Employee
    GROUP BY managerid)

SELECT name
FROM Employee e LEFT JOIN Manager_Count_Table m ON (e.id = m.managerid)
WHERE m.manager_count >= 5
```
### Confirmation Rate
```sql
WITH
    Total_Count_Table AS (
    SELECT user_id, count(*) total_count
    FROM Confirmations
    GROUP BY user_id),

    Confirm_Count_Table AS (
    SELECT user_id, SUM(CASE WHEN action = 'confirmed' THEN 1 ELSE 0 end) confirm_count
    FROM Confirmations
    GROUP BY user_id)

SELECT s.user_id, COALESCE(ROUND((c.confirm_count / t.total_count), 2), 0) confirmation_rate
FROM Signups s LEFT JOIN Total_Count_Table t USING(user_id)
               LEFT JOIN Confirm_Count_Table c USING(user_id)
```

## Basic Aggregate Functions
### Not Boring Movies
```sql
SELECT *
FROM Cinema
WHERE id % 2 != 0 AND description != 'boring'
ORDER BY rating DESC
```
### Average Selling Price
```sql
SELECT p.product_id, coalesce(round(sum(p.price*u.units)/sum(u.units),2),0) average_price
FROM Prices p LEFT JOIN Unitssold u ON (p.product_id = u.product_id)
                                    AND (u.purchase_date BETWEEN p.start_date AND p.end_date)
GROUP BY p.product_id
```
### Project Employees I
```sql
SELECT p.project_id, ROUND(SUM(e.experience_years)/COUNT(*),2) average_years
FROM Project p JOIN Employee e ON (p.employee_id = e.employee_id)
GROUP BY p.project_id
```
### Percentage of Users Attended a Contest
```sql
SELECT r.contest_id, ROUND(COUNT(DISTINCT r.user_id)/(SELECT COUNT(*) FROM users)*100, 2) percentage
FROM Users u RIGHT JOIN Register r ON (u.user_id = r.user_id)
GROUP BY r.contest_id
ORDER BY percentage DESC, contest_id
```
### Queries Quality and Percentage
```sql
SELECT query_name, 
    ROUND(SUM(rating/position) / COUNT(*), 2) quality,
    ROUND(SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 end) * 100 / COUNT(*), 2) poor_query_percentage
FROM Queries
WHERE query_name IS NOT NULL
GROUP BY query_name
```
### Monthly Transactions I
```sql
SELECT DATE_FORMAT(trans_date, '%Y-%m') month, 
  country, 
  COUNT(*) trans_count, 
  SUM(CASE WHEN state = 'approved' THEN 1 ELSE 0 END) approved_count,
  SUM(amount) trans_total_amount,
  SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END) approved_total_amount
FROM transactions
GROUP BY month, country
```
### Immediate Food Delivery II
```sql
WITH 
    Modified_Delivery AS (
        SELECT *, 
            CASE WHEN order_date = customer_pref_delivery_date THEN 'immediate' ELSE 'scheduled' END
            order_type,
            RANK() OVER (PARTITION BY customer_id ORDER BY order_date) order_rank
        FROM Delivery),
    Order_Counts AS (
        SELECT COUNT(*) first_order,
            SUM(CASE WHEN order_type = 'immediate' THEN 1 ELSE 0 END) immediate_count
        FROM Modified_Delivery
        WHERE order_rank = 1)
    
SELECT round(immediate_count / first_order * 100, 2) immediate_percentage
FROM Order_Counts
```
### Game Play Analysis IV
```sql
WITH 
    First_Login_Table AS (
        SELECT player_id, MIN(event_date) first_login_date
        FROM Activity
        GROUP BY player_id),
    Login_After_First_Table AS (
        SELECT a.player_id, COUNT(*) login_after_first
        FROM Activity a JOIN First_Login_Table ftl USING(player_id)
        WHERE event_date = DATE_ADD(first_login_date, INTERVAL 1 DAY)
        GROUP BY a.player_id)

SELECT ROUND(COUNT(DISTINCT player_id) / (SELECT COUNT(DISTINCT player_id) FROM Activity), 2) fraction
FROM Login_After_First_Table
```

## Sorting and Grouping
### Number of Unique Subjects Taught by Each Teacher
```sql
SELECT teacher_id, COUNT(DISTINCT subject_id) cnt
FROM Teacher
GROUP BY teacher_id
```
### User Activity for the Past 30 Days I
```sql
SELECT activity_date day, count(DISTINCT user_id) active_users
FROM Activity
WHERE activity_date BETWEEN DATE_SUB('2019-07-27', INTERVAL 29 DAY) AND '2019-07-27'
GROUP BY activity_date
```
### Product Sales Analysis III
```sql
WITH Min_Year_Table AS (
    SELECT product_id, MIN(year) min_year
    FROM Sales
    GROUP BY product_id)

SELECT s.product_id, myt.min_year first_year, s.quantity, s.price
FROM Sales s JOIN Min_Year_Table myt USING(product_id)
WHERE s.year = myt.min_year
```
