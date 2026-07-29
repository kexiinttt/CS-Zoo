# Sentinel

Sentinel adds a monitoring cluster on top of the **primary-replica model**. If the primary fails, an automated process selects a replica and promotes it to handle client requests.

![Overview](../pic/7_overview.png)

The typical Sentinel workflow is:
1. Clients do not connect directly to the primary as they would in a simple primary-replica setup. They connect to Sentinel, which tells them the current primary address.
2. Clients subscribe to Sentinel channels through **Pub/Sub** to receive updated state.
3. Clients connect to the current primary and work normally.
4. If a failure occurs, Sentinel performs failover, selects a replica as the new primary, and tells clients through a channel.
5. Clients connect to the new primary and resume normal work.

# Failure Detection

Sentinel periodically pings primaries and replicas. If it receives an ACK within `down-after-milliseconds`, the node is considered healthy; otherwise it is **subjectively down**.

> [!NOTE]
> A primary has two possible states: **subjectively down** and **objectively down**. Network delay or packet loss means one Sentinel cannot immediately conclude that the primary has failed.
>
> When a Sentinel detects subjective failure, it sends `is-master-down-by-addr` to other Sentinels. When the number of Sentinels acknowledging that the primary is down exceeds `quorum`, the primary becomes objectively down and should be replaced.

# Selecting a Sentinel

Failure detection requires a cluster vote. After the cluster agrees that the primary is subjectively down, which Sentinel should perform the promotion?

This resembles the **Raft algorithm**, which can be used for leader election.

> [!IMPORTANT]
> Only the Sentinel cluster uses a Raft-like election. The primary-replica structure itself does not need Raft.
>
> Raft attempts to select a leader among nodes without external coordination. Here, the Sentinel cluster monitors and controls the primary-replica structure, so the data nodes do not need to negotiate directly.

1. Every Sentinel that votes the primary subjectively down becomes a candidate; only candidates can become leader.
2. A candidate broadcasts a request to all other Sentinels and votes for itself.
3. Each Sentinel votes for only one candidate in a round.
4. A candidate wins if it receives (1) a majority and (2) more than `quorum`. Otherwise another round begins. Each Sentinel has a timer and starts another round if it does not receive enough votes within the configured time.

> [!IMPORTANT]
> `quorum` should be an **odd number greater than half of the cluster**:
> * An even number can waste time during elections. Two candidates may repeatedly split the votes evenly.
> * A value no greater than half can incorrectly declare a primary objectively down. Objective failure uses `quorum`, while leader election requires `max(quorum, half)`.

# Failover

Primary-replica failover has five steps:
1. Select a new primary from the old primary's replicas.
2. Promote that replica.
3. Point the other replicas to the new primary.
4. Send the new primary's information to clients through Pub/Sub.
5. Monitor the old primary and turn it into a replica when it returns.

### Select a New Primary

**Step 1. Select the highest-priority node**

Replicas can have a `replica-priority`. A replica with more memory or lower network latency can receive a higher priority.

**Step 2. Select the node with the most replication progress**

More replication progress means less data needs to be copied. Calculate the missing amount with `master_repl_offset - slave_repl_offset`; the smallest value has retained the most data.

**Step 3. Select the node with the smallest ID**

If the previous criteria cannot select one node, use attributes such as the node ID.

### Promote the Replica

1. Sentinel sends `REPLICAOF NO ONE` to detach the selected node from its primary.
2. Sentinel uses `INFO` heartbeats to read the node's state.
3. When the state changes from replica to standalone and then to primary, the promotion has succeeded.

### Point Replicas to the New Primary

Send `REPLICAOF <new primary IP> <new primary PORT>` to the other replicas.

### Notify Clients

Clients connect to Sentinel before connecting to a Redis primary. During initialization, they subscribe to Sentinel channels through **Pub/Sub**. After the new primary starts working, Sentinel publishes its address so clients can change their connection.

> [!NOTE]
> Sentinel has many channels that allow clients to query or monitor primary and replica state.

### Turn the Old Node into a Replica

When the old node comes back online, send it a `REPLICAOF` command pointing to the new primary.

# Sentinel Cluster

Sentinel clusters are built through **Pub/Sub**.

**How Sentinels discover other Sentinels**

The primary has a channel named `__sentinel__:hello`. When a new Sentinel connects to the primary, it publishes its information to this channel and attempts to discover other Sentinels. They can then learn about one another and form a cluster.

**How Sentinels discover primary-replica topology**

Because the primary knows about its replicas, Sentinels periodically send `INFO` to the primary to retrieve topology information. They can then discover and monitor replicas.

> [!NOTE]
> There are many connections between Sentinel and the primary-replica topology:
> 1. **Sentinel &rarr; primary/replicas**: subscribe to `__sentinel__:hello` to publish its own information and learn about other Sentinels.
> 2. **Sentinel &rarr; primary/replicas**: send `INFO` every 10 seconds to retrieve topology information.
> 3. **Sentinel &rarr; primary/replicas**: send `PING` every second.
> 4. **Sentinel &rarr; Sentinel**: send `PING` every second.
> 5. **Sentinel &rarr; primary**: during failover, send `INFO` every second to track the promotion.
