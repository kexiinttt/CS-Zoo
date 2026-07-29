# Redis Data Types

## String

Strings store text, numbers, or binary data. Common uses include counters, distributed locks, and shared session data. Use atomic commands such as `INCR` for counters and `SET key value NX EX seconds` for a basic lock with an expiry.

## Hash

Hashes store fields and values under one key and are useful for representing objects. They allow individual fields to be read or updated without serializing the whole object.

## List

Lists are ordered collections. `LPUSH` and `BRPOP` can implement a simple queue, although Streams are usually a better choice for durable consumer workflows.

## Set and Sorted Set

Sets contain unique members and support membership and set operations. Sorted sets associate each member with a score and support ranking and range queries.

## Bitmap and HyperLogLog

Bitmaps provide compact bit-level counters and flags. HyperLogLog estimates cardinality with very low memory usage when an approximate count is sufficient.

## Stream

Streams are append-only event logs with IDs, consumer groups, acknowledgements, and pending-message tracking. They provide features that are closer to a message queue than ordinary lists.
