Keep your transactions short
============================

It is a common rule.
Large transactions with affection of many tables and rows acquires a lot of locks and causes performance degradation.

Instead of a single huge `UPDATE`, `INSERT`, or `DELETE` organize a seria of short transactions.
It is acceptable in many cases.

Do in your favourite language or by `psql`:

    for i in `seq 0 100`; do \
    psql -p 5432 -h localhost -d DB -U USER -c \
    "UPDATE t SET f = 1 WHERE id BETWEEN $i * 100 AND ($i + 1) * 100 - 1;"; \
    done;

(Note slashes and semicolons.)
