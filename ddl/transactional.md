DDL is transactional
====================

All DDL statements (except `CREATE DATABASE` and `CREATE TABLESPACE`) are transactional.

### Usage: rotating tables

Log of events:

    CREATE TABLE rotate_me (
        id bigserial NOT NULL PRIMARY KEY
    );

There are writing transactions:

    INSERT INTO rotate_me VALUES (DEFAULT);
    INSERT INTO rotate_me VALUES (DEFAULT);
    INSERT INTO rotate_me VALUES (DEFAULT);
    ...

It is time to rotate table (`rotate_me` to `rotate_me_old`, `rotate_me_new` to `rotate_me`):

    BEGIN;

    CREATE TABLE rotate_me_new
        (LIKE rotate_me INCLUDING ALL);

    ALTER TABLE rotate_me
        RENAME TO rotate_me_old;  -- now all writing transactions will wait

    ALTER TABLE rotate_me_new
        RENAME TO rotate_me;

    ALTER INDEX rotate_me_pkey
        RENAME TO rotate_me_old_pkey;

    ALTER INDEX rotate_me_new_pkey
        RENAME TO rotate_me_pkey;

    COMMIT;

Renaming is very fast, so transaction won't wait for a long time.

After commit all awaiting transactions continue with the new empty table.

