# PostgreSQL 50 Practice Questions

_Complete LeetCode SQL-50 Study Plan for Local Practice_

## 🚀 Setup Instructions

1. **Install PostgreSQL** (if not already installed):

```bash
# Ubuntu/Debian
sudo apt-get install postgresql postgresql-contrib

# macOS
brew install postgresql

# Start PostgreSQL
sudo service postgresql start  # Linux
brew services start postgresql  # macOS
```

2. **Create Practice Database**:

```bash
# Login to PostgreSQL
sudo -u postgres psql  # Linux
psql postgres  # macOS

# Create database
CREATE DATABASE sql_practice;
\c sql_practice;
```

3. **Run the table creation and sample data scripts below**

---

## 📊 Database Schema Setup

### Drop existing tables (if any)

```sql
DROP TABLE IF EXISTS Examinations CASCADE;
DROP TABLE IF EXISTS OrderDetails CASCADE;
DROP TABLE IF EXISTS Orders CASCADE;
DROP TABLE IF EXISTS Products CASCADE;
DROP TABLE IF EXISTS Customers CASCADE;
DROP TABLE IF EXISTS Customer CASCADE;
DROP TABLE IF EXISTS Employees CASCADE;
DROP TABLE IF EXISTS Employee CASCADE;
DROP TABLE IF EXISTS Students CASCADE;
DROP TABLE IF EXISTS Subjects CASCADE;
DROP TABLE IF EXISTS World CASCADE;
DROP TABLE IF EXISTS Person CASCADE;
DROP TABLE IF EXISTS Visits CASCADE;
DROP TABLE IF EXISTS Transactions CASCADE;
DROP TABLE IF EXISTS Views CASCADE;
DROP TABLE IF EXISTS Tweets CASCADE;
DROP TABLE IF EXISTS EmployeeUNI CASCADE;
DROP TABLE IF EXISTS Sales CASCADE;
DROP TABLE IF EXISTS Product CASCADE;
DROP TABLE IF EXISTS Activity CASCADE;
DROP TABLE IF EXISTS Signups CASCADE;
DROP TABLE IF EXISTS Confirmations CASCADE;
DROP TABLE IF EXISTS Cinema CASCADE;
DROP TABLE IF EXISTS Prices CASCADE;
DROP TABLE IF EXISTS UnitsSold CASCADE;
DROP TABLE IF EXISTS Seat CASCADE;
DROP TABLE IF EXISTS Tree CASCADE;
DROP TABLE IF EXISTS Logs CASCADE;
DROP TABLE IF EXISTS Employee_Table CASCADE;
DROP TABLE IF EXISTS Bonus CASCADE;
DROP TABLE IF EXISTS Triangle CASCADE;
DROP TABLE IF EXISTS MovieRating CASCADE;
DROP TABLE IF EXISTS Movies CASCADE;
DROP TABLE IF EXISTS Users CASCADE;
DROP TABLE IF EXISTS Activities CASCADE;
DROP TABLE IF EXISTS MyNumbers CASCADE;
DROP TABLE IF EXISTS Project CASCADE;
DROP TABLE IF EXISTS Queue CASCADE;
DROP TABLE IF EXISTS Teacher CASCADE;
DROP TABLE IF EXISTS Weather CASCADE;
DROP TABLE IF EXISTS Department CASCADE;
DROP TABLE IF EXISTS SalesPerson CASCADE;
DROP TABLE IF EXISTS Company CASCADE;
DROP TABLE IF EXISTS ActorDirector CASCADE;
DROP TABLE IF EXISTS RequestAccepted CASCADE;
DROP TABLE IF EXISTS Queries CASCADE;
DROP TABLE IF EXISTS Accounts CASCADE;
DROP TABLE IF EXISTS Insurance CASCADE;
```

---

## 🟢 SECTION 1: SELECT (Questions 1-5)

### Question 1: Recyclable and Low Fat Products

**Difficulty:** Easy
**LeetCode:** 1757

```sql
-- Create table
CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    low_fats VARCHAR(1) CHECK (low_fats IN ('Y', 'N')),
    recyclable VARCHAR(1) CHECK (recyclable IN ('Y', 'N'))
);

-- Insert sample data
INSERT INTO Products VALUES
(0, 'Y', 'N'),
(1, 'Y', 'Y'),
(2, 'N', 'Y'),
(3, 'Y', 'Y'),
(4, 'N', 'N');

-- Solution: Find products that are both low fat and recyclable
SELECT product_id
FROM Products
WHERE low_fats = 'Y' AND recyclable = 'Y';
```

### Question 2: Find Customer Referee

**Difficulty:** Easy
**LeetCode:** 584

```sql
-- Create table
CREATE TABLE Customer (
    id INT PRIMARY KEY,
    name VARCHAR(25),
    referee_id INT
);

-- Insert sample data
INSERT INTO Customer VALUES
(1, 'Will', NULL),
(2, 'Jane', NULL),
(3, 'Alex', 2),
(4, 'Bill', NULL),
(5, 'Zack', 1),
(6, 'Mark', 2);

-- Solution: Find customers not referred by customer with id = 2
SELECT name
FROM Customer
WHERE referee_id != 2 OR referee_id IS NULL;
```

### Question 3: Big Countries

**Difficulty:** Easy
**LeetCode:** 595

```sql
-- Create table
CREATE TABLE World (
    name VARCHAR(255) PRIMARY KEY,
    continent VARCHAR(255),
    area INT,
    population INT,
    gdp BIGINT
);

-- Insert sample data
INSERT INTO World VALUES
('Afghanistan', 'Asia', 652230, 25500100, 20343000000),
('Albania', 'Europe', 28748, 2831741, 12960000000),
('Algeria', 'Africa', 2381741, 37100000, 188681000000),
('Andorra', 'Europe', 468, 78115, 3712000000),
('Angola', 'Africa', 1246700, 20609294, 100990000000);

-- Solution: Find countries with area >= 3000000 OR population >= 25000000
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```

### Question 4: Article Views I

**Difficulty:** Easy
**LeetCode:** 1148

```sql
-- Create table
CREATE TABLE Views (
    article_id INT,
    author_id INT,
    viewer_id INT,
    view_date DATE
);

-- Insert sample data
INSERT INTO Views VALUES
(1, 3, 5, '2019-08-01'),
(1, 3, 6, '2019-08-02'),
(2, 7, 7, '2019-08-01'),
(2, 7, 6, '2019-08-02'),
(4, 7, 1, '2019-07-22'),
(3, 4, 4, '2019-07-21'),
(3, 4, 4, '2019-07-21');

-- Solution: Find authors who viewed their own articles
SELECT DISTINCT author_id AS id
FROM Views
WHERE author_id = viewer_id
ORDER BY author_id;
```

