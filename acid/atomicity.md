A for Atomicity
===============

Let's begin with a table:

    CREATE TABLE acid (
        user_id integer NOT NULL PRIMARY KEY REFERENCES users(id),
        money integer NOT NULL
    );

Insert some money:

    INSERT INTO acid SELECT 1, 50;
    INSERT INTO acid SELECT 2, 0;

    TABLE acid;
     user_id | money
    ---------+-------
           1 |    50
           2 |     0

Withdraw and deposit in a transaction:

    BEGIN; -- start a transaction

    UPDATE acid SET money = money - 50 WHERE user_id = 1;
    UPDATE acid SET money = money + 50 WHERE user_id = 2;

    TABLE acid;
     user_id | money
    ---------+-------
           1 |    0
           2 |   50 -- now 2 owns 50

    ROLLBACK; -- cancel 2 operations

    TABLE acid;
     user_id | money
    ---------+-------
           1 |    50
           2 |     0 -- 2 operations were cancelled

Withdraw and deposit form an atomic operation. They are committed or rolled back together.

