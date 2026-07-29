# Data Consistency

With a read/write cache, synchronous write-back can keep data consistent when transactions are used. With asynchronous write-back, data can be lost.

With a read-only cache, the database and cache are updated separately. The operations have an **order**, and either step can **fail**, so the cache and database can become inconsistent or a read after an update can still return an old value.

## Ordering

The following diagram shows that both "delete the cache first, then update the database" and "update the database first, then delete the cache" can become inconsistent when multiple threads run concurrently.

![Inconsistency: ordering](../pic/10_inconsistency_order.png)

> Thread A executes `UPDATE` while thread B executes `GET`, but the cache and database become inconsistent.

> [!NOTE]
> The solution shown in the upper half is [delayed double deletion](#delayed-double-deletion).
>
> Client B misses the cache, reads the old value from the database, and puts the old value into the cache. Without another deletion, the cache would continue to hold the old value.
>
> Delayed double deletion clears the cache again after Client A finishes updating the database. This removes any stale value written during the interval between Client A's cache deletion and database update completion.

## Failures

The following diagram shows that both operation orders can still return an old value if one step fails.

![Inconsistency: failure](../pic/10_inconsistency_failure.png)

> Execute `UPDATE key "xyz"`, then execute `GET key`, but still receive the old value.

# Solving Inconsistency

## Retry Mechanism

Temporarily place the cache value to delete or the database value to update into a message queue. If the application fails to delete the cache or update the database, it can read the value from the queue and retry.
* Successful operation &rarr; remove the value from the queue to avoid repeating the operation.
* Failed retry &rarr; after a configured number of retries, send a failure message.

## Delayed Double Deletion

When using "delete the cache first, then update the database," a thread can wait for a period after the update and delete the cache again.

See the upper half of the first diagram in the Ordering section.

The problem is that the correct waiting duration cannot be predicted.

```
redis.delKey(X)
db.update(X)
Thread.sleep(N)
redis.delKey(X)
```

# Summary

<table>
    <tr>
        <td>Operation order</td>
        <td>Concurrent?</td>
        <td>Problem</td>
        <td>Symptom</td>
        <td>Solution</td>
   </tr>
    <tr>
        <td rowspan="2">Delete cache first<br>then update database</td>
        <td>No</td>
        <td>✅ Cache deletion<br>❌ Database update</td>
        <td>Old value</td>
        <td>Retry database update</td>
    </tr>
    <tr>
        <td>Yes</td>
        <td>Concurrent read/write between cache deletion and database update</td>
        <td>A concurrent request reads the old database value and puts it into the cache</td>
        <td>Delayed double deletion</td>
    </tr>
    <tr>
        <td rowspan="2">Update database first<br>then delete cache</td>
        <td>No</td>
        <td>✅ Database update<br>❌ Cache deletion</td>
        <td>Old value</td>
        <td>Retry cache deletion</td>
    </tr>
    <tr>
        <td>Yes</td>
        <td>Concurrent read/write between database update and cache deletion</td>
        <td>A concurrent request reads the old cached value</td>
        <td>Less common because cache operations are much faster than database operations</td>
    </tr>
</table>