### Question 5: Invalid Tweets

**Difficulty:** Easy
**LeetCode:** 1683

```sql
-- Create table
CREATE TABLE Tweets (
    tweet_id INT PRIMARY KEY,
    content VARCHAR(200)
);

-- Insert sample data
INSERT INTO Tweets VALUES
(1, 'Vote for Biden'),
(2, 'Let us make America great again!');

-- Solution: Find tweets with content length > 15 characters
SELECT tweet_id
FROM Tweets
WHERE LENGTH(content) > 15;
```

---

## 🟡 SECTION 2: BASIC JOINS (Questions 6-12)

### Question 6: Replace Employee ID With The Unique Identifier

**Difficulty:** Easy
**LeetCode:** 1378

```sql
-- Create tables
CREATE TABLE Employees (
    id INT PRIMARY KEY,
    name VARCHAR(20)
);

CREATE TABLE EmployeeUNI (
    id INT,
    unique_id INT
);

-- Insert sample data
INSERT INTO Employees VALUES
(1, 'Alice'),
(7, 'Bob'),
(11, 'Meir'),
(90, 'Winston'),
(3, 'Jonathan');

INSERT INTO EmployeeUNI VALUES
(3, 1),
(11, 2),
(90, 3);

-- Solution: LEFT JOIN to get unique_id for each employee
SELECT eu.unique_id, e.name
FROM Employees e
LEFT JOIN EmployeeUNI eu ON e.id = eu.id;
```

### Question 7: Product Sales Analysis I

**Difficulty:** Easy
**LeetCode:** 1068

```sql
-- Create tables
CREATE TABLE Sales (
    sale_id INT,
    product_id INT,
    year INT,
    quantity INT,
    price INT,
    PRIMARY KEY (sale_id, year)
);

CREATE TABLE Product (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(30)
);

-- Insert sample data
INSERT INTO Sales VALUES
(1, 100, 2008, 10, 5000),
(2, 100, 2009, 12, 5000),
(7, 200, 2011, 15, 9000);

INSERT INTO Product VALUES
(100, 'Nokia'),
(200, 'Apple'),
(300, 'Samsung');

-- Solution: JOIN to get product names with sales info
SELECT p.product_name, s.year, s.price
FROM Sales s
LEFT JOIN Product p ON s.product_id = p.product_id;
```

### Question 8: Customer Who Visited but Did Not Make Any Transactions

**Difficulty:** Easy
**LeetCode:** 1581

```sql
-- Create tables
CREATE TABLE Visits (
    visit_id INT PRIMARY KEY,
    customer_id INT
);

CREATE TABLE Transactions (
    transaction_id INT PRIMARY KEY,
    visit_id INT,
    amount INT
);

-- Insert sample data
INSERT INTO Visits VALUES
(1, 23),
(2, 9),
(4, 30),
(5, 54),
(6, 96),
(7, 54),
(8, 54);

INSERT INTO Transactions VALUES
(2, 5, 310),
(3, 5, 300),
(9, 5, 200),
(12, 1, 910),
(13, 2, 970);

-- Solution: Find customers who visited but didn't make transactions
SELECT v.customer_id, COUNT(*) as count_no_trans
FROM Visits v
LEFT JOIN Transactions t ON v.visit_id = t.visit_id
WHERE t.transaction_id IS NULL
GROUP BY v.customer_id;
```

### Question 9: Rising Temperature

**Difficulty:** Easy
**LeetCode:** 197

```sql
-- Create table
CREATE TABLE Weather (
    id INT PRIMARY KEY,
    recordDate DATE,
    temperature INT
);

-- Insert sample data
INSERT INTO Weather VALUES
(1, '2015-01-01', 10),
(2, '2015-01-02', 25),
(3, '2015-01-03', 20),
(4, '2015-01-04', 30);

-- Solution: Find dates with higher temperature than previous day
SELECT w1.id
FROM Weather w1
JOIN Weather w2 ON w1.recordDate = w2.recordDate + INTERVAL '1 day'
WHERE w1.temperature > w2.temperature;
```

### Question 10: Average Time of Process per Machine

**Difficulty:** Easy
**LeetCode:** 1661

```sql
-- Create table
CREATE TABLE Activity (
    machine_id INT,
    process_id INT,
    activity_type VARCHAR(10) CHECK (activity_type IN ('start', 'end')),
    timestamp FLOAT
);

-- Insert sample data
INSERT INTO Activity VALUES
(0, 0, 'start', 0.712),
(0, 0, 'end', 1.520),
(0, 1, 'start', 3.140),
(0, 1, 'end', 4.120),
(1, 0, 'start', 0.550),
(1, 0, 'end', 1.550),
(1, 1, 'start', 0.430),
(1, 1, 'end', 1.420),
(2, 0, 'start', 4.100),
(2, 0, 'end', 4.512);

-- Solution: Calculate average processing time per machine
SELECT
    a1.machine_id,
    ROUND(AVG(a2.timestamp - a1.timestamp)::numeric, 3) as processing_time
FROM Activity a1
JOIN Activity a2
    ON a1.machine_id = a2.machine_id
    AND a1.process_id = a2.process_id
    AND a1.activity_type = 'start'
    AND a2.activity_type = 'end'
GROUP BY a1.machine_id;
```

### Question 11: Employee Bonus

**Difficulty:** Easy
**LeetCode:** 577

```sql
-- Create tables
CREATE TABLE Employee_Table (
    empId INT PRIMARY KEY,
    name VARCHAR(255),
    supervisor INT,
    salary INT
);

CREATE TABLE Bonus (
    empId INT PRIMARY KEY,
    bonus INT
);

-- Insert sample data
INSERT INTO Employee_Table VALUES
(3, 'Brad', NULL, 4000),
(1, 'John', 3, 1000),
(2, 'Dan', 3, 2000),
(4, 'Thomas', 3, 4000);

INSERT INTO Bonus VALUES
(2, 500),
(4, 2000);

-- Solution: Find employees with bonus < 1000
SELECT e.name, b.bonus
FROM Employee_Table e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL;
```

