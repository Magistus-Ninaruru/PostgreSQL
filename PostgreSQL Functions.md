# SQL Functions

Functions can be created using SQL statements in PostgreSQL, but they are slightly different from PL/pgSQL functions. These functions are known as **SQL functions** and are simpler, focusing primarily on single queries rather than procedural logic.

The basic syntax is as follow:

```
CREATE FUNCTION function_name(parameter1 data_type_1, parameter2 data_type_2, ...)
RETURNS return_data_type
AS
$$
    SQL QUERY
$$
LANGUAGE sql;
-- above is the same as
CREATE FUNCTION function_name(parameter1 data_type_1, parameter2 data_type_2, ...)
RETURNS return_data_type
AS
'
    SQL QUERY
'
LANGUAGE sql;
```

