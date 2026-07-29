# Persistence

Persistence saves in-memory data to disk so that data can survive a server restart.

Redis has three main persistence methods:
1. AOF (Append Only File)
2. RDB Snapshot
3. `aof-use-rdb-preamble`

---

# AOF

Append Only File persistence stores every **write command** in a file. During recovery, Redis re-executes the commands in order.

The command is executed first, and then the operation is recorded in the file.

| Advantages | Disadvantages |
|---|---|
| No extra validation cost: a failed write does not need to be recorded | Data-loss risk: if recording fails after the write succeeds, recovery may lose the data |
| Does not block the current write | May block later writes |

## Write-Back Policies

A write-back policy determines when in-memory data is written to disk.

| Policy | Timing | Advantage | Disadvantage |
|---|---|---|---|
| Always | Write after every operation | Reliable, with little data loss | Poor performance |
| Everysec | Write once per second through an asynchronous task | | |
| No | Controlled by the operating system | High performance | Low reliability |

Data is first placed in the kernel buffer. The kernel calls `fsync()` to write it to disk; the policies differ in when `fsync()` is called.

## Rewrite Mechanism

Redis cannot record every historical operation and replay all of them forever. The file would grow continuously and recovery would become increasingly expensive.

**AOF rewriting compresses the AOF file.** No matter how many times a key has changed, the rewritten AOF only needs to record its latest in-memory state.

```
SET X 10  => log "X = 10" in AOF
SET X 100 => log "X = 100" in AOF
AOF rewrite => scan memory and log "X = 100" to a new AOF, then replace the old one
```

Rewriting writes a new file first and replaces the old file only after completion, avoiding corruption of the existing AOF.

The work runs in a background child process so the main process is not blocked.

> [!IMPORTANT]
> **Q**: Why use a child process instead of a thread?
>
> **A**: Threads share memory and therefore need locks, which reduces efficiency. A child and its parent share data as read-only; when either side changes data, **COW (Copy On Write)** creates an independent copy. The parent copies its virtual page table for the child, and both page tables initially point to the same read-only physical memory. Physical memory is copied only when data changes.

Rewriting has two problems:
1. If the main process modifies a large key while rewriting is active, COW must copy physical pages. This can take time and block the main process.
2. The data modified by the main process can differ from the child process's snapshot.

Redis addresses the second problem with an **AOF rewrite buffer**. The main process:
1. Executes the command and modifies memory, possibly triggering COW.
2. Appends the command to the normal AOF buffer.
3. Appends the command to the AOF rewrite buffer. This buffer is used only during rewriting and records changes made after the child starts but before it finishes.

After the child scans memory, turns the latest key-value state into commands, and writes the new AOF, it sends an asynchronous message to the main process. The main process then:
1. Writes the rewrite-buffer contents to the new AOF.
2. Replaces the existing AOF with the new file.

> [!WARNING]
> This finalization function blocks the main process.

---

# RDB

RDB records the **actual in-memory data at one point in time**. Recovery is faster because Redis reads the data directly instead of replaying commands.

RDB provides `save` and `bgsave`. `save` takes the snapshot in the main process and blocks it; `bgsave` uses a child process.

## Full Snapshot

RDB is a full snapshot: it persists all in-memory data. This is expensive, so frequent snapshots reduce performance, while infrequent snapshots increase possible data loss.

## Consistency

`save` provides strong consistency because only one process is involved and the snapshot blocks changes.

With `bgsave`, COW means the snapshot contains the child process's data. The parent's latest changes are not included in that snapshot, so it represents an earlier point in time.

---

# `aof-use-rdb-preamble` (Hybrid Persistence)

Hybrid persistence still uses AOF to record operations, but AOF rewriting works as follows:
1. Use an RDB snapshot to save in-memory data into the AOF.
2. Save the main process's latest operations during the snapshot in the AOF rewrite buffer.
3. After the RDB data is written, replace the old AOF with a new file containing both the RDB data and AOF commands.

```
------------
|          |
| RDB data |
|          |
------------
|          |
| AOF cmds |
|          |
------------
```

---

# AOF vs. RDB

||AOF|RDB|
|---|---|---|
|**Concept**|Records operations|Records data|
|**Persistence frequency**|Records the process, so it can run frequently with lower cost|Records memory, so it is expensive and should not run too frequently|
|**Data loss**|Frequent recording makes loss less likely|Infrequent snapshots make loss more likely|
|**Recovery**|Slow; commands must be executed|Fast; data is loaded directly|
|**Storage cost**|Small, with rewriting|Large, because every snapshot is full|
|**Process blocking**|COW and the rewrite-buffer finalization function|`save`, or COW under `bgsave`|
