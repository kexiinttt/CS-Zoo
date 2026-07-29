# Other Redis Notes

## Thread Model

Redis processes commands sequentially on its main execution path. Its speed comes from in-memory data access, efficient data structures, and I/O multiplexing rather than from running every command on a separate thread.

## Persistence and High Availability

The notes in this section cover AOF buffering and rewriting, primary-replica consistency, failover, expired keys, and Redis Cluster hash-slot behavior.

## Common Applications

Topics include delayed queues, transactions, pipelines, distributed locks and Redlock, and the operational impact and deletion strategies for large keys.
