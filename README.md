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
### 1757. Recyclable and Low Fat Products
```sql
SELECT product_id
FROM Products
WHERE low_fats = 'Y' AND recyclable = 'Y'
```
### 584. Find Customer Referee
```sql
SELECT name
FROM Customer
WHERE referee_id != 2 OR referee_id is NULL
```
### 595. Big Countries
```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000
```
### 1148. Article Views I
```sql
SELECT DISTINCT author_id AS id
FROM Views
WHERE author_id = viewer_id
ORDER BY author_id
```
### 1683. Invalid Tweets
```sql
SELECT tweet_id
FROM Tweets
WHERE LENGTH(content) > 15
```

## Basic Joins
### 1378. Replace Employee ID With The Unique Identifier
```sql
SELECT EmployeeUNI.unique_id, Employees.name
FROM Employees LEFT JOIN EmployeeUNI ON (Employees.id = EmployeeUNI.id)
```
### 1068. Product Sales Analysis I
```sql
SELECT Product.product_name, Sales.year, Sales.price
FROM Sales JOIN Product ON (Sales.product_id = Product.product_id)
```
### 1581. Customer Who Visited but Did Not Make Any Transactions
```sql
SELECT v.customer_id, COUNT(*) count_no_trans
FROM Visits v LEFT JOIN Transactions t ON v.visit_id = t.visit_id
WHERE t.transaction_id IS NULL
GROUP BY v.customer_id
```
### 197. Rising Temperature
```sql
SELECT w1.id
FROM Weather w1 JOIN Weather w2 ON w1.recordDate = DATE_ADD(w2.recordDate, Interval 1 DAY)
WHERE w1.temperature > w2.temperature
```
### 1661. Average Time of Process per Machine
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
### 577. Employee Bonus
```sql
SELECT e.name, b.bonus
FROM Employee e LEFT JOIN Bonus b ON (e.empId = b.empId)
WHERE COALESCE(b.bonus, 0) < 1000
```
### 1280. Students and Examinations
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
### 570. Managers with at Least 5 Direct Reports
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
### 1934. Confirmation Rate
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
### 620. Not Boring Movies
```sql
SELECT *
FROM Cinema
WHERE id % 2 != 0 AND description != 'boring'
ORDER BY rating DESC
```
### 1251. Average Selling Price
```sql
SELECT p.product_id, coalesce(round(sum(p.price*u.units)/sum(u.units),2),0) average_price
FROM Prices p LEFT JOIN Unitssold u ON (p.product_id = u.product_id)
                                    AND (u.purchase_date BETWEEN p.start_date AND p.end_date)
GROUP BY p.product_id
```
### 1075. Project Employees I
```sql
SELECT p.project_id, ROUND(SUM(e.experience_years)/COUNT(*),2) average_years
FROM Project p JOIN Employee e ON (p.employee_id = e.employee_id)
GROUP BY p.project_id
```
### 1633. Percentage of Users Attended a Contest
```sql
SELECT r.contest_id, ROUND(COUNT(DISTINCT r.user_id)/(SELECT COUNT(*) FROM users)*100, 2) percentage
FROM Users u RIGHT JOIN Register r ON (u.user_id = r.user_id)
GROUP BY r.contest_id
ORDER BY percentage DESC, contest_id
```
### 1211. Queries Quality and Percentage
```sql
SELECT query_name, 
    ROUND(SUM(rating/position) / COUNT(*), 2) quality,
    ROUND(SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 end) * 100 / COUNT(*), 2) poor_query_percentage
FROM Queries
WHERE query_name IS NOT NULL
GROUP BY query_name
```
### 1193. Monthly Transactions I
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
### 1174. Immediate Food Delivery II
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
### 550. Game Play Analysis IV
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
### 2356. Number of Unique Subjects Taught by Each Teacher
```sql
SELECT teacher_id, COUNT(DISTINCT subject_id) cnt
FROM Teacher
GROUP BY teacher_id
```
### 1141. User Activity for the Past 30 Days I
```sql
SELECT activity_date day, count(DISTINCT user_id) active_users
FROM Activity
WHERE activity_date BETWEEN DATE_SUB('2019-07-27', INTERVAL 29 DAY) AND '2019-07-27'
GROUP BY activity_date
```
### 1070. Product Sales Analysis III
```sql
WITH 
    Min_Year_Table AS (
    SELECT product_id, MIN(year) min_year
    FROM Sales
    GROUP BY product_id)

SELECT s.product_id, myt.min_year first_year, s.quantity, s.price
FROM Sales s JOIN Min_Year_Table myt USING(product_id)
WHERE s.year = myt.min_year
```
### 596. Classes More Than 5 Students
```sql
SELECT class
FROM Courses
GROUP BY class
HAVING count(student) >= 5
```
### 1729. Find Followers Count
```sql
SELECT user_id, count(*) followers_count
FROM Followers
GROUP BY user_id
ORDER BY user_id
```
### 619. Biggest Single Number
```sql
WITH
    num_count_table AS (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(num) = 1)

SELECT max(num) num
FROM num_count_table
```
### 1045. Customers Who Bought All Products
```sql
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(DISTINCT product_key) FROM Product)
```