### Question 12: Students and Examinations

**Difficulty:** Easy
**LeetCode:** 1280

```sql
-- Create tables
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(20)
);

CREATE TABLE Subjects (
    subject_name VARCHAR(20) PRIMARY KEY
);

CREATE TABLE Examinations (
    student_id INT,
    subject_name VARCHAR(20)
);

-- Insert sample data
INSERT INTO Students VALUES
(1, 'Alice'),
(2, 'Bob'),
(13, 'John'),
(6, 'Alex');

INSERT INTO Subjects VALUES
('Math'),
('Physics'),
('Programming');

INSERT INTO Examinations VALUES
(1, 'Math'),
(1, 'Physics'),
(1, 'Programming'),
(2, 'Programming'),
(1, 'Physics'),
(1, 'Math'),
(13, 'Math'),
(13, 'Programming'),
(13, 'Physics'),
(2, 'Math'),
(1, 'Math');

-- Solution: Count exams attended by each student for each subject
SELECT
    s.student_id,
    s.student_name,
    sub.subject_name,
    COUNT(e.subject_name) AS attended_exams
FROM Students s
CROSS JOIN Subjects sub
LEFT JOIN Examinations e
    ON s.student_id = e.student_id
    AND sub.subject_name = e.subject_name
GROUP BY s.student_id, s.student_name, sub.subject_name
ORDER BY s.student_id, sub.subject_name;
```

---

## 🔵 SECTION 3: BASIC AGGREGATE FUNCTIONS (Questions 13-19)

### Question 13: Not Boring Movies

**Difficulty:** Easy
**LeetCode:** 620

```sql
-- Create table
CREATE TABLE Cinema (
    id INT PRIMARY KEY,
    movie VARCHAR(255),
    description VARCHAR(255),
    rating FLOAT
);

-- Insert sample data
INSERT INTO Cinema VALUES
(1, 'War', 'great 3D', 8.9),
(2, 'Science', 'fiction', 8.5),
(3, 'irish', 'boring', 6.2),
(4, 'Ice song', 'Fantacy', 8.6),
(5, 'House card', 'Interesting', 9.1);

-- Solution: Find movies with odd ID and description != 'boring'
SELECT *
FROM Cinema
WHERE id % 2 = 1 AND description != 'boring'
ORDER BY rating DESC;
```

### Question 14: Average Selling Price

**Difficulty:** Easy
**LeetCode:** 1251

```sql
-- Create tables
CREATE TABLE Prices (
    product_id INT,
    start_date DATE,
    end_date DATE,
    price INT,
    PRIMARY KEY (product_id, start_date, end_date)
);

CREATE TABLE UnitsSold (
    product_id INT,
    purchase_date DATE,
    units INT
);

-- Insert sample data
INSERT INTO Prices VALUES
(1, '2019-02-17', '2019-02-28', 5),
(1, '2019-03-01', '2019-03-22', 20),
(2, '2019-02-01', '2019-02-20', 15),
(2, '2019-02-21', '2019-03-31', 30);

INSERT INTO UnitsSold VALUES
(1, '2019-02-25', 100),
(1, '2019-03-01', 15),
(2, '2019-02-10', 200),
(2, '2019-03-22', 30);

-- Solution: Calculate average selling price per product
SELECT
    p.product_id,
    ROUND(SUM(p.price * u.units)::numeric / SUM(u.units), 2) AS average_price
FROM Prices p
LEFT JOIN UnitsSold u
    ON p.product_id = u.product_id
    AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY p.product_id;
```

### Question 15: Project Employees I

**Difficulty:** Easy
**LeetCode:** 1075

```sql
-- Create tables
CREATE TABLE Project (
    project_id INT,
    employee_id INT,
    PRIMARY KEY (project_id, employee_id)
);

CREATE TABLE Employee (
    employee_id INT PRIMARY KEY,
    name VARCHAR(50),
    experience_years INT
);

-- Insert sample data
INSERT INTO Project VALUES
(1, 1),
(1, 2),
(1, 3),
(2, 1),
(2, 4);

INSERT INTO Employee VALUES
(1, 'Khaled', 3),
(2, 'Ali', 2),
(3, 'John', 1),
(4, 'Doe', 2);

-- Solution: Calculate average experience years per project
SELECT
    p.project_id,
    ROUND(AVG(e.experience_years)::numeric, 2) AS average_years
FROM Project p
LEFT JOIN Employee e ON p.employee_id = e.employee_id
GROUP BY p.project_id;
```

### Question 16: Percentage of Users Attended a Contest

**Difficulty:** Easy
**LeetCode:** 1633

```sql
-- Create tables
CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    user_name VARCHAR(50)
);

CREATE TABLE Register (
    contest_id INT,
    user_id INT,
    PRIMARY KEY (contest_id, user_id)
);

-- Insert sample data
INSERT INTO Users VALUES
(6, 'Alice'),
(2, 'Bob'),
(7, 'Alex');

INSERT INTO Register VALUES
(215, 6),
(209, 2),
(208, 2),
(210, 6),
(208, 6),
(209, 7),
(209, 6),
(215, 7),
(208, 7),
(210, 2),
(207, 2),
(210, 7);

-- Solution: Calculate percentage of users registered for each contest
SELECT
    r.contest_id,
    ROUND(COUNT(DISTINCT r.user_id) * 100.0 / (SELECT COUNT(*) FROM Users), 2) AS percentage
FROM Register r
GROUP BY r.contest_id
ORDER BY percentage DESC, r.contest_id;
```

### Question 17: Queries Quality and Percentage

**Difficulty:** Easy
**LeetCode:** 1211

```sql
-- Create table
CREATE TABLE Queries (
    query_name VARCHAR(30),
    result VARCHAR(50),
    position INT,
    rating INT
);

-- Insert sample data
INSERT INTO Queries VALUES
('Dog', 'Golden Retriever', 1, 5),
('Dog', 'German Shepherd', 2, 5),
('Dog', 'Mule', 200, 1),
('Cat', 'Shirazi', 5, 2),
('Cat', 'Siamese', 3, 3),
('Cat', 'Sphynx', 7, 4);

-- Solution: Calculate quality and poor query percentage
SELECT
    query_name,
    ROUND(AVG(rating::numeric / position), 2) AS quality,
    ROUND(SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS poor_query_percentage
FROM Queries
WHERE query_name IS NOT NULL
GROUP BY query_name;
```

### Question 18: Monthly Transactions I

