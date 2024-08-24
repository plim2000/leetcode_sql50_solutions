# leetcode_sql50_solutions
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
