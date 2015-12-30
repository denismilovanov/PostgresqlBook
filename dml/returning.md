DML statements can return data
==============================

`INSERT`, `UPDATE`, `DELETE` statements can return data (affected rows).
Use keyword `RETURNING` for this.

Table:

    CREATE TABLE users (
        id bigserial NOT NULL PRIMARY KEY,
        uuid uuid NOT NULL,
        name varchar NULL
    );

Insert:

    INSERT INTO users
        (uuid)
        SELECT uuid_generate_v4()
            FROM generate_series(1, 5)

        RETURNING id, uuid; -- it will be a table with 2 columns and 5 rows

Output:

     id |                 uuid
    ----+--------------------------------------
      1 | 1fbe66a3-ef5b-41c0-ae3c-e01e5531790a
      2 | 2eb1780a-7268-49fc-9c94-dab932b6c2b1
      3 | 12ce0d9b-3f54-45c2-b160-00579dc6e0cb
      4 | 1dcc3040-df93-4f16-9956-85a7a6af5861
      5 | 3e863d66-67d4-47c0-aaed-2e1c003905f3
    (5 rows)

More complicated:

    WITH upd AS (
        UPDATE users
            SET name = substring(uuid::varchar from 1 for 8)
            RETURNING *
            -- updated rows becomes temporary table with name 'upd'
    )
    SELECT *
        FROM upd
        WHERE name ~ '^1';

Output:

     id |                 uuid                 |   name
    ----+--------------------------------------+----------
      1 | 1fbe66a3-ef5b-41c0-ae3c-e01e5531790a | 1fbe66a3
      3 | 12ce0d9b-3f54-45c2-b160-00579dc6e0cb | 12ce0d9b
      4 | 1dcc3040-df93-4f16-9956-85a7a6af5861 | 1dcc3040

`RETURNING` can contain expressions:

    DELETE FROM users
        RETURNING md5(name) AS hash, id * 2 AS d;

will output:

                  hash                |     d
    ----------------------------------+----------
     879d5d3473bd056ce0f5885feb1047c6 |        2
     d8db0d5aee8eb51f3a705374847e3272 |        4
     bd20abc1031dcc4f8a3ce8ea592a93bd |        6
     c282eda2129e9f4f6a521e2aadbc8e69 |        8
     f4547fb1e9c63d535903e85f0c850e42 |       10






