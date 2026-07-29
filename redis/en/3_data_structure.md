# Mapping

![Mapping](../pic/3_DataType_DataStructure_Mapping.png)

In newer versions, ListPack replaced ZipList, and QuickList replaced LinkedList.

# SDS (Simple Dynamic String)

## Features

1. Get length in O(1).
2. Avoid overflow &rarr; double capacity below 1 MB; above 1 MB, expand by 1 MB at a time.
3. Binary safe &rarr; termination is determined by the array length rather than `\0`, so more complex content can be stored.
4. Lazy space reclamation &rarr; when a string becomes shorter, allocated space is not immediately released. The `free` area is tracked for later expansion.

## Internal Representation

Strings can use three encodings: `int`, `embstr`, and `raw`.

For `int`, the value is stored directly in the `ptr` field of `redisObject`.
![encoding=int](../pic/3_SDS_Encoding_Int.png)

For a short string, `redisObject` and SDS are allocated in one contiguous block by a single allocation call.
![encoding=embstr](../pic/3_SDS_Encoding_Embstr.png)

For a long string, `redisObject` and SDS are allocated by two calls.
![encoding=raw](../pic/3_SDS_Encoding_Raw.png)

The advantages and disadvantages are:
1. `embstr` reduces the number of allocations and deallocations. Its contiguous memory also improves efficiency.
2. Because its memory is contiguous, it can effectively be treated as **read-only**. When it needs to grow, it must first become `raw`.

---

# LinkedList

The underlying structure of a List is a doubly linked list.
```c
typedef struct ListNode {
  struct ListNode* prev;
  struct ListNode* next;
  void* value;
} ListNode;

typedef struct List {
  ListNode* head;
  ListNode* tail;
  unsigned long len;
  // other functions like: free, dup, match
} List;
```

| Advantages | Disadvantages |
| :---: | :---: |
| Get the previous and next nodes in O(1) | List-node memory is not contiguous |
| Get the total length in O(1) | Every node requires an additional allocation |
| Access the head and tail in O(1) | |
| `void *` stores the value, so a node can hold any type | |

---

# ZipList

ZipList saves memory. It is essentially a **sequential data structure in contiguous memory**, similar to an array that can hold different types.

* ✅ Saves memory
* ❌ Lookup, insertion, and modification are inefficient

It trades time for space and is therefore suitable only when there are few elements or the data is simple.

![ziplist](../pic/3_ziplist.png)
* `zlbytes` &rarr; the total number of bytes occupied by the list
* `zltail` &rarr; the offset of the list tail
* `zllen` &rarr; the number of entries
* `zlend` &rarr; marks the end of the list

Except for the head and tail, whose offsets can be calculated in O(1), finding an element requires an O(N) traversal.

Each entry has three attributes. The space used by `prevlen` and `encoding` depends on the node size:
1. `prevlen` &rarr; the length of the previous node, allowing backward traversal. Its size depends on the previous node's length.
2. `encoding` &rarr; the data type and length.
3. `data` &rarr; the data itself.

> When the previous node exceeds a threshold, the space used by `prevlen` increases.

## Cascading Updates

Inserting or modifying a ZipList entry may require reallocating space.

Assume every node's `prevlen` is exactly at a threshold:
1. Insert a new node before the current node, and the new node exceeds the threshold.
2. The new node changes the current node's `prevlen` size.
3. The larger `prevlen` makes the current node exceed the threshold.
4. The next node's `prevlen` grows, and the process repeats like a domino effect.

---

# QuickList

QuickList is essentially a doubly linked list whose nodes contain ZipLists.

ZipList can suffer cascading updates when the collection is large. QuickList limits the size of each ZipList to reduce that cost. When the current ZipList already contains enough data, a new ZipList is created as another linked-list node.

![quicklist](../pic/3_quicklist.png)

---

# ListPack

![listpack](../pic/3_listpack.png)

The main problem with ZipList is that storing the previous node's size can cause cascading updates. ListPack's `len` records only the current node's size, so updates no longer cascade through later nodes.

---

# HashTable

Redis's hash table has the following structure:
```c
typedef struct HashTable {
  DictEntry **table;  // an array of pointers to DictEntry objects
  unsigned long size;
  unsigned long mask;  // used to calculate the hash index
  unsigned long used;
} HashTable;

typedef struct DictEntry {
  void *key;
  union {
    void* val;
    int i_num;
    double d_num;
  } value;  // objects use void*; constants need no extra allocation
  DictEntry *next;  // chained hashing resolves collisions
} DictEntry
```

