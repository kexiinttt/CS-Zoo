# Sentinel

Redis Sentinel monitors a primary and its replicas, detects failures, performs failover, and provides clients with the current primary address.

## Failure Detection

Sentinels use periodic health checks. A primary first becomes subjectively down from one Sentinel's perspective, and can become objectively down when a quorum of Sentinels agrees.

## Failover

The Sentinel cluster elects a leader, selects a suitable replica, promotes it to primary, reconfigures the other replicas, and notifies clients. The old primary is reconfigured as a replica after it returns.

Multiple Sentinels are needed to avoid making a failover decision based on one failed monitor or one network partition.
