# Memory Eviction and Expiration Deletion

## Memory Eviction

When Redis reaches its configured memory limit, its eviction policy determines which keys may be removed. Policies include no eviction, all-keys LRU or LFU, and LRU, LFU, or random eviction among keys with a TTL.

LRU approximates least-recently-used behavior using sampled keys. LFU tracks access frequency and is useful when frequently accessed keys should remain cached.

## Expiration Deletion

Redis stores an expiration time for a key. It removes expired keys lazily when they are accessed and actively by periodically sampling keys with expiration times. Expiration is therefore not guaranteed to happen at the exact deadline.
