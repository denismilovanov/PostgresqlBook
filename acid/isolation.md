I for Isolation
===============

By default transaction isolation level is `READ COMMITTED`. 

This means one transaction does not see uncommited changes of another transactions. This is because they can be rolled back.

Transaction 1 says:

    BEGIN;

    INSERT INTO acid SELECT 3, 100;

Transaction 2 says:

    BEGIN;

    TABLE acid;
     user_id | money
    ---------+-------
           1 |    50
           2 |     0 -- there is no user 3 in this transaction yet

Then transaction 1 says:

    COMMIT;

Transaction 2 says:

    TABLE acid;
     user_id | money
    ---------+-------
           1 |    50
           2 |     0
           3 |   100 -- user 3 appears only after commit in 1st transaction
