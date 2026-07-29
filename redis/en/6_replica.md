# Replication

## Primary-Replica Replication

A replica connects to the primary, performs an initial synchronization, and then receives the primary's write stream. Replicas can serve read traffic and reduce pressure on the primary, but replication is asynchronous by default.

## Partial Resynchronization

The primary keeps a replication backlog. If a replica disconnects briefly and its replication offset is still available in that backlog, it can receive only the missing commands instead of transferring the full dataset again.

The backlog size should reflect the expected write rate and the longest interruption that should be recovered through partial resynchronization.
