Settings
========

Some important settings developers have to know.

[Complete list](http://www.postgresql.org/docs/current/static/runtime-config.html)

/etc/postgresql/{VERSION}/{CLUSTER}/postgresql.conf

###max_connections

Maximum number of concurrent connections to the database server.
100 by default. Don't forget about pgbouncer' pool size.

###shared_buffers

Amount of memory the database server uses for shared memory buffers, they are shared by all backends. A reasonable starting value for `shared_buffers` is 25% of the memory in your system.

###work_mem

Amount of memory every backend uses for sorts, hashes, aggregations, etc. It is not shared memory. So rough estimation is `(RAM - shared_buffers) / max_connections`. Divide it by 2 or 3 (estimation of sorts or hashes operations per query).
Backend will start using temporary files if this amount is not enough for current operation.

###synchronous_commit

Specifies whether transaction commit will wait for WAL records to be written to disk before the command returns a "success" indication to the client.
In many situations, turning off synchronous_commit for noncritical transactions can provide potential performance benefit without the attendant risks of data corruption.