**Difficulty:** Medium
**LeetCode:** 1193

```sql
-- Create table
CREATE TABLE Transactions (
    id INT PRIMARY KEY,
    country VARCHAR(20),
    state VARCHAR(20) CHECK (state IN ('approved', 'declined')),
    amount INT,
    trans_date DATE
);

-- Insert sample data
INSERT INTO Transactions VALUES
(121, 'US', 'approved', 1000, '2018-12-18'),
(122, 'US', 'declined', 2000, '2018-12-19'),
(123, 'US', 'approved', 2000, '2019-01-01'),
(124, 'DE', 'approved', 2000, '2019-01-07');

-- Solution: Monthly transactions summary
SELECT
    TO_CHAR(trans_date, 'YYYY-MM') AS month,
    country,
    COUNT(*) AS trans_count,
    SUM(CASE WHEN state = 'approved' THEN 1 ELSE 0 END) AS approved_count,
    SUM(amount) AS trans_total_amount,
    SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END) AS approved_total_amount
FROM Transactions
GROUP BY TO_CHAR(trans_date, 'YYYY-MM'), country;
```

### Question 19: Immediate Food Delivery II

**Difficulty:** Medium
**LeetCode:** 1174

```sql
-- Create table
CREATE TABLE Delivery (
    delivery_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    customer_pref_delivery_date DATE
);

-- Insert sample data
INSERT INTO Delivery VALUES
(1, 1, '2019-08-01', '2019-08-02'),
(2, 2, '2019-08-02', '2019-08-02'),
(3, 1, '2019-08-11', '2019-08-12'),
(4, 3, '2019-08-24', '2019-08-24'),
(5, 3, '2019-08-21', '2019-08-22'),
(6, 2, '2019-08-11', '2019-08-13'),
(7, 4, '2019-08-09', '2019-08-09');

-- Solution: Find percentage of immediate first orders
WITH first_orders AS (
    SELECT
        customer_id,
        MIN(order_date) AS first_order_date
    FROM Delivery
    GROUP BY customer_id
)
SELECT
    ROUND(
        SUM(CASE WHEN d.order_date = d.customer_pref_delivery_date THEN 1 ELSE 0 END) * 100.0 / COUNT(*),
        2
    ) AS immediate_percentage
FROM Delivery d
JOIN first_orders f ON d.customer_id = f.customer_id AND d.order_date = f.first_order_date;
```

---

## 🟣 SECTION 4: SORTING AND GROUPING (Questions 20-26)

### Question 20: Number of Unique Subjects Taught by Each Teacher

**Difficulty:** Easy
**LeetCode:** 2356

```sql
-- Create table
CREATE TABLE Teacher (
    teacher_id INT,
    subject_id INT,
    dept_id INT,
    PRIMARY KEY (subject_id, dept_id)
);

-- Insert sample data
INSERT INTO Teacher VALUES
(1, 2, 3),
(1, 2, 4),
(1, 3, 3),
(2, 1, 1),
(2, 2, 1),
(2, 3, 1),
(2, 4, 1);

-- Solution: Count unique subjects per teacher
SELECT
    teacher_id,
    COUNT(DISTINCT subject_id) AS cnt
FROM Teacher
GROUP BY teacher_id;
```

### Question 21: User Activity for the Past 30 Days I

**Difficulty:** Easy
**LeetCode:** 1141

```sql
-- Create table
CREATE TABLE Activity (
    user_id INT,
    session_id INT,
    activity_date DATE,
    activity_type VARCHAR(20)
);

-- Insert sample data
INSERT INTO Activity VALUES
(1, 1, '2019-07-20', 'open_session'),
(1, 1, '2019-07-20', 'scroll_down'),
(1, 1, '2019-07-20', 'end_session'),
(2, 4, '2019-07-20', 'open_session'),
(2, 4, '2019-07-21', 'send_message'),
(2, 4, '2019-07-21', 'end_session'),
(3, 2, '2019-07-21', 'open_session'),
(3, 2, '2019-07-21', 'send_message'),
(3, 2, '2019-07-21', 'end_session'),
(4, 3, '2019-06-25', 'open_session'),
(4, 3, '2019-06-25', 'end_session');

-- Solution: Daily active users for past 30 days from 2019-07-27
SELECT
    activity_date AS day,
    COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
GROUP BY activity_date;
```

### Question 22: Product Sales Analysis III

**Difficulty:** Medium
**LeetCode:** 1070

```sql
-- Using Sales and Product tables from Question 7

-- Solution: Find first year of sales for each product
WITH first_year AS (
    SELECT
        product_id,
        MIN(year) AS first_year
    FROM Sales
    GROUP BY product_id
)
SELECT
    s.product_id,
    s.year AS first_year,
    s.quantity,
    s.price
FROM Sales s
JOIN first_year f ON s.product_id = f.product_id AND s.year = f.first_year;
```

### Question 23: Classes More Than 5 Students

**Difficulty:** Easy
**LeetCode:** 596

```sql
-- Create table
CREATE TABLE Courses (
    student VARCHAR(255),
    class VARCHAR(255),
    PRIMARY KEY (student, class)
);

-- Insert sample data
INSERT INTO Courses VALUES
('A', 'Math'),
('B', 'English'),
('C', 'Math'),
('D', 'Biology'),
('E', 'Math'),
('F', 'Computer'),
('G', 'Math'),
('H', 'Math'),
('I', 'Math');

-- Solution: Find classes with at least 5 students
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(student) >= 5;
```

### Question 24: Find Followers Count

**Difficulty:** Easy
**LeetCode:** 1729

```sql
-- Create table
CREATE TABLE Followers (
    user_id INT,
    follower_id INT,
    PRIMARY KEY (user_id, follower_id)
);

-- Insert sample data
INSERT INTO Followers VALUES
(0, 1),
(1, 0),
(2, 0),
(2, 1);

-- Solution: Count followers for each user
SELECT
    user_id,
    COUNT(follower_id) AS followers_count
FROM Followers
GROUP BY user_id
ORDER BY user_id;
```

### Question 25: Biggest Single Number

**Difficulty:** Easy
**LeetCode:** 619

```sql
-- Create table
CREATE TABLE MyNumbers (
    num INT
);

-- Insert sample data
INSERT INTO MyNumbers VALUES
(8),
(8),
(3),
(3),
(1),
(4),
(5),
(6);

-- Solution: Find largest single number
SELECT MAX(num) AS num
FROM (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(*) = 1
) AS single_numbers;
```

