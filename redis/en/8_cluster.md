# Redis Cluster

Redis Cluster distributes keys across 16,384 hash slots. Each primary owns a subset of the slots, and replicas provide failover capacity.

## Hash Slots

The slot is calculated from the key. Hash tags can force related keys into the same slot, which is required for multi-key commands involving those keys.

## Communication and Failover

Cluster nodes exchange membership and health information. A failed primary can be replaced by a suitable replica after the cluster reaches the required failure consensus.

## Resizing and Access

Adding or removing nodes moves hash slots between primaries. A client follows `MOVED` and `ASK` redirections when a key is on another node or is temporarily migrating.
