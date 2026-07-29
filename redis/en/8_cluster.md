# Cluster Mode

When a distributed system has performance problems, there are generally two solutions:
* Scale up &rarr; improve the performance of one node.
* Scale out &rarr; add more nodes.

Primary-replica mode cannot eliminate the performance limit because every primary and replica stores all the data. Cluster mode provides sharding, so each node needs to store only part of the data.

![Overview](../pic/8_cluster_overview.png)

# Hash Slots

Redis divides the key space into `2^14 = 16,384` slots, with each node managing some of them. Every key entering Redis is hashed according to its key.

> [!NOTE]
> `hash algorithm = CRC16(key) & 0x3FFF (16383)`. The bitwise `&` is used because `x & (2^n - 1) == x % 2^n`.

Sometimes related keys must be placed on the same node. A hash tag does this: add `{tag}` to the key, and the algorithm hashes the tag instead of the entire key.

```
CLUSTER KEYSLOT key1{1}
CLUSTER KEYSLOT key2
CLUSTER KEYSLOT key3{1}
```

# Node Communication

Cluster mode has no primary-replica distinction among all nodes, so nodes are peers. They use the Gossip protocol to communicate.

> Gossip protocol: each node periodically selects some nodes and communicates with them. Information eventually converges, and every node learns the topology and state.
> * `MEET` &rarr; add a new node
> * `PING/PONG` &rarr; heartbeat
> * `FAIL` &rarr; mark a node down

# Failover

**Subjective failure**

During Gossip, a node sends `PING` to another node. If no `PONG` arrives within the configured time, it considers that node subjectively down and marks it `PFAIL`.

**Objective failure**

When nodes exchange Gossip information, if more than half believe a node is subjectively down (`PFAIL`), the cluster enters the objective-failure process and broadcasts the new state.

Because Cluster has no center, any node can broadcast objective failure if:
* More than half of the nodes in its current state table vote that a node is `PFAIL`.
* It also considers that node `PFAIL` itself.

**Primary-replica switch**

If a replica fails, nothing important happens. If a primary fails, failover begins using a Raft-like leader election similar to Sentinel.

# Scaling Out and In

Cluster's most important feature is scale-out, so servers can be added or removed dynamically.

When adding a server, it initially manages no slots. Configuration must transfer slot ownership from existing servers. Before removing a server, its slot ownership must likewise be transferred to other servers.

Slot migration takes time because a slot may contain many keys. During migration, the source marks the slot `MIGRATING` and the destination marks it `IMPORTING`.

# Data Access

A client can connect to any server, primary or replica. The connected node may not manage the slot containing the requested key, so it must redirect the client.

**MOVED redirection**

![Moved](../pic/8_moved.png)

If the key belongs to a slot managed by the connected server, the server returns the result directly. Otherwise it returns a `MOVED` redirection.

**ASK redirection**

![ASK](../pic/8_ask.png)

If the key belongs to a slot managed by the connected server, the result is returned directly. If the key belongs to a slot that is not migrating, Redis returns `MOVED`. If the key's slot is being migrated, Redis returns an `ASK` redirection as shown above.
