Table-level locks
=================

Table for experiments:

    CREATE TABLE public.names (
        id bigserial PRIMARY KEY,
        name varchar UNIQUE
    );

An example; transaction 1:

    BEGIN;

    ALTER TABLE public.names RENAME TO names_old;
    -- current lock mode is ACCESS EXCLUSIVE

Transaction 2:

    BEGIN;

    INSERT INTO public.names
        (name)
        VALUES ('Ann');
    -- requested lock mode is ROW EXCLUSIVE
    -- it conflicts with current ACCESS EXCLUSIVE, so transaction waits

Transaction 1:

    ROLLBACK;

Transaction 2 continues;

    INSERT 0 1

Every statement tries to acquire lock of some mode.
If requested mode conflicts with current mode transaction waits.

####ACCESS SHARE

`SELECT` acquires a lock of this mode.

Conflicts only with `ACCESS EXCLUSIVE`.

So `DROP`, many `ALTERs`, `TRUNCATE` stop `SELECT`.

####ROW EXCLUSIVE

`UPDATE`, `DELETE`, and `INSERT` acquire a lock of this mode.

Conflicts with the `SHARE`, and `ACCESS EXCLUSIVE` lock modes.

So `CREATE INDEX`, `DROP`, many `ALTERs`, `TRUNCATE` stop `UPDATE`, `DELETE`, and `INSERT`.

**The main thing to remember**: `SELECT` doesn't stop `UPDATE` and vise versa.

####SHARE UPDATE EXCLUSIVE

`CREATE INDEX CONCURRENTLY` acquires a lock of this mode.

Conflicts with the `SHARE UPDATE EXCLUSIVE`, `SHARE`, and `ACCESS EXCLUSIVE`.

So `CREATE INDEX CONCURRENTLY`, `CREATE INDEX`, `DROP`, many `ALTERs`, `TRUNCATE` stop `CREATE INDEX CONCURRENTLY`.

####SHARE

`CREATE INDEX` acquires a lock of this mode.

Conflicts with the `ROW EXCLUSIVE`, `SHARE UPDATE EXCLUSIVE`, and `ACCESS EXCLUSIVE` lock modes.

So `UPDATE`, `DELETE`, `INSERT`, `CREATE INDEX CONCURRENTLY`, `DROP`, many `ALTERs`, `TRUNCATE` stop `CREATE INDEX`.

`SHARE` **does not conflict** with `SHARE`.

####ACCESS EXCLUSIVE

`DROP`, many `ALTERs`, `TRUNCATE` acquire a lock of this mode.

Conflicts with locks of all modes.

So `SELECT`, `UPDATE`, `DELETE`, `INSERT`, `CREATE INDEX [CONCURRENTLY]`, `DROP`, many `ALTERs`, `TRUNCATE` stop `DROP`, other `ALTERs`, `TRUNCATE`.

Yes, `SELECT` stops `DROP`. Transaction 1:

    SELECT pg_sleep(1000)
        FROM public.names;
    -- acquires ACCESS SHARE

Transaction 2:

    DROP TABLE public.names;
    -- requests ACCESS EXCLUSIVE, waits

Some modes are omitted.

Refer to the [complete list of modes](http://www.postgresql.org/docs/current/static/explicit-locking.html).

