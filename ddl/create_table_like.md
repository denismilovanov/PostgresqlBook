CREATE TABLE (LIKE ... INCLUDING ...)
=====================================

One can create a table using another table as a template.

### Syntax

    CREATE TABLE test (
        id bigint PRIMARY KEY,
        data integer
    );

    CREATE TABLE test_copy (LIKE test INCLUDING ALL);


