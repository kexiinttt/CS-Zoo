# Primary-Replica Replication

Servers are divided into primaries and replicas:
* Primary &rarr; accepts client reads and writes, and continuously sends write updates to replicas.
* Replica &rarr; accepts client reads and continuously receives write updates from the primary.

![Master Slave](../pic/6_master_slave.png)

Write replication synchronizes newly written primary data to replicas so clients can read fresh data.

> [!IMPORTANT]
> Replication is asynchronous: the primary puts updates into a write buffer.

## Process

![Replica Process](../pic/6_master_slave_replica_process.png)

The overall process is shown above.

**1. Establish a connection**

```bash
# on the replica
REPLICAOF <primary IP> <primary port>
```

The replica sends `PSYNC {runId} {offset}` to request synchronization:
* `runId` &rarr; the server's random UUID. On the first connection the replica does not know the primary's UUID, so it uses `?`.
* `offset` &rarr; the replication progress. It is `-1` on the first connection; see [Partial Replication](#partial-replication).

After receiving `PSYNC`, the primary returns `FULLRESYNC {runId} {offset}` to indicate **full replication**:
* `runId` &rarr; the primary's UUID.
* `offset` &rarr; the primary's current replication progress.

**2. The primary synchronizes data to the replica**

The primary uses `BGSAVE` to `fork()` a process that generates an RDB snapshot, then sends the RDB file to the replica for loading.

> [!NOTE]
> `BGSAVE`, RDB transfer, and RDB loading all take time. The primary may process new writes during that period, so it puts those writes into the **replica buffer**.

After loading the RDB, the replica returns an ACK.

**3. The primary sends writes to the replica**

After receiving the ACK, the primary sends the writes in the replica buffer to the replica so both sides become synchronized.

**4. Long-lived TCP connection**

After the initial connection, the primary and replica maintain a long-lived TCP connection. For every write, the primary updates its data first and then sends the write over TCP to the replica.

## Distributing the Load

Replication requires the primary to (1) generate an RDB file and (2) transfer it. If too many replicas connect directly to the primary:
1. The primary must fork multiple processes for `BGSAVE`, which can block the main process.
2. Transferring a large RDB file to many replicas consumes the primary's bandwidth.

To solve this, configure a "replica of a replica": use one replica as the primary for other replicas. RDB generation and transfer are then distributed across replicas.

```bash
# on replica 2.1
REPLICAOF <replica 2 IP> 6379
```

![Master Slave Proxy](../pic/6_master_slave_proxy.png)

# Partial Replication

If the TCP connection breaks and reconnects, the primary may have processed many writes. Full replication would generate, transfer, and load an RDB every time, which is expensive. Redis therefore uses partial replication when possible.

![Incremental Replica](../pic/6_incremental_replica.png)

The `Slave1` on the left was disconnected too long and lost too much data, so it uses **full replication (`FULLRESYNC`)**. The `Slave2` on the right was disconnected briefly, so it can use **partial replication (`CONTINUE`)**.

Two concepts are important:
* `repl_backlog_buffer` &rarr; a **ring buffer**. When the primary sends a write command to a replica, it also saves that command in the buffer.
* Replication offset &rarr; tracks synchronization progress in the ring buffer.
  * `master_repl_offset` &rarr; the position written by the primary.
  * `slave_repl_offset` &rarr; the position read by the replica.

On reconnection:
1. The replica sends `PSYNC {runId} {slave_repl_offset}`. The primary learns the last position read by the replica.
2. Calculate `offset = master_repl_offset - slave_repl_offset` and let `size = size(repl_backlog_buffer)`.
   * If `offset < size`, new primary data has not overwritten the replica's previous position. Copy the data in `[slave_repl_offset % size, master_repl_offset % size]` into the **replication buffer** and perform partial replication.
   * If `offset >= size`, new primary data has overwritten the old content, so perform full replication.

![Ring Buffer](../pic/6_ring_buffer.png)

## Setting the Buffer Size

To avoid full replication after the network recovers, buffer size should reflect the primary's average write speed and network recovery time:

`buffer_size >= average_network_recovery_time * average_write_speed`
