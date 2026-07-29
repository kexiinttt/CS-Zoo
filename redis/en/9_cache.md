# Caching

## Cache Update Strategies

Cache Aside reads from the cache first and loads from the database on a miss; writes update the database and then invalidate or refresh the cache. Read/Write Through delegates reads and writes to the cache layer. Write Back delays database writes and requires stronger failure handling.

## Cache Types

Read-only caches primarily accelerate reads. Read/write caches also participate in updates and therefore require a clear consistency policy.

## Cache Misses and Failures

Cache avalanche means many keys expire together. Cache breakdown means a hot key expires and many requests hit the database at once. Cache penetration means requests repeatedly query keys that do not exist. TTL jitter, request coalescing, null caching, rate limits, and protection of hot keys help mitigate these problems.

For ordinary cache-aside writes, update the database before invalidating the cache. See [Consistency](./10_consistency.md) for the concurrency details.
