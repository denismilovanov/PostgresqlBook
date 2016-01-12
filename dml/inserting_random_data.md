Inserting random data for tests
===============================

`generate_series` is a plain and powerful tool for this.

Users of our social network:

    CREATE TYPE sex AS ENUM ('male', 'female');

    CREATE TABLE public.users (
        id bigserial NOT NULL PRIMARY KEY,
        uuid uuid NOT NULL UNIQUE,
        name varchar NOT NULL,
        sex sex NOT NULL,
        avatar_url varchar NOT NULL,
        registered_at timestamptz NOT NULL DEFAULT now(),
        facebook_id bigint NULL CHECK (facebook_id > 0),
        rank integer NOT NULL CHECK (rank > 0),
        status integer NOT NULL -- REFERENCES dictionaries.users_statuses ...
    );

Insert 100k females:

    INSERT INTO users
        (id, uuid, name, sex, avatar_url, rank, status, registered_at)

        SELECT  -- sequence
                nextval('public.users_id_seq'),
                -- let's form special uuid for test users
                (lpad(to_hex(num), 8, '0') || '-0000-0000-0000-000000000000')::uuid,
                -- name
                (array['Ann', 'Barbara', 'Carol', 'Diana', 'Eve', 'Fabiana', 'Gabriella'])[(random() * 6)::integer + 1],
                -- sex
                'female',
                -- avatar_url
                'http://api.randomuser.me/portraits/med/women/' || ((random() * 80)::integer + 1) || '.jpg',
                -- rank
                (random() * 100)::integer + 1,
                -- status
                1,
                -- registered_at
                now() - interval '1 minute' *  (100000 - num)

            -- from table of integers
            FROM generate_series(1, 100000) AS num;

Ten of them:

     id |                 uuid                 |   name    |                     avatar_url                      |         registered_at
    ----+--------------------------------------+-----------+-----------------------------------------------------+-------------------------------
      1 | 00000001-0000-0000-0000-000000000000 | Barbara   | http://api.randomuser.me/portraits/med/women/55.jpg | 2015-11-04 06:34:01.729804+01
      2 | 00000002-0000-0000-0000-000000000000 | Fabiana   | http://api.randomuser.me/portraits/med/women/76.jpg | 2015-11-04 06:35:01.729804+01
      3 | 00000003-0000-0000-0000-000000000000 | Diana     | http://api.randomuser.me/portraits/med/women/48.jpg | 2015-11-04 06:36:01.729804+01
      4 | 00000004-0000-0000-0000-000000000000 | Barbara   | http://api.randomuser.me/portraits/med/women/48.jpg | 2015-11-04 06:37:01.729804+01
      5 | 00000005-0000-0000-0000-000000000000 | Ann       | http://api.randomuser.me/portraits/med/women/49.jpg | 2015-11-04 06:38:01.729804+01
      6 | 00000006-0000-0000-0000-000000000000 | Barbara   | http://api.randomuser.me/portraits/med/women/75.jpg | 2015-11-04 06:39:01.729804+01
      7 | 00000007-0000-0000-0000-000000000000 | Gabriella | http://api.randomuser.me/portraits/med/women/78.jpg | 2015-11-04 06:40:01.729804+01
      8 | 00000008-0000-0000-0000-000000000000 | Fabiana   | http://api.randomuser.me/portraits/med/women/31.jpg | 2015-11-04 06:41:01.729804+01
      9 | 00000009-0000-0000-0000-000000000000 | Barbara   | http://api.randomuser.me/portraits/med/women/70.jpg | 2015-11-04 06:42:01.729804+01
     10 | 0000000a-0000-0000-0000-000000000000 | Barbara   | http://api.randomuser.me/portraits/med/women/32.jpg | 2015-11-04 06:43:01.729804+01
