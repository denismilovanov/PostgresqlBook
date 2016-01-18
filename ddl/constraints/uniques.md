Uniques
=======

`UNIQUE` is a constraint to support some natural uniquenesses in your database.

Users:

    CREATE TABLE public.users(
        id bigserial PRIMARY KEY,
        name varchar NOT NULL,
        email varchar(255) NOT NULL,
        facebook_id bigint NULL,
        status integer NOT NULL DEFAULT 0,
        -- there are generations of users, each user has a number inside his/her generation:
        generation integer NOT NULL,
        num_inside_generation integer NOT NULL
    );

`email`, `facebook_id`, `generation` with `num_inside_generation` may be considered as natural uniquenesses.

`UNIQUE` is based on `btree` index:

    -- forbid two users share one email (case insensitive)
    CREATE UNIQUE INDEX
        ON public.users
        USING btree(lower(email));
        -- lower is a weak constraint, but is better than plain btree(email)
        -- you may consider some version of normalize_email(email)

    -- forbid two users share one facebook_id
    -- quite simple
    -- facebook_id can be NULL, these records will be skipped
    CREATE UNIQUE INDEX
        ON public.users
        USING btree(facebook_id);

    -- (generation, num_inside_generation) pair must be unique
    CREATE UNIQUE INDEX
        ON public.users
        USING btree(generation, num_inside_generation);

Unique violation throws an exception and forces transaction to be rolled back:

    BEGIN;

    INSERT INTO public.users
        (name, email, generation, num_inside_generation)
        VALUES ('Ann', 'supergirl@gmail.com', 0, 1);
        -- that's ok

    INSERT INTO public.users
        (name, email, generation, num_inside_generation)
        VALUES ('Barbara', 'SUPERGIRL@GMAIL.COM', 0, 2); -- the same email in fact
        -- that's an exception

    COMMIT; -- ROLLBACK

Exception is:

    ERROR:  duplicate key value violates unique constraint "users_lower_idx"
    DETAIL:  Key (lower(email::text))=(supergirl@gmail.com) already exists.
    ROLLBACK

