psql console
============

`psql` should be your new friend.

From `root` shell:

    # su postgres
    $ psql

`psql` without params connects to default port `5432` via socket (not tcp):

    psql (9.4.5)
    Type "help" for help.

    postgres=#

Let's create a database:

    CREATE DATABASE my_db
        ENCODING = 'UTF8'
        LC_CTYPE = 'en_US.UTF-8'
        LC_COLLATE = 'en_US.UTF-8'
        TEMPLATE = template0;

    CREATE USER my_db_user
        LOGIN
        ENCRYPTED PASSWORD '1pQ_tZOh'
        ; --SUPERUSER;

    ALTER DATABASE my_db
        OWNER TO my_db_user;

    \q

Now:

    nano ~/.pgpass

Add a line:

    *:5432:my_db:my_db_user:1pQ_tZOh

Do not forget to run `chown`:

    chmod 0600 ~/.pgpass

Then (`-h` specifies host, `-p` - port, `-U` - user, `-d` - database):

    psql -h localhost -p 5432 -U my_db_user -d my_db

Now we may use all features:

List of databases - `\l`:

    my_db=> \l

                                                        List of databases
              Name          |          Owner           | Encoding |   Collate   |    Ctype    |   Access privileges
    ------------------------+--------------------------+----------+-------------+-------------+-----------------------
     my_db                  | my_db_user               | UTF8     | en_US.UTF-8 | en_US.UTF-8 |


Connection - `\c`:

    my_db=> \c my_db

    You are now connected to database "my_db" as user "my_db_user".

Arbitrary queries:

    CREATE TABLE users (
        id bigserial NOT NULL PRIMARY KEY,
        uuid uuid NOT NULL,
        name varchar NULL
    );

Table definition - `\d`:

     my_db=> \d users

                                      Table "public.users"
     Column |       Type        |                     Modifiers
    --------+-------------------+----------------------------------------------------
     id     | bigint            | not null default nextval('users_id_seq'::regclass)
     uuid   | uuid              | not null
     name   | character varying |
    Indexes:
        "users_pkey" PRIMARY KEY, btree (id)

Extensions - `\x`:

    my_db=> \x

                     List of installed extensions
      Name   | Version |   Schema   |         Description
    ---------+---------+------------+------------------------------
     plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language


Type `\?` to get the ultimate guide.
