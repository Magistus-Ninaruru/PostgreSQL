# Get Started

To start a PostgreSQL session on *Git Bash*:

```
psql -U postgres -p 5444
```

# Create Database

To create a new database:

```
CREATE DATABASE sales_db;
```

We can also check a list of databases using

```
\l
```

To link to a database within psql, use

```
\c sales_db
```

# Create Table

We can create a new table using CREATE TABLE commands:

```
CREATE TABLE customer(
    first_name   VARCHAR(30) NOT NULL,
    last_name    VARCHAR(30) NOT NULL,
    email        VARCHAR(60) NOT NULL,
    company      VARCHAR(60) NULL,
    street       VARCHAR(50) NOT NULL,
    city         VARCHAR(40) NOT NULL,
    state        CHAR(2) NOT NULL DEFAULT 'PA',  -- set default value
    zip          SMALLINT NOT NULL,              -- SMALLINT only occupies 2 bytes but INT occupies 4 bytes
    phone        VARCHAR(20) NOT NULL,
    birth_date   DATE NULL,
    sex CHAR(1)  NOT NULL,
    date_entered TIMESTAMP NOT NULL,
    id           SERIAL PRIMARY KEY              -- use SERIAL to identify a auto-incrementing column, especially for PK
);
```

To drop a table if it previously exists, use the following syntax:

```
DROP TABLE IF EXISTS table_name [ CASCADE | RESTRICT ];
```

Note:
 - **IF EXISTS** ensures the command does not throw an error if the table does not exist.
 - **CASCADE** automatically drops all objects (like views, constraints, or foreign key references) that depend on this table.
 - **RESTRICT** is the default behavior.

To insert values into the table, use the following:

```
INSERT INTO customer (first_name, last_name, email, company, street, city, state, zip, phone, birth_date, sex, date_entered) VALUES 
    ('Matthew', 'Martinez', 'matthewmartinez@ge.com', 'GE', '602 Main Place', 'Fontana', 'CA', '92336', '117-997-7764', '1931-09-04', 'M', '2015-01-01 22:39:28'), 
    ('Melissa', 'Moore', 'melissamoore@aramark.com', 'Aramark', '463 Park Rd', 'Lakewood', 'NJ', '08701', '269-720-7259', '1967-08-27', 'M', '2017-10-20 21:59:29'), 
    ('Melissa', 'Brown', 'melissabrown@verizon.com', 'Verizon', '712 View Ave', 'Houston', 'TX', '77084', '280-570-5166', '1948-06-14', 'F', '2016-07-16 12:26:45'), 
    ('Jennifer', 'Thomas', 'jenniferthomas@aramark.com', 'Aramark', '231 Elm St', 'Mission', 'TX', '78572', '976-147-9254', '1998-03-14', 'F', '2018-01-08 09:27:55'), 
    ('Stephanie', 'Martinez', 'stephaniemartinez@albertsons.com', 'Albertsons', '386 Second St', 'Lakewood', 'NJ', '08701', '820-131-6053', '1998-01-24', 'M', '2016-06-18 13:27:34'), 
    ('Daniel', 'Williams', 'danielwilliams@tjx.com', 'TJX', '107 Pine St', 'Katy', 'TX', '77449', '744-906-9837', '1985-07-20', 'F', '2015-07-03 10:40:18'), 
    ('Lauren', 'Anderson', 'laurenanderson@pepsi.com', 'Pepsi', '13 Maple Ave', 'Riverside', 'CA', '92503', '747-993-2446', '1973-09-09', 'F', '2018-02-01 16:43:51'), 
    ('Michael', 'Jackson', 'michaeljackson@disney.com', 'Disney', '818 Pine Ave', 'Mission', 'TX', '78572', '126-423-3144', '1951-03-03', 'F', '2017-04-02 21:57:36'), 
    ('Ashley', 'Johnson', 'ashleyjohnson@boeing.com', 'Boeing', '874 Oak Ave', 'Pacoima', 'CA', '91331', '127-475-1658', '1937-05-10', 'F', '2015-01-04 08:58:56'), 
    ('Brittany', 'Thomas', 'brittanythomas@walmart.com', 'Walmart', '187 Maple Ave', 'Brownsville', 'TX', '78521', '447-788-4913', '1986-10-22', 'F', '2018-05-23 08:04:32'), 
    ('Matthew', 'Smith', 'matthewsmith@ups.com', 'UPS', '123 Lake St', 'Brownsville', 'TX', '78521', '961-108-3758', '1950-06-16', 'F', '2018-03-15 10:08:54'), 
    ('Lauren', 'Wilson', 'laurenwilson@target.com', 'Target', '942 Fifth Ave', 'Mission', 'TX', '78572', '475-578-8519', '1965-12-26', 'M', '2017-07-16 11:01:01'), 
    ('Justin', 'Smith', 'justinsmith@boeing.com', 'Boeing', '844 Lake Ave', 'Lawrenceville', 'GA', '30044', '671-957-1492', '1956-03-16', 'F', '2017-10-07 10:50:08'), 
    ('Jessica', 'Garcia', 'jessicagarcia@toyota.com', 'Toyota', '123 Pine Place', 'Fontana', 'CA', '92336', '744-647-2359', '1996-08-05', 'F', '2016-09-14 12:33:05'), 
    ('Matthew', 'Jackson', 'matthewjackson@bp.com', 'BP', '538 Cedar Ave', 'Katy', 'TX', '77449', '363-430-1813', '1966-02-26', 'F', '2016-05-01 19:25:17'), 
    ('Stephanie', 'Thomas', 'stephaniethomas@apple.com', 'Apple', '804 Fourth Place', 'Brownsville', 'TX', '78521', '869-582-9955', '1988-08-26', 'F', '2018-10-21 22:01:57'), 
    ('Jessica', 'Jackson', 'jessicajackson@aramark.com', 'Aramark', '235 Pine Place', 'Chicago', 'IL', '60629', '587-334-1054', '1991-07-22', 'F', '2015-08-28 03:11:35'), 
    ('James', 'Martinez', 'jamesmartinez@kroger.com', 'Kroger', '831 Oak St', 'Brownsville', 'TX', '78521', '381-428-3119', '1927-12-22', 'F', '2018-01-27 07:41:48'), 
    ('Christopher', 'Robinson', 'christopherrobinson@ibm.com', 'IBM', '754 Cedar St', 'Pharr', 'TX', '78577', '488-694-7677', '1932-06-25', 'F', '2016-08-19 16:11:31');
```

