$LITERAL$ notation, EXECUTE
===========================

$LITERAL$ notation is just a way to get a string:

    SELECT $S$Hello 'world' $S$ AS s;

Output:

          s
    ---------------
     Hello 'world'

(Note that you don't need to escape `'`)

Another example:

    SELECT $$CREATE TABLE table_$$ || num || $$ (id bigserial PRIMARY KEY); $$ AS query
        FROM generate_series(1, 3) AS num;

Output:

                           query
    ---------------------------------------------------
     CREATE TABLE table_1 (id bigserial PRIMARY KEY);
     CREATE TABLE table_2 (id bigserial PRIMARY KEY);
     CREATE TABLE table_3 (id bigserial PRIMARY KEY);

`EXECUTE` performs an arbitrary query passed as a string. Let's combine `EXECUTE` and `$$`:

    -- creates stats_1_new like stats_1
    -- renames stats_1 to stats_1_old
    -- renames stats_1_new to stats_1
    -- all with the PKs
    CREATE OR REPLACE FUNCTION rotate_table (
        i_num integer
    )
    RETURNS void AS
    $BODY$
    BEGIN

        EXECUTE $$

        --
        CREATE TABLE stats_$$ || i_num || $$_new (LIKE stats_$$ || i_num || $$ INCLUDING ALL);
        --
        ALTER TABLE stats_$$ || i_num || $$ RENAME TO stats_$$ || i_num || $$_old;
        ALTER INDEX stats_$$ || i_num || $$_pkey RENAME TO stats_$$ || i_num || $$_old_pkey;
        --
        ALTER TABLE stats_$$ || i_num || $$_new RENAME TO stats_$$ || i_num || $$;
        ALTER INDEX stats_$$ || i_num || $$_new_pkey RENAME TO stats_$$ || i_num || $$_pkey;

        $$;

    END
    $BODY$
    LANGUAGE plpgsql VOLATILE;