## Hash Collisions

### Chained Hashing

Each element of the hash array is a linked list. A collision appends an entry to that list. This is simple, but when collisions are frequent the time complexity approaches O(N).

### Rehashing

Rehashing expands the hash array:
1. Store two hash tables, `table1` and `table2`; `table1` contains the data and `table2` is empty.
2. When rehashing begins, expand `table2` based on `table1`.
3. Rehash the data from `table1` into `table2` and use `table2` for new data.
4. Release `table1` and use `table2` as the new `table1`.

The problem is that step 3 may copy a very large amount of data and block Redis from responding to other requests.

### Incremental Rehashing

Incremental rehashing also uses two hash tables, but does not migrate all of `table1` at once. Each lookup, insertion, deletion, or update migrates the corresponding bucket until `table1` is empty.

1. Lookup, delete, or update &rarr; check `table1` first; migrate and operate if the bucket has content, otherwise check `table2`.
2. Insert &rarr; write only to `table2`.

> [!IMPORTANT]
> Rehash trigger &rarr; `load factor = number of hash-table nodes / hash-table size`.
>
> When the load factor is greater than 1, Redis rehashes as long as it is not persisting. When the load factor is greater than 5, Redis forces rehashing.

---

# Skip List

The average lookup and insertion complexity of a skip list is **O(log N)**.

In practice, a `zset` contains both a hash table and a skip list, supporting efficient **range lookup** and **point lookup**. Insertions update both structures to keep them consistent.

> [!NOTE]
> The hash table is mainly used to obtain an element's score. Most operations still rely on the skip list.

```c
typedef struct zset {
  dict *dict;  // hash table
  zskiplist *zsl; // skip list
} zset;

typedef struct zskiplist {
  zskiplistNode *head, *tail;
  uint_t size;
  int level;  // maximum skip-list level
} zskiplist;

typedef struct zskiplistNode {
  void *value;
  double score;
  zskiplistNode *backward; // previous element

  struct zskiplistLevel {
    zskiplistNode *forward;  // next node at this level
    uint_t span;  // number of nodes crossed to reach the next node
  } level[];
} zskiplistNode;
```

The special part is the `level[]` array in `zskiplistNode`. Each element represents one level: `level[0]` is the first level and `level[1]` is the second.

![skiplist](../pic/3_skiplist.png)

## Skip-List Lookup

Lookup starts at the highest level because its span is largest. If the next node's score is less than the target, move forward and compare again. If the next node's score is greater than the target, descend to the next level and continue.

## Skip-List Insertion

The number of levels strongly affects lookup efficiency. The ideal shape resembles a pyramid, with an approximate **2:1 ratio of nodes between adjacent levels**.

Adjusting levels after every insertion or deletion would create considerable overhead. Redis uses a practical approximation: **generate the level randomly when creating a node**.

> [!NOTE]
> Each new node generates a random number in [0, 1]. If it is less than 0.25, another level is created and the process continues until the number is at least 0.25. Thus higher levels are increasingly unlikely.
>
> The first level always exists, the second has probability 0.25, the third has probability 0.25^2, and level n has probability 0.25^(n-1).

## Why Not Use a Balanced Tree?

* More compact memory usage &rarr; every balanced-tree node contains two pointers, while a skip-list node contains fewer pointers on average.
* Better range lookup &rarr; a balanced tree needs an in-order traversal after finding the lower bound, while a skip list can simply walk forward.
* Simpler implementation &rarr; balanced-tree insertion and deletion require restructuring the tree; a skip list mainly changes pointers.

---

# Integer Set

An integer set is a contiguous memory region:

```c
typedef struct intset {
  uint32_t encoding;
  uint32_t length;
  int8_t contents[];
} intset;
```

The `contents` array depends on `encoding`; for example, `encoding=INT_16` stores `int16_t` values.

## Upgrade

When a newly inserted element requires a wider type, such as inserting an `INT_16` value into an `INT_8` array, the array is automatically upgraded. Redis expands the existing representation rather than creating a separate array, saving space.

![intset](../pic/3_intset.jpg)

> [!IMPORTANT]
> An integer set can be upgraded, but it cannot be downgraded.