## Advanced Select and Joins
### 1731. The Number of Employees Which Report to Each Employee
```sql
SELECT e1.employee_id, e1.name, COUNT(e2.reports_to) reports_count, ROUND(AVG(e2.age),0) average_age
FROM employees e1 JOIN employees e2 ON e1.employee_id = e2.reports_to
GROUP BY e1.employee_id, e1.name
ORDER BY e1.employee_id
```
### 1789. Primary Department for Each Employee
```sql
SELECT employee_id, department_id
FROM (
    SELECT *, COUNT(*) OVER (PARTITION BY employee_id) employee_count
    FROM Employee
    ) Employee_Count_Table
WHERE employee_count = 1 OR primary_flag = 'Y'
```
### 610. Triangle Judgement
```sql
SELECT *,
    CASE WHEN (x < y + z) AND (y < x + z) AND (z < x + y) THEN 'Yes' ELSE 'No' END triangle
FROM Triangle
```
### 180. Consecutive Numbers
```sql
WITH 
    Adjacent_Num_Table AS (
    SELECT *,
        LAG(id, 1) OVER (ORDER BY id) prior_id,
        LEAD(id, 1) OVER (ORDER BY id) next_id,
        LAG(num, 1) OVER (ORDER BY id) prior_num,
        LEAD(num, 1) OVER (ORDER BY id) next_num
    FROM Logs),

    Consecutive_Table AS (
    SELECT num,
        SUM(CASE WHEN (id - 1 = prior_id)
            AND (id + 1 = next_id)
            AND (num = prior_num)
            AND (num = next_num) THEN 1 ELSE 0 END) consecutive_count
    FROM Adjacent_Num_Table
    GROUP BY num)

SELECT num ConsecutiveNums
FROM Consecutive_Table
WHERE consecutive_count >= 1
```
### 1164. Product Price at a Given Date
```sql
WITH
    Price_Rank_Table AS (
    SELECT *,
        RANK() OVER (PARTITION BY product_id ORDER BY change_date DESC) price_rank
    FROM Products
    WHERE change_date <= '2019-08-16'),
   
    Product_Id_Table AS(
    SELECT DISTINCT product_id
    FROM Products)

SELECT product_id,
    CASE WHEN new_price IS NULL THEN 10 ELSE new_price END price
FROM  Product_Id_Table LEFT JOIN Price_Rank_Table USING(product_id)
WHERE price_rank = 1 OR price_rank IS NULL
```
### 1204. Last Person to Fit in the Bus
```sql
SELECT person_name
FROM (
    SELECT *,
    SUM(weight) OVER (ORDER BY turn) running_weight
    FROM Queue
    ) running_weight_table
WHERE running_weight <= 1000
ORDER BY turn DESC
LIMIT 1
```
### 1907. Count Salary Categories
```sql
WITH
    Categorized_Table AS (
    SELECT *,
        CASE WHEN income < 20000 THEN 'Low Salary' WHEN income BETWEEN 20000 AND 50000 THEN 'Average Salary' ELSE 'High Salary' END category
    FROM Accounts),

    Category_Id_Table AS (
    SELECT 'Low Salary' category
    UNION
    SELECT 'Average Salary'
    UNION
    SELECT 'High Salary')

SELECT cit.category, COUNT(ct.category) accounts_count
FROM Category_Id_Table cit LEFT JOIN Categorized_Table ct USING(category)
GROUP BY cit.category
```

## Subqueries
### 1978. Employees Whose Manager Left the Company
```sql
SELECT employee_id
FROM Employees
WHERE manager_id NOT IN (SELECT DISTINCT employee_ID FROM Employees)
    AND salary < 30000
ORDER BY employee_id
```
### 626. Exchange Seats
```sql
WITH 
    Aggregate_Table AS (
    SELECT MAX(id) max_id
    FROM Seat),

    New_Id_Table AS (
    SELECT *, 
        MOD(id, 2) even_or_odd,
        CASE WHEN MOD(id, 2) = 1 THEN id + 1 ELSE id - 1 END new_id
    FROM Seat)
    
Select 
    CASE WHEN (nit.id = at.max_id AND MOD(nit.id,2) = 1) THEN nit.id ELSE nit.new_id END id,
    student 
FROM Aggregate_Table at CROSS JOIN New_Id_Table nit
ORDER BY id
```
### 1341. Movie Rating
```sql
WITH
    Highest_Reviewer AS (
    SELECT u.user_id, u.name, COUNT(*) user_count
    FROM Users u JOIN MovieRating mr USING (user_id)
    GROUP BY u.user_id
    ORDER BY user_count DESC, u.name
    LIMIT 1),

    Average_Movie_Review AS (
    SELECT m.title, AVG(mr.rating) avg_rating
    FROM MovieRating mr JOIN Movies m USING (movie_id)
    WHERE DATE_FORMAT(created_at, '%Y-%m') = '2020-02'
    GROUP BY m.title
    ORDER BY avg_rating DESC, m.title
    LIMIT 1)

SELECT (SELECT name FROM Highest_Reviewer) results
UNION ALL
SELECT (SELECT title FROM Average_movie_Review)

```
### 1321. Restaurant Growth
```sql
WITH 
    Smallest_Date_Table AS (
    SELECT MIN(visited_on) smallest_date
    FROM Customer
    ORDER BY visited_on
    LIMIT 1),

    Consolidate_Dates_Table AS (
    SELECT visited_on, SUM(amount) amount
    FROM Customer
    GROUP BY visited_on),
    
    Prior_Amounts_Table AS(
    SELECT *,
        SUM(cdt.amount) OVER (ORDER BY cdt.visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) last_7_days_amount
    FROM Consolidate_Dates_Table cdt CROSS JOIN Smallest_Date_Table sdt)

SELECT visited_on, last_7_days_amount AS amount, ROUND(last_7_days_amount / 7, 2) AS average_amount
FROM Prior_Amounts_Table
WHERE visited_on >= smallest_date + 6
```
