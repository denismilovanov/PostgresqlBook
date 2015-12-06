C for Consistency
=================

Consistency is integrity + business logic restrictions.

Assume that user 3 exists, and user 1000000 does not.

    BEGIN;

    INSERT INTO acid SELECT 3, 500; -- correct

    INSERT INTO acid SELECT 1000000, 500;
    ERROR:  insert or update on table "acid" violates foreign key constraint "acid_user_id_fkey"

    COMMIT; -- server will answer ROLLBACK however

Server denies whole transaction because 2nd query violates foreign key.

And we see:

    TABLE acid;
     user_id | money
    ---------+-------
           1 |    50
           2 |     0


