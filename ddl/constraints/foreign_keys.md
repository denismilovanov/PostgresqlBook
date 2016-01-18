Foreign keys
============

`FOREIGN KEY` is a constraint to keep data integrity in your database.

Users, their statuses and likes (a user has a status, a like involves two users):

    -- statuses
    CREATE TABLE public.users_statuses(
        id integer NOT NULL PRIMARY KEY,
        name varchar NOT NULL
    );

    -- seeds
    INSERT INTO public.users_statuses
        VALUES (1, 'Active'), (2, 'Deleted');

    -- main table
    CREATE TABLE public.users(
        id bigserial PRIMARY KEY,
        name varchar NOT NULL,
        status_id integer NOT NULL DEFAULT 1
    );

    -- likes
    CREATE TABLE public.likes(
        user_id bigint NOT NULL
            -- add foreign key just inside table definition
            REFERENCES public.users (id) ON UPDATE CASCADE ON DELETE CASCADE,
        liked_user_id bigint NOT NULL
            REFERENCES public.users (id) ON UPDATE CASCADE ON DELETE CASCADE,
        -- natural PK
        CONSTRAINT likes_pkey PRIMARY KEY (user_id, liked_user_id)
    );

    -- add a foreign key outside table definition, FK is deferrable
    ALTER TABLE public.users
        ADD CONSTRAINT users_status_fkey
        FOREIGN KEY (status_id) REFERENCES public.users_statuses(id)
        DEFERRABLE INITIALLY DEFERRED;

Now create two users with ids 1 and 2:

    INSERT INTO public.users
        (name)
        VALUES ('Alice'), ('Bob');

Trying to set an unexisting status:

    UPDATE public.users
        SET status_id = 3
        WHERE id = 1;

Exception:

    ERROR:  insert or update on table "users" violates foreign key constraint "users_status_fkey"
    DETAIL:  Key (status_id)=(3) is not present in table "users_statuses".

`users_status_fkey` is marked as `DEFERRABLE`, it means that integrity is delayed (checked right before `COMMIT`, not after `UPDATE`):

    BEGIN;

    UPDATE public.users
        SET status_id = 3
        WHERE id = 1;

    -- UPDATE worked!

    INSERT INTO public.users_statuses
        VALUES (3, 'Suspended');

    COMMIT; -- all is correct

Now Bob likes Alice:

    INSERT INTO public.likes
        (user_id, liked_user_id)
        VALUES (2, 1);

    TABLE public.likes;

     user_id | liked_user_id
    ---------+---------------
           2 |             1

Now someone likes Bob:

    INSERT INTO public.likes
        (user_id, liked_user_id)
        VALUES (3, 2);

    ERROR:  insert or update on table "likes" violates foreign key constraint "likes_user_id_fkey"
    DETAIL:  Key (user_id)=(3) is not present in table "users".

Now Bob disappears:

    DELETE FROM public.users
        WHERE id = 2;

Like disappears too (see `ON DELETE CASCADE` in `public.likes`):

    TABLE public.likes;

     user_id | liked_user_id
    ---------+---------------
    (0 rows)

`REFERENCES` part can reference both `PRIMARY KEY` and `UNIQUE` constraints of another tables (actually, `PRIMARY KEY` is a special case of `UNIQUE`).


