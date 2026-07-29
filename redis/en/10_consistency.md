# Data Consistency

Cache and database state can diverge when reads, writes, failures, or delayed operations interleave. The correct strategy depends on whether the cache is authoritative and how much stale data the application can tolerate.

## Retry and Invalidation

Retries can handle transient failures, but they must be bounded and idempotent. Delayed double deletion invalidates the cache, updates the database, and invalidates the cache again after a delay so an overlapping stale read is less likely to repopulate old data.

## Summary

There is no universal cache consistency guarantee from invalidation alone. Define the ordering, failure behavior, retry policy, and acceptable staleness explicitly, then monitor invalidation failures and database load.
