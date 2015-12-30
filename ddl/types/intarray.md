intarray
========

The `intarray` extension allows you to perform fast operations with columns of type `integer[]`.

It is useful for processing of list of homogenic data represented by integer identifiers (interests, friends, you name it).

From superuser:

    CREATE EXTENSION intarray;

From application user:

    CREATE TABLE users (
        id bigserial PRIMARY KEY,
        name varchar NOT NULL,
        interests_ids integer[] NOT NULL
    );

    INSERT INTO users
        (name, interests_ids)
        VALUES
        ('Alice',   array[1, 2, 3, 4, 5]),
        ('Bob',     array[4, 5, 6, 7, 8]),
        ('Eve',     array[1, 8, 9, 10, 11]);

Let's assume you maintain list of interests in other table, and 1, 2, ..., 11 - are ids from it.

Operator `&&` tells whether 2 arrays intersect (have at least one element in common):

    SELECT  name,
            CASE WHEN interests_ids && array[1] THEN 'contains int. 1'
                 ELSE 'does not contain int. 1'
            END
        FROM users
        ORDER BY id;

Outputs:

     name  |          case
    -------+-------------------------
     Alice | contains int. 1
     Bob   | does not contain int. 1
     Eve   | contains int. 1

Operator `&` gets the result of intersection:

    WITH my_interests AS (
        SELECT interests_ids
            FROM users
            WHERE id = 1
    )
    SELECT  u.name,
            icount(my_interests.interests_ids & u.interests_ids) AS "common interests with me"
        FROM users AS u, my_interests
        WHERE u.id != 1
        ORDER BY u.id;

This outputs:

     name | common interests with me
    ------+--------------------------
     Bob  |                        2
     Eve  |                        1


