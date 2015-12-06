Entities, logs
==============


    CREATE [UNLOGGED] TABLE public.users AS (
        id bigserial PRIMARY KEY,

        name varchar(255) NOT NULL,
        registered_at timestamptz NOT NULL DEFAULT now(),
        ...
    );

### Types available

- `integer`, `bigint`,
- `varchar`, `text`,
- `date`, `timestamp with time zone` (`timestamptz`),
- `jsonb`.

Arrays of these types also available.

### Sequences

`bigserial` is a pseudotype, it is equal to `bigint DEFAULT nextval(‘users_id_seq’)`.

`users_id_seq` is a sequence. Sequence value is always increasing. Even `ROLLBACK` can't make sequence to decrease its value.

But one can explicitly restart sequence:

    ALTER SEQUENCE logs_id_seq RESTART WITH 100;

One can create 2 sequences in 2 databases by `START WITH` and `INCREMENT BY`:

    CREATE SEQUENCE public.users_id_seq
        START WITH 1 -- or with 2 on the second base
        INCREMENT BY 2
        NO MINVALUE
        NO MAXVALUE
        CACHE 1;

1st database sequence values will be: 1, 3, 5, 7, 9, ...
2nd database sequence values will be: 2, 4, 6, 8, 10, ...

### Primary key

`PRIMARY KEY` is `UNIQUE` + `NOT NULL`.

And uniqueness always assumes creating `btree` index.

One table can contain only 1 PK. Table without PK is very strange thing.

### UNLOGGED

`UNLOGGED` tables working without WAL, so they are faster.

