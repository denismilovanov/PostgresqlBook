D for Durability
================

Transaction 1 says:

    BEGIN;
    INSERT INTO acid SELECT 4, 1000;
    COMMIT;

Right after this there happens a big problem:

    sudo pkill postgres;
    
Transaction 1 continues:

    TABLE acid;
    
And gets:

    The connection to the server was lost. Attempting reset: Failed.
    
Let's start server again:

    sudo service postgresql start;
    
Now transaction sees committed data:
    
    TABLE acid;
     user_id | money 
    ---------+-------
           1 |    50
           2 |     0
           3 |   100    
           4 |  1000  -- last committed data
    
Committed data are never (1) lost. This is gained by Write-Ahead Log maintained.


###Notes:

1. Also see `synchronous_commit` option.
    
