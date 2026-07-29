# String

Redis strings are implemented using **SDS (Simple Dynamic String)**. SDS stores the string length to identify its end instead of checking for `\0`.

```
buffer => ['R', 'e', 'd', 'i', 's', '\0', '', '']
len => 5
free => 3
```

`len` records the content length, `free` records the remaining available space, and `buffer` is the pre-allocated memory area.

## Use Cases

### Counter

Because Redis executes commands atomically on its main thread, an integer value can be used directly as a counter.

### Distributed Lock

A value can be set only when a key does not already exist:
* An existing key means insertion is not allowed because the lock is held.
* A missing key means insertion is allowed because the lock is free.

### Shared Session Information

A user may access different servers after logging in. If session data is kept on one server, the session may be lost when requests reach another server. Storing session data in Redis allows every server to retrieve it from the same place.

---

# Hash

Hashes store objects. Their value is itself a collection of key-value pairs.

When the number of elements and their sizes are small, Redis uses a compact list representation; otherwise it uses a hash table. These structures are described in the next tutorial.

## Use Cases

There are several ways to store an object:
1. Use a String with one key for each property.
2. Use a String whose value is a serialized object.
3. Use a Hash.

```
# method1
SET user:1:name tom
SET user:1:age 18

# method2
SET user:1 "{name: tom, age: 18}"

# method3
HMSET user:1 name tom age 18
```

Method 1 uses too many keys, has low memory efficiency, and does not express a relationship between keys.

Method 2 is suitable for read-only content. Otherwise, every update requires serialization and deserialization.

Method 3 is the most direct, but its underlying representation may use more memory and switching representations can have a cost.

---

# List

Lists support insertion and deletion at both ends. They were implemented using a doubly linked list or ziplist; recent Redis versions use QuickList instead of those two structures.

As with hashes, small collections use a compact representation while larger collections use a doubly linked list.

## Use Cases

### Message Queue

A message queue needs three properties:
1. Message ordering &rarr; insert at one end and pop from the other.
2. Duplicate-message handling &rarr; the producer should include a global message ID, and the consumer should record processed IDs.
3. Reliability &rarr; `BRPOPLPUSH` prevents a message from being deleted immediately after retrieval. It moves the message to a backup list so it can be read again after a failure.

The obvious limitation is that each message in a List can be popped only once, so Lists cannot properly serve multiple consumers.

---

# Set

Like a Python `set`, a Set stores unique values without ordering.

---

# Sorted Set (ZSet)

Compared with a Set, every member also has a score, and members are ordered by score. Its implementation is a ziplist or skip list.

---

# Bitmap

A Bitmap is an array of zeros and ones in which elements are located by offset. Its underlying structure is a String.

## Use Cases

Bitmaps are commonly used to record user logins, check-ins, and similar flags.

---

# HyperLogLog

HyperLogLog is an algorithm for cardinality estimation.
* ✅ &rarr; Even when the number or volume of input elements is extremely large, the space required for cardinality estimation remains fixed and very small.
* ❌ &rarr; The result is probabilistic rather than exact. It only counts cardinality and cannot provide the original data.

> [!NOTE]
> Cardinality means the number of distinct values. For `nums=[1, 2, 3, 4, 2, 3, 1, 5]`, the distinct set is `(1, 2, 3, 4, 5)`.

## Use Cases

To count unique visitors to a web page, submit all user IDs and request the estimated cardinality. There may be a small error, but at a very large scale it can be ignored, such as for millions of clicks where the difference of a few thousand is not important.

---

# Stream

Streams can implement message queues. Redis Pub/Sub cannot persist messages, so messages are lost after a server failure. Using Lists as queues means each message can be consumed only once and cannot properly serve multiple consumers.

> [!NOTE]
> Redis Pub/Sub has these limitations:
> 1. Messages cannot be persisted to disk through RDB or AOF, so they are cleared after a failure.
> 2. It has no acknowledgement mechanism and does not guarantee that a consumer receives a message. It is effectively "fire and forget."
> 3. When messages accumulate, Redis may disconnect the client directly.

Streams address these problems:
1. Like Lists, they preserve message order.
2. A global ID is generated automatically when a message is created.
3. Consumer groups are supported.
4. Streams support acknowledgements. Until an acknowledgement is received, a message remains in the **PEL (Pending Entries List)** so it can be retried after consumption fails.

![Stream](../pic/2_DataType_Stream.png)

Multiple consumer groups can use the same message, but within one group a message is consumed by at most one consumer. When any consumer in a group reads a message, that group's cursor advances.

## Stream vs. Other Message Queues

Compared with a disk-based professional message queue such as Kafka, Redis has these disadvantages:
1. Messages may be lost &rarr; Redis is memory-based. Even with persistence, data may still be lost during a failure.
2. Messages can accumulate &rarr; Redis is memory-based, so a queue cannot grow without limit. Once its limit is reached, old messages are removed.
