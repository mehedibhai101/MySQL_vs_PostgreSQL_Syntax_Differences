
# 🐘 PostgreSQL vs. 🐬 MySQL: The Data Analyst's Syntax Cheatsheet

If you already know PostgreSQL, learning MySQL is mostly about understanding the dialect differences. They are both powerful relational database management systems based on SQL, meaning the core logic remains the same. 

This guide maps out the core syntax, data type, and functional differences between the two systems so you can translate your Postgres knowledge directly into MySQL.

---

## 1. Quotation Marks & Identifiers

How you quote strings and table/column names is one of the most frequent causes of syntax errors when switching dialects.

| Feature | PostgreSQL | MySQL |
| :--- | :--- | :--- |
| **String Literals** | Always use single quotes: `'string'` | Can use single or double: `'string'` or `"string"` |
| **Identifiers (Tables/Columns)** | Double quotes: `"User Table"` | Backticks: `` `User Table` `` |
| **Case Sensitivity (Strings)** | Case-sensitive by default (`'A' != 'a'`) | Case-insensitive by default (`'A' = 'a'`) |

---

## 2. Common Data Types & Auto-Increment

Data types handle slightly differently, especially regarding booleans and auto-incrementing IDs.

| Concept | PostgreSQL | MySQL |
| :--- | :--- | :--- |
| **Auto-Incrementing ID** | `SERIAL` or `GENERATED ALWAYS AS IDENTITY` | `AUTO_INCREMENT` |
| **Boolean** | `BOOLEAN` (Stores `true`/`false`) | `TINYINT(1)` (Stores `1`/`0`) |
| **Character/String** | `VARCHAR(n)`, `TEXT` | `VARCHAR(n)`, `TEXT` |
| **Date/Time** | `TIMESTAMP` (without timezone), `TIMESTAMPTZ` | `DATETIME`, `TIMESTAMP` (converts to UTC) |

**Example: Table Creation**

*PostgreSQL:*
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    is_active BOOLEAN
);

```

*MySQL:*

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50),
    is_active TINYINT(1)
);

```

---

## 3. String Manipulation & Concatenation

PostgreSQL uses standard SQL operators for strings, while MySQL relies heavily on built-in functions.

| Operation | PostgreSQL | MySQL |
| --- | --- | --- |
| **Concatenation** | `col1 \|\| ' ' \|\| col2` | `CONCAT(col1, ' ', col2)` |
| **Length of String** | `LENGTH(col)` or `CHAR_LENGTH(col)` | `LENGTH(col)` (bytes) or `CHAR_LENGTH(col)` (chars) |
| **Substring** | `SUBSTRING(col FROM 1 FOR 3)` | `SUBSTRING(col, 1, 3)` |
| **Regex Match** | `col ~ '^pattern$'` | `col REGEXP '^pattern$'` |
| **Regex Not Match** | `col !~ '^pattern$'` | `col NOT REGEXP '^pattern$'` |

> **Note:** In MySQL, the `||` operator acts as a logical `OR` by default, not a concatenator!

---

## 4. Date and Time Functions

Date math is arguably where the two languages differ the most. Postgres uses the `INTERVAL` keyword heavily, while MySQL uses specific date arithmetic functions.

| Operation | PostgreSQL | MySQL |
| --- | --- | --- |
| **Current Date & Time** | `NOW()`, `CURRENT_TIMESTAMP` | `NOW()`, `CURRENT_TIMESTAMP` |
| **Current Date Only** | `CURRENT_DATE` | `CURDATE()` |
| **Add/Subtract Time** | `NOW() ± INTERVAL '1 day'` | `DATE_ADD(NOW(), INTERVAL 1 DAY)` <br> `DATE_SUB(NOW(), INTERVAL 1 DAY)` |
| **Difference in Days** | `date1 - date2` | `DATEDIFF(date1, date2)` |
| **Extract Parts** | `EXTRACT(MONTH FROM date_col)` | `EXTRACT(MONTH FROM date_col)` OR `MONTH(date_col)` |
| **Truncate Date** | `DATE_TRUNC('month', date_col)` | `DATE_FORMAT(date_col, '%Y-%m-01')` |

---

## 5. Type Casting

| Feature | PostgreSQL | MySQL |
| --- | --- | --- |
| **Standard Cast** | `CAST(col AS type)` | `CAST(col AS type)` |
| **Shorthand Cast** | `col::type` (e.g., `amount::INT`) | *Not Supported* |
| **Convert Function** | *Not Supported* | `CONVERT(col, type)` |