### Question 26: Customers Who Bought All Products

**Difficulty:** Medium
**LeetCode:** 1045

```sql
-- Create tables
CREATE TABLE Customer (
    customer_id INT,
    product_key INT
);

CREATE TABLE Product (
    product_key INT PRIMARY KEY
);

-- Insert sample data
INSERT INTO Customer VALUES
(1, 5),
(2, 6),
(3, 5),
(3, 6),
(1, 6);

INSERT INTO Product VALUES
(5),
(6);

-- Solution: Find customers who bought all products
SELECT customer_id
FROM Customer
GROUP BY customer_id
HAVING COUNT(DISTINCT product_key) = (SELECT COUNT(*) FROM Product);
```

---

## 🟠 SECTION 5: ADVANCED SELECT AND JOINS (Questions 27-33)

### Question 27: The Number of Employees Which Report to Each Employee

**Difficulty:** Easy
**LeetCode:** 1731

```sql
-- Create table
CREATE TABLE Employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(50),
    reports_to INT,
    age INT
);

-- Insert sample data
INSERT INTO Employees VALUES
(9, 'Hercy', NULL, 43),
(6, 'Alice', 9, 41),
(4, 'Bob', 9, 36),
(2, 'Winston', NULL, 37);

-- Solution: Find managers with their direct reports count and average age
SELECT
    e1.employee_id,
    e1.name,
    COUNT(e2.employee_id) AS reports_count,
    ROUND(AVG(e2.age)) AS average_age
FROM Employees e1
JOIN Employees e2 ON e1.employee_id = e2.reports_to
GROUP BY e1.employee_id, e1.name
ORDER BY e1.employee_id;
```

### Question 28: Primary Department for Each Employee

**Difficulty:** Easy
**LeetCode:** 1789

```sql
-- Create table
CREATE TABLE Employee (
    employee_id INT,
    department_id INT,
    primary_flag VARCHAR(1) CHECK (primary_flag IN ('Y', 'N')),
    PRIMARY KEY (employee_id, department_id)
);

-- Insert sample data
INSERT INTO Employee VALUES
(1, 1, 'N'),
(2, 1, 'Y'),
(2, 2, 'N'),
(3, 3, 'N'),
(4, 2, 'N'),
(4, 3, 'Y'),
(4, 4, 'N');

-- Solution: Find primary department for each employee
SELECT employee_id, department_id
FROM Employee
WHERE primary_flag = 'Y'
UNION
SELECT employee_id, department_id
FROM Employee
WHERE employee_id IN (
    SELECT employee_id
    FROM Employee
    GROUP BY employee_id
    HAVING COUNT(*) = 1
);
```

### Question 29: Triangle Judgement

**Difficulty:** Easy
**LeetCode:** 610

```sql
-- Create table
CREATE TABLE Triangle (
    x INT,
    y INT,
    z INT
);

-- Insert sample data
INSERT INTO Triangle VALUES
(13, 15, 30),
(10, 20, 15);

-- Solution: Determine if three sides can form a triangle
SELECT
    x, y, z,
    CASE
        WHEN x + y > z AND x + z > y AND y + z > x THEN 'Yes'
        ELSE 'No'
    END AS triangle
FROM Triangle;
```

### Question 30: Consecutive Numbers

**Difficulty:** Medium
**LeetCode:** 180

```sql
-- Create table
CREATE TABLE Logs (
    id SERIAL PRIMARY KEY,
    num VARCHAR(50)
);

-- Insert sample data
INSERT INTO Logs (num) VALUES
('1'),
('1'),
('1'),
('2'),
('1'),
('2'),
('2');

-- Solution: Find numbers appearing at least 3 times consecutively
SELECT DISTINCT l1.num AS ConsecutiveNums
FROM Logs l1
JOIN Logs l2 ON l1.id = l2.id - 1 AND l1.num = l2.num
JOIN Logs l3 ON l2.id = l3.id - 1 AND l2.num = l3.num;
```

### Question 31: Product Price at a Given Date

**Difficulty:** Medium
**LeetCode:** 1164

```sql
-- Create table
CREATE TABLE Products (
    product_id INT,
    new_price INT,
    change_date DATE,
    PRIMARY KEY (product_id, change_date)
);

-- Insert sample data
INSERT INTO Products VALUES
(1, 20, '2019-08-14'),
(2, 50, '2019-08-14'),
(1, 30, '2019-08-15'),
(1, 35, '2019-08-16'),
(2, 65, '2019-08-17'),
(3, 20, '2019-08-18');

-- Solution: Find product prices on 2019-08-16
WITH latest_price AS (
    SELECT
        product_id,
        new_price,
        ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY change_date DESC) AS rn
    FROM Products
    WHERE change_date <= '2019-08-16'
)
SELECT
    p.product_id,
    COALESCE(lp.new_price, 10) AS price
FROM (SELECT DISTINCT product_id FROM Products) p
LEFT JOIN latest_price lp ON p.product_id = lp.product_id AND lp.rn = 1;
```

### Question 32: Last Person to Fit in the Bus

**Difficulty:** Medium
**LeetCode:** 1204

```sql
-- Create table
CREATE TABLE Queue (
    person_id INT PRIMARY KEY,
    person_name VARCHAR(50),
    weight INT,
    turn INT
);

-- Insert sample data
INSERT INTO Queue VALUES
(5, 'Alice', 250, 1),
(4, 'Bob', 175, 5),
(3, 'Alex', 350, 2),
(6, 'John Cena', 400, 3),
(1, 'Winston', 500, 6),
(2, 'Marie', 200, 4);

-- Solution: Find last person to fit in bus (limit 1000kg)
WITH running_weight AS (
    SELECT
        person_name,
        SUM(weight) OVER (ORDER BY turn) AS total_weight
    FROM Queue
)
SELECT person_name
FROM running_weight
WHERE total_weight <= 1000
ORDER BY total_weight DESC
LIMIT 1;
```

### Question 33: Count Salary Categories

**Difficulty:** Medium
**LeetCode:** 1907

