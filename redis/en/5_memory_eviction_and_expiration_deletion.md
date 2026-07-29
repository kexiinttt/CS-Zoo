# Memory Eviction

When Redis stores too much data, it must evict some entries, even if those keys have not expired.

## Eviction Policies

* Do not evict data
  * `noeviction` &rarr; new writes fail with an out-of-memory error
* Evict data
  * Evict among keys with an expiration time
    * `volatile-random` &rarr; evict randomly
    * `volatile-ttl` &rarr; prefer keys with the nearest expiration time
    * `volatile-lru` &rarr; prefer the least recently used keys
    * `volatile-lfu` &rarr; prefer the least frequently used keys
  * Evict among all keys
    * `allkeys-random` &rarr; evict randomly
    * `allkeys-lru` &rarr; prefer the least recently used keys
    * `allkeys-lfu` &rarr; prefer the least frequently used keys

## LRU

Least Recently Used evicts the key that has not been used for the longest time. A typical implementation uses a linked list: move a key to the head whenever it is accessed, then evict from the tail.

A linked list has problems:
1. Redis emphasizes memory efficiency, while linked lists require considerable extra space.
2. Frequent Redis access would constantly move list nodes.

Redis therefore uses **approximate LRU**. Each key stores a field for its last-access time. During eviction, Redis randomly samples a number of keys and evicts the one with the oldest access time.

## LFU

Least Frequently Used evicts the key with the fewest accesses over a period of time.

Each Redis key stores two additional fields:
* `ldt` &rarr; the last-access timestamp
* `logc` &rarr; an access-frequency score; a smaller value is easier to evict

`logc` is not the literal access count. It is a frequency score that **decays over time**.

> [!NOTE]
> Suppose two keys are created at the same time. One is accessed after a short interval and the other after a long interval. Their access counts are equal, but their frequencies differ.
>
> The longer the interval, the lower the frequency. `logc` decays more and becomes smaller, making the key easier to evict.

1. When a key is created, `logc` receives an initial value and `ldt` records the current timestamp.
2. On the next access, Redis uses the difference between the current time and `ldt` to determine how much `logc` decays, then updates both values.
3. Redis increases `logc` with a probability inversely related to its current value. A key with a high frequency is therefore less likely to gain another frequency point.

---

# Expiration Deletion

Redis can set an expiration time for a key:
* `EXPIRE <key> <n>` &rarr; expire after `n` seconds
* `PEXPIRE <key> <n>` &rarr; expire after `n` milliseconds
* `EXPIREAT <key> <n>` &rarr; expire at timestamp `n`, to the nearest second
* `PEXPIREAT <key> <n>` &rarr; expire at timestamp `n`, to the nearest millisecond

## How Redis Checks Expiration

Redis maintains a dedicated **expiration dictionary** containing key expiration times.

When a key is accessed, Redis looks up its expiration time in O(1) through a hash table and compares it with the current time to determine whether the key is still valid.

## Expiration Strategies

* **Scheduled deletion** &rarr; create a timer when the expiration is set and delete the key when it fires.
* **Periodic deletion** &rarr; periodically sample keys, check their expiration, and delete expired keys.
* **Lazy deletion** &rarr; check expiration only when the key is accessed.

| Strategy | Advantage | Disadvantage |
|---|---|---|
| Scheduled | Deletes immediately after expiration and is memory-friendly | Frequent timers consume CPU |
| Periodic | Reduces both CPU and memory pressure through periodic work | A compromise: too frequent resembles scheduled deletion, too infrequent resembles lazy deletion |
| Lazy | Checks only on access and is CPU-efficient | An unaccessed expired key continues occupying memory |

Redis uses **lazy deletion plus periodic deletion**. This balances CPU usage and memory waste. Lazy deletion checks the key during access and deletes it synchronously or asynchronously depending on configuration. Periodic deletion works as follows:
1. Randomly sample `n` keys.
2. Delete the `x` expired keys among them.
3. If `x/n > 25%`, immediately repeat step 1; otherwise wait for the next cycle.

> [!NOTE]
> To prevent an endless immediate deletion loop from blocking Redis, the immediate loop also has a time limit.