# Data Types

## Character Types

Some common character types used in Postgres:
 - CHAR(n)
 - VARCHAR
 - VARCHAR(n)
 - TEXT

Differences:
 - CHAR(n) represents **fixed-length character**. Stores a fixed-length string of n characters. If the input string is shorter than n, it is padded with spaces to match the length.
 - VARCHAR(n) represents **variable-length character with limit**. Stores a variable-length string with a maximum length of n. Unlike CHAR(n), it does not pad with spaces.
 - VARCHAR represents **Unlimited variable-length character**. Same as TEXT, except that it allows you to specify constraints like CHECK(length < 255).
 - TEXT represents unlimited text. Stores variable-length strings with no explicit length limit.

## Numeric Types

Some common numeric types used in Postgres:
 - SERIAL (for auto-increment integers; includes SERIAL, BIGSERIAL and SMALLSERIAL)
 - INTEGER (= INT, includes INT, BIGINT and SMALLINT)
 - Floats (this includes DECIMAL = NUMERIC, REAL which has 6 places of precision, DOUBLE PRECISION which has 15 places of precision, and FLOAT = DOUBLE)
 - BOOLEAN (TRUE, FALSE, NULL)
 - DATE/TIME (DATE: '2014-12-24', TIME: TIME [ (precision) ] [ WITHOUT TIME ZONE ] or TIME [ (precision) ] WITH TIME ZONE, TIMESTAMP: '2024-01-02 14:30:00', INTERVAL: INTERVAL [ fields ] [ (precision) ], e.g. '1 year 2 months 3 days 4 hours 5 minutes 6.789 seconds') Note: we can use keyword **EXTRACT** to obtain the year, month and day from the **DATE** type value. The general syntax is:
   ```
   EXTRACT(field FROM source)
   ```

## Change Data Types for Columns

To change the data type for a specified column, use the following syntax:

```
ALTER TABLE table_name
ALTER COLUMN column_name SET DATA TYPE new_data_type [ USING expression ];
-- this is the same as
ALTER TABLE table_name
ALTER COLUMN column_name TYPE new_data_type [ USING expression ];
```

If the new data type is not directly compatible, you must provide a conversion expression (**USING expression**) using the USING clause. For example:

```
ALTER TABLE employees
ALTER COLUMN salary SET DATA TYPE INTEGER USING salary::INTEGER;
```

## Change the Name of Columns

If we would like to rename a column, follow the example:

```
ALTER TABLE sales_item RENAME COLUMN day_of_week TO weekday;
```

## Create New Types

To create a new sex_type that contains two distinct values 'M' and 'F', use the following:

```
CREATE TYPE sex_type AS enum ('M', 'F');
```

When we are creating a new type, we should first drop the existing type of the same name, otherwise an error would occur.

```
DROP TYPE IF EXISTS sex_type CASCADE;
```

## Add a New Column

To add a new column with a specific data type, use the following example:

```
ALTER TABLE sales_item ADD day_of_week VARCHAR(8);
```

## Drop a Column

To drop an existed column, use the following syntax:

```
ALTER TABLE sales_item DROP COLUMN weekday;
```

## TRUNCATE a Table

**TRUNCATE** keyword removes all rows in one operation. The basic syntax is:

```
TRUNCATE [ TABLE ] table_name [ CASCADE | RESTRICT ];
```

Note: there are two keywords CASCADE | RESTRICT:
 - CASCADE: Automatically truncates all tables that have foreign key dependencies on the specified table.
 - RESTRICT (default): Raises an error if any table has foreign key dependencies.

# Index

## Create an Index

The basic syntax for creating an index is:

```
CREATE INDEX index_name
    ON table_name (column_name);
```

More specifically, we could create a b-tree index (also the default type) using

