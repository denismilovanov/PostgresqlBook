EXPLAIN statement
=================

`EXPLAIN` executes a query (so be careful with `EXPLAIN DELETE`) and shows its plan.

Let's remember the table from [Inserting random data for tests](/dml/inserting_random_data.md).

    CREATE TABLE public.users (
        id bigserial NOT NULL PRIMARY KEY,
        uuid uuid NOT NULL UNIQUE,
        name varchar NOT NULL,
        sex sex NOT NULL,
        avatar_url varchar NOT NULL,
        registered_at timestamptz NOT NULL DEFAULT now(),
        facebook_id bigint NULL,
        rank integer NOT NULL,
        status integer NOT NULL
    );

Restart server and get 10 last users:

    EXPLAIN (analyze on, buffers on)
    SELECT *
        FROM public.users
        ORDER BY registered_at DESC
        LIMIT 10;

Output is:

                                                            QUERY PLAN
    ---------------------------------------------------------------------------------------------------------------------------
     Limit  (cost=4891.96..4891.99 rows=10 width=109) (actual time=91.840..91.842 rows=10 loops=1)
       Buffers: shared hit=3 read=1731
       ->  Sort  (cost=4891.96..5141.96 rows=100000 width=109) (actual time=91.839..91.839 rows=10 loops=1)
             Sort Key: registered_at
             Sort Method: top-N heapsort  Memory: 26kB
             Buffers: shared hit=3 read=1731
             ->  Seq Scan on users  (cost=0.00..2731.00 rows=100000 width=109) (actual time=0.016..64.868 rows=100000 loops=1)
                   Buffers: shared read=1731
     Planning time: 5.364 ms
     Execution time: 92.448 ms

* `Seq Scan on users` tells us that a backend has to scan the whole table in order to get the result;
* `shared read=1731` - the data were read from a disk into shared memory.

Try again:

                                                            QUERY PLAN
    --------------------------------------------------------------------------------------------------------------------------
     Limit  (cost=4891.96..4891.99 rows=10 width=109) (actual time=28.744..28.745 rows=10 loops=1)
       Buffers: shared hit=1731
       ->  Sort  (cost=4891.96..5141.96 rows=100000 width=109) (actual time=28.742..28.742 rows=10 loops=1)
             Sort Key: registered_at
             Sort Method: top-N heapsort  Memory: 26kB
             Buffers: shared hit=1731
             ->  Seq Scan on users  (cost=0.00..2731.00 rows=100000 width=109) (actual time=0.007..8.236 rows=100000 loops=1)
                   Buffers: shared hit=1731
     Planning time: 0.084 ms
     Execution time: 28.768 ms

* `Seq Scan on users` still presents;
* `shared hit=1731` - data are in shared memory now (that is why execution time is 3 times less).

Sequential scan is a slow thing. The main way to speed up the queries is to use indexes.
