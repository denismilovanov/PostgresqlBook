# PostgresqlBook

Structured English version of the document I use for the seminars.

- ACID
    - [Atomicity](/acid/atomicity.md)
    - [Consistency](/acid/consistency.md)
    - [Isolation](/acid/isolation.md)
    - [Durability](/acid/durability.md)
- Another important things
    - [psql console](/important/psql.md)
    - I've said 'COMMIT', now what?(/important/commit.md)
- Data definition, DDL
    - Table
        - [Entities, logs](/ddl/tables/entities.md)
        - Mappings
        - Dictionaries
        - Statistics
    - Useful column types from extensions
        - hstore
        - [intarray](/ddl/types/intarray.md)
    - [CREATE TABLE (LIKE ... INCLUDING ...)](/ddl/create_table_like.md)
    - Constraints
        - Foreign keys
        - Uniques
        - [Checks](/ddl/constraints/checks.md)
    - [DDL is transactional, rotating tables](/ddl/transactional.md)
- Data manipulation, DML
    - [DML statements can return data](/dml/returning.md)
    - DO statement
    - Inserting random data for tests
    - EXPLAIN statement
    - btree indexes
    - Indexing long varchars
    - gin/gist indexes
    - Functional indexes
- Locks
    - Table-level locks
    - Row-level locks
    - Adding a column in large table
    - Deadlocks
    - Advisory locks
- Stored functions
    - VOLATILE
    - SECURITY DEFINER
    - EXCEPTION, correct UPSERT in 9.4
    - $LITERAL$ notation, EXECUTE
    - Deploying
- Extensions
    - PostGIS
    - FDW
- Replication
    - Hot standby
    - Slony
