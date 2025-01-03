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

The function does not need to have parameters to pass to it, nor needs to return anything (in this case, it actually returns *void*).

```
-- if the state of the sales person is null, we set it to 'PA'
CREATE OR REPLACE FUNCTION fn_update_employee_state()
RETURNS void -- do not need to return anything, so write void
AS
$$
    UPDATE sales_person
    SET state = 'PA'
    WHERE state IS NULL
$$
LANGUAGE SQL;
```

