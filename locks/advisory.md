Advisory locks
==============

An (exclusive, not shared) advisory lock is like a mutex. It allows you to perform a piece of code exclusively.
All concurrent processes wait until you release a lock (it can be done automatically after the end of a transaction, see `xact`).

Being placed at the beginning of transansactions, advisory lock is a way to serialize them,
order strictly one after another, to avoid side affects taking place in a concurrent case.

Let's consider `pg_advisory_xact_lock(key bigint)`.

Transaction 1 calls `pg_advisory_xact_lock`:

    BEGIN;

    SELECT pg_advisory_xact_lock(1);

    SELECT 1;

Transaction 2 calls `pg_advisory_xact_lock` too and hangs:

    BEGIN;

    SELECT pg_advisory_xact_lock(1);
    -- awaiting

Transaction 1 does its job and commits the transaction:

    SELECT 1; -- some work here

    COMMIT;

Transaction 2 unfreezes and continues after this commit.

Instead of using plain integers for `key` you may use built-in function `hashtext`:

    SELECT pg_advisory_xact_lock(hashtext('get_from_queue'));

    SELECT pg_advisory_xact_lock(hashtext('exclusive_number_' || number::varchar))
        FROM ... (source of number)

Ok, now we have a queue:

    CREATE TABLE public.queue (
        id bigserial NOT NULL PRIMARY KEY,
        status integer NOT NULL,
        data jsonb NOT NULL
    );

Let's take some records with status 1 and lock them by updating status to 2:

    -- application makes the 1st query
    SELECT id
        FROM public.queue
        WHERE status = 1
        LIMIT 10
        ORDER BY id;

    -- application makes the 2nd query and passes ids from the 1st query
    UPDATE public.queue
        SET status = 2
        WHERE id IN (...);

It is a very naive approach. Two parallel workers may select the same ids, we have to avoid this.

Let's use `SELECT FOR UPDATE`:

    BEGIN; -- transaction is mandatory

    -- application makes the 1st query within transaction
    SELECT id
        FROM public.queue
        WHERE status = 1
        ORDER BY id
        LIMIT 10
        FOR UPDATE; -- acquire locks

    -- application makes the 2nd query within transaction and passes ids from the 1st query
    UPDATE public.queue
        SET status = 2
        WHERE id IN (...);

    COMMIT;


Now,
* transaction 1 performs `SELECT FOR UPDATE` and gets 10 ids,
* concurrent transaction 2 performs this query too, but hangs, because it gets the same ids, and they are locked (by `FOR UPDATE`),
* transaction 1 performs `UPDATE` and `COMMIT`, locks release,
* after this transaction 2 selects the next 10 ids and locks them.

(Note: since 9.5 we have `FOR UPDATE SKIP LOCKED`.)

The same with the advisory locks:

    BEGIN; -- transaction is mandatory in this case too

    SELECT pg_advisory_xact_lock(hashtext('get_from_queue'));

    -- application makes the 1st query
    SELECT id
        FROM public.queue
        WHERE status = 1
        LIMIT 10
        ORDER BY id;

    -- application makes the 2nd query and passes ids from the 1st query
    UPDATE public.queue
        SET status = 2
        WHERE id IN (...);

    COMMIT; -- xact advisory lock releases

In more complex cases `FOR UPDATE` is not a solution, and advisory locks are very helpful.

Advisory locks are very straightforward and dangerous tool (at least their `xact` versions).

Let's assume you have 100 concurrent transactions:
* they try to acquire an `xact` lock,
* transaction 1 (winner) obtains the lock and works 10ms, 99 transactions waits 10ms,
* then transaction 2 obtains the lock and works 10ms, 98 waits 10 + 10 = 20ms,
* the last transaction (loser) waits 99 * 10ms ~ 1s.

It likely violates the rule of [keeping the transactions short](/important/short.md).