```
CREATE INDEX index_name
    ON table_name USING BTREE (column_name);
```

or hash index using

```
CREATE INDEX index_name
    ON table_name USING HASH (column_name);
```

## Check an Index

We can check the existing index in a table using the following example:

```
SELECT indexname, indexdef
    FROM pg_indexes
    WHERE tablename = 'employees';
```

## Drop an Index

The index can be dropped by

```
DROP INDEX IF EXISTS idx_employees_lastname;
```

# Strings

## String Concatenation

In PostgreSQL, we can concatenate strings using the following methods:
 - ||
   ```
   string1 || string2 || string3
   ```
 - CONCAT
   ```
   CONCAT(string1, string2, string3, ...)
   ```
 - CONCAT_WS (concatenate with separator)
   ```
   CONCAT_WS(separator, string1, string2, ...)
   ```

## Pattern Matching

### SIMILAR TO

In PostgreSQL, **SIMILAR TO** is a pattern matching operator that works similarly to LIKE but supports regular expressions using SQL standard syntax instead of PostgreSQL's POSIX-style regular expressions (used in ~ or ~*). The syntax is as follow:

```
expression SIMILAR TO pattern
```

Wildcards:
 - % – Matches zero or more characters (same as LIKE).
 - _ – Matches exactly one character (same as LIKE).

Character Classes:
 - [abc] – Matches any one character from the list (e.g., a, b, or c).
 - [a-z] – Matches any one character in the range a to z.

Quantifiers:
 - + – Matches one or more occurrences of the preceding element.
 - * – Matches zero or more occurrences.
 - ? – Matches zero or one occurrence.

For example:

```
SELECT first_name, last_name
FROM customer
WHERE first_name SIMILAR TO 'M%';
-- above is the same as
SELECT first_name, last_name
FROM customer
WHERE first_name SIMILAR TO 'M[a-z]*';
```

### LIKE

In PostgreSQL, **LIKE** is a pattern matching operator used to filter rows based on partial string matches. It is commonly used with WHERE clauses to search for patterns in text data.

Basic syntax:

```
expression LIKE pattern
-- or we can use negation
expression NOT LIKE pattern
```

Wildcards:
 - %: Matches zero or more characters.
 - _: Matches exactly one character.
 - []: Matches any single character in the list (not standard). E.g. '[aeiou]%'
 - [^]: Matches any single character not in the list (not standard). E.g. '[^aeiou]%'

Example:

```
SELECT first_name || ' ' || last_name
FROM customer
WHERE last_name LIKE '%oo%';
```

### POSIX Regex Alternative

For more complex patterns, use POSIX regular expressions:
 - Match Case-Sensitive:
   ```
   SELECT 'hello123' ~ '[a-z]+[0-9]+' AS result;
   -- Output: TRUE
   ```
 - Match Case-Insensitive:
   ```
   SELECT 'HELLO123' ~* '[a-z]+[0-9]+' AS result;
   -- Output: TRUE
   ```

# Joins

## INNER JOIN (JOIN)

Purpose: Returns rows only when there is a match in both tables (keywords INNER JOIN = JOIN).

## OUTER JOIN

### LEFT JOIN (LEFT OUTER JOIN)

Purpose: Returns all rows from the left table and matched rows from the right table. If no match, NULL values are returned for the right table.

### RIGHT JOIN (RIGHT OUTER JOIN)

Purpose: Returns all rows from the right table and matched rows from the left table. If no match, NULL values are returned for the left table.

### FULL JOIN (FULL OUTER JOIN)

Purpose: Returns all rows when there is a match in either table. Rows with no match in the other table will have NULL values.

## CROSS JOIN

Purpose: Returns the Cartesian product of the two tables, i.e., all possible combinations. Example:

```
SELECT p.name AS product_name, pt.name AS product_type
FROM product AS p
CROSS JOIN product_type AS pt;
```

## Self JOIN

Purpose: Joins a table to itself. Often used for hierarchical or relationship-based queries.

## NATURAL JOIN

Purpose: Performs a JOIN based on columns with the *same name* in both tables.

## Theta Join and Equi Join

A **theta join** is a join operation in SQL where the condition used to match rows between two tables includes comparison operators other than just equality (=).

An **equi join** is a special case of a theta join where the condition used to match rows only involves equality (=). It is the most common type of join and is often referred to as just INNER JOIN.

# UNION and INTERSECT

## UNION

Purpose: Combines the results of two or more queries into a single result set, *removing duplicates*. The syntax is as follow:

```
SELECT column_list FROM table1
UNION
SELECT column_list FROM table2;
```

If we want to keep duplicates, we can use **UNION ALL**:

```
SELECT column_list FROM table1
UNION ALL
SELECT column_list FROM table2;
```

## INTERSECT

Purpose: Returns only the rows that are common in the result sets of two queries. Syntax:

```
SELECT column_list FROM table1
INTERSECT
SELECT column_list FROM table2;
```

Similarly, if we want to keep duplicates, we could use **INTERSECT ALL**:

```
SELECT column_list FROM table1
INTERSECT ALL
SELECT column_list FROM table2;
```