---

## 6. Null Handling & Control Flow

| Feature | PostgreSQL | MySQL |
| --- | --- | --- |
| **Standard IF/ELSE** | `CASE WHEN... THEN... END` | `CASE WHEN... THEN... END` |
| **Shorthand IF** | *Not Supported* (Must use `CASE`) | `IF(condition, true_val, false_val)` |
| **Replace NULL** | `COALESCE(col, 'Default')` | `COALESCE(col, 'Default')` OR `IFNULL(col, 'Default')` |

---

## 7. Limiting, Offsetting, and Returning Results

| Feature | PostgreSQL | MySQL |
| --- | --- | --- |
| **Limit & Offset** | `LIMIT 10 OFFSET 5` | `LIMIT 10 OFFSET 5` <br> OR `LIMIT 5, 10` *(Note: Offset comes first here!)* |
| **Return Inserted Row** | `INSERT INTO table (...) RETURNING id;` | *Not natively supported in INSERT.* <br> Must use `SELECT LAST_INSERT_ID();` after insert. |

---

## 8. Window Functions

Both databases support standard window functions (`ROW_NUMBER()`, `RANK()`, `SUM() OVER()`, etc.). However, Postgres has a specific `FILTER` clause that MySQL lacks.

*PostgreSQL (Filtering an aggregate):*

```sql
SELECT 
    SUM(amount) FILTER (WHERE status = 'paid') 
FROM sales;

```

*MySQL (Must use CASE inside the aggregate instead):*

```sql
SELECT 
    SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) 
FROM sales;

```

---

## 9. Advanced Joins & Set Operations

MySQL lacks some of Postgres's out-of-the-box join functionality, requiring structural workarounds.

| Feature | PostgreSQL | MySQL |
| --- | --- | --- |
| **FULL OUTER JOIN** | Natively supported (`FULL OUTER JOIN`) | *Not supported.* Must use `LEFT JOIN ... UNION ... RIGHT JOIN` |
| **INTERSECT & EXCEPT** | Natively supported | Only supported in MySQL 8.0.31 and later. (Older versions require `INNER JOIN` / `NOT EXISTS` workarounds) |

---

## 10. String Aggregation (Group Concat)

When you need to collapse multiple rows of strings into a single comma-separated list.

*PostgreSQL:*

```sql
SELECT team, STRING_AGG(player_name, ', ') 
FROM players 
GROUP BY team;

```

*MySQL:*

```sql
SELECT team, GROUP_CONCAT(player_name SEPARATOR ', ') 
FROM players 
GROUP BY team;

```

---

## 11. "Upserts" (Insert or Update)

How to handle inserting a row when a primary key or unique constraint already exists.

*PostgreSQL:*

```sql
INSERT INTO users (id, name) VALUES (1, 'John')
ON CONFLICT (id) DO UPDATE SET name = 'John';

```

*MySQL:*

```sql
INSERT INTO users (id, name) VALUES (1, 'John')
ON DUPLICATE KEY UPDATE name = 'John';

```

---

## 12. Pattern Matching (Case Sensitivity)

Because MySQL strings are case-insensitive by default, its `LIKE` operator behaves differently.

| Operation | PostgreSQL | MySQL |
| --- | --- | --- |
| **Case-Sensitive Like** | `LIKE` | `LIKE BINARY` |
| **Case-Insensitive Like** | `ILIKE` | `LIKE` (Default behavior) |

---

## 🛡️ License

This project is licensed under the [MIT License](LICENSE). You are free to use, modify, and distribute it with proper attribution.

---

## 🌟 About Me

Hi! I’m **Mehedi Hasan**, well known as **Mehedi Bhai**, a Certified Data Analyst with strong proficiency in *Excel*, *Power BI*, and *SQL*. I specialize in data visualization, transforming raw data into clear, meaningful insights that help businesses make impactful data-driven decisions.

Let’s connect:

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge\&logo=linkedin\&logoColor=white)](https://www.linkedin.com/in/mehedi-hasan-b3370130a/)
[![YouTube](https://img.shields.io/badge/YouTube-red?style=for-the-badge\&logo=youtube\&logoColor=white)](https://youtube.com/@mehedibro101?si=huk7eZ05dOwHTs1-)