```sql
-- Create table
CREATE TABLE Accounts (
    account_id INT PRIMARY KEY,
    income INT
);

-- Insert sample data
INSERT INTO Accounts VALUES
(3, 108939),
(2, 12747),
(8, 87709),
(6, 91796);

-- Solution: Count accounts in salary categories
SELECT
    'Low Salary' AS category,
    COUNT(CASE WHEN income < 20000 THEN 1 END) AS accounts_count
FROM Accounts
UNION ALL
SELECT
    'Average Salary' AS category,
    COUNT(CASE WHEN income >= 20000 AND income <= 50000 THEN 1 END) AS accounts_count
FROM Accounts
UNION ALL
SELECT
    'High Salary' AS category,
    COUNT(CASE WHEN income > 50000 THEN 1 END) AS accounts_count
FROM Accounts;
```

---

## 🔴 SECTION 6: SUBQUERIES (Questions 34-40)

### Question 34: Employees Whose Manager Left the Company

**Difficulty:** Easy
**LeetCode:** 1978

```sql
-- Create table
CREATE TABLE Employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(50),
    manager_id INT,
    salary INT
);

-- Insert sample data
INSERT INTO Employees VALUES
(3, 'Mila', 9, 60301),
(12, 'Antonella', NULL, 31000),
(13, 'Emery', NULL, 67084),
(1, 'Kalel', 11, 21241),
(9, 'Mikaela', NULL, 50937),
(11, 'Joziah', 6, 28485);

-- Solution: Find employees with salary < 30000 whose manager left
SELECT employee_id
FROM Employees
WHERE salary < 30000
  AND manager_id IS NOT NULL
  AND manager_id NOT IN (SELECT employee_id FROM Employees)
ORDER BY employee_id;
```

### Question 35: Exchange Seats

**Difficulty:** Medium
**LeetCode:** 626

```sql
-- Create table
CREATE TABLE Seat (
    id INT PRIMARY KEY,
    student VARCHAR(50)
);

-- Insert sample data
INSERT INTO Seat VALUES
(1, 'Abbot'),
(2, 'Doris'),
(3, 'Emerson'),
(4, 'Green'),
(5, 'Jeames');

-- Solution: Swap every two consecutive students
SELECT
    CASE
        WHEN id % 2 = 0 THEN id - 1
        WHEN id % 2 = 1 AND id < (SELECT MAX(id) FROM Seat) THEN id + 1
        ELSE id
    END AS id,
    student
FROM Seat
ORDER BY id;
```

### Question 36: Movie Rating

**Difficulty:** Medium
**LeetCode:** 1341

```sql
-- Create tables
CREATE TABLE Movies (
    movie_id INT PRIMARY KEY,
    title VARCHAR(50)
);

CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE MovieRating (
    movie_id INT,
    user_id INT,
    rating INT,
    created_at DATE,
    PRIMARY KEY (movie_id, user_id)
);

-- Insert sample data
INSERT INTO Movies VALUES
(1, 'Avengers'),
(2, 'Frozen 2'),
(3, 'Joker');

INSERT INTO Users VALUES
(1, 'Daniel'),
(2, 'Monica'),
(3, 'Maria'),
(4, 'James');

INSERT INTO MovieRating VALUES
(1, 1, 3, '2020-01-12'),
(1, 2, 4, '2020-02-11'),
(1, 3, 2, '2020-02-12'),
(1, 4, 1, '2020-01-01'),
(2, 1, 5, '2020-02-17'),
(2, 2, 2, '2020-02-01'),
(2, 3, 2, '2020-03-01'),
(3, 1, 3, '2020-02-22'),
(3, 2, 4, '2020-02-25');

-- Solution: Find user with most ratings and highest avg rated movie in Feb 2020
(
    SELECT name AS results
    FROM Users u
    JOIN MovieRating mr ON u.user_id = mr.user_id
    GROUP BY u.user_id, u.name
    ORDER BY COUNT(*) DESC, name
    LIMIT 1
)
UNION ALL
(
    SELECT title AS results
    FROM Movies m
    JOIN MovieRating mr ON m.movie_id = mr.movie_id
    WHERE DATE_TRUNC('month', created_at) = '2020-02-01'
    GROUP BY m.movie_id, m.title
    ORDER BY AVG(rating) DESC, title
    LIMIT 1
);
```

### Question 37: Restaurant Growth

**Difficulty:** Medium
**LeetCode:** 1321

```sql
-- Create table
CREATE TABLE Customer (
    customer_id INT,
    name VARCHAR(50),
    visited_on DATE,
    amount INT,
    PRIMARY KEY (customer_id, visited_on)
);

-- Insert sample data
INSERT INTO Customer VALUES
(1, 'Jhon', '2019-01-01', 100),
(2, 'Daniel', '2019-01-02', 110),
(3, 'Jade', '2019-01-03', 120),
(4, 'Khaled', '2019-01-04', 130),
(5, 'Winston', '2019-01-05', 110),
(6, 'Elvis', '2019-01-06', 140),
(7, 'Anna', '2019-01-07', 150),
(8, 'Maria', '2019-01-08', 80),
(9, 'Jaze', '2019-01-09', 110),
(1, 'Jhon', '2019-01-10', 130),
(3, 'Jade', '2019-01-10', 150);

-- Solution: Calculate 7-day moving average
WITH daily_revenue AS (
    SELECT
        visited_on,
        SUM(amount) AS daily_amount
    FROM Customer
    GROUP BY visited_on
)
SELECT
    visited_on,
    SUM(daily_amount) OVER (ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS amount,
    ROUND(
        AVG(daily_amount) OVER (ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)::numeric,
        2
    ) AS average_amount
FROM daily_revenue
WHERE visited_on >= (SELECT MIN(visited_on) + INTERVAL '6 days' FROM Customer)
ORDER BY visited_on;
```

### Question 38: Friend Requests II: Who Has the Most Friends

**Difficulty:** Medium
**LeetCode:** 602

```sql
-- Create table
CREATE TABLE RequestAccepted (
    requester_id INT,
    accepter_id INT,
    accept_date DATE,
    PRIMARY KEY (requester_id, accepter_id)
);

-- Insert sample data
INSERT INTO RequestAccepted VALUES
(1, 2, '2016-06-03'),
(1, 3, '2016-06-08'),
(2, 3, '2016-06-08'),
(3, 4, '2016-06-09');

-- Solution: Find person with most friends
WITH all_friends AS (
    SELECT requester_id AS person FROM RequestAccepted
    UNION ALL
    SELECT accepter_id AS person FROM RequestAccepted
)
SELECT
    person AS id,
    COUNT(*) AS num
FROM all_friends
GROUP BY person
ORDER BY num DESC
LIMIT 1;
```

