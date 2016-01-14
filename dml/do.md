DO statement
============

`DO` allows you to write complex queries without having stored function explicitly defined.
In other words, `DO` executes an anonymous code block.

`DO` cannot return data (it is very sad).

Statements inside `DO` are performed in a single transaction.

Let's continue with table from [Inserting random data for tests](/dml/inserting_random_data.md)

and with a table:

    CREATE TABLE public.invites_history (
        user_id bigint NOT NULL,
        invited_user_id bigint NOT NULL,
        CONSTRAINT invites_history_pkey PRIMARY KEY(user_id, invited_user_id)
    );

Now we organize chain of invitations:

    DO $$
    DECLARE
        r_she public.users;
        r_prev public.users;
    BEGIN

        -- select some
        FOR r_she IN SELECT *
                        FROM users
                        WHERE sex = 'female'
                        ORDER BY registered_at ASC
                        LIMIT 100

        -- iterate through them
        LOOP
            -- for 2nd, 3rd, 4th ...
            IF r_prev.id IS NOT NULL AND random() > 0.5 THEN
                RAISE NOTICE '% invites %', r_prev.id, r_she.id;

                INSERT INTO invites_history
                    SELECT r_prev.id, r_she.id;
            END IF;

            -- remember previous
            r_prev := r_she;
        END LOOP;

    END$$;

Output:

    ...
    NOTICE:  7 invites 8
    NOTICE:  10 invites 11
    ...

