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

