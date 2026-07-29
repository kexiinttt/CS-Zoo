# Persistence

Redis supports two primary persistence mechanisms: AOF and RDB.

## AOF

Append-only file persistence records write commands. The write-back policy controls when buffered data is flushed: every command offers the strongest durability, while periodic or deferred flushing improves throughput.

AOF rewriting creates a compact representation of the current dataset. During rewriting, new writes are tracked separately so they are not lost.

## RDB

RDB creates a point-in-time snapshot of the in-memory dataset. It is compact and fast to restore, but it can lose changes made after the latest snapshot.

## Hybrid Persistence

With `aof-use-rdb-preamble`, the rewritten AOF begins with an RDB-format snapshot followed by AOF commands. This combines faster loading with better recent-write coverage.

The choice between AOF and RDB depends on the required recovery point, restore time, storage cost, and operational complexity.