### Question 39: Investment in 2016

**Difficulty:** Medium
**LeetCode:** 585

```sql
-- Create table
CREATE TABLE Insurance (
    pid INT PRIMARY KEY,
    tiv_2015 FLOAT,
    tiv_2016 FLOAT,
    lat FLOAT,
    lon FLOAT
);

-- Insert sample data
INSERT INTO Insurance VALUES
(1, 10, 5, 10, 10),
(2, 20, 20, 20, 20),
(3, 10, 30, 20, 20),
(4, 10, 40, 40, 40);

-- Solution: Find sum of tiv_2016 with specific conditions
SELECT ROUND(SUM(tiv_2016)::numeric, 2) AS tiv_2016
FROM Insurance
WHERE tiv_2015 IN (
    SELECT tiv_2015
    FROM Insurance
    GROUP BY tiv_2015
    HAVING COUNT(*) > 1
)
AND (lat, lon) IN (
    SELECT lat, lon
    FROM Insurance
    GROUP BY lat, lon
    HAVING COUNT(*) = 1
);
```

### Question 40: Department Top Three Salaries

**Difficulty:** Hard
**LeetCode:** 185

```sql
-- Create tables
CREATE TABLE Employee (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    salary INT,
    departmentId INT
);

CREATE TABLE Department (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- Insert sample data
INSERT INTO Employee VALUES
(1, 'Joe', 85000, 1),
(2, 'Henry', 80000, 2),
(3, 'Sam', 60000, 2),
(4, 'Max', 90000, 1),
(5, 'Janet', 69000, 1),
(6, 'Randy', 85000, 1),
(7, 'Will', 70000, 1);

INSERT INTO Department VALUES
(1, 'IT'),
(2, 'Sales');

-- Solution: Find top 3 salaries in each department
WITH ranked_salaries AS (
    SELECT
        e.name AS Employee,
        e.salary AS Salary,
        d.name AS Department,
        DENSE_RANK() OVER (PARTITION BY d.name ORDER BY e.salary DESC) AS salary_rank
    FROM Employee e
    JOIN Department d ON e.departmentId = d.id
)
SELECT Department, Employee, Salary
FROM ranked_salaries
WHERE salary_rank <= 3
ORDER BY Department, Salary DESC;
```

---

## 🟡 SECTION 7: ADVANCED STRING FUNCTIONS (Questions 41-47)

### Question 41: Fix Names in a Table

**Difficulty:** Easy
**LeetCode:** 1667

```sql
-- Create table
CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- Insert sample data
INSERT INTO Users VALUES
(1, 'aLice'),
(2, 'bOB');

-- Solution: Capitalize first letter, lowercase rest
SELECT
    user_id,
    CONCAT(UPPER(SUBSTRING(name, 1, 1)), LOWER(SUBSTRING(name, 2))) AS name
FROM Users
ORDER BY user_id;
```

### Question 42: Patients With a Condition

**Difficulty:** Easy
**LeetCode:** 1527

```sql
-- Create table
CREATE TABLE Patients (
    patient_id INT PRIMARY KEY,
    patient_name VARCHAR(50),
    conditions VARCHAR(100)
);

-- Insert sample data
INSERT INTO Patients VALUES
(1, 'Daniel', 'YFEV COUGH'),
(2, 'Alice', ''),
(3, 'Bob', 'DIAB100 MYOP'),
(4, 'George', 'ACNE DIAB100'),
(5, 'Alain', 'DIAB201');

-- Solution: Find patients with Type I Diabetes (DIAB1 prefix)
SELECT *
FROM Patients
WHERE conditions LIKE 'DIAB1%' OR conditions LIKE '% DIAB1%';
```

### Question 43: Delete Duplicate Emails

**Difficulty:** Easy
**LeetCode:** 196

```sql
-- Create table
CREATE TABLE Person (
    id INT PRIMARY KEY,
    email VARCHAR(100)
);

-- Insert sample data
INSERT INTO Person VALUES
(1, 'john@example.com'),
(2, 'bob@example.com'),
(3, 'john@example.com');

-- Solution: Delete duplicate emails keeping smallest id
DELETE FROM Person
WHERE id NOT IN (
    SELECT MIN(id)
    FROM (SELECT * FROM Person) AS p
    GROUP BY email
);
```

### Question 44: Second Highest Salary

**Difficulty:** Medium
**LeetCode:** 176

```sql
-- Create table
CREATE TABLE Employee (
    id INT PRIMARY KEY,
    salary INT
);

-- Insert sample data
INSERT INTO Employee VALUES
(1, 100),
(2, 200),
(3, 300);

-- Solution: Find second highest salary
SELECT
    (SELECT DISTINCT salary
     FROM Employee
     ORDER BY salary DESC
     LIMIT 1 OFFSET 1) AS SecondHighestSalary;
```

### Question 45: Group Sold Products By The Date

**Difficulty:** Easy
**LeetCode:** 1484

```sql
-- Create table
CREATE TABLE Activities (
    sell_date DATE,
    product VARCHAR(50)
);

-- Insert sample data
INSERT INTO Activities VALUES
('2020-05-30', 'Headphone'),
('2020-06-01', 'Pencil'),
('2020-06-02', 'Mask'),
('2020-05-30', 'Basketball'),
('2020-06-01', 'Bible'),
('2020-06-02', 'Mask'),
('2020-05-30', 'T-Shirt');

-- Solution: Group products by date
SELECT
    sell_date,
    COUNT(DISTINCT product) AS num_sold,
    STRING_AGG(DISTINCT product, ',' ORDER BY product) AS products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

### Question 46: List the Products Ordered in a Period

**Difficulty:** Easy
**LeetCode:** 1327

```sql
-- Create tables
CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(50),
    product_category VARCHAR(50)
);

CREATE TABLE Orders (
    product_id INT,
    order_date DATE,
    unit INT
);

-- Insert sample data
INSERT INTO Products VALUES
(1, 'Leetcode Solutions', 'Book'),
(2, 'Jewels of Stringology', 'Book'),
(3, 'HP', 'Laptop'),
(4, 'Lenovo', 'Laptop'),
(5, 'Leetcode Kit', 'T-shirt');

INSERT INTO Orders VALUES
(1, '2020-02-05', 60),
(1, '2020-02-10', 70),
(2, '2020-01-18', 30),
(2, '2020-02-11', 80),
(3, '2020-02-17', 2),
(3, '2020-02-24', 3),
(4, '2020-03-01', 20),
(4, '2020-03-04', 30),
(4, '2020-03-04', 60),
(5, '2020-02-25', 50),
(5, '2020-02-27', 50),
(5, '2020-03-01', 50);

