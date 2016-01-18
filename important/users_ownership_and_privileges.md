Users, ownership, and privileges
================================

PostgreSQL allows you to create different database users (or roles).

psql session, ORM or direct connection from your application to your server needs a role and a password provided.

    CREATE ROLE front_api_user
        LOGIN
        ENCRYPTED PASSWORD 'password_here';

    CREATE USER owner_user
        -- no need to use LOGIN keyword
        ENCRYPTED PASSWORD 'password_here'
        ; --SUPERUSER;

`CREATE ROLE` is the same as `CREATE USER`, but `CREATE USER` implies `LOGIN`.

See the [full CREATE ROLE specification](http://www.postgresql.org/docs/current/static/sql-createrole.html).

User creates an object (database, schema, table, function, etc) and becomes its owner automatically.

    CREATE DATABASE db
        ENCODING = 'UTF8'
        LC_CTYPE = 'en_US.UTF-8'
        LC_COLLATE = 'en_US.UTF-8'
        TEMPLATE = template0;

    -- in database db
    CREATE SCHEMA main;

    -- in schema main
    CREATE TABLE main.users (
        id bigserial PRIMARY KEY,
        name varchar
    );

Object can have only one owner. Ownership can be transferred.

    ALTER DATABASE db
        OWNER TO owner_user;

    ALTER SCHEMA main
        OWNER TO owner_user;

    ALTER TABLE main.users
        OWNER TO owner_user;

Owner (or superuser) can give (grant) access privileges to database objects he owns:

    GRANT USAGE
        ON SCHEMA main
        TO PUBLIC; -- see below

    GRANT SELECT
        ON TABLE main.users
        TO front_api_user;

Now `front_api_user` can perform `SELECT` from `main.users`.

(The key word `PUBLIC` indicates that the privileges are to be granted to all roles,
including those that might be created later. `PUBLIC` can be thought of as an implicitly defined
group that always includes all roles.)

See the [full GRANT specification](http://www.postgresql.org/docs/current/static/sql-grant.html).

Owner (or superuser) can take away (revoke) access privileges from database objects he owns:

    REVOKE UPDATE
        ON TABLE main.users
        FROM front_api_user;

Now `front_api_user` cannot perform `UPDATE` from `main.users`. (It makes no effect because he couldn't before.)

See the [full REVOKE specification](http://www.postgresql.org/docs/current/static/sql-revoke.html).

There are [default priveleges](http://www.postgresql.org/docs/current/static/sql-alterdefaultprivileges.html).

For example, one may want to deny execution of the functions allowed by default to any user:

    ALTER DEFAULT PRIVILEGES
        REVOKE EXECUTE ON FUNCTIONS FROM PUBLIC;




