# Features

The features below are introduced in detail in the following tutorials. They are listed here as a quick reference.

* High performance
  * In-memory database
  * Single-threaded asynchronous I/O, avoiding multi-thread contention and blocking
* Multiple data structures &rarr; String, Hash, List, ZSet, Sorted Set, and more
* Persistence
  * RDB snapshot &rarr; writes a binary representation of memory to disk
  * AOF (append-only file) &rarr; records commands
* High availability
  * Primary-replica architecture
  * Sentinel
  * Cluster
* Common uses
  * Message queues
  * Distributed caching
  * Distributed locks
* Transactions
* Lua scripts and atomic operations

---

# Comparison

||Redis|Memcached|MongoDB|Elasticsearch|
|---|---|---|---|---|
|**Data structures**|Rich|String key-value pairs|Basic types plus JSON|Documents represented as structured JSON|
|**Persistence**|AOF/RDB|No persistence; everything is in memory|WAL (write-ahead log)|Persistence, real-time search, indexing, and data-stream processing|
|**Memory management**|LRU/LFU eviction|No eviction mechanism|Not an in-memory database|Not an in-memory database|
|**Multithreading**|Single-threaded model with asynchronous I/O and no lock contention|Multithreaded model|N/A|N/A|
|**Consistency and availability**|Primary-replica replication, Sentinel, and Cluster|N/A|Replicas and sharding|Replicas and sharding|