-- Solution: Products with 100+ units ordered in Feb 2020
SELECT
    p.product_name,
    SUM(o.unit) AS unit
FROM Products p
JOIN Orders o ON p.product_id = o.product_id
WHERE o.order_date BETWEEN '2020-02-01' AND '2020-02-29'
GROUP BY p.product_id, p.product_name
HAVING SUM(o.unit) >= 100;
```

### Question 47: Find Users With Valid E-Mails

**Difficulty:** Easy
**LeetCode:** 1517

```sql
-- Create table
CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    name VARCHAR(50),
    mail VARCHAR(100)
);

-- Insert sample data
INSERT INTO Users VALUES
(1, 'Winston', 'winston@leetcode.com'),
(2, 'Jonathan', 'jonathanisgreat'),
(3, 'Annabelle', 'bella-@leetcode.com'),
(4, 'Sally', 'sally.come@leetcode.com'),
(5, 'Marwan', 'quarz#2020@leetcode.com'),
(6, 'David', 'david69@gmail.com'),
(7, 'Shapiro', '.shapo@leetcode.com');

-- Solution: Find users with valid email format
SELECT *
FROM Users
WHERE mail ~ '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\.com$';
```

---

## 🔵 SECTION 8: WINDOW FUNCTIONS (Questions 48-50)

### Question 48: Nth Highest Salary

**Difficulty:** Medium
**LeetCode:** 177

```sql
-- Create function to find Nth highest salary
CREATE OR REPLACE FUNCTION getNthHighestSalary(N INT)
RETURNS INT AS $$
BEGIN
    RETURN (
        SELECT DISTINCT salary
        FROM Employee
        ORDER BY salary DESC
        LIMIT 1 OFFSET N - 1
    );
END;
$$ LANGUAGE plpgsql;

-- Test with Employee table from Question 44
SELECT getNthHighestSalary(2);
```

### Question 49: Rank Scores

**Difficulty:** Medium
**LeetCode:** 178

```sql
-- Create table
CREATE TABLE Scores (
    id INT PRIMARY KEY,
    score DECIMAL(10,2)
);

-- Insert sample data
INSERT INTO Scores VALUES
(1, 3.50),
(2, 3.65),
(3, 4.00),
(4, 3.85),
(5, 4.00),
(6, 3.65);

-- Solution: Rank scores (same scores get same rank)
SELECT
    score,
    DENSE_RANK() OVER (ORDER BY score DESC) AS rank
FROM Scores
ORDER BY score DESC;
```

### Question 50: Game Play Analysis IV

**Difficulty:** Medium
**LeetCode:** 550

```sql
-- Create table
CREATE TABLE Activity (
    player_id INT,
    device_id INT,
    event_date DATE,
    games_played INT,
    PRIMARY KEY (player_id, event_date)
);

-- Insert sample data
INSERT INTO Activity VALUES
(1, 2, '2016-03-01', 5),
(1, 2, '2016-03-02', 6),
(2, 3, '2016-06-25', 1),
(3, 1, '2016-03-02', 0),
(3, 4, '2016-07-03', 5);

-- Solution: Fraction of players who logged in day after first login
WITH first_login AS (
    SELECT
        player_id,
        MIN(event_date) AS first_date
    FROM Activity
    GROUP BY player_id
),
next_day_login AS (
    SELECT
        f.player_id
    FROM first_login f
    JOIN Activity a ON f.player_id = a.player_id
                   AND a.event_date = f.first_date + INTERVAL '1 day'
)
SELECT
    ROUND(COUNT(DISTINCT n.player_id)::numeric / COUNT(DISTINCT f.player_id), 2) AS fraction
FROM first_login f
LEFT JOIN next_day_login n ON f.player_id = n.player_id;
```

---

## 📝 Quick Reference - Key SQL Patterns

### JOINs

```sql
-- LEFT JOIN
SELECT * FROM table1 t1 LEFT JOIN table2 t2 ON t1.id = t2.id;

-- SELF JOIN
SELECT * FROM table t1 JOIN table t2 ON t1.parent_id = t2.id;

-- CROSS JOIN
SELECT * FROM table1 CROSS JOIN table2;
```

### Window Functions

```sql
-- ROW_NUMBER()
ROW_NUMBER() OVER (PARTITION BY column ORDER BY column)

-- RANK() and DENSE_RANK()
RANK() OVER (ORDER BY column DESC)
DENSE_RANK() OVER (ORDER BY column DESC)

-- Running totals
SUM(column) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- LAG and LEAD
LAG(column, 1) OVER (ORDER BY date)
LEAD(column, 1) OVER (ORDER BY date)
```

### CTEs (Common Table Expressions)

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

### Conditional Aggregation

```sql
SUM(CASE WHEN condition THEN 1 ELSE 0 END)
COUNT(CASE WHEN condition THEN 1 END)
```

### String Functions

```sql
-- PostgreSQL specific
STRING_AGG(column, ',' ORDER BY column)  -- GROUP_CONCAT equivalent
column ~ 'regex_pattern'                  -- Regex matching
SUBSTRING(string FROM position FOR length)
```

### Date Functions

```sql
DATE_TRUNC('month', date_column)
date_column + INTERVAL '1 day'
TO_CHAR(date_column, 'YYYY-MM')
EXTRACT(YEAR FROM date_column)
```

---

## 🎯 Practice Tips

1. **Start Simple**: Begin with SELECT queries, then progress to JOINs
2. **Understand Execution Order**: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
3. **Practice Daily**: Solve 2-3 problems daily for consistency
4. **Analyze Query Plans**: Use `EXPLAIN ANALYZE` to understand performance
5. **Learn Patterns**: Most problems follow common patterns - master them

---

## 🚦 Difficulty Progression

- **Week 1**: Questions 1-15 (Basic SELECT and JOINs)
- **Week 2**: Questions 16-26 (Aggregations and Grouping)
- **Week 3**: Questions 27-40 (Advanced JOINs and Subqueries)
- **Week 4**: Questions 41-50 (String Functions and Window Functions)

---

_Remember: The key to mastering SQL is consistent practice and understanding the underlying concepts, not just memorizing syntax._

**Happy Querying! 🚀**
