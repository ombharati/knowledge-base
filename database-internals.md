# Database Internals — Notes

**Source:** *Database Internals: A Deep Dive into How Distributed Data Systems Work*, Alex Petrov. 

## Part I — Storage Engines

# Chapter 1: Introduction and Overview

## 1. Database Workload Types

### Core idea

A database should be evaluated according to its **workload**, not merely its category, popularity, or implementation technology.

| Workload | Main activity                 | Typical queries                          | Primary concern                    |
| -------- | ----------------------------- | ---------------------------------------- | ---------------------------------- |
| **OLTP** | Many user-facing transactions | Short, predefined reads and writes       | Low latency, concurrency           |
| **OLAP** | Analysis over large datasets  | Long scans, aggregations, ad hoc queries | Scan and computation efficiency    |
| **HTAP** | Transactions and analytics    | Mixture of OLTP and OLAP                 | Balancing conflicting requirements |



### Mental model

* **OLTP:** operate on individual items.
* **OLAP:** examine the collection.
* **HTAP:** attempt both without maintaining entirely separate systems.

### Key relationship

```text
Workload
   ↓
Access pattern
   ↓
Physical data layout
   ↓
Suitable storage structure
   ↓
Observed performance
```

### Common mistakes

* Choosing a database because it is labelled “SQL,” “NoSQL,” or “distributed.”
* Assuming one architecture works equally well for transactions and analytics.
* Benchmarking a workload that does not resemble production.

### Quick example

* Updating one customer’s address → OLTP.
* Computing average revenue across five years → OLAP.
* Updating orders while running near-real-time sales reports → HTAP.

### Recall questions

1. Why can a system optimised for OLTP perform poorly on OLAP queries?
2. A workload performs short writes but regularly scans 80% of a table. Which requirements conflict?
3. Why is “relational versus document database” insufficient for predicting performance?

---

## 2. DBMS Architecture

### Core idea

A DBMS transforms a high-level request into physical data operations through several specialised layers.

### Mental model: compiler plus runtime

A database request resembles a program:

```text
Client query
    ↓
Transport
    ↓
Parse and validate
    ↓
Optimise
    ↓
Execution plan
    ↓
Local/remote execution
    ↓
Storage operations
```

Figure 1-1 on PDF page 29 presents this layered flow from client communication down to storage-engine components. 

### Components

| Component                  | Responsibility                                               | Main output                   |
| -------------------------- | ------------------------------------------------------------ | ----------------------------- |
| **Transport subsystem**    | Receives client requests and communicates with cluster nodes | Query/request                 |
| **Query processor/parser** | Parses, interprets and validates the query                   | Internal query representation |
| **Access control**         | Checks whether the requested operation is permitted          | Authorisation decision        |
| **Query optimiser**        | Selects an efficient way to execute the query                | Execution plan                |
| **Execution engine**       | Runs local and remote plan operations                        | Result stream                 |
| **Storage engine**         | Stores, retrieves and manages records                        | Physical data operations      |

### Query optimiser

The same logical query may have many valid physical plans.

The optimiser considers information such as:

* Available indexes
* Estimated cardinalities
* Expected intersection sizes
* Data placement
* Network-transfer costs
* Possible access methods
* Operation ordering

### Cause and effect

```text
Poor estimates
   ↓
Poor execution-plan choice
   ↓
Unnecessary scans, joins or network transfers
   ↓
Higher latency and resource usage
```

### Storage-engine components

| Component               | Role                                                                           |
| ----------------------- | ------------------------------------------------------------------------------ |
| **Transaction manager** | Schedules transactions and preserves logical consistency                       |
| **Lock manager**        | Coordinates concurrent access and protects physical integrity                  |
| **Access methods**      | Organise and locate data using structures such as heaps, B-Trees and LSM Trees |
| **Buffer manager**      | Caches disk pages in memory                                                    |
| **Recovery manager**    | Maintains logs and restores state after failures                               |

Transaction and lock management jointly implement concurrency control: preserve correctness while allowing useful parallelism. 

### Important exception

**Component boundaries are conceptual, not universal rules.**

Real implementations may combine or tightly couple components for:

* Performance
* Edge-case handling
* Simpler coordination
* Historical architectural reasons

### Common mistakes

* Treating the SQL text as the operations the database executes directly.
* Assuming a valid execution plan is necessarily an efficient plan.
* Assuming all databases use identical component boundaries.
* Blaming storage when the main cost is query planning or remote data transfer.
* Treating transaction management and locking as interchangeable concepts.

### Quick example

Query:

```sql
SELECT *
FROM orders
WHERE customer_id = 42;
```

Possible plans:

1. Scan every order and filter.
2. Use an index on `customer_id`.
3. Ask another cluster node that owns the relevant partition.
4. Use an index, then fetch complete records from a separate data file.

The query meaning is unchanged, but the physical cost can differ substantially.

### Recall questions

1. Why must access-control checks occur after at least part of the query is interpreted?
2. How can inaccurate cardinality estimates produce a slow execution plan?
3. Which DBMS component is responsible when data is recovered after a crash?
4. Why might a distributed query optimiser consider network cost?
5. What correctness responsibilities differ between transaction and lock managers?

---

## 3. Memory-Based vs Disk-Based Databases

### Core idea

The primary storage medium changes more than latency. It affects data structures, layout, recovery, cost and implementation strategy.

| Property              | Memory-based DBMS           | Disk-based DBMS                   |
| --------------------- | --------------------------- | --------------------------------- |
| Primary data location | RAM                         | Persistent storage                |
| Disk usage            | Logging and recovery copies | Main storage                      |
| Memory usage          | Main working representation | Cache and temporary workspace     |
| Random access         | Comparatively cheap         | Comparatively expensive           |
| Durability challenge  | RAM is volatile             | Storage is persistent             |
| Capacity cost         | Higher                      | Lower                             |
| Layout constraints    | More flexible               | Optimised around pages and blocks |



### Mental model

* **RAM:** fast workspace.
* **Disk:** durable archive.
* A memory database makes RAM the authoritative working state but still usually needs disk to rebuild that state.

### Why disk structures differ

Disk access normally occurs in blocks, not arbitrary byte-sized reads.

Therefore disk-oriented structures try to:

* Reduce random I/O
* Read useful data in each block
* Keep trees wide and shallow
* Control serialization
* Manage fragmentation
* Track reusable space
* Handle variable-sized records explicitly

Memory structures can follow pointers cheaply and allocate objects more freely.

### Important exception

An in-memory database is **not simply a disk database with a very large cache**.

A cached disk database still carries disk-oriented constraints:

* Page-oriented layout
* Serialization overhead
* Disk-compatible representations
* Buffer-manager semantics

A purpose-built memory database can use structures and representations designed specifically for RAM. 

### Trade-off

```text
More RAM
   → lower access latency
   → simpler random access

But:

More RAM
   → higher cost
   → greater volatility exposure
   → stronger recovery requirements
```

### Common mistakes

* Assuming “in memory” means no disk is involved.
* Treating cache hits in a disk database as equivalent to native in-memory storage.
* Comparing only latency while ignoring capacity cost and recovery.
* Designing disk structures around pointer-heavy random access.

### Quick example

A bounded inventory dataset may fit entirely in RAM. Reads become fast, but a power failure would destroy the working state unless it can be reconstructed from durable logs and backups.

### Recall questions

1. Why do disk databases prefer wide, shallow trees?
2. Why does fitting the dataset in RAM not remove the need for durable storage?
3. Which disk-oriented overheads remain when all pages happen to be cached?
4. When could the cost of RAM outweigh its latency advantage?

---

## 4. Durability in Memory-Based Stores

### Core idea

A memory database becomes durable by maintaining enough persistent information to reconstruct its volatile state.

### Recovery model

```text
Client update
    ↓
Append update to durable sequential log
    ↓
Acknowledge operation
    ↓
Apply logged updates to disk backup asynchronously
    ↓
Create checkpoint
    ↓
Discard log portion already included in checkpoint
```



### Key concepts

#### Write-ahead log

The durable log is written before the operation is considered complete.

**Purpose:** preserve the change even if RAM is lost.

#### Backup or snapshot

A disk-resident representation of database state.

**Purpose:** avoid rebuilding the entire database from the beginning of the log.

#### Checkpoint

A point at which the backup includes all log records up to a known position.

**Effect:**

* Older log records become unnecessary.
* Recovery requires replaying only records after the checkpoint.
* Client writes need not wait for every backup update.

### Mental model: save file plus recent actions

* **Checkpoint:** latest saved game.
* **Log:** actions performed since that save.
* **Recovery:** load the save, then replay the remaining actions.

### Trade-off

| Frequent checkpoints | Infrequent checkpoints |
| -------------------- | ---------------------- |
| Faster recovery      | Longer recovery        |
| More background I/O  | Less checkpoint I/O    |
| Smaller retained log | Larger retained log    |

### Common mistakes

* Acknowledging writes before the required log is durable.
* Deleting log records before confirming they are represented in the checkpoint.
* Assuming the backup always reflects the latest committed state.
* Making checkpoint work block client operations unnecessarily.

### Quick example

* Checkpoint contains updates through log entry `10,000`.
* Crash occurs after entry `10,125`.
* Recovery loads the checkpoint and replays entries `10,001–10,125`.

### Recall questions

1. Why is a log needed when a disk backup already exists?
2. What failure occurs if the log is truncated before checkpoint completion?
3. How does checkpoint frequency trade runtime overhead against recovery time?
4. Why are backup modifications often batched?

---

## 5. Row-Oriented vs Column-Oriented Layout

### Core idea

Logical tables do not determine physical layout.

The same table can be stored:

* **Horizontally:** all fields from one row together.
* **Vertically:** all values from one column together.



## Row-oriented storage

### Mental model

Store each record as a complete package:

```text
[ID, name, birth_date, phone]
[ID, name, birth_date, phone]
[ID, name, birth_date, phone]
```

### Works well when

* Most columns of a record are read together.
* Queries retrieve individual records.
* Writes insert or update complete records.
* Workloads contain point lookups or short range scans.

### Why it works

Disk reads bring nearby data into memory. Keeping one row together provides **spatial locality** when the application consumes the whole record.

### Cost

A query reading one column from many rows also loads unwanted columns.

```text
Need: phone numbers
Read: names + dates + phone numbers
Discard: names + dates
```



---

## Column-oriented storage

### Mental model

Store each attribute as a separate stream:

```text
ID:       [1, 2, 3, 4]
Symbol:   [DOW, DOW, S&P, S&P]
Date:     [...]
Price:    [...]
```

### Works well when

* Queries scan many rows.
* Only a subset of columns is needed.
* Workloads compute aggregates.
* Similar values can be compressed together.
* Operations can be vectorised.

### Why it works

Reading only the requested columns reduces irrelevant I/O.

Values of the same type stored together also improve:

* Cache utilisation
* Compression
* CPU vectorisation
* Batch processing

 

### Reconstructing rows

Because columns are stored separately, the system must determine which values belong to the same logical row.

Possible strategies:

* Store an explicit row identifier with each value.
* Use an implicit or virtual identifier based on value position.

#### Trade-off

| Explicit identifier | Positional identifier                    |
| ------------------- | ---------------------------------------- |
| Clear association   | Less duplicated metadata                 |
| Additional storage  | Requires positions to remain coordinated |

### Row vs column decision table

| Access pattern                               | Better starting choice |
| -------------------------------------------- | ---------------------- |
| Fetch one complete customer                  | Row                    |
| Update one order                             | Row                    |
| Scan one field across millions of records    | Column                 |
| Compute average price                        | Column                 |
| Read most columns from a small key range     | Row                    |
| Aggregate three columns over an entire table | Column                 |

### Rule

Choose according to **how data is consumed**, not how it is displayed logically.

### Common mistakes

* Assuming a table-shaped interface implies row-oriented storage.
* Choosing columnar storage merely because the schema has many columns.
* Ignoring row-reconstruction costs.
* Assuming columnar layouts automatically improve point lookups.
* Comparing layouts without considering compression and vectorisation.

### Quick example

Table with 100 columns and 10 million rows:

```sql
SELECT AVG(price) FROM trades;
```

* Row store may read data from all 100 columns.
* Column store can primarily read the `price` stream.

### Recall questions

1. Why does a row store waste I/O during a single-column table scan?
2. How does column storage improve compression?
3. What metadata is required to reconstruct logical rows?
4. Would a query reading 95 of 100 columns necessarily benefit from columnar storage?
5. Why can SIMD processing favour column-oriented layouts?

---

## 6. Column Stores vs Wide Column Stores

### Rule

**Column-oriented databases and wide column stores are different concepts.**

| Column-oriented database               | Wide column store                                |
| -------------------------------------- | ------------------------------------------------ |
| Stores values column-by-column         | Organises columns into families                  |
| Commonly targets analytics             | Commonly targets key-based retrieval             |
| Scans column streams                   | Locates rows by row key                          |
| Physical layout is vertically oriented | Data inside a family is commonly stored row-wise |

Wide column systems such as Bigtable conceptually use a multidimensional sorted map:

```text
row key
  → column family
      → qualifier
          → timestamp/version
              → value
```

Related column families are stored separately, while values belonging to the same row key within a family remain together. Figures 1-3 and 1-4 on PDF pages 36–37 show the conceptual nested map and its physical separation into column-family tables. 

### Mental model

* **Column store:** rearrange a table by attribute.
* **Wide column store:** sparse, versioned map indexed primarily by row key.

### Common mistake

Assuming “wide column” means an analytical columnar layout.

### Quick example

A web-page record may use:

* Row key: reversed URL
* Column family: page content
* Qualifier: content type
* Timestamp: snapshot version
* Value: stored page content

### Recall questions

1. Why is a wide column store still suitable for row-key lookups?
2. What function does a column family serve?
3. How does timestamped storage allow multiple versions?
4. Why should “columnar” and “wide column” not be used interchangeably?

---

## 7. Data Files and Index Files

### Core idea

A database uses specialised file formats to optimise three competing goals:

1. **Storage efficiency** — minimise overhead.
2. **Access efficiency** — locate records with few operations.
3. **Update efficiency** — minimise disk changes.



### Mental model

* **Data file:** contains the actual records.
* **Index file:** contains a faster route to those records.
* An index resembles a book’s index: smaller than the content and designed for navigation.

### Pages

Database files are divided into pages, usually aligned with one or more storage blocks.

```text
Database file
 ├── Page
 │    ├── Record
 │    ├── Record
 │    └── Record
 └── Page
```

Pages are important because storage I/O occurs in units larger than individual fields.

### Updates and deletion

Many storage engines do not immediately erase old data.

Instead:

```text
Old record
    +
New version or tombstone
    ↓
Old record becomes shadowed
    ↓
Garbage collection copies live records
    ↓
Obsolete records are discarded
```

A **tombstone** records that a key was deleted, often with metadata such as a timestamp. 

### Why deferred deletion matters

Immediate removal can require expensive reorganisation.

Deferred reclamation makes foreground writes cheaper but causes:

* Temporary space amplification
* More complex reads
* Background garbage-collection work

### Common mistakes

* Assuming a delete immediately frees disk space.
* Treating pages as equivalent to individual records.
* Ignoring stale record versions when estimating storage use.
* Assuming the filesystem’s directory structure serves as the database index.

### Recall questions

1. Why might a database write a tombstone instead of removing a record immediately?
2. What costs are shifted from foreground deletion to garbage collection?
3. Why are index files normally smaller than data files?
4. How does page size influence I/O behaviour?

---

## 8. Data-File Organisations

### Heap files

Records are stored without a required key order, often in insertion order.

**Advantages**

* Cheap appends
* Little reorganisation during insertion

**Costs**

* Efficient search requires an additional index
* Range scans are not naturally ordered

### Hashed files

A hash of the key selects a bucket.

**Advantages**

* Efficient equality lookup
* Direct bucket selection

**Costs**

* Poor natural support for ordered range scans
* Bucket overflow and distribution must be managed

### Index-organised tables

Records are stored inside the index in key order.

**Advantages**

* No second file lookup after finding the key
* Efficient ordered range scans

**Costs**

* Record movement may affect index structure
* Large records reduce index density



### Comparison

| Organisation    | Physical order          |       Extra index needed |  Equality lookup |         Range scan |
| --------------- | ----------------------- | -----------------------: | ---------------: | -----------------: |
| Heap            | Usually insertion order |                  Usually | Depends on index | Weak without index |
| Hash            | Hash bucket             |  Structure is hash-based |           Strong |               Weak |
| Index-organised | Key order               | Data is in primary index |           Strong |             Strong |

### Quick example

For keys `10, 20, 30`:

* Heap: records may appear as `20, 10, 30`.
* Hash: records are divided by `hash(key)`.
* Index-organised: records appear as `10, 20, 30`.

### Recall questions

1. Why are appends usually cheap in a heap file?
2. Why does hash organisation perform poorly for `WHERE key BETWEEN 100 AND 200`?
3. How does an index-organised table eliminate one disk lookup?
4. What happens to index density when records become large?

---

## 9. Primary, Secondary, Clustered and Nonclustered Indexes

### Primary index

The main index used to identify records, normally associated with a primary key.

### Secondary index

An additional access path using another field.

Multiple secondary indexes may refer to one record.

Unlike a primary key, a secondary search key may have duplicate values.

### Clustered index

The data records follow the index’s search-key order.

```text
Index order: A, B, C
Data order:  A, B, C
```

### Nonclustered index

The index order differs from the physical data order.

```text
Index order: A, B, C
Data order:  C, A, B
```



### Key relationships

* Index-organised tables are clustered by definition.
* Primary indexes are commonly clustered.
* Secondary indexes provide access by keys other than the primary ordering and are therefore nonclustered relative to that data organisation.
* A clustered design may store records directly in the index or in a separate ordered data file.

### Mental model

A database can maintain only a limited number of physical orders.

Every additional secondary index provides another **logical ordering**, but the underlying record cannot be physically colocated according to every index simultaneously.

### Common mistakes

* Assuming “primary index” always means “clustered index.”
* Assuming a secondary key is unique.
* Assuming every index stores the complete record.
* Confusing logical index ordering with physical record ordering.

### Quick example

Employee data physically ordered by `employee_id`:

* Primary clustered index: `employee_id`
* Secondary nonclustered index: `department`
* Searching by department first locates matching references, then retrieves employees from their primary locations.

### Recall questions

1. Why can several secondary-index entries point to one data record?
2. Why can a table not normally be physically clustered by several independent keys?
3. How does a duplicate secondary key affect index entries?
4. Can a clustered index use separate data and index files?

---

## 10. Primary Index as Indirection

### Core idea

A secondary index can locate a record in two ways:

### Direct reference

```text
Secondary index
      ↓
Physical record location
```

### Primary-key indirection

```text
Secondary index
      ↓
Primary key
      ↓
Primary index
      ↓
Record
```

Figure 1-6 on PDF page 41 contrasts these two paths. 

### Trade-off

| Direct physical pointer                           | Primary-key indirection                          |
| ------------------------------------------------- | ------------------------------------------------ |
| Faster read path                                  | Additional primary-index lookup                  |
| Fewer lookup steps                                | Stable logical reference                         |
| Must update pointers when records move            | Fewer secondary-index updates after relocation   |
| Expensive with many indexes and frequent movement | Better suited to write-heavy or reorganised data |

### Cause and effect

```text
Record relocation
   ↓
Direct references become stale
   ↓
Every affected secondary index must be updated
```

With primary-key indirection:

```text
Record relocation
   ↓
Primary index location changes
   ↓
Secondary indexes can retain the same primary key
```

### Hybrid approach

A secondary entry may store:

* A cached physical location
* The primary key as a fallback

The engine tries the fast pointer first. If stale, it resolves through the primary index and refreshes the pointer.

### Common mistakes

* Assuming an extra lookup is always worse.
* Ignoring index-maintenance cost when evaluating read performance.
* Storing physical offsets without considering compaction or page movement.
* Assuming primary keys and physical locations have the same stability.

### Quick example

A table has six secondary indexes.

If a record moves:

* Direct-pointer design may need six pointer updates.
* Indirection design may update only the primary index, while secondary entries retain the primary key.

### Recall questions

1. Why does primary-key indirection increase read cost?
2. Why can it reduce write amplification?
3. Which approach is likely to favour a read-heavy workload with immobile records?
4. How does the hybrid approach detect and repair a stale pointer?

---

## 11. Three Axes of Storage-Engine Design

### Core idea

Many storage structures can be understood through three independent design choices:

1. **Buffering**
2. **Mutability**
3. **Ordering**

These describe important behaviour that the underlying data structure alone does not capture. 

## Axis 1: Buffering

### Question

Does the engine accumulate updates in memory before writing them to their final disk structure?

### Effect

```text
More buffering
   → fewer, larger writes
   → amortised I/O cost
   → delayed propagation
   → more recovery and memory-management complexity
```

Some buffering is unavoidable because disks transfer blocks. The design decision concerns **additional, avoidable buffering**.

---

## Axis 2: Mutability

### Mutable structure

Reads an existing location, changes it and writes the modified content back.

```text
Page P → modify → overwrite Page P
```

### Immutable structure

Previously written content is not modified.

Possible techniques:

* Append a new record version.
* Append a deletion marker.
* Copy the modified page to a new location.

```text
Old Page P remains
New Page P′ is written elsewhere
```

### Trade-off

| Mutable                               | Immutable                            |
| ------------------------------------- | ------------------------------------ |
| Updates existing locations            | Writes new versions                  |
| Less obsolete-version buildup         | Requires reclamation                 |
| Random writes may be expensive        | Sequential writes are possible       |
| Recovery must handle in-place changes | Reads may reconcile several versions |

---

## Axis 3: Ordering

### Ordered storage

Nearby keys are stored in nearby disk regions.

**Benefit:** efficient range scans.

### Unordered storage

Records are often stored in insertion order.

**Benefit:** cheaper writes and appends.

### Trade-off

```text
Maintain key order now
   → more write-time work
   → cheaper ordered reads later

Append without sorting
   → cheaper writes now
   → more read-time reconciliation later
```



### Combined mental model

A storage engine is a point in a three-dimensional design space:

| Axis       | Option A              | Option B                 |
| ---------- | --------------------- | ------------------------ |
| Buffering  | Immediate propagation | Accumulate and batch     |
| Mutability | In-place updates      | Append/copy new versions |
| Ordering   | Key ordered           | Insertion/out-of-order   |

The axes are not strict database categories.

Examples of possible combinations:

* Ordered + mutable + lightly buffered
* Ordered + immutable + heavily buffered
* Unordered + immutable + append-oriented

### Important exception

Do not reduce the distinction to:

```text
B-Tree = mutable
LSM Tree = immutable
```

B-Tree-inspired structures can also use immutable or copy-on-write techniques.

### Common mistakes

* Classifying a storage engine using only its headline data structure.
* Assuming buffering is either completely present or absent.
* Assuming immutable storage removes update costs; it moves them to reconciliation and garbage collection.
* Assuming unordered storage is unsuitable for every workload.
* Optimising writes without accounting for later read amplification.

### Quick example

A write-heavy event system could append events out of order and periodically reorganise them.

* Fast foreground writes
* More background maintenance
* More work for ordered reads before organisation completes

### Recall questions

1. Why is some level of buffering unavoidable for disk storage?
2. How does immutability convert overwrite work into reclamation work?
3. Why does ordering benefit range scans but burden writes?
4. Can a B-Tree-inspired design be immutable?
5. Which axes most directly influence write amplification and read amplification?
6. Design a storage strategy for write-heavy logs that are rarely range-scanned. Which choices would you make on each axis?

# Chapter 2: B-Tree Basics

## 1. Binary Search Trees

### Core idea

A binary search tree partitions the key space at every node:

```text
left subtree  <  node key  <  right subtree
```

Each node contains:

* One key
* Its associated value
* At most two child pointers

The smallest key is found by repeatedly following left pointers; the largest by following right pointers. Values may exist at any tree level. 

### Mental model

Each comparison asks one binary question:

```text
Is the target smaller or larger than this key?
```

A balanced tree repeatedly cuts the remaining search space roughly in half.

### Balanced vs unbalanced

| Shape        |                 Height | Lookup cost |
| ------------ | ---------------------: | ----------: |
| Balanced     | Approximately `log₂ N` |  `O(log N)` |
| Pathological |              Up to `N` |      `O(N)` |

Insertion order can determine tree shape unless the structure actively rebalances itself.

```text
Insert: 1, 2, 3, 4, 5

Unbalanced result:

1
 \
  2
   \
    3
     \
      4
       \
        5
```

The tree has become a linked list. 

### How balancing works

Balanced BST variants reorganise nodes after inserts and deletes.

A common operation is a **rotation**:

* Promote a middle node.
* Move its former parent below it.
* Preserve key ordering.
* Reduce the height difference between branches.

### Why it matters

Complexity guarantees depend on tree height—not merely on keys being sorted.

```text
Sorted + balanced
    → logarithmic traversal

Sorted + unbalanced
    → potentially linear traversal
```

### Common mistakes

* Assuming every binary search tree is automatically balanced.
* Treating average-case `O(log N)` as a guaranteed worst case.
* Ignoring insertion order.
* Thinking rotations change sorted order; they change structure while preserving order.

### Quick example

For a balanced tree containing one million keys:

```text
log₂(1,000,000) ≈ 20
```

A lookup requires roughly 20 tree-level decisions rather than scanning one million items.

### Recall questions

1. Why can sorted insertion turn a BST into a linked list?
2. How can a rotation change tree height without breaking key order?
3. Which property determines the number of nodes visited during lookup?
4. Why is `O(log N)` not guaranteed for an ordinary unbalanced BST?

---

## 2. Why Binary Trees Are Poor Disk Structures

### Core idea

A data structure that is efficient in RAM is not automatically efficient on disk.

A binary tree has a fanout of only two:

```text
One node
   → at most two subtrees
```

This creates many levels, and each level may require another disk-page access.

### Cause and effect

```text
Low fanout
   ↓
Greater tree height
   ↓
More pointer traversals
   ↓
More page reads or disk seeks
   ↓
Higher lookup latency
```

A binary tree with `N` elements has approximately `log₂ N` levels. If nodes reside on unrelated pages, a lookup may require roughly one disk access per level.  

### Locality problem

Nodes are commonly created in insertion order.

A child may therefore be stored far from its parent:

```text
Parent on page 12
Child on page 8,340
Grandchild on page 91
```

Following each pointer may fetch a different page.

### Maintenance problem

Balanced binary trees require frequent:

* Rotations
* Node relocation
* Pointer updates
* Page reorganisation

These operations are much more expensive on persistent storage than in memory.

### Rule for disk-oriented trees

An effective disk search tree should have:

1. **High fanout** — many children per node.
2. **Low height** — few page transitions.
3. **Good locality** — useful neighbouring keys stored together.
4. **Few external pointers** — fewer cross-page dependencies.

### Relationship

Fanout and height are inversely related:

```text
Higher fanout
    → more key ranges per node
    → fewer levels
    → fewer disk accesses
```

### Paged binary trees

Several binary-tree nodes can be grouped into one page.

This improves locality because several comparisons occur after one page read.

However, it does not fully solve:

* Low logical fanout
* Pointer overhead
* Page reorganisation
* Random insertion
* Balancing complexity



### Common mistakes

* Comparing structures using CPU complexity alone.
* Treating one pointer traversal as constant-cost on disk.
* Assuming a page contains exactly one node.
* Ignoring the cost of updating persistent pointers.
* Assuming paging a BST makes it equivalent to a B-Tree.

### Quick example

Suppose:

* Binary-tree height: 25
* B-Tree height: 4

Even when both have logarithmic complexity, the first may require roughly 25 page accesses while the second requires about 4.

### Recall questions

1. Why can two `O(log N)` trees have very different disk performance?
2. How does high fanout improve locality?
3. Why is frequent rotation less problematic in RAM than on disk?
4. Which problems remain after grouping binary-tree nodes into pages?

---

## 3. Storage Hardware Constraints

## Hard disk drives

### Core idea

HDD random access is expensive because the device must physically position its read/write head.

```text
Random access
    → head movement + rotation
    → high seek cost

Sequential access
    → head already positioned
    → lower incremental cost
```

Once positioned, reading contiguous bytes is relatively cheap.

HDDs transfer data in sectors, commonly between 512 bytes and 4 KiB. 

### Mental model

Opening the correct page in a physical book is expensive; reading the next paragraph is cheap.

---

## Solid-state drives

### Core idea

SSDs remove mechanical seek latency, but still have page- and block-level constraints.

Typical hierarchy:

```text
Cell
  → string
    → array
      → page
        → block
          → plane
            → die
```

* Reads and writes operate on pages.
* Erasure operates on larger blocks.
* A page generally must be erased before being rewritten.
* Pages within an empty block are written sequentially.

 

### Flash Translation Layer

The **FTL** maps logical page identifiers to physical locations.

It also tracks:

* Empty pages
* Written pages
* Discarded pages
* Blocks available for erasure

### SSD garbage collection

```text
Block contains:
    live pages + obsolete pages
              ↓
Copy live pages elsewhere
              ↓
Update logical-to-physical mappings
              ↓
Erase entire old block
              ↓
Reuse block
```

### Why random writes can still be costly

Random or unaligned writes can:

* Scatter live pages
* Trigger extra relocation
* Increase garbage collection
* Create write amplification
* Cause unpredictable write latency

### HDD vs SSD

| Property                    | HDD                  | SSD                         |
| --------------------------- | -------------------- | --------------------------- |
| Mechanical movement         | Yes                  | No                          |
| Random-read penalty         | High                 | Lower                       |
| Sequential benefit          | Strong               | Still present               |
| Read/write unit             | Sector/block         | Page                        |
| Erase-before-rewrite        | No                   | Yes                         |
| Internal garbage collection | Not in the same form | Yes                         |
| Main write concern          | Seek cost            | Erase blocks and relocation |

### Important exception

SSDs reduce the difference between random and sequential reads, but do not eliminate storage-layout concerns.

Contiguous access can still benefit from:

* Prefetching
* Internal parallelism
* Fewer mapping operations
* Better write coalescing



### Common mistakes

* Treating an SSD as byte-addressable RAM.
* Assuming SSD overwrites occur in place.
* Ignoring erase-block size.
* Assuming background garbage collection has no foreground impact.
* Concluding that sequential layout no longer matters on SSDs.

### Recall questions

1. Why must an SSD relocate live pages before erasing a block?
2. How can random writes increase write amplification?
3. Why does sequential access still help on SSDs?
4. Which hardware constraint most strongly shaped classic disk-tree designs?

---

## 4. Block-Oriented Storage

### Core idea

Persistent storage is accessed in chunks, not one field or pointer at a time.

Even when an application requests one word, the storage stack normally reads the block containing it.

### Mental model

A page read has a largely fixed entry cost.

Therefore:

```text
Already paying to read the page
    ↓
Place many useful keys and pointers in that page
    ↓
Perform many comparisons without another I/O
```

### On-disk pointers

An in-memory pointer is usually a directly usable address.

An on-disk pointer is commonly:

* A file offset
* A page identifier
* A block identifier
* A logical reference translated by another layer

The engine must explicitly calculate, store and resolve these references.

### Design rule

Minimise:

* Number of page reads
* Number of cross-page pointers
* Pointer-chain length
* Pointer changes during reorganisation

Maximise:

* Useful information per page
* Key locality
* Full-block writes
* Reuse of already fetched pages



### Common mistake

Optimising the number of comparisons while ignoring the number of pages transferred.

Ten comparisons in one page may be much cheaper than three comparisons across three pages.

### Recall questions

1. Why can several comparisons within one page be inexpensive?
2. How does an on-disk pointer differ from a normal memory pointer?
3. Why are long pointer dependency chains undesirable?
4. What should a disk structure maximise after a page has been loaded?

---

## 5. B-Tree Mental Model

### Core idea

A B-Tree is a balanced, multiway search tree designed around storage pages.

Instead of one key and two children per node, one B-Tree node stores many ordered separator keys and many child pointers.

### Library mental model

```text
Building
  → cabinet
    → shelf
      → drawer
        → card
```

Each level narrows the search to a smaller range.

### Binary tree vs B-Tree

| Property           | Binary tree   | B-Tree                        |
| ------------------ | ------------- | ----------------------------- |
| Keys per node      | Usually 1     | Many                          |
| Maximum children   | 2             | Many                          |
| Fanout             | Low           | High                          |
| Height             | Greater       | Smaller                       |
| Disk suitability   | Weak          | Strong                        |
| Structural changes | More frequent | Amortised across larger nodes |

B-Trees reduce height by storing dozens or hundreds of keys in each page. One page access therefore resolves several comparisons at once.  

### Supported queries

#### Point query

Find one exact key:

```sql
WHERE id = 42
```

#### Range query

Find an ordered interval:

```sql
WHERE id >= 40 AND id < 60
```

B-Trees support both because keys are stored in sorted order. 

### Why it matters

```text
Many keys per page
   ↓
High fanout
   ↓
Low tree height
   ↓
Few page reads
   ↓
Efficient point and range access
```

### Common mistakes

* Thinking a B-Tree is a binary tree variant with different balancing.
* Assuming the “B” means binary.
* Measuring only key comparisons.
* Treating every key comparison as a separate disk access.
* Assuming B-Trees support only equality lookups.

### Quick example

Suppose one node has 200 child pointers.

A three-level tree can address approximately:

```text
200³ = 8,000,000 key ranges
```

A fourth level increases this dramatically while adding only one more page traversal.

### Recall questions

1. Why does storing more keys per node reduce disk access?
2. How can a B-Tree support range scans as well as point lookups?
3. Why is tree height more important than comparison count for disk I/O?
4. How does node size influence fanout?

---

## 6. B-Tree Hierarchy

### Node types

| Node type         | Position                | Responsibility                                  |
| ----------------- | ----------------------- | ----------------------------------------------- |
| **Root**          | Top                     | Chooses a coarse key range                      |
| **Internal node** | Between root and leaves | Refines the range                               |
| **Leaf**          | Bottom                  | Contains terminal keys and, in B+ Trees, values |

A node with up to `N` keys may have up to `N + 1` child pointers.

Because B-Trees organise fixed-size pages, **node** and **page** are often used interchangeably. 

### Occupancy

**Occupancy** is the fraction of node capacity currently used.

```text
Node capacity: 100 entries
Stored entries: 70
Occupancy: 70%
```

Nodes reserve free space for later writes.

Higher occupancy:

* Improves storage utilisation
* Increases effective fanout
* Leaves less immediate room for insertion

Lower occupancy:

* Wastes space
* Reduces effective fanout
* Delays future splits

### Structural growth

B-Trees grow primarily from the bottom upward:

```text
More records
   ↓
More leaf nodes
   ↓
More internal references
   ↓
Possible new root
```

The tree grows horizontally at ordinary levels. Its height increases only when the root splits.

### Typical occupancy rule

Classical B-Tree variants maintain minimum occupancy bounds for non-root nodes, often near half capacity.

**Exception:** exact thresholds differ by implementation and variant.

### Common mistakes

* Assuming every node is always full.
* Treating occupancy as the same as fanout.
* Assuming inserting one record always increases tree height.
* Assuming leaves can exist at different depths in a balanced B-Tree.

### Recall questions

1. What is the difference between node capacity and occupancy?
2. Why does reserved free space reduce immediate split frequency?
3. Which event increases B-Tree height?
4. Why does effective fanout fall when occupancy is low?

---

## 7. B-Trees vs B+ Trees

### Core distinction

| B-Tree                                            | B+ Tree                               |
| ------------------------------------------------- | ------------------------------------- |
| Values may appear in root, internal or leaf nodes | Values appear only in leaves          |
| Internal keys may also represent records          | Internal keys guide navigation only   |
| Search may terminate above the leaf level         | Record lookup normally reaches a leaf |

The book uses **B-Tree** as an umbrella term, while the described database structure is more precisely a **B+ Tree**. 

### Why B+ Trees are common

Keeping values out of internal nodes allows each internal page to hold more:

* Separator keys
* Child pointers

This increases fanout and may reduce tree height.

### Operational consequence

In a B+ Tree:

* Insert
* Update
* Delete
* Record retrieval

primarily affect leaves.

Changes propagate upward only when structural operations such as splits or merges alter subtree boundaries.

### Mental model

Internal nodes are a map; leaves are the destination.

```text
Internal pages: "Which direction?"
Leaf pages:     "Here is the record."
```

### Trade-off

| Values in internal nodes           | Values only in leaves                    |
| ---------------------------------- | ---------------------------------------- |
| Some searches terminate earlier    | All record lookups follow a uniform path |
| Less space for routing information | Greater internal-node fanout             |
| Records spread across levels       | Records concentrated at leaf level       |

### Common mistakes

* Treating B-Tree and B+ Tree as perfectly interchangeable in every technical context.
* Assuming internal separator keys necessarily contain full records.
* Assuming a B+ Tree lookup can return a value directly from an internal node.
* Forgetting that database products may call B+ Trees “B-trees.”

### Recall questions

1. Why can B+ Tree internal nodes have greater fanout?
2. Which operations normally modify leaf nodes?
3. When must an update propagate to an internal node?
4. What trade-off allows an ordinary B-Tree search to terminate early?

---

## 8. Separator Keys

### Core idea

Internal keys divide the complete key space into ordered subranges.

For ordered separator keys:

```text
K1, K2, K3
```

the pointers represent ranges similar to:

```text
P0: key < K1
P1: K1 ≤ key < K2
P2: K2 ≤ key < K3
P3: key ≥ K3
```

Separator keys are also called:

* Index entries
* Divider cells
* Routing keys



### How navigation works

1. Search the separator keys within the current node.
2. Determine which interval contains the target.
3. Follow the interval’s child pointer.
4. Repeat until reaching a leaf.

### Mental model

A separator does not necessarily say:

> “The record is here.”

It says:

> “Keys in this range are below this pointer.”

### Sibling pointers

Many B+ Tree implementations connect neighbouring leaf pages.

```text
Leaf A ↔ Leaf B ↔ Leaf C ↔ Leaf D
```

This allows range scans to move directly between leaves without returning to their parents.

Some implementations use:

* Forward links only
* Bidirectional links for forward and reverse scans



### Why it matters

Without sibling links, moving to the next leaf may require:

* Ascending to a parent
* Finding the next subtree
* Descending again

Sibling links turn ordered leaf traversal into a sequential chain.

### Common mistakes

* Assuming every separator key is a complete stored record.
* Using the wrong inclusive/exclusive range boundary.
* Assuming leaf siblings must always be doubly linked.
* Returning to the root for every element in a range scan.

### Quick example

Internal node:

```text
[20 | 50]
```

Child ranges:

```text
P0 → keys below 20
P1 → keys from 20 through below 50
P2 → keys 50 and above
```

A search for `37` follows `P1`.

### Recall questions

1. What information does a separator key provide?
2. Why are there `N + 1` pointers for `N` separator keys?
3. How do sibling links improve range scans?
4. Which pointer is followed for a key greater than the last separator?

---

## 9. B-Tree Lookup Cost

### Core idea

Lookup cost has two different dimensions:

1. **Page transfers**
2. **Key comparisons**

These should not be confused.

### Page-transfer cost

A traversal reads at most approximately one page per tree level.

```text
Page-transfer cost ≈ tree height
```

High fanout reduces height because each node eliminates a large fraction of the search space.

### Comparison cost

Within each page, keys are commonly searched using binary search.

Across the entire lookup, the comparison complexity remains logarithmic in the number of keys.



### Mental model

```text
Disk cost:
How many rooms must be entered?

CPU cost:
How many labels must be compared inside those rooms?
```

### Key relationship

Increasing node capacity may:

* Increase comparisons within one node
* Reduce the number of nodes visited
* Reduce expensive page transfers

This is usually favourable because in-page CPU comparisons are much cheaper than additional storage access.

### Common mistake

Saying “B-Tree lookup is `O(log N)`” without clarifying whether the concern is:

* CPU comparisons
* Cache misses
* Page transfers
* Physical disk seeks

The asymptotic notation is the same, but the practical costs differ.

### Quick example

Tree A:

* 20 levels
* 1 comparison per page

Tree B:

* 4 levels
* 8 comparisons per page

Tree B performs more comparisons per page but far fewer storage transfers.

### Recall questions

1. Why can more comparisons produce a faster lookup?
2. Which quantity is approximately equal to tree height?
3. How does node fanout alter the base of the page-transfer logarithm?
4. Why does asymptotic notation hide important storage costs?

---

## 10. Lookup Algorithm

### Goal

Find either:

* The exact key
* Its predecessor or insertion position

### Algorithm

```text
Start at root
    ↓
Binary-search separator keys
    ↓
Select matching key range
    ↓
Follow child pointer
    ↓
Repeat through internal nodes
    ↓
Search target leaf
```

At the leaf:

* Exact match → point query succeeds.
* No match → conclude absence or use the nearest position.
* Range query → start at the closest valid entry and follow leaf siblings.



### Coarse-to-fine mental model

Each level gives a more precise view:

```text
Root:          continent
Internal:      country
Internal:      city
Leaf:          exact address
```

### Why predecessor search matters

A predecessor or insertion location is useful for:

* Starting a range scan
* Inserting while preserving key order
* Finding the nearest lower key
* Determining that an exact key is absent

### Quick example

Search for `46`:

```text
Root [20, 60]
    → choose [20, 60)

Internal [30, 45, 55]
    → choose [45, 55)

Leaf [45, 48, 51]
    → 46 absent
    → insertion point between 45 and 48
```

### Common mistakes

* Scanning every key in every node linearly by default.
* Starting a range scan from the root for each returned record.
* Treating a missing exact match as a failed traversal.
* Forgetting that lookup also determines insertion position.

### Recall questions

1. Why does insertion begin with the lookup algorithm?
2. How does a range scan continue after finding its starting leaf?
3. What does a failed exact lookup still tell the engine?
4. Why is the root considered a coarse-grained view?

---

## 11. Node Splits

### Core idea

A node splits when inserting another entry would exceed its capacity.

This condition is called **overflow**.

### Leaf overflow

A leaf that can hold at most `N` key-value pairs overflows when insertion would produce `N + 1`.

### Internal-node overflow

An internal node overflows when it would exceed its allowed number of keys or child pointers.

### Split process

1. Allocate a new sibling node.
2. Divide entries around a split point.
3. Move roughly half to the new node.
4. Insert the new entry into the correct half.
5. Add a separator and child pointer to the parent.

 

### Mental model

A full filing drawer is divided into two drawers, and the cabinet receives a new label describing where the second drawer begins.

### Propagation

The parent must receive the new separator.

```text
Leaf splits
   ↓
Parent gains separator + pointer
   ↓
Parent may overflow
   ↓
Parent splits
   ↓
Propagation may reach root
```

### Root split

If the root splits:

1. Allocate a new root.
2. Place the promoted separator in it.
3. Point it to the two resulting child nodes.
4. Increase tree height by one.

This is the principal way a B-Tree becomes taller. 

### Leaf vs internal split

| Leaf split                   | Internal split                            |
| ---------------------------- | ----------------------------------------- |
| Moves keys with their values | Moves routing keys and pointers           |
| Parent receives a separator  | Middle separator is promoted              |
| Leaves retain actual records | Internal pages retain routing information |

### Cause and effect

```text
Larger nodes
   → more insertions before overflow
   → fewer splits
   → higher fanout

But:

Larger nodes
   → larger page transfers
   → more in-page search work
```

### Common mistakes

* Assuming every insertion causes a split.
* Splitting before locating the correct leaf.
* Forgetting to update the parent.
* Assuming a leaf split always increases tree height.
* Losing key order while dividing entries.
* Treating the promoted separator identically in all B-Tree variants.

### Quick example

Capacity: four entries.

Before:

```text
[10, 20, 30, 40]
```

Insert `25`:

```text
Left:  [10, 20]
Right: [25, 30, 40]
Parent receives separator identifying Right
```

The exact split distribution and separator-copying rule depend on the B-Tree variant.

### Recall questions

1. Why can one leaf insertion trigger several internal splits?
2. Which split changes tree height?
3. Why must a separator be added to the parent?
4. How does node capacity affect split frequency?
5. What differs between splitting a leaf and splitting an internal node?

---

## 12. Node Merges

### Core idea

Deletion can leave a node below its minimum occupancy.

This condition is called **underflow**.

Possible responses:

1. **Redistribute** entries between siblings.
2. **Merge** siblings when their combined contents fit in one node.

### Merge condition

Two neighbouring nodes with a common parent can merge when their combined entries fit within one node’s capacity.

### Leaf merge

```text
Left leaf + Right leaf
        ↓
Move entries into one leaf
        ↓
Remove obsolete child pointer from parent
        ↓
Delete empty page
```

### Internal-node merge

An internal merge also pulls the relevant separator down from the parent so the combined child ranges remain correctly represented.

 

### Merge process

Assuming the target entry is already deleted:

1. Copy entries from one sibling into the other.
2. Remove or demote the corresponding parent separator.
3. Remove the unused node.
4. Repair sibling and parent references as required.

### Propagation

Removing a child pointer may make the parent underfull:

```text
Leaf merge
   ↓
Parent loses pointer and separator
   ↓
Parent may underflow
   ↓
Internal merge may propagate upward
```

### Root contraction

If merging leaves the root with only one child, that child can become the new root.

```text
Root split  → height + 1
Root merge  → height - 1
```

### Merge vs redistribution

| Merge                            | Redistribution                                    |
| -------------------------------- | ------------------------------------------------- |
| Combines nodes                   | Moves entries between nodes                       |
| Removes one page                 | Keeps both pages                                  |
| Removes parent separator/pointer | Adjusts parent separator                          |
| Used when combined data fits     | Used when it does not fit but balance can improve |

### Trade-off

Aggressive merging:

* Improves occupancy
* Reduces wasted space
* May create more structural writes

Delayed merging:

* Reduces immediate write work
* Leaves sparse pages
* Can lower effective fanout

### Common mistakes

* Assuming every deletion immediately merges nodes.
* Merging siblings whose combined entries exceed capacity.
* Forgetting the parent separator.
* Treating leaf and internal merges identically.
* Ignoring recursive parent underflow.
* Rebalancing nodes that do not share an appropriate parent relationship.

### Quick example

Capacity: six entries.

```text
Left:  [10, 20]
Right: [30, 40]
```

Combined size is four, so they can fit in one node:

```text
Merged: [10, 20, 30, 40]
```

The parent’s pointer and separator for the removed sibling must also be updated.

### Recall questions

1. When should siblings merge rather than redistribute?
2. Why can a leaf merge cause an internal-node merge?
3. What happens to the parent separator during an internal merge?
4. How can delayed merging affect future lookup cost?
5. Which structural events can change tree height?

---

## 13. B-Tree Design Summary

### Core mental model

B-Trees exchange cheap CPU work inside a page for fewer expensive page accesses.

```text
Large ordered pages
    + high fanout
    + balanced height
    + controlled splits and merges
    =
Efficient disk search structure
```

### Key relationships

| Design choice       | Benefit                      | Cost                                 |
| ------------------- | ---------------------------- | ------------------------------------ |
| High fanout         | Lower height                 | Larger nodes                         |
| Reserved free space | Fewer immediate splits       | Lower storage utilisation            |
| Sorted keys         | Point and range queries      | Ordered insertion work               |
| Leaf sibling links  | Efficient range scans        | Extra pointer maintenance            |
| Splitting           | Restores capacity invariant  | Additional writes and parent updates |
| Merging             | Restores occupancy invariant | Structural reorganisation            |
| B+ Tree leaves      | High internal fanout         | Every record lookup reaches a leaf   |

### Rules to retain

* **Balanced height** prevents pathological traversal.
* **High fanout** reduces page transfers.
* **Separator keys route; leaves store data** in B+ Trees.
* **Overflow leads to split.**
* **Underflow leads to redistribution or merge.**
* **Root split increases height.**
* **Root contraction decreases height.**
* **Sibling links make ordered scans efficient.**

### Common conceptual mistakes

* Equating logarithmic CPU complexity with good disk behaviour.
* Confusing key comparisons with page transfers.
* Treating B-Trees as binary trees.
* Assuming all database “B-trees” follow precisely the same variant.
* Ignoring node occupancy.
* Forgetting that splits and merges can propagate.
* Evaluating lookup speed without considering update and maintenance costs.

### Applied recall questions

1. A B-Tree has excellent lookup complexity but poor occupancy. How can this affect height and storage use?
2. Why might increasing page size improve lookup performance but hurt some workloads?
3. A range query repeatedly returns to the root between records. What structural feature is probably missing or unused?
4. An insertion increased tree height. What sequence of events must have occurred?
5. A deletion left two sparse siblings whose combined data exceed one page. What should the engine try before merging?
6. Why might a B+ Tree require more levels than a theoretical B-Tree yet still perform better in practice?
7. How would SSD erase-block behaviour influence the cost of frequent B-Tree page rewrites?

# Chapter 3: File Formats

## 1. Why File Formats Matter

### Core idea

An on-disk data structure is not only an algorithm. It also needs a precise binary layout that defines:

* Where data is stored
* How records are addressed
* How bytes are interpreted
* How free space is managed
* How the format evolves safely

Disk access requires explicit file offsets and conversion between bytes and in-memory objects. 

### Mental model

A file format is a contract between the writer and reader:

```text
Writer:
object → bytes at known positions

Reader:
bytes at known positions → object
```

Both sides must agree on:

* Field order
* Field size
* Byte order
* Record boundaries
* Page boundaries
* Version
* Validation rules

### Memory vs disk

| In memory                              | On disk                              |
| -------------------------------------- | ------------------------------------ |
| Allocation handled by runtime or OS    | Layout must be explicitly designed   |
| Pointers directly reference objects    | References are offsets or page IDs   |
| Variable-size allocation is convenient | Free-space management is manual      |
| Fragmentation is often abstracted away | Fragmentation must be handled        |
| Representation can be runtime-specific | Representation must survive restarts |



### Cause and effect

```text
Poor layout
   ↓
More offset calculations
More I/O
More fragmentation
Harder upgrades
   ↓
Lower performance and reliability
```

### Common mistakes

* Treating a B-Tree algorithm as a complete B-Tree implementation.
* Writing runtime memory structures directly to disk.
* Assuming pointers remain valid after process restart.
* Ignoring fragmentation and reclaimed space.
* Designing a format that is easy to write but difficult to decode.

### Quick example

An in-memory record may contain:

```text
{name_pointer, name_length, age}
```

The pointer is meaningless after restart. The file must instead store something stable, such as:

```text
{name_offset, name_length, age}
```

### Recall questions

1. Why cannot ordinary memory pointers be persisted directly?
2. Which responsibilities are hidden in memory but explicit in a file format?
3. How can a poor layout increase both read and update costs?
4. Why is a correct B-Tree algorithm insufficient for an on-disk implementation?

---

## 2. Binary Encoding

### Core idea

Persistent data must be converted into a compact, unambiguous byte sequence.

```text
Object
  ──serialize──▶ bytes
  ◀─deserialize─
```

A binary format begins with primitive values and combines them into larger records, cells and pages. 

### Requirements

A useful binary representation should be:

* Compact
* Fast to encode
* Fast to decode
* Deterministic
* Portable where required
* Sufficiently self-describing to recover boundaries

### Rule

A reader must have enough information to answer:

```text
What type begins here?
How many bytes belong to it?
How should those bytes be interpreted?
Where does the next value begin?
```

### Common mistakes

* Omitting length information for variable-sized values.
* Depending on language-specific object layout.
* Assuming all platforms represent primitive values identically.
* Failing to validate that enough bytes remain before decoding.

### Recall questions

1. What information is required to decode a byte sequence unambiguously?
2. Why should persistent formats not depend directly on runtime object layout?
3. How does compactness trade off against ease of decoding?

---

## 3. Primitive Types and Endianness

### Fixed-size primitives

Typical primitive sizes:

| Type    | Typical size |
| ------- | -----------: |
| Byte    |       1 byte |
| Short   |      2 bytes |
| Integer |      4 bytes |
| Long    |      8 bytes |
| Float   |      4 bytes |
| Double  |      8 bytes |

These primitives form the building blocks for larger structures. 

## Endianness

### Core idea

Endianness determines the order in which bytes of a multibyte value are stored.

For the hexadecimal value:

```text
0xAABBCCDD
```

| Order         | Stored bytes from lowest address |
| ------------- | -------------------------------- |
| Big-endian    | `AA BB CC DD`                    |
| Little-endian | `DD CC BB AA`                    |

### Mental model

* **Big-endian:** most significant part first.
* **Little-endian:** least significant part first.

### Rule

Encoding and decoding must use the same byte order.

```text
Different byte order
   ↓
Same bytes interpreted as a different number
```

### Portability

A format can:

1. Define one fixed byte order for every platform.
2. Store byte-order metadata.
3. Detect the host architecture and transform bytes when necessary.

 

### Common mistakes

* Using the machine’s native byte order without documenting it.
* Assuming network and storage byte order match host byte order.
* Reversing bytes twice.
* Treating endianness as relevant only to networking.

### Quick example

Bytes:

```text
01 00
```

Interpretation:

* Little-endian 16-bit integer → `1`
* Big-endian 16-bit integer → `256`

### Recall questions

1. Why can the same two bytes represent different integers?
2. How can a file format remain portable across architectures?
3. What should happen when the stored and host byte orders differ?
4. Why does endianness affect only multibyte values?

---

## 4. Floating-Point Representation

### Core idea

Floating-point values are usually encoded using:

* Sign
* Exponent
* Fraction or significand

A 32-bit single-precision float commonly uses:

| Part     | Bits |
| -------- | ---: |
| Sign     |    1 |
| Exponent |    8 |
| Fraction |   23 |



### Mental model

Floating point stores a value in a form resembling scientific notation:

```text
sign × fraction × base^exponent
```

### Important fact

Many decimal values do not have exact binary floating-point representations.

```text
Decimal input
   ↓
Nearest representable binary value
   ↓
Possible rounding error
```

### Common mistakes

* Assuming every decimal fraction is represented exactly.
* Comparing floating-point values using exact equality without considering rounding.
* Treating a floating-point binary format as an arbitrary sequence of numeric bytes.
* Changing representation between writer and reader.

### Quick example

A decimal such as `0.1` may be stored as a nearby binary approximation rather than exactly `0.1`.

### Recall questions

1. Why can a decoded floating-point value differ slightly from its original decimal form?
2. What roles do the sign, exponent and fraction play?
3. Why must the reader know the exact floating-point standard used?

---

## 5. Variable-Size Data

### Core idea

Fixed-size values reveal their boundaries from their type. Variable-sized values need explicit boundary information.

## Length-prefixed encoding

```text
[length][data bytes]
```

Example:

```text
String {
    size: uint16
    data: byte[size]
}
```

This is commonly called a length-prefixed or Pascal-style string. 

### Advantages

* Length available in constant time
* Embedded zero bytes are possible
* Reader knows exactly how many bytes to consume
* Next field can be located directly

## Null-terminated encoding

```text
[data bytes][terminator]
```

The reader scans until it finds the terminator.

### Trade-off

| Length-prefixed                     | Null-terminated                              |
| ----------------------------------- | -------------------------------------------- |
| Constant-time length lookup         | Requires scanning                            |
| Length field adds overhead          | Terminator adds overhead                     |
| Can contain zero bytes              | Terminator may need escaping or restrictions |
| Corrupt length can misdirect parser | Missing terminator can overrun boundaries    |

### Offset-and-length encoding

For records containing several variable fields, a fixed header can store:

```text
field offset + field length
```

This allows direct access to any field without decoding every preceding field.

### Cause and effect

```text
Store only lengths
   → compact header
   → later fields require summing earlier lengths

Store offsets + lengths
   → larger header
   → direct field access
```

### Common mistakes

* Trusting a stored length without checking page boundaries.
* Using too small a length type for possible values.
* Calculating later-field offsets repeatedly.
* Failing to define string character encoding separately from byte length.
* Assuming character count equals byte count.

### Quick example

```text
Header:
first_name_length = 3
last_name_length  = 5

Payload:
"TomSmith"
```

The first three bytes form `Tom`; the next five form `Smith`.

### Recall questions

1. Why does a variable-sized field require additional metadata?
2. When are offsets preferable to lengths alone?
3. How can a corrupt length field affect subsequent decoding?
4. Why are byte length and character length not always equal?

---

## 6. Struct Layout and Padding

### Core idea

Combining primitives resembles creating a low-level structure:

```text
Record {
    integer
    short
    byte
}
```

However, compiler-generated in-memory layouts may contain padding inserted for alignment.

### Rule

Do not assume that:

```text
sizeof(runtime struct)
=
sum of declared field sizes
```

Padding can vary by:

* Compiler
* Architecture
* Alignment rules
* Packing configuration

The book warns that architecture-dependent padding can invalidate assumptions about byte offsets. 

### Safe approaches

* Encode each field explicitly.
* Define exact offsets in the format specification.
* Use standard serialization routines.
* Avoid writing raw runtime structures directly.
* Test representation across supported platforms.

### Common mistake

```c
write(file, &record, sizeof(record));
```

This can persist:

* Padding bytes
* Native endianness
* Architecture-specific field alignment
* Pointer values

### Recall questions

1. Why can two compilers lay out the same logical structure differently?
2. What problems arise when raw structs are written directly to disk?
3. How does explicit field encoding improve portability?

---

## 7. Bit Packing: Booleans, Enums and Flags

## Booleans

### Core idea

A boolean needs only one bit, although simple encodings often use one byte.

Packing eight booleans into one byte reduces space:

```text
bit 7 6 5 4 3 2 1 0
    1 0 1 1 0 0 1 0
```



## Enums

Enums encode one value from a predefined set:

```text
ROOT     = 0
INTERNAL = 1
LEAF     = 2
```

### Rule

Enums represent **mutually exclusive alternatives**.

Only one node type applies at a time.

## Flags

Flags encode independent boolean properties:

```text
IS_LEAF             = 0000 0001
VARIABLE_SIZE_VALUE = 0000 0010
HAS_OVERFLOW        = 0000 0100
```

Several flags can be enabled simultaneously.

### Mental model

| Enum              | Flags                              |
| ----------------- | ---------------------------------- |
| Choose one option | Toggle several independent options |
| One value         | Bit set                            |
| `ROOT` or `LEAF`  | `LEAF` and `HAS_OVERFLOW`          |

### Bit operations

```text
Set flag:    flags |= MASK
Clear flag:  flags &= ~MASK
Test flag:   (flags & MASK) != 0
```

Each mask normally uses a power of two so that exactly one bit is set. 

### Trade-off

Bit packing:

* Reduces representation size
* Improves density
* Requires masks and bitwise operations
* Makes debugging less immediately readable
* Limits the number of flags to available bits

### Common mistakes

* Assigning overlapping masks.
* Using flags for mutually exclusive states without validation.
* Testing equality instead of masking:

```text
Wrong: flags == HAS_OVERFLOW
Right: flags & HAS_OVERFLOW != 0
```

* Reusing a bit for a different meaning in a later format version.

### Recall questions

1. Why must flag masks normally be powers of two?
2. How do enums differ conceptually from flags?
3. Why can direct equality fail when testing a flag?
4. What compatibility problem occurs if an existing bit changes meaning?

---

## 8. General File Organisation

### Core idea

A database file commonly consists of:

```text
[fixed header][page][page][page]...[optional trailer]
```

The header or trailer stores metadata needed to:

* Identify the file
* Decode pages
* Find sections
* Check format version
* Locate lookup tables
* Validate integrity



### Fixed-size pages

Many mutable storage structures divide files into equally sized pages.

Benefits:

* Simple address calculation
* Predictable I/O
* Straightforward caching
* Easier allocation and reuse
* Direct mapping from page ID to file offset

```text
page_offset = header_size + page_id × page_size
```

### Append-only formats

Append-only structures also commonly buffer records into pages:

```text
Accumulate records in memory
   ↓
Page becomes full
   ↓
Flush complete page
```

### Fixed schema

A fixed schema defines:

* Field count
* Field order
* Field types

This avoids repeatedly storing field names.

```text
Schema says field #2 is birth_date
```

The record only needs to store the value, not the string `"birth_date"`.

### Trade-off

| Fixed schema              | Self-describing records           |
| ------------------------- | --------------------------------- |
| Smaller records           | Easier independent interpretation |
| Faster positional access  | More metadata per record          |
| Schema required to decode | More flexible evolution           |
| Less repetition           | Greater storage overhead          |

### Hierarchical composition

Formats are commonly built in layers:

```text
Primitive values
   ↓
Fields
   ↓
Cells
   ↓
Pages
   ↓
Sections
   ↓
Regions
   ↓
File
```



### Common mistakes

* Repeating schema information in every fixed-schema record.
* Choosing page size without considering storage and workload.
* Making file sections impossible to locate without scanning.
* Storing critical decoding metadata only in a variable format.
* Assuming one page size is optimal for all workloads.

### Recall questions

1. How do fixed-size pages simplify addressing?
2. Why does a fixed schema reduce storage overhead?
3. What metadata belongs in a file header?
4. When might a self-describing format be preferable despite its overhead?

---

## 9. Page Structure

### Core idea

Database files are partitioned into pages, often around 4–16 KiB, although exact sizes vary.

A B-Tree page may contain:

* Leaf key-value records
* Internal separator keys
* Child page references
* Page metadata
* Free space



### Simple fixed-record page

A simple layout can concatenate fixed-size entries:

```text
[pointer][key][value][pointer][key][value]...
```

### Advantages

* Easy offset calculation
* Simple implementation
* Efficient when every entry has the same size

### Limitations

* Inserting in the middle requires shifting later entries.
* Variable-sized records are difficult to manage.
* Deletion can leave unusable gaps.
* Relocation can invalidate external references.



### Cause and effect

```text
Variable record sizes
   ↓
Uneven holes after delete/update
   ↓
Internal fragmentation
   ↓
Need indirection or compaction
```

### Common mistakes

* Using a fixed-entry layout for highly variable records.
* Ignoring the cost of shifting entries during sorted insertion.
* Assuming total free bytes means a sufficiently large contiguous region exists.
* Exposing direct physical record offsets outside a page.

### Recall questions

1. Why does middle insertion require relocation in a packed fixed-record page?
2. How can a page have enough total free space but still reject a record?
3. Why do variable-sized records complicate space reuse?

---

## 10. Internal Fragmentation

### Core idea

Deleting or replacing variable-sized records creates holes of different sizes.

Example:

```text
Free segments: 20 B, 35 B, 10 B
New record:    50 B
```

Total free space is 65 B, but no single 50 B segment exists.

### Fixed-segment alternative

A page could be divided into equal-sized slots, such as 64 bytes.

This simplifies allocation but causes wasted space:

```text
record size = 50 B
slot size   = 64 B
waste       = 14 B
```

For record size `n` and segment size `s`:

```text
waste = s - (n mod s)
```

when `n` is not a multiple of `s`.

### Trade-off

| Variable segments      | Fixed segments            |
| ---------------------- | ------------------------- |
| Better packing         | Simpler allocation        |
| External fragmentation | Internal slot waste       |
| Harder reuse           | Predictable addressing    |
| Needs size tracking    | May require chained slots |

### Design goal

A useful page format should:

* Store variable-sized records efficiently
* Reclaim deleted space
* Keep stable logical references
* Allow physical relocation within a page

 

### Recall questions

1. What is the difference between total free space and contiguous free space?
2. Why do fixed-size slots waste space?
3. Which fragmentation problem arises with variable-size allocation?
4. What page abstraction allows records to move without changing external references?

---

## 11. Slotted Pages

### Core idea

A slotted page separates:

1. Logical record references
2. Physical record locations

Typical layout:

```text
┌──────────────────────────────────────┐
│ Header                               │
├──────────────▶                       │
│ Slot directory      Free space       │
│                                      │
│                  ◀──────────── Cells │
└──────────────────────────────────────┘
```

* Slot entries grow from one side.
* Cells grow from the opposite side.
* Free space remains between them.

The diagram on PDF page 73 shows the header and pointer area at the top and variable-sized cells stored separately within the page. 

### Mental model

A slot is a stable apartment number; the cell is the tenant’s current physical location.

The tenant may move within the building, but external users still refer to the apartment number.

### Slot entry

A slot commonly stores:

* Cell offset
* Possibly cell length
* Status information

External references use:

```text
(page ID, slot ID)
```

rather than:

```text
absolute cell byte offset
```

### Benefits

| Requirement       | Slotted-page solution          |
| ----------------- | ------------------------------ |
| Variable records  | Cells can have different sizes |
| Stable references | Slot ID remains stable         |
| Reordering        | Sort slots, not cell bodies    |
| Deletion          | Remove or invalidate slot      |
| Compaction        | Move cells and update offsets  |
| Low overhead      | Small slot directory           |



### Cause and effect

```text
Indirection through slot
   ↓
Cell can move within page
   ↓
Compaction does not change external record ID
```

### Common mistakes

* Treating a slot ID as a physical byte offset.
* Forgetting to update the slot after moving a cell.
* Compacting a page while external users reference raw cell locations.
* Assuming slot order must equal physical cell order.

### Quick example

Before compaction:

```text
Slot 4 → offset 800
```

After moving the cell:

```text
Slot 4 → offset 500
```

External reference `(page 12, slot 4)` remains unchanged.

### Recall questions

1. How does slot indirection make page compaction safe?
2. Why can logical order differ from physical order?
3. Which information must change when a cell moves?
4. Why are page-local slot IDs preferable to exposed byte offsets?

---

## 12. Cell Layout

### Core idea

A cell is the smallest independently addressed record within a page.

Two common cell types are:

| Cell type      | Contains                      |
| -------------- | ----------------------------- |
| Key cell       | Separator key + child page ID |
| Key-value cell | Key + associated data record  |



## Key cell

Example layout:

```text
[key_size][child_page_id][key bytes]
```

Purpose:

* Used in internal B-Tree pages
* Routes searches to a child page

## Key-value cell

Example layout:

```text
[flags][key_size][value_size][key bytes][value bytes]
```

Purpose:

* Used in leaf pages
* Stores the actual record or value



### Page-level metadata

When every cell on a page shares properties, store those properties once in the page header.

Examples:

* All cells are key-only
* All values are fixed-size
* All keys use the same encoding

### Relationship

```text
More page-level metadata
   → less repeated cell metadata
   → smaller cells

But:
   → stronger uniformity assumptions
   → less per-cell flexibility
```

### Field ordering

Fixed-size fields are commonly placed first:

```text
[fixed header][variable key][variable value]
```

This makes header offsets statically known.

### Common mistakes

* Repeating cell-type metadata when the entire page has one type.
* Mixing fixed- and variable-sized cells without describing the distinction.
* Placing variable-sized data before fields needed to locate it.
* Failing to check that calculated boundaries remain inside the cell.
* Confusing child page IDs with page-local cell offsets.

### Recall questions

1. Why are fixed-size metadata fields commonly stored before variable data?
2. How does page-level metadata reduce cell size?
3. What distinguishes an internal-page cell from a leaf-page cell?
4. Why should cell boundaries be validated before reading key or value bytes?

---

## 13. Page IDs vs Cell Offsets

### Core idea

References use different coordinate systems at different levels.

| Reference   | Scope            | Meaning                                  |
| ----------- | ---------------- | ---------------------------------------- |
| Page ID     | File or database | Identifies a page                        |
| Cell offset | One page         | Locates bytes inside that page           |
| Slot ID     | One page         | Logical identifier mapped to cell offset |

### Page address

With fixed-size pages:

```text
file offset = page table lookup
```

or, in simpler layouts:

```text
file offset = base + page_id × page_size
```

### Cell address

```text
cell address = page start + cell offset
```

Because cell offsets are page-local, they can often use smaller integer types.



### Mental model

* Page ID: building number.
* Slot ID: apartment number.
* Cell offset: physical room position inside the building.

### Common mistakes

* Treating page IDs as raw file offsets.
* Using unnecessarily large integers for page-local offsets.
* Adding a cell offset to the beginning of the file instead of the page.
* Persisting an in-memory cache address as a page reference.

### Recall questions

1. Why can a cell offset use fewer bits than a file-wide offset?
2. What translation occurs between page ID and physical file position?
3. Which reference should remain stable when a cell is moved during compaction?

---

## 14. Variable-Sized Cells

### Core idea

A variable-sized cell needs enough fixed metadata to divide its payload into fields.

Example:

```text
[key_size][value_size][key bytes][value bytes]
```

Decoding:

```text
key_start   = header_end
key_end     = key_start + key_size
value_start = key_end
value_end   = value_start + value_size
```

An alternative stores total cell size and derives one field size through subtraction. 

### Rule

The representation must contain enough independent information to reconstruct every boundary.

### Safety checks

Before decoding:

```text
key_size ≥ 0
value_size ≥ 0
header + key_size + value_size ≤ cell boundary
cell boundary ≤ page boundary
```

### Trade-off

| Store every length | Derive some lengths               |
| ------------------ | --------------------------------- |
| Simpler decoding   | Smaller header                    |
| More metadata      | More arithmetic                   |
| Easier validation  | Greater dependence between fields |

### Common mistakes

* Allowing size arithmetic to overflow.
* Trusting total size without validating component sizes.
* Reading beyond the cell or page boundary.
* Forgetting that malformed length metadata can corrupt subsequent parsing.

### Recall questions

1. What minimum metadata is required to separate a key and value?
2. When can one size be derived rather than stored?
3. Which checks prevent a corrupted size from causing an out-of-bounds read?

---

## 15. Logical Order vs Physical Order

### Core idea

Slotted pages allow cells to be physically stored in insertion order while slots preserve sorted key order.

Example insertion order:

```text
Tom, Leslie, Ron
```

Physical cell order:

```text
Tom | Leslie | Ron
```

Logical slot order:

```text
Leslie → Ron → Tom
```



### How insertion works

1. Append the new cell to available physical space.
2. Find its sorted position using its key.
3. Insert a slot pointer at that logical position.
4. Shift only slot entries, not full records.

### Cause and effect

```text
Move small offsets instead of large records
   ↓
Cheaper sorted insertion
   ↓
Physical order can remain append-oriented
```

### Why it matters

A cell might contain kilobytes, while its offset may require only a few bytes.

Moving ten offsets is much cheaper than moving ten records.

### Trade-off

| Sorted physical cells         | Sorted slot directory           |
| ----------------------------- | ------------------------------- |
| Sequential physical key order | Cheap cell append               |
| Expensive middle insertions   | Extra indirection               |
| Less pointer lookup           | Physical fragmentation possible |
| Compaction may be simpler     | Slot maintenance required       |

### Common mistakes

* Assuming binary search requires physical cell order.
* Relocating every cell during each sorted insertion.
* Forgetting to shift slot entries after inserting a pointer.
* Treating insertion order as key order.

### Recall questions

1. How can binary search work when cells are physically unordered?
2. Why is shifting offsets cheaper than shifting records?
3. Which order does a range scan observe: physical cell order or slot order?
4. What additional indirection cost does this design introduce?

---

## 16. Deletion and Free-Space Tracking

### Core idea

Deleting a record does not necessarily move surrounding cells immediately.

Instead:

1. Mark the cell or slot as deleted.
2. Record the freed segment.
3. Reuse it for a future compatible cell.
4. Defragment later when necessary.

### Availability list

An availability list stores:

```text
(free offset, free length)
```

for each reusable segment.

SQLite calls such segments **freeblocks** and tracks related metadata in the page header. 

### Mental model

Deletion creates vacant rooms of different sizes. The allocator must find a room large enough for the next tenant.

### First fit

Choose the first sufficiently large segment.

**Benefits**

* Fast search
* Simple implementation

**Cost**

* Can leave small unusable remainders

### Best fit

Choose the segment that leaves the smallest remainder.

**Benefits**

* Attempts to reduce leftover waste

**Costs**

* More search work
* Can create many tiny fragments
* Does not guarantee optimal future allocation

 

### Common mistakes

* Assuming best fit always minimises long-term fragmentation.
* Reusing a segment without checking the new record’s size.
* Counting deleted bytes as one contiguous area.
* Forgetting to remove reused segments from the availability list.
* Allowing overlapping free segments.

### Recall questions

1. Why can first fit create unusable leftovers?
2. Why can best fit still cause fragmentation?
3. What information must an availability-list entry contain?
4. How can stale free-space metadata corrupt live records?

---

## 17. Defragmentation and Overflow

### Defragmentation

When enough total free space exists but it is fragmented:

```text
Read live cells
   ↓
Pack them together
   ↓
Update slot offsets
   ↓
Create one larger free region
```

Slot IDs remain stable while physical offsets change.

### Overflow page

If the record still does not fit after defragmentation, the system may allocate an overflow page.

```text
Main cell
   ↓
Pointer to overflow page(s)
   ↓
Remaining value bytes
```

### Decision sequence

```text
Can record fit in contiguous free space?
    ├─ Yes → insert
    └─ No
        ↓
Is total free space sufficient?
    ├─ Yes → defragment, then insert
    └─ No → use overflow page or split page
```



### Trade-offs

| Defragmentation           | Overflow                                 |
| ------------------------- | ---------------------------------------- |
| Preserves locality        | Avoids rewriting entire page immediately |
| Costs page rewrite        | Adds pointer traversal                   |
| Consolidates free space   | Can increase read amplification          |
| Updates many slot offsets | Requires overflow lifecycle management   |

### Key-value separation

Some systems store leaf keys separately from large values.

Benefits:

* More keys fit together
* Better lookup locality
* Key search loads fewer value bytes

Cost:

* Additional pointer or index required to locate values

### Common mistakes

* Allocating overflow pages before checking whether compaction is sufficient.
* Exposing physical offsets that become invalid after defragmentation.
* Ignoring overflow-page cleanup when a value is deleted.
* Storing large values inline when it severely reduces key density.
* Assuming total free space equals immediately usable space.

### Recall questions

1. Why do stable slot IDs make defragmentation practical?
2. When should an overflow page be considered?
3. How can separating keys and values improve search locality?
4. What new read cost is introduced by overflow pages?

---

## 18. File-Format Versioning

### Core idea

A storage engine may need to read files written by several software versions.

```text
New software
   ↓
Detect file version
   ↓
Select matching decoder
   ↓
Read legacy or current representation
```



### Version location options

| Location              | Example benefit             | Limitation                           |
| --------------------- | --------------------------- | ------------------------------------ |
| Filename prefix       | Detect without opening file | Naming becomes part of protocol      |
| Separate version file | Simple global version       | Must stay consistent with data files |
| File header           | Version travels with file   | Header prefix must remain readable   |
| Magic number          | Identifies kind or format   | Must reserve stable values           |

### Stable bootstrap problem

To read the version field, the reader must already know how that portion is encoded.

Therefore, some initial header region must have a stable format across versions.

### Backward compatibility

Possible strategies:

* Continue reading old formats.
* Upgrade files in place.
* Rewrite files into a new format.
* Support old and new readers temporarily.
* Reject unsupported versions clearly.

### Rule

Adding a field is not automatically compatible.

Compatibility depends on whether old readers can:

* Skip unknown fields
* Determine record length
* Recognise new flags
* Preserve unknown data
* Decode changed semantics

### Common mistakes

* Changing the layout of the version field itself.
* Reusing an old version identifier.
* Assuming software version equals file-format version.
* Removing legacy readers before all files are upgraded.
* Silently interpreting an unknown version as the current one.

### Quick example

A header begins with:

```text
[MAGIC][FORMAT_VERSION][PAGE_SIZE]
```

The first fields retain a stable representation, allowing the reader to choose the correct decoder for the rest.

### Recall questions

1. Why must the version-discovery region remain stable?
2. What are the operational risks of an in-place format upgrade?
3. Why should software and file-format versions be independent?
4. How can length-prefixed fields help forward compatibility?

---

## 19. Checksums and Corruption Detection

### Core idea

A checksum stores a compact summary of data so accidental changes can be detected later.

Write path:

```text
data
   ↓ compute checksum
[data][checksum]
```

Read path:

```text
read data + stored checksum
   ↓
recompute checksum
   ↓
compare
```

Mismatch means the bytes are not the same as when written. 

### Page checksums

Checksums are often computed per page rather than for the whole file.

Benefits:

* Validate only the page being read
* Isolate corruption
* Avoid reading the entire file
* Identify the damaged region
* Allow unaffected pages to remain usable

### Checksums vs CRCs vs cryptographic hashes

| Mechanism              | Primary purpose                     |
| ---------------------- | ----------------------------------- |
| Basic checksum         | Detect simple accidental corruption |
| CRC                    | Detect structured and burst errors  |
| Noncryptographic hash  | Fast accidental-change detection    |
| Cryptographic hash/MAC | Resist intentional manipulation     |

### Important exception

A CRC or ordinary checksum does **not** prove that data was not intentionally modified.

An attacker can often modify both:

* Data
* Corresponding noncryptographic checksum

### Cause and effect

```text
No integrity check
   ↓
Corrupted bytes may be interpreted as valid metadata
   ↓
Bad offsets, sizes or keys propagate damage
```

### Limits

A valid checksum means only:

* The checked bytes match the stored summary with high probability.

It does not prove:

* The writer generated logically correct data.
* The latest version was written.
* The page belongs at the correct file location.
* The data was not maliciously altered, unless a security mechanism is used.

### Common mistakes

* Using CRC as an authentication mechanism.
* Checksumming only payload while leaving critical metadata unchecked.
* Failing to specify whether the checksum field itself is excluded.
* Ignoring checksum mismatches and continuing to decode.
* Assuming a checksum detects logical application errors.
* Computing one whole-file checksum when pages are accessed independently.

### Quick example

A corrupted `value_size` changes from `100` to `10,000`.

Without a checksum, the reader may attempt to read outside the cell. With a page checksum, the corruption can be detected before interpreting the field.

### Recall questions

1. Why are page-level checksums useful for database files?
2. What does a valid checksum fail to guarantee?
3. Why should CRC not be used to detect malicious tampering?
4. Which parts of a page should normally be protected by integrity checks?
5. At what point should checksum verification occur relative to decoding?

---

## 20. File-Format Design Summary

### Core mental model

A durable page is a small, self-contained binary system:

```text
Page
├── Stable header
│   ├── Type/version
│   ├── Counts
│   ├── Free-space boundaries
│   └── Checksum
├── Slot directory
│   └── Logical ordering and cell references
├── Free space
└── Variable-size cells
    ├── Size metadata
    ├── Keys
    └── Values or child references
```

### Key relationships

| Design decision    | Benefit                        | Cost                                |
| ------------------ | ------------------------------ | ----------------------------------- |
| Fixed-size pages   | Simple addressing and caching  | Large records need overflow         |
| Length prefixes    | Clear boundaries               | Metadata overhead                   |
| Bit packing        | Compact representation         | More decoding complexity            |
| Slot indirection   | Stable IDs and easy compaction | Extra lookup                        |
| Logical slot order | Cheap physical append          | Physical and logical order differ   |
| Free-space list    | Reuse deleted regions          | Fragmentation management            |
| Defragmentation    | Recovers contiguous space      | Page rewrite                        |
| Version metadata   | Supports evolution             | Multiple readers or migration logic |
| Page checksums     | Local corruption detection     | CPU and storage overhead            |

### Rules to retain

* Define byte order explicitly.
* Never depend on raw runtime structure layout.
* Variable-sized fields need boundaries.
* Page IDs and cell offsets operate at different scopes.
* Slotted pages separate logical identity from physical location.
* Sorted slots allow unordered physical cells.
* Total free space is not the same as contiguous free space.
* Format version must be discoverable before decoding the variable portion.
* Checksums detect accidental corruption, not malicious modification.
* Validate lengths, offsets and checksums before trusting page contents.

### Applied recall questions

1. Design a cell containing a variable-sized key and two variable-sized values. Which fields belong in its fixed header?
2. A page reports 500 free bytes but cannot insert a 300-byte record. What is the likely cause and next action?
3. How can records remain externally addressable after page compaction?
4. When would sorting slot offsets be better than physically sorting cells?
5. A new file-format version adds a flag. What compatibility checks are needed before assigning it a bit?
6. Why might key-value separation improve B-Tree fanout?
7. A checksum passes, but a page contains semantically invalid keys. How is that possible?
8. How would you distinguish page corruption from reading the wrong file offset?

# Chapter 4: Implementing B-Trees

## 1. Implementation View

### Core idea

A real B-Tree implementation must manage more than lookup, split and merge algorithms.

It must also define:

* Page headers
* Page-to-page links
* Separator and pointer layouts
* Oversized values
* Parent tracking
* Space reclamation
* Compression
* Special-case insertion paths

### Mental model

```text
Abstract B-Tree
    +
Binary page format
    +
Navigation metadata
    +
Maintenance logic
    =
Usable storage engine
```

---

## 2. Page Headers

### Core idea

A page header contains metadata needed to interpret, navigate and maintain the page.

Common fields include:

* Page type or flags
* Number of cells
* Tree level
* Lower free-space boundary
* Upper free-space boundary
* Layout version
* Sibling links
* Rightmost child pointer
* Checksum
* Overflow-page pointer

Different systems store different implementation-specific values. 

### Mental model

The header is the page’s control panel:

```text
Page header
├── What kind of page is this?
├── How many records exist?
├── Where is free space?
├── Which pages are related?
└── How should the bytes be decoded?
```

### Why it matters

Without a trustworthy header, the reader cannot safely determine:

* Where cells begin
* Which layout version applies
* Whether the page is a leaf
* How to continue navigation
* Whether the page is valid

### Common mistakes

* Storing essential metadata only in memory.
* Failing to validate free-space boundaries.
* Updating page contents but not corresponding header fields.
* Adding header fields without handling version compatibility.
* Allowing cell regions and slot regions to overlap.

### Recall questions

1. Which header fields are required to locate free space in a slotted page?
2. Why might a leaf and an internal page require different header metadata?
3. What corruption could result from an incorrect cell count?
4. Which header fields must be updated after page compaction?

---

## 3. Magic Numbers

### Core idea

A **magic number** is a constant byte sequence used to identify or validate a file, page or format.

Example:

```text
50 41 47 45
```

This hexadecimal sequence represents the ASCII text `PAGE`.

### How it works

Write:

```text
header.magic = expected_constant
```

Read:

```text
if header.magic != expected_constant:
    reject page
```

A matching magic number makes it likely that:

* The reader is at the correct offset.
* The page is aligned correctly.
* The bytes represent the expected page type or format.



### Important limitation

A magic number is a **sanity check**, not a complete integrity guarantee.

It does not prove that:

* The rest of the page is uncorrupted.
* The page belongs at this location.
* The page was written completely.
* The data is authentic.

### Common mistakes

* Treating a matching magic number as full validation.
* Reusing the same magic number for incompatible formats.
* Reading variable fields before validating the magic number.
* Choosing a very short or common byte sequence.

### Recall questions

1. How can a magic number detect an incorrect page offset?
2. Why should checksum validation still be performed?
3. What should the reader do when a magic number is unknown?
4. How can magic numbers assist format version detection?

---

## 4. Sibling Links

### Core idea

Pages on the same B-Tree level may store direct links to their left and right siblings.

```text
Page A ↔ Page B ↔ Page C
```

Without sibling links, finding a neighbouring page can require:

1. Ascending to a parent.
2. Finding the next child pointer.
3. Descending back to the same level.

In some cases, the traversal may rise several levels before finding the next subtree. 

### Why sibling links matter

They improve:

* Leaf-level range scans
* Forward iteration
* Reverse iteration
* Some concurrent B-Tree algorithms
* Same-level navigation

### Trade-off

| Benefit                         | Cost                               |
| ------------------------------- | ---------------------------------- |
| Direct neighbouring-page access | Additional header fields           |
| Faster range traversal          | More pointer updates               |
| Avoids parent traversal         | More complex split and merge logic |
| Supports concurrent navigation  | Additional locking may be required |

### Split example

Before:

```text
A ↔ B ↔ C
```

Split `B` into `B` and `B2`:

```text
A ↔ B ↔ B2 ↔ C
```

Required updates may include:

* `B.right = B2`
* `B2.left = B`
* `B2.right = C`
* `C.left = B2`

The last update modifies a page other than the one being split, potentially requiring extra locking. 

### Common mistakes

* Updating only one direction of a doubly linked sibling chain.
* Forgetting to update the old right sibling after a split.
* Assuming sibling links never become temporarily inconsistent during concurrency.
* Following stale sibling links without validation.
* Using sibling links without defining their crash-recovery behaviour.

### Recall questions

1. Why can updating sibling links require locking an additional page?
2. What references change when a middle page splits?
3. When might parent-based traversal be preferred to sibling links?
4. How would a missing backward link affect reverse scans?

---

## 5. Rightmost Child Pointers

### Core idea

An internal B-Tree node with `N` separator keys requires `N + 1` child pointers.

Example:

```text
Pointers: P0   P1   P2   P3
Keys:          K1   K2   K3
```

Ranges:

```text
P0 → key < K1
P1 → K1 ≤ key < K2
P2 → K2 ≤ key < K3
P3 → key ≥ K3
```

The last pointer has no separator key to its right, so many implementations store it separately in the page header. 

### Mental model

Separator cells represent:

```text
[key boundary, child pointer]
```

but one final range extends beyond the last boundary:

```text
[last key, +∞)
```

That range needs the extra rightmost pointer.

### Rightmost-child split

When the rightmost child splits:

1. A promoted separator is inserted into the parent.
2. The promoted separator points to one resulting child.
3. The parent’s separate rightmost pointer is redirected to the new far-right child.

### Common mistakes

* Allocating the same number of pointers as separator keys.
* Losing the rightmost range during serialization.
* Leaving the old rightmost pointer after a child split.
* Pairing the promoted key with the wrong resulting child.
* Forgetting that exact separator conventions vary by implementation.

### Quick example

Before:

```text
Parent: [20, 50]
Children: P0, P1, P2
```

Split rightmost child `P2`, promoting `80`:

```text
Parent: [20, 50, 80]
Children: P0, P1, old-P2, new-P3
```

The new rightmost pointer must identify `new-P3`.

### Recall questions

1. Why are there more child pointers than separator keys?
2. Where can the unpaired pointer be stored?
3. Which pointer changes when the far-right child splits?
4. What key range does the rightmost pointer represent?

---

## 6. Node High Keys

### Core idea

Instead of treating the final child range as extending implicitly to `+∞`, a node may store a **high key** representing its upper key boundary.

```text
Node range: [low boundary, high key)
```

A high key is the highest key allowed within the node or subtree, depending on the exact design.

This technique is used in B-link Trees and simplifies some navigation and concurrency cases. 

### Without a high key

```text
Last pointer → keys greater than or equal to final separator
              with no explicit local upper bound
```

### With a high key

```text
Last pointer → keys below explicit high key
```

Each pointer can therefore be associated with a key boundary.

### Mental model

A normal separator says:

> “This child begins here.”

A high key additionally says:

> “This node’s responsibility ends here.”

### Why it matters

During concurrent splits, a search may reach an older page whose parent has not yet been fully updated.

The high key lets the reader recognise:

```text
searched key ≥ page high key
```

and move to the right sibling.

### Benefits

* Explicit node-range boundaries
* More uniform key-pointer pairing
* Fewer special cases around the rightmost pointer
* Useful for concurrent navigation
* Helps detect that a search landed on an outdated page

### Costs

* Additional stored key
* High-key updates during structural changes
* More invariants to maintain
* Coordination with sibling links

### Common mistakes

* Treating a high key as a stored user record.
* Failing to update high keys after split or merge.
* Confusing the node high key with its largest present key.
* Using inclusive and exclusive bounds inconsistently.
* Assuming every B-Tree implementation stores high keys.

### Recall questions

1. How does a high key differ from the largest actual key in a node?
2. Why is a high key useful during concurrent splits?
3. How can high keys simplify rightmost-pointer handling?
4. What should a search do when its key exceeds the page’s high key?

---

## 7. Overflow Pages

### Core idea

A fixed-size B-Tree page may be logically under capacity but physically unable to fit another large variable-sized value.

```text
Entry-count limit not reached
    but
Byte capacity exhausted
```

Instead of enlarging or relocating the page, the engine can store excess payload in linked **overflow pages**. 

### Structure

```text
Primary page
├── key
├── inline value prefix
└── overflow page ID
        ↓
Overflow page
├── remaining value bytes
└── next overflow page ID
        ↓
Additional overflow page
```

### Why fixed-size pages remain useful

Without overflow pages, supporting arbitrary-size values could require:

* Variable-size nodes
* Relocating pages
* Changing page-address calculations
* Reducing predictable caching
* Complicating free-space allocation

Overflow pages preserve fixed-size page management while permitting large logical records.

### Inline payload limit

Many systems define a maximum amount stored directly in the primary page:

```text
inline payload ≤ max_payload_size
```

The remainder spills into overflow pages.

This prevents one large value from consuming the whole page and destroying node fanout.

### Allocation process

1. Determine whether payload exceeds the inline limit.
2. Store an inline prefix in the primary page.
3. Reuse an existing overflow page if it has suitable free space.
4. Otherwise allocate a new overflow page.
5. Link additional pages when one is insufficient.



### Trade-offs

| Inline storage             | Overflow storage                 |
| -------------------------- | -------------------------------- |
| Fewer page reads           | Preserves primary-page density   |
| Better value locality      | Supports very large values       |
| Large values reduce fanout | Adds pointer traversal           |
| Simpler lifecycle          | Needs allocation and reclamation |

### Key-prefix optimisation

For oversized keys, keeping a key prefix in the primary page may allow most comparisons without reading overflow pages.

```text
Compare prefix
    ├── mismatch → decision without overflow read
    └── equal → fetch remaining key if needed
```

### Exception

If most records are very large, a B-Tree with overflow chains may be the wrong abstraction. Dedicated blob storage may be more suitable. 

### Common mistakes

* Storing a large value fully inline and severely reducing fanout.
* Failing to reclaim overflow pages after delete or shrink.
* Losing the overflow chain during crash recovery.
* Creating long chains without considering read amplification.
* Assuming overflow pages cannot fragment.
* Reading every overflow page merely to compare a key prefix.

### Recall questions

1. How can a node be logically nonfull but physically out of space?
2. Why does limiting inline payload protect fanout?
3. What metadata is needed to traverse an overflow chain?
4. When should specialised blob storage replace overflow pages?
5. How can storing a key prefix avoid overflow reads?

---

## 8. Binary Search Inside Pages

### Core idea

B-Tree navigation depends on keeping separator keys logically sorted.

Binary search returns either:

* An exact key position
* An insertion point
* The first key greater than the target



### Internal-node goal

At internal levels, exact matches are less important than choosing the correct child range.

```text
Find first separator > target
    ↓
Follow corresponding child pointer
```

### Leaf goal

At a leaf:

* Exact match → retrieve or update record.
* No exact match → identify insertion position or range starting point.

### Negative-result convention

Some APIs encode absence and insertion position together:

```text
result ≥ 0 → exact index
result < 0 → key absent; decode insertion point
```

The exact encoding differs by implementation.

### Common mistakes

* Returning only “not found” and discarding the insertion position.
* Using binary search when keys are not logically sorted.
* Following the separator rather than its associated child range.
* Applying internal-node equality semantics directly to leaf records.
* Mishandling duplicate keys or custom comparators.

### Recall questions

1. Why is the insertion point useful even when a key is absent?
2. What boundary does an internal-node search usually seek?
3. How can comparator inconsistency corrupt B-Tree navigation?
4. Why may most internal searches not produce an exact match?

---

## 9. Binary Search Through Indirection

### Core idea

In a slotted page, cells may remain in insertion order while slot offsets remain in key order.

Binary search therefore operates over slot entries:

```text
Sorted slot array
    ↓
Read middle slot
    ↓
Follow offset to physical cell
    ↓
Compare cell key
```



### Mental model

```text
Binary search indexes the directory,
not the physical occupants.
```

### Algorithm

1. Select the middle logical slot.
2. Read its cell offset.
3. Locate the physical cell.
4. Decode the key.
5. Compare it with the target.
6. Continue in the left or right portion of the slot array.

### Trade-off

| Benefit                                | Cost                         |
| -------------------------------------- | ---------------------------- |
| Cells need not be physically reordered | One extra indirection        |
| Cheap insertion of large records       | Less contiguous key access   |
| Stable slot-based references           | More pointer chasing         |
| Physical append remains simple         | Potential cache inefficiency |

### Common mistakes

* Binary-searching physical cell order.
* Comparing slot offsets instead of cell keys.
* Moving cells merely to maintain logical order.
* Updating a cell key without repositioning its slot.
* Leaving slots sorted according to an outdated comparator.

### Recall questions

1. Which structure must remain sorted for binary search?
2. What extra memory access does slot indirection introduce?
3. What must happen if an update changes a record’s key?
4. Why can physical insertion order remain unchanged?

---

## 10. Propagating Structural Changes

### Core idea

A leaf split or merge may require changes in its parent, which can propagate toward the root.

The implementation therefore needs a way to navigate upward.

Two main strategies:

1. Persistent or cached parent pointers
2. Breadcrumbs collected during descent



---

## 11. Parent Pointers

### Core idea

Each child may record which page is its parent.

```text
child.parent_page_id = parent
```

### Benefits

* Direct upward navigation
* Useful when propagating splits or merges
* May support same-level traversal through ancestors
* Avoids storing the complete path for long operations

### Costs

Parent pointers must change whenever a child moves between parents during:

* Parent split
* Parent merge
* Redistribution
* Rebalancing

### Persistent vs transient

A parent pointer may not need to be persisted if:

* The child was reached through a cached parent.
* The buffer manager tracks the relationship.
* The operation retains the relevant parent page in memory.

### Concurrency trade-off

Some implementations use parent-based traversal instead of sibling links to avoid certain deadlock patterns.

However, locating the next leaf through parents may require:

```text
ascend
    until a next child exists
descend
    to leaf level
```

### Common mistakes

* Failing to update child parent pointers after redistribution.
* Persisting unnecessary duplicate metadata.
* Trusting a stale parent pointer during concurrent structural changes.
* Assuming a direct parent always identifies the next leaf.
* Creating inconsistent parent-child relationships after crash recovery.

### Recall questions

1. Which operations can make a parent pointer stale?
2. Why might parent pointers remain only in memory?
3. How can parent-based leaf traversal avoid sibling-link locking?
4. Why may traversal need to ascend multiple levels?

---

## 12. Breadcrumbs

### Core idea

Instead of storing parent pointers, a root-to-leaf operation can record every visited page and child position.

```text
Breadcrumb stack:
[(root, child 2),
 (internal A, child 1),
 (internal B, child 4)]
```

If a split occurs, the stack is popped to find the immediate parent and propagate changes upward. 

### Why a stack fits

Traversal order:

```text
root → internal → internal → leaf
```

Propagation order:

```text
leaf → internal → internal → root
```

This is last-in, first-out.

### Breadcrumb contents

A breadcrumb may store:

* Parent page ID or reference
* Child-pointer index
* Separator position
* Latch or version information
* Other validation metadata

### Insert process

1. Traverse from root.
2. Push each visited internal page.
3. Modify the target leaf.
4. If it splits, pop its parent breadcrumb.
5. Insert the promoted separator.
6. Continue popping if the parent also splits.

### Important assumption

A stored cell index may become invalid if concurrent operations modify the parent.

The implementation may need to:

* Hold locks
* Validate page versions
* Repeat the search
* Recalculate the insertion position

### Parent pointers vs breadcrumbs

| Parent pointers                   | Breadcrumbs                          |
| --------------------------------- | ------------------------------------ |
| Stored relationship               | Per-operation path                   |
| Direct upward access              | No persistent parent metadata        |
| Must be maintained after movement | Stack discarded after operation      |
| Useful across operations          | Valid only for current traversal     |
| Can become stale                  | Can become invalid under concurrency |

### Common mistakes

* Collecting breadcrumbs only after discovering a split.
* Assuming recorded indices remain valid indefinitely.
* Popping breadcrumbs in root-to-leaf order.
* Storing raw in-memory pointers that may be evicted.
* Forgetting to include enough information to relocate the parent position.

### Recall questions

1. Why must breadcrumbs be collected before knowing whether a split occurs?
2. Why is a stack the natural structure?
3. How can concurrent parent modification invalidate a breadcrumb?
4. When are persistent parent pointers preferable?

---

## 13. Rebalancing Before Splitting or Merging

### Core idea

A node does not always need to split immediately on overflow or merge immediately on underflow.

Entries can first be redistributed between siblings.

```text
Overfull node + underfull sibling
    ↓
Move entries between them
    ↓
Delay split
```

Similarly:

```text
Underfull node + fuller sibling
    ↓
Borrow entries
    ↓
Delay merge
```



### Why it matters

Rebalancing can improve:

* Average occupancy
* Effective fanout
* Storage utilisation
* Tree height
* Number of structural operations

### Cost

Rebalancing modifies more pages:

* Source sibling
* Destination sibling
* Parent separators
* Possibly sibling metadata

This increases:

* Write work
* Locking scope
* Implementation complexity
* Recovery-log volume

### Parent separator update

Moving entries changes sibling boundaries.

Therefore, the parent’s separator key must also change.

```text
Before:
Left max < separator ≤ Right min

Move key between siblings
    ↓
Boundary changes
    ↓
Update separator
```

### B*-Tree strategy

B*-Trees attempt to keep nodes more densely occupied.

When two sibling nodes are full, they may be split into three nodes instead of splitting one node into two:

```text
2 full nodes
    ↓ redistribute
3 approximately ⅔-full nodes
```

This provides higher minimum occupancy than ordinary half-full splits. 

### Trade-off

| Immediate split                 | Rebalance first      |
| ------------------------------- | -------------------- |
| Simpler                         | Better occupancy     |
| Touches fewer existing siblings | May avoid new page   |
| Produces half-full nodes        | More pages modified  |
| Predictable path                | More complex locking |

### Common mistakes

* Moving entries without changing the parent separator.
* Redistributing between unrelated nodes.
* Rebalancing when the sibling cannot accept enough entries.
* Assuming higher occupancy always means lower total cost.
* Ignoring concurrent readers of both sibling pages.

### Recall questions

1. How can redistribution prevent a split?
2. Why must parent separators change after rebalancing?
3. How does a B*-Tree improve minimum occupancy?
4. When might an immediate split be cheaper than redistribution?
5. Why can better occupancy reduce tree height?

---

## 14. Right-Only Appends

### Core idea

Monotonically increasing keys create a predictable insertion pattern:

```text
1, 2, 3, 4, 5, ...
```

Every new largest key targets the rightmost leaf.

This enables specialised fast paths. 

### Fast-path insertion

If the rightmost leaf is cached and has space:

```text
new key > current maximum
    ↓
insert directly into cached rightmost leaf
    ↓
skip full root-to-leaf traversal
```

### Rightmost-page allocation

If the rightmost page is full, an implementation may:

1. Allocate a new page to its right.
2. Insert the new maximum there.
3. Add a pointer to the parent.
4. Avoid redistributing old entries.

The new page begins sparsely occupied, but continued appends are expected to fill it quickly. 

### Benefits

* Fewer tree traversals
* Less data movement
* Predictable split location
* Reduced fragmentation in internal pages
* Efficient append-heavy workloads

### Costs and assumptions

The optimisation relies on:

* Keys continuing to increase
* Few out-of-order inserts
* Few updates and deletes
* Rightmost-page metadata remaining valid

### Common mistake: hot spot

Monotonic keys concentrate writes on one page or one partition.

This can cause:

* Latch contention
* Cache-line contention
* One-node write concentration in distributed systems
* Uneven load

### Common mistakes

* Applying the fast path without confirming the key is a new maximum.
* Assuming auto-increment keys always distribute writes well.
* Leaving a new rightmost page sparse when future keys are not monotonic.
* Caching the rightmost page without detecting root or page replacement.
* Ignoring contention on the right edge.

### Recall questions

1. Which traversal can a cached rightmost leaf eliminate?
2. Why is a nearly empty new rightmost page acceptable?
3. What workload breaks the right-only-append assumption?
4. How can monotonic keys create a concurrency hot spot?
5. Why may internal pages be less fragmented under this workload?

---

## 15. Bulk Loading

### Core idea

When input is already sorted, building a B-Tree record by record wastes work.

A bulk loader can construct the tree page-wise from the bottom upward.

```text
Sorted records
    ↓
Fill leaf pages sequentially
    ↓
Collect first/high keys and page IDs
    ↓
Build parent pages
    ↓
Repeat until root
```



### How it works

1. Sort input, if not already sorted.
2. Pack records into leaf pages.
3. Write leaves sequentially.
4. Generate separator entries for parents.
5. Pack parent entries into internal pages.
6. Continue level by level.
7. Create the root last.

### Why it is faster

Bulk loading avoids or reduces:

* Repeated root-to-leaf searches
* Ordinary node splits
* Rebalancing
* Random writes
* Partially occupied intermediate states

### Memory use

Only a small active frontier may need to remain in memory:

* Current leaf
* Current parent pages
* Ancestors of the active right edge

### Immutable-tree advantage

If the built tree will never be modified, pages can be filled almost completely.

A mutable tree needs free space for later inserts; an immutable tree does not.

```text
Mutable tree
    → reserve update space

Immutable tree
    → maximise initial occupancy
```

### Trade-off

| Incremental insertion           | Bulk loading                   |
| ------------------------------- | ------------------------------ |
| Accepts arbitrary arrival order | Best with sorted input         |
| Immediately available           | Requires build phase           |
| Supports continuous writes      | Optimises initial construction |
| Causes splits over time         | Constructs pages directly      |
| Lower preprocessing             | May require sorting            |

### Common mistakes

* Bulk-loading unsorted input without sorting.
* Filling mutable-tree pages to 100%, leaving no update space.
* Using normal insertion for a complete sorted rebuild.
* Generating parent separators before leaf boundaries are known.
* Assuming bulk loading produces an optimal layout for future random writes.

### Recall questions

1. Why does sorted input eliminate most ordinary splits?
2. Why is the tree built from leaves upward?
3. How does immutable-tree bulk loading differ from mutable-tree loading?
4. Which part of the tree must remain in memory during construction?
5. When might sorting cost outweigh bulk-loading benefits?

---

## 16. Compression

### Core idea

Compression trades CPU and memory work for reduced storage and I/O.

```text
Higher compression
    → fewer stored bytes
    → more data per read

But:
    → more compression/decompression work
```



### Main evaluation metrics

* Compression ratio
* Compression speed
* Decompression speed
* Memory overhead

The best algorithm depends on:

* Data characteristics
* Page size
* Read/write ratio
* Latency requirements
* CPU availability

### Compression granularity

## Whole-file compression

**Benefit**

* Potentially high compression ratio

**Problems**

* Random page access becomes difficult.
* Updates may require recompressing large sections.
* Locating one page may require reading compression metadata or earlier data.
* Poor fit for mutable indexes.

## Page-level compression

Each page is compressed independently.

**Benefits**

* Random page access remains possible.
* Compression integrates with page load and flush.
* Updating one page does not require recompressing the file.

**Costs**

* Lower ratio than whole-file compression
* Compressed size may not align with storage blocks
* Page may cross block boundaries
* Metadata required to locate variable compressed extents

## Record- or column-level compression

Only stored data is compressed.

**Benefits**

* Page addressing remains independent.
* Selective fields can use different algorithms.
* Suitable when records or columns have distinct patterns.

**Costs**

* More per-record metadata or decoding
* May reduce effectiveness compared with larger compression units



### Block-alignment issue

A compressed page may:

* Occupy only part of a disk block, causing unrelated bytes to be read.
* Span an extra block, increasing I/O.

Compression ratio alone therefore does not directly predict read cost.

### Common mistakes

* Choosing the algorithm with the highest ratio without measuring decompression cost.
* Compressing an entire mutable index as one unit.
* Assuming compressed pages always require fewer physical blocks.
* Ignoring memory needed during decompression.
* Comparing algorithms on data unlike the production dataset.
* Forgetting that compressed size is variable.

### Recall questions

1. Why is whole-file compression poor for mutable random-access indexes?
2. How can a smaller compressed page still require an extra block read?
3. Which metric matters most for a read-heavy latency-sensitive workload?
4. When is record-level compression preferable to page-level compression?
5. Why must compression be benchmarked on representative data?

---

## 17. Vacuum and Maintenance

### Core idea

Foreground operations often leave cleanup work behind to reduce their immediate latency.

Background maintenance later:

* Reclaims dead space
* Defragments pages
* Rewrites cells
* Releases unused pages
* Restores physical organisation
* Updates free-page metadata



### Mental model

```text
Fast foreground update
    → leaves garbage or fragmentation

Background vacuum
    → pays deferred cleanup cost
```

### Addressable vs garbage data

A record is **live** when it can be reached from the B-Tree root through valid pointers.

A record is **garbage** when no valid pointer reaches it.

```text
Reachable from root → live
Unreachable          → garbage
```

Garbage bytes may still physically contain old values, but they are no longer part of the logical database. 

### Important distinction

```text
Physically present ≠ logically visible
```

Deleting a slot or pointer can make a cell unreachable without erasing its bytes.

### Why zero-filling is often skipped

Clearing every deleted byte would add writes.

Instead, the space is commonly:

* Marked reusable
* Overwritten by future records
* Reclaimed during compaction

### Security exception

Systems with strong data-remanence requirements may need explicit secure erasure, encryption-key destruction or storage-layer guarantees.

### Recall questions

1. What determines whether a cell is live?
2. Why can deleted data remain physically visible?
3. Why is zero-filling normally avoided?
4. How does logical reachability simplify garbage identification?

---

## 18. Fragmentation from Deletes and Splits

### Delete behaviour

In a slotted page, deletion may remove only the cell’s offset from the slot directory.

```text
Remove slot
    ↓
Cell bytes remain
    ↓
Cell becomes unreachable
    ↓
Hole remains
```

### Split behaviour

When cells are logically transferred to another page, an implementation may initially remove or truncate their slot entries while leaving old bytes behind.

Those bytes become garbage and can later be overwritten or reclaimed.



### Cause and effect

```text
Cheap logical deletion
    ↓
No immediate cell movement
    ↓
Scattered unused regions
    ↓
Fragmentation
    ↓
Later compaction required
```

### Why fragmentation matters

A page may have:

```text
total free bytes ≥ required bytes
```

but still lack:

```text
one contiguous segment ≥ required bytes
```

This may trigger:

* Page compaction
* Overflow allocation
* Page split
* Write failure or retry

### Common mistakes

* Treating unreachable data as available contiguous space immediately.
* Assuming page split physically removes all transferred cells.
* Counting garbage bytes without tracking their locations.
* Creating overflow pages before considering compaction.
* Ignoring the additional writes caused by deferred cleanup.

### Recall questions

1. Why does deleting a slot not immediately reclaim contiguous space?
2. How can page splitting create unreachable bytes?
3. Which measurement determines whether a new cell fits immediately?
4. When should compaction occur before allocating overflow storage?

---

## 19. Updates and Multiple Cell Versions

### Core idea

Updating a value in place may be impossible when its size changes.

A common approach is:

1. Write a new cell version.
2. Redirect the slot to the new cell.
3. Leave the old cell temporarily unreachable.
4. Reclaim the old version later.

```text
Slot → old cell

Update:
write new cell
Slot → new cell
old cell → garbage
```

Updates generally do not change leaf ordering when the key remains unchanged. 

### MVCC exception

An old version may remain logically reachable for older concurrent transactions.

```text
New transaction → newest version
Old transaction → earlier version
```

Such a record is not garbage until no transaction can still observe it.

### Ghost records

Some systems explicitly track deleted or obsolete records that remain needed by current transactions.

They can be reclaimed when all relevant transactions complete.

### Common mistakes

* Reclaiming an old version while an active transaction can still read it.
* Assuming an overwritten cell is immediately garbage.
* Changing a key without repositioning its slot.
* Updating a variable-sized value in place when the region is too small.
* Losing the relationship between record versions.

### Recall questions

1. Why may an update create a new cell instead of rewriting the old one?
2. When does an old version become reclaimable under MVCC?
3. Why does a same-key update usually avoid B-Tree structural changes?
4. What happens when the new value exceeds the available inline space?

---

## 20. Page Defragmentation

### Core idea

Defragmentation rewrites live cells into a compact physical layout.

```text
Before:
[live][hole][live][hole][live]

After:
[live][live][live][large free area]
```

### Process

1. Identify live cells.
2. Read or copy them.
3. Rewrite them contiguously.
4. Restore logical key order if desired.
5. Update slot offsets.
6. Recalculate free-space boundaries.
7. Update checksum and page metadata.
8. Release unused pages where possible.

### Synchronous vs asynchronous compaction

| Synchronous                       | Asynchronous                    |
| --------------------------------- | ------------------------------- |
| Runs when a write cannot fit      | Runs in background              |
| Avoids unnecessary overflow/split | Reduces foreground latency      |
| Adds write latency                | Requires maintenance scheduling |
| Guarantees immediate usable space | Garbage may persist longer      |

The broader maintenance process may be called:

* Compaction
* Vacuum
* Garbage collection
* Page maintenance



### Page relocation

Maintenance may move a page to a different physical position.

Stable page IDs or updated parent pointers are required so logical references remain valid.

### Free-page list

Pages no longer in use are recorded in a persistent **freelist**.

The freelist must survive crashes to prevent:

* Lost free space
* Reuse of live pages
* Storage leaks
* Double allocation

### Crash-safety concern

A page must not appear free until no live reference can reach it.

A safe update order or recovery log is required:

```text
Remove references durably
    ↓
Mark page free durably
    ↓
Allow page reuse
```

### Common mistakes

* Marking a page free before removing all references.
* Updating slot offsets but not the header boundaries.
* Failing to persist freelist changes.
* Running compaction without accounting for concurrent readers.
* Reusing a page whose deletion was not committed.
* Assuming physical relocation is invisible without updating references.

### Recall questions

1. What metadata changes when cells are compacted?
2. Why must freelist updates be crash-safe?
3. When is synchronous compaction justified?
4. What could happen if a page is reused before references are removed?
5. How can a storage engine keep page identity stable after relocation?

---

## 21. Chapter 4 Design Summary

### Core mental model

A practical B-Tree is a network of fixed-size pages whose metadata must remain consistent through navigation, mutation and cleanup.

```text
Page structure
    +
Tree invariants
    +
Cross-page links
    +
Structural propagation
    +
Deferred maintenance
    =
Operational B-Tree
```

### Key relationships

| Mechanism         | Main benefit                               | Main cost                       |
| ----------------- | ------------------------------------------ | ------------------------------- |
| Page header       | Self-contained interpretation              | Metadata maintenance            |
| Magic number      | Early sanity check                         | Limited validation              |
| Sibling links     | Fast same-level traversal                  | Extra updates and locks         |
| Rightmost pointer | Represents final key range                 | Special-case handling           |
| High key          | Explicit node upper bound                  | Extra invariant                 |
| Overflow page     | Supports large values                      | Read amplification              |
| Breadcrumb stack  | Upward propagation without parent pointers | Per-operation state             |
| Rebalancing       | Higher occupancy                           | More pages modified             |
| Right-only append | Fast monotonic insertion                   | Right-edge hot spot             |
| Bulk loading      | Efficient initial construction             | Requires sorted input           |
| Compression       | Lower storage and I/O                      | CPU and alignment cost          |
| Vacuum            | Reclaims deferred garbage                  | Background write work           |
| Freelist          | Reuses abandoned pages                     | Crash-safe persistence required |

### Rules to retain

* An internal node has one more child pointer than separator keys.
* A high key represents a page boundary, not necessarily a stored record.
* Large values should not destroy internal page fanout.
* Logical order may be maintained through slots while cells remain physically unordered.
* Splits and merges require an upward path through parent pointers or breadcrumbs.
* Redistribution can delay splits and merges but touches more pages.
* Sorted input should normally be bulk-loaded rather than inserted individually.
* A physically present cell may be logically unreachable.
* Total free space does not imply contiguous free space.
* Old record versions cannot be reclaimed while active transactions may still observe them.
* Freelist and page-reclamation changes must survive crashes safely.

### Applied recall questions

1. A search reaches a page whose high key is below the target key. What structural event may have occurred, and where should the search continue?
2. A 20 KiB value is inserted into a tree with 4 KiB pages. Design an inline-and-overflow representation that preserves useful fanout.
3. A leaf splits, but the implementation stores neither parent pointers nor breadcrumbs. What problem occurs next?
4. Redistribution raises occupancy but slows writes. Under which workload would that trade-off be worthwhile?
5. Why can monotonically increasing primary keys improve insertion efficiency yet worsen concurrency?
6. During vacuum, a page is added to the freelist before its parent pointer is removed. What failure can result after a crash?
7. A compressed page is 3 KiB on a system with 4 KiB blocks. Why might reading it still consume 4 KiB or more?
8. An old cell is unreachable from the latest slot directory but remains visible to one transaction. Is it garbage? Explain the deciding rule.
9. Which invariants and metadata must be updated when moving entries between two sibling pages?
10. How do high keys, sibling links and page versions together support safer concurrent navigation?

# Chapter 5: Transaction Processing and Recovery

## 1. Transactions and ACID

### Core idea

A transaction groups several database operations into one logical unit.

```text
BEGIN
  read A
  update A
  update B
COMMIT
```

The transaction should behave as one indivisible operation, even though it contains multiple reads and writes.

### ACID model

| Property        | Core guarantee                                              | Main mechanism                    |
| --------------- | ----------------------------------------------------------- | --------------------------------- |
| **Atomicity**   | All operations happen or none do                            | Undo, rollback, transaction log   |
| **Consistency** | Valid state becomes another valid state                     | Constraints and application logic |
| **Isolation**   | Concurrent transactions do not produce invalid interference | Locks, MVCC, validation           |
| **Durability**  | Committed changes survive failures                          | WAL, page flushing, recovery      |

 

## Atomicity

A transaction has two final outcomes:

```text
Commit → make all effects permanent
Abort  → remove all effects
```

A partially applied transaction must not remain visible as a valid result.

### Important exception

Abort may require physical cleanup after the logical transaction has already been declared failed.

## Consistency

Consistency means preserving database invariants, such as:

* Unique-key constraints
* Referential integrity
* Valid account balances
* Application-specific business rules

### Important distinction

Consistency is not supplied entirely by the storage engine.

The database can enforce declared constraints, but the application must define correct invariants and transaction logic.

## Isolation

Concurrent transactions should produce results equivalent to an allowed execution in which their interference is controlled.

Practical databases often weaken isolation to improve throughput.

## Durability

Once commit is acknowledged, the system must have enough persistent information to reconstruct the committed state after:

* Process crash
* Operating-system failure
* Power loss
* Node restart

### Common mistakes

* Treating consistency as the same concept as replica consistency.
* Assuming a transaction is atomic merely because each individual statement is atomic.
* Acknowledging commit before the required log record is durable.
* Assuming all isolation levels provide serializability.
* Believing rollback means the original bytes are immediately erased.

### Quick example

Transferring £100:

```text
1. Subtract £100 from account A
2. Add £100 to account B
```

Atomicity prevents only step 1 from remaining after failure.

Consistency requires the total balance invariant to remain correct.

Isolation prevents concurrent transfers from reading misleading intermediate states.

Durability ensures the committed transfer survives restart.

### Recall questions

1. Which ACID property prevents half of a transfer from remaining?
2. Why can a database not guarantee application consistency by itself?
3. How can a transaction be durable before its modified pages are flushed?
4. Why do weaker isolation levels usually improve concurrency?

---

## 2. Components Supporting Transactions

### Core idea

Transactions emerge from coordination among several storage-engine components.

| Component               | Responsibility                                     |
| ----------------------- | -------------------------------------------------- |
| **Transaction manager** | Tracks, schedules, commits and aborts transactions |
| **Lock manager**        | Controls conflicting access to logical resources   |
| **Page cache**          | Holds and modifies database pages in memory        |
| **Log manager**         | Persists operations needed for redo and undo       |
| **Recovery manager**    | Reconstructs a correct state after failure         |



### Mental model

```text
Transaction manager
        │
        ├── coordinates locks
        ├── changes cached pages
        └── records durable log entries
```

### Cause and effect

```text
Page cache permits deferred disk writes
    ↓
Unflushed changes can disappear in a crash
    ↓
WAL preserves enough information to reconstruct them
```

### Common mistake

Thinking that transaction support belongs to one isolated component.

Correctness depends on agreements between:

* Cache flushing
* Log flushing
* Lock release
* Commit acknowledgement
* Recovery

### Recall questions

1. Why must the log manager coordinate with the page cache?
2. Which component determines whether a requested exclusive lock can be granted?
3. What information is required to undo an aborted transaction?

---

# Buffer Management

## 3. Page Cache

### Core idea

A page cache stores frequently used disk pages in RAM.

```text
Request page
    ↓
Is page cached?
 ├─ Yes → return cached page
 └─ No  → read from disk and cache
```

The cache is also called a **buffer pool**, although the book prefers “page cache.” 

### Why it matters

RAM access is faster than persistent-storage access.

Caching provides two major benefits:

* Repeated reads avoid disk I/O.
* Several writes to one page can be combined before flushing.

### Page states

| State          | Meaning                                       |
| -------------- | --------------------------------------------- |
| **Cached**     | An in-memory copy exists                      |
| **Clean**      | Memory and disk versions match                |
| **Dirty**      | Memory contains changes not reflected on disk |
| **Referenced** | A current operation is using the page         |
| **Pinned**     | The eviction system must retain the page      |
| **Evictable**  | The page may be removed from cache            |

### Important rule

The physical order of cache slots does not need to match page order on disk.

```text
Disk page 100 → cache slot 3
Disk page 4   → cache slot 8
Disk page 72  → cache slot 1
```

### Page-cache responsibilities

* Locate cached pages
* Translate page IDs into disk locations
* Read missing pages
* Track references
* Buffer changes
* Mark dirty pages
* Select eviction victims
* Flush dirty pages
* Prevent active pages from eviction



### Common mistakes

* Treating a cached page as a separate logical copy that may diverge freely from disk.
* Evicting a page while another thread still uses it.
* Losing track of whether a page is dirty.
* Assuming cache position can be derived from page ID.
* Assuming a large cache removes the need for recovery.

### Recall questions

1. What changes when a clean page becomes dirty?
2. Why can multiple writes to a cached page reduce disk I/O?
3. Which page states prevent immediate eviction?
4. Why are cache-slot order and disk-page order independent?

---

## 4. Database Cache vs Kernel Page Cache

### Core idea

The operating system may cache file contents, while the database maintains its own page cache.

Without direct I/O:

```text
Database cache
     ↓
Kernel page cache
     ↓
Storage device
```

This can create duplicate caching and reduce database control.

### Direct I/O

Some databases use mechanisms such as `O_DIRECT` to bypass the kernel page cache and perform database-specific buffering.

Benefits:

* Explicit eviction control
* Predictable memory ownership
* Database-aware prefetching
* Reduced duplicate caching

Costs:

* Database must implement more I/O logic
* Direct-I/O restrictions may complicate alignment
* Kernel readahead may be unavailable
* Platform behaviour differs



### Memory mapping alternative

Memory mapping can reduce explicit read/write system calls, but gives the operating system more control over paging and eviction.

### Trade-off

| Database-managed I/O        | OS-managed I/O                           |
| --------------------------- | ---------------------------------------- |
| Workload-aware policies     | Simpler application                      |
| Explicit dirty-page control | Kernel handles paging                    |
| More implementation effort  | Less predictable eviction                |
| Can avoid duplicate cache   | May benefit from general OS optimisation |

### Common mistakes

* Assuming bypassing the kernel is always faster.
* Maintaining two large caches without accounting for duplicated memory.
* Expecting advisory calls to guarantee kernel behaviour.
* Using direct I/O without satisfying alignment requirements.

### Recall questions

1. Why might two cache layers waste memory?
2. What control does direct I/O return to the database?
3. Why might a database still prefer the kernel cache?
4. What cache-management control is lost with memory mapping?

---

## 5. Cache References, Pinning and Dirty Pages

### Core idea

When the storage engine requests a page, it receives a reference to a cached buffer.

The engine must release that reference when finished.

```text
reference(page)
    ↓
read or modify
    ↓
dereference(page)
```

A page that must remain resident can be **pinned**. 

### Referenced vs pinned

| Referenced               | Pinned                               |
| ------------------------ | ------------------------------------ |
| In active use now        | Intentionally retained               |
| Temporary protection     | Potentially long-lived               |
| Released after operation | Released by explicit policy          |
| Prevents unsafe eviction | Improves expected future performance |

### Dirty-page rule

A modified cached page is marked dirty:

```text
cached version ≠ disk version
```

It must eventually be flushed, but durability may already be protected by the WAL.

### Pinning higher B-Tree levels

Root and upper internal pages are accessed by nearly every lookup.

Because there are relatively few upper-level pages, they can often remain permanently cached.

```text
Tree height = 4
Root and level 1 pinned
    ↓
Lookup may require disk I/O only for levels 2–4
```



### Benefit of buffered structural changes

Several in-memory changes can cancel or combine before disk flush:

```text
split-related change
merge-related change
additional update
       ↓
one final page write
```

### Common mistakes

* Using “pinned” and “locked” interchangeably.
* Forgetting to decrement reference counts.
* Pinning too many pages and leaving no eviction candidates.
* Assuming dirty means corrupted.
* Flushing every small modification immediately.

### Recall questions

1. Why are upper B-Tree pages strong pinning candidates?
2. How does pinning differ from transaction locking?
3. What happens if all cache pages are pinned?
4. Why can delaying a dirty-page flush reduce writes?

---

## 6. Cache Eviction

### Core idea

When the cache is full, one page must leave before another can enter.

```text
Need new page
    ↓
Select victim
    ↓
Is victim dirty?
 ├─ Yes → flush safely
 └─ No  → evict directly
```

### Eviction conditions

A page can generally be evicted when:

* It is not referenced.
* It is not pinned.
* Its dirty contents have been safely flushed.
* WAL ordering requirements are satisfied.



### Background flushing

Flushing only when eviction is required places I/O directly on the request path.

A background writer can pre-flush likely victims:

```text
Background:
dirty page → flush → clean eviction candidate

Foreground:
need slot → evict quickly
```

### Competing goals

* Delay writes to combine modifications.
* Flush early enough to avoid eviction stalls.
* Avoid flooding storage with background writes.
* Keep cache inside its memory limit.
* Preserve durability.
* Select low-value pages for eviction.



### Common mistakes

* Evicting dirty pages before their log records are durable.
* Performing every flush synchronously on the user request path.
* Letting the background writer saturate storage.
* Choosing victims without considering whether they are currently referenced.
* Keeping dirty pages indefinitely and creating large recovery work.

### Recall questions

1. Why can a clean page be evicted more cheaply than a dirty one?
2. How does background flushing reduce tail latency?
3. Why should dirty pages not always be flushed immediately?
4. Which ordering constraint links WAL flushing and page eviction?

---

## 7. Prefetching and Immediate Eviction

### Prefetching

The database can load a page before it is explicitly requested.

Example: range scan

```text
Read leaf 10
    ↓
Predict leaf 11
    ↓
Load leaf 11 while processing leaf 10
```

### Immediate eviction

Some pages are unlikely to be reused after one operation, such as pages read during:

* Full sequential scans
* Backup
* Vacuum
* Bulk maintenance

These pages can be placed in a temporary or circular cache region and evicted quickly.



### Cache pollution

A large sequential scan can replace highly reused pages with one-time pages:

```text
Hot index pages evicted
    ↓
Scan pages used once
    ↓
Later queries reload hot pages
```

A dedicated scan buffer prevents the full scan from destroying the main working set.

### Trade-off

| Aggressive prefetch     | Conservative prefetch                |
| ----------------------- | ------------------------------------ |
| Hides I/O latency       | Lower wasted I/O                     |
| Helps predictable scans | Better for random access             |
| Uses more cache space   | May stall when prediction is correct |
| Can read unused pages   | Less cache pollution                 |

### Common mistakes

* Prefetching random access patterns with low predictability.
* Allowing one large scan to occupy the entire cache.
* Immediately evicting a page that may soon be revisited.
* Prefetching faster than the consumer can process.

### Recall questions

1. How does prefetching hide storage latency?
2. Why can full-table scans harm unrelated queries?
3. When is immediate eviction appropriate?
4. How can over-prefetching reduce performance?

---

# Page-Replacement Policies

## 8. Replacement Policy Mental Model

### Core idea

The eviction policy approximates this impossible ideal:

> Evict the page whose next access is furthest in the future.

Since future access is unknown, practical policies use past behaviour as a prediction.



### Possible signals

* Time loaded
* Most recent access
* Access frequency
* Number of recent accesses
* Scan classification
* Page type
* Dirty status
* Eviction cost

### Bélády’s anomaly

For some policies such as FIFO, increasing cache capacity can unexpectedly increase the number of misses.

The problem is not that larger caches are inherently harmful; it is that some policies do not preserve useful inclusion properties. 

### Common mistakes

* Assuming the oldest page is the least useful.
* Evaluating only hit ratio while ignoring eviction and flush cost.
* Assuming a larger cache compensates for a poor policy.
* Applying one policy identically to point queries and sequential scans.

### Recall questions

1. What future information would an optimal policy need?
2. Why is recency only an approximation of reuse?
3. How can increasing FIFO cache capacity increase misses?
4. Why might a dirty page be retained over a clean page even if both seem equally cold?

---

## 9. FIFO

### How it works

FIFO evicts the page loaded earliest.

```text
Load order:
A → B → C → D

First victim:
A
```

### Benefit

* Simple queue
* Low tracking overhead

### Failure mode

It ignores all accesses after initial loading.

A root page loaded at startup may be frequently used but still become the first eviction candidate. 

### Common mistake

Equating “oldest loaded” with “least likely to be used again.”

### Recall questions

1. Why does FIFO perform poorly for repeatedly accessed root pages?
2. Which workload might make FIFO acceptable?
3. What information does FIFO discard?

---

## 10. LRU and Variants

## Least Recently Used

LRU evicts the page whose latest access occurred furthest in the past.

```text
Access A
Access B
Access A

Recency:
B older than A
```

### Benefit

Recent use often predicts near-future reuse.

### Cost

Updating a global ordering on every access can become expensive under concurrency.



## 2Q

Two queues distinguish:

* Recently accessed once
* Repeatedly accessed and therefore “hot”

This prevents a one-time scan page from receiving the same status as a frequently reused page.

## LRU-K

LRU-K tracks the last `K` accesses rather than only the latest one.

Mental model:

```text
One access  → possibly accidental
Many accesses → stronger reuse evidence
```

### Common mistakes

* Implementing exact LRU with a globally contended list.
* Letting sequential scans make every page appear recently used.
* Treating one recent access as proof of frequent reuse.
* Forgetting to remove evicted pages from recency metadata.

### Recall questions

1. Why is exact LRU expensive in concurrent systems?
2. How does 2Q distinguish scan pages from hot pages?
3. What extra evidence does LRU-K collect?
4. When can recency be a poor predictor?

---

## 11. CLOCK

### Core idea

CLOCK approximates LRU using a circular array and access bits.

```text
[A:1] [B:0] [C:1] [D:0]
        ↑ clock hand
```

### Algorithm

When the hand inspects a page:

* Access bit `1` → clear it and continue.
* Access bit `0` → consider the page for eviction.
* Currently referenced page → skip it.



### Mental model: second chance

A recently used page receives one more cycle before eviction.

### Benefits

* Compact metadata
* No exact recency list
* Cache-friendly
* Lower coordination cost
* Can use atomic operations



### Variants

Some implementations use counters rather than one-bit flags:

```text
Higher counter → more chances before eviction
```

### Common mistakes

* Clearing the bit of a page currently in active use.
* Assuming CLOCK reproduces exact LRU order.
* Allowing the hand to loop indefinitely when every page is pinned.
* Failing to account for dirty-page flush cost.

### Recall questions

1. Why is CLOCK called a second-chance algorithm?
2. How does it reduce LRU metadata overhead?
3. What happens when every candidate is referenced or pinned?
4. How could counters make CLOCK frequency-aware?

---

## 12. LFU and TinyLFU

### Least Frequently Used

LFU prioritises access frequency rather than access recency.

```text
Page A: 100 accesses
Page B: 2 accesses

B is a stronger eviction candidate
```

### Limitation of simple LFU

A page that was popular long ago can remain protected indefinitely unless counts decay.

## TinyLFU

TinyLFU uses a compact frequency estimator and separates pages into logical regions:

| Region        | Purpose                       |
| ------------- | ----------------------------- |
| **Admission** | Newly seen pages              |
| **Probation** | Pages competing for retention |
| **Protected** | Frequently reused pages       |

A candidate is promoted only when its estimated frequency justifies displacing another page.  

### Mental model

Instead of asking only:

> Which page should leave?

TinyLFU also asks:

> Does this new page deserve admission over an existing page?

### Trade-off

| Frequency-based                | Recency-based                    |
| ------------------------------ | -------------------------------- |
| Protects repeatedly used pages | Adapts quickly to recent changes |
| Resistant to one-time scans    | Simpler metadata                 |
| Needs approximate counters     | Can forget long-term popularity  |
| Requires ageing/decay          | Vulnerable to cache pollution    |

### Common mistakes

* Maintaining unbounded lifetime frequency counts.
* Admitting every new page into the protected region.
* Treating approximate frequency estimates as exact.
* Ignoring workload phase changes.

### Recall questions

1. Why can lifetime LFU retain obsolete pages?
2. What decision does TinyLFU make before promotion?
3. How does an admission region protect the main cache?
4. Under what workload could LRU outperform frequency-based retention?

---

# Recovery

## 13. Write-Ahead Logging

### Core idea

A write-ahead log is an append-only record of database changes used for transaction and crash recovery.

```text
Modify cached page
    ↓
Page may remain only in RAM
    ↓
WAL preserves the change durably
```



### Fundamental WAL rules

1. A log record describing a change must be created before the modified page can be safely flushed.
2. The log record must reach durable storage before the corresponding dirty page reaches durable storage.
3. A commit record must be durable before commit is acknowledged.
4. Log records must be flushed in LSN order.

 

### Mental model

```text
WAL = durable instructions
Page file = materialised result
```

If the materialised result is lost or incomplete, the instructions can recreate it.

### Why append-only helps

* Sequential writes
* Simpler ordering
* Efficient batching
* Immutable earlier records
* Readers can scan stable log prefixes while the writer appends

### Common mistakes

* Flushing a dirty page before its WAL record.
* Acknowledging commit when only an in-memory log buffer contains the commit.
* Writing log records out of LSN order.
* Truncating log records merely because a transaction committed.
* Assuming WAL eliminates all need for page checksums.

### Quick example

```text
LSN 100: update page 7
LSN 101: commit transaction T
```

Before acknowledging `T`:

* The log must be durable through LSN 101.
* Page 7 itself may still be dirty in RAM under a no-force policy.

### Recall questions

1. Why can a transaction commit while its data page remains unflushed?
2. What must be durable before a dirty page is written?
3. Why is the WAL sequential?
4. Why can committed log records not always be discarded immediately?

---

## 14. Log Sequence Numbers

### Core idea

Each WAL record receives a monotonically increasing **log sequence number**.

```text
LSN 100
LSN 101
LSN 102
...
```

LSNs establish:

* Log order
* Recovery position
* Page freshness
* Checkpoint boundaries
* Commit durability threshold

### Page LSN

A page can record the LSN of the latest change applied to it.

During redo:

```text
page_LSN ≥ record_LSN
    → change already applied

page_LSN < record_LSN
    → redo may be required
```

### Force operation

A log force flushes the log buffer through a requested LSN.

```text
force(LSN 250)
```

means records up to that point must reach durable storage before returning successfully.

### Common mistakes

* Treating LSN as a wall-clock timestamp with perfect temporal meaning.
* Reapplying a log record without checking page state.
* Allowing commit acknowledgement before forcing through the commit LSN.
* Reusing LSN values after restart.

### Recall questions

1. How does a page LSN make redo idempotent?
2. What guarantee does forcing through a particular LSN provide?
3. Why must LSNs be monotonically ordered?
4. How do checkpoints use LSNs?

---

## 15. Commit Records and Compensation Records

## Commit record

A transaction is not durable merely because its data operations have log entries.

Its commit decision must also be logged and forced.

```text
operation records
    ↓
commit record
    ↓
force log
    ↓
acknowledge commit
```

## Compensation log record

A compensation log record records work performed while undoing another log record.

Purpose:

* Make rollback itself recoverable.
* Prevent the same undo action from being repeated incorrectly after another crash.
* Record progress through recovery.



### Mental model

```text
Normal log record: “I changed X.”
CLR:               “I reversed that change.”
```

### Common mistakes

* Assuming recovery cannot crash.
* Performing undo without logging its progress.
* Treating a compensation record as an ordinary user transaction update.
* Undoing an operation twice after repeated crashes.

### Recall questions

1. Why must the commit decision itself be durable?
2. What problem do compensation log records solve?
3. Why must rollback be restartable?
4. How can a CLR prevent repeated undo?

---

## 16. Checkpoints

### Core idea

A checkpoint establishes a position from which recovery can begin without scanning the entire historical log.

```text
Earlier WAL records
    ↓
Changes reflected safely in database pages
    ↓
Checkpoint boundary
    ↓
Earlier log may eventually be trimmed
```



## Synchronous checkpoint

Flush all required dirty pages before completing the checkpoint.

### Problem

This may:

* Pause normal work
* Produce a large I/O spike
* Increase latency
* Require substantial time

## Fuzzy checkpoint

A fuzzy checkpoint records the recovery state while transactions continue.

Typical information:

* Dirty-page table
* Active-transaction table
* Checkpoint starting LSN
* Checkpoint completion record

Pages may be flushed asynchronously after checkpoint initiation.  

### Mental model

A checkpoint does not necessarily mean:

> Every page is clean at this exact instant.

It means:

> Recovery has a durable description of where to begin and which work may remain.

### Log-trimming rule

A WAL segment can be removed only when no recovery path still depends on it.

That may require considering:

* Dirty pages
* Active transactions
* Replication
* Backups
* Archive recovery
* Long-running snapshots

### Common mistakes

* Truncating the WAL at checkpoint start rather than successful completion.
* Assuming a fuzzy checkpoint flushes every page immediately.
* Pausing all transaction processing unnecessarily.
* Forgetting active transactions when calculating the recovery boundary.
* Treating checkpoints as substitutes for backups.

### Recall questions

1. Why does checkpointing reduce restart time?
2. What does a fuzzy checkpoint record?
3. Why can log truncation lag behind commit?
4. Which state can prevent an old WAL segment from being deleted?

---

## 17. Storage and `fsync` Failure Semantics

### Core idea

Recovery correctness depends on the actual guarantees provided by the storage stack.

A successful-looking write path may still fail because of:

* Delayed writeback
* Device-cache behaviour
* Lost error reporting
* Filesystem semantics
* Partial writes
* Torn pages
* Hardware failure

The book discusses historical cases where writeback errors and dirty-page handling could make reliable failure detection difficult. 

### Mental model

```text
Application requested flush
    ≠ automatically
Bytes safely stored forever
```

The database must understand and test its complete persistence path.

### Rule

Recovery mechanisms should be tested under injected failures, including:

* Failure before log write
* Failure during log write
* Failure after log but before page write
* Failure during page write
* Failure during checkpoint
* Failure during recovery
* Repeated recovery failure

### Common mistakes

* Assuming all storage devices honour flushes identically.
* Ignoring write errors after a retry.
* Testing only clean shutdown.
* Assuming a system call contract is implemented perfectly on every platform.
* Failing to test crash recovery during checkpoint or rollback.

### Recall questions

1. Why does database durability depend on lower storage layers?
2. Which failure point tests the WAL ordering guarantee?
3. Why must recovery itself be fault-tested?
4. What is a torn-page scenario?

---

## 18. Physical vs Logical Logging

### Core idea

A log can describe either data state or operations.

## Physical logging

Records exact byte or page changes:

```text
Before image
After image
```

Examples:

* Replace bytes 100–120.
* Restore the full previous page.
* Write this exact page image.

## Logical logging

Records the operation’s meaning:

```text
Insert key K with value V
Delete key K
Increment field X
```



### Comparison

| Physical log                  | Logical log                         |
| ----------------------------- | ----------------------------------- |
| Tied to page representation   | Tied to operation semantics         |
| Fast, direct redo             | More flexible application           |
| Can be large                  | Often smaller                       |
| Requires correct page state   | Requires operation interpretation   |
| Easier byte-level restoration | Better for logical undo/concurrency |

### Physiological logging

Many systems combine both ideas:

* Identify a physical page.
* Describe a logical change within that page.

### Before-image and after-image

```text
redo(before-image) → after-image
undo(after-image)  → before-image
```

### Shadow paging alternative

Copy-on-write can place updated content in a new page, then atomically switch a pointer from the old page to the new page.

Benefits:

* Old version remains intact until publication.
* Atomic pointer switch can publish the change.

Costs:

* Page copying
* Fragmentation
* Pointer maintenance
* Garbage collection
* Poor fit for some concurrent workloads

### Common mistakes

* Applying a logical operation to an unexpected page state.
* Logging full pages for every tiny modification without measuring overhead.
* Assuming physical logs require no operation metadata.
* Treating shadow paging as free of recovery or reclamation complexity.
* Using an undo operation that is not the true inverse under concurrency.

### Recall questions

1. Why can physical redo be faster than logical redo?
2. Why is logical undo useful under concurrency?
3. What page-state assumption must be validated before applying a log record?
4. How does shadow paging publish a new version?

---

## 19. Steal and Force Policies

### Core idea

Two independent decisions determine when dirty pages may reach disk.

## Steal vs no-steal

**Steal:** the cache may flush a page containing uncommitted changes.

**No-steal:** pages with uncommitted changes cannot be flushed.

## Force vs no-force

**Force:** all pages changed by a transaction must be flushed before commit completes.

**No-force:** commit can complete before those data pages are flushed.

 

### Recovery requirements

| Policy       | Consequence                                                   |
| ------------ | ------------------------------------------------------------- |
| **Steal**    | Uncommitted data may be on disk → **undo required**           |
| **No-steal** | Uncommitted data stays in RAM → disk undo not required        |
| **Force**    | Committed changes already on pages → redo usually unnecessary |
| **No-force** | Committed pages may be unflushed → **redo required**          |

### Four combinations

| Combination         | Undo? | Redo? | Main trade-off                               |
| ------------------- | ----: | ----: | -------------------------------------------- |
| No-steal + force    |    No |    No | Simple recovery, expensive memory and commit |
| No-steal + no-force |    No |   Yes | More memory pressure                         |
| Steal + force       |   Yes |    No | Slow commit, permits eviction                |
| Steal + no-force    |   Yes |   Yes | Flexible and efficient; complex recovery     |

### Mental model

```text
Steal asks:
“May uncommitted changes reach disk?”

Force asks:
“Must committed changes reach data pages before commit?”
```

### Why steal is useful

A long transaction may dirty more pages than fit in memory.

Steal allows some pages to be flushed and evicted before transaction completion.

### Why no-force is useful

Many transactions may update the same page.

No-force allows:

* Log commit now
* Combine page changes
* Flush the page later once

### Common mistakes

* Confusing force of the WAL with force of data pages.
* Assuming no-force weakens durability.
* Forgetting undo information under steal.
* Forgetting redo information under no-force.
* Treating steal and force as opposites; they are separate axes.

### Recall questions

1. Which policy permits uncommitted data on disk?
2. Why does no-force require redo?
3. Why can steal reduce cache pressure?
4. Which combination does ARIES use?
5. Why is WAL forcing still required under no-force?

---

## 20. ARIES

### Core idea

ARIES is a recovery approach designed for **steal/no-force** systems.

It combines:

* WAL
* Physical redo
* Logical undo
* Fuzzy checkpoints
* Dirty-page tracking
* Compensation log records
* Repeat-history recovery



### Recovery phases

## Phase 1: Analysis

Determine:

* Which transactions were active at crash time
* Which transactions committed
* Which pages may be dirty
* Where redo should begin

Outputs include:

* Transaction table
* Dirty-page table

## Phase 2: Redo

Repeat history from the required point.

Redo includes operations from:

* Committed transactions whose pages were unflushed
* Transactions that were still incomplete at crash time

The purpose is to reconstruct the exact pre-crash state before selectively undoing losers.

## Phase 3: Undo

Roll back incomplete transactions in reverse log order.

Undo progress is itself logged using compensation records.



### Mental model

```text
Analysis: What was happening?
Redo:     Rebuild what happened.
Undo:     Remove what never committed.
```

### Why redo incomplete transactions?

It may seem wasteful to redo work that will later be undone.

The benefit is a uniform process:

1. Recreate the precise historical state.
2. Apply correct transaction-level rollback.
3. Support idempotent repeated recovery.

### Winner and loser transactions

| Term       | Meaning                        |
| ---------- | ------------------------------ |
| **Winner** | Committed before crash         |
| **Loser**  | Active or uncommitted at crash |

### Common mistakes

* Undoing immediately without first reconstructing history.
* Redoing every record without checking page LSN.
* Forgetting to log undo progress.
* Starting redo at the latest checkpoint without using dirty-page information.
* Treating every transaction active at checkpoint time as a loser.

### Recall questions

1. What does the ARIES analysis phase reconstruct?
2. Why does redo include loser transactions?
3. In what order are loser operations undone?
4. How do compensation records make recovery restartable?
5. Why is ARIES suited to steal/no-force buffering?

---

# Concurrency Control

## 21. Core Approaches

### Core idea

Concurrency control permits overlap while preserving a correct transaction schedule.

Three broad approaches are:

| Approach        | Assumption                         | Conflict response                       |
| --------------- | ---------------------------------- | --------------------------------------- |
| **Optimistic**  | Conflicts are rare                 | Validate near commit; abort on conflict |
| **MVCC**        | Readers can use older versions     | Select visible version; resolve writes  |
| **Pessimistic** | Conflicts must be controlled early | Block, order or abort during execution  |



### Trade-off

```text
More coordination before conflict
    → fewer wasted retries
    → more blocking

Less coordination before conflict
    → greater concurrency
    → more possible abort work
```

### Common mistake

Treating these categories as mutually exclusive.

A database may combine:

* MVCC for reads
* Locks for writes
* Validation at commit
* Latches for physical pages

### Recall questions

1. Which approach delays conflict detection until validation?
2. Why does MVCC reduce read/write interference?
3. Under what workload is pessimistic control preferable?
4. How can one system combine MVCC and locks?

---

## 22. Schedules and Serializability

### Schedule

A schedule is the database-visible ordering of operations from one or more transactions:

* Read
* Write
* Commit
* Abort

A complete schedule includes all operations of all participating transactions. 

## Serial schedule

Transactions run without interleaving:

```text
T1: R A, W A, COMMIT
T2: R B, W B, COMMIT
```

Serial execution is simple but limits throughput.

## Serializable schedule

Operations may interleave, but the final observable result must be equivalent to some serial transaction order.

```text
Concurrent history
    ≡
T2 then T1
```



### Mental model

Serializable does not mean:

> Transactions literally run one at a time.

It means:

> Their combined outcome can be explained as though they did.

### Serializability vs linearizability

| Serializability                               | Linearizability                                                   |
| --------------------------------------------- | ----------------------------------------------------------------- |
| Concerns transaction-operation ordering       | Concerns real-time behaviour of individual operations             |
| Requires equivalence to some serial order     | Must preserve external real-time order                            |
| Serial order may differ from wall-clock order | Later invocation cannot appear before earlier completed operation |



### Common mistakes

* Equating concurrent execution with nonserializable execution.
* Assuming serializability fixes a specific transaction order.
* Confusing serializability with linearizability.
* Checking only final values while ignoring intermediate observations.

### Recall questions

1. How can an interleaved schedule still be serializable?
2. Why is literal serial execution inefficient?
3. What additional ordering requirement does linearizability impose?
4. Why is matching only the final database state sometimes insufficient?

---

## 23. Isolation and Coordination Cost

### Core idea

An isolation level defines which concurrent effects a transaction may observe.

Stronger isolation usually requires more:

* Locking
* Validation
* Version tracking
* Abort handling
* Communication
* Waiting



### Trade-off

```text
Stronger isolation
    → fewer anomalies
    → more coordination
    → potentially lower throughput

Weaker isolation
    → greater concurrency
    → application must tolerate more anomalies
```

### Rule

Isolation should be selected from the application’s invariants, not only from benchmark performance.

### Common mistakes

* Selecting read committed because it is a default without analysing anomalies.
* Assuming database constraints compensate for every weak-isolation anomaly.
* Testing transactions only without concurrency.
* Treating isolation-level names as identical across all database products.

### Recall questions

1. Why does stronger isolation require coordination?
2. Which application invariants determine whether weaker isolation is safe?
3. Why should isolation behaviour be tested on the actual DBMS?

---

# Read and Write Anomalies

## 24. Dirty Read

### Pattern

```text
T1: write X = 20
T2: read X → 20
T1: abort
```

`T2` observed a value that never committed.



### Why it matters

`T2` may:

* Return invalid data
* Perform dependent writes
* Trigger external side effects
* Make decisions using a nonexistent state

### Recall questions

1. Why can a dirty read not be corrected merely by rolling back the writer?
2. What external side effect could make a dirty read especially harmful?

---

## 25. Nonrepeatable Read

### Pattern

```text
T1: read X → 10
T2: write X = 20
T2: commit
T1: read X → 20
```

The same row yields different values inside one transaction. 

### Distinction

The row already existed during both reads; one of its values changed.

### Recall questions

1. How does a nonrepeatable read differ from a dirty read?
2. Which transaction commits in the nonrepeatable-read example?

---

## 26. Phantom Read

### Pattern

```text
T1: SELECT rows WHERE score > 80 → 5 rows
T2: INSERT row with score 90
T2: COMMIT
T1: same query → 6 rows
```

A repeated predicate query returns a different set of rows. 

### Distinction

| Nonrepeatable read      | Phantom read                      |
| ----------------------- | --------------------------------- |
| Existing row changes    | Matching set changes              |
| Row-level phenomenon    | Predicate/range phenomenon        |
| Same identifier differs | New or removed identifier appears |

### Recall questions

1. Why is row locking alone insufficient to prevent every phantom?
2. What logical resource must be protected to prevent a range insertion?

---

## 27. Lost Update

### Pattern

```text
Initial X = 10

T1: read X → 10
T2: read X → 10
T1: write X = 11
T2: write X = 12
```

`T2` overwrites `T1` without incorporating its change. 

### Common prevention

* Exclusive write locks
* Compare-and-swap/version checks
* First-committer-wins
* Serializable execution
* Atomic database expressions such as `X = X + 1`

### Common mistake

Using read-modify-write logic without checking whether the value changed after the read.

### Recall questions

1. Why does an atomic increment prevent this lost-update pattern?
2. How can a version number detect the conflict?

---

## 28. Dirty Write

### Pattern

A transaction overwrites or bases a write on another transaction’s uncommitted value.

```text
T1: write X, not committed
T2: overwrite X
T1: abort
```

Rollback becomes ambiguous because `T2` depends on an unstable state. 

### Rule

Dirty writes are normally prevented even at weak practical isolation levels.

### Recall questions

1. Why is rollback difficult after a dirty write?
2. How does an exclusive write lock prevent it?

---

## 29. Write Skew

### Core idea

Two transactions modify different records, so no direct write/write conflict occurs, but together they violate a shared invariant.

Example:

```text
Invariant:
account_A + account_B ≥ £0

Initial:
A = £100
B = £150

T1 reads both and withdraws £200 from A
T2 reads both and withdraws £200 from B
```

Each sees total funds of £250 and considers its update valid.

Final:

```text
A = -£100
B = -£50
Total = -£150
```

The combined result violates the invariant. 

### Why simple write-conflict detection misses it

```text
T1 writes A
T2 writes B
```

The write sets do not overlap, but their read dependencies do.

### Common prevention

* Serializable isolation
* Predicate or range locks
* Explicitly lock all records participating in the invariant
* Materialise the invariant into one contested record
* Serializable validation of read/write dependencies

### Common mistakes

* Checking only write/write conflicts.
* Assuming snapshot isolation is serializable.
* Validating an invariant only against a transaction’s starting snapshot.
* Splitting one invariant across independent records without coordination.

### Recall questions

1. Why does first-committer-wins not prevent this example?
2. Which read/write dependency creates the conflict?
3. How could the schema be changed to make the conflict explicit?

---

# Isolation Levels

## 30. Standard Isolation Hierarchy

| Isolation level      | Dirty reads | Nonrepeatable reads |        Phantoms |
| -------------------- | ----------: | ------------------: | --------------: |
| **Read uncommitted** |    Possible |            Possible |        Possible |
| **Read committed**   |   Prevented |            Possible |        Possible |
| **Repeatable read**  |   Prevented |           Prevented | May be possible |
| **Serializable**     |   Prevented |           Prevented |       Prevented |



### Important exception

Actual database implementations may provide semantics stronger or different from this simplified SQL-standard table.

For example, a product’s repeatable-read implementation may use snapshots and prevent some phantom observations while still allowing write skew.

## Read uncommitted

Transactions may observe uncommitted values.

## Read committed

Each statement reads committed data, but two statements in one transaction may see different committed states.

## Repeatable read

Repeated reads of an already observed row remain stable.

Predicate-level changes may still appear, depending on implementation.

## Serializable

The outcome must correspond to some serial schedule.

### Common mistakes

* Assuming repeatable read means the entire query result remains unchanged.
* Assuming serializable means transactions do not run concurrently.
* Depending only on level names rather than tested product semantics.
* Treating read committed as safe for multi-statement read-modify-write logic.

### Recall questions

1. Which anomaly distinguishes read committed from repeatable read?
2. Why can repeatable row values coexist with phantoms?
3. What outcome does serializable isolation guarantee?
4. Why may two databases implement “repeatable read” differently?

---

## 31. Snapshot Isolation

### Core idea

A transaction reads from a stable snapshot representing committed state at its start time.

```text
Transaction starts at timestamp S
    ↓
All reads use snapshot S
```

Its snapshot does not change during execution.

At commit, write conflicts are checked. If another transaction has already changed the same written item, one transaction aborts. 

### Guarantees

Typically prevents:

* Dirty reads
* Nonrepeatable reads
* Lost updates between writers of the same item

### Does not necessarily prevent

* Write skew
* Predicate invariant violations
* Every serializability anomaly



### Mental model

```text
Reads:
stable historical photograph

Writes:
accepted only if direct write conflicts permit
```

### Common mistakes

* Equating snapshot isolation with serializability.
* Assuming a stable snapshot automatically preserves multi-record invariants.
* Keeping snapshots indefinitely without accounting for old-version retention.
* Checking only direct write/write overlap.

### Recall questions

1. Why does snapshot isolation prevent nonrepeatable reads?
2. Why can two transactions writing different records both commit?
3. How does first-committer-wins prevent lost updates?
4. How do long-running snapshots affect garbage collection?

---

# Concurrency-Control Mechanisms

## 32. Optimistic Concurrency Control

### Assumption

Conflicts are uncommon, so transactions should execute without blocking and validate before commit.

### Phases

| Phase          | Work                                          |
| -------------- | --------------------------------------------- |
| **Read**       | Read data and prepare writes in private state |
| **Validation** | Check read/write conflicts                    |
| **Write**      | Publish changes if validation succeeds        |



### Read and write sets

* **Read set:** values observed by the transaction.
* **Write set:** values the transaction intends to change.

### Validation questions

* Did anything read by the transaction change?
* Would its write overwrite a newer committed value?
* Do concurrent transaction dependencies violate serializability?

### Conflict result

```text
Validation succeeds → commit
Validation fails    → abort and retry
```

### Backward validation

Compare against transactions that committed during the current transaction’s execution.

### Forward validation

Compare against other transactions currently validating or executing.

Validation and publication require a short critical section to prevent inconsistent interleaving. 

### Best workload

* Low contention
* Short transactions
* Small read/write sets
* Cheap retries
* Mostly independent data

### Poor workload

* Hot records
* Long transactions
* Expensive business logic
* High conflict rates
* External side effects before commit

### Common mistakes

* Performing irreversible external effects before validation.
* Retrying indefinitely under sustained contention.
* Validating only write/write conflicts.
* Publishing part of the write set before validation completes.
* Assuming “lock-free transaction execution” means no internal latches are used.

### Recall questions

1. Why is OCC attractive when conflicts are rare?
2. What work is wasted after a validation failure?
3. Why must validation and write publication be coordinated?
4. What makes a transaction expensive to retry?

---

## 33. MVCC

### Core idea

MVCC keeps multiple timestamped record versions so readers can access an older consistent value while writers create newer versions.

```text
X@10 = old committed version
X@20 = new uncommitted version
```

A reader’s transaction timestamp determines which version is visible. 

### Mental model

```text
Writer creates a new timeline entry
Reader continues observing an earlier timeline point
```

### Visibility questions

For each version:

* Which transaction created it?
* Has that transaction committed?
* When did it commit?
* Was the version deleted?
* Is it visible to this reader’s snapshot?

### Benefits

* Readers often avoid blocking writers.
* Writers need not overwrite values in place.
* Snapshot isolation becomes practical.
* Read consistency is tied to transaction timestamps.

### Costs

* Multiple physical versions
* Visibility metadata
* Vacuum/garbage collection
* Longer reads through version chains
* Storage amplification
* Old-version retention for long snapshots

### Important rule

A version is reclaimable only when no active transaction can still see it.

### Common mistakes

* Deleting an old version immediately after a new commit.
* Assuming MVCC removes write/write conflicts.
* Allowing multiple uncommitted versions without a resolution rule.
* Ignoring version-chain lookup cost.
* Treating timestamp ordering and wall-clock time as identical.

### Recall questions

1. How does MVCC allow a reader and writer to proceed concurrently?
2. Why do long transactions delay vacuum?
3. Which conflicts remain even with MVCC?
4. What metadata determines version visibility?

---

## 34. Timestamp Ordering

### Core idea

Each transaction receives a timestamp that defines its logical order.

Each value tracks information such as:

* Maximum read timestamp
* Maximum write timestamp

Operations that violate the timestamp order are rejected or ignored. 

### Read rule

A transaction trying to read a value overwritten by a logically newer transaction may have to abort.

### Write rule

A write that conflicts with a logically later read may have to abort.

### Thomas Write Rule

An obsolete write older than an already accepted write can sometimes be ignored rather than aborting the transaction:

```text
newer value already exists
    ↓
older write has no useful effect
    ↓
discard older write
```



### Restart rule

An aborted transaction normally receives a new timestamp; restarting with the same timestamp may reproduce the same conflict.

### Common mistakes

* Using physical wall-clock timestamps without handling skew or ties.
* Accepting an old read that violates logical write order.
* Applying an obsolete write that should be ignored.
* Restarting with an unchanged timestamp.
* Assuming timestamp ordering cannot cause starvation.

### Recall questions

1. Why can an outdated write sometimes be ignored safely?
2. Why may a restarted transaction need a new timestamp?
3. Which metadata must each record maintain?
4. How can timestamp ordering cause repeated aborts?

---

## 35. Two-Phase Locking

### Core idea

Two-phase locking divides lock ownership into two phases.

## Growing phase

* Acquire locks
* Do not release locks

## Shrinking phase

* Release locks
* Do not acquire new locks

```text
Acquire → acquire → acquire
                  peak
Release → release → release
```



### Core rule

Once a transaction releases its first lock, it may not acquire another.

### Why it works

The lock point—the moment at which the transaction owns its maximum lock set—helps establish a serial ordering.

### Variants

| Variant              | Additional rule                                    |
| -------------------- | -------------------------------------------------- |
| **Basic 2PL**        | Growing then shrinking                             |
| **Strict 2PL**       | Hold exclusive locks until commit/abort            |
| **Rigorous 2PL**     | Hold shared and exclusive locks until commit/abort |
| **Conservative 2PL** | Acquire all required locks before beginning        |

### Critical distinction

**Two-phase locking is not two-phase commit.**

| 2PL                               | 2PC                                    |
| --------------------------------- | -------------------------------------- |
| Concurrency-control protocol      | Distributed commit protocol            |
| Controls lock acquisition/release | Coordinates commit across participants |
| Often implements serializability  | Implements distributed atomicity       |

### Common mistakes

* Acquiring a new lock after releasing one.
* Confusing 2PL with 2PC.
* Assuming basic 2PL prevents cascading aborts.
* Holding locks only for statement duration when transaction duration is required.
* Acquiring locks in inconsistent order without deadlock handling.

### Recall questions

1. What event ends the growing phase?
2. Why does strict 2PL simplify recovery from cascading aborts?
3. How does conservative 2PL reduce deadlock risk?
4. Why are 2PL and 2PC unrelated despite similar names?

---

# Deadlocks

## 36. Deadlock Model

### Core idea

A deadlock occurs when transactions wait in a cycle.

```text
T1 holds L1, waits for L2
T2 holds L2, waits for L1
```

Neither can proceed. 

### Necessary pattern

```text
Transaction A waits for B
Transaction B waits for C
...
Transaction N waits for A
```

### Waits-for graph

* Vertex → transaction
* Edge `T1 → T2` → `T1` waits for a resource held by `T2`
* Cycle → deadlock



### Resolution

Choose a victim transaction:

1. Abort it.
2. Release its locks.
3. Wake blocked transactions.
4. Optionally retry the victim.

Possible victim criteria:

* Youngest transaction
* Least work completed
* Fewest locks held
* Lowest rollback cost
* Lowest priority

### Timeout limitation

A timeout detects long waiting, not deadlock itself.

It can:

* Abort transactions that were merely slow.
* Take too long to resolve a real deadlock.
* Require workload-specific tuning.

### Common mistakes

* Assuming lock waiting always indicates deadlock.
* Detecting a cycle but aborting no participant.
* Retrying immediately with the same acquisition pattern.
* Selecting victims without considering repeated starvation.
* Leaving victim locks held during rollback.

### Recall questions

1. What graph structure proves a deadlock?
2. Why can a timeout produce false positives?
3. Which transaction makes a good victim?
4. How can repeated victim selection cause starvation?

---

## 37. Deadlock Prevention: Wait-Die and Wound-Wait

Transactions receive priorities, commonly based on age:

```text
Older transaction → higher priority
```

## Wait-die

When requester `T1` conflicts with holder `T2`:

* `T1` older → `T1` waits.
* `T1` younger → `T1` aborts or “dies.”

## Wound-wait

* `T1` older → abort younger holder `T2`.
* `T1` younger → `T1` waits.



### Comparison

| Wait-die                           | Wound-wait                       |
| ---------------------------------- | -------------------------------- |
| Older waits for younger            | Older pre-empts younger          |
| Younger requester aborts           | Younger requester waits          |
| Nonpreemptive                      | Preemptive                       |
| May leave old transactions waiting | Favours old transaction progress |

### Why timestamps prevent cycles

Waiting direction is constrained consistently by transaction age, preventing arbitrary cyclic dependencies.

### Common mistakes

* Reversing older/younger rules.
* Assigning a fresh priority on every retry and causing starvation.
* Confusing deadlock prevention with deadlock detection.
* Applying transaction deadlock policies directly to low-level latches.

### Recall questions

1. What happens when a younger transaction requests a lock from an older holder under wait-die?
2. Who is aborted when an older requester conflicts with a younger holder under wound-wait?
3. Why should a retried transaction often retain its original priority?
4. How do these policies eliminate cycles before they form?

---

# Locks and Latches

## 38. Logical Locks

### Core idea

Transaction locks protect logical database contents.

They may cover:

* Existing key
* Nonexistent key
* Key range
* Table
* Row
* Predicate

Locks are normally managed outside the B-Tree by the lock manager and are held for transaction-level correctness. 

### Examples

```text
Lock key = 42
Lock range [100, 200)
Lock all rows matching status = 'pending'
```

### Purpose

* Prevent dirty writes
* Control visibility
* Preserve serializable schedules
* Protect absent keys from phantom insertions
* Coordinate transaction-level invariants

---

## 39. Physical Latches

### Core idea

Latches protect in-memory physical structures while threads inspect or modify them.

Examples:

* Page contents
* Slot directory
* Parent pointer
* Sibling pointer
* B-Tree split
* Buffer-cache metadata



### Locks vs latches

| Logical lock                   | Physical latch                               |
| ------------------------------ | -------------------------------------------- |
| Protects transaction data      | Protects internal structure                  |
| Key/range/table scope          | Page/object scope                            |
| Held relatively long           | Held briefly                                 |
| Managed by lock manager        | Managed by storage implementation            |
| Deadlock manager may intervene | Programmer must design safe ordering         |
| Needed for isolation           | Needed even in lock-free transaction schemes |

### Rule

Hold a latch for the smallest safe duration.

### Important distinction

A transaction may own a logical row lock without continuously holding the page latch containing that row.

### Common mistakes

* Holding a page latch for an entire transaction.
* Assuming MVCC removes the need for latches.
* Using a short latch where transaction-duration protection is required.
* Passing a raw page pointer to code after releasing its latch.
* Relying on the transaction deadlock detector to resolve latch deadlocks.

### Recall questions

1. Why does a lock-free transaction protocol still need page latches?
2. Which mechanism protects a B-Tree during a split?
3. Which mechanism protects a row from another transaction?
4. Why should latches be held briefly?

---

## 40. Readers-Writer Latches

### Core idea

A readers-writer latch permits:

* Multiple simultaneous readers
* One exclusive writer
* No reader/writer overlap

| Requested/current | Shared reader | Exclusive writer |
| ----------------- | ------------: | ---------------: |
| Shared reader     |    Compatible |     Incompatible |
| Exclusive writer  |  Incompatible |     Incompatible |

 

### Why it improves concurrency

Readers do not modify physical state, so they can inspect the same stable page concurrently.

### Potential problems

* Writer starvation when readers continuously arrive
* Reader starvation under writer preference
* Upgrade deadlocks
* Queue-management overhead

### Common mistakes

* Modifying page metadata while holding only a shared latch.
* Allowing a writer to overlap with a reader.
* Upgrading several shared holders simultaneously without a policy.
* Ignoring starvation in the admission policy.

### Recall questions

1. Why can several readers safely share a page latch?
2. What conflict arises when two readers both attempt upgrade?
3. How can reader preference starve a writer?
4. Which page operations require exclusive ownership?

---

## 41. Blocking vs Busy-Waiting

### Blocking

The waiting thread is descheduled and later awakened.

Best when:

* Wait may be long
* CPU is scarce
* Lock holder may be paused

### Busy-waiting

The thread repeatedly checks until the latch becomes available.

Best when:

* Critical sections are very short
* Context switching costs more than expected wait
* CPU resources are available



### Queue locks

A queued latch can give each waiter a predecessor:

```text
Thread A → Thread B → Thread C
```

Each thread waits primarily on state changed by its immediate predecessor, reducing shared-cache-line contention.

### Trade-off

| Spinning                        | Blocking                |
| ------------------------------- | ----------------------- |
| Low handoff latency             | Low wasted CPU          |
| Wastes cycles during long waits | Scheduler overhead      |
| Good for short latches          | Good for long locks     |
| Sensitive to pre-emption        | Better under contention |

### Common mistakes

* Spinning while the holder is blocked or descheduled.
* Blocking for a critical section lasting only a few instructions.
* Having every waiter spin on one shared variable.
* Using unfair queues that starve some threads.

### Recall questions

1. When is spinning cheaper than blocking?
2. Why is spinning harmful when the owner is descheduled?
3. How does queueing reduce cache-line contention?
4. Why do transaction locks usually favour blocking over spinning?

---

# Concurrent B-Tree Traversal

## 42. Latch Crabbing

### Core idea

Latch crabbing acquires the child latch before releasing the parent latch.

Read traversal:

```text
Latch parent
    ↓
Find child
    ↓
Latch child
    ↓
Release parent
```

This ensures the child reference does not disappear between reading it and protecting the child. 

### Insert rule

The parent latch can be released when the child is **safe** from split.

For insertion, safe usually means:

```text
child has enough free space
```

### Delete rule

The parent can be released when the child is safe from merge or redistribution.

```text
child remains above minimum occupancy after delete
```



### Why it is optimistic

Most operations:

* Affect one leaf
* Do not split or merge
* Do not propagate upward

Therefore most ancestor latches can be released early.

### Page-miss complication

If the child is not cached, holding the parent latch during slow I/O increases contention.

Options:

1. Retain protection while loading.
2. Release the parent, load the page and restart traversal.
3. Use page versions to detect structural changes.

### Common mistakes

* Releasing the parent before securing the child.
* Declaring a full child safe for insertion.
* Holding the root latch throughout every traversal.
* Waiting on disk I/O while unnecessarily blocking the root.
* Reusing a stale path after releasing latches without validation.

### Recall questions

1. Why must the child be latched before the parent is released?
2. What makes a node safe during insertion?
3. Why does restart-based traversal improve concurrency?
4. Why do structural changes become less probable at higher levels?

---

## 43. Latch Upgrading

### Core idea

Traversal can begin with shared latches and upgrade only pages requiring modification.

```text
Shared traversal
    ↓
Exclusive leaf latch
    ↓
Split needed?
 ├─ No → modify leaf
 └─ Yes → upgrade affected ancestors
```



### Benefit

Most traversals do not require exclusive ownership of upper pages.

### Risks

* Two readers may both attempt upgrade.
* Upgrade may block while the transaction holds lower latches.
* Waiting order can deadlock.
* Failed upgrade may require restart.

### Root bottleneck

Every traversal reaches the root, but root splits are rare.

An implementation can read or latch the root optimistically, then restart if a structural version changed.

### Common mistakes

* Waiting indefinitely for upgrade while holding conflicting latches.
* Assuming upgrade always succeeds atomically.
* Taking an exclusive root latch for every write.
* Failing to restart after an invalidated path.

### Recall questions

1. Why does shared traversal improve concurrency?
2. What conflict occurs when two threads upgrade the same page?
3. Why is optimistic root access usually safe?
4. When must traversal restart?

---

## 44. B-link Trees

### Core idea

A B-link Tree adds:

* A high key
* A right-sibling link

These allow searches to remain correct during concurrent splits. 

### Half-split state

During a split:

1. Create the new right sibling.
2. Link the old page to it.
3. Adjust high keys.
4. Later add the new child pointer to the parent.

For a short period, the new page is reachable through its sibling but not yet directly from the parent.

```text
Parent
  ↓
Old page → New page
```

### Search correction

A reader may descend through an outdated parent into the old page.

It checks:

```text
search_key ≥ old_page.high_key
```

If true, it follows the sibling link.

### Mental model

The parent route may be stale, but the page itself contains directions to the correct neighbour.

### Benefits

* Parent latch need not be held throughout split.
* Readers can continue during structural modification.
* Structural changes use a bounded number of latches.
* Less root-to-leaf restarting
* Reduced deadlock risk



### Cost

Occasionally, a search performs one extra sibling-page access.

Since splits are comparatively rare, this is generally cheaper than locking ancestors aggressively.

### Required invariants

* High keys correctly bound page ranges.
* Sibling links connect to the next range.
* New page becomes sibling-reachable before parent publication.
* Parent is eventually updated.
* Searches follow right when a high-key violation occurs.

### Common mistakes

* Publishing the parent pointer before the new page is fully initialised.
* Failing to install the sibling link before releasing latches.
* Treating the half-split state as corruption.
* Following left when the search exceeds the high key.
* Never completing the delayed parent update.

### Recall questions

1. How can a new page be reachable before its parent points to it?
2. What tells a reader that it descended through stale routing information?
3. Why can the parent latch be released earlier?
4. Which ordering makes a split safely visible?
5. What is the performance cost of correcting through a sibling link?

---

## 45. Chapter 5 Design Summary

### Core mental model

Transaction processing coordinates three forms of state:

```text
Logical transaction state
    ↕
Cached page state
    ↕
Durable log and page state
```

Concurrency control governs **who may observe or modify data**.

Latches govern **who may inspect or alter physical structures**.

Recovery governs **how volatile and persistent state are reconciled after failure**.

### Key relationships

| Mechanism             | Problem solved                 | Main cost                        |
| --------------------- | ------------------------------ | -------------------------------- |
| Page cache            | Repeated disk access           | Memory and eviction complexity   |
| Pinning               | Retain high-value pages        | Reduces available cache          |
| Replacement policy    | Select eviction victims        | Tracking overhead                |
| WAL                   | Recover unflushed changes      | Log I/O                          |
| Checkpoint            | Bound recovery work            | Background flush cost            |
| Steal                 | Reduce cache pressure          | Requires undo                    |
| No-force              | Fast commit and write batching | Requires redo                    |
| ARIES                 | Recovery for steal/no-force    | Complex logging                  |
| Serializable schedule | Correct concurrent outcome     | Coordination                     |
| Snapshot isolation    | Stable reads                   | Write skew possible              |
| OCC                   | Avoid blocking                 | Retry cost                       |
| MVCC                  | Readers avoid writers          | Version storage and vacuum       |
| 2PL                   | Early conflict control         | Blocking and deadlocks           |
| Logical locks         | Transaction isolation          | Long ownership                   |
| Physical latches      | Structural integrity           | Short-term contention            |
| Latch crabbing        | Reduce ancestor latch duration | Safety checks and retries        |
| B-link Tree           | Concurrent structural changes  | High-key and sibling maintenance |

### Rules to retain

* Commit durability depends on the WAL, not necessarily immediate data-page flush.
* WAL must reach disk before its corresponding dirty page.
* Dirty pages require safe flushing before eviction.
* Steal implies undo; no-force implies redo.
* ARIES performs analysis, redo, then undo.
* Serializability means equivalence to some serial order.
* Snapshot isolation is not automatically serializable.
* Write skew can occur without overlapping write sets.
* OCC works best when conflicts and retries are rare.
* MVCC trades blocking for version-management work.
* Two-phase locking is unrelated to two-phase commit.
* Locks protect logical data; latches protect physical structures.
* A B-Tree operation should hold ancestor latches only while propagation remains possible.
* B-link sibling pointers and high keys make temporary half-splits searchable.

### Applied recall questions

1. A committed transaction’s page is still dirty when the server crashes. Which policy permits this, and how is the change recovered?
2. An uncommitted transaction’s page reached disk before a crash. Which policy permitted it, and what recovery action is needed?
3. A cache has a high hit ratio but poor write latency. Which dirty-page and eviction interactions should be investigated?
4. A full-table scan evicts frequently used index roots. Which replacement or admission mechanism could reduce this?
5. Two snapshot-isolated transactions update different records and violate a shared invariant. Name the anomaly and explain why direct write-conflict checks miss it.
6. A transaction releases one row lock and then requests another under 2PL. Which rule has it violated?
7. During recovery, why does ARIES redo an incomplete transaction before undoing it?
8. A reader enters a B-link page and finds its target above the page high key. What happened, and how should it proceed?
9. A long-running MVCC transaction causes database storage to grow. Explain the version-retention relationship.
10. Why can holding an exclusive root latch during every insertion make a multi-core system scale poorly?

# Chapter 6: B-Tree Variants

## 1. Why B-Tree Variants Exist

### Core idea

Standard B-Trees provide efficient lookup, but their implementation creates several costs:

* Entire-page rewrites for small updates
* Reserved empty space for future inserts
* Random I/O during updates
* Complex latch coordination
* Splits and merges that modify multiple pages

B-Tree variants preserve the basic tree model while changing how updates, pages and pointers are represented. 

### Main design directions

| Variant                | Main technique                       | Primary goal                            |
| ---------------------- | ------------------------------------ | --------------------------------------- |
| Copy-on-write B-Tree   | Copy modified paths                  | Simple snapshots and reader concurrency |
| Lazy B-Tree            | Buffer per-node or subtree updates   | Batch page modifications                |
| FD-Tree                | Mutable head + immutable sorted runs | Reduce random writes                    |
| Bw-Tree                | Base nodes + append-only deltas      | Reduce rewrites and latching            |
| Cache-oblivious B-Tree | Recursive memory layout              | Work across memory hierarchies          |

### Mental model

```text
Traditional B-Tree:
find page → modify page → rewrite page

Variants:
delay, redirect, append or reorganise the modification
```

### Key trade-off

```text
Less work during writes
    → more indirection, reconciliation or maintenance during reads
```

### Recall questions

1. Which standard B-Tree costs arise from in-place updates?
2. How can immutability simplify concurrency?
3. Why do many variants shift work from writes to reads or background maintenance?

---

# Copy-on-Write B-Trees

## 2. Copy-on-Write

### Core idea

A page is never modified directly.

To change a page:

1. Copy it.
2. Modify the copy.
3. Copy affected ancestors.
4. Construct a new root-to-leaf path.
5. Atomically publish the new root.

Untouched subtrees are shared by the old and new versions. 

### Mental model

```text
Old root ──▶ old path ──▶ old leaf
    │
    └────── shared unchanged subtrees

New root ──▶ copied path ──▶ new leaf
```

The diagram on PDF page 132 shows the old and new trees existing together while unchanged pages are reused.

### Publication rule

```text
Build complete new path
    ↓
Persist required pages
    ↓
Atomically switch root pointer
```

Readers see either:

* The complete old tree
* The complete new tree

They should not see a partially constructed version.

### Benefits

* Readers need little or no page latching.
* Readers do not block writers.
* Old versions naturally support snapshots.
* Incomplete page modifications are not published.
* Crash recovery can use the last valid root.
* Original pages remain available until old readers finish.

### Costs

* Copies every modified page on the root-to-leaf path.
* Temporarily stores multiple page versions.
* Requires garbage collection of unreachable pages.
* Writers modifying overlapping paths may still require serialisation.
* Root publication must be atomic and durable.

### Cause and effect

```text
Immutability
    → readers can safely retain old references
    → no partially modified pages
    → simpler read concurrency

But:
    → old and new page versions coexist
    → greater space and copy cost
```

### Common mistakes

* Modifying a shared old page after publishing it as immutable.
* Switching the root before all new pages are durable.
* Reclaiming old pages while a reader still references them.
* Assuming copy-on-write removes write/write conflicts.
* Forgetting that page copying causes write amplification of its own.

### Quick example

Updating one leaf in a height-four tree may require writing:

* New leaf
* New parent
* New grandparent
* New root

Every unaffected subtree can remain shared.

### Recall questions

1. Why can readers access old pages without latches?
2. Which pages must be copied for a leaf update?
3. What ordering is required before the root pointer changes?
4. When can an old page safely be reclaimed?
5. How does copy-on-write trade latch complexity for write amplification?

---

## 3. LMDB Copy-on-Write Design

### Core idea

LMDB combines:

* Memory-mapped files
* Copy-on-write B-Trees
* A single writer
* Lock-free readers
* Naturally multiversioned pages

It can read pages directly through the memory map without an application-level page cache or page materialisation layer. 

### Update path

For an update:

```text
root
  ↓ copy
affected internal node
  ↓ copy
target leaf
  ↓ modify
new root published
```

Only nodes on the affected path are copied.

### Root versions

LMDB maintains conceptually:

* The latest published root
* The root being constructed for the next commit

Once the new root is published:

* New readers use it.
* Existing readers may continue using the old root.
* Old pages are reclaimed after those readers finish.

### Why no WAL is needed in this design

The old tree remains valid until the new tree is fully published.

```text
Crash before root switch
    → old tree remains authoritative

Crash after durable root switch
    → new tree is authoritative
```

This replaces much of the role normally played by in-place-update recovery logging.

### Important exception

This does not mean every copy-on-write implementation automatically requires no recovery machinery.

The complete persistence protocol must still handle:

* Atomic metadata publication
* Storage ordering
* Torn writes
* Free-page tracking
* Reader lifecycle

### Sibling-pointer trade-off

LMDB does not use sibling pointers in this design, so sequential traversal may need to ascend to parents to find the next leaf.

### Common mistakes

* Assuming memory mapping makes persistence automatic.
* Allowing multiple unrestricted writers without coordinating root construction.
* Reclaiming pages based only on the latest root.
* Assuming no WAL means no crash-consistency protocol.
* Expecting sibling-based scans when sibling pointers are absent.

### Recall questions

1. Why can LMDB serve reads directly from mapped pages?
2. How does its old root support MVCC-like snapshots?
3. Why can it avoid an ordinary WAL?
4. What traversal cost arises from omitting sibling links?
5. Why is a single-writer model useful for this architecture?

---

# In-Memory Node Representations

## 4. Representing a Cached Node

### Core idea

An on-disk page and its in-memory representation do not have to be identical.

Three broad approaches are possible:

| Approach            | Description                                      |
| ------------------- | ------------------------------------------------ |
| Raw page access     | Manipulate the binary page directly              |
| Native object       | Decode the page into language-level structures   |
| Wrapper over buffer | Expose structured operations backed by raw bytes |

 

## Raw binary access

### Benefits

* No duplicate representation
* Low materialisation cost
* Direct correspondence with persisted bytes

### Costs

* Complex offset arithmetic
* Greater risk of memory corruption
* Harder concurrency
* Encoding details leak into algorithms

## Native object materialisation

```text
On-disk bytes
    ↓ decode
Native node object
    ↓ modify
Re-encode on flush
```

### Benefits

* Simpler algorithms
* Easier concurrency around logical objects
* Convenient variable-sized structures

### Costs

* Memory duplication
* Encode/decode overhead
* Must reconcile object and raw page lifecycles

## Wrapper over backing buffer

A wrapper provides fields and operations while immediately applying changes to the underlying byte buffer.

### Benefits

* Avoids full duplicate materialisation
* Hides offset calculations
* Useful in managed languages

### Costs

* Changes remain coupled to physical layout
* Wrapper correctness is critical
* May still need synchronisation around the buffer

### Key relationship

```text
More abstraction
    → easier update logic
    → greater memory or translation overhead

Less abstraction
    → compact and direct
    → more complex, layout-dependent code
```

### Common mistakes

* Keeping two representations without defining which is authoritative.
* Flushing stale raw bytes after modifying a native object.
* Exposing mutable buffers without synchronisation.
* Letting object lifetimes outlive their page-cache references.
* Assuming an in-memory object layout is suitable for persistence.

### Recall questions

1. What memory cost arises from materialising native node objects?
2. Why can wrappers be useful in managed languages?
3. Which representation simplifies direct persistence?
4. How should conflicts between raw and materialised versions be resolved?

---

# Lazy B-Trees

## 5. Lazy Update Mental Model

### Core idea

A lazy B-Tree does not immediately apply every update to the persistent page image.

Instead, it stores changes in a lighter in-memory structure and reconciles them later.

```text
Base page on disk
    +
In-memory update buffer
    =
Current logical page
```



### Why it matters

Several updates to one page can be combined:

```text
insert A
update B
delete C
    ↓
one reconciliation
    ↓
one page rewrite
```

### Trade-off

| Immediate update           | Lazy buffered update         |
| -------------------------- | ---------------------------- |
| Reads see one page image   | Reads merge base and updates |
| More page rewrites         | Fewer batched rewrites       |
| Simpler lookup             | More reconciliation          |
| Foreground structural work | More background work         |

### Common mistakes

* Reading only the base page and ignoring buffered updates.
* Flushing updates in an order that violates transaction visibility.
* Allowing buffers to grow without bounds.
* Losing buffered changes during a crash.
* Assuming delayed physical updates imply weak durability.

### Recall questions

1. Why can buffering several writes reduce write amplification?
2. What extra work does a read perform?
3. Which mechanism must protect buffered changes before reconciliation?
4. What determines when a buffer should be flushed?

---

## 6. WiredTiger Page Reconciliation

### Core idea

WiredTiger uses different representations for in-memory and on-disk B-Tree pages.

A cached page contains:

* An index built from the disk page
* An update buffer containing newer modifications

Reads combine both to produce the current result. 

### Read path

```text
Read base page
    +
Search update buffer
    +
Apply visibility rules
    =
Latest visible value
```

### Flush path: reconciliation

```text
Base page
    +
Buffered inserts/updates/deletes
    ↓
Reconcile
    ↓
Write new page image
```

If the reconciled result exceeds page capacity, the background process may split it into several pages.

### Update-buffer structure

WiredTiger uses skiplists for update buffers because they provide:

* Ordered lookup
* Search-tree-like complexity
* Useful concurrent-access properties

### Benefits

* Foreground writes avoid immediate page rewrites.
* Structural operations can be deferred.
* Background reconciliation batches work.
* Several updates to the same record may collapse into one persisted result.

### Costs

* Reads consult both base and updates.
* More memory per dirty page.
* Reconciliation can be CPU-intensive.
* Large reconciliation batches may create latency or I/O bursts.
* Crash recovery must preserve unflushed updates.

### Common mistakes

* Treating the on-disk image as the latest page state.
* Evicting a dirty page without reconciliation or recovery support.
* Ignoring transaction visibility when merging updates.
* Allowing background reconciliation to fall permanently behind.
* Assuming all dirty pages have identical update-buffer costs.

### Quick example

Disk page:

```text
A=1, B=2
```

Update buffer:

```text
B=3, delete A, C=4
```

Logical page:

```text
B=3, C=4
```

### Recall questions

1. Why must reads inspect the update buffer?
2. What is reconciliation?
3. When can reconciliation cause a split?
4. Why is a skiplist suitable for buffered updates?
5. How does this design move structural work off the foreground path?

---

## 7. Lazy-Adaptive Trees

### Core idea

An LA-Tree attaches buffers to groups of nodes or subtrees instead of only individual pages.

Updates enter at a high-level buffer and cascade downward in batches. 

### Write path

```text
Insert update into root buffer
    ↓ buffer fills
Partition updates by child subtree
    ↓
Push batches to lower buffers
    ↓
Eventually apply batches at leaves
```

### Mental model

The tree routes **packages of updates**, not only individual operations.

### Why batching helps

Suppose 100 updates target several leaves under one subtree.

Instead of:

```text
100 independent root-to-leaf writes
```

the system can:

```text
group by subtree
    ↓
propagate grouped updates
    ↓
modify each destination page once
```

### Benefits

* Fewer random I/O operations
* Fewer individual page rewrites
* Batched splits and merges
* Better SSD write behaviour
* Amortised navigation cost

### Costs

* Reads must search several buffer levels.
* A value may be represented by pending operations at different levels.
* Buffer cascading adds scheduling complexity.
* Large buffers consume RAM.
* Updates reach leaves with delay.



### Key trade-off

```text
Larger buffers
    → larger, more efficient batches
    → more read reconciliation and memory use
    → longer update propagation delay
```

### Common mistakes

* Applying lower-level updates without respecting newer upper-level operations.
* Cascading all updates to every child instead of partitioning by key range.
* Ignoring deletes or updates during read reconciliation.
* Letting one hot subtree monopolise buffer flushing.
* Assuming buffered operations are already reflected in leaf pages.

### Recall questions

1. How does an LA-Tree differ from a per-page update buffer?
2. Why do subtree buffers reduce random writes?
3. Which structures must a read inspect?
4. How does buffer size affect read and write performance?
5. Why can structural changes also be batched?

---

# FD-Trees

## 8. FD-Tree Structure

### Core idea

An FD-Tree consists of:

* A small mutable B-Tree called the **head tree**
* Several larger immutable sorted runs

Random writes are limited mainly to the head tree.  

### Write path

```text
Write to mutable head tree
    ↓ head becomes full
Flush its sorted contents to run L1
    ↓ L1 exceeds threshold
Merge L1 into L2
    ↓
Continue cascading downward
```

### Mental model

```text
Small mutable intake area
        ↓
Increasingly large immutable storage levels
```

### Why it reduces random writes

The head absorbs small updates.

Lower levels are rewritten through large sequential merges rather than individual in-place page changes.

### Benefits

* Small random-write surface
* Sequential lower-level writes
* Dense immutable runs
* No reserved per-page update space in runs
* Batches updates across many keys

### Costs

* Data for one key may exist in several levels.
* Reads may search multiple runs.
* Merges rewrite existing data.
* Deletes require tombstones.
* Background merging consumes I/O.

### Comparison with lazy B-Trees

| Lazy B-Tree                   | FD-Tree                            |
| ----------------------------- | ---------------------------------- |
| Buffers near target nodes     | Buffers all updates in a head tree |
| Retains mutable main tree     | Uses immutable lower runs          |
| Reconciles page with buffer   | Merges sorted levels               |
| Primarily node-local batching | Cross-node sequential batching     |

### Common mistakes

* Treating lower runs as mutable pages.
* Looking only in the largest run during lookup.
* Removing an older value without considering newer levels.
* Allowing the head tree to grow until it loses its cache advantage.
* Running merges without rate control.

### Recall questions

1. Why is the mutable head tree kept small?
2. What causes data to propagate to a lower level?
3. How does an FD-Tree reduce random writes?
4. What extra work does a lookup require?
5. How does the design resemble an LSM Tree?

---

## 9. Fractional Cascading

### Core idea

Fractional cascading reduces the cost of searching the same key across several sorted arrays.

Search the first level normally, then use bridge pointers to start near the correct location in later levels. 

### Without bridges

```text
binary search L1
binary search L2
binary search L3
```

Approximate cost:

```text
O(log n1 + log n2 + log n3)
```

### With bridges

```text
binary search L1
    ↓ follow bridge near target in L2
small local search
    ↓ follow bridge near target in L3
small local search
```

### Building bridges

Some lower-level elements are copied or referenced in the higher level.

These promoted entries point to corresponding positions below.

```text
Higher level: ... 25 ───────┐
                             ↓
Lower level:  ... 22, 25, 28 ...
```

### Why not bridge every element?

Mapping every item would cause:

* High pointer overhead
* More maintenance
* Larger upper levels

Instead, every `N`th lower-level element may be represented above, limiting the maximum gap between bridge points.

### Mental model

The first level provides a coarse search result that becomes a hint for every later level.

### Trade-off

| More bridges                 | Fewer bridges           |
| ---------------------------- | ----------------------- |
| Smaller later search windows | Lower metadata overhead |
| Faster lookup                | Larger gaps             |
| More copied fence entries    | Simpler merges          |
| More pointer maintenance     | More local searching    |

### Common mistakes

* Treating bridge entries as independent user records.
* Failing to rebuild bridges after run merging.
* Assuming a bridge points to an exact target.
* Creating gaps so large that later searches approach full binary search.
* Returning a promoted fence entry as the latest value without checking its role.

### Recall questions

1. Which search remains a full binary search?
2. What does a bridge pointer represent?
3. Why are only some lower-level entries promoted?
4. How does bridge density trade storage for lookup speed?
5. Why must bridges be updated after a merge?

---

## 10. Logarithmically Sized Runs

### Core idea

FD-Tree runs increase in capacity by a multiplicative factor.

```text
Head < L1 < L2 < L3
```

When a level exceeds its threshold, it is merged into the next level. 

### Example

Assume size ratio `k = 4`:

```text
Head: 1 unit
L1:   4 units
L2:  16 units
L3:  64 units
```

### Merge process

```text
New head flush + existing L1
    ↓ sorted merge
new L1

If new L1 too large:
new L1 + existing L2
    ↓
new L2
```

### Why geometric growth matters

Larger lower levels fill less often.

This limits the number of runs while allowing substantial data capacity.

### Trade-off: size ratio

| Smaller ratio                            | Larger ratio                     |
| ---------------------------------------- | -------------------------------- |
| More frequent merges                     | Less frequent lower-level merges |
| More levels possible                     | Fewer levels                     |
| Lower temporary merge size               | More data rewritten per merge    |
| Potentially faster lookup per level size | Larger compaction bursts         |

### Common mistakes

* Appending unsorted head contents directly to a sorted run.
* Keeping obsolete run versions reachable after replacement.
* Ignoring temporary disk space needed during merging.
* Choosing thresholds without considering write bandwidth.
* Assuming geometric levels eliminate write amplification.

### Recall questions

1. Why do run sizes grow geometrically?
2. What happens when an existing destination run is present?
3. Which temporary storage is required during a merge?
4. How does the size ratio affect merge frequency and burst size?

---

## 11. Tombstones in FD-Trees

### Core idea

Because lower runs are immutable, deletion cannot remove older records immediately.

Instead, a newer tombstone shadows older versions.

```text
Upper level: key K → tombstone
Lower level: key K → old value
```

Lookup result:

```text
K is deleted
```



### Tombstone lifecycle

```text
Insert tombstone in upper level
    ↓
Merge downward
    ↓
Discard shadowed older records
    ↓
Tombstone reaches lowest relevant level
    ↓
Discard tombstone
```

### Rule

A tombstone can be removed only when no older version it could shadow remains in any lower level.

### Common mistakes

* Returning a lower-level value after finding an upper-level tombstone.
* Dropping a tombstone before all older versions are eliminated.
* Treating a tombstone as absence during merging without deleting shadowed entries.
* Ignoring tombstones during range scans.
* Allowing deletion markers to accumulate without compaction.

### Recall questions

1. Why can an immutable run not delete a value directly?
2. When is a tombstone safe to discard?
3. Which version wins when a value and tombstone exist at different levels?
4. How do tombstones affect read and space amplification?

---

# Bw-Trees

## 12. Problems Addressed by Bw-Trees

### Core idea

Bw-Trees target three standard B-Tree costs:

| Problem                | Standard B-Tree cause                      |
| ---------------------- | ------------------------------------------ |
| Write amplification    | Rewriting an entire page for small changes |
| Space amplification    | Reserving free page space                  |
| Concurrency complexity | Latches and multistep structural updates   |

The Bw-Tree uses append-only delta records, logical node IDs and atomic mapping-table updates. 

### Mental model

```text
Logical B-Tree node
    =
base node
    +
chain of later modifications
```

The node is a logical object rather than one fixed physical page.

---

## 13. Base Nodes and Delta Chains

### Core idea

A Bw-Tree stores:

* A stable base node
* A linked chain of delta nodes representing later changes

```text
Newest delta
    ↓
Older delta
    ↓
Older delta
    ↓
Base node
```

Delta nodes may represent:

* Insert
* Update
* Delete
* Split
* Merge
* Other structural operations



### Write path

A small update appends one small delta instead of rewriting the full base node.

```text
Base: [A, B, C]
Delta: update B
Delta: insert D
```

Logical result:

```text
apply deltas newest-to-oldest over base
```

### Benefits

* Small append-only writes
* No reserved empty space inside base nodes
* Physical records need not be page-sized
* Updates to different logical nodes can be batched in one log
* Existing nodes remain immutable

### Read cost

The reader must traverse and reconcile the chain.

```text
Longer chain
    → more pointer traversal
    → more key comparisons
    → greater reconstruction cost
```

### Insert vs update

An update can be represented as a newer insertion for the same key; the latest applicable delta wins.

### Common mistakes

* Applying deltas in the wrong temporal order.
* Returning a base value shadowed by a delete delta.
* Assuming the base node alone is current.
* Allowing unbounded delta-chain growth.
* Reserving fixed page space even though nodes are logically assembled.

### Quick example

```text
Base:       X=1, Y=2
Delta 1:    X=3
Delta 2:    delete Y
Delta 3:    Z=4
```

Current logical state:

```text
X=3, Z=4
```

### Recall questions

1. How does a delta avoid a full page rewrite?
2. Why is an update similar to an insert?
3. What happens to read cost as the chain grows?
4. Which delta type shadows an older value?
5. Why does the logical node not need contiguous storage?

---

## 14. Logical Node IDs and Mapping Table

### Core idea

Parent nodes do not point directly to the latest physical delta location.

They point to a stable logical node ID.

An in-memory mapping table resolves:

```text
logical node ID → newest delta or base location
```



### Why indirection is needed

Without the mapping table, every new delta would require updating the parent’s physical pointer.

With it:

```text
Parent pointer remains logical ID 42

Mapping table:
42 → new delta location
```

### Update algorithm

1. Traverse to the target logical node.
2. Read its current mapping-table pointer.
3. Create a delta pointing to that current head.
4. Atomically replace the mapping entry with the new delta pointer.



### Compare-and-swap

```text
CAS(mapping[id], expected_old, new_delta)
```

* Success → delta installed.
* Failure → another thread changed the entry; retry with the new head.

### Reader ordering

A concurrent reader observes either:

* Old mapping pointer → state before update
* New mapping pointer → state after update

It should not observe a partially installed pointer.

### Benefits

* No page latch for ordinary delta insertion
* Stable parent-child links
* Atomic publication
* Readers and writers can proceed concurrently

### Costs

* Mapping table consumes memory.
* Mapping recovery is required after restart.
* Failed CAS operations cause retries.
* Hot logical nodes create CAS contention.
* Physical location requires an extra lookup.

### Common mistakes

* Publishing a delta before it points to the old chain head.
* Using a non-atomic mapping-table update.
* Reusing logical IDs unsafely.
* Assuming CAS success under contention.
* Persisting deltas without ensuring mapping recovery.

### Recall questions

1. Why do parents point to logical IDs?
2. What does the expected value in CAS protect against?
3. What does a failed CAS imply?
4. How does the mapping table remove ordinary page latching?
5. Which state must be reconstructed after a restart?

---

## 15. Bw-Tree Splits

### Core idea

A Bw-Tree still needs splits when a logical node becomes too large, but the split is published through deltas.

### Split process

1. Consolidate the current logical node.
2. Divide contents at a split key.
3. Create a new right sibling.
4. Append a split delta to the old node.
5. Later add the sibling to the parent.



### Split delta

The split delta contains:

* Split separator
* Link to the new right sibling
* Range information indicating which keys moved

### Half-split state

Before parent update:

```text
Parent
   ↓
Old logical node ──▶ New right sibling
```

A reader reaches the old node, notices the split boundary and follows the sibling.

This resembles a B-link Tree half-split.

### Important fact

The parent update is primarily a routing optimisation.

Correctness can remain intact because the new sibling is reachable through the split delta.

### Cooperative completion

Any thread encountering an unfinished structural modification may help finish it before continuing. 

### Benefits of helping

* No single paused thread permanently blocks completion.
* Structural progress becomes system-wide.
* Lock-free guarantees are easier to maintain.

### Costs

* Ordinary operations may perform unexpected maintenance work.
* Structural states must be restartable and idempotent.
* Threads require logic for recognising incomplete operations.

### Common mistakes

* Making the sibling visible before fully initialising it.
* Removing moved records from the old range without installing a forwarding path.
* Assuming the parent must update atomically with the split.
* Allowing two helpers to complete the operation inconsistently.
* Treating an incomplete split as unreachable corruption.

### Recall questions

1. How is a new sibling reachable before parent update?
2. Why is parent modification primarily a performance optimisation?
3. What does the split delta tell a reader?
4. Why may another thread complete the split?
5. What properties must make cooperative completion safe?

---

## 16. Bw-Tree Merges

### Core idea

Merges also occur as a sequence of published delta operations.

Simplified process:

1. Add a remove delta to the right sibling.
2. Add a merge delta to the left sibling.
3. Remove the right-child reference from the parent.



### Intermediate state

After the merge delta:

```text
Left logical node
    → includes right node contents
```

The parent may still point to both nodes temporarily, but the right node is marked as being removed.

### Concurrent structural modifications

A special abort delta may act like a temporary exclusive marker so conflicting split or merge operations do not proceed simultaneously on the same structure.

### Root growth

When the logical root becomes too large:

1. Split it.
2. Create a new root.
3. Make the old root and new sibling its children.

### Common mistakes

* Removing the parent pointer before right-node contents are reachable from the left.
* Allowing reads to treat a remove delta as ordinary deletion of one key.
* Running conflicting split and merge operations on the same parent.
* Physically reclaiming the right node before old readers finish.
* Assuming lock-free means no operation needs exclusive logical ownership.

### Recall questions

1. Why is the right sibling marked before its parent pointer is removed?
2. What role does the merge delta serve?
3. Why may an abort delta resemble a write lock?
4. Which update changes tree height?

---

## 17. Consolidation

### Core idea

Delta chains are periodically merged into a new base node.

```text
Base + all deltas
    ↓ reconstruct current state
    ↓
Write new compact base
    ↓
Atomically update mapping table
```



### Trigger

Consolidation may occur when:

* Chain length exceeds a threshold
* Read cost becomes excessive
* Node size requires split
* Background maintenance selects the node

### Benefit

* Shorter read path
* Better locality
* Removes obsolete deltas
* Can compress current state
* Reclaims logical fragmentation

### Cost

* Reads and applies the complete chain
* Writes a new base node
* Old base and deltas remain temporarily allocated
* Competes with ordinary I/O and CPU work

### Threshold trade-off

| Short threshold        | Long threshold                      |
| ---------------------- | ----------------------------------- |
| Faster reads           | Fewer consolidations                |
| More background writes | Lower maintenance frequency         |
| More base rewrites     | Longer reconstruction path          |
| Better locality        | Lower immediate write amplification |

### Common mistakes

* Modifying the existing base during consolidation.
* Replacing the mapping pointer before the new base is complete.
* Freeing old deltas immediately after pointer replacement.
* Consolidating using an inconsistent chain snapshot.
* Choosing one fixed threshold without workload measurement.

### Recall questions

1. Why is consolidation required?
2. What atomic operation publishes the new base?
3. How does the threshold trade read amplification against write amplification?
4. Why must the old chain survive after replacement?

---

## 18. Epoch-Based Reclamation

### Core idea

After consolidation, old nodes are unreachable to new readers but may still be used by readers that started earlier.

Bw-Trees use epochs to delay physical reclamation. 

### Mental model

```text
Epoch E:
reader R1 may have seen old node

Mapping changes during E
    ↓
Readers beginning later cannot see old node

Wait until all readers from E or earlier finish
    ↓
Reclaim old node safely
```

### Why reference counting is difficult

Latch-free readers may access nodes without registering on every node individually.

Epochs track broad reader generations rather than per-node ownership.

### Reclamation rule

A retired node can be freed when:

* It is no longer reachable from the mapping table.
* Every reader that could have observed it has left its epoch.

### Benefits

* Low per-read overhead
* Compatible with lock-free access
* Batch reclamation

### Costs

* One stalled reader can delay reclamation.
* Memory usage may grow under long read sections.
* Epoch entry and exit must be correct.
* Thread failure needs handling.

### Common mistakes

* Reclaiming based only on mapping-table reachability.
* Letting a reader access nodes after leaving its epoch.
* Failing to advance epochs.
* Ignoring suspended or crashed readers.
* Reusing freed memory while stale pointers may remain.

### Recall questions

1. Why is an unreachable node not immediately safe to free?
2. Which readers may still hold its pointer?
3. How does a long-running reader affect memory usage?
4. Why are epochs attractive for latch-free structures?
5. What must a reader do before accessing reclaimable nodes?

---

## 19. Bw-Tree Trade-Off Summary

| Property          | Benefit                         | Cost                         |
| ----------------- | ------------------------------- | ---------------------------- |
| Delta updates     | Avoid full-page rewrites        | Read reconstruction          |
| Logical IDs       | Stable tree links               | Mapping-table lookup         |
| CAS publication   | Latch-free ordinary updates     | Retries under contention     |
| Cooperative SMOs  | Progress despite paused threads | Complex state machine        |
| Consolidation     | Shorter chains                  | Rewrite and maintenance work |
| Epoch reclamation | Safe lock-free reads            | Delayed memory reuse         |

### Core mental model

```text
Traditional B-Tree:
physical page is the node

Bw-Tree:
node is a logical history resolved through indirection
```

### Recall questions

1. Which three amplification or concurrency problems does the Bw-Tree target?
2. Why is a logical node more flexible than a fixed physical page?
3. What becomes the main read-amplification source?
4. Which hot-spot problem can remain despite latch-free updates?
5. How do consolidation and epoch reclamation interact?

---

# Cache-Oblivious B-Trees

## 20. Cache-Aware vs Cache-Oblivious

### Cache-aware design

A conventional B-Tree chooses parameters using known hardware sizes:

* Disk block size
* Page size
* CPU cache line
* Memory hierarchy

### Cache-oblivious design

The algorithm does not explicitly know those sizes but arranges data so that it performs asymptotically well across many adjacent memory levels. 

### Mental hierarchy

```text
CPU registers
    ↓
L1 cache
    ↓
L2/L3 cache
    ↓
RAM
    ↓
SSD/HDD
```

A cache-oblivious layout attempts to provide locality at several scales simultaneously.

### Core principle

If an arrangement behaves efficiently for an arbitrary two-level hierarchy, it can also behave efficiently between adjacent levels of a larger hierarchy.



### Benefits

* Less platform-specific tuning
* Portable locality properties
* Can exploit several cache levels
* Useful when block sizes differ across systems

### Costs and limitations

* Hardware-independent asymptotic optimality may hide constants.
* Real systems still require page allocation and eviction.
* Updates and concurrency remain complex.
* Conventional B-Trees already provide similar asymptotic block-transfer bounds.
* Implementation support is limited.

### Common mistakes

* Assuming cache-oblivious means caches are ignored.
* Expecting identical real-world performance on every machine.
* Treating asymptotic optimality as proof of lower latency.
* Forgetting page-management and persistence concerns.
* Assuming no tuning parameters exist anywhere in the system.

### Recall questions

1. What information does a cache-aware B-Tree use explicitly?
2. How can one layout benefit several cache levels?
3. Why might asymptotic equivalence not imply practical superiority?
4. Which database concerns remain outside the layout algorithm?

---

## 21. van Emde Boas Layout

### Core idea

A van Emde Boas layout recursively groups related tree substructures into contiguous memory regions.

The tree is divided near its middle height:

1. Store the upper portion.
2. Recursively store each lower subtree.
3. Repeat the division inside every subtree.



### Mental model

Instead of storing nodes level by level:

```text
root | all level-1 nodes | all level-2 nodes
```

store recursively related regions together:

```text
top subtree | first lower subtree | second lower subtree | ...
```

The diagram on PDF page 146 contrasts logical tree relationships with their contiguous physical arrangement.

### Why locality improves

A root-to-leaf traversal tends to stay within one contiguous region for several levels before crossing into another region.

This benefits unknown block sizes because:

* Small blocks capture small subtrees.
* Larger blocks capture larger recursive subtrees.

### Trade-off

| Recursive contiguous layout | Ordinary page-node layout     |
| --------------------------- | ----------------------------- |
| Multiscale locality         | Explicit page alignment       |
| Hardware-size independent   | Simpler updates               |
| Good static traversal       | Easier node-level split/merge |
| Complex dynamic movement    | Stable page identifiers       |

### Common mistakes

* Confusing logical tree shape with physical array order.
* Assuming every subtree fits one block.
* Updating elements without maintaining the recursive layout.
* Treating the layout as a complete dynamic-tree algorithm.

### Recall questions

1. Why is the tree divided recursively near its middle height?
2. How does contiguous subtree storage help unknown block sizes?
3. Which operation is more difficult than in a page-based B-Tree?
4. Does physical order change the logical search-tree relationships?

---

## 22. Packed Memory Arrays

### Core idea

A packed array stores sorted elements in contiguous memory while deliberately leaving gaps for future insertions.

```text
[2][ ][5][ ][ ][11][14][ ][18]
```



### Insertion

```text
Find logical position
    ↓
Use nearby gap if available
    ↓
Otherwise redistribute a surrounding region
```

### Density thresholds

Each region must remain between lower and upper occupancy bounds.

* Too dense → spread elements or grow structure.
* Too sparse → compact elements or shrink structure.

### Role in a dynamic cache-oblivious B-Tree

* Static recursive tree provides the index.
* Packed array stores bottom-level elements.
* Index pointers must be updated when array elements relocate.

### Benefits

* Preserves sorted order
* Reduces relocation compared with a completely packed array
* Provides contiguous scanning
* Supports dynamic insertion

### Costs

* Reserved gaps cause space overhead.
* Insertion may relocate many nearby elements.
* Rebuilds are occasionally required.
* Index references must track relocation.
* Concurrency around movement is difficult.

### Relationship to B-Tree occupancy

Both B-Trees and packed arrays reserve unused capacity to avoid immediate global restructuring.

```text
B-Tree → free entries inside pages
Packed array → distributed gaps inside array
```

### Common mistakes

* Filling every position and eliminating insertion gaps.
* Relocating elements without updating the index.
* Applying one density threshold to every region size.
* Assuming insertion always moves only one element.
* Ignoring rebuild cost in performance analysis.

### Recall questions

1. Why are gaps deliberately preserved?
2. When must a region be redistributed?
3. What must happen to the static index after relocation?
4. How is packed-array slack similar to B-Tree free space?
5. Which workload could cause frequent rebuilding?

---

## 23. Chapter 6 Design Summary

### Core mental model

B-Tree variants alter where and when update costs are paid.

```text
Copy-on-write:
copy path now, reclaim old path later

Lazy B-Tree:
buffer now, reconcile later

FD-Tree:
write small mutable head now, merge immutable runs later

Bw-Tree:
append delta now, consolidate chain later

Cache-oblivious tree:
arrange recursively now, reduce hierarchy-specific tuning
```

### Key relationships

| Mechanism            | Reduces                          | Introduces                     |
| -------------------- | -------------------------------- | ------------------------------ |
| Copy-on-write        | Reader latching, partial updates | Path copying and reclamation   |
| Per-page buffering   | Repeated page writes             | Read reconciliation            |
| Subtree buffering    | Random writes                    | Multilevel buffer lookup       |
| Immutable runs       | Random lower-level updates       | Merge and lookup amplification |
| Fractional cascading | Repeated full searches           | Fence and bridge metadata      |
| Delta chains         | Full-node rewrites               | Chain traversal                |
| Mapping table        | Parent-pointer rewrites          | Memory indirection             |
| CAS updates          | Latch contention                 | Retry contention               |
| Epoch reclamation    | Unsafe immediate freeing         | Delayed memory reuse           |
| Recursive layout     | Hardware-specific tuning         | Dynamic-update complexity      |

### Rules to retain

* Immutability simplifies reader concurrency but requires version reclamation.
* Copy-on-write publishes a complete version through an atomic root switch.
* Lazy trees calculate current state from a base page plus buffered updates.
* FD-Trees restrict random writes to a small mutable head.
* Fractional cascading uses earlier search results to accelerate later levels.
* Immutable-level deletion requires tombstones.
* A Bw-Tree node is a logical base-plus-delta chain.
* Logical IDs prevent every delta from forcing parent-pointer updates.
* CAS publication orders readers before or after an update.
* Long delta chains require consolidation.
* Unreachable nodes cannot be reclaimed until old readers finish.
* Cache-oblivious layouts optimise locality without fixed block-size knowledge.
* Packed arrays preserve insertion gaps and periodically rebalance density.

### Applied recall questions

1. A workload contains many readers and one writer, with strict snapshot requirements. Which variant’s core technique is naturally suitable, and what reclamation problem follows?
2. A lazy B-Tree read finds a base value, an update and a later deletion. What result should it return?
3. Why can an FD-Tree write efficiently even when the destination key belongs logically to a lower run?
4. A tombstone reaches the lowest FD-Tree level. Under which condition can it be discarded?
5. A Bw-Tree CAS fails while installing a delta. What must the writer do before retrying?
6. Why can a Bw-Tree split remain searchable before its parent is updated?
7. A consolidated Bw-Tree chain is no longer referenced by the mapping table. Why can it still be unsafe to free?
8. How does bridge density in fractional cascading affect read and space costs?
9. Compare the free-space purpose in standard B-Trees and packed memory arrays.
10. Which variant shifts the greatest amount of work into read-time reconstruction, and why?

# Chapter 7: Log-Structured Storage

## 1. Mutable vs Immutable Storage

### Core idea

A mutable structure updates existing disk locations.

An immutable structure never changes an existing file. Newer records are written elsewhere and reconciled with older records during reads or maintenance.

```text
Mutable:
find old record → overwrite/update page

Immutable:
append new version → retain old version → reconcile later
```

LSM Trees are the standard example of immutable, append-oriented storage. B-Trees are the standard example of mutable, in-place storage. 

### Mental model: accounting ledger

Corrections are added as new entries instead of erasing earlier entries.

To determine the current state:

```text
collect relevant entries
    ↓
order by recency
    ↓
apply updates and deletions
    ↓
return latest state
```

### Trade-off

| Mutable storage                       | Immutable storage                      |
| ------------------------------------- | -------------------------------------- |
| Faster direct reads                   | Faster sequential writes               |
| One main current record               | Several versions may coexist           |
| Requires locating record before write | Write can be appended immediately      |
| Page fragmentation possible           | Immutable files remain densely packed  |
| In-place concurrency is complex       | Reads and file creation interfere less |
| Maintenance reclaims page holes       | Maintenance merges files and versions  |

In-place structures optimise lookup by storing the current record at a known location, but updates require locating and rewriting that location. Append-only storage avoids the lookup on the write path but pushes reconciliation work to reads and compaction. 

### Common mistakes

* Assuming immutable storage contains only one record per key.
* Treating append-only as maintenance-free.
* Comparing only foreground write speed while ignoring compaction.
* Assuming immutable files mean the entire database is immutable.
* Treating an old physical record as logically visible.

### Quick example

```text
File A: customer42 → address X
File B: customer42 → address Y
```

The current value is `Y` if File B contains the newer record.

### Recall questions

1. Why does immutable storage improve the foreground write path?
2. Which work moves from writes to reads?
3. Why are immutable files generally more densely packed?
4. How do mutable and immutable systems reclaim obsolete space differently?

---

## 2. LSM Tree Mental Model

### Core idea

An LSM Tree buffers writes in memory and periodically converts them into immutable, sorted disk files.

```text
Client write
    ↓
Write-ahead log
    ↓
Mutable in-memory table
    ↓ flush
Immutable sorted file
    ↓ compaction
Larger immutable sorted file
```

LSM files usually contain their own indexes. Those internal indexes may themselves use B-Trees or other lookup structures. 

### Why “merge”?

New files do not replace old files immediately.

Several sorted sources are merged to:

* Resolve duplicate keys
* Apply updates
* Apply tombstones
* Remove shadowed records
* Reduce the number of files
* Produce larger sorted files

### Key relationship

```text
Buffering
    → groups many writes

Sorting
    → permits sequential flush and merge

Immutability
    → avoids in-place modification

Compaction
    → controls duplicate and file growth
```

### Why it matters

Immutable files allow:

* Sequential writes
* High storage density
* No reserved update space
* Simpler file-level concurrency
* Reduced random write I/O

They are especially useful when ingest volume is high relative to read volume. 

### Important exception

LSM Trees do not eliminate random reads. Lookups may still access:

* The memtable
* A flushing memtable
* Several immutable files
* Index blocks
* Data blocks

### Common mistakes

* Thinking an LSM Tree is one physical tree.
* Assuming every LSM file has the same internal format.
* Ignoring the WAL because the memtable is in memory.
* Assuming a write is fully persisted once it reaches the memtable.
* Treating compaction as an optional optimisation.

### Recall questions

1. Which LSM property makes disk writes sequential?
2. Why can a B-Tree still appear inside an LSM implementation?
3. What would happen if immutable files were never merged?
4. Why are LSM writes fast even when the previous key value exists on disk?

---

## 3. Core LSM Components

### Memtable

A **memtable** is the mutable, memory-resident component.

It:

* Accepts writes
* Participates in reads
* Maintains sorted key order
* Flushes when it reaches a threshold

### Write-ahead log

The WAL protects memtable changes against crashes.

Correct write path:

```text
append operation to WAL
    ↓
make WAL durable as required
    ↓
apply operation to memtable
    ↓
acknowledge write
```

### Disk-resident table

A flushed table is:

* Sorted
* Immutable
* Used for reads
* Created through sequential output
* Eventually merged by compaction



### Mental model

| Component         | Role                            |
| ----------------- | ------------------------------- |
| WAL               | Durable history                 |
| Current memtable  | Mutable working set             |
| Flushing memtable | Frozen snapshot being persisted |
| Immutable table   | Durable sorted state            |
| Compaction        | Reconciliation and cleanup      |

### Why the memtable is sorted

Sorted memory contents allow flush to write records sequentially in final key order.

Without in-memory ordering, the engine would need to:

* Sort before flush
* Write unordered files
* Build additional structures later

### Common mistakes

* Removing a record from the WAL before its table is durable.
* Modifying a memtable after it has entered the flushing state.
* Returning results from disk without checking memory components.
* Assuming memtable writes have no durability cost.
* Choosing a memtable threshold without considering flush bandwidth.

### Recall questions

1. Why does an LSM Tree need both a WAL and a memtable?
2. What makes a memtable eligible for sequential flushing?
3. When can the WAL records for one memtable be removed?
4. Which components must a lookup inspect before a flush completes?

---

## 4. Two-Component LSM Tree

### Core idea

A two-component LSM Tree contains:

1. One mutable memory component
2. One immutable disk-resident tree

The disk component may be a fully occupied, read-only B-Tree. 

### Flush process

```text
Select memory subtree
    +
Find matching disk subtree
    ↓
Merge both sorted streams
    ↓
Write replacement disk subtree
    ↓
Atomically publish replacement
```

### Merge mechanics

Because both sources are sorted:

1. Hold one iterator position in each source.
2. Compare current keys.
3. Emit the smaller key.
4. Resolve equal keys.
5. Advance corresponding iterator.

### Required invariants

During a flush:

1. New writes must go to a new memory component.
2. The old memory subtree must remain readable.
3. The old disk subtree must remain readable.
4. The replacement must not become visible while incomplete.
5. Publication and retirement must behave atomically.



### Trade-off

| Benefit                               | Cost                                            |
| ------------------------------------- | ----------------------------------------------- |
| Read path has one main disk structure | Every flush may rewrite an existing disk region |
| Few disk components                   | Frequent merge work                             |
| Dense read-only disk pages            | Higher write amplification                      |
| Simple component model                | Complex atomic subtree replacement              |

### Common mistakes

* Sending new writes into the memtable currently being flushed.
* Removing the old disk subtree before publishing the replacement.
* Making partially written merged output readable.
* Forgetting that readers may still use the old components.
* Assuming full node occupancy leaves room for later in-place updates.

### Recall questions

1. Why must a new memtable be installed when flushing starts?
2. Which components remain visible while the merge runs?
3. Why does this design have potentially high write amplification?
4. How is the merge similar to copy-on-write?

---

## 5. Multicomponent LSM Tree

### Core idea

A multicomponent LSM Tree flushes each memtable into a new immutable file instead of immediately merging it with one disk tree.

```text
Memtable 1 → Table A
Memtable 2 → Table B
Memtable 3 → Table C
```

This makes flushing simple and sequential, but increases the number of components that reads may need to inspect.

### Compaction

Compaction periodically:

1. Selects several immutable tables.
2. Reads them sequentially.
3. Merge-sorts their records.
4. Reconciles duplicate keys.
5. Writes one or more replacement tables.
6. Atomically replaces the old tables.



### Mental model

```text
Flush:
small and frequent immutable file creation

Compaction:
larger and less frequent immutable file replacement
```

### Key trade-off

```text
Less work per flush
    → more files
    → more potential read work
    → compaction required
```

### Common mistakes

* Treating each flush file as independent permanent storage.
* Running compaction without enough temporary disk capacity.
* Publishing compacted files before they are complete.
* Deleting source tables while active readers still reference them.
* Assuming compaction must always produce exactly one output file.

### Recall questions

1. Why does multicomponent flushing cost less than immediate subtree merging?
2. What problem results from repeated flushes?
3. Which operations must be atomic at compaction completion?
4. Why might compaction split one input table into several outputs?

---

## 6. Memtable State Transitions

### Core idea

A memtable must be frozen before it can be flushed.

```text
Current memtable
    ↓ threshold reached
Atomic switch
    ├── new current memtable
    └── old flushing memtable
```

### Component states

| Component                    | Writes | Reads |
| ---------------------------- | -----: | ----: |
| Current memtable             |    Yes |   Yes |
| Flushing memtable            |     No |   Yes |
| Incomplete flush target      |     No |    No |
| Completed immutable table    |     No |   Yes |
| Compaction input             |     No |   Yes |
| Incomplete compaction output |     No |    No |



### Flush publication sequence

```text
Freeze old memtable
    ↓
Install new current memtable
    ↓
Write old memtable sequentially
    ↓
Finish and validate table
    ↓
Publish table
    ↓
Retire flushing memtable
    ↓
Trim corresponding WAL
```

The current and flushing memtables both participate in reads until flush publication completes. 

### Why incomplete output is hidden

An incomplete file may:

* Lack its final index
* Lack checksums
* Contain only a prefix of keys
* Have incomplete metadata
* Fail to include all memtable updates

### Common mistakes

* Exposing the flush target before finalisation.
* Discarding the flushing memtable when file writing begins.
* Trimming the WAL before the completed table is durable.
* Allowing post-freeze writes into the old memtable.
* Failing to include both memtables in reads.

### Recall questions

1. Why must memtable switching be atomic?
2. Why is the flush target excluded from reads?
3. Which event permits WAL trimming?
4. Which memory structures are read while flushing is in progress?

---

# Updates and Deletes

## 7. Updates as New Records

### Core idea

LSM Trees do not distinguish structurally between insert and update.

Both create a newer record for the key:

```text
Older table: K → V1
Memtable:    K → V2
```

During reconciliation, `V2` shadows `V1`.

This behaviour is commonly described as an **upsert**. 

### Why it matters

The write path does not need to determine whether:

* The key already exists
* The record is in memory
* The record is in one of several files
* The key has several historical versions

### Trade-off

```text
No read-before-write
    → faster writes

But:
    → duplicate versions
    → read reconciliation
    → compaction work
```

### Common mistakes

* Performing a disk lookup before every update.
* Returning the first value found without considering recency.
* Assuming duplicate keys indicate corruption.
* Treating file creation order as sufficient when record timestamps differ.

### Recall questions

1. Why are inserts and updates structurally identical in an LSM Tree?
2. Which cost is avoided by not checking for an old value?
3. Where is the duplicate-version cost eventually paid?

---

## 8. Point Tombstones

### Core idea

Deleting only the newest in-memory value can reveal an older disk value.

Example:

```text
Disk:      K → V1
Memtable:  K → V2
```

Removing `V2` from memory would expose `V1` again.

Therefore deletion is recorded as a tombstone:

```text
Disk:      K → V1
Memtable:  K → TOMBSTONE
```

The tombstone shadows every older version of `K`.  

### Mental model

A tombstone means:

> Ignore older values for this key.

It does not initially mean:

> No bytes for this key remain anywhere.

### Tombstone lifecycle

```text
Insert tombstone
    ↓
Flush tombstone
    ↓
Propagate through compaction
    ↓
Remove older versions
    ↓
Drop tombstone when no shadowed value can remain
```

### Common mistakes

* Physically removing only the newest record.
* Returning an older value after encountering a newer tombstone.
* Dropping tombstones during the first compaction.
* Counting tombstones as ordinary user values.
* Ignoring transaction or replication visibility before reclamation.

### Recall questions

1. Why can physical removal from the memtable resurrect a deleted value?
2. What does a tombstone logically shadow?
3. When is a tombstone no longer needed?
4. Why do deletes initially increase storage usage?

---

## 9. Range Tombstones

### Core idea

A range tombstone deletes every key matching an ordered predicate.

Example:

```text
delete keys in [K2, K4)
```

Possible representation:

```text
K2 → start tombstone, inclusive
K4 → end tombstone, exclusive
```

Records `K2` and `K3` are hidden; `K4` remains visible. 

### Why it matters

Deleting one million consecutive keys individually would require one million point tombstones.

A range marker can represent the deletion more compactly.

### Complexity

The reconciliation engine must account for:

* Overlapping ranges
* Inclusive and exclusive boundaries
* Different timestamps
* Range markers split across files
* Point updates inside deleted ranges
* Table boundaries

### Rule

A newer point update inside an older deleted range may become visible, depending on timestamp ordering.

```text
Range delete at time 10: [A, Z)
Insert M at time 20
    → M may be visible
```

### Common mistakes

* Treating range markers as ordinary point keys.
* Losing a range tombstone when output files are partitioned.
* Applying a newer range deletion to an even newer point update.
* Using inconsistent endpoint rules.
* Dropping one boundary without preserving the range’s meaning.

### Recall questions

1. Why are range tombstones useful?
2. How do inclusive and exclusive endpoints affect visibility?
3. What happens to range metadata when compaction splits output files?
4. Which wins: an older range tombstone or a newer point update?

---

# Lookups and Reconciliation

## 10. LSM Lookup Model

### Core idea

A lookup may have to inspect several ordered sources:

* Current memtable
* Flushing memtable
* Level-0 tables
* Tables in later levels
* Compaction inputs still visible to readers

The engine must combine and reconcile matching records before returning a result. 

### Recency rule

The latest visible record generally determines the result:

| Latest record | Result                              |
| ------------- | ----------------------------------- |
| Value         | Return value                        |
| Tombstone     | Return not found                    |
| Update        | Return updated value                |
| Expired value | Treat according to expiration rules |

### Point lookup strategies

Possible approaches:

1. Search components newest to oldest and stop when the result is final.
2. Collect matching versions and reconcile them.
3. Use indexes and filters to skip impossible components.

### Range lookup

A range query must merge ordered iterators from every relevant source.

It cannot simply concatenate files because their key ranges and versions may overlap.

### Common mistakes

* Searching only immutable tables.
* Returning the first physical match without knowing component recency.
* Ignoring tombstones during range scans.
* Treating a Bloom-filter positive result as proof of presence.
* Reading compacted output before publication.

### Recall questions

1. Why might a point lookup inspect several components?
2. When can a newest-to-oldest lookup stop early?
3. Why must range scans use merge iteration?
4. Which structures help skip irrelevant files?

---

## 11. Multiway Merge Iteration

### Core idea

Sorted iterators can be combined using a priority queue or min-heap.

The queue holds the current head record from each source. 

### Algorithm

1. Open an iterator for every source.
2. Insert each iterator’s first record into the heap.
3. Remove the smallest key.
4. Add the next record from that record’s source.
5. Reconcile all records with the same key.
6. Continue until the query ends or all sources are exhausted.

```text
Iterator A head ─┐
Iterator B head ─┼→ min-heap → smallest next key
Iterator C head ─┘
```

### Heap invariant

The heap contains the smallest unconsumed record from each active iterator.

Therefore, its minimum is the smallest remaining record across all sources.

### Complexity

For `S` sources and `R` output/consumed records:

| Resource          | Approximate cost |
| ----------------- | ---------------- |
| Heap memory       | `O(S)`           |
| Each queue update | `O(log S)`       |
| Total merge work  | `O(R log S)`     |

The text describes `O(N)` memory for the number of iterators and logarithmic maintenance of ordered iterator heads. 

### Common mistakes

* Loading every source completely into memory.
* Forgetting to refill the source whose head was removed.
* Emitting duplicate same-key records independently.
* Comparing values rather than keys in the merge heap.
* Losing source-recency metadata during heap insertion.

### Quick example

```text
A: K2, K4
B: K1, K2, K3
```

Output key order:

```text
K1, K2, K3, K4
```

The two `K2` entries must be reconciled before output. 

### Recall questions

1. Why does the heap contain only one item per iterator initially?
2. How do sorted inputs guarantee globally sorted output?
3. What happens when one iterator is exhausted?
4. Why must same-key records be grouped before emission?

---

## 12. Reconciliation

### Core idea

Merge sorting determines key order.

**Reconciliation** determines which same-key record is logically valid.

```text
Same key from several sources
    ↓
Compare timestamps and metadata
    ↓
Select newest visible state
    ↓
Discard or retain older versions as required
```

### Required metadata

Records may contain:

* Timestamp
* Sequence number
* Transaction identifier
* Tombstone flag
* Expiration time
* Version
* Source priority

### Basic rule

A record with a greater logical timestamp shadows records with lower timestamps.

Shadowed records:

* Are not returned to the client
* Usually are not copied to compacted output
* May need retention for snapshot or replication semantics



### Important exception

Wall-clock timestamps are not automatically a safe total order.

Systems may instead use:

* Monotonic sequence numbers
* Hybrid timestamps
* Transaction commit order
* Per-partition ordering

### Common mistakes

* Resolving conflicts from file age alone.
* Comparing timestamps generated by unsynchronised clocks without a defined policy.
* Removing a version still visible to an older snapshot.
* Ignoring tombstones during reconciliation.
* Assuming the physically newest file always contains the logically newest record.

### Recall questions

1. What is the difference between merge iteration and reconciliation?
2. Which metadata establishes record precedence?
3. Why might a shadowed version still need temporary retention?
4. Why is file creation time not always sufficient for conflict resolution?

---

# Maintenance and Compaction

## 13. Compaction

### Core idea

Compaction transforms several immutable files into a smaller or better-organised set of immutable files.

```text
Table A
Table B
Table C
    ↓ sequential merge + reconciliation
New Table D
    ↓ atomic publication
Remove A, B and C
```



### Main purposes

* Reduce number of files
* Remove shadowed versions
* Remove safe tombstones
* Improve key-range organisation
* Reduce read amplification
* Reclaim disk space

### Resource behaviour

Because inputs are sorted:

* Reads are sequential.
* Output writes are sequential.
* Memory can remain bounded to iterator and buffering state.
* Temporary disk space is required for old and new tables simultaneously.

### Concurrent compactions

Several compactions may run concurrently when they operate on non-overlapping table sets or key ranges.

### Compaction output

Compaction may:

* Merge several files into one
* Merge several files into multiple partitioned outputs
* Split a large file
* Move data between levels

### Common mistakes

* Starting more compactions than storage bandwidth can support.
* Compacting overlapping source sets concurrently.
* Removing source files before old readers finish.
* Ignoring temporary disk-space requirements.
* Assuming compaction always reduces total byte count.

### Recall questions

1. Why is compaction memory-bounded despite processing large files?
2. Why must input files remain available until publication completes?
3. When can compactions safely run in parallel?
4. Why might one compaction produce several output tables?

---

## 14. Tombstones During Compaction

### Core idea

A tombstone can be discarded only when no older matching value can survive outside the compaction input set.

```text
Compaction sees:
Tombstone K
Older value K

But another unprocessed table may still contain K
    → tombstone must remain
```



### Safe-drop condition

A tombstone is safe to remove when the engine can prove:

```text
No lower/older table contains a value it must shadow
```

Additional distributed-system constraints may include:

* All replicas have observed the deletion.
* Repair windows have passed.
* No snapshot requires the older version.
* Backup or change-stream semantics permit removal.

### Data resurrection

Dropping a tombstone too early can cause:

```text
Delete marker disappears
    ↓
older value remains elsewhere
    ↓
lookup returns deleted value
```

### Common mistakes

* Dropping tombstones based only on age.
* Assuming compaction includes every table containing the key.
* Ignoring replicas that missed the delete.
* Treating tombstone retention as purely a space issue.
* Removing range tombstones without proving complete coverage.

### Recall questions

1. What proof is required before dropping a tombstone?
2. How does early removal cause resurrection?
3. Why may distributed stores retain tombstones for a grace period?
4. How do snapshots affect tombstone reclamation?

---

## 15. Leveled Compaction

### Core idea

Leveled compaction organises files into levels with increasing target capacities.

```text
L0 → L1 → L2 → L3
fresh             old
small capacity    large capacity
```



### Level 0

* Created directly from memtable flushes
* Files may have overlapping key ranges
* Point lookups may need to inspect several L0 files

### Level 1 and later

* Files within one level normally have non-overlapping key ranges
* Metadata can identify the one file whose range may contain a key
* Each later level has greater total capacity

### Compaction process

For level `Li`:

1. Select one or more source files.
2. Find overlapping files in `Li+1`.
3. Merge and reconcile them.
4. Partition output into non-overlapping files.
5. Publish output at `Li+1`.
6. Remove old inputs safely.

### Capacity growth

Level capacities generally grow geometrically:

```text
L1 = X
L2 = X × ratio
L3 = X × ratio²
```

Fresh data begins at low-index levels and migrates toward higher-index levels. 

### Benefits

* Limited file overlap at later levels
* Predictable point-read cost
* Good space reclamation
* Efficient range selection
* Clear table-range metadata

### Costs

* Data may be rewritten at every level.
* One source file may overlap many next-level files.
* Compaction can create substantial write amplification.
* Level 0 can become a write stall bottleneck.

### Common mistakes

* Assuming L0 files have disjoint ranges.
* Compacting into a later level without merging overlapping files.
* Allowing level size to grow unbounded.
* Confusing level index with physical storage speed.
* Ignoring compaction debt at level 0.

### Recall questions

1. Why are level-0 reads more expensive?
2. Why do later-level files avoid overlapping ranges?
3. What determines which next-level files enter a compaction?
4. How does geometric growth limit the number of levels?
5. Why can leveled compaction cause high write amplification?

---

## 16. Size-Tiered Compaction

### Core idea

Size-tiered compaction groups files of similar size.

```text
four small tables
    ↓ merge
one medium table

several medium tables
    ↓ merge
one large table
```



### Mental model

Leveled compaction groups primarily by destination level and key overlap.

Size-tiered compaction groups primarily by similar file size.

### Benefits

* Large sequential merges
* Lower rewrite frequency than aggressive leveled compaction
* Good write throughput
* Simple grouping policy

### Costs

* Key ranges may overlap extensively.
* Reads may inspect several files.
* Duplicate versions remain longer.
* Space amplification may be higher.
* Temporary disk usage during large merges can be substantial.

### Table starvation

A compaction output may remain too small to qualify for the next tier, especially when:

* Tombstones remove many records.
* Duplicate records collapse.
* Expired records are discarded.

Then later levels may not compact, causing:

* Old tombstones to remain
* More files to be searched
* Increased read cost

Forced compaction may be required. 

### Common mistakes

* Assuming files of equal age are also similar in size.
* Waiting forever for a tier to reach its normal trigger.
* Ignoring overlapping ranges during reads.
* Underestimating temporary disk space.
* Treating fewer rewrites as universally better.

### Recall questions

1. What characteristic groups tables in size-tiered compaction?
2. Why can size-tiered compaction improve write throughput?
3. How does table starvation occur?
4. Why may a forced compaction be necessary?
5. Which amplification commonly increases compared with leveled compaction?

---

## 17. Time-Window Compaction

### Core idea

Time-window compaction groups records and files by time interval.

It is useful when records:

* Arrive roughly in timestamp order
* Have time-to-live values
* Expire in groups
* Are queried primarily by recent time ranges



### Mental model

```text
Files for Monday
Files for Tuesday
Files for Wednesday
```

Once Monday’s complete time window expires, the system may delete whole files rather than rewriting them to remove individual expired records.

### Benefits

* Efficient time-based expiry
* Reduced compaction of old immutable windows
* Whole-file deletion
* Good fit for time-series workloads

### Assumptions

* Late-arriving data is limited or handled explicitly.
* Timestamps map records to suitable windows.
* Queries align reasonably with time ranges.
* Entire files eventually become obsolete.

### Common mistakes

* Using event timestamps that arrive heavily out of order without accounting for late data.
* Mixing long-lived and short-lived records in one window.
* Compacting expired files instead of dropping them.
* Choosing windows much smaller than normal write batches.
* Assuming time-window compaction is best for arbitrary key access.

### Recall questions

1. Why can time-window compaction drop entire files?
2. Which workload characteristic makes it effective?
3. How does late-arriving data complicate the design?
4. Why should records with different retention periods be separated?

---

# Amplification and Cost Models

## 18. Read, Write and Space Amplification

### Read amplification

Extra work caused by consulting several components for one logical read.

Examples:

* Searching multiple tables
* Reading false-positive Bloom-filter candidates
* Reconciling versions
* Following indexes and data blocks

### Write amplification

Total physical bytes written relative to logical user bytes written.

In an LSM Tree, it is mainly caused by repeated compaction migration.

```text
User writes 1 MB
Storage writes 8 MB through flushes and compactions
Write amplification = 8×
```

### Space amplification

Extra storage occupied beyond the latest logical dataset.

Sources include:

* Duplicate versions
* Tombstones
* Compaction inputs and outputs coexisting
* Temporary files
* Indexes and filters



### Core trade-off

```text
Compact aggressively
    → lower read amplification
    → lower long-term space amplification
    → higher write amplification

Compact less
    → lower immediate write amplification
    → higher read and space amplification
```

### B-Tree comparison

Write amplification has different causes:

| B-Tree                         | LSM Tree                                  |
| ------------------------------ | ----------------------------------------- |
| Page rewrite for small updates | Repeated file migration during compaction |
| Split and merge writes         | Multi-level rewriting                     |
| Writes unused page bytes       | Writes older live records again           |

Direct comparison requires workload, page size, compaction policy and hardware context.

### Common mistakes

* Defining write amplification using operation count rather than bytes.
* Ignoring temporary compaction space.
* Claiming all LSM Trees have lower write amplification than all B-Trees.
* Optimising write amplification while causing unacceptable read latency.
* Measuring amplification only during idle periods.

### Recall questions

1. What is the primary source of LSM write amplification?
2. How can aggressive compaction reduce read amplification?
3. Why do compaction inputs and outputs increase temporary space amplification?
4. Why is a universal B-Tree-versus-LSM amplification claim unreliable?

---

## 19. RUM Conjecture

### Core idea

The RUM model considers three overheads:

* **R — Read**
* **U — Update**
* **M — Memory**

Improving two generally worsens the third. 

### Mental model

```text
Fast reads
Fast updates
Low memory

Choose two to optimise strongly;
the third usually pays.
```

### Examples

| Design choice              | Improves        | Costs                     |
| -------------------------- | --------------- | ------------------------- |
| Large Bloom filters        | Read            | Memory                    |
| More aggressive compaction | Read, space     | Update/write work         |
| Large memtables            | Update batching | Memory and recovery size  |
| More indexes               | Read            | Update and memory/storage |
| Fewer compactions          | Update          | Read and space            |

### B-Tree tendency

* Strong read performance
* More update work
* Reserved node space

### LSM tendency

* Fast foreground updates
* Dense immutable files
* More read reconciliation and auxiliary memory structures



### Important limitation

RUM does not directly capture:

* Tail latency
* Compaction stalls
* Hardware behaviour
* Operational complexity
* Replication
* Distributed consistency
* Workload-specific access patterns

### Common mistakes

* Treating RUM as a precise benchmark equation.
* Ignoring memory consumed by filters and indexes.
* Evaluating average cost while ignoring latency spikes.
* Assuming one storage engine is optimal for all three dimensions.

### Recall questions

1. How can a Bloom filter illustrate the RUM trade-off?
2. Which RUM dimension does a large memtable primarily consume?
3. Why is RUM only a first-order model?
4. How could lowering update cost increase read cost?

---

# SSTable Implementation

## 20. Sorted String Tables

### Core idea

An SSTable is an immutable file whose records are stored in sorted key order.

Typical structure:

```text
Index file
    key or key prefix → data-file offset

Data file
    sorted key-value records
```



### Index options

An SSTable index may use:

* B-Tree
* Sparse sorted index
* Hash table
* Multi-level index
* Fence pointers

### Range scans with hash indexes

A hash index can still support range scans because it only needs to locate the beginning of the range.

After that:

```text
locate first matching record
    ↓
read sorted data sequentially
    ↓
stop when upper bound is crossed
```

### File construction

During flush or compaction:

1. Write sorted records sequentially.
2. Record each relevant data offset.
3. Build the index using known offsets.
4. Write metadata and checksums.
5. Finalise the file.
6. Publish it as immutable.

Data offsets must be known when index entries are produced. 

### Why compaction does not require indexes for scanning

Compaction consumes every input record in order.

It can read the data sections directly and sequentially because:

* Input files are sorted.
* No random key lookup is needed.
* Merge iteration preserves order.

### Common mistakes

* Modifying an SSTable after publication.
* Building index offsets before final record positions are known.
* Assuming a hash index prevents sequential range scanning.
* Reading index blocks during full-file compaction unnecessarily.
* Publishing a file without complete footer or metadata.

### Recall questions

1. What makes an SSTable suitable for sequential compaction?
2. What does an index entry point to?
3. How can a hash-indexed SSTable support a range scan?
4. Why is the file immutable after completion?
5. Which construction order ensures valid offsets?

---

## 21. SSTable-Attached Secondary Indexes

### Core idea

A secondary index can share the lifecycle of its SSTable.

For every SSTable:

```text
Primary data file
Primary-key index
Secondary index A
Secondary index B
```

When the SSTable is created, compacted or deleted, its attached indexes follow the same transition. 

### Memory component

Because recent records still reside in the memtable, the system also needs an in-memory secondary-index structure.

A secondary-index lookup may therefore inspect:

* Memtable secondary index
* Flushing-memtable index
* Several SSTable-attached indexes

### Read process

1. Search relevant secondary indexes.
2. Produce candidate primary keys.
3. Merge and reconcile index results.
4. Fetch primary records.
5. Reconcile current record versions.
6. Verify the secondary predicate when necessary.

### Benefits

* Index construction fits naturally into flush and compaction.
* Index files are immutable.
* Obsolete index entries disappear through compaction.
* Index and data-file lifecycles remain coupled.

### Costs

* One index per SSTable creates many index components.
* Queries may merge several index result sets.
* Secondary-key updates produce stale entries until compaction.
* Candidate primary records may require verification.
* Index construction increases flush and compaction work.

### Common mistakes

* Maintaining only disk secondary indexes and ignoring memtable data.
* Returning secondary-index results without primary-record reconciliation.
* Deleting an SSTable while retaining its attached index.
* Assuming a secondary-index hit represents the newest record.
* Ignoring stale secondary values after a field update.

### Recall questions

1. Why is a separate in-memory index required?
2. How does compaction simplify index cleanup?
3. Why must secondary-index results be reconciled?
4. What happens to old secondary-key entries after an indexed field changes?

---

# Bloom Filters

## 22. Bloom Filter Core Model

### Core idea

A Bloom filter answers:

```text
Definitely not present
or
Might be present
```

It may return false positives, but not false negatives when implemented correctly. 

### Why LSM Trees use them

Without a filter, an absent-key lookup may access every candidate SSTable.

With one filter per table:

```text
Bloom says no
    → skip table

Bloom says maybe
    → search table index/data
```

### Structure

A Bloom filter contains:

* A bit array
* Several hash functions

Insert key:

```text
hash1(key) → set bit a
hash2(key) → set bit b
hash3(key) → set bit c
```

Lookup key:

```text
Any required bit = 0
    → definitely absent

All required bits = 1
    → possibly present
```



### Why false positives occur

Different keys may set overlapping bit positions.

A query key that was never inserted may hash only to bits set by other keys.

### Why false negatives should not occur

Every inserted key sets all of its required bits.

If even one required bit is zero, that key could not have been inserted—assuming the filter has not been incorrectly modified.

### Important rule

Bloom filters do not contain values and cannot confirm exact membership.

### Common mistakes

* Returning a record merely because the filter is positive.
* Skipping a table after a positive result.
* Clearing bits when individual keys are deleted.
* Building the filter from only part of an SSTable.
* Treating a negative result as probabilistic rather than definitive.
* Using inconsistent hash functions between construction and lookup.

### Recall questions

1. Why can a Bloom filter safely skip a table after a negative result?
2. Why must a positive result still access the table?
3. What causes false positives?
4. Why can individual bits not be cleared safely on deletion?

---

## 23. Bloom Filter Tuning

### Core idea

False-positive probability depends mainly on:

* Number of inserted keys
* Number of filter bits
* Number of hash functions



### Trade-off

| Larger bit array          | Smaller bit array  |
| ------------------------- | ------------------ |
| Lower false-positive rate | Less memory        |
| Better read avoidance     | More table lookups |
| More cache footprint      | Greater collisions |

| More hash functions                   | Fewer hash functions         |
| ------------------------------------- | ---------------------------- |
| More tested positions                 | Lower CPU cost               |
| Can reduce false positives to a point | May provide weaker filtering |
| More hashing work                     | Faster query                 |

### Important exception

Adding hash functions indefinitely does not always improve accuracy.

Too many hashes set too many bits, saturating the filter and eventually increasing false positives.

### Why immutable files help

The SSTable’s key count is known during construction.

The filter can therefore be sized for:

* Expected item count
* Desired false-positive rate
* Memory budget

### Common mistakes

* Choosing filter size without expected key count.
* Assuming more hash functions are always better.
* Measuring only memory cost and not avoided I/O.
* Giving every level the same bits-per-key despite different read patterns.
* Keeping filters off-heap or on disk without considering their access latency.

### Quick example

A 16-bit example may report a key as “possibly present” because all three of its hashed positions were set by other keys, even though the key itself was never added. 

### Recall questions

1. How does increasing bits per key affect false positives?
2. Why can too many hash functions hurt?
3. Which workload benefits most from lower false-positive rates?
4. Why is SSTable immutability useful when sizing the filter?

---

# Memtable Structures

## 24. Skiplists

### Core idea

A skiplist is an ordered linked structure with multiple shortcut levels.

```text
Level 3: HEAD ───────────────→ 40
Level 2: HEAD ───→ 10 ──────→ 40
Level 1: HEAD → 5 → 10 → 20 → 40
```

Higher levels skip over larger key ranges.

Skiplists use probabilistic node heights rather than tree rotations for balancing.  

### Node structure

A node contains:

* Key
* Value or record metadata
* One or more forward pointers
* Randomly selected height

There are exponentially fewer nodes at higher levels.

### Lookup algorithm

1. Begin at the highest level.
2. Move forward while the next key is below the target.
3. When the next key exceeds the target, move down one level.
4. Continue until the target or predecessor is found.



### Mental model

```text
High levels → express lanes
Low level   → local streets
```

### Insert

1. Locate predecessors at every relevant level.
2. Randomly select node height.
3. Link the new node to its successors.
4. Redirect predecessor pointers to the new node.

### Delete

Redirect predecessor pointers around the removed node at each level.



### Why useful for memtables

* Maintains sorted order
* Supports point and range lookup
* No tree rotations
* Simpler concurrent implementation
* Incremental insertion
* Flush can iterate the bottom level sequentially

### Trade-off

| Skiplist                       | In-memory B-Tree                   |
| ------------------------------ | ---------------------------------- |
| Simple probabilistic balancing | Deterministic page/node balancing  |
| Pointer-heavy                  | Better key density                 |
| No rotations                   | May require node splits            |
| Convenient concurrency         | Often more cache-friendly          |
| Sorted linked iteration        | Wider nodes reduce pointer chasing |

Skiplists can be less cache-friendly because their nodes are small and scattered in memory. 

### Concurrent insertion

A node may be linked across several levels in multiple steps.

A completion flag can indicate whether all links have been installed.

Readers must avoid treating a partially linked node as fully published.

Safe memory reclamation may require:

* Reference counting
* Hazard pointers
* Epoch-based reclamation



### Common mistakes

* Selecting every node at the maximum height.
* Publishing a node before required links are installed.
* Freeing a removed node while readers still hold pointers.
* Searching only the bottom level.
* Assuming probabilistic balancing means unbounded practical height.
* Ignoring poor cache locality in pointer-heavy layouts.

### Quick example

Search for key `7`:

```text
High level jumps to 10
10 > 7
    ↓ move down from predecessor

Jump to 5
5 < 7
    ↓ move forward/down

Reach 7
```

### Recall questions

1. How do skiplist levels reduce lookup work?
2. Why are there exponentially fewer high-level nodes?
3. Which operation replaces tree rotation?
4. Why is a publication flag useful in a concurrent skiplist?
5. What makes a skiplist suitable for sequential memtable flushing?
6. Why can an in-memory B-Tree have better cache locality?

## 25. Disk Access and Record Alignment

### Core idea

LSM files are immutable, but their records do not necessarily align with database pages or storage blocks.

```text
Disk blocks:
| Block 1        | Block 2        |

Records:
| Record A | Record B------------|
```

A record crossing a block boundary may require loading multiple blocks even when only one logical record is requested. 

### Differences from mutable pages

| Mutable B-Tree page                       | Immutable LSM file                               |
| ----------------------------------------- | ------------------------------------------------ |
| Records addressed mainly through page IDs | Records may use absolute file offsets            |
| Cached page may be modified               | Cached blocks are read-only                      |
| Page latch protects modifications         | Reference tracking mainly protects file lifetime |
| Records normally contained in page layout | Records may cross block boundaries               |

### Immutable-page concurrency

Immutable cached data does not need write latches.

It still needs lifetime management to ensure:

* A block is not evicted while being read.
* A file is not deleted while a request uses it.
* Compaction inputs remain accessible to old readers.

### Common mistakes

* Assuming every record requires exactly one block read.
* Deleting an SSTable immediately after replacing it in the current view.
* Treating immutability as eliminating reference tracking.
* Using a page ID where the format stores an absolute byte offset.

### Recall questions

1. Why can one logical record require two physical reads?
2. Why do immutable blocks not need write latches?
3. What prevents compaction from deleting a file still used by a reader?
4. How does absolute-offset addressing differ from page-based addressing?

---

## 26. Compression in LSM Tables

### Core idea

Immutable LSM tables can compress blocks independently as they are written sequentially.

Compressed blocks normally have variable sizes, so their physical positions can no longer be calculated using:

```text
offset = page_number × fixed_page_size
```

An indirection structure must record each compressed block’s:

* Physical offset
* Compressed size
* Logical or uncompressed page identity



### Read path

```text
Logical block ID
    ↓
Look up compressed offset and size
    ↓
Read compressed bytes
    ↓
Decompress
    ↓
Materialise uncompressed block in cache
```

During flush or compaction, compressed blocks and their offset metadata can both be generated sequentially. 

### Why padding is undesirable

Padding every compressed block back to the original page size would restore simple addressing but waste most of the saved space.

```text
Original page:   16 KiB
Compressed data: 5 KiB
Padding:         11 KiB
```

### Rule

A block should normally be stored compressed only when the result is smaller than the original representation.

### Trade-off

| Benefit                           | Cost                      |
| --------------------------------- | ------------------------- |
| Lower disk usage                  | Offset table required     |
| More logical data per I/O         | Decompression CPU         |
| Reduced storage bandwidth         | Variable physical extents |
| Immutable blocks compress cleanly | More complex caching      |

### Common mistakes

* Assuming compressed blocks remain fixed-sized.
* Calculating physical position from logical block number alone.
* Padding every block and eliminating the space benefit.
* Keeping a compressed block when it is larger than the original.
* Forgetting that the offset metadata must also be crash-safe.

### Recall questions

1. Why does compression require an extra address-translation layer?
2. What metadata is needed to read one compressed block?
3. When would storing a block uncompressed be preferable?
4. How does immutable sequential writing simplify compression?

---

# Unordered Log-Structured Storage

## 27. Ordered vs Unordered Storage

### Core idea

Sorted LSM Trees pay an in-memory sorting and maintenance cost so they can support ordered retrieval.

Unordered log stores append records directly in arrival order.

```text
Ordered:
buffer → sort → write

Unordered:
append immediately
```

Unordered storage can reduce foreground write work and may use the data log itself as the durability log. 

### Trade-off

| Ordered LSM                             | Unordered log store                |
| --------------------------------------- | ---------------------------------- |
| Supports range scans                    | Primarily point lookup             |
| Sorted compaction                       | Simple append                      |
| Index can exploit key order             | Requires separate key-location map |
| Flush normally follows memtable sorting | May write directly to log          |
| Sequential range access                 | Range values may be scattered      |

### Mental model

* **Sorted storage:** organise data before or during persistence.
* **Unordered storage:** persist cheaply, then maintain a directory pointing to the latest values.

### Common mistakes

* Assuming sequential appends imply sorted keys.
* Expecting range scans from a hash-based key directory.
* Treating absence of a separate WAL as absence of durability.
* Ignoring the memory cost of the key-location directory.

### Recall questions

1. Which write-path work does unordered storage avoid?
2. Why does insertion order not support efficient key-range scans?
3. What structure is required to locate the latest value?
4. How can the data log also provide durability?

---

## 28. Bitcask

### Core idea

Bitcask stores records directly in append-only log files and maintains an in-memory hash map called **keydir**.

```text
keydir:
K1 → file 3, offset 820
K2 → file 1, offset 140
```

Each keydir entry points only to the latest record for that key. Older physical records remain on disk until compaction. 

### Write path

```text
Append [key, value] to active logfile
    ↓
Update keydir[key] to new location
```

### Read path

```text
Hash lookup in keydir
    ↓
Read exact record location
    ↓
Return latest value
```

A point query does not need to reconcile multiple files because keydir already selects the newest location.

### Compaction

```text
Read old logfiles sequentially
    ↓
Copy only live records
    ↓
Write compacted logfile
    ↓
Update keydir to new offsets
    ↓
Delete obsolete files safely
```



### Why a separate WAL is unnecessary

The appended data record is itself the persistent log entry.

```text
One physical write:
durability record + stored value
```

This can reduce:

* Duplicate logging
* Space overhead
* Write amplification

### Strengths

* Simple architecture
* Fast point writes
* Fast point reads
* Sequential disk writes
* No read-time multi-file reconciliation

### Limitations

* Every key must fit in memory.
* Keydir must be rebuilt after restart.
* No efficient range scans.
* Compaction must relocate live values and repair pointers.
* Startup time grows with log volume unless auxiliary summaries exist.

### Common mistakes

* Assuming keydir contains the values themselves.
* Returning an old record found by scanning rather than following keydir.
* Compacting records without updating their keydir offsets.
* Selecting Bitcask for workloads requiring ordered scans.
* Underestimating keydir memory for billions of keys.

### Quick example

```text
Log:
K1=V1
K2=V2
K1=V3

keydir:
K1 → V3 location
K2 → V2 location
```

`K1=V1` is physically present but logically dead.

### Recall questions

1. Why does Bitcask avoid read-time version reconciliation?
2. Which data must be reconstructed during startup?
3. Why can its data file double as the WAL?
4. What prevents efficient range queries?
5. Which records can compaction discard?

---

## 29. WiscKey: Separating Keys from Values

### Core idea

WiscKey stores:

* Keys and value pointers in a sorted LSM Tree
* Large values in an unordered append-only **value log**, or vLog

```text
Sorted key LSM:
K1 → vLog offset 800
K2 → vLog offset 120

Unordered vLog:
... V2 ... V1 ...
```



### Mental model

```text
Small ordered index
    +
Large unordered payload log
```

### Why it helps

Keys are usually much smaller than values.

Compacting only keys and pointers means fewer bytes are repeatedly rewritten.

```text
Value size: 10 KiB
Key + pointer: 40 B

Compaction can move 40 B
instead of repeatedly moving 10 KiB
```

### Point lookup

```text
Search sorted key LSM
    ↓
Obtain latest vLog pointer
    ↓
Read value from vLog
```

### Range scan

1. Scan keys in sorted order.
2. Collect their vLog locations.
3. Fetch values from unordered positions.

The keys are sequential, but value reads may be random. WiscKey attempts to reduce latency by using SSD parallelism and prefetching several value blocks concurrently. 

### Value-log garbage collection

Compaction or garbage collection:

1. Reads vLog segments.
2. Determines which records remain live using the key index.
3. Copies live values elsewhere.
4. Updates pointers in the key LSM Tree.
5. Reclaims old segments.

Head and tail metadata can help limit which vLog regions need examination. 

### Trade-off

| Benefit                                        | Cost                                    |
| ---------------------------------------------- | --------------------------------------- |
| Much lower value rewrite amplification         | Extra pointer lookup                    |
| Smaller LSM levels                             | Random value reads                      |
| Ordered key scans                              | Range payloads are physically scattered |
| Large values append cheaply                    | Complex vLog garbage collection         |
| Key directory need not contain all keys in RAM | Key tree must validate value liveness   |

### Best fit

WiscKey is especially attractive when:

* Values are much larger than keys.
* SSD random-read parallelism is available.
* Updates and deletes are not so frequent that vLog garbage collection dominates.
* Range scans need key order but can tolerate scattered value reads.

### Common mistakes

* Assuming sorted keys imply sorted values.
* Compacting the key tree without preserving correct vLog pointers.
* Treating every vLog record as live.
* Ignoring full-block reads for small scattered values.
* Using the approach when values are tiny and indirection dominates.

### Recall questions

1. How does key-value separation reduce compaction bytes?
2. Why do range scans still incur random I/O?
3. How does garbage collection determine whether a vLog value is live?
4. When would value separation provide little benefit?
5. How does WiscKey solve Bitcask’s all-keys-in-memory limitation?

---

# LSM Concurrency

## 30. Atomic Table-View Changes

### Core idea

An LSM reader observes a **table view**: the current collection of memory and disk components.

Flush and compaction replace that collection while operations continue.

```text
Old view:
current memtable + flushing memtable + tables A, B

New view:
current memtable + new table C + tables A, B
```

Correctness depends on publishing complete new views atomically. 

### Flush synchronisation points

1. **Memtable switch**

   * New writes move to a new memtable.
   * Old memtable becomes immutable.
   * Both remain readable.

2. **Flush finalisation**

   * Completed SSTable replaces the flushing memtable in the view.

3. **WAL truncation**

   * Log records protected by the durable SSTable become removable.



### Required ordering

```text
Freeze old memtable
    ↓
Install new write target
    ↓
Complete all accepted old writes
    ↓
Flush old memtable
    ↓
Make SSTable durable
    ↓
Atomically publish SSTable
    ↓
Retire old memtable
    ↓
Truncate covered WAL
```

### Failure consequences

| Incorrect action                               | Result                                  |
| ---------------------------------------------- | --------------------------------------- |
| Continue writing to partially flushed memtable | Some writes may never reach the SSTable |
| Remove old memtable before table publication   | Reads can miss records                  |
| Truncate WAL before flush durability           | Crash can lose acknowledged writes      |
| Expose incomplete output                       | Reads see partial state                 |
| Compact the same table in two jobs             | Conflicting replacements                |

 

### Compaction publication

Old tables remain readable until all replacement files are complete.

Then the table view atomically changes:

```text
{A, B, C} → {D, E}
```

Readers using the old view may finish against `A`, `B` and `C`; new readers use `D` and `E`.

### Common mistakes

* Updating the table-view collection entry by entry.
* Truncating WAL by time rather than flush coverage.
* Deleting old files immediately after current-view replacement.
* Allowing one SSTable into overlapping compactions.
* Forgetting writes accepted just before memtable freezing.

### Recall questions

1. Why must the old memtable remain readable during flush?
2. What exact event makes WAL truncation safe?
3. Why should table-view replacement be atomic?
4. How can old and new readers safely use different table views?
5. What happens if writes continue into an already flushed memtable region?

---

# Log Stacking

## 31. Multiple Log-Structured Layers

### Core idea

Log structuring may exist at several layers:

```text
Database LSM or WAL
    ↓
Log-structured filesystem
    ↓
SSD flash translation layer
    ↓
Physical flash blocks
```

Each layer independently:

* Batches writes
* Relocates live data
* Reclaims obsolete regions
* Maintains metadata
* Performs garbage collection

Stacking them without coordination can recreate the write amplification and fragmentation each layer was designed to reduce. 

### Mental model

```text
Upper layer says:
“This segment is sequential.”

Lower layer may see:
several interleaved, misaligned fragments
```

### Causes of inefficiency

* Different segment sizes
* Misaligned boundaries
* Independent garbage-collection schedules
* Duplicate liveness tracking
* Multiple simultaneous write streams
* Lower layer moving data the upper layer will soon delete

### Common mistakes

* Assuming sequential application calls produce sequential physical placement.
* Measuring database write amplification without device-level amplification.
* Ignoring filesystem and SSD garbage collection.
* Using arbitrary segment sizes unrelated to underlying erase or page units.
* Running several write streams without considering physical interleaving.

### Recall questions

1. Why can several individually efficient logs be inefficient when stacked?
2. Which work is duplicated between layers?
3. Why might the lower layer relocate data that is about to be deleted?
4. What information could upper and lower layers share to reduce work?

---

## 32. SSD Flash Translation Layer

### Core idea

Flash memory cannot overwrite a programmed page directly.

```text
Write page:
must target empty page

Reuse old page:
mark old page invalid
write new copy elsewhere
erase old block later
```

Pages are erased only in larger groups called blocks. An SSD’s flash translation layer maps logical addresses to current physical pages and tracks whether pages are:

* Live
* Discarded
* Empty



### Garbage collection

```text
Choose block
    ↓
Copy its live pages elsewhere
    ↓
Update logical mappings
    ↓
Erase entire block
    ↓
Return pages to free pool
```



### Wear levelling

Flash cells tolerate a limited number of program/erase cycles.

Wear levelling distributes erasures and writes across blocks to avoid early failure of heavily used regions. 

### Why batching helps

Combining small random writes into larger sequential writes can:

* Use flash pages more fully
* Reduce relocation frequency
* Reduce garbage-collection pressure
* Improve device lifespan

### Important exception

Application-level sequentiality does not guarantee low device write amplification.

The outcome also depends on:

* Free-space availability
* Block alignment
* FTL policy
* Overprovisioning
* Concurrent write streams
* TRIM/discard information

### Common mistakes

* Treating logical overwrite as physical overwrite.
* Ignoring erase-block granularity.
* Assuming garbage collection moves only obsolete pages.
* Concentrating writes on a small physical region.
* Equating filesystem offsets with stable flash locations.

### Recall questions

1. Why must live pages be relocated before block erasure?
2. What information does the FTL mapping maintain?
3. How does wear levelling extend device lifetime?
4. Why may a sequential database workload still cause physical relocation?

---

## 33. Filesystem Logging and Alignment

### Core idea

A log-structured filesystem may rewrite and collect data independently of the database log above it.

When segment boundaries differ:

```text
Database segment:
|---------- A ----------|---------- B ----------|

Filesystem segments:
|------ 1 ------|------ 2 ------|------ 3 ------|
```

Deleting database segment `A` may leave filesystem segments `1` and `2` only partially free, forcing relocation of neighbouring live data. 

### Multiple write streams

A database may write concurrently to:

* WAL
* SSTables
* Temporary compaction files
* Metadata
* Checkpoints

Although each stream is individually sequential, their writes may interleave physically:

```text
WAL1, DATA1, WAL2, DATA2, META, DATA3
```

The device therefore sees a fragmented aggregate stream. 

### Mitigations

* Align write sizes with storage pages.
* Align partitions and file extents appropriately.
* Use compatible segment sizes.
* Isolate WAL and data streams where useful.
* Communicate discarded ranges to lower layers.
* Limit unnecessary concurrent write streams.
* Consider filesystem and device allocation behaviour.

### Rule

Separating the WAL onto another device may reduce stream interference, but alignment and layer coordination remain important.

### Common mistakes

* Calling every append workload physically sequential.
* Separating logs without measuring the actual bottleneck.
* Writing blocks smaller than the device’s efficient unit.
* Assuming deleting an application file immediately releases aligned erase blocks.
* Ignoring temporary compaction streams.

### Recall questions

1. How do mismatched segment boundaries create fragmentation?
2. Why can two sequential streams produce random physical placement?
3. What benefit can a separate WAL device provide?
4. Why is write alignment still important after stream separation?

---

## 34. LLAMA and Mindful Stacking

### Core idea

Layering is not inherently inefficient.

It becomes efficient when the lower storage layer understands the upper access method’s semantics.

LLAMA is a log-structured storage subsystem designed with awareness of Bw-Tree logical nodes. 

### Ordinary unaware storage

Updates for different logical nodes may be interleaved by arrival order:

```text
Delta A1
Delta B1
Delta A2
Delta C1
Delta B2
```

Even though writes are sequential, one logical node becomes physically fragmented.

### Access-method-aware storage

LLAMA can identify which deltas belong to the same Bw-Tree node and:

* Place related deltas together
* Cancel redundant operations
* Consolidate deltas into a new base node
* Combine garbage collection with logical maintenance



### Example of semantic consolidation

```text
Delta 1: insert K
Delta 2: delete K
```

A semantics-aware layer may avoid writing both final effects and preserve only the required resulting state.

### Combined garbage collection

Without coordination:

```text
Bw-Tree consolidates deltas
    ↓
Storage layer later relocates and collects old physical records
```

The data may be processed twice.

With LLAMA awareness:

```text
Storage garbage collection
    +
Bw-Tree delta consolidation
    ↓
one rewritten compact base node
```

### Why it matters

Mindful stacking can reduce:

* Read-time delta traversal
* Physical fragmentation
* Stored bytes
* Duplicate maintenance
* Write and garbage-collection work

### Design principle

Expose enough semantic information across layers to coordinate optimisation without forcing all functionality into one tightly coupled component.

### Common mistakes

* Assuming abstraction boundaries should hide all lower-level information.
* Co-locating bytes without logically consolidating them.
* Consolidating deltas but leaving storage garbage collection to repeat the same work.
* Allowing lower layers to reinterpret upper-layer semantics incorrectly.
* Coupling layers so tightly that they cannot evolve independently.

### Recall questions

1. How does access-method awareness improve physical placement?
2. Why is simply storing delta nodes contiguously insufficient?
3. Which two maintenance processes can LLAMA combine?
4. What information must the Bw-Tree expose to the storage layer?
5. How can a useful abstraction expose semantics without eliminating modularity?

---

## 35. Open-Channel SSDs

### Core idea

An Open-Channel SSD exposes internal storage-management responsibilities to software instead of hiding them behind a conventional FTL.

Software may control:

* Data placement
* Garbage collection
* Wear levelling
* Page relocation
* I/O scheduling
* Device parallelism



### Mental model

```text
Conventional SSD:
application → filesystem → FTL → flash

Open-channel:
storage engine → exposed flash geometry
```

### Benefits

* Avoid duplicated garbage collection.
* Coordinate database layout directly with erase units.
* Exploit separate flash channels.
* Reduce layer-induced write amplification.
* Make placement decisions using application semantics.

### Costs

* Much greater implementation complexity
* Hardware-specific behaviour
* Responsibility for wear levelling
* More failure cases
* Less portability
* More demanding operational testing

### Asymmetric I/O

Some software-defined-flash designs expose different read and write units.

A write unit aligned with the erase unit can reduce rewrite and relocation overhead, particularly for log-structured storage. 

### Relationship to direct I/O

Both approaches exchange abstraction convenience for greater control:

| Direct I/O                       | Open-channel SSD                      |
| -------------------------------- | ------------------------------------- |
| Bypasses kernel page cache       | Bypasses conventional FTL abstraction |
| Database manages buffering       | Software manages flash placement      |
| More predictable cache control   | More predictable erase and GC control |
| Increased application complexity | Much deeper hardware responsibility   |

### Common mistakes

* Assuming removing layers automatically improves performance.
* Ignoring wear levelling and bad-block handling.
* Applying HDD assumptions to flash geometry.
* Building a device-specific engine without a portability strategy.
* Exposing hardware details without sufficient correctness testing.

### Recall questions

1. Which FTL responsibilities move into software?
2. Why can aligning write and erase units reduce amplification?
3. How does exposing flash channels improve parallelism?
4. What engineering costs replace the removed abstraction layers?
5. How is Open-Channel storage conceptually similar to direct I/O?

---

## 36. Chapter 7 Design Summary

### Core mental model

```text
LSM write:
log → memory → immutable file

LSM read:
memory + candidate files → merge → reconcile

LSM maintenance:
immutable inputs → compacted outputs → retire old files
```

### Key relationships

| Mechanism              | Benefit                            | Cost                             |
| ---------------------- | ---------------------------------- | -------------------------------- |
| Memtable               | Batches and sorts writes           | RAM and flush management         |
| WAL                    | Crash durability                   | Additional append I/O            |
| Immutable SSTable      | Dense sequential storage           | Multiple versions                |
| Compaction             | Reduces files and obsolete records | Write amplification              |
| Tombstone              | Represents deletion across files   | Read and space overhead          |
| Bloom filter           | Skips definitely absent tables     | Memory and false positives       |
| Skiplist               | Concurrent sorted memtable         | Pointer and cache overhead       |
| Key-value separation   | Reduces large-value rewrites       | Random value access              |
| Unordered value log    | Fast append                        | Garbage collection               |
| Table-view publication | Lock-light component replacement   | Lifecycle complexity             |
| Layer coordination     | Reduces duplicated work            | Cross-layer interface complexity |

### Rules to retain

* LSM Trees write immutable files and reconcile versions later.
* A memtable requires a WAL unless another durable append serves the same role.
* A flushing memtable remains visible until its table is complete.
* The WAL can be truncated only after covered data is durably flushed.
* Deletes require tombstones while older values may exist.
* Point tombstones and range tombstones participate in normal ordering.
* Multiway iteration determines key order; reconciliation determines visibility.
* Compaction must preserve tombstones until no older record can reappear.
* Bloom-filter negatives are definitive; positives are only possibilities.
* Bitcask optimises point lookups but cannot naturally provide range scans.
* WiscKey orders keys while leaving large values unordered.
* Several log-structured layers can amplify one another unless coordinated.
* Logical sequentiality does not guarantee physical sequentiality.

### Applied recall questions

1. A memtable has been flushed, but its SSTable footer is not yet durable. Can its WAL segment be discarded?
2. A Bloom filter reports that a key may exist in three tables. What work remains?
3. A Bitcask deployment runs out of RAM while disk capacity remains plentiful. Which design requirement caused this?
4. WiscKey reduces compaction bytes but range-query latency rises. Explain the cause.
5. A range tombstone and a newer point insertion overlap. Which record should become visible?
6. A compaction output is published while old readers still reference its inputs. Which lifecycle mechanism is needed?
7. An application performs sequential WAL and SSTable writes, but device traces show fragmented placement. Why?
8. How can access-method-aware garbage collection reduce both read and write costs?
9. Which amplification type increases when compaction is delayed too long?
10. Why does removing the FTL require the database to understand erase-block behaviour?

---

# Part I Conclusion: Storage-Engine Mental Models

## 37. Three Independent Design Properties

### Core idea

Storage structures can be compared through three properties:

1. **Buffering**
2. **Mutability**
3. **Ordering**

These properties can be combined rather than treated as fixed categories. 

### Comparison

| Structure            |    Buffered | Mutable | Ordered |
| -------------------- | ----------: | ------: | ------: |
| B+ Tree              |          No |     Yes |     Yes |
| WiredTiger lazy tree |         Yes |     Yes |     Yes |
| LA-Tree              |         Yes |     Yes |     Yes |
| Copy-on-write B-Tree |          No |      No |     Yes |
| Two-component LSM    |         Yes |      No |     Yes |
| Multicomponent LSM   |         Yes |      No |     Yes |
| FD-Tree              |         Yes |      No |     Yes |
| Bitcask              |          No |      No |      No |
| WiscKey values       |   Mostly no |      No |      No |
| WiscKey key index    |         Yes |      No |     Yes |
| Bw-Tree              | Delta-based |      No |  Partly |

The book notes that WiscKey uses buffering mainly for maintaining its ordered key structure, while only consolidated Bw-Tree nodes necessarily contain physically ordered records. 

---

## 38. Buffering

### Core idea

Buffering groups multiple logical changes into fewer physical writes.

```text
Many small updates
    ↓
Accumulate in memory
    ↓
One larger write
```

### In mutable structures

Examples:

* WiredTiger
* LA-Trees

Buffering combines repeated modifications to the same page, reducing immediate page rewrites.

### In immutable structures

Examples:

* LSM Trees
* FD-Trees

Buffering creates larger sequential runs, but those runs may later be rewritten as data moves through levels.



### Mental model

```text
Buffering reduces present write cost
but may create future propagation work.
```

### Trade-off

| More buffering                  | Less buffering        |
| ------------------------------- | --------------------- |
| Better write batching           | Lower memory usage    |
| Fewer small I/Os                | Faster propagation    |
| Larger failure-recovery state   | Simpler reads         |
| Potentially larger flush bursts | More immediate writes |

### Recall questions

1. How does buffering reduce mutable-page write amplification?
2. Why can buffering create deferred amplification in immutable structures?
3. What read or recovery costs grow with buffer size?

---

## 39. Immutability

### Core idea

Immutability allows pages and files to be:

* Fully occupied
* Read without observing in-place mutation
* Written sequentially
* Shared safely by concurrent readers

But updates create new versions, requiring later reconciliation and reclamation.



### Cause and effect

```text
No in-place modification
    → simpler reader concurrency
    → dense pages
    → duplicate versions
    → compaction or garbage collection
```

### Important exception

Immutability does not always mean sorted storage.

Examples:

* Bitcask stores immutable records in insertion order.
* WiscKey stores values unordered.
* Bw-Tree deltas may be scattered physically.

### Recall questions

1. Why can immutable pages reach higher occupancy?
2. Which maintenance work is caused by duplicate immutable versions?
3. Why does immutability not imply key ordering?

---

## 40. Ordering

### Core idea

Ordering improves:

* Range scans
* Merge efficiency
* Locality between adjacent keys
* Ordered iteration
* Predicate operations

Maintaining order adds cost through:

* In-place movement
* Sorting
* Merging
* Buffering
* Index maintenance

### Mental model

```text
Pay to organise now
    → cheaper ordered retrieval later

Avoid organisation now
    → cheap append
    → expensive range access later
```

### Examples

| Design  | How order is maintained                        |
| ------- | ---------------------------------------------- |
| B-Tree  | Sorted pages and structural updates            |
| SSTable | Sort in memory before flush                    |
| FD-Tree | Merge sorted runs                              |
| WiscKey | Sort keys only                                 |
| Bitcask | Does not maintain key order                    |
| Bw-Tree | Reconstruct logical order from base and deltas |

### Recall questions

1. Why does key ordering benefit merge-based compaction?
2. How does WiscKey preserve ordered lookup without ordering values?
3. Which workload can reasonably choose unordered storage?

---

## 41. Unified Trade-Off Model

### Core idea

No storage structure minimises every cost.

```text
Optimise foreground writes
    → reads or maintenance pay later

Optimise direct reads
    → updates perform more immediate work

Optimise space
    → compaction or encoding becomes more aggressive
```

Storage-engine design is primarily a choice about:

* **Where work occurs**
* **When work occurs**
* **Which workload pays for it**



### Practical comparison

| Goal                        | Likely design tendency             | Main resulting cost               |
| --------------------------- | ---------------------------------- | --------------------------------- |
| Fast point reads            | Mutable ordered index              | Random update I/O                 |
| Fast ingest                 | Buffered immutable runs            | Compaction and read amplification |
| Fast ranges                 | Ordered physical or logical layout | Sorting and maintenance           |
| Simple snapshots            | Copy-on-write or MVCC versions     | Version reclamation               |
| Low RAM                     | More direct disk structures        | Additional I/O                    |
| Low write amplification now | Defer propagation                  | Future maintenance debt           |
| High concurrency            | Immutable or append-only state     | More reconciliation               |

### Rules to retain

* Algorithm names are less informative than their buffering, mutability and ordering choices.
* Write cost can be immediate or deferred; it is rarely eliminated.
* Dense immutable storage reduces reserved-space overhead but stores historical versions.
* Ordered storage improves scans but must maintain that order somehow.
* Background work is part of foreground performance because it consumes the same CPU, memory and I/O.
* Hardware and lower software layers can reverse expected benefits.
* Evaluate storage engines using the real workload and full lifecycle, not isolated operations.

### Final Part I recall questions

1. Classify a design that appends unordered values but stores sorted pointers to them.
2. Which property most directly allows multiple readers to inspect files without write latches?
3. How can two engines both use immutable storage but have completely different range-query performance?
4. Why is low foreground write latency not evidence of low total write amplification?
5. A structure has fast reads, fast writes and large auxiliary indexes. Which RUM dimension is paying?
6. Compare how a B-Tree, an LSM Tree and Bitcask resolve an update to an existing key.
7. Which three questions should be asked before choosing a storage engine?
8. How can storage-layer semantics change when the underlying device changes from HDD to SSD?
9. Why should compaction, garbage collection and recovery be included in performance testing?
10. Design a storage approach for large values, frequent point reads and rare range scans. Which properties would you choose and what cost would you accept?

# Part II — Distributed Systems

## 1. Why Distributed Systems?

### Core idea

A distributed system consists of multiple independent machines that communicate over a network while appearing as one logical system.

Horizontal scaling adds machines instead of continually upgrading one machine.

| Scaling method | Approach                                      | Main limitation                               |
| -------------- | --------------------------------------------- | --------------------------------------------- |
| **Vertical**   | Add CPU, RAM or faster storage to one machine | Cost, hardware limits and single-node failure |
| **Horizontal** | Add networked machines                        | Coordination and failure complexity           |

Distributed databases use clusters to increase:

* Storage capacity
* Processing throughput
* Availability
* Fault tolerance



### Mental model

```text
Single-node system:
one state + one failure boundary

Distributed system:
many local states + unreliable communication
```

### Important trade-off

```text
More machines
    → more capacity and redundancy
    → more messages, coordination and failure modes
```

### Common mistakes

* Treating horizontal scaling as vertical scaling across several computers.
* Assuming adding replicas automatically improves correctness.
* Ignoring network and coordination costs when estimating scalability.
* Treating all nodes as one shared-memory machine.

### Recall questions

1. Why can horizontal scaling improve capacity but reduce conceptual simplicity?
2. Which new failure boundaries appear after adding a second node?
3. Why does replication require a synchronisation protocol?

---

## 2. Basic Distributed-System Model

### Core idea

A distributed system contains:

* **Processes or nodes**
* Local state at each process
* Communication links
* Messages exchanged over those links
* Local physical or logical clocks

A process cannot directly inspect another process’s memory. It learns about remote state only through messages. 

### Mental model

```text
Process A                    Process B
[local state]                [local state]
      └────── messages ────────┘
```

### Logical vs physical clocks

| Clock              | Meaning                                |
| ------------------ | -------------------------------------- |
| **Logical clock**  | Counter representing event order       |
| **Physical clock** | Local approximation of real-world time |

### Why distribution is difficult

Messages may be:

* Delayed
* Reordered
* Duplicated
* Lost

Processes may:

* Pause
* Slow down
* Crash
* Produce incorrect results
* Become unreachable

### Distributed algorithm

A distributed algorithm defines:

* Participant roles
* Local states
* Messages
* State transitions
* Timing assumptions
* Delivery guarantees
* Failure assumptions

### Common mistakes

* Assuming a process can know another process’s current state.
* Treating a received message as evidence that the sender is still alive.
* Assuming message order equals send order.
* Omitting timing and failure assumptions from an algorithm description.

### Recall questions

1. Why is remote state always knowledge about the past?
2. What must an algorithm specify besides its normal execution steps?
3. Why can two correct processes temporarily hold different views of the system?

---

## 3. Purposes of Distributed Algorithms

| Purpose           | Goal                                            |
| ----------------- | ----------------------------------------------- |
| **Coordination**  | Direct or schedule work across participants     |
| **Cooperation**   | Complete work that depends on several processes |
| **Dissemination** | Spread information through the system           |
| **Consensus**     | Make participants agree on one value            |



### Key relationship

```text
Reliable communication
    → information dissemination
    → coordination
    → agreement and consensus
```

The later abstractions depend on guarantees supplied by the earlier ones.

### Recall questions

1. Why does consensus require communication but communication alone not guarantee consensus?
2. Is replication primarily dissemination, consensus or both?
3. Which purpose best describes a central task scheduler?

---

# Chapter 8: Introduction and Overview

## 4. Concurrent Execution

### Core idea

Once multiple execution streams can read and write the same state, their operations may interleave in several valid orders.

Example:

```text
Initial x = 1

Thread A: x = x + 2
Thread B: x = x × 2
```

Possible results include:

| Result | Interleaving                       |
| -----: | ---------------------------------- |
|    `2` | Multiplication overwrites addition |
|    `3` | Addition overwrites multiplication |
|    `4` | Multiplication completes first     |
|    `6` | Addition completes first           |



### Mental model

```text
Individual operations may be correct
        but
their interleaving may be incorrect
```

### Why consistency models matter

A consistency model constrains:

* Which operation orders are permitted
* When writes become visible
* Which states participants may observe

Stronger constraints produce fewer possible histories but generally require more coordination. 

### Common mistakes

* Treating a read-modify-write expression as one atomic operation.
* Testing only one execution order.
* Assuming thread scheduling is deterministic.
* Assuming a correct final state proves that no invalid intermediate state existed.

### Recall questions

1. Why can two individually correct operations produce four results?
2. Which synchronisation would force the result to be either `4` or `6`?
3. How does a consistency model reduce possible execution histories?

---

## 5. Concurrency vs Parallelism

| Concurrency                         | Parallelism                                |
| ----------------------------------- | ------------------------------------------ |
| Multiple operations are in progress | Multiple operations execute simultaneously |
| May use one processor               | Requires multiple execution resources      |
| Operations overlap in time          | Operations run at the same instant         |



### Mental model

* **Concurrency:** two queues sharing one coffee machine.
* **Parallelism:** two queues using two coffee machines.

### Important exception

Technical discussions often use “concurrency” broadly enough to include parallel execution.

### Recall questions

1. Can a single-core system execute tasks concurrently?
2. Can two operations be concurrent without being parallel?
3. Which concept describes simultaneous CPU execution?

---

## 6. Shared State and Remote Uncertainty

### Core idea

Placing shared state in a database does not make distributed participants synchronised.

Every access still depends on:

```text
request transmission
    + queueing
    + remote processing
    + response transmission
```

A missing response does not reveal whether:

* The request was lost.
* The server is overloaded.
* The server crashed.
* The response was lost.
* Processing is merely slow.



### Mental model

```text
No response
    ≠ proof of failure
    ≠ proof of nonexecution
```

### Fault tolerance

Fault tolerance describes whether a system can continue functioning correctly despite component failures.

Redundancy removes a single point of failure but introduces a new problem:

```text
Multiple copies
    → copies may diverge
    → synchronisation required
```

### Common mistakes

* Retrying because no response was received while assuming the first attempt failed.
* Treating a timeout as proof of a crash.
* Adding a backup without defining replication semantics.
* Assuming all replicas instantly contain the same state.

### Recall questions

1. Why is “no response” an ambiguous observation?
2. What new correctness problem appears when a backup is introduced?
3. How can retries duplicate a successfully executed operation?

---

# Distributed-Computing Fallacies

## 7. The Network Is Not Reliable

### Core idea

A successful connection does not guarantee continued communication.

Failures may occur:

* Before message transmission
* During transmission
* After delivery but before the response
* During response transmission



### Rule

Treat every remote request as having an uncertain result until the protocol establishes otherwise.

### Quick example

```text
Client sends payment request
Server processes payment
Response is lost
Client sees timeout
```

The client cannot determine from the timeout whether the payment occurred.

### Recall questions

1. Why does successful connection establishment not guarantee request completion?
2. Which side effects make blind retrying dangerous?

---

## 8. Latency Is Not Zero

### Core idea

Remote calls include:

* Serialization
* Network traversal
* Queueing
* Scheduling
* Processing
* Deserialization
* Return transport

Even a healthy remote call is substantially different from a local function call. 

### Cause and effect

```text
More dependent network round trips
    → greater latency
    → greater exposure to failure
```

### Common mistakes

* Calling a remote service once per loop iteration.
* Hiding a blocking remote operation behind a property or local-looking function.
* Ignoring tail latency while optimising average latency.
* Assuming colocated services have zero transport cost.

### Recall questions

1. Why should an API reveal that an operation is remote?
2. How can reducing round trips improve both speed and reliability?
3. Why is average network latency insufficient for timeout selection?

---

## 9. Bandwidth Is Not Infinite

### Core idea

Increasing the number, size or frequency of messages consumes finite network capacity.

The classic distributed-computing fallacies also include assumptions that:

* The network is secure.
* Topology does not change.
* Transport has no cost.
* One administrator controls the entire system.



### Mental model

```text
Network capacity is a shared finite resource,
not a transparent function-call channel.
```

### Common mistakes

* Broadcasting large state to every node unnecessarily.
* Designing protocols whose message volume grows quadratically.
* Ignoring serialization and transfer costs.
* Assuming service locations and routes remain fixed.

### Recall questions

1. How can adding nodes increase total network load?
2. Why is a full-cluster broadcast difficult to scale?
3. Which topology changes could invalidate cached routing information?

---

## 10. Processing Is Not Instantaneous

### Core idea

Delivery is only the beginning of remote work.

A request may wait in:

* Network buffers
* Operating-system queues
* Application queues
* Thread pools
* Storage queues

Nodes may also have different hardware, software versions and workloads. A multi-node request often completes at the speed of its slowest required participant. 

### Straggler effect

```text
Wait for A, B and C
A = 10 ms
B = 12 ms
C = 900 ms

Total completion ≈ 900 ms
```

### Common mistakes

* Attributing all delay to the network.
* Assuming equal processing rates across replicas.
* Waiting for every replica when the protocol requires only a quorum.
* Ignoring queueing time in service latency.

### Recall questions

1. Why is a parallel request often limited by its slowest participant?
2. How can heterogeneous nodes complicate timeout configuration?
3. When can quorum completion reduce straggler impact?

---

## 11. Queues and Backpressure

### Core idea

A queue decouples request receipt from processing, but it does not increase the consumer’s processing rate.

```text
Arrival rate > processing rate
    → queue grows
    → waiting time grows
    → memory fills
    → failure
```

**Backpressure** slows producers when consumers cannot keep up. 

### Queue purposes

| Purpose              | Effect                                      |
| -------------------- | ------------------------------------------- |
| **Decoupling**       | Receipt and processing occur independently  |
| **Pipelining**       | Different stages process different requests |
| **Burst absorption** | Short load spikes do not immediately fail   |



### Important rule

```text
Larger queue
    → more burst tolerance
    but not
    → greater steady-state throughput
```

### Trade-off

| Small queue              | Large queue             |
| ------------------------ | ----------------------- |
| Fast overload detection  | Better burst absorption |
| More immediate rejection | Higher waiting latency  |
| Lower memory use         | Greater memory use      |
| Stronger backpressure    | More hidden overload    |

### Common mistakes

* Treating queue growth as increased throughput.
* Using unbounded queues.
* Adding workers without checking the real bottleneck.
* Retrying rejected work immediately and recreating the load.
* Measuring service time without queue time.

### Recall questions

1. Why does increasing queue capacity not fix sustained overload?
2. When is queueing preferable to immediate rejection?
3. What feedback should producers receive when consumers are saturated?
4. How can an unbounded queue cause a cascading failure?

---

## 12. Clocks and Time

### Core idea

Physical clocks on different machines do not remain perfectly synchronised.

They may:

* Drift at different rates
* Jump after clock correction
* Report different times for the same event
* Move backward unless a monotonic source is used

Therefore, ordinary timestamps are unsafe as a universal event-ordering mechanism. 

### Mental model

```text
Timestamp from node A < timestamp from node B
does not necessarily mean
event A happened first
```

### Physical vs monotonic time

| Physical clock                       | Monotonic clock                   |
| ------------------------------------ | --------------------------------- |
| Represents date and wall time        | Measures elapsed local time       |
| Can be corrected or move backward    | Should not move backward          |
| Useful for logs and external meaning | Useful for durations and timeouts |

### Important exception

Some systems use specialised clocks with explicit uncertainty bounds to establish stronger ordering guarantees.

### Common mistakes

* Ordering distributed writes solely by local wall-clock timestamps.
* Calculating durations using a clock that may move backward.
* Ignoring uncertainty in source-generated event times.
* Assuming clock synchronisation eliminates network delay.

### Recall questions

1. Why can two correctly functioning machines disagree about the current time?
2. Which clock should normally measure a timeout?
3. What does a timestamp uncertainty bound represent?
4. Why is clock synchronisation insufficient for proving causality?

---

## 13. State Consistency Is Not Automatic

### Core idea

Replicas may temporarily disagree not only about user data, but also about:

* Schema
* Membership
* Partition ownership
* Configuration
* Routing information



### Cause and effect

```text
Different schema views
    → one node encodes data one way
    → another decodes it differently
    → corruption or incorrect results
```

Different membership views can cause nodes to:

* Route a key to the wrong owner.
* Miss data that exists elsewhere.
* Write multiple conflicting copies.
* Report false absence.



### Rule

Metadata consistency is part of data correctness.

### Common mistakes

* Making data strongly consistent while letting schema changes propagate unsafely.
* Treating cluster membership as harmless cached metadata.
* Deploying incompatible software versions without protocol negotiation.
* Assuming every node agrees on partition ownership.

### Recall questions

1. How can schema disagreement corrupt otherwise valid data?
2. Why does membership consistency affect routing correctness?
3. Which metadata changes require coordinated publication?

---

## 14. Local and Remote APIs

### Core idea

A remote iterator or function has fundamentally different semantics from a local one.

It may involve:

* Pagination
* Network retries
* Data reconciliation
* Snapshot changes
* Partial results
* Remote failures
* Resource lifetimes



### Rule

Useful abstractions should hide implementation details without hiding important semantic differences.

### Common mistakes

* Making a network call appear as an inexpensive local property access.
* Hiding retry and consistency behaviour completely.
* Offering no cancellation, timeout or observability controls.
* Assuming iteration over remote data uses one stable local snapshot.

### Recall questions

1. Which controls should a remote API expose that a local API may not need?
2. Why can transparent retry change operation semantics?
3. How does remote pagination affect consistency?

---

# Failure Behaviour

## 15. Partial Failures

### Core idea

A distributed system can be partly operational:

* Some processes work.
* Some processes crash.
* Some links work in only one direction.
* Some network groups cannot communicate.
* Different observers reach different conclusions.

 

### Network partition

A network partition divides participants into groups that cannot communicate but may continue operating independently.

```text
Group A: nodes 1, 2
        ✕
Group B: nodes 3, 4
```

### Why partitions are dangerous

Both groups may:

* Accept writes
* Elect leaders
* Allocate the same resource
* Produce contradictory results

### Asymmetric failure

```text
A → B works
B → A fails
```

A and B can therefore form different opinions about each other.

### Common mistakes

* Treating failure as a system-wide binary state.
* Assuming all participants detect a partition simultaneously.
* Assuming a failed request means the remote operation did not execute.
* Designing only for complete cluster failure.

### Recall questions

1. Why are partitions more dangerous than occasional packet loss?
2. How can asymmetric communication create conflicting membership views?
3. What decisions should a partitioned minority be allowed to make?

---

## 16. Failure Testing

### Core idea

Distributed failures are difficult to enumerate mentally. Systems should be tested by intentionally introducing:

* Latency
* Packet loss
* Bandwidth limits
* Network partitions
* Clock divergence
* Process crashes
* Storage corruption
* Filesystem errors
* Unequal processing speeds



### Mental model

```text
Fault tolerance that has not been fault-tested
    is an assumption.
```

### Common mistakes

* Testing only graceful shutdowns.
* Testing one failure at a time when failures can overlap.
* Assuming staging timing resembles production.
* Verifying availability but not data correctness after recovery.
* Ignoring failures during recovery itself.

### Recall questions

1. Which failure injection tests an operation’s duplicate-handling logic?
2. Why should clock divergence be tested?
3. What should be verified after a node rejoins the cluster?

---

## 17. Cascading Failures

### Core idea

One failure can increase load on healthy components and make them fail too.

```text
Node fails
    ↓
Traffic moves to remaining nodes
    ↓
Remaining nodes overload
    ↓
More nodes fail
```

A recovering node can also trigger a cascade when peers flood it with catch-up traffic before it is ready. 

### Protection mechanisms

| Mechanism           | Purpose                                        |
| ------------------- | ---------------------------------------------- |
| **Circuit breaker** | Temporarily stop calls to a failing dependency |
| **Backoff**         | Increase delay between retries                 |
| **Jitter**          | Randomise retries across clients               |
| **Validation**      | Prevent corrupted data from spreading          |
| **Coordination**    | Schedule work according to available capacity  |



### Backoff with jitter

```text
Retry delay:
base × growth_factor + random_jitter
```

Jitter prevents all clients from retrying simultaneously after the same delay.

### Common mistakes

* Retrying immediately in a tight loop.
* Using identical retry schedules across all clients.
* Sending unrestricted recovery traffic to a newly restarted node.
* Replicating corrupted bytes without checksums.
* Leaving circuit breakers open or closed permanently.

### Recall questions

1. How can retries turn one slow server into a full outage?
2. What problem does jitter solve that backoff alone does not?
3. Why should catch-up traffic be rate-limited?
4. How can corruption propagate through replication?

---

# Communication Abstractions

## 18. Fair-Loss Links

### Core idea

A fair-loss link may lose or delay messages, but it obeys three properties:

| Property               | Guarantee                                                                        |
| ---------------------- | -------------------------------------------------------------------------------- |
| **Fair loss**          | Infinite retransmission between correct processes eventually produces deliveries |
| **Finite duplication** | One sent message is not delivered infinitely many times                          |
| **No creation**        | Unsent messages are never invented                                               |

 

### Sender uncertainty

After sending a message, the sender cannot distinguish:

* Delayed
* Lost
* Delivered

### Mental model

A fair-loss link resembles a basic unreliable datagram channel.

### Common mistakes

* Treating message transmission as delivery.
* Assuming absence of an error means successful delivery.
* Assuming messages cannot be duplicated.
* Assuming delivery order is preserved.

### Recall questions

1. What does fair loss guarantee only under repeated sending?
2. Which property prevents spontaneous messages?
3. Can the sender know whether one unacknowledged message was delivered?

---

## 19. Acknowledgments

### Core idea

The recipient sends an acknowledgment containing the original message’s identifier.

```text
A → B: M(42)
B → A: ACK(42)
```

Unique identifiers allow the sender and receiver to associate acknowledgments with specific messages. 

### Guarantee

After receiving `ACK(42)`, A knows that B received `M(42)`.

### Limitation

Before the acknowledgment arrives, A still does not know whether:

* The message was lost.
* The message was delivered.
* The acknowledgment was lost.
* B crashed after receiving it.



### Common mistakes

* Treating a missing acknowledgment as evidence of nondelivery.
* Reusing identifiers before previous messages are no longer relevant.
* Acknowledging before reaching the protocol’s required durable state.
* Failing to authenticate or validate acknowledgments where needed.

### Recall questions

1. What fact does an acknowledgment establish?
2. What does it fail to establish when it is absent?
3. When should a database acknowledge receipt versus durable processing?

---

## 20. Retransmission and Stubborn Links

### Core idea

A stubborn link repeatedly sends an unacknowledged message.

```text
send M
wait
send M
wait
send M
...
```

Assuming communication eventually becomes possible, the message cannot remain permanently lost. 

### Cost

Retransmission creates duplicate delivery.

The sender cannot tell whether an earlier attempt:

* Failed
* Is delayed
* Was already processed

### Common mistake

Assuming retries only repeat failed operations.

### Recall questions

1. Which failure does indefinite retransmission overcome?
2. Why does it create duplicate-processing risk?
3. Why should practical retries include limits or backoff?

---

## 21. Idempotence

### Core idea

An operation is idempotent when repeating it produces the same logical result as executing it once.

```text
set status = "closed"       → usually idempotent
increment balance by £10    → not idempotent
charge card £10             → not idempotent
```

 

### Mental model

```text
f(f(x)) = f(x)
```

### Deduplication

When the underlying operation is not idempotent, the receiver can create equivalent behaviour by remembering processed message IDs.

```text
receive request ID 123
    ↓
already processed?
├─ Yes → return stored result
└─ No  → execute and record result
```

### Important rule

The operation and the recording of its deduplication result must be coordinated atomically.

Otherwise:

```text
execute side effect
    ↓ crash
record request ID never written
    ↓ retry executes side effect again
```

### Common mistakes

* Calling every `PUT` idempotent without considering side effects.
* Deduplicating only in volatile memory.
* Forgetting identifier expiry and retention.
* Recording the message ID before the operation succeeds.
* Generating a new request ID for every retry.

### Recall questions

1. Why is “set value to 10” idempotent but “add 10” is not?
2. What must a retry reuse for deduplication to work?
3. Why must result recording and execution be atomic?
4. How long should processed request identifiers be retained?

---

## 22. Ordering and Deduplication

### Core idea

Sequence numbers allow the receiver to restore FIFO order and reject duplicates.

The receiver can track:

* `n_consecutive`: highest sequence number received without gaps
* `n_processed`: highest sequence number already processed



### Reordering example

```text
Received: 3, 5
Missing:  4

Hold 5 in buffer
Receive 4
Process 4, then 5
```

### Deduplication rule

Messages with sequence numbers already processed are discarded or answered from cached results.

### Trade-off

| Larger reordering buffer          | Smaller buffer                    |
| --------------------------------- | --------------------------------- |
| Tolerates more reordering         | Lower memory use                  |
| Waits longer for missing messages | More gaps may cause failure       |
| Better FIFO reconstruction        | May require retransmission sooner |

### Common mistakes

* Processing sequence `5` before missing sequence `4` when FIFO is required.
* Treating a duplicate as a new logical request.
* Allowing sequence counters to wrap without a protocol.
* Buffering forever when a missing message will never arrive.

### Recall questions

1. What distinction exists between received and processed sequence numbers?
2. Why can a receiver safely discard an already processed sequence?
3. Which timeout or recovery policy is needed for permanent gaps?

---

## 23. Perfect Links

### Core idea

A perfect-link abstraction provides:

1. **Reliable delivery**
2. **No duplication**
3. **No creation**



### How it is constructed

```text
fair-loss link
    + retransmission
    + acknowledgments
    + sequence numbers
    + reordering
    + deduplication
    =
perfect-link abstraction
```

### Important exception

“Perfect” describes the abstraction under its assumptions. It does not guarantee:

* That processes never crash
* Infinite availability
* Delivery across every future session
* Application-level durable processing

### Common mistakes

* Equating transport delivery with application success.
* Assuming a closed connection proves all prior data was processed.
* Treating protocol-level ordering as global distributed ordering.
* Ignoring session boundaries.

### Recall questions

1. Which mechanism supplies reliable delivery?
2. Which mechanism supplies no duplication?
3. Why can a perfect transport link still fail to produce a successful transaction?

---

# Delivery Semantics

## 24. At-Most-Once and At-Least-Once

| Semantic             | Behaviour                        | Risk                                |
| -------------------- | -------------------------------- | ----------------------------------- |
| **At-most-once**     | Send without indefinite retry    | Message may never be processed      |
| **At-least-once**    | Retry until acknowledgment       | Message may be processed repeatedly |
| **Effectively once** | Retry plus durable deduplication | State and retention complexity      |



### Mental model

```text
At-most-once:
possible loss, no protocol-generated repeat

At-least-once:
possible repeat, eventual delivery under assumptions
```

### Common mistakes

* Describing at-least-once delivery as exactly-once processing.
* Using at-most-once for operations that cannot tolerate loss.
* Using at-least-once for non-idempotent operations without deduplication.
* Ignoring acknowledgement durability.

### Recall questions

1. Which semantic favours avoiding duplicates?
2. Which semantic favours avoiding loss?
3. How can an application turn at-least-once delivery into effectively-once processing?

---

## 25. Exactly-Once Processing

### Core idea

Physical transmission may occur more than once.

The practical objective is usually:

> Process the logical operation once and make duplicate deliveries harmless.



### Delivery vs processing

| Stage        | Meaning                                     |
| ------------ | ------------------------------------------- |
| Delivered    | Bytes reached the recipient                 |
| Processed    | Recipient applied the operation             |
| Persisted    | Result survives failure                     |
| Acknowledged | Sender learned the required stage completed |

### Fundamental uncertainty

If the recipient processes an operation but its acknowledgment is lost, the sender cannot determine whether retrying will duplicate the effect.

### Practical illusion

Effectively-once processing normally requires:

* Stable request identifiers
* Durable deduplication records
* Atomic state update and deduplication
* Stored previous results
* Defined retry and retention windows

### Common mistakes

* Claiming exactly-once behaviour without defining the boundary.
* Acknowledging before durable application when durability is required.
* Deduplicating transport packets but not business operations.
* Forgetting duplicate requests after service restart.

### Recall questions

1. Why is exactly-once transmission different from exactly-once processing?
2. Which state must survive a recipient restart?
3. At what point should an acknowledgment be emitted for a durable operation?
4. Why must an exactly-once claim specify its scope?

---

# Agreement Limits

## 26. Two Generals’ Problem

### Core idea

Two parties cannot establish certain mutual agreement over an unreliable asynchronous channel using a finite chain of acknowledgments.

```text
A sends message
B sends acknowledgment
A acknowledges the acknowledgment
B acknowledges that acknowledgment
...
```

There is always a final message whose successful delivery is unknown to its sender.  

### Mental model

```text
Knowing X
    ≠
knowing that the other party knows X
```

### Why another acknowledgment does not solve it

Every confirmation creates a new uncertainty:

> Did the confirmation itself arrive?

### Important assumption

Communication is asynchronous and messages may be lost. There is no guaranteed response-time bound.

### Common mistakes

* Believing one additional acknowledgment creates common knowledge.
* Treating local commit as proof that every participant committed.
* Assuming silence means disagreement.
* Applying the result as proof that useful coordination is impossible.

### Recall questions

1. Why does every additional acknowledgment create another uncertainty?
2. What form of knowledge do the generals lack?
3. Which timing or reliability assumptions would change the problem?

---

## 27. Consensus Properties

### Core idea

A consensus protocol makes multiple processes decide one value.

A correct protocol requires:

| Property        | Requirement                                          |
| --------------- | ---------------------------------------------------- |
| **Agreement**   | All deciding correct processes choose the same value |
| **Validity**    | The chosen value was proposed by a participant       |
| **Termination** | Correct processes eventually decide                  |



### Safety vs liveness relationship

* Agreement and validity are primarily **safety** properties.
* Termination is a **liveness** property.

### Common mistakes

* Calling identical default output consensus when no proposal matters.
* Considering a protocol correct when only some healthy nodes decide.
* Sacrificing agreement merely to guarantee fast termination.
* Assuming all participants must know every proposal.

### Recall questions

1. Which property prevents two different decisions?
2. Which property prevents inventing an unrelated value?
3. Which property requires eventual progress?
4. Can a protocol preserve agreement while failing termination?

---

## 28. FLP Impossibility

### Core idea

In a completely asynchronous system, no deterministic consensus protocol can guarantee termination in bounded time while tolerating even one unannounced crash.

The system cannot reliably distinguish:

```text
process crashed
from
process is extremely slow
```



### What FLP does mean

Under full asynchrony:

* Message delay has no known upper bound.
* Process speed has no known lower bound.
* Timeouts cannot prove failure.
* Some execution can prevent consensus from terminating.

### What FLP does **not** mean

* Consensus never succeeds.
* Consensus is unusable in practice.
* Agreement and validity cannot be preserved.
* Randomisation or additional assumptions cannot help.

### Common mistakes

* Summarising FLP as “consensus is impossible.”
* Assuming a timeout converts suspicion into proof.
* Ignoring the distinction between possible termination and guaranteed termination.
* Assuming practical systems are completely asynchronous forever.

### Recall questions

1. Which consensus property does FLP prevent from being guaranteed?
2. Why is one slow process indistinguishable from one crashed process?
3. How can practical consensus systems make progress despite FLP?
4. Does FLP say that a run can never reach agreement?

---

# Timing Models

## 29. Asynchronous Systems

### Assumptions

* No upper bound on message delay
* No upper bound on processing time
* No reliable relative process-speed bound
* No dependable timeout for proving failure



### Benefit

Algorithms make few timing assumptions and tolerate unpredictable environments.

### Cost

Failure detection and guaranteed consensus termination become difficult or impossible.

---

## 30. Synchronous Systems

### Assumptions

* Message delays are bounded.
* Processing times are bounded.
* Processes progress at comparable rates.
* Clock differences remain within known limits.



### Why it matters

Known bounds permit:

* Timeouts
* Failure detection
* Leader election
* Bounded response expectations
* Simpler consensus reasoning

### Risk

If the real system violates the assumed bounds, the algorithm may:

* Falsely suspect a live process.
* Elect competing leaders.
* Trigger unnecessary recovery.
* Lose availability.

---

## 31. Partial Synchrony

### Core idea

Most practical systems are partly synchronous:

* Timing bounds hold eventually or most of the time.
* Exact bounds may initially be unknown.
* Temporary periods may behave asynchronously.



### Mental model

```text
Normal period:
timing behaves predictably

Disturbance:
delays become unbounded from the protocol’s perspective

Recovery:
predictable bounds return
```

### Comparison

| Model                 | Timing guarantees                 | Main effect                                          |
| --------------------- | --------------------------------- | ---------------------------------------------------- |
| Asynchronous          | None                              | Strong assumptions avoided; weak progress guarantees |
| Synchronous           | Known fixed bounds                | Easier detection and bounded progress                |
| Partially synchronous | Bounds eventually or usually hold | Practical foundation for consensus                   |

### Common mistakes

* Designing for perfect synchrony because normal latency is stable.
* Treating timeout expiry as mathematical proof.
* Choosing timeouts from averages rather than tail behaviour.
* Ignoring prolonged asynchronous periods.

### Recall questions

1. Why do practical systems commonly use partial synchrony?
2. What may happen when synchronous assumptions temporarily fail?
3. How does eventual timing stability enable consensus progress?

---

# Failure Models

## 32. Why Failure Models Matter

### Core idea

An algorithm is correct only relative to the failures it assumes.

Possible assumptions include:

* Processes stop permanently.
* Processes recover later.
* Messages are omitted.
* Processes return arbitrary or malicious results.



### Rule

```text
Fault tolerance claim
    must specify
fault type + number of tolerated faults
```

### Common mistakes

* Saying a system is “fault tolerant” without naming the fault model.
* Applying a crash-tolerant protocol in an adversarial environment.
* Assuming recovery does not require persistent state.
* Treating network partitions as only process crashes.

### Recall questions

1. Why is “tolerates one failure” incomplete?
2. Which failure types require stronger protocols than crash faults?
3. How does the assumed fault model affect redundancy requirements?

---

## 33. Crash-Stop Faults

### Core idea

A crash-stop process stops taking algorithm steps and does not return during that algorithm instance.

```text
running → crashed
```

The algorithm must not rely on its recovery for correctness or progress. 

### Important exception

The physical machine may later restart, but the current algorithm treats the old participant as permanently stopped.

### Common mistakes

* Allowing a restarted process to resume using stale volatile state.
* Assuming crash-stop means hardware can never restart.
* Counting a restarted identity as the original participant without protocol support.

### Recall questions

1. Can a physical process restart under the crash-stop model?
2. Why should the current algorithm not depend on that restart?
3. What state may a replacement process need before rejoining?

---

## 34. Crash-Recovery Faults

### Core idea

A process may crash and later resume participation.

This requires:

* Durable state
* Recovery procedures
* Stable identity rules
* Handling every possible persisted recovery point



### Mental model

```text
running
    ↓ crash
volatile state lost
    ↓ restart
recover durable state
    ↓
rejoin safely
```

### Risks

A recovering process may have:

* Old membership information
* An incomplete log
* A stale term or epoch
* Partially applied operations
* Messages from an earlier execution

### Common mistakes

* Restarting with empty state as though the process were new.
* Replaying non-idempotent actions without deduplication.
* Rejoining before catching up.
* Reusing stale leadership authority.

### Recall questions

1. Why does crash recovery require durable state?
2. How can a recovering leader threaten safety?
3. Which information must be refreshed before rejoining?

---

## 35. Omission Faults

### Core idea

An omission fault occurs when a required step or message is missing or invisible.

Examples:

* Send omission
* Receive omission
* Lost message
* Network partition
* Full queue
* Intermittent pause
* Extremely delayed response



### Relationship to crash faults

A complete omission of all incoming and outgoing communication can appear identical to a crashed process.

### Mental model

```text
The process may still run,
but some required effects never become visible.
```

### Common mistakes

* Assuming a responsive process cannot suffer omission faults.
* Treating an overloaded queue as merely a performance issue.
* Ignoring receive-side omissions.
* Assuming all messages across a partition fail symmetrically.

### Recall questions

1. How can a live process appear crashed?
2. Which omission fault can be caused by a full queue?
3. Why can a network partition be modelled as message omissions?

---

## 36. Byzantine or Arbitrary Faults

### Core idea

A Byzantine process may continue participating while violating the protocol arbitrarily.

It may:

* Send contradictory messages
* Invent values
* Lie about state
* Collude with other processes
* Execute incompatible software
* Act maliciously



### Mental model

| Crash fault                | Byzantine fault                       |
| -------------------------- | ------------------------------------- |
| Participant becomes silent | Participant may actively mislead      |
| Absence of information     | Potentially false information         |
| Simpler redundancy         | Cross-validation and stronger quorums |

### Important exception

Not every Byzantine fault is malicious. Bugs or incompatible versions can create arbitrary protocol behaviour.

### Common mistakes

* Trusting every signed message as semantically correct.
* Using crash-fault quorum sizes against Byzantine participants.
* Assuming internal services cannot behave arbitrarily.
* Failing to validate values received from replicas.

### Recall questions

1. Why is a lying participant harder to tolerate than a silent one?
2. How can software-version mismatch resemble a Byzantine fault?
3. Why does Byzantine tolerance require more redundancy?

---

## 37. Handling Failures

### Core idea

Failures can often be masked through process groups and redundancy.

```text
one participant fails
    ↓
another participant continues service
```

This usually creates a slower failure path, but preserves user-visible operation. Most algorithms in the following chapters assume crash failures rather than arbitrary Byzantine behaviour. 

### Prevention vs tolerance

| Prevention             | Tolerance          |
| ---------------------- | ------------------ |
| Testing                | Redundancy         |
| Code review            | Retry and recovery |
| Validation             | Failover           |
| Capacity planning      | Quorums            |
| Correct local ordering | Replication        |

Prevention reduces failure probability. Tolerance limits damage after failure occurs.

### Key relationships

```text
Redundancy without coordination
    → conflicting copies

Coordination without redundancy
    → single failure can halt progress

Redundancy + coordination
    → fault-tolerant distributed system
```

### Common mistakes

* Treating redundancy alone as fault tolerance.
* Designing only the normal execution path.
* Assuming failover has no performance cost.
* Testing failure detection without testing recovery.
* Claiming Byzantine tolerance for a crash-only design.

### Applied recall questions

1. A client times out after submitting a non-idempotent operation. What must it know before retrying safely?
2. A node responds slowly during a network congestion event. Why can a failure detector not prove that it crashed?
3. Two partitions independently elect leaders. Which assumption or protocol property failed?
4. A system retries requests using exponential backoff, but thousands of clients still retry simultaneously. What is missing?
5. A message is transmitted twice but processed once. Which guarantee has been achieved?
6. A consensus protocol preserves agreement but remains blocked indefinitely. Which property failed?
7. A recovered node resumes using its old leadership term. Which failure-model concern does this demonstrate?
8. A replica sends syntactically valid but fabricated state. Is this a crash, omission or Byzantine fault?
9. Why does adding a larger queue often increase latency without increasing throughput?
10. Which timing model best describes a system where latency is usually bounded but occasionally becomes unpredictable?

# Chapter 9: Failure Detection

## 1. Failure Detection Is Suspicion, Not Proof

### Core idea

In an asynchronous system, a missing response cannot distinguish:

```text
process crashed
from
process is slow
from
message or response is delayed
```

A failure detector therefore forms a **local suspicion** about whether another process is reachable or functioning. 

### Mental model

```text
No heartbeat received
    ↓
Evidence of possible failure
    ≠
Proof of failure
```

### Why it matters

Continuing to contact a failed node:

* Increases latency
* Wastes resources
* Blocks algorithm progress
* May propagate overload

Excluding a live but slow node:

* Reduces available capacity
* May cause unnecessary failover
* Can split the cluster
* May threaten correctness if exclusion is unsafe

### Core trade-off

| Aggressive detection       | Conservative detection  |
| -------------------------- | ----------------------- |
| Detects failures quickly   | Fewer false suspicions  |
| More false positives       | Slower failover         |
| Better liveness response   | Higher request latency  |
| Risks excluding live nodes | Keeps dead nodes longer |

### Common mistakes

* Treating timeout expiry as proof of death.
* Using one fixed timeout across all network conditions.
* Assuming every observer sees the same failure.
* Confusing “suspected” with “crashed.”

### Recall questions

1. Why can a failure detector not perfectly distinguish a slow node from a crashed node?
2. Which failure-detector error harms availability?
3. Which error delays system progress?

---

## 2. Safety, Liveness and Completeness

### Liveness

A desired event eventually occurs.

For failure detection:

> A failed process is eventually identified.

### Safety

An undesired event never occurs.

For an ideal failure detector:

> A live process is never identified as failed.

### Completeness

Every relevant nonfaulty participant eventually notices an actual failure.

### Accuracy

The detector avoids incorrectly suspecting live processes.

### Efficiency

The detector identifies failures quickly and with acceptable resource cost.



### Relationship

```text
Short suspicion timeout
    → faster detection
    → lower accuracy

Long suspicion timeout
    → slower detection
    → higher accuracy
```

Perfect accuracy and immediate detection are not simultaneously achievable under asynchronous uncertainty.

### Important exception

Failure detectors may make mistakes and still be useful for consensus, provided their guarantees become sufficiently reliable under the algorithm’s assumptions.

### Common mistakes

* Equating completeness with accuracy.
* Assuming a detector that makes false positives is useless.
* Optimising detection speed without measuring unnecessary failovers.
* Evaluating detector correctness independently of the algorithm using it.

### Recall questions

1. Can a detector be complete but inaccurate?
2. Which property guarantees that a real failure is eventually noticed?
3. Why can consensus use an imperfect failure detector?

---

## 3. Pings and Heartbeats

### Core idea

Two common liveness signals are:

| Mechanism     | Direction                                              |
| ------------- | ------------------------------------------------------ |
| **Ping**      | Observer asks the remote node to respond               |
| **Heartbeat** | Monitored node periodically announces that it is alive |



### Ping model

```text
P1 → P2: PING
P2 → P1: ACK
```

Each process tracks:

* Last successful response time
* Current state: alive, suspected or failed
* Number of missed responses

### Fixed-deadline detector

```text
time since last heartbeat > timeout
    → mark node suspected
```

### Tuning relationship

| Frequent heartbeat        | Infrequent heartbeat   |
| ------------------------- | ---------------------- |
| Faster detection          | Lower message overhead |
| More network traffic      | Slower detection       |
| Sensitive to short delays | Tolerates brief pauses |

| Short timeout               | Long timeout          |
| --------------------------- | --------------------- |
| Fast suspicion              | Fewer false positives |
| Sensitive to latency spikes | Slow failure response |



### Common mistakes

* Making the timeout equal to the normal average latency.
* Ignoring scheduling pauses and garbage collection.
* Assuming a successful indirect response proves direct connectivity.
* Broadcasting heartbeats to every node in a large cluster.

### Recall questions

1. How do ping and heartbeat responsibilities differ?
2. Why should timeout configuration consider tail latency?
3. What costs increase as heartbeat frequency rises?

---

## 4. Timeout-Free Heartbeat Detection

### Core idea

A timeout-free detector can compare heartbeat **progress** instead of declaring failure after a fixed period.

Each process maintains counters for known participants.

Heartbeat messages contain:

* Unique message identifier
* Traversed process path
* Counter information



### Propagation

```text
P1 creates heartbeat [P1]
    ↓
P2 receives it
increments counters for path members
appends P2
    ↓
forwards [P1, P2]
```

A process stops forwarding when all known participants are already represented in the path.

### Mental model

Instead of asking:

> Has P3 missed a deadline?

the system asks:

> Is P3’s progress falling behind the progress observed for other processes?

### Benefit

Indirect paths can show that a process is alive even when one direct link is broken.

```text
P1 ✕ P3
P1 ↔ P2 ↔ P3

P1 learns about P3 through P2
```



### Limitation

Counter interpretation still requires a threshold for deciding when one process has fallen too far behind.

The algorithm avoids fixed delivery-time assumptions but does not eliminate judgment or false suspicion.

### Common mistakes

* Assuming timeout-free means assumption-free.
* Broadcasting duplicate heartbeat paths without identifiers.
* Treating a broken direct link as proof of node failure.
* Selecting progress thresholds without workload testing.

### Recall questions

1. How can heartbeat counters detect progress without a fixed timeout?
2. Why does indirect propagation improve reachability information?
3. What decision threshold remains necessary?

---

## 5. Outsourced Heartbeats

### Core idea

When a direct ping fails, the observer asks other nodes to test the suspected process.

```text
P1 → P2: direct ping fails

P1 → P3: ping P2
P1 → P4: ping P2

P3/P4 → P2
    ↓
report result to P1
```

This technique is used in SWIM-style membership protocols. 

### Why it works

A direct failure may indicate:

* P2 is down.
* P1’s outgoing link is broken.
* P2’s return path to P1 is broken.
* A temporary local network problem exists.

Indirect probes add perspectives from other network locations.

### Benefits

* Distinguishes node failure from one-link failure more accurately.
* Avoids full-cluster broadcast.
* Runs indirect probes in parallel.
* Requires awareness of only a subset of peers.



### Trade-off

```text
More indirect probes
    → more evidence
    → greater message overhead
```

### Common mistakes

* Treating one failed indirect probe as conclusive.
* Selecting all helpers from the same network region.
* Allowing probe storms during a widespread outage.
* Ignoring asymmetric communication.

### Recall questions

1. What ambiguity does an indirect ping help resolve?
2. Why should helper nodes be selected from different network perspectives?
3. How does outsourced probing avoid full broadcast?

---

## 6. Phi-Accrual Failure Detector

### Core idea

A phi-accrual detector produces a continuous **suspicion score** rather than a binary up/down result.

```text
Low φ  → heartbeat timing looks normal
High φ → current delay is increasingly unusual
```

It adapts to observed heartbeat timing rather than relying only on one fixed deadline. 

### Three subsystems

| Subsystem          | Responsibility                                 |
| ------------------ | ---------------------------------------------- |
| **Monitoring**     | Collect heartbeat arrival samples              |
| **Interpretation** | Convert current delay into suspicion level     |
| **Action**         | Trigger behaviour after a configured threshold |

### How it works

1. Store recent heartbeat inter-arrival times in a fixed-size sliding window.
2. Discard old samples as new samples arrive.
3. Estimate the distribution’s mean and variance.
4. Measure time since the latest heartbeat.
5. Calculate how unlikely that delay is.
6. Produce suspicion score `φ`.
7. Compare `φ` with an application-selected threshold.



### Mental model

```text
Expected heartbeat interval: about 1 second

Current delay:
1.1 s → unsurprising
2.0 s → somewhat suspicious
8.0 s → highly suspicious
```

### Why it matters

Different consumers can use different thresholds:

| Consumer          | Possible policy                     |
| ----------------- | ----------------------------------- |
| Request router    | Avoid node at moderate suspicion    |
| Replica manager   | Replace node only at high suspicion |
| Monitoring system | Warn before removal                 |

### Important assumptions

* Recent samples reasonably predict near-future arrivals.
* The chosen statistical model approximates observed timing.
* The sliding window adapts quickly enough to network changes.

### Common mistakes

* Interpreting `φ` as a direct probability that the node is dead.
* Using the same threshold for every operation.
* Retaining stale samples after major network changes.
* Assuming heartbeat delays follow the chosen distribution perfectly.
* Treating a continuous score as absolute truth.

### Recall questions

1. Why is a continuous suspicion score more flexible than a binary timeout?
2. How does the sliding window adapt to changing network conditions?
3. Why might routing and replica replacement use different thresholds?
4. What happens when the sample distribution no longer represents current conditions?

---

## 7. Gossip-Based Failure Detection

### Core idea

Each node maintains and gossips a table containing:

* Member identifiers
* Heartbeat counters
* Last-update timestamps

Periodically, a node:

1. Increments its own counter.
2. Sends its table to a random peer.
3. Merges received counters with its local table.
4. Suspects members whose counters stop advancing.



### Mental model

```text
Direct knowledge
    +
knowledge received from peers
    =
aggregated cluster view
```

### Indirect propagation

If `P1` cannot communicate directly with `P3`, `P3`’s heartbeat may still reach `P1` through `P2`.

```text
P3 → P2 → P1
```

A genuine crash eventually causes the counter to stop advancing everywhere. 

### Benefits

* Avoids dependence on one observer.
* Tolerates some broken links.
* Spreads information gradually.
* Caps communication frequency per node.
* Produces an aggregated view.

### Costs

* Detection is not instantaneous.
* Nodes may temporarily disagree.
* More messages than direct pairwise observation.
* Stale gossip can delay accurate conclusions.

### Common mistakes

* Assuming all nodes learn a failure simultaneously.
* Replacing counters with unsynchronised wall-clock timestamps alone.
* Accepting an older heartbeat state over a newer one.
* Gossiping complete large membership tables without scaling controls.

### Recall questions

1. How can gossip distinguish a broken direct link from a crashed node?
2. Why do cluster views temporarily diverge?
3. What merge rule should apply to heartbeat counters?
4. How does gossip trade detection speed for scalability?

---

## 8. FUSE: Propagating Failure Through Silence

### Core idea

FUSE reverses the usual problem.

Instead of explicitly broadcasting that one process failed, processes propagate the failure by deliberately stopping their own responses.

```text
P2 stops responding
    ↓
P4 detects P2
    ↓
P4 stops responding
    ↓
others detect P4
    ↓
entire group enters failure state
```



### Mental model

```text
Individual failure
    → converted into
group-wide detectable silence
```

### Why it matters

Every connected group member eventually learns that the group can no longer satisfy its required communication condition.

This works across complex combinations of:

* Node failures
* Link failures
* Network partitions
* Disconnected process groups

### Benefit

Failure notification does not depend on successfully delivering a failure message through the same damaged network.

### Cost

A single inaccessible process or broken link may deliberately make the whole group unavailable.



### Key trade-off

| Group-failure propagation              | Independent continuation    |
| -------------------------------------- | --------------------------- |
| Uniform reaction                       | Better partial availability |
| Avoids inconsistent subgroup behaviour | Risks split-brain decisions |
| Failure spreads deliberately           | Fault remains isolated      |

### Common mistakes

* Assuming deliberate silence is another accidental failure.
* Using group-wide shutdown where partial service is safe.
* Defining groups too broadly.
* Ignoring the availability cost of converting one link fault into group failure.

### Recall questions

1. Why can silence propagate more reliably than an explicit failure message?
2. What safety benefit comes from converting one failure into group failure?
3. When would this approach sacrifice too much availability?

---

## 9. Failure-Detector Comparison

| Detector           | Main signal               | Strength                            | Main weakness                   |
| ------------------ | ------------------------- | ----------------------------------- | ------------------------------- |
| Fixed timeout      | Missed heartbeat deadline | Simple                              | Poor adaptation                 |
| Heartbeat counters | Relative progress         | Avoids fixed deadline               | Hard threshold selection        |
| Outsourced probe   | Neighbour perspectives    | Distinguishes link and node failure | More messages                   |
| Phi-accrual        | Statistical suspicion     | Adaptive and tunable                | Model-dependent                 |
| Gossip             | Aggregated peer state     | Scalable dissemination              | Eventual convergence            |
| FUSE               | Deliberate silence        | Uniform group failure               | Sacrifices partial availability |

Failure detectors augment asynchronous systems by making an explicit trade-off between accuracy and progress. 

### Applied recall questions

1. A node experiences frequent stop-the-world pauses. Which detector is likely to outperform one fixed timeout?
2. Direct pings between two racks fail, but both racks can reach a third rack. Which technique can avoid false suspicion?
3. A protocol must prevent isolated subgroups from operating independently. Which detector’s core idea supports this?
4. Why can a detector that occasionally makes mistakes still enable consensus?
5. Which detector would be simplest for a small, stable internal network?

---

# Chapter 10: Leader Election

## 10. Why Use a Leader?

### Core idea

A leader, or coordinator, centralises decisions that would otherwise require repeated peer-to-peer synchronisation.

A leader may:

* Order messages
* Coordinate writes
* Assign work
* Manage reconfiguration
* Maintain global state
* Drive recovery



### Mental model

```text
Peer-to-peer:
every participant coordinates with many others

Leader-based:
participants coordinate mainly through one process
```

### Benefits

* Fewer message round trips
* Simpler ordering
* Stable coordination point
* Reduced state synchronisation
* Easier normal-case execution

### Costs

* Bottleneck risk
* Leader failure stalls progress
* Election overhead
* Split-brain risk
* Uneven load

### Election goals

| Property         | Requirement                                        |
| ---------------- | -------------------------------------------------- |
| **Liveness**     | An election eventually completes                   |
| **Availability** | A leader exists most of the time                   |
| **Safety**       | At most one valid leader exists for the same scope |

In practice, some algorithms temporarily permit competing leaders and resolve conflicts later. 

### Common mistakes

* Treating leadership as a permanent role.
* Assuming leader election automatically prevents split brain.
* Routing all data through one global leader regardless of scale.
* Failing to define what scope the leader controls.

### Recall questions

1. Which coordination costs does a stable leader reduce?
2. Why does leader failure primarily threaten liveness?
3. What property prevents two active leaders for the same resource?

---

## 11. Leader Election vs Distributed Locking

### Core idea

Both mechanisms grant special status to one process, but their semantics differ.

| Leader election                              | Distributed lock                      |
| -------------------------------------------- | ------------------------------------- |
| Participants must know the leader’s identity | Others need not know the owner        |
| Long-lived ownership is desirable            | Eventual release is required          |
| Leader coordinates ongoing work              | Lock protects a critical section      |
| Re-election occurs after failure             | Ownership rotates among contenders    |
| Preference may be stable                     | Permanent preference risks starvation |



### Mental model

* **Leader:** recognised coordinator.
* **Lock holder:** temporary exclusive user.

### Common mistakes

* Using a lock lease as proof of permanent leadership.
* Allowing a former leader to continue after losing ownership.
* Expecting fairness from a leader-election protocol.
* Expecting long-lived ownership from a fair lock.

### Recall questions

1. Why must every participant learn the leader’s identity?
2. Why is starvation undesirable for a lock but stable preference useful for leadership?
3. What additional mechanism prevents an expired leader from continuing writes?

---

## 12. Leader Scope and Bottlenecks

### Core idea

A system-wide leader can become a throughput and availability bottleneck.

A common solution is to partition responsibility:

```text
Partition A → Leader A
Partition B → Leader B
Partition C → Leader C
```

Each replica set independently elects a leader for its own data. 

### Trade-off

| One global leader   | Per-partition leaders        |
| ------------------- | ---------------------------- |
| Simple global order | Greater parallelism          |
| Central bottleneck  | Cross-partition coordination |
| One election domain | Many election domains        |
| Easy global state   | More distributed metadata    |

### Common mistakes

* Partitioning leadership without partitioning state.
* Assuming per-partition leaders remove cross-partition transactions.
* Allowing overlapping ownership between replica sets.
* Using one leader for unrelated actions that could be independent.

### Recall questions

1. How do per-partition leaders improve throughput?
2. Which operations still require coordination between leaders?
3. What metadata must identify each leader’s responsibility?

---

# Bully Algorithm

## 13. Rank-Based Election

### Core idea

Every process has a unique rank. The highest-ranked active process becomes leader.

### Election process

1. A process detects that no leader is available.
2. It sends election messages to all higher-ranked processes.
3. It waits for responses.
4. If none responds, it declares itself leader.
5. If higher-ranked processes respond, the highest responding process proceeds.
6. The winner announces leadership to lower-ranked processes.



### Mental model

```text
Can anyone with greater priority take over?
    ├─ Yes → defer to the highest responder
    └─ No  → become leader
```

### Example

Leader `6` fails.

```text
3 starts election
    ↓
4 and 5 respond
    ↓
5 is highest active rank
    ↓
5 announces leadership
```

### Benefits

* Simple
* Deterministic under a connected, stable membership
* Naturally selects the highest-priority active node

### Common mistakes

* Electing the initiator before waiting for higher ranks.
* Assigning duplicate ranks.
* Assuming absence of response proves higher nodes are dead.
* Failing to inform lower-ranked nodes of the result.

### Recall questions

1. Why does a low-ranked node contact only higher-ranked nodes?
2. Under ideal communication, which process wins?
3. What failure-detector assumption does the algorithm rely on?

---

## 14. Bully Algorithm Failure Modes

### Split brain

During a network partition, each connected subgroup may independently elect its own highest-ranked leader.

```text
Partition A → Leader 5
Partition B → Leader 6
```

Both believe they are valid. 

### Unstable high-ranked process

A high-ranked process may repeatedly:

1. Recover.
2. Win leadership.
3. Fail.
4. Trigger another election.

This can prevent stable progress.

### Possible mitigation

Election preference can include quality signals such as:

* Uptime
* Health
* Capacity
* Network position
* Failure history



### Common mistakes

* Equating highest identifier with best leader.
* Allowing a flapping node to immediately reclaim leadership.
* Using local connectivity as proof of cluster-wide authority.
* Allowing both partition leaders to perform irreversible actions.

### Recall questions

1. Why does ranking not prevent split brain?
2. How can a high-ranked unstable node damage liveness?
3. Which mechanism can prevent an old leader’s writes after a newer election?

---

## 15. Next-in-Line Failover

### Core idea

The current leader designates ranked successor candidates.

```text
Leader 6
Successors: [5, 4]
```

After leader failure:

1. Contact the highest-ranked successor.
2. If available, it becomes leader.
3. If unavailable, try the next successor.
4. Run a complete election only when necessary.



### Benefit

* Fewer election messages
* Faster failover
* Predictable successor
* Less disruption

### Cost

* Successor list may become stale.
* Leader and successors may fail together.
* Does not solve network-partition split brain.
* Successor placement may create correlated failure.

### Common mistakes

* Placing all successors in one failure domain.
* Treating the successor list as permanent.
* Promoting a successor without validating its state.
* Assuming designated succession replaces majority authority.

### Recall questions

1. Which election work does next-in-line failover avoid?
2. Why should successors be placed in different failure domains?
3. What happens when every listed successor is unavailable?

---

## 16. Candidate/Ordinary Optimisation

### Core idea

Processes are divided into:

* **Candidates:** eligible to lead
* **Ordinary nodes:** may initiate election but cannot become leader

An ordinary process:

1. Detects leader failure.
2. Queries candidates.
3. Selects the highest-ranked responding candidate.
4. Announces the result.



### Tiebreaker delay

A process-specific delay `δ` reduces simultaneous elections.

```text
Higher priority
    → smaller δ
    → starts election sooner
```

The delay is chosen to exceed normal message round-trip time.

### Benefits

* Fewer election participants
* Lower message volume
* Limits leadership to suitable nodes
* Reduces simultaneous election rounds

### Costs

* Candidate set becomes a special failure domain.
* Incorrect delay assumptions can cause concurrent elections.
* Ordinary nodes depend on candidate availability.
* Still vulnerable to partitioned election results.

### Common mistakes

* Making the candidate group too small.
* Selecting delays below realistic tail latency.
* Assuming delayed initiation eliminates all races.
* Failing to update candidate membership.

### Recall questions

1. Why does separating candidates reduce election traffic?
2. How does `δ` reduce simultaneous elections?
3. What happens when all candidate nodes are unavailable?

---

# Other Election Algorithms

## 17. Invitation Algorithm

### Core idea

Every process initially leads its own one-member group.

Group leaders invite other processes or groups to merge.

```text
{P1} + {P2} → {P1, P2}
{P3} + {P4} → {P3, P4}

then:

{P1, P2} + {P3, P4}
    → one larger group
```



### Merge optimisation

To minimise notifications, the leader of the larger group can remain leader.

Only members of the smaller group need to learn the new leader. 

### Benefits

* Forms groups incrementally.
* Avoids restarting election from scratch.
* Merges independently formed components.
* Can reduce message volume.

### Limitation

During partitions or incomplete merging, several groups and leaders may coexist.

### Common mistakes

* Assuming temporary multiple leaders violate this algorithm’s design.
* Not propagating the new leader to all affected members.
* Merging groups without a deterministic membership result.
* Failing to handle two simultaneous merge invitations.

### Recall questions

1. Why does every process begin as a leader?
2. Why is retaining the larger group’s leader message-efficient?
3. Under what condition can several leaders remain?

---

## 18. Ring Election Algorithm

### Core idea

Processes form a logical ring and know their successors.

```text
P1 → P2 → P3 → P4 → P5 → P1
```

When a leader fails:

1. A node starts an election message.
2. The message circulates around the ring.
3. Each live process adds its identifier or updates the current maximum.
4. Failed successors are skipped.
5. The initiator receives the completed result.
6. The winning identifier circulates around the ring.



### Optimisation

The message needs to retain only the current maximum rank:

```text
max(max_seen, current_node_rank)
```

Because `max` is associative and commutative, the complete live-node set is unnecessary when only the winner matters. 

### Benefits

* Structured message path
* No broadcast to every node at once
* Simple highest-rank selection
* Message size can remain constant

### Costs

* Requires correct ring topology.
* Election requires a full ring traversal.
* Failed nodes must be skipped.
* Ring partitions can elect separate leaders.
* Topology repair complicates operation.

### Common mistakes

* Treating one broken successor as a broken ring.
* Forgetting the second circulation announcing the winner.
* Retaining the full node set when only maximum rank is needed.
* Assuming ring traversal crosses network partitions.

### Recall questions

1. Why is one full traversal required before deciding?
2. How can the message retain only one identifier?
3. Why can a partitioned ring produce split brain?
4. What happens when the immediate successor is unavailable?

---

## 19. Leader Election and Consensus

### Core idea

Leader election requires participants to agree on a leader identity, making it closely related to consensus.

Simple election algorithms often fail to provide cluster-wide safety during partitions. Avoiding split brain generally requires majority-based agreement. 

### Local knowledge problem

A process may believe `P1` is leader even after another group has elected `P2`.

```text
Leadership changed globally
    but
local leader knowledge is stale
```

Failure detection and election must therefore work together. 

### Temporary competing leaders

Protocols such as Multi-Paxos and Raft may temporarily contain competing leader candidates.

Safety is preserved through:

* Quorum intersection
* Terms or epochs
* Rejection of stale authority
* Conflict resolution



### Fencing mental model

```text
Leader authority:
term 8 < term 9

Requests from term 8
    → rejected as stale
```

### Important distinction

Leadership often improves **liveness and performance**.

Quorums, terms and protocol rules preserve **safety**.

### Common mistakes

* Treating leader identity alone as proof of authority.
* Allowing leaders from old terms to modify shared state.
* Assuming election based only on local failure suspicion is safe.
* Concluding that temporary multiple leaders always imply corrupted state.

### Recall questions

1. Why does safe leader election resemble consensus?
2. How do terms or epochs limit stale leaders?
3. Why may an algorithm temporarily tolerate multiple leader candidates?
4. Which mechanism prevents two candidates from committing conflicting results?

---

## 20. Leader-Election Comparison

| Algorithm          | Main idea                           | Strength                 | Main weakness                    |
| ------------------ | ----------------------------------- | ------------------------ | -------------------------------- |
| Bully              | Highest active rank wins            | Simple                   | Split brain and rank instability |
| Next-in-line       | Preselected successors              | Fast failover            | Stale/correlated successors      |
| Candidate/ordinary | Restricted candidate set            | Lower traffic            | Candidate-set dependency         |
| Invitation         | Merge leader-led groups             | Incremental formation    | Multiple groups may persist      |
| Ring               | Circulate election through topology | Structured communication | Slow traversal and partitions    |
| Consensus-based    | Majority and terms                  | Strong safety            | More coordination                |

### Applied recall questions

1. Which algorithm best suits a small fixed membership where identifiers encode preference?
2. Which optimisation reduces election latency when the normal leader has predictable successors?
3. A former leader regains network access and continues writing. Which missing mechanism allowed this?
4. Why can every rank-based election algorithm still need quorum authority?
5. How does partitioning leadership improve scalability without solving cross-partition transactions?

---

# Chapter 11: Replication and Consistency

## 21. Why Replicate Data?

### Core idea

Replication stores multiple copies of data to provide:

* Redundancy
* Failover
* Higher read capacity
* Geographic proximity
* Datacentre-failure tolerance



### Mental model

```text
One copy
    → simple authority
    → single failure risk

Several copies
    → redundancy
    → synchronisation problem
```

### Failover models

| Primary/replica                | Multi-participant                      |
| ------------------------------ | -------------------------------------- |
| One source of truth            | Several replicas answer operations     |
| Replica promoted after failure | Reads/writes collect several responses |
| Explicit leadership change     | Quorums determine operation success    |
| Simpler ordering               | More conflict handling                 |

### Fundamental cost

Updating several copies atomically is an agreement problem and may require consensus-level coordination.

Systems often relax immediate agreement and allow temporary divergence to reduce cost.

### Common mistakes

* Treating replication as simple file copying.
* Assuming redundant data is automatically current.
* Promoting a replica without checking its freshness.
* Confusing replication factor with number of failure domains.

### Recall questions

1. Why does replication remove one failure mode while introducing another?
2. What must be checked before promoting a replica?
3. Why might synchronous replication increase write latency?

---

## 22. Replication Timeline

### Core idea

Replication involves three distinct events:

1. Client write
2. Replica update
3. Client read

These events may occur at different times and on different nodes. 

### Possible asynchronous sequence

```text
Client writes V2 to primary
    ↓
Primary acknowledges
    ↓
Client reads replica before propagation
    ↓
Replica returns V1
```

### Why it matters

Consistency models define which event orders and observations are permitted.

### Common mistakes

* Assuming acknowledgment means every replica has applied the write.
* Reading from arbitrary replicas without defining staleness.
* Using replication delay as the only source of inconsistency.
* Ignoring concurrent writes from other clients.

### Recall questions

1. How can a successful write be followed by a stale read?
2. Which guarantee would ensure the writer observes its own update?
3. When is replica-update completion part of write latency?

---

## 23. High Availability

### Core idea

A highly available system continues serving useful requests despite failures of some participants.

Replication helps by allowing healthy replicas to take over work. 

### Cause and effect

```text
Add redundancy
    → tolerate component loss
    → copies may diverge
    → recovery and reconciliation required
```

### Availability is not only uptime

A process may be running but unable to:

* Respond before a useful deadline
* Accept writes
* Reach required peers
* Serve correct results
* Keep up with workload

### Common mistakes

* Reporting process uptime as service availability.
* Counting an incorrect response as successful availability.
* Ignoring latency objectives.
* Assuming failover is instantaneous.

### Recall questions

1. Why can a running node still be unavailable?
2. Which recovery steps follow replica failover?
3. How can replication improve both availability and read latency?

---

# CAP

## 24. CAP Properties

### Consistency

CAP consistency means linearizable behaviour:

* Operations appear atomic.
* All clients observe one legal operation order.
* Real-time precedence is preserved.

### Availability

Every request to a nonfailing node eventually receives a successful response.

### Partition tolerance

The system continues operating according to its guarantees despite lost communication between groups of nodes.



### Core result

During a network partition, a system cannot guarantee both:

* Linearizable consistency
* Response availability from every nonfailing node



### Mental model

```text
Partition separates replicas

To remain consistent:
some side must reject or delay operations

To remain available:
both sides may respond
but can diverge
```

---

## 25. CP and AP Behaviour

| Choice during partition | Behaviour                                                          |
| ----------------------- | ------------------------------------------------------------------ |
| **CP**                  | Rejects or blocks operations that cannot be made safely consistent |
| **AP**                  | Continues responding but may expose or create divergent state      |



### CP example

A consensus group requires a majority.

A minority partition cannot commit writes.

```text
Safety retained
Availability reduced
```

### AP example

Every reachable replica continues accepting operations.

```text
Availability retained
Conflicts or stale reads possible
```

### Important rule

CAP describes behaviour **during partitions**, not the system’s entire normal operating mode.

### Common mistakes

* Describing a database permanently as only “CP” or “AP” without operation-level detail.
* Assuming AP means no consistency guarantees.
* Assuming CP means the system never responds during any failure.
* Treating one crashed node as identical to an active partitioned node.

### Recall questions

1. Why must a CP minority reject some operations?
2. What reconciliation work follows AP divergence?
3. Can one system make different CAP choices for different operations?

---

## 26. PACELC

### Core idea

PACELC extends CAP:

```text
If Partition:
    choose Availability or Consistency

Else:
    choose Latency or Consistency
```



### Mental model

Even when the network is healthy, strong consistency may require waiting for remote replicas.

```text
More coordination
    → stronger consistency
    → higher latency
```

### Examples

| Normal-operation preference | Result                                       |
| --------------------------- | -------------------------------------------- |
| Wait for multiple replicas  | Stronger consistency, higher latency         |
| Acknowledge locally         | Lower latency, possible temporary divergence |
| Local-region quorum         | Regional latency, weaker global immediacy    |
| Global quorum               | Global ordering, long-distance delay         |

### Common mistakes

* Discussing consistency trade-offs only during outages.
* Ignoring geographic round-trip latency.
* Assuming a system’s normal-case consistency has no availability effect.
* Treating “low latency” as one universal value across regions.

### Recall questions

1. Which trade-off exists even when no partition occurs?
2. Why does geo-replication make the latency-consistency choice more visible?
3. How can a local quorum differ from a global quorum?

---

## 27. Using CAP Carefully

### Partition tolerance is not a dial

Real distributed systems cannot guarantee that network partitions never occur.

The practical choice during a partition is how to trade consistency and availability. 

### CAP consistency vs ACID consistency

| CAP consistency                    | ACID consistency                        |
| ---------------------------------- | --------------------------------------- |
| Replica-visible operation ordering | Database invariant preservation         |
| Usually means linearizability      | Valid-state transition                  |
| Distributed history property       | Transaction/data-rule property          |
| Concerns copies and visibility     | Concerns constraints and business rules |



### CAP availability vs operational availability

| CAP availability                             | Practical high availability                      |
| -------------------------------------------- | ------------------------------------------------ |
| Every nonfailing node responds eventually    | Service meets useful success and latency targets |
| No latency bound                             | Usually measured by deadlines and SLOs           |
| Requires response from every nonfailing node | May route only to selected healthy nodes         |
| Theoretical property                         | Operational metric                               |



### Important distinctions

* A partitioned node may still run and serve requests.
* A crashed node cannot respond.
* Consistency problems can occur while every process is alive.
* AP systems may still provide useful consistency when enough replicas communicate.
* CAP is a reasoning model, not a complete database classification.

### Common mistakes

* Drawing CAP as three independently adjustable percentages.
* Using ACID consistency and CAP consistency interchangeably.
* Claiming partition tolerance was sacrificed.
* Concluding that an AP system always returns arbitrary values.
* Evaluating CAP without defining the operation and failure scenario.

### Recall questions

1. Why can a cluster violate consistency while every node remains alive?
2. How does CAP availability differ from an uptime SLO?
3. Why is “partition tolerance” not normally optional?
4. What information is missing from a simple CP/AP label?

---

## 28. Harvest and Yield

### Core idea

CAP uses strong binary definitions. Harvest and yield describe weaker, tunable service quality.

### Harvest

The fraction of complete data returned by a query.

```text
Expected rows: 100
Returned rows: 99

Harvest = 99%
```

### Yield

The fraction of attempted requests that complete successfully.

```text
Successful requests: 980
Attempted requests: 1,000

Yield = 98%
```



### Mental model

| Metric      | Question                        |
| ----------- | ------------------------------- |
| **Harvest** | How complete was this result?   |
| **Yield**   | How often did requests succeed? |

### Trade-off

A system can increase yield by returning partial results from currently available partitions.

```text
Some partitions unavailable
    ↓
Return available records
    ↓
higher yield
lower harvest
```



### Application-dependent policy

| Request                   | Possible policy           |
| ------------------------- | ------------------------- |
| Financial account balance | Require complete result   |
| Social feed               | Permit missing posts      |
| Product recommendations   | Return available subset   |
| Access-control decision   | Fail closed if incomplete |

### Common mistakes

* Returning incomplete data without marking it as partial.
* Treating a partial financial result as better than failure.
* Equating yield with server uptime.
* Applying one harvest policy to every query.
* Measuring request success without measuring result completeness.

### Recall questions

1. How can yield increase while harvest decreases?
2. Which applications should reject partial results?
3. Why is a busy but running node relevant to yield?
4. What metadata should accompany an incomplete response?

## 29. Shared Memory and Registers

### Core idea

A distributed database can present the illusion of shared memory.

A **register** is one independently readable and writable storage location.

```text
Distributed replicas
    ↓ hidden behind an interface
Logical register x
    ↓
read(x), write(x, value)
```

Each operation has:

* **Invocation:** operation begins.
* **Completion:** operation returns.

Two operations are:

* **Sequential** when one completes before the other begins.
* **Concurrent** when their time intervals overlap.

 

### Mental model

```text
Sequential:
[ operation A ][ operation B ]

Concurrent:
[ operation A       ]
       [ operation B ]
```

### Why it matters

Concurrent reads and writes may produce different results depending on the register’s consistency guarantees.

### Common mistakes

* Treating invocation time as completion time.
* Assuming two operations are sequential because one started first.
* Assuming a remote read or write occurs instantaneously.
* Ignoring operation overlap when analysing histories.

### Recall questions

1. Can an operation that starts second complete first and still be concurrent?
2. Why must consistency models describe overlapping operations?
3. Which events define an operation’s time interval?

---

## 30. Safe, Regular and Atomic Registers

### Core idea

Registers can provide different guarantees when reads overlap writes.

| Register    | Concurrent-read guarantee                                             |
| ----------- | --------------------------------------------------------------------- |
| **Safe**    | May return any value in the register’s value domain                   |
| **Regular** | Returns the previous completed value or the overlapping write’s value |
| **Atomic**  | Behaves as though every operation occurs at one instant               |



## Safe register

During a concurrent write, a read may return an arbitrary valid value.

```text
Binary register during write:
0, 1, 0, 1 ...
```

This model supplies almost no useful ordering guarantee during overlap.

## Regular register

A read overlapping a write may return:

* The value from the latest completed write
* The value being written by the overlapping write

It cannot return an unrelated arbitrary value.

## Atomic register

An atomic register has one conceptual transition point:

```text
Before transition → all reads see old value
After transition  → all reads see new value
```

Atomic registers provide linearizable behaviour.

### Key relationship

```text
Safe
  ↓ stronger
Regular
  ↓ stronger
Atomic
```

### Common mistakes

* Assuming regular registers make a new value visible simultaneously everywhere.
* Treating “atomic” as meaning physically instantaneous.
* Assuming a safe register returns only the old or new value.
* Confusing atomic-register behaviour with ACID transaction atomicity.

### Recall questions

1. Which values may a regular-register read return during an overlapping write?
2. What extra guarantee distinguishes an atomic register?
3. Why is a safe register usually impractical for database values?

---

# Consistency Models

## 31. Consistency as a Contract

### Core idea

A consistency model defines which operation histories clients are allowed to observe.

It acts as a contract between:

* Replicas implementing the system
* Clients reading and writing data



### Mental model

```text
All physically possible histories
    ↓ apply consistency rules
Permitted observable histories
```

### Two perspectives

| Perspective               | Main question                                         |
| ------------------------- | ----------------------------------------------------- |
| **State consistency**     | Which relationships between replica states are valid? |
| **Operation consistency** | Which read/write orders may clients observe?          |

### Synchronisation cost

Stronger consistency may require more:

* Network round trips
* Waiting
* CPU work
* Disk persistence
* Coordination
* Recovery metadata



### Trade-off

```text
Stronger ordering guarantees
    → fewer possible histories
    → easier application reasoning
    → higher coordination cost
```

### Common mistakes

* Assuming one consistency model is correct for every application.
* Describing consistency without specifying its scope.
* Ignoring metadata and synchronisation costs.
* Treating replica-state agreement and client-visible ordering as identical.

### Recall questions

1. How does a consistency model simplify client reasoning?
2. Why does stronger consistency generally increase latency?
3. What is the difference between state and operation consistency?

---

## 32. Strict Consistency

### Core idea

Strict consistency requires every write to become instantly visible to every later read according to one global real-time clock.

```text
write(x, 1) at t₁

Any read at t₂ > t₁
    → must return 1 or a later value
```

### Important fact

Strict consistency is a theoretical model.

Instantaneous propagation across distributed machines is physically impossible. 

### Mental model

Strict consistency assumes:

```text
zero propagation time
+
perfect global clock
```

Neither assumption holds in real distributed systems.

### Common mistakes

* Using “strict consistency” as a synonym for linearizability.
* Assuming clock synchronisation makes propagation instantaneous.
* Claiming a geographically replicated system provides strict consistency.

### Recall questions

1. Which physical limitation makes strict consistency impossible?
2. How does strict consistency differ from linearizability?
3. Why is a global clock insufficient by itself?

---

## 33. Linearizability

### Core idea

Each operation appears to take effect atomically at one point between its invocation and completion.

```text
invoke write
    ─────●─────
         ↑
linearization point
    ───────────
complete write
```



### Rules

* Real-time order must be preserved.
* A completed write must be visible to later operations.
* Partial or unfinished effects cannot be observed.
* Concurrent operations may be ordered either way.
* All clients must agree on one legal total order.

### Mental model

The system may execute concurrently internally, but externally behaves like one correct sequential machine.

### Overlapping operations

Suppose two writes overlap:

```text
W1: write x = 1
W2: write x = 2
```

The system may order them as:

```text
W1 → W2
```

or:

```text
W2 → W1
```

Once one order becomes observable, later reads cannot reverse it.

### Non-overlapping operations

```text
W1 completes
    ↓
W2 begins
```

Linearizability must place `W1` before `W2`.



### Why it matters

Linearizability prevents:

* Stale reads after completed writes
* Values appearing and disappearing
* Real-time order reversal
* Observation of partially applied operations

### Common mistakes

* Assuming every replica must physically update at the linearization point.
* Reordering operations that did not overlap.
* Treating eventual convergence as linearizability.
* Assuming linearizability automatically provides multi-object transaction semantics.

### Recall questions

1. Which operations may linearizability reorder?
2. Why can a completed write not become invisible again?
3. Can two overlapping writes be linearized in either order?
4. Why does a linearizable key-value store not automatically provide atomic transfers across two keys?

---

## 34. Linearization Point

### Core idea

The **linearization point** is the conceptual instant at which an operation’s effect becomes visible.

```text
Before point:
old value

After point:
new value or a later value
```



### Rule

The point must occur:

```text
after invocation
and
before completion
```

An effect cannot appear before the request exists or only after the operation has already reported completion.

### Monotonic visibility

Once a read sees version `V`:

```text
later reads
    → V or a newer version
    → never an older version
```

### Implementation mechanisms

Possible linearization points include:

* Lock-protected update
* Atomic pointer swap
* Compare-and-swap
* Consensus commitment
* Durable log position

### Common mistakes

* Confusing the linearization point with the start of execution.
* Returning success before the operation has a valid linearization point.
* Allowing later reads to return an older value.
* Assuming the physical update happens at only one machine instruction.

### Recall questions

1. Where must the linearization point lie?
2. How does it create the illusion of instantaneous execution?
3. Which visibility rule prevents “time travel” to an older value?

---

## 35. Compare-and-Swap and the ABA Problem

### Compare-and-swap

CAS changes a value only when the current value still equals an expected value.

```text
CAS(location, expected=A, new=C)
```

* Current value is `A` → install `C`.
* Current value differs → fail.

### ABA problem

```text
Thread 1 reads A

Thread 2:
A → B → A

Thread 1 performs CAS expecting A
    → succeeds
```

The value is again `A`, but its history changed.



### Mental model

```text
Same current value
    ≠
same unchanged state
```

### Common mitigation

Associate a version with the value:

```text
(A, version 1)
→ (B, version 2)
→ (A, version 3)
```

A CAS expecting `(A, 1)` now fails against `(A, 3)`.

### Common mistakes

* Treating value equality as proof that no modification occurred.
* Using CAS without considering object reclamation or reuse.
* Performing read-modify-write without validating the version.

### Recall questions

1. Why does ordinary CAS miss an `A → B → A` transition?
2. How can version tags detect ABA?
3. Why is ABA relevant to lock-free pointer structures?

---

## 36. Cost and Composition of Linearizability

### Cost

Distributed linearizability requires coordination and ordering, commonly through consensus.

```text
Client operation
    ↓
replica coordination
    ↓
agreement on order
    ↓
publish result
```



### Sources of cost

* Multiple network messages
* Quorum waiting
* Durable logging
* Cache invalidation
* Synchronisation
* Reduced availability during partitions

### Composition property

Linearizability is **composable**:

> A system composed of independently linearizable objects remains linearizable at the object level.



### Important exception

Composition does not create a transaction spanning several objects.

```text
Account A is linearizable
Account B is linearizable
```

This alone does not make:

```text
debit A + credit B
```

one atomic operation.

### Common mistakes

* Assuming linearizable objects imply serializable multi-object transactions.
* Comparing linearizability cost only through average latency.
* Ignoring loss of availability when a quorum cannot form.
* Adding consensus to operations that do not require real-time ordering.

### Recall questions

1. Why does distributed linearizability often require consensus?
2. What does composability guarantee?
3. Which additional mechanism is needed for atomic multi-object changes?

---

## 37. RIFL: Linearizable RPCs

### Core idea

Reusable Infrastructure for Linearizability assigns every RPC a stable identifier:

```text
(client ID, client sequence number)
```

Retries use the same identifier.



### Deduplication path

```text
Receive RPC ID
    ↓
Completion object exists?
├─ Yes → return stored result
└─ No  → execute operation
          + store completion object atomically
```

### Completion object

A durable completion object records:

* That the operation executed
* Its result
* The client and sequence number

The data mutation and completion object must be committed atomically.

### Client leases

Client IDs are associated with renewable leases.

When a lease expires:

* Old requests are rejected.
* Completion objects can eventually be reclaimed.
* A recovering client must acquire a new lease.



### Cause and effect

```text
Stable RPC ID
    + durable completion record
    + atomic mutation
    =
retry without repeated execution
```

### Common mistakes

* Assigning a new sequence number to a retry.
* Keeping completion records only in memory.
* Executing the mutation before atomically reserving its completion state.
* Accepting operations under an expired lease.
* Deleting completion objects while the client may still retry.

### Recall questions

1. Why must a retry preserve its original request ID?
2. What should the server return for a completed duplicate request?
3. Why must the completion object be stored atomically with the mutation?
4. How do leases bound deduplication-state lifetime?

---

## 38. Sequential Consistency

### Core idea

All operations appear in one shared sequential order that preserves each process’s program order.

Unlike linearizability, this order does not have to respect real-time order between different processes.



### Mental model

```text
One global sequence exists
but
it may not match wall-clock order
```

### Rules

* Operations from one process remain in program order.
* All clients observe the same global operation order.
* Operations from different processes may be rearranged.
* A completed write may become visible later.

### Example

Wall-clock order:

```text
P1 writes 1
P2 later writes 2
```

A sequentially consistent system may globally order them:

```text
2 → 1
```

provided every observer sees `2` before `1`.

### Linearizability vs sequential consistency

| Linearizability                        | Sequential consistency                |
| -------------------------------------- | ------------------------------------- |
| Preserves real-time precedence         | Does not require real-time precedence |
| Completed write constrains later reads | Visibility may lag completion         |
| Composable                             | Not generally composable              |
| Stronger model                         | Weaker model                          |

 

### Common mistakes

* Assuming sequential consistency means every process sees changes immediately.
* Allowing different readers to observe conflicting operation orders.
* Treating process-local program order as optional.
* Using “sequential” to mean operations physically execute one at a time.

### Recall questions

1. Which ordering constraint does sequential consistency relax?
2. Can an operation become visible after it has completed?
3. Must all clients observe writes in the same order?
4. Why is sequential consistency weaker than linearizability?

---

## 39. Causal Consistency

### Core idea

All processes must observe causally related operations in the same order.

Concurrent operations with no causal relationship may appear in different orders to different clients.



### Causal relationship

```text
Write W1
    ↓ observed by a client
Client issues W2 because of W1

Therefore:
W1 → W2
```

Every replica must reveal `W1` before `W2`.

### Concurrent independent writes

```text
P1 writes A
P2 independently writes B
```

Possible observations:

```text
P3 sees A → B
P4 sees B → A
```

Both are valid because neither write caused the other.

### Mental model

```text
Cause before effect
but
unrelated events need no global order
```

### Dependency buffering

An update carrying dependencies cannot be exposed until all preceding operations are available.

```text
Receive W2 depending on W1
    ↓
W1 missing
    ↓
buffer W2
    ↓
apply W1
    ↓
publish W2
```



### Implementation

Causal systems may attach:

* Logical timestamps
* Version metadata
* Dependency context
* Prior operation identifiers



### Common mistakes

* Ordering every independent write globally.
* Publishing an effect before its dependency.
* Using wall-clock timestamps as proof of causality.
* Assuming causal consistency automatically resolves concurrent conflicts.

### Recall questions

1. Which operations must every replica observe in the same order?
2. Why may independent writes appear in different orders?
3. What should a replica do when a dependency is missing?
4. How does causal consistency reduce coordination relative to linearizability?

---

## 40. Vector Clocks

### Core idea

A vector clock records one logical counter per participant.

```text
[P1 counter, P2 counter, P3 counter]
```

It establishes a **partial order** between events and detects concurrent histories. 

### Local update

When process `Pi` creates an event:

```text
vector[i] += 1
```

### Merge

When receiving another vector:

```text
merged[j] = max(local[j], received[j])
```

for every participant `j`.

### Comparing vectors

For vectors `A` and `B`:

| Relationship                               | Meaning                         |
| ------------------------------------------ | ------------------------------- |
| `A ≤ B` in every component                 | A causally precedes or equals B |
| `A < B` in at least one component          | A causally precedes B           |
| Some components greater and others smaller | A and B are concurrent          |

### Divergence

```text
Replica 1 history: [2,1,0]
Replica 2 history: [1,2,0]
```

Neither dominates the other, so the writes conflict concurrently.

### Important limitation

Vector clocks detect conflict; they do not determine the application-correct resolution.

Possible resolution policies include:

* Keep all versions
* Application merge
* Last-write-wins
* CRDT merge

 

### Trade-off

```text
More tracked participants
    → more precise causality
    → larger metadata
```

### Common mistakes

* Treating incomparable vectors as corrupt.
* Selecting one conflicting version arbitrarily.
* Assuming vector clocks provide a total order.
* Growing vectors indefinitely without membership and garbage-collection rules.
* Using last-write-wins while claiming causal conflict preservation.

### Recall questions

1. What does component-wise maximum represent?
2. How does a vector clock distinguish causality from concurrency?
3. Why can it detect but not solve a business conflict?
4. What scaling problem appears as participant count grows?

---

# Session Models

## 41. Client-Centric Consistency

### Core idea

Session models describe consistency from one client’s perspective while it may communicate with different replicas.



### Mental model

```text
Global replicas may diverge
but
one client’s experience follows defined rules
```

### Scope

Session guarantees:

* Apply separately to every client session.
* Do not impose a complete order between different clients.
* Assume a client’s own operations are sequential.
* Make no guarantee about an incomplete operation after client failure.

### Why they matter

A client may move between replicas:

```text
Write V2 to replica A
    ↓
Read from lagging replica B
    ↓
Unexpectedly observe V1
```

Session guarantees prevent specific forms of this behaviour.

### Common mistakes

* Treating session consistency as global consistency.
* Losing session context when routing to another replica.
* Starting a new logical session without understanding that guarantees may reset.
* Assuming incomplete operations succeeded or failed.

### Recall questions

1. Why do mobile clients need session guarantees?
2. Do session models order writes from different clients?
3. What metadata must follow a client between replicas?

---

## 42. Read-Your-Writes

### Rule

After a client successfully writes `V`, its later reads must observe:

* `V`
* A version newer than `V`

```text
write(x, V2)
    ↓
later read(x)
    → V2 or newer
```



### Why it matters

Without it, users may:

* Update a profile and see old data.
* Submit an item and not find it.
* Change a setting that appears to revert.

### Common implementation

Route the client to:

* The same replica
* A replica known to have applied the write
* A replica that waits until the required version arrives

### Recall questions

1. Does read-your-writes guarantee that another client sees the write?
2. How can version tokens preserve the guarantee across replicas?
3. What should happen if the selected replica has not applied the required version?

---

## 43. Monotonic Reads

### Rule

After a client observes version `V`, later reads cannot return an older version.

```text
read → V5
later read → V5, V6, V7...
never V4
```



### Mental model

The client’s visible timeline only moves forward.

### Common mistakes

* Routing each read randomly without tracking observed versions.
* Assuming monotonic reads imply the latest global value.
* Resetting version context silently during failover.

### Recall questions

1. Can a monotonic read still be stale globally?
2. How does the guarantee differ from read-your-writes?
3. Which client metadata can enforce it?

---

## 44. Monotonic Writes

### Rule

Writes from one client must become visible in that client’s submission order.

```text
Client:
write V1
write V2

Visibility:
V1 → V2
never V2 → V1
```



### Why it matters

If `V2` propagates before `V1`, the delayed older write may later overwrite the newer state.

```text
new state visible
    ↓
late old write arrives
    ↓
old data resurrected
```

### Common mistakes

* Sending one client’s writes through independent unordered channels.
* Retrying an older write after a newer write has committed.
* Ignoring client sequence numbers.

### Recall questions

1. How can out-of-order propagation cause data loss?
2. Which metadata preserves per-client write order?
3. Does monotonic writes order operations from different clients?

---

## 45. Writes-Follow-Reads

### Rule

A client’s write must be ordered after every write whose effect the client previously observed.

```text
Client reads V1
    ↓
Client writes V2 based on V1
    ↓
V1 must become visible before V2
```



### Mental model

A response cannot appear before the message it responds to.

### Quick example

```text
Read: article version 4
Write: comment on version 4
```

Other clients should not see the comment without first being able to observe the article state that caused it.

### Recall questions

1. Why is writes-follow-reads a causal guarantee?
2. What dependency metadata must accompany the new write?
3. Can the write be exposed while its observed dependency is absent?

---

## 46. PRAM/FIFO Consistency

### Core idea

PRAM consistency combines:

* Read-your-writes
* Monotonic reads
* Monotonic writes

Writes from each individual process appear in program order.

Writes from different processes may appear in different orders to different observers.



### Comparison

| Sequential consistency                  | PRAM consistency      |
| --------------------------------------- | --------------------- |
| One global order for all writes         | Per-writer order only |
| All readers agree on cross-writer order | Readers may disagree  |
| Stronger                                | Weaker and cheaper    |

### Recall questions

1. Which order does PRAM preserve?
2. Can two readers disagree about writes from different clients?
3. Why does PRAM require less coordination than sequential consistency?

---

# Eventual and Tunable Consistency

## 47. Eventual Consistency

### Core idea

When updates stop, all replicas eventually converge to the latest resolved value.

```text
No new updates
    +
replication continues
    +
conflicts are resolved
    ↓
all replicas converge
```



### Important limitations

“Eventually” specifies no fixed upper time bound.

During convergence, clients may observe:

* Stale values
* Missing values
* Different replica states
* Concurrent versions

### Conflict resolution

When replicas accepted conflicting writes, convergence requires a rule such as:

* Last-write-wins
* Vector-clock conflict handling
* Application merge
* CRDT merge

### Important exception

Eventual consistency does not mean arbitrary permanent inconsistency.

The guarantee depends on:

* Updates eventually stopping
* Communication eventually recovering
* Replication continuing
* Conflict resolution being deterministic

### Common mistakes

* Claiming a specific staleness bound from eventual consistency alone.
* Assuming replicas converge while writes continue indefinitely.
* Omitting a conflict-resolution policy.
* Treating stale data as a protocol violation under this model.

### Recall questions

1. Which conditions are required for convergence?
2. What timing guarantee does eventual consistency provide?
3. Why is conflict resolution necessary after concurrent writes?
4. Can eventual consistency provide session guarantees?

---

## 48. Tunable Consistency

### Core parameters

| Symbol | Meaning                        |
| ------ | ------------------------------ |
| `N`    | Number of replicas             |
| `W`    | Write acknowledgments required |
| `R`    | Read responses required        |



### Overlap rule

```text
R + W > N
```

ensures every read set overlaps every successful write set in at least one replica.

Example:

```text
N = 3
W = 2
R = 2
```

Any two read replicas overlap the two replicas that acknowledged the write.



### Trade-offs

| Configuration      | Benefit                         | Cost                                |
| ------------------ | ------------------------------- | ----------------------------------- |
| Low `W`, high `R`  | Fast writes                     | Expensive and less available reads  |
| High `W`, low `R`  | Cheap reads of completed writes | Expensive and less available writes |
| Quorum `R` and `W` | Balanced overlap                | Requires majority availability      |
| Low `R` and `W`    | High availability               | Greater stale-read risk             |

### Important assumption

Set overlap alone is insufficient unless the read reconciles versions correctly.

The overlapping replica must:

* Return its latest value
* Include sufficient version metadata
* Be compared with other replies

### Common mistakes

* Treating `R + W > N` as a universal linearizability guarantee.
* Sending writes to only `W` replicas instead of attempting all `N`.
* Returning the first read response instead of reconciling `R` replies.
* Ignoring failed or partial writes.
* Assuming the same `R` and `W` are ideal for every operation.

### Recall questions

1. Why does `R + W > N` create an overlap?
2. What extra assumptions are required for the overlap to reveal the latest value?
3. Which configuration favours write latency?
4. How do higher consistency levels affect availability?

---

## 49. Quorums

### Core idea

A majority quorum contains:

```text
⌊N / 2⌋ + 1
```

participants.

For:

```text
N = 2f + 1
```

a majority system can tolerate up to `f` unavailable replicas while still forming a quorum. 

### Quorum intersection

Any two majorities share at least one participant.

```text
N = 5
Quorum size = 3

{A, B, C}
{C, D, E}

Overlap: C
```

### Why it matters

A read quorum can intersect the quorum that accepted the latest completed write.

### Important exception: incomplete writes

Suppose a failed write reaches only one replica.

Different quorum reads may:

* Include that replica and return the new value
* Exclude it and return the old value

Later reads can therefore alternate between versions unless repair or stricter rules are applied. 

### Common mistakes

* Assuming quorum intersection makes every observed value monotonic.
* Counting an incomplete write as either universally committed or universally absent.
* Using stale replica membership to calculate quorum size.
* Assuming a quorum tolerates failure of a majority.

### Recall questions

1. Why do any two majorities overlap?
2. How many failures can a `2f + 1` group tolerate while retaining a majority?
3. How can an incomplete write break monotonic reads?
4. Which repair mechanism can restore monotonicity?

---

## 50. Witness Replicas

### Core idea

A witness participates in quorum decisions without permanently storing a full copy of every record.

| Copy replica            | Witness replica                               |
| ----------------------- | --------------------------------------------- |
| Stores normal data copy | Usually stores evidence that a write occurred |
| Serves record reads     | Primarily supports quorum                     |
| Full storage cost       | Lower normal storage cost                     |



### Temporary upgrade

When a copy replica is unavailable, a witness may temporarily store the full value.

```text
Copies: 1c, 2c
Witness: 3w

2c unavailable
    ↓
3w temporarily stores value
    ↓
2c recovers
    ↓
repair 2c
    ↓
3w discards temporary copy
```



### Required rules

With `n` copy replicas and `m` witnesses:

* Reads and writes use majorities of `n + m`.
* Every quorum must contain at least one replica holding the data.



### Benefits

* Lower storage duplication
* Majority availability
* Temporary failover storage
* Retains evidence of completed writes

### Costs

* More complex repair
* Witness promotion and demotion
* Reads cannot rely on witness metadata alone
* Temporary copies must be tracked
* Failure combinations must preserve at least one accessible value

### Common mistakes

* Forming a read quorum containing only metadata witnesses.
* Removing the temporary witness value before repairing copies.
* Treating witnesses as ordinary permanent replicas.
* Counting a witness acknowledgment without persisting required evidence.

### Recall questions

1. How does a witness reduce storage cost?
2. When must it store a full record temporarily?
3. Why must a read quorum contain a data-holding replica?
4. What repair occurs after the original copy returns?

---

# Strong Eventual Consistency and CRDTs

## 51. Strong Eventual Consistency

### Core idea

Replicas may receive updates:

* Late
* In different orders
* During network partitions

But replicas that have received the same set of updates converge to the same valid state.



### Mental model

```text
Same updates
    +
deterministic conflict-free merge
    ↓
same final state
```

### Difference from ordinary eventual consistency

| Eventual consistency                                    | Strong eventual consistency                                 |
| ------------------------------------------------------- | ----------------------------------------------------------- |
| Replicas eventually converge under a resolution process | Same update set is sufficient for deterministic convergence |
| Conflict handling may be external                       | Data type embeds merge semantics                            |
| Resolution may discard or ask application               | Operations or states are designed to merge                  |

### Why it matters

Replicas can accept local updates without synchronising first, improving:

* Availability
* Partition tolerance
* Local latency

### Common mistakes

* Assuming strong eventual consistency provides linearizability.
* Applying arbitrary non-mergeable operations and expecting convergence.
* Ignoring duplicate or reordered delivery.
* Treating convergence as immediate.

### Recall questions

1. What condition guarantees that two replicas converge?
2. Which coordination cost can SEC avoid during a partition?
3. Why does SEC not guarantee that clients immediately see the same state?

---

## 52. Conflict-Free Replicated Data Types

### Core idea

A CRDT defines state or operations whose merges converge regardless of arrival order under the data type’s assumptions.

```text
Replica A updates locally
Replica B updates locally
    ↓ communication restored
Merge
    ↓
same result on both replicas
```



### Mental model

Do not detect a conflict after it happens.

Design the data type so concurrent updates have a valid deterministic combination.

### Useful properties

CRDT merge behaviour commonly relies on operations or merge functions that are:

* Commutative
* Deterministic
* Compatible with duplicates and reordering
* Causally constrained where required

The material highlights commutativity and causal ordering for operation-based CRDTs. 

### Trade-off

| Benefit                        | Cost                                     |
| ------------------------------ | ---------------------------------------- |
| Local writes during partitions | Additional metadata                      |
| Deterministic convergence      | Restricted operation semantics           |
| No central conflict resolver   | State growth or tombstones               |
| Reordering tolerance           | Application must choose appropriate CRDT |

### Common mistakes

* Calling any application-level merge a CRDT.
* Assuming every business invariant can be encoded conflict-free.
* Using a CRDT whose merge semantics do not match user intent.
* Forgetting metadata and garbage-collection costs.
* Treating convergence as serial execution.

### Recall questions

1. Why do commutative operations tolerate reordering?
2. Which application invariants may still require coordination?
3. What cost replaces synchronous conflict prevention?

---

## 53. Grow-Only Counter

### Core idea

Each node owns one position in a counter vector and may increment only its own component.

```text
Node 1: [1, 0, 0]
Node 2: [0, 0, 0]
Node 3: [0, 0, 1]
```

### Merge

Take the component-wise maximum:

```text
merge([1,0,0], [0,0,1])
    =
[1,0,1]
```

### Value

Sum all components:

```text
value([1,0,1]) = 2
```



### Why it converges

* Nodes never decrease their own components.
* Maximum is independent of merge order.
* Replaying the same state does not increase the result twice.

### Limitation

The grow-only counter supports increments but not decrements.

### Common mistakes

* Adding vectors component-wise during merge, which double-counts repeated messages.
* Allowing one node to modify another node’s component.
* Summing before merging all known state.
* Expecting the vector to remain small as membership grows.

### Recall questions

1. Why is component-wise maximum used instead of addition?
2. What prevents duplicate delivery from increasing the count twice?
3. Why may only one node update each component?

---

## 54. PN-Counter

### Core idea

A positive-negative counter uses two grow-only vectors:

* `P` for increments
* `N` for decrements

```text
value = sum(P) - sum(N)
```



### Update rules

```text
increment at node i:
P[i] += 1

decrement at node i:
N[i] += 1
```

Both vectors merge using component-wise maximum.

### Trade-off

* Supports increments and decrements.
* Requires twice the vector metadata.
* Membership growth increases state size.

### Recall questions

1. Why cannot a grow-only counter simply decrement its component?
2. How is the final PN-counter value calculated?
3. Which merge rule applies to both vectors?

---

## 55. Replicated Registers

### Last-write-wins register

Stores:

```text
(value, globally ordered timestamp)
```

During merge, retain the value with the greatest timestamp. 

### Benefit

* Simple deterministic merge
* One final value

### Cost

* Concurrent values may be discarded.
* Correctness depends on a safe ordering mechanism.
* Clock errors can select the wrong winner.

### Multi-value register

When concurrent values must not be silently discarded:

```text
Replica A writes VA
Replica B writes VB
    ↓
merge → {VA, VB}
```

The application later resolves or displays both values.

### Common mistakes

* Calling physical-clock last-write-wins conflict-free without accounting for clock errors.
* Assuming “last” means causally latest.
* Discarding concurrent values when the application requires preservation.
* Letting a multi-value register grow without conflict resolution.

### Recall questions

1. What information determines the LWW winner?
2. Which data can LWW silently lose?
3. When is a multi-value register more appropriate?
4. Why is a globally ordered timestamp difficult to guarantee?

---

## 56. Replicated Sets

### Grow-only set

Each replica may add elements.

Merge uses set union:

```text
merge(A, B) = A ∪ B
```

Union is independent of merge order.

### Add-remove set

Use two sets:

* `A`: elements ever added
* `R`: elements removed

Current state:

```text
A − R
```

Only previously added elements should enter the removal set. 

### Trade-off

* Removes cannot simply erase historical add information.
* Removal metadata may grow.
* Re-adding a removed element requires a more precise set design than this simple model.

### Common mistakes

* Physically deleting an element and losing convergence information.
* Removing an element that was never observed as added.
* Assuming a simple two-set model supports every add/remove ordering.
* Garbage-collecting removal markers before all replicas have observed them.

### Recall questions

1. Why does union make a grow-only set converge?
2. Why must removal information be retained?
3. What ambiguity appears when an element is removed and later re-added?

---

## 57. Structured CRDTs

### Core idea

CRDT principles can be applied to nested structures such as:

* Maps
* Lists
* JSON documents
* Collaborative text

A replicated JSON structure can merge nested insertions, deletions and assignments without requiring one global delivery order. 

### Why it matters

Users can edit different parts of a document while disconnected, then merge their changes after reconnection.

### Trade-off

* Operation identifiers and causal metadata
* Complex deletion semantics
* Retention of tombstones
* Potentially surprising concurrent edits
* More difficult debugging

### Common mistakes

* Assuming every pair of text edits has one obvious merge.
* Removing operation metadata too early.
* Treating deterministic convergence as proof of user-intended output.
* Ignoring document growth caused by retained history.

### Recall questions

1. How can nested operations be merged without one global order?
2. Why can converged output still differ from user intent?
3. Which metadata enables concurrent edits to remain distinguishable?

---

## 58. Chapter 11 Consistency Hierarchy

### Single-operation models

From stronger global ordering to weaker ordering:

| Model                      | Main guarantee                                   |
| -------------------------- | ------------------------------------------------ |
| **Linearizability**        | Real-time order and atomic visibility            |
| **Sequential consistency** | One global order preserving each process’s order |
| **Causal consistency**     | One order only for causally related operations   |
| **PRAM/FIFO consistency**  | Per-writer order                                 |
| **Eventual consistency**   | Eventual convergence after updates stop          |



### Session guarantees

| Guarantee           | Prevents                                       |
| ------------------- | ---------------------------------------------- |
| Read-your-writes    | Own successful update disappearing             |
| Monotonic reads     | Returning to an older observed state           |
| Monotonic writes    | One client’s writes appearing out of order     |
| Writes-follow-reads | Effects appearing before their observed causes |



### Core relationships

```text
More global ordering
    → stronger guarantees
    → greater coordination

More local/session ordering
    → lower coordination
    → more cross-client divergence
```

### Rules to retain

* Linearizability respects real-time order.
* Sequential consistency preserves program order but may violate wall-clock order.
* Causal consistency orders causes before effects.
* Vector clocks detect concurrent histories but do not decide business resolution.
* Session models protect one client’s experience, not global state.
* Eventual consistency supplies convergence, not a fixed staleness limit.
* `R + W > N` supplies set overlap, not complete correctness by itself.
* Quorum reads may still alternate after incomplete writes.
* Witnesses reduce storage only when data remains recoverable through the quorum.
* CRDTs embed deterministic merge semantics but restrict allowed operations.
* Stronger underlying consistency can be weakened by an incorrectly designed upper layer.



### Applied recall questions

1. A write completes, but a later real-time read returns an older value. Which model was violated?
2. All clients observe the same write order, but that order contradicts wall-clock completion order. Which model may still hold?
3. Two independent writes are observed in opposite orders by two replicas. Which model allows this?
4. A user updates a profile and immediately sees the old profile from another replica. Which session guarantee is missing?
5. `N = 5`, `W = 2`, `R = 3`. Does every read necessarily overlap every write?
6. A failed write reaches one replica. Why can majority reads alternate between old and new values?
7. Two vector clocks are incomparable. What does this reveal, and what does it not decide?
8. A last-write-wins register uses unsynchronised physical clocks. Which failure can occur?
9. Why can CRDT convergence preserve every increment but still fail a cross-record business invariant?
10. Which consistency model is the weakest one that still preserves cause-before-effect ordering?

# Chapter 12: Anti-Entropy and Dissemination

## 1. Why Anti-Entropy Is Needed

### Core idea

Primary replication may fail to deliver an update to every replica.

```text
Write accepted
    ↓
Some replicas updated
Some replicas unavailable
    ↓
Replica states diverge
    ↓
Anti-entropy detects and repairs differences
```

**Entropy** means the degree of disorder or divergence between replica states.

**Anti-entropy** is the set of background or request-driven mechanisms that brings replicas back into agreement. 

### Two-stage mental model

```text
1. Primary delivery
   Attempt to send the update immediately

2. Periodic or opportunistic repair
   Reconcile anything primary delivery missed
```

### Why not require perfect primary delivery?

Immediate broadcast to every node:

* Depends heavily on the originating node
* Becomes expensive as the cluster grows
* Requires sender and receiver availability to overlap
* May fail during temporary outages
* Requires complete membership knowledge

Anti-entropy distributes responsibility across the cluster and allows primary delivery to be best-effort. 

### Anti-entropy categories

| Type           | When it runs                 | Examples                                     |
| -------------- | ---------------------------- | -------------------------------------------- |
| **Foreground** | During client operations     | Read repair, hinted handoff                  |
| **Background** | Periodically or continuously | Merkle-tree comparison, version-log exchange |

### Key trade-off

```text
Stronger immediate replication
    → higher write latency
    → less later repair

Weaker immediate replication
    → lower write latency and higher availability
    → more divergence and repair work
```

### Common mistakes

* Treating missed replication as permanent data loss.
* Assuming eventual consistency means repair happens automatically without a mechanism.
* Requiring every write coordinator to remain alive until all replicas converge.
* Measuring only foreground request cost while ignoring repair traffic.
* Assuming every repair mechanism provides complete dataset coverage.

### Recall questions

1. Why can primary replication succeed from the client’s perspective while replicas remain divergent?
2. How does anti-entropy reduce dependence on the original coordinator?
3. Which repair methods are limited to actively accessed records?
4. How does weaker write coordination shift cost into later maintenance?

---

## 2. Dissemination Patterns

### Core idea

Cluster-wide information can be spread using three broad communication patterns.

| Pattern                   | How it works                                | Main limitation                  |
| ------------------------- | ------------------------------------------- | -------------------------------- |
| **Broadcast**             | One process sends to all known participants | Sender bottleneck and dependency |
| **Pairwise anti-entropy** | Peers periodically compare state            | Convergence may be slow          |
| **Cooperative gossip**    | Recipients relay information further        | Duplicate messages               |



### Mental model

```text
Broadcast:
one speaker → everyone

Anti-entropy:
pairs compare notes periodically

Gossip:
everyone who hears becomes another speaker
```

### Data vs metadata

Fast cooperative dissemination is especially useful for small, important metadata such as:

* Membership changes
* Failure suspicions
* Schema changes
* Partition ownership
* Node status

Large user datasets are often repaired through more targeted comparison mechanisms. 

### Common mistakes

* Broadcasting large state on every change.
* Assuming every node maintains an accurate complete membership list.
* Using slow periodic repair for urgent leader or schema metadata.
* Using gossip without accounting for duplicates.

### Recall questions

1. Why is direct broadcast suitable for small clusters but problematic at scale?
2. Which information normally requires faster dissemination: user records or cluster membership?
3. How does cooperative propagation remove the original sender as a single delivery dependency?

---

# Read-Side Repair

## 3. Read Repair

### Core idea

Read repair detects divergence while reading a requested key or range.

```text
Coordinator queries replicas
    ↓
Compare returned versions
    ↓
Select correct/latest result
    ↓
Send missing updates to stale replicas
```

Only data involved in the client request is compared. 

### Mental model

A read doubles as a consistency inspection.

```text
Client asks: “What is value K?”
Coordinator also asks:
“Do the replicas agree about K?”
```

### How it works

1. Send the read to multiple replicas.
2. Collect values and version metadata.
3. Reconcile conflicting results.
4. Return the selected result.
5. Update replicas missing the chosen version.

### Scope

Read repair is efficient when:

* Replicas are mostly synchronised.
* Divergence is rare.
* Frequently accessed data should be repaired quickly.
* Full-dataset comparison would be excessive.

### Limitation

Data that is never read may remain divergent indefinitely unless another anti-entropy process repairs it.

### Common mistakes

* Repairing only the coordinator’s local copy.
* Comparing values without version metadata.
* Assuming matching one field proves complete record equality.
* Relying solely on read repair for cold data.
* Returning a stale value before reconciling quorum responses.

### Quick example

```text
Replica A: K = V3
Replica B: K = V3
Replica C: K = V2
```

The coordinator returns `V3` and sends `V3` to replica C.

### Recall questions

1. Why does read repair naturally limit comparison scope?
2. Which records may never be repaired by this mechanism?
3. What metadata is needed to determine which replica is stale?
4. Why should read repair assume replicas are usually already close to agreement?

---

## 4. Blocking vs Asynchronous Read Repair

| Blocking repair                    | Asynchronous repair                           |
| ---------------------------------- | --------------------------------------------- |
| Client waits for repair            | Client receives result before repair finishes |
| Stronger immediate convergence     | Lower read latency                            |
| Can provide monotonic quorum reads | Subsequent reads may still hit stale replicas |
| Lower availability                 | Repair may fail after response                |



### Blocking repair

```text
Read replicas
    ↓
Detect divergence
    ↓
Repair stale replicas
    ↓
Wait for acknowledgments
    ↓
Return to client
```

Because the contacted replicas are repaired before completion, a later quorum read should not move backward to an older value, assuming no intervening write.

### Asynchronous repair

```text
Read replicas
    ↓
Return reconciled result
    ↓
Schedule repair
```

This protects request latency but does not guarantee that repair completes.

### Trade-off

```text
Wait for repair
    → stronger monotonicity
    → more latency and failure exposure

Repair later
    → faster response
    → temporary divergence remains
```

### Important exception

Blocking repair cannot guarantee monotonic reads when later requests use insufficient consistency levels or contact replicas outside the repaired set.

### Common mistakes

* Calling asynchronous repair complete when it has only been scheduled.
* Assuming blocking repair always contacts every replica.
* Using blocking repair on every read without accounting for tail latency.
* Claiming monotonicity when subsequent reads use a weaker replica set.

### Recall questions

1. Why can blocking repair improve monotonic reads?
2. Which availability cost does blocking repair introduce?
3. Under what later read policy can monotonicity still be lost?
4. When is asynchronous repair preferable?

---

## 5. Digest Reads

### Core idea

The coordinator can avoid transferring complete values from every replica.

```text
Replica A → full value
Replica B → hash digest
Replica C → hash digest
```

The coordinator hashes the full value and compares it with the digests. 

### Happy path

```text
All digests match
    ↓
Replicas probably contain identical data
    ↓
Return full value
```

### Mismatch path

A digest mismatch reveals divergence but does not reveal:

* Which replica is newest
* Which replica is stale
* What data differs

The coordinator must request full values, reconcile them and repair lagging replicas.

### Benefits

* Lower normal-case network traffic
* Less serialization work
* Fast equality comparison
* Full values transferred only when necessary

### Cost

* Extra hash computation
* A mismatch requires another round trip
* Hash collisions are theoretically possible
* Digests must cover the same canonical representation

### Rule

A digest establishes likely equality, not ordering or recency.

### Common mistakes

* Selecting a winner based only on digest value.
* Hashing different serialization formats on different replicas.
* Treating a hash mismatch as proof that the digest replica is stale.
* Using a digest as a replacement for version metadata.
* Forgetting the second full-read round after mismatch.

### Recall questions

1. What does a matching digest establish?
2. Why can a mismatch not identify the latest replica?
3. Under which workload do digest reads save the most bandwidth?
4. Why must all replicas hash a canonical representation?

---

# Write-Side Repair

## 6. Hinted Handoff

### Core idea

When a target replica is unavailable, another node temporarily stores the update with a **hint** identifying its intended destination.

```text
Write intended for replica B
    ↓ B unavailable
Node D stores:
[value + “deliver to B” hint]
    ↓ B recovers
D replays update to B
    ↓
Hint removed
```



### Mental model

A healthy node acts as a temporary post office for an unavailable replica.

### Purpose

Hinted handoff:

* Preserves recent missed writes
* Accelerates recovery after short outages
* Avoids immediate full-state comparison
* Improves write availability

### Hint contents

A hint generally needs:

* Destination replica
* Key and value or mutation
* Version or timestamp
* Expiration or retention metadata
* Delivery status

### Important distinction

A hint may not count as a normal readable replica because its primary purpose is deferred delivery.

Whether it counts toward write consistency depends on the system’s consistency level and implementation.

### Common mistakes

* Serving normal reads from a hint without defined semantics.
* Deleting the hint before destination acknowledgment.
* Retaining hints forever after a destination is permanently removed.
* Replaying an old hint over a newer value.
* Counting hints toward replication guarantees when the database does not.

### Recall questions

1. Why is hinted handoff best suited to temporary failures?
2. What metadata identifies where a hint should be replayed?
3. Why might a hinted copy not count as a normal replica?
4. How should replay handle a destination containing a newer version?

---

## 7. Sloppy Quorums

### Core idea

A sloppy quorum allows nonowner nodes to temporarily substitute for unavailable target replicas.

Example:

```text
Intended replicas: A, B, C
B unavailable
Temporary write set: A, C, D
```

Node `D` stores the value with a hint for `B`.  

### Why it matters

A strict quorum requires responses from the key’s designated replicas.

A sloppy quorum requires enough healthy nodes, even when some are not designated owners.

### Benefit

* Higher write availability
* Better tolerance of temporary replica loss
* Maintains the requested acknowledgment count

### Consistency cost

The temporary nodes used for a write may not be contacted by a later read.

```text
Write goes to A, D, E
Read goes to B, C
    ↓
Read may miss latest value
```

Sloppy quorum size does not guarantee read/write-set intersection over the same replica group. 

### Strict vs sloppy quorum

| Strict quorum                  | Sloppy quorum                    |
| ------------------------------ | -------------------------------- |
| Uses designated replicas       | May use substitute nodes         |
| Better read/write intersection | Higher failure-time availability |
| May reject during owner outage | Accepts more writes              |
| Less hinted data               | More handoff and repair work     |

### Common mistakes

* Applying the formula `R + W > N` as though substitutes belonged to the same replica set.
* Forgetting to forward temporary copies to their owners.
* Treating acknowledgment count as proof of immediate consistency.
* Leaving duplicate substitute copies indefinitely.
* Reading only designated replicas immediately after a sloppy write and expecting the latest value.

### Recall questions

1. How does a sloppy quorum differ from an ordinary quorum?
2. Why can a later read miss a successfully acknowledged sloppy write?
3. Which mechanism eventually restores the intended replica placement?
4. What availability improvement is purchased with weaker intersection?

---

# Dataset-Wide Repair

## 8. Merkle Trees

### Core idea

A Merkle tree is a hierarchy of hashes representing data ranges.

```text
                  Root hash
                /           \
        Range-group hash   Range-group hash
          /       \          /       \
      Range A   Range B   Range C   Range D
```

Leaf hashes represent record ranges. Each parent hash is calculated from its child hashes. 

### Comparison process

1. Compare replica root hashes.
2. Root hashes match → represented datasets probably match.
3. Root hashes differ → compare child hashes.
4. Recurse only into differing subtrees.
5. Locate inconsistent key ranges.
6. Exchange and repair records in those ranges.



### Mental model

```text
Compare summary of everything
    ↓ mismatch
Compare summaries of halves
    ↓
Narrow repeatedly
    ↓
Inspect only differing ranges
```

### Why it matters

Without Merkle trees, full comparison could require exchanging every record.

Merkle trees let replicas exchange compact summaries and inspect only likely differences.

### Tree precision trade-off

| Smaller leaf ranges             | Larger leaf ranges                     |
| ------------------------------- | -------------------------------------- |
| Precise difference localisation | Smaller tree                           |
| More hashes and metadata        | Less metadata                          |
| More comparison messages        | More records transferred during repair |
| Higher recomputation cost       | Coarser repair                         |

### Update cost

A change to one record requires recomputing hashes from its leaf range up to the root.

```text
Record change
    ↓
Leaf hash changes
    ↓
Ancestor hashes change
    ↓
Root changes
```

### Important assumptions

Replica trees must use the same:

* Key-range boundaries
* Record encoding
* Hash function
* Snapshot or comparison point
* Treatment of tombstones and versions

### Common mistakes

* Comparing Merkle roots built from different snapshots.
* Using different range partitioning on each replica.
* Assuming a matching root proves semantic application correctness.
* Rebuilding the entire tree for every small update when incremental maintenance is available.
* Choosing leaves so coarse that nearly the whole dataset is transferred after one mismatch.

### Recall questions

1. How does a Merkle tree reduce repair comparison cost?
2. Why must replicas agree on range boundaries?
3. How does leaf granularity affect metadata and repair precision?
4. Which hashes change after one record update?
5. What does a root mismatch reveal, and what does it not reveal?

---

## 9. Bitmap Version Vectors

### Core idea

Bitmap version vectors track which individual write events each replica has observed.

Each write is represented as a **dot**:

```text
(node ID, node-local sequence number)
```

Example:

```text
(P2, 8)
```

means the eighth write coordinated by process `P2`. 

### Per-node event ordering

Each coordinator increments its own sequence number without gaps:

```text
P1: 1, 2, 3, 4, ...
P2: 1, 2, 3, 4, ...
```

A replica may have gaps in another node’s sequence because some writes were not delivered.

### Compact representation

A replica can track:

* Highest consecutively observed sequence number
* Bitmap of additional out-of-order events

Example:

```text
Consecutive through: 3
Bitmap after 3: 01101
```

This represents selected later events that arrived despite earlier gaps. 

### Repair process

```text
Replica A sends event summary
Replica B compares summaries
    ↓
Identify missing dots
    ↓
Retrieve records mapped to those dots
    ↓
Replicate only missing writes
```

A dotted causal container maps write dots to record and causal information. 

### Benefits

* Precisely identifies missing recent writes
* Avoids full dataset comparison
* Retains causal relationships
* Efficient when outages are temporary
* Supports out-of-order delivery

### Costs

* Requires retaining write logs
* Lagging replicas delay log truncation
* Metadata grows with gaps
* Node identity and sequence-number reuse must be controlled
* Records must remain recoverable from their dots

### Log truncation

A prefix can be discarded only when every relevant replica has observed all events through that position.

```text
All replicas observed P2 events 1…100
    ↓
Metadata and logs ≤100 may be truncated
```

### Common mistakes

* Treating the highest observed number as proof that all earlier events arrived.
* Reusing a node’s sequence-number stream after losing its identity.
* Truncating event logs while a replica remains behind.
* Keeping the bitmap without a way to recover the represented records.
* Ignoring causal relationships during repair.

### Recall questions

1. What does a dot uniquely identify?
2. Why are bitmaps needed in addition to a maximum sequence number?
3. How do two replicas determine exactly which writes are missing?
4. Why can a long-offline node prevent log truncation?
5. Under what failure pattern are bitmap version vectors especially effective?

---

## 10. Anti-Entropy Mechanism Comparison

| Mechanism                 | Repair scope                       | Best assumption                     | Main limitation                   |
| ------------------------- | ---------------------------------- | ----------------------------------- | --------------------------------- |
| **Read repair**           | Requested keys or ranges           | Divergence is rare and data is read | Cold data remains stale           |
| **Hinted handoff**        | Individual missed writes           | Failures are short-lived            | Hints can expire or accumulate    |
| **Merkle tree**           | Entire dataset, narrowed by ranges | Replicas need complete comparison   | Tree construction and maintenance |
| **Bitmap version vector** | Exact missing write events         | Recent logs remain available        | Long outages block truncation     |

### Three optimisation dimensions

Anti-entropy mechanisms mainly optimise one of:

1. **Scope reduction**

   * Repair only queried or missed records.
2. **Recency**

   * Retain recent event history for fast catch-up.
3. **Completeness**

   * Compare whole datasets efficiently.



### Selection mental model

```text
Need to repair accessed records?
    → Read repair

Need to bridge a short node outage?
    → Hinted handoff

Need complete replica verification?
    → Merkle tree

Need exact recent missing operations?
    → Bitmap version vector
```

### Common mistake

Expecting one mechanism to optimise scope, precision, completeness and storage simultaneously.

### Recall questions

1. Which mechanism finds cold-data divergence?
2. Which mechanism minimises repair to exact missing recent operations?
3. Why are multiple anti-entropy mechanisms commonly combined?
4. Which mechanism depends most directly on retained operation history?

---

# Gossip Dissemination

## 11. Epidemic Mental Model

### Core idea

Gossip distributes information cooperatively through randomly selected peers.

Each process can be in one of three states relative to an update:

| State           | Meaning                                 |
| --------------- | --------------------------------------- |
| **Susceptible** | Has not received the update             |
| **Infective**   | Has update and actively propagates it   |
| **Removed**     | Stops propagating after losing interest |



### Propagation

```text
One node receives update
    ↓ becomes infective
Contacts random peers
    ↓
New peers become infective
    ↓
Propagation expands
    ↓
Nodes eventually stop forwarding
```

### Why gossip is robust

* No single broadcaster must remain available.
* Multiple paths may carry the same update.
* Random peer choice adapts around failed links.
* Nodes need not maintain one fixed topology.
* Flexible membership is supported.



### Fundamental trade-off

```text
Redundant delivery
    → resilience to message and node failure
    → extra network traffic
```

Redundancy is not merely inefficiency; it is part of gossip’s reliability mechanism.

### Suitable uses

* Membership
* Failure information
* Schema metadata
* Cluster configuration
* Small state updates
* Decentralised or mesh networks

### Common mistakes

* Removing duplicate delivery and accidentally reducing reliability.
* Gossiping large payloads without size controls.
* Assuming every node receives the update at the same moment.
* Requiring complete membership knowledge at each node.
* Treating probabilistic dissemination as deterministic broadcast.

### Recall questions

1. Why are duplicate messages essential to gossip robustness?
2. What causes a susceptible process to become infective?
3. Why is gossip suitable for changing membership?
4. What state prevents an update from circulating forever?

---

## 12. Gossip Mechanics

### Fanout

Each process periodically contacts `f` randomly chosen peers.

```text
fanout = f
```

A larger fanout spreads information faster but generates more messages. 

### Latency

Gossip latency is the time required for the cluster to converge.

This differs slightly from the moment every node first receives the update because gossip may continue briefly afterward.

### Redundancy

Message redundancy measures repeated delivery overhead.

```text
Higher redundancy
    → more alternate propagation paths
    → greater network and processing cost
```

### Cluster-size relationship

```text
Cluster grows
    ↓
Keep fanout fixed → convergence latency rises

or

Increase fanout → latency stabilises, traffic rises
```

### Interest loss

Nodes need a termination rule to stop forwarding an old update.

Possible rules:

* Stop with a probability that increases over time.
* Stop after receiving a threshold number of duplicates.
* Stop after a fixed number of rounds.
* Stop after an expiry period.

Duplicate counting can signal that the update is already widespread. 

### Convergent consistency

Older events are increasingly likely to have reached the same set of participants.

```text
Recent event
    → views may differ

Older event
    → views are more likely to agree
```



### Common mistakes

* Increasing fanout without estimating cluster-wide message volume.
* Stopping after one successful peer delivery.
* Forwarding every old message indefinitely.
* Treating convergence time as a strict deterministic bound.
* Assuming fanout alone determines performance.

### Recall questions

1. How does fanout affect latency and redundancy?
2. Why can gossip continue after every node has received an update?
3. How can duplicate count estimate dissemination progress?
4. What happens when cluster size grows but fanout remains unchanged?

---

## 13. Gossip Scalability

### Core idea

Gossip can disseminate an update in approximately logarithmic rounds under suitable assumptions, but round complexity does not capture total message overhead.



### Mental model

```text
Round 0: 1 informed node
Round 1: several informed nodes
Round 2: many informed nodes
Round 3: most nodes informed
```

Each new receiver becomes another sender, producing exponential early growth.

### Important distinction

| Metric                  | Question                                      |
| ----------------------- | --------------------------------------------- |
| Round count             | How many propagation stages are needed?       |
| Message count           | How much total network traffic is generated?  |
| Redundancy              | How many deliveries repeat known information? |
| Convergence probability | How likely is complete dissemination?         |

### Common mistakes

* Calling gossip efficient based only on `O(log N)` rounds.
* Ignoring payload size when calculating overhead.
* Assuming probabilistic delivery gives an absolute completion guarantee.
* Choosing random peers from a biased or isolated subset.

### Recall questions

1. Why can gossip spread rapidly in early rounds?
2. Why does low round complexity not imply low message cost?
3. Which peer-selection bias could prevent cluster-wide convergence?

---

# Overlay Networks

## 14. Fixed Overlay Networks

### Core idea

A gossip system can create a temporary structured communication topology.

An **overlay network** is a logical set of communication links constructed above the physical network.

Peers may be selected using criteria such as:

* Network latency
* Geographic proximity
* Reliability
* Capacity



### Spanning tree

A spanning tree connects every node without cycles or redundant edges.

```text
        A
      /   \
     B     C
    / \     \
   D   E     F
```

### Benefits

* Predictable message paths
* Low duplicate delivery
* Fixed number of forwarding steps
* Lower network overhead than random gossip

### Weakness

One failed edge can disconnect an entire subtree.

```text
Parent link fails
    ↓
All descendants become unreachable through the tree
```



### Island risk

Selecting peers mainly by proximity can form tightly connected groups with weak cross-group links.

```text
Low local latency preference
    ↓
Regional clusters
    ↓
Few bridging edges
    ↓
Partition vulnerability
```

### Common mistakes

* Treating logical overlay links as independent of physical failure domains.
* Optimising only latency and creating isolated islands.
* Assuming a spanning tree remains valid after membership changes.
* Providing no fallback path for a failed parent link.

### Recall questions

1. Why does a spanning tree reduce duplicate messages?
2. How can one failed edge disconnect many nodes?
3. Why can proximity-based selection create network islands?
4. Which mechanism can provide fallback connectivity?

---

## 15. Hybrid Tree and Gossip Dissemination

### Core idea

A hybrid design uses:

* Structured tree broadcast during normal operation
* Gossip-based recovery when the tree is damaged

```text
Normal:
efficient spanning-tree delivery

Failure:
redundant gossip paths repair or bypass tree
```



### Trade-off

| Tree                    | Gossip               |
| ----------------------- | -------------------- |
| Message-efficient       | Failure-resilient    |
| Deterministic routes    | Probabilistic routes |
| Fragile to broken edges | Redundant traffic    |
| Low normal overhead     | Better recovery      |

### Mental model

```text
Use structure for efficiency
Use randomness for resilience
```

### Common mistakes

* Running full redundant gossip continuously when the tree is healthy.
* Disabling gossip before the tree can be repaired.
* Assuming the tree’s current membership is authoritative.
* Failing to detect broken overlay edges.

### Recall questions

1. Which part of the hybrid reduces normal message traffic?
2. Which part restores connectivity after failure?
3. Why is neither the tree nor gossip alone optimal for every condition?

---

## 16. Plumtree: Eager and Lazy Push

### Core idea

Plumtree combines:

* **Eager push:** send full messages through a low-overhead broadcast tree.
* **Lazy push:** send only message identifiers through additional gossip links.



### Normal delivery

```text
Sender
    ↓ full message
Eager tree peers
    ↓
Fast, low-duplicate dissemination
```

### Lazy recovery

```text
Node receives message ID
    ↓
Does not have message body
    ↓
Requests body from peer
    ↓
Repairs missing delivery
```

### Why IDs help

Sending an identifier is cheaper than redundantly sending the complete payload.

```text
Full message on efficient path
+
small IDs on redundant paths
=
low normal overhead + recovery evidence
```

### Tree repair

If a node learns through lazy gossip that it missed a message, this signals a broken or inefficient eager path.

The system can fetch the message and adjust the broadcast overlay.

### Latency adaptation

Fast-responding peers tend to form the eager tree, so under stable conditions the tree can naturally favour lower-latency paths. 

### Common mistakes

* Treating a lazy message ID as the complete update.
* Requesting message bodies the node already holds.
* Removing all lazy links after forming the tree.
* Allowing message IDs to expire before delayed peers can recover.
* Assuming the fastest peer remains optimal indefinitely.

### Recall questions

1. What information is sent through eager links?
2. What information is sent through lazy links?
3. How does a node detect that it missed the full message?
4. Why does Plumtree usually generate less traffic than full epidemic gossip?
5. How can lazy delivery help repair the eager tree?

---

# Partial Membership Views

## 17. Why Partial Views?

### Core idea

Maintaining a complete current list of every cluster member becomes expensive when:

* The cluster is large
* Membership changes frequently
* Nodes join and leave often
* Full broadcast is unnecessary

A peer-sampling service gives each node a smaller overlapping subset of peers. 

### Churn

**Churn** is the rate at which nodes join and leave the system.

Higher churn increases the cost of maintaining complete membership views.

### Mental model

```text
Each node knows a few neighbours
    ↓
Neighbour sets overlap
    ↓
Collectively the cluster remains connected
```

### Trade-off

| Larger local view       | Smaller local view                  |
| ----------------------- | ----------------------------------- |
| More path redundancy    | Lower memory and traffic            |
| Faster discovery        | More dependence on sampling quality |
| More membership updates | Greater risk of isolation           |
| Higher connection cost  | Slower repair                       |

### Common mistakes

* Making partial views disjoint.
* Choosing only nearby peers.
* Letting peer lists become stale.
* Assuming every node needs to know every member.
* Reducing view size without validating overlay connectivity.

### Recall questions

1. Why does churn make complete membership expensive?
2. How can overlapping partial views preserve connectivity?
3. Which failure appears if peer sampling becomes biased?

---

## 18. HyParView

### Core idea

HyParView maintains two membership sets.

| View             |   Size | Purpose                                    |
| ---------------- | -----: | ------------------------------------------ |
| **Active view**  |  Small | Live communication and dissemination links |
| **Passive view** | Larger | Candidate replacements and recovery        |



### Normal operation

```text
Active peers
    → maintain connections
    → exchange cluster information

Passive peers
    → no primary connection
    → retained as backups
```

### Failure recovery

When active peer `P2` fails:

1. Remove `P2` from the active view.
2. Choose `P3` from the passive view.
3. Attempt a connection.
4. If connection succeeds, promote `P3`.
5. If it fails, remove `P3` and try another candidate.



### Shuffle operation

Nodes periodically exchange samples of their views.

```text
Exchange peer samples
    ↓
Add received peers to passive view
    ↓
Remove oldest entries if capacity exceeded
```

This:

* Refreshes membership
* Increases diversity
* Removes stale bias
* Preserves bounded view size

### Bootstrap priority

A node with no active connections may receive priority.

An already-connected peer may replace one of its current active links to help the isolated node join the overlay.

### Benefits

* Low normal message volume
* Bounded connection count
* Fast replacement after failures
* No full membership list
* Good recovery during topology changes

### Costs

* Partial views can still become biased.
* Passive entries may be stale.
* View-size tuning affects resilience.
* Connection replacement may disrupt existing paths.
* Correct bootstrap behaviour is required.

### Common mistakes

* Disseminating through passive peers as though they were active links.
* Allowing active and passive views to grow without bounds.
* Never refreshing the passive view.
* Rejecting every connection from an isolated joining node.
* Selecting replacements without removing failed candidates.

### Recall questions

1. What is the functional difference between active and passive views?
2. How does shuffling prevent peer lists from becoming stale?
3. Why may a node with an empty active view receive special treatment?
4. How does HyParView balance message efficiency and resilience?
5. What happens when a passive replacement also fails?

---

## 19. Plumtree and HyParView Relationship

### Core idea

The protocols solve complementary problems:

| Protocol      | Responsibility                                     |
| ------------- | -------------------------------------------------- |
| **HyParView** | Maintains a robust partial peer overlay            |
| **Plumtree**  | Disseminates messages efficiently over the overlay |

```text
HyParView:
Who should I connect to?

Plumtree:
How should messages flow through those connections?
```

Both rely on:

* Small normal communication sets
* Wider fallback connectivity
* No complete cluster view
* Recovery after failures or partitions



### Common mistakes

* Treating membership sampling and message dissemination as the same concern.
* Building Plumtree over an overlay with no recovery mechanism.
* Assuming HyParView alone determines message ordering.
* Maintaining a global peer list despite using partial-view protocols.

### Recall questions

1. Which protocol maintains peer membership?
2. Which protocol chooses eager and lazy message propagation?
3. Why are partial views useful even in clusters that are not extremely large?

---

## 20. Chapter 12 Design Summary

### Core mental model

```text
Primary replication
    ↓ may miss updates
Anti-entropy
    ↓ repairs divergence

Cluster metadata
    ↓ needs broad dissemination
Gossip and overlays
    ↓ spread information robustly
```

### Key relationships

| Mechanism             | Main benefit                            | Main cost                      |
| --------------------- | --------------------------------------- | ------------------------------ |
| Read repair           | Repairs accessed records                | Adds read latency              |
| Digest read           | Reduces normal response size            | Extra round on mismatch        |
| Hinted handoff        | Fast recovery from temporary outage     | Hint storage and replay        |
| Sloppy quorum         | Higher write availability               | Weaker read/write intersection |
| Merkle tree           | Finds differences across large datasets | Hash maintenance               |
| Bitmap version vector | Precisely identifies missing writes     | Log and metadata retention     |
| Random gossip         | Resilient dissemination                 | Duplicate traffic              |
| Spanning tree         | Message efficiency                      | Fragile links                  |
| Plumtree              | Efficient normal path plus repair       | Dual-path complexity           |
| HyParView             | Bounded robust membership view          | Sampling and view maintenance  |

### Anti-entropy selection rules

* Use **read repair** when queried data should be repaired opportunistically.
* Use **hinted handoff** for recent writes missed during short outages.
* Use **Merkle trees** to inspect complete datasets without exchanging every record.
* Use **bitmap version vectors** when retained write history can identify exact gaps.
* Combine mechanisms because none covers every failure and access pattern efficiently.

### Gossip rules

* Redundancy is part of reliability, not merely waste.
* Fanout trades message cost against convergence latency.
* Gossip requires an explicit interest-loss or expiry rule.
* Structured overlays reduce normal traffic but require redundant recovery paths.
* Partial peer views must overlap and be refreshed.
* Membership and message dissemination are distinct problems.
* Probabilistic convergence is not instantaneous global agreement.

### Common conceptual mistakes

* Assuming eventual propagation eliminates the need for anti-entropy.
* Treating acknowledgment count as proof that all designated replicas contain a write.
* Dropping hints or event logs before lagging replicas recover.
* Comparing Merkle trees built from inconsistent snapshots.
* Using only read repair for rarely accessed records.
* Removing gossip duplication without providing alternative failure paths.
* Creating low-latency peer islands with inadequate cross-group links.
* Requiring every node to maintain complete membership.
* Confusing dissemination convergence with consensus.

### Applied recall questions

1. A replica was offline for ten minutes and missed 100 recent writes. Which mechanism can replay those exact writes most efficiently?
2. Two replicas may have diverged anywhere across a terabyte dataset. Which structure can narrow the differing ranges?
3. A key has never been read since one replica missed its update. Why has read repair not fixed it?
4. A sloppy-quorum write succeeds, but an immediate read returns the old value. Explain how the read and write sets failed to intersect.
5. Digest responses disagree. What additional operations must the coordinator perform?
6. Increasing gossip fanout reduced convergence time but overloaded the network. Which trade-off was encountered?
7. A spanning-tree edge fails and disconnects a whole subtree. How can lazy gossip restore delivery?
8. A HyParView active peer fails. Describe the passive-view recovery path.
9. A node has observed event 10 from another coordinator but not event 9. Why cannot it represent its state using only “latest event = 10”?
10. Why does combining read repair, hints and Merkle trees provide better coverage than any one mechanism?

# Chapter 13: Distributed Transactions

## 1. Why Distributed Transactions Are Hard

### Core idea

A local transaction already contains many internal steps.

A distributed transaction adds:

* Multiple storage nodes
* Independent failures
* Network delays
* Partial communication
* Cross-node visibility
* Global commit or rollback

```text
Local transaction:
operations on one node
    ↓
local concurrency control and recovery

Distributed transaction:
operations on several nodes
    ↓
coordination + distributed commit + distributed recovery
```

Single-partition concurrency-control mechanisms do not by themselves solve multipartition atomicity. 

### Mental model

A distributed transaction must make several independently executing systems behave as though they performed one indivisible state transition:

```text
Global state A
    ↓
transaction executes across partitions
    ↓
Global state B

Allowed outcomes:
A or B

Forbidden:
partly A and partly B
```

### Why atomic visibility is difficult

Even one logical write may involve:

1. Receiving a request
2. Parsing it
3. Locating data
4. Updating memory
5. Writing logs
6. Writing storage pages
7. Sending acknowledgments

A multipartition transaction repeats such work on several independent nodes.

### Required guarantees

* Every participant commits, or every participant aborts.
* Failed participants learn the final decision after recovery.
* Intermediate results remain hidden.
* Rollback restores the previous logical state.
* Node-local durability agrees with global transaction visibility.



### Common mistakes

* Treating several individually atomic writes as one atomic transaction.
* Making one participant’s result visible before the global decision.
* Assuming a timeout means no participant executed its operation.
* Coordinating commit without coordinating recovery.
* Confusing serializable local execution with distributed atomic commitment.

### Quick example

Transfer £100 between accounts stored on different partitions:

```text
Partition A:
debit £100

Partition B:
credit £100
```

Valid outcomes:

* Both changes commit.
* Neither change commits.

Invalid outcome:

* Debit commits while credit aborts.

### Recall questions

1. Why does local serializability not guarantee distributed atomicity?
2. Which additional failure states appear when two partitions participate?
3. Why must transaction results remain hidden until a global decision exists?
4. What recovery information must a failed participant retrieve?

---

## 2. Atomic Commitment

### Core idea

**Atomic commitment** decides whether every participant should apply or reject one proposed transaction.

```text
Coordinator proposes transaction T
    ↓
Participants vote
    ↓
Global decision:
COMMIT or ABORT
```

A participant cannot replace the proposed transaction with an alternative. It can only vote whether it is willing to commit the given transaction. 

### Unanimity rule

```text
All participants vote YES
    → transaction may commit

Any participant votes NO
    → transaction aborts
```

### Mental model

Atomic commitment asks:

> “Will everyone perform this exact transaction?”

Consensus asks a broader question:

> “Which proposed value should everyone choose?”

### Roles

| Role                    | Responsibility                                                       |
| ----------------------- | -------------------------------------------------------------------- |
| **Coordinator**         | Starts the protocol, gathers votes and announces the decision        |
| **Cohort/participant**  | Executes its local transaction portion and votes                     |
| **Transaction manager** | Schedules, coordinates, tracks and recovers distributed transactions |

### Implementation-specific responsibilities

The commitment protocol does not define exactly:

* How local data is prepared
* When changes become durable
* How commit becomes visible
* How rollback is implemented
* Which locks or versions are retained

Those are storage-engine and transaction-manager decisions. 

### Important exception

Classical atomic commitment assumes participants follow the protocol.

It does not tolerate Byzantine participants that lie, make arbitrary decisions or report contradictory state. 

### Common mistakes

* Allowing one cohort to commit after another voted to abort.
* Treating a missing vote as a positive vote.
* Assuming commitment automatically provides transaction isolation.
* Assuming participants can modify the proposed transaction.
* Applying crash-fault atomic commitment to Byzantine participants.

### Recall questions

1. Why does atomic commitment require unanimity?
2. What choice is a cohort allowed to make?
3. Which local transaction details are outside the commitment protocol?
4. How does atomic commitment differ from general consensus?

---

# Two-Phase Commit

## 3. 2PC Roles and State

### Core idea

Two-phase commit uses one coordinator and several cohorts.

Every participant durably records its protocol state so it can recover after failure. 

### Roles

```text
             Coordinator
          /       |       \
     Cohort A  Cohort B  Cohort C
```

Cohorts commonly represent:

* Data partitions
* Database shards
* Independent resource managers
* External transactional systems

### Coordinator selection

The coordinator may be:

* The request-receiving node
* A fixed node
* An elected node
* A randomly selected participant
* A role transferred for reliability

### Durable state

The coordinator records:

* Transaction identifier
* Participants
* Votes
* Final decision

Each cohort records:

* Local transaction state
* Its vote
* Prepared changes
* Final commit or abort result

### Common mistakes

* Keeping protocol state only in memory.
* Losing the participant list after coordinator restart.
* Reusing transaction identifiers.
* Assuming the coordinator role itself guarantees fault tolerance.

### Recall questions

1. Which information must the coordinator recover after restart?
2. Why must cohorts persist their votes?
3. Can the coordinator role move between processes?
4. Why is a unique distributed transaction identifier necessary?

---

## 4. Phase 1: Prepare

### Core idea

The coordinator asks every cohort whether it can commit.

```text
Coordinator → cohorts:
PROPOSE / PREPARE transaction T

Each cohort:
execute or validate local part
persist prepared state
vote YES or NO
```



### Cohort response

#### Vote YES

The cohort promises:

* Its local changes are ready.
* Required state is durable.
* It can commit later without further application-level validation.
* It will not independently reverse the decision.
* It will wait for the coordinator’s final instruction.

#### Vote NO

The cohort cannot safely commit because of conditions such as:

* Constraint violation
* Lock conflict
* Validation failure
* Local storage failure
* Transaction timeout
* Missing required data

### Prepared state

A prepared transaction is commonly:

* Durably recorded
* Not yet publicly visible
* Holding required locks or reservations
* Capable of quickly committing or aborting

```text
Executed locally
    +
durably prepared
    +
not yet visible
    =
in-doubt / precommitted transaction
```

By the time a transaction is ready for final commit, its contents have normally been durably stored at all cohorts. 

### Why preparation matters

The second phase should ideally be a small publication operation:

```text
Prepared state
    ↓ commit decision
Make existing durable changes visible
```

### Common mistakes

* Voting YES before local durability requirements are satisfied.
* Releasing required locks immediately after voting.
* Exposing prepared values to normal readers.
* Re-running validation after promising readiness without a defined rule.
* Allowing a prepared cohort to abort independently.

### Recall questions

1. What promise does a YES vote represent?
2. Why should transaction contents be durable before voting YES?
3. Which resources may remain held while a cohort is prepared?
4. Why are prepared values normally hidden?

---

## 5. Phase 2: Commit or Abort

### Decision rule

```text
Votes = YES from every cohort
    → coordinator records COMMIT
    → coordinator broadcasts COMMIT

Any NO or required vote missing
    → coordinator records ABORT
    → coordinator broadcasts ABORT
```



### Commit path

```text
All YES votes
    ↓
Persist global commit decision
    ↓
Send COMMIT
    ↓
Cohorts publish local changes
    ↓
Release locks/resources
```

### Abort path

```text
At least one NO
    ↓
Persist abort decision
    ↓
Send ABORT
    ↓
Cohorts roll back prepared changes
    ↓
Release resources
```

### Durability ordering rule

The coordinator must durably record the final decision before announcing it.

Otherwise:

```text
Coordinator sends COMMIT
    ↓
crashes before recording decision
    ↓
recovery cannot determine what was announced
```

Every protocol transition must be recoverable from durable local logs. 

### Common mistakes

* Sending commit before persisting the decision.
* Committing with a majority rather than unanimity.
* Forgetting cohorts that did not answer.
* Releasing resources before the final decision.
* Treating prepare as equivalent to commit.

### Recall questions

1. Which vote pattern permits commit?
2. Why must the decision be logged before being sent?
3. What is the difference between prepared and committed?
4. When can cohorts release transaction locks?

---

## 6. 2PC Message Flow

### Successful transaction

```text
Coordinator                Cohorts
     |                         |
     |------ PREPARE --------->|
     |<------- YES ------------|
     |                         |
     |--- durable COMMIT ------|
     |------ COMMIT ---------->|
     |<------- ACK ------------|
```

### Aborted transaction

```text
Coordinator                Cohorts
     |------ PREPARE --------->|
     |<--- YES / NO -----------|
     |                         |
     |---- durable ABORT ------|
     |------ ABORT ----------->|
```

### Cost model

A successful round normally requires:

* Prepare request
* Vote response
* Commit message
* Optional acknowledgment
* Durable logging at coordinator and cohorts
* Local transaction execution

### Why 2PC remains common

* Conceptually simple
* Easy to reason about
* Relatively few protocol rounds
* Clear commit and abort states
* Integrates with local transaction systems

Its simplicity does not eliminate its failure and availability costs. 

### Recall questions

1. How many decision phases does 2PC contain?
2. Which messages belong to each phase?
3. Which operations add durable-storage latency beyond network latency?
4. Why can a simple protocol still have difficult recovery behaviour?

---

# 2PC Failure Scenarios

## 7. Cohort Failure Before Voting

### Pattern

```text
Coordinator sends PREPARE
    ↓
One cohort crashes or becomes unreachable
    ↓
Coordinator cannot collect every YES
    ↓
Transaction aborts
```

A single unavailable required cohort prevents successful commit. 

### Why availability falls

2PC requires every participant to be ready.

```text
Transaction availability
    ≤
availability of least-available required cohort
```

Increasing the number of participants increases the chance that one is unavailable.

### Timeout behaviour

If a cohort does not answer during prepare:

* The coordinator may time out.
* It can safely choose abort.
* Cohorts that prepared must later receive the abort decision.

### Common mistakes

* Interpreting silence as YES.
* Committing with the responding participants only.
* Omitting the failed cohort from the transaction after execution began.
* Assuming replicas automatically make a cohort highly available.

### Recall questions

1. Why can the coordinator safely abort after a missing prepare vote?
2. Why can it not safely commit?
3. How does transaction participant count affect availability?
4. How can replication improve cohort availability?

---

## 8. Cohort Failure After Voting YES

### Core idea

After voting YES, a cohort has promised to follow the coordinator’s final decision.

It cannot safely decide independently. 

### Recovery problem

```text
Cohort votes YES
    ↓
Cohort crashes
    ↓
Other cohorts may commit or abort
    ↓
Cohort recovers in prepared state
    ↓
Must discover final decision
```

The recovering cohort cannot serve affected values correctly until it knows whether the transaction committed.

### Recovery sources

* Coordinator decision log
* Backup coordinator
* Peer transaction logs
* Replicated transaction-manager state

### Why the cohort cannot guess

Other participants may have:

* Voted NO
* Timed out
* Caused coordinator abort
* Received a commit decision before the failure

### Common mistakes

* Automatically committing every recovered prepared transaction.
* Automatically aborting after a positive vote.
* Serving affected rows before resolving the decision.
* Discarding prepared state during restart.

### Recall questions

1. Why is a recovered YES-voting cohort in an uncertain state?
2. Which participant owns the final decision?
3. What durable evidence can resolve the uncertainty?
4. Why may the cohort need to withhold affected data?

---

## 9. Coordinator Failure Before Final Decision

### Core idea

If the coordinator fails before deciding, cohorts may remain prepared but unable to determine whether to commit or abort.

```text
Cohorts vote YES
    ↓
Coordinator crashes before final decision
    ↓
Cohorts hold prepared state
    ↓
No participant can safely choose alone
```



### Blocking property

2PC is a **blocking protocol** because prepared participants may wait indefinitely for the coordinator or a replacement capable of reconstructing the decision.

### Why peer communication may be insufficient

A cohort knowing that it voted YES does not know:

* Whether every other cohort voted YES
* Whether the coordinator recorded commit
* Whether another cohort received the final decision
* Whether an abort condition occurred

### Replacement coordinator

A replacement may need to:

* Recover the original coordinator’s durable log
* Query every participant
* Reconstruct votes
* Resolve any already-recorded decision
* Repeat safe portions of the protocol

### Resource consequences

Prepared cohorts may continue holding:

* Locks
* Memory
* Version metadata
* Log records
* Capacity reservations

A prolonged coordinator outage can therefore block unrelated work.

### Common mistakes

* Assuming cohorts can use majority voting to infer the 2PC decision.
* Releasing prepared locks because the coordinator timed out.
* Choosing abort when another cohort may already have committed.
* Ignoring transactions stuck in prepared state.

### Recall questions

1. Why is 2PC described as blocking?
2. What information is missing from one prepared cohort’s local state?
3. Which resources may remain unavailable while the transaction is in doubt?
4. How can replicated coordinator state reduce blocking risk?

---

## 10. Coordinator Failure After Recording a Decision

### Core idea

The coordinator may durably decide but fail before every cohort receives the result.

```text
Coordinator records COMMIT
    ↓
Some cohorts receive COMMIT
    ↓
Coordinator crashes
    ↓
Other cohorts remain prepared
```

The missing decision can safely be replayed from durable logs or replicated coordinator state. 

### Why replay is safe

The final decision is immutable:

```text
Recorded COMMIT
    → every cohort must eventually commit

Recorded ABORT
    → every cohort must eventually abort
```

### Recovery distinction

| Failure point                | Recoverability               |
| ---------------------------- | ---------------------------- |
| Before durable decision      | Cohorts may remain uncertain |
| After durable decision       | Decision can be replayed     |
| After every cohort completes | Normal cleanup remains       |

### Common mistakes

* Re-running the whole transaction instead of replaying the decision.
* Changing a durable commit into abort after restart.
* Assuming message loss changed the decision.
* Deleting coordinator logs before every participant learns the outcome.

### Recall questions

1. Why is a logged decision safe to replay?
2. How does this failure differ from failure before the decision?
3. When can the coordinator’s decision record be removed?

---

## 11. Replicated Cohorts and Coordinators

### Core idea

A participant can represent a replicated consensus group rather than one physical node.

```text
2PC cohort
    =
replica group with one logical decision
```

If the group retains a quorum, the cohort remains available despite individual-node failures.

### Example relationship

```text
Cross-shard atomicity:
2PC between shard groups

Within one shard:
consensus replicates the shard’s state
```

The book notes that systems such as Spanner run 2PC across Paxos groups rather than individual nodes to improve availability. 

### Benefits

* Cohort survives minority replica failures
* Prepared state is replicated
* Coordinator decision can survive node loss
* Leadership can transfer within the group

### Cost

* Every 2PC step may itself require consensus
* More messages
* More durable log replication
* Higher latency
* More complex recovery

### Mental model

```text
2PC solves:
all-or-nothing across shards

Consensus solves:
one durable shard decision despite replica failures
```

### Common mistakes

* Treating consensus and 2PC as interchangeable.
* Running 2PC over one nonreplicated coordinator and assuming cohorts alone provide fault tolerance.
* Forgetting that consensus adds another coordination layer.
* Assuming replication removes partition-related unavailability.

### Recall questions

1. Which problem does consensus solve inside a cohort?
2. Which problem does 2PC solve across cohorts?
3. How can a Paxos group remain available after one node fails?
4. Why does this design increase transaction latency?

---

# Three-Phase Commit

## 12. Why Add a Third Phase?

### Core idea

Three-phase commit adds an intermediate prepared state so cohorts can sometimes make a deterministic decision after coordinator failure.

```text
2PC:
vote → final decision

3PC:
vote → globally prepared state → final decision
```



### Goal

Avoid the 2PC state where:

* Cohorts voted YES
* Coordinator disappeared
* No cohort knows whether commit was decided

### Critical assumption

3PC assumes a synchronous environment with reliable communication bounds and no network partition.

Without that assumption, its nonblocking guarantees do not hold. 

### Common mistakes

* Applying 3PC in an asynchronous partitionable network and claiming nonblocking safety.
* Treating the extra phase as sufficient without time bounds.
* Assuming more phases always improve fault tolerance.
* Ignoring contradictory decisions across partitions.

### Recall questions

1. Which uncertain 2PC state is 3PC designed to avoid?
2. What timing assumption does 3PC require?
3. Why does the extra phase expose more global state?

---

## 13. 3PC Phases

### Phase 1: Propose

```text
Coordinator proposes transaction
    ↓
Cohorts validate and vote
```

Any NO vote leads to abort.

### Phase 2: Prepare

If every vote is positive:

```text
Coordinator sends PREPARE
    ↓
Every cohort enters prepared-to-commit state
    ↓
Cohorts acknowledge
```

This communicates that all participants voted positively.

### Phase 3: Commit

```text
All prepare acknowledgments received
    ↓
Coordinator sends COMMIT
    ↓
Cohorts publish changes
```



### State mental model

```text
Initial
  ↓
Proposed
  ↓
Prepared-to-commit
  ↓
Committed
```

The middle state helps cohorts infer which timeout decision is safe under the synchronous assumptions.

### Timeout rules

| Cohort state          | Timeout action           |
| --------------------- | ------------------------ |
| Before prepared state | Abort                    |
| Fully prepared state  | Commit may be forced     |
| Final decision known  | Follow recorded decision |



### Common mistakes

* Committing after timeout before all participants reached prepared state.
* Aborting after the protocol has established universal preparation.
* Confusing 3PC prepare with 2PC prepare.
* Failing to persist each phase transition.

### Recall questions

1. What new knowledge does the prepare phase provide?
2. When may a cohort abort on timeout?
3. When may a cohort commit despite coordinator failure?
4. Why must all cohorts advance phase by phase?

---

## 14. 3PC Coordinator Failure

### Before prepare completes

```text
Coordinator fails
    ↓
Not every cohort is known to be prepared
    ↓
Cohorts time out
    ↓
Abort
```

### After every cohort is prepared

```text
Coordinator fails
    ↓
All cohorts share prepared state
    ↓
Cohorts time out
    ↓
Commit
```

This avoids indefinite blocking under the protocol’s synchrony assumptions. 

### Why the state matters

```text
Before prepared:
commit safety not established

After prepared:
all participants have agreed readiness
```

### Common mistakes

* Deciding solely from elapsed time without current protocol state.
* Assuming one cohort’s prepared state proves every cohort is prepared.
* Allowing participants to skip phases.
* Using inconsistent timeout thresholds.

### Recall questions

1. Which state determines the timeout decision?
2. Why can a cohort commit after universal preparation?
3. Why does synchrony matter for inferring other cohorts’ state?

---

## 15. 3PC and Network Partitions

### Core idea

A network partition can cause different groups to take different valid timeout actions.

```text
Group A reached prepared state
    → times out and commits

Group B did not reach prepared state
    → times out and aborts
```

The result is split brain and violated atomicity. 

### Cause and effect

```text
Timeout-based autonomous progress
    +
incomplete communication
    ↓
different groups infer different safe states
    ↓
contradictory decisions
```

### Practical trade-off

| 2PC                                  | 3PC                                      |
| ------------------------------------ | ---------------------------------------- |
| Can block after coordinator failure  | Can progress under synchrony assumptions |
| Fewer messages                       | More messages                            |
| Preserves one decision while waiting | May split under partition                |
| Widely used                          | Less common                              |

The protocol’s extra message cost and partition vulnerability limit its practical use. 

### Common mistakes

* Calling 3PC nonblocking without stating the no-partition assumption.
* Preferring availability over atomicity without acknowledging the contradiction risk.
* Using local timeouts as a substitute for quorum agreement.
* Assuming all participants enter the same phase simultaneously.

### Recall questions

1. How can two 3PC groups reach opposite decisions?
2. Which system assumption was violated?
3. Why is blocking sometimes safer than autonomous timeout progress?
4. Why is 3PC uncommon despite reducing some 2PC blocking?

---

# Deterministic Transactions with Calvin

## 16. Calvin’s Core Idea

### Core idea

Calvin establishes a deterministic global transaction order before execution.

```text
Receive transactions
    ↓
Agree on order
    ↓
Replicate ordered input
    ↓
Acquire data and execute
```

This shifts coordination from execution time to sequencing time. 

### Traditional execution

```text
Start transaction
    ↓
discover conflicts dynamically
    ↓
coordinate locks or validation
    ↓
possibly abort and retry
```

### Calvin execution

```text
Predetermine order
    ↓
every replica receives same ordered inputs
    ↓
execute consistently
```

### Mental model

Instead of allowing transactions to race and deciding the winner afterward, Calvin fixes the race result before the runners start.

### Why it matters

* Lock acquisition order is predetermined.
* Replicas execute equivalent input.
* Execution requires less cross-replica ordering coordination.
* Failure recovery can replay the same ordered transactions.
* Some aborts caused by timing races are avoided.

### Common mistakes

* Assuming deterministic execution requires no consensus.
* Changing transaction input after sequencing.
* Allowing replicas to choose different execution orders.
* Treating deterministic order as equivalent to no locking.
* Ignoring transactions whose read and write sets cannot be known early.

### Recall questions

1. Which coordination is performed before execution?
2. Why do replicas produce equivalent outputs?
3. How can predetermined order reduce transaction aborts?
4. Which workload characteristic makes Calvin difficult to use?

---

## 17. Sequencer and Epochs

### Sequencer

The sequencer:

* Receives transaction requests
* Assigns global order
* Groups transactions into batches
* Establishes transaction boundaries
* Replicates the ordered input

### Epochs

Transactions are grouped into short time windows called **epochs**.

```text
Epoch 1: T1, T2, T3
Epoch 2: T4, T5
Epoch 3: T6, T7, T8
```

Epochs serve as:

* Batching units
* Ordering boundaries
* Replication units
* Scheduling inputs



### Why batching helps

```text
Many small transactions
    ↓
one ordered replicated batch
    ↓
amortised consensus and messaging
```

### Trade-off

| Short epoch             | Long epoch                    |
| ----------------------- | ----------------------------- |
| Lower batching delay    | Better amortisation           |
| Smaller batches         | More scheduling opportunities |
| More replication rounds | Higher queueing latency       |
| Faster individual start | Coarser failure recovery unit |

### Common mistakes

* Executing an epoch before its ordered input is safely replicated.
* Treating transaction arrival time as final execution order without sequencing.
* Making epochs so long that batching dominates latency.
* Allowing different sequencers to produce conflicting epoch contents.

### Recall questions

1. What does the sequencer determine?
2. Why can an epoch be used as a replication unit?
3. How does epoch length trade latency against throughput?
4. Which mechanism makes sequencers agree on epoch contents?

---

## 18. Calvin Scheduler and Workers

### Execution flow

Once a batch is replicated:

1. The scheduler preserves the sequencer’s serial order.
2. Independent transaction parts execute in parallel.
3. Workers determine local read and write dependencies.
4. Required read values are exchanged.
5. Active participant nodes execute and persist their writes.



### Read set

Data required to calculate transaction results.

### Write set

Data the transaction may modify.

```text
Read set
    → determines required inputs

Write set
    → determines active participants
```

### Worker process

```text
Analyse read/write sets
    ↓
Collect local read values
    ↓
Forward values to active participants
    ↓
Receive remote inputs
    ↓
Execute transaction in fixed order
    ↓
Persist local output
```

### Why outputs need not be copied

Each replica receives:

* The same transaction
* The same ordering
* The same required inputs

Therefore deterministic execution should generate equivalent outputs locally.

### Important limitation

Calvin does not naturally support transactions whose later reads dynamically determine additional read or write sets. 

### Common mistakes

* Omitting a possible write from the declared write set.
* Allowing nondeterministic code such as uncontrolled local time or randomness.
* Executing a later ordered transaction before an earlier conflicting transaction.
* Assuming identical transaction text guarantees identical output when inputs differ.
* Using Calvin for highly interactive transactions without adaptation.

### Recall questions

1. Why must read and write sets be known before execution?
2. Which participants receive remote read values?
3. How can transaction parts execute in parallel while preserving serial order?
4. Which sources of nondeterminism must be controlled?

---

## 19. Calvin Replication and Failure Recovery

### Core idea

Sequencers must agree on exactly which transactions belong to an epoch.

Possible mechanisms include:

* Paxos consensus
* Leader-based replication



### Consensus-based input log

```text
Sequencers agree on batch
    ↓
batch becomes durable ordered history
    ↓
failed replica can replay history
```

### Leader-based sequencing

**Benefit**

* Lower normal latency
* Simpler ordering path

**Cost**

* Leader state must be reconstructed after failure
* Progress depends on leader recovery or replacement
* Stale leaders must be fenced

### Failure model

Because execution order and input are replicated, a failed worker can recover by replaying the deterministic history rather than forcing every transaction to abort.

### Common mistakes

* Replicating execution results but not transaction order.
* Recovering from unordered client requests.
* Allowing a replacement leader to reuse stale sequence positions.
* Assuming deterministic execution makes storage durability unnecessary.

### Recall questions

1. Which state must survive a sequencer failure?
2. How does replay avoid aborting transactions after a worker failure?
3. Why can leader-based sequencing improve latency?
4. What additional recovery cost follows leader failure?

---

# Distributed Transactions with Spanner

## 20. Spanner’s Core Model

### Core idea

Spanner combines different mechanisms for different responsibilities:

| Mechanism             | Responsibility                         |
| --------------------- | -------------------------------------- |
| **Paxos**             | Replicate each shard’s state and log   |
| **Two-phase commit**  | Atomic commit across shards            |
| **Two-phase locking** | Isolation for read-write transactions  |
| **TrueTime**          | Globally meaningful timestamp ordering |
| **MVCC versions**     | Snapshot and timestamp reads           |

 

### Mental model

```text
Within each shard:
consensus + leader + replicated log

Across shards:
2PC among shard leaders

Across time:
timestamp ordering with uncertainty bounds
```

### Operation types

| Operation                  | Main behaviour                                     |
| -------------------------- | -------------------------------------------------- |
| **Read-write transaction** | Locks, leader participation and distributed commit |
| **Read-only transaction**  | Lock-free consistent snapshot                      |
| **Snapshot read**          | Read immutable versions at a chosen timestamp      |

### Common mistakes

* Treating Spanner as one global Paxos group.
* Assuming TrueTime alone provides transaction atomicity.
* Assuming 2PC alone provides replica fault tolerance.
* Taking locks for historical immutable snapshot reads.
* Treating all reads as leader-dependent.

### Recall questions

1. Which mechanism handles cross-shard atomicity?
2. Which mechanism replicates a shard?
3. Why can historical reads avoid locks?
4. Which operations require a shard leader?

---

## 21. Paxos Groups and Shard Leaders

### Architecture

* Data is divided into tablets or shards.
* Replicas for one shard form a Paxos group.
* Each group has a long-lived leader.
* Multishard transactions coordinate through group leaders.



### Write path

```text
Client write
    ↓
Shard leader
    ↓
Paxos replication inside group
    ↓
Apply write
```

### Lock table

The group leader maintains:

* Transaction locks
* Write conflict state
* Transaction-manager information

Read-write operations acquire locks.

Snapshot reads can access immutable timestamped versions directly.

### Why group leaders help 2PC availability

A physical cohort node may fail while the logical Paxos group continues.

```text
One replica fails
    ↓
group retains quorum
    ↓
new or surviving leader represents cohort
    ↓
2PC continues
```

### Common mistakes

* Contacting arbitrary replicas as independent 2PC cohorts.
* Allowing a minority replica to represent the group.
* Serving a latest read from a stale replica without validation.
* Failing to transfer lock-table or transaction state after leadership change.

### Recall questions

1. What is the logical 2PC participant in Spanner?
2. How can one physical node fail without making the cohort unavailable?
3. Which state must transfer to a new shard leader?
4. Why can an old snapshot be read from a nonleader replica?

---

## 22. Spanner Commit Timestamps

### Core idea

A read-write transaction receives a commit timestamp greater than:

* Earlier committed transaction timestamps
* Every participant’s prepare timestamp



### Simplified commit path

```text
Shard leaders acquire locks
    ↓
Each Paxos group records PREPARE
    ↓
Coordinator collects prepare timestamps
    ↓
Choose greater commit timestamp
    ↓
Record COMMIT through Paxos
    ↓
Wait until timestamp is definitely in the past
    ↓
Publish result and release locks
```

### Commit wait

TrueTime returns an uncertainty interval rather than pretending the current physical time is exact.

The system waits until it can prove:

```text
real time > chosen commit timestamp
```

This prevents clients from observing a transaction whose timestamp appears to be in the future.

### External consistency

If:

```text
T1 commits before T2 starts
```

then:

```text
timestamp(T1) < timestamp(T2)
```

This provides real-time transaction ordering analogous to linearizability. 

### Trade-off

```text
Smaller clock uncertainty
    → shorter commit wait

Larger clock uncertainty
    → longer latency
```

### Common mistakes

* Using unsynchronised wall-clock timestamps as commit order.
* Releasing locks before the commit decision is replicated.
* Returning before the uncertainty interval has passed.
* Assuming timestamp ordering removes the need for 2PC.
* Confusing snapshot timestamp with transaction start wall time.

### Recall questions

1. Why must the commit timestamp exceed every prepare timestamp?
2. What purpose does commit wait serve?
3. How does clock uncertainty affect latency?
4. What real-time property does external consistency provide?

---

## 23. Single-Shard vs Multishard Transactions

| Single shard       | Multiple shards                            |
| ------------------ | ------------------------------------------ |
| One Paxos group    | Several Paxos groups                       |
| Local lock table   | Locks across leaders                       |
| No cross-shard 2PC | Requires 2PC                               |
| Lower latency      | Extra network and logging rounds           |
| One failure domain | More participant availability dependencies |

A single-shard transaction can use one shard’s consensus log and lock table without consulting a cross-partition transaction manager. 

### Cause and effect

```text
More shards touched
    → more leaders
    → more locks
    → more network messages
    → greater abort and failure exposure
```

### Design rule

Keep frequently co-transacted records in the same shard when practical.

### Common mistakes

* Partitioning related records without considering transaction boundaries.
* Treating one-row operations as single-shard without checking indexes or constraints.
* Ignoring secondary-index shards.
* Assuming distributed transaction cost grows only with data size.

### Recall questions

1. Why does a single-shard transaction avoid 2PC?
2. How can data placement reduce transaction cost?
3. Which hidden structures may turn a simple write into a multishard operation?
4. Why does participant count affect failure probability?

---

# Database Partitioning

## 24. Partitioning and Sharding

### Core idea

Partitioning divides a large logical dataset into smaller independently managed ranges or groups.

```text
Complete dataset
    ↓
Shard A: subset
Shard B: subset
Shard C: subset
```

A replica set becomes the authoritative owner for each subset. 

### Routing key

A routing key determines the target shard.

Examples:

* Customer ID
* Tenant ID
* Account ID
* Hashed record key
* Geographic region

### Request path

```text
Client/query coordinator
    ↓ derive routing key
Partition map
    ↓
Target replica group
```

### Why partition

* Exceed one node’s storage capacity
* Spread write throughput
* Spread read load
* Isolate failure domains
* Move hot ranges independently
* Scale horizontally

### Common mistakes

* Selecting a routing key unrelated to access patterns.
* Requiring full-cluster scans for common queries.
* Ignoring transactions spanning partition keys.
* Treating replicas as separate shards.
* Assuming equal key ranges contain equal data or load.

### Recall questions

1. What determines the target shard?
2. Why can equal-size key ranges have unequal load?
3. How does partition choice affect distributed transactions?
4. What is the difference between a shard and one of its replicas?

---

## 25. Partition Sizing and Hotspots

### Core idea

Partitions should be sized according to:

* Data volume
* Read rate
* Write rate
* Key distribution
* Transaction locality

A dense or frequently accessed range may need smaller partitions. 

### Range hotspot

Sequential or nearby keys often target one range:

```text
new key 1001
new key 1002
new key 1003
    ↓
same range partition
    ↓
write hotspot
```

### Uneven distribution

A routing key such as postcode may divide the key space evenly while population and activity remain uneven.

### Mitigations

* Split hot ranges
* Merge cold ranges
* Hash the routing key
* Add salting or buckets
* Use workload-aware placement
* Redesign the partition key

### Trade-off

| Smaller partitions         | Larger partitions                |
| -------------------------- | -------------------------------- |
| Fine-grained load movement | Less metadata                    |
| Better hotspot isolation   | Fewer cross-partition operations |
| More routing entries       | Simpler management               |
| More possible participants | Larger movement units            |

### Common mistakes

* Measuring only stored bytes and ignoring request rate.
* Waiting for a partition to overload before collecting split metrics.
* Splitting a hot partition along a boundary that preserves the hotspot.
* Creating very small partitions that increase metadata and transaction overhead.

### Recall questions

1. Why can a small partition still be overloaded?
2. How does range partitioning create right-edge hotspots?
3. Which metrics should drive partition splitting?
4. How can smaller partitions increase transaction cost?

---

## 26. Repartitioning

### Core idea

When nodes join, leave or become imbalanced, ownership must move.

Safe ordering:

```text
Copy or stream data to new owner
    ↓
Verify new owner is ready
    ↓
Atomically update partition metadata
    ↓
Route new requests to new owner
    ↓
Retire old copy when safe
```

The book emphasises moving data before routing metadata changes. 

### Why metadata-first movement fails

```text
Routing points to new owner
    ↓
new owner lacks complete data
    ↓
false absence or lost writes
```

### Movement concerns

* Writes arriving during migration
* Snapshot consistency
* Duplicate ownership
* Catch-up logs
* Client routing caches
* Failed transfers
* Old-reader lifetimes

### Common mistakes

* Switching ownership before transfer completion.
* Accepting writes independently at both owners without reconciliation.
* Deleting the old copy immediately after metadata publication.
* Ignoring cached old routing information.
* Moving too many partitions simultaneously.

### Recall questions

1. Why should data movement precede routing publication?
2. How can writes be captured while a partition is copied?
3. Why may the old owner remain temporarily accessible?
4. Which state must be atomically updated?

---

## 27. Modulo Hash Partitioning

### Rule

For `N` nodes:

```text
target = hash(key) mod N
```

### Benefit

* Simple
* Fast
* Often distributes keys evenly
* Reduces natural-key range hotspots

### Main problem

Changing cluster size changes the result for many keys.

```text
hash(key) mod N
    ≠
hash(key) mod (N + 1)
```

Therefore, adding or removing one node may relocate most records. 

### Common mistakes

* Using cluster size directly in a placement function expected to scale elastically.
* Assuming uniform hashing prevents every load hotspot.
* Ignoring hot individual keys.
* Recalculating placement without planning bulk migration.

### Recall questions

1. Why does modulo hashing distribute ordered keys?
2. What happens when `N` changes?
3. Can good key distribution prevent a single hot-key bottleneck?
4. Which property is missing for elastic resizing?

---

## 28. Consistent Hashing

### Core idea

Consistent hashing maps both keys and nodes onto a circular hash space.

```text
0 ───────────── hash ring ──────────── max
↑                                      |
└──────────────────────────────────────┘
```

Each node owns a range between ring positions.

When membership changes, only nearby ranges need to move. 

### Mental model

```text
Key hash
    ↓
move clockwise around ring
    ↓
first responsible node
```

### Membership change

#### Node joins

* Takes responsibility for part of a neighbour’s range.
* Only affected range moves.

#### Node leaves

* Its range moves to the next responsible node.
* Most other keys retain placement.

### Movement property

With `K` keys and `n` nodes, resizing ideally relocates roughly a fraction near:

```text
K / n
```

rather than most keys.

### Trade-off

| Benefit                 | Cost                                   |
| ----------------------- | -------------------------------------- |
| Limited movement        | Ring metadata                          |
| Elastic membership      | Load may remain uneven                 |
| Decentralised placement | Virtual-node management                |
| Reduced range hotspots  | Natural range scans become distributed |

### Important exception

Basic consistent hashing does not guarantee equal load.

Implementations often use:

* Virtual nodes
* Weighted positions
* Explicit token allocation
* Load-aware movement

### Common mistakes

* Assigning one ring point per physical node and expecting perfect balance.
* Confusing consistent hashing with replica consistency.
* Assuming nearby logical keys remain colocated.
* Ignoring replication placement around the ring.
* Moving data without changing ownership metadata safely.

### Recall questions

1. Why does ring placement reduce movement after membership changes?
2. What range does a joining node take?
3. Why are virtual nodes useful?
4. Which query type becomes harder after hashing destroys natural order?

---

# Percolator

## 29. Snapshot Isolation Review

### Core idea

Snapshot isolation gives each transaction a stable view containing values committed before its start timestamp.

```text
Transaction starts at TS
    ↓
All reads use versions committed before TS
```

Concurrent transactions writing the same cell conflict; usually one commits and the other aborts.



### Prevented behaviour

* Dirty reads
* Nonrepeatable reads
* Read skew from mixing incompatible committed states
* Direct lost updates on the same cell

### Remaining anomaly

Snapshot isolation is not serializable.

Two transactions can update disjoint records while jointly violating an invariant:

```text
T1 reads A and B, writes A
T2 reads A and B, writes B
```

No same-cell write conflict occurs, so both may commit.

### Why reads are efficient

Snapshot versions are immutable.

Readers do not require locks merely to prevent concurrent writers from changing their chosen version.

### Common mistakes

* Calling snapshot isolation serializable.
* Checking only direct write/write conflicts.
* Reading the newest committed value instead of the transaction snapshot.
* Reclaiming versions still visible to active snapshots.
* Assuming lock-free reads mean write transactions use no locks.

### Recall questions

1. Why does snapshot isolation prevent read skew?
2. Which conflict causes first-committer-wins?
3. Why can write skew still occur?
4. How does immutable versioning permit lock-free reads?

---

## 30. Percolator Architecture

### Core idea

Percolator builds distributed transactions on top of Bigtable using:

* Timestamped data versions
* Write metadata
* Lock columns
* Conditional atomic mutations
* Client-driven two-phase commit



### Stored information

| Column group       | Purpose                         |
| ------------------ | ------------------------------- |
| **Data**           | Timestamped record versions     |
| **Write metadata** | Points to committed versions    |
| **Locks**          | Tracks in-progress transactions |

### Timestamp oracle

Each transaction obtains:

1. Start timestamp
2. Commit timestamp

The oracle supplies monotonically increasing cluster-wide timestamps.

### Why conditional mutation matters

A conditional read-modify-write operation can atomically:

* Check expected state
* Install a lock
* Reject conflicting state

This prevents races that would occur across multiple independent RPCs.

### Common mistakes

* Using unsynchronised client clocks as transaction timestamps.
* Acquiring a lock through separate non-atomic read and write calls.
* Storing data without corresponding write metadata.
* Assuming the timestamp oracle performs the complete transaction.

### Recall questions

1. Which information does the timestamp oracle provide?
2. Why are data and write metadata separate?
3. How does conditional mutation avoid lock races?
4. Which component drives the transaction protocol?

---

## 31. Percolator Phase 1: Prewrite

### Core idea

The transaction attempts to lock and prepare every cell it will write.

```text
For each write:
check conflicts
    ↓
install lock
    ↓
store pending value
```



### Conflict checks

A prewrite fails if:

* A newer committed value already exists.
* Another unreleased lock conflicts.
* Transaction assumptions are invalid.

### Primary lock

One lock is selected as the **primary**.

```text
Transaction status
    ≈
status of primary lock
```

The primary acts as the recovery and commit reference for secondary writes.

### Mental model

The primary lock is the transaction’s decision record.

Other cells can inspect it to determine whether the transaction:

* Is still running
* Committed
* Aborted
* Needs cleanup

### Common mistakes

* Installing secondary locks before establishing a recoverable primary relationship.
* Ignoring a newer committed timestamp.
* Treating an abandoned lock as a permanent conflict.
* Making prewritten values visible.

### Recall questions

1. Which conflicts are checked during prewrite?
2. What role does the primary lock play?
3. Are prewritten values committed?
4. Why must lock installation be atomic?

---

## 32. Percolator Phase 2: Commit

### Core idea

After all writes are successfully prewritten:

1. Obtain commit timestamp.
2. Commit the primary lock.
3. Replace its lock with write metadata.
4. Commit or clean secondary locks.
5. Expose the transaction versions.



### Publication rule

```text
Primary committed
    → transaction is considered committed
    → secondaries can be completed
```

### Client failure recovery

A client may crash midway.

A later transaction encountering a leftover lock:

1. Checks the primary lock.
2. Primary committed → finish committing the secondary.
3. Primary absent/aborted → roll back the secondary.
4. Clean stale lock state.

### Cooperative recovery

Transactions encountering incomplete state help finish or remove it.

```text
Incomplete metadata
    → not necessarily permanent blocking
    → next observer can resolve it
```

### Common mistakes

* Committing secondary records before the primary decision.
* Deleting a primary lock without leaving commit evidence.
* Treating every old lock as aborted.
* Allowing two recovery workers to perform non-atomic conflicting transitions.
* Returning prewritten data before commit metadata exists.

### Recall questions

1. Which event defines transaction commitment?
2. How does a secondary determine whether it should commit?
3. Why can another transaction safely help with recovery?
4. What metadata replaces a committed lock?

---

## 33. Percolator Trade-Offs

| Benefit                                    | Cost                        |
| ------------------------------------------ | --------------------------- |
| Snapshot-isolated distributed transactions | Not fully serializable      |
| Lock-free snapshot reads                   | Version storage             |
| Client-driven protocol                     | Client-failure cleanup      |
| Atomic per-cell mutations                  | Multiple RPCs               |
| Cooperative recovery                       | Stale-lock detection        |
| Bigtable-based implementation              | Timestamp-oracle dependency |

### Key relationship

```text
Per-cell atomic primitive
    +
timestamps
    +
distributed locks
    +
primary-based recovery
    =
multipartition snapshot isolation
```

### Common mistakes

* Expecting serializable cross-record invariants.
* Ignoring the timestamp oracle’s availability.
* Allowing long transactions to retain old versions indefinitely.
* Failing to periodically resolve abandoned locks.

### Applied recall questions

1. A transaction prewrites all cells but crashes before committing the primary. How should later readers recover?
2. The primary is committed, but one secondary still has a lock. Is the transaction committed?
3. Why can first-committer-wins prevent same-cell lost updates but not write skew?
4. Which state must remain available for cooperative recovery?

---

# Coordination Avoidance

## 34. Invariant Confluence

### Core idea

Coordination can be avoided when independently valid states can always merge into another valid state.

This property is called **invariant confluence**, or **I-confluence**. 

### Formal mental model

Given invariant-valid states `S1` and `S2`:

```text
valid(S1) = true
valid(S2) = true

I-confluence requires:
valid(merge(S1, S2)) = true
```

### Why it matters

If merging independently committed states cannot violate the invariant:

* Partitions can process transactions locally.
* Transactions need not synchronously coordinate.
* Diverged states can later converge safely.
* Availability and scalability improve.

### Example: grow-only set

Invariant:

```text
Every stored element must be valid individually.
```

Two nodes independently add valid elements.

Union preserves the invariant, so coordination is unnecessary.

### Counterexample: unique username

```text
Node A assigns username "om"
Node B assigns username "om"
```

Each local state is valid, but merging creates duplicate ownership.

The operation is not I-confluent without additional design.

### Important rule

Coordination need depends on the combination of:

* Operation
* Invariant
* Merge function

It is not simply a property of the database system.

### Common mistakes

* Declaring an operation coordination-free without defining the invariant.
* Checking local validity but not merge validity.
* Assuming deterministic merge always preserves business rules.
* Using last-write-wins to hide an invariant violation.
* Treating every commutative operation as I-confluent.

### Recall questions

1. What must be true after merging two valid states?
2. Why is uniqueness difficult to preserve without coordination?
3. Can one operation be I-confluent for one invariant but not another?
4. What three elements must be analysed together?

---

## 35. Coordination-Avoiding System Properties

A coordination-avoiding model should provide:

| Property                 | Meaning                                                                    |
| ------------------------ | -------------------------------------------------------------------------- |
| **Global validity**      | Committed and merged states always satisfy invariants                      |
| **Availability**         | Reachable local participants can commit or reject based on invariants      |
| **Convergence**          | Replicas eventually reach the same state after updates and partitions stop |
| **Coordination freedom** | Local transaction progress does not depend on concurrent remote execution  |



### Mental model

```text
Execute locally
    +
preserve invariant
    +
merge safely
    =
coordination avoided
```

### Important exception

A transaction may still require remote data to be copied locally before execution.

Coordination avoidance means avoiding synchronous conflict coordination during local execution, not ignoring required input data.

### Trade-off

| Less coordination              | Required cost              |
| ------------------------------ | -------------------------- |
| Lower latency                  | More metadata              |
| Greater partition availability | Merge logic                |
| More concurrency               | Restricted invariant class |
| Better scalability             | Possible repair reads      |

### Common mistakes

* Calling local-only execution available when required data is unreachable.
* Accepting locally valid changes whose merge is invalid.
* Assuming eventual convergence implies global validity.
* Omitting a deterministic merge procedure.

### Recall questions

1. How does global validity differ from convergence?
2. Does coordination freedom mean remote state is never needed?
3. Which property ensures local transactions do not stall one another?
4. Why are some invariants inherently coordination-requiring?

---

# RAMP Transactions

## 36. Read Atomicity

### Core idea

Read-Atomic Multi-Partition transactions ensure readers see either all or none of another transaction’s writes.

```text
Writer updates A and B

Allowed read:
old A + old B

Allowed read:
new A + new B

Forbidden:
new A + old B
```

The forbidden result is a **fractured read**. 

### What read atomicity prevents

* Uncommitted values
* Aborted values
* Partial transaction visibility
* Fractured reads

### What it does not necessarily provide

* Full serializability
* Real-time ordering
* Prevention of every write skew
* One global transaction order
* Mutual exclusion

### Mental model

Read atomicity protects transaction boundaries in the read result, not every possible transaction dependency.

### Common mistakes

* Treating read atomicity as serializability.
* Returning each key’s individually newest version without considering transaction membership.
* Assuming atomic visibility requires locks.
* Ignoring transaction metadata during multi-key reads.

### Recall questions

1. What is a fractured read?
2. Which combinations of old and new values are allowed?
3. Why is read atomicity weaker than serializability?
4. Can read atomicity be implemented without mutual exclusion?

---

## 37. RAMP Independence Properties

### Synchronisation independence

One client’s transaction should not force another client’s transaction to:

* Block
* Abort
* Wait
* Stall

### Partition independence

A client contacts only partitions containing values involved in its transaction. 

### Mental model

```text
No shared data
    → no shared coordination

Data on partitions A and B
    → do not contact unrelated C and D
```

### Why it matters

System-wide coordination makes every transaction depend on:

* Unrelated nodes
* Slow participants
* Global membership
* Broad failure domains

RAMP limits the transaction’s coordination scope.

### Common mistakes

* Contacting every partition for a transaction involving two keys.
* Requiring a global lock manager for read atomicity.
* Assuming partition independence means no cross-partition messages.
* Allowing unrelated transactions to share a global commit bottleneck.

### Recall questions

1. Which dependency does synchronisation independence remove?
2. Which nodes does partition independence permit the client to contact?
3. Why does limiting transaction scope improve availability?

---

## 38. RAMP Metadata and Repair Reads

### Core idea

RAMP attaches metadata identifying which writes belong to the same transaction.

A reader that sees one transaction version can determine whether sibling writes are missing.

```text
Read A from transaction T
    ↓
Metadata says T also wrote B
    ↓
Read returned old B
    ↓
Fetch T’s version of B
```

 

### Read path

1. Read requested keys.
2. Inspect transaction metadata.
3. Detect partial visibility.
4. Fetch missing sibling versions in another round.
5. Return an atomic transaction view.

### Trade-off

```text
No blocking coordination
    → possible additional read round
```

RAMP moves cost from write-time exclusion to read-time detection and repair.

### Required metadata

Depending on the variant:

* Transaction identifier
* Complete write set
* Version identifiers
* Commit status
* Related partition locations

### Common mistakes

* Omitting one write from the transaction metadata.
* Returning before repairing a detected fractured view.
* Deleting old versions while an in-progress read may require them.
* Treating extra read rounds as protocol failure.
* Assuming metadata alone makes unprepared writes visible.

### Recall questions

1. How does a reader detect a fractured result?
2. Why may an extra communication round be necessary?
3. Which versions must remain available during concurrent reads?
4. Where has RAMP shifted the coordination cost?

---

## 39. RAMP Write Publication

### Simplified phases

#### Prepare

```text
Write new versions to target partitions
    ↓
Keep them invisible
    ↓
Attach transaction metadata
```

#### Commit/abort

```text
Commit:
publish all transaction versions

Abort:
discard or mark prepared versions invalid
```



### Global visibility requirement

By the time one transaction write is visible in one partition, sibling writes must be discoverable and visible to readers in every other involved partition. 

### Version retention

RAMP may maintain:

* Latest committed version
* In-flight uncommitted version
* Older overwritten versions

Old versions remain until concurrent readers that might require them have completed. 

### Mental model

```text
Keep enough history
    → reconstruct a whole transaction view
    → reclaim after old readers finish
```

### Common mistakes

* Exposing one partition’s write before sibling versions are recoverable.
* Reclaiming stale versions based only on latest commit time.
* Making prepared writes visible to normal reads.
* Treating 2PC publication as necessarily involving locks.

### Recall questions

1. Why are prepared writes initially invisible?
2. Which condition allows one transaction version to become visible?
3. Why must stale versions survive active reads?
4. How does version retention replace some blocking?

---

## 40. RAMP Trade-Offs

| Benefit                                | Cost                             |
| -------------------------------------- | -------------------------------- |
| Atomic visibility across partitions    | Transaction metadata             |
| Readers and writers need not block     | Extra read round on fractures    |
| Unrelated partitions remain uninvolved | Write-set tracking               |
| Concurrent versions are supported      | Storage and garbage collection   |
| No mutual exclusion requirement        | Weaker than full serializability |

### Best fit

RAMP is useful when:

* Atomic visibility matters.
* Full serializability is unnecessary.
* High concurrency is required.
* Transactions touch known partitions.
* Occasional additional reads are acceptable.

### Common mistakes

* Using RAMP where cross-transaction invariants require serializability.
* Assuming nonblocking means communication-free.
* Ignoring metadata size for very large write sets.
* Using a version-retention policy unaware of slow readers.

### Applied recall questions

1. A reader gets new `A` and old `B`, both written by transaction `T`. What should it do?
2. Why can a RAMP transaction avoid locking unrelated writers?
3. Which anomaly can still occur even though fractured reads are prevented?
4. How do transaction size and metadata size relate?

---

# Chapter 13 Design Summary

## 41. Atomic Commitment vs Consensus

| Atomic commitment                                | Consensus                                 |
| ------------------------------------------------ | ----------------------------------------- |
| Decide commit or abort for one fixed transaction | Choose one value from proposals           |
| Participants vote willingness                    | Participants may influence selected value |
| Commonly requires unanimity                      | Commonly uses intersecting quorums        |
| 2PC can block on coordinator failure             | Replicated decision can outlive proposer  |
| Solves cross-resource atomicity                  | Solves replicated agreement and ordering  |

The chapter notes that modern systems often combine commitment protocols with consensus because consensus has stronger fault-tolerance properties and separates the decision from one initiator. 

### Mental model

```text
Consensus:
“What is the durable group decision?”

2PC:
“Did every transaction participant agree to commit?”
```

---

## 42. Distributed Transaction Approaches

| Approach         | Core technique                          | Main benefit                                 | Main cost                        |
| ---------------- | --------------------------------------- | -------------------------------------------- | -------------------------------- |
| **2PC**          | Unanimous prepare and decision          | Simple atomic commit                         | Blocking and low availability    |
| **3PC**          | Extra prepared state and timeouts       | Progress after some coordinator failures     | Unsafe under partitions          |
| **Calvin**       | Determine order before execution        | Less execution-time coordination             | Requires known dependencies      |
| **Spanner**      | Paxos groups + 2PC + TrueTime           | Externally consistent transactions           | Several coordination layers      |
| **Percolator**   | Timestamped locks and primary-based 2PC | Snapshot-isolated transactions over Bigtable | Client recovery and lock cleanup |
| **I-confluence** | Merge-valid invariants                  | Avoid unnecessary coordination               | Limited invariant classes        |
| **RAMP**         | Metadata-assisted atomic reads          | Nonblocking atomic visibility                | Read repair and version storage  |

---

## 43. Core Relationships

### Coordination placement

```text
2PC:
coordinate at commit

Calvin:
coordinate ordering before execution

Spanner:
coordinate replication within shards
+
coordinate commit across shards

RAMP:
avoid mutual exclusion
+
repair partial visibility during reads
```

### Fault-tolerance relationship

```text
Nonreplicated coordinator
    → single recovery dependency

Replicated consensus group
    → decision survives node failures

But:
network partition without quorum
    → progress may still stop
```

### Data-placement relationship

```text
Good transaction locality
    → fewer shards
    → fewer participants
    → lower latency and failure exposure
```

### Versioning relationship

```text
More nonblocking reads
    → retain more historical versions
    → more reclamation work
```

---

## 44. Rules to Retain

* Distributed atomicity requires every participant to reach the same commit or abort result.
* Voting YES in 2PC is a durable promise.
* A prepared cohort cannot safely decide independently.
* 2PC blocks when the decision cannot be reconstructed.
* Replicating cohorts improves node-failure tolerance but adds consensus cost.
* 3PC avoids some blocking only under strong timing and communication assumptions.
* Network partitions can make 3PC participants commit and abort independently.
* Calvin shifts coordination into deterministic pre-execution ordering.
* Calvin works best when read and write sets are known before execution.
* Spanner uses consensus within shards and 2PC across shards.
* TrueTime uncertainty creates explicit commit-wait latency.
* Cross-shard transactions cost more than single-shard transactions.
* Safe repartitioning moves data before publishing new routing metadata.
* Modulo hashing causes widespread movement when membership changes.
* Consistent hashing limits movement but does not guarantee perfect balance.
* Snapshot isolation prevents direct write conflicts but not write skew.
* Percolator uses a primary lock as the transaction recovery reference.
* I-confluence identifies operations that preserve invariants without coordination.
* RAMP prevents fractured reads but does not provide full serializability.
* Reducing coordination usually increases metadata, versioning or repair work.

---

## 45. Common Conceptual Mistakes

* Confusing two-phase commit with two-phase locking.
* Treating prepare as final commit.
* Assuming a timeout creates knowledge of the global decision.
* Using 3PC without stating its synchrony assumptions.
* Assuming deterministic execution eliminates replication.
* Treating consensus as a replacement for cross-shard atomic commitment.
* Ignoring shard placement when designing transaction-heavy schemas.
* Claiming snapshot isolation prevents every anomaly.
* Treating local invariant validity as proof of merge validity.
* Calling atomic visibility serializability.
* Reclaiming historical versions while active readers may need them.
* Evaluating transaction latency without including logging, consensus and lock wait.

---

## 46. Applied Recall Questions

1. A cohort voted YES, crashed and recovered. Why can it neither commit nor abort from local state alone?
2. The 2PC coordinator recorded COMMIT but failed before notifying one cohort. What recovery action is safe?
3. Why can replacing each cohort with a consensus group improve availability without removing 2PC?
4. A 3PC partition leaves one group prepared and another unprepared. What contradictory timeout actions can occur?
5. A Calvin transaction discovers a new write key halfway through execution. Which core assumption has failed?
6. Why do deterministic replicas still need identical external inputs such as timestamps and random values?
7. A Spanner transaction modifies one shard. Which cross-shard protocol can it avoid?
8. How does TrueTime uncertainty become user-visible commit latency?
9. A range-partitioned database receives monotonically increasing IDs. Which hotspot is likely?
10. Why does modulo hashing trigger widespread movement after adding one node?
11. A Percolator client crashes after committing the primary but before committing secondary locks. How should another transaction recover them?
12. Two locally valid states assign the same unique username to different users. Why is the operation not I-confluent?
13. A RAMP reader observes only one of three writes from another transaction. Which metadata and repair action are required?
14. How does RAMP trade blocking write coordination for read amplification?
15. Which approach is preferable when read and write sets are known, deterministic ordering is acceptable and execution-time coordination is expensive?

# Chapter 14: Consensus

## 1. Consensus Problem

### Core idea

Consensus allows several processes to agree on one value despite:

* Process crashes
* Message delays
* Message loss
* Temporary false failure suspicions
* Competing proposals

```text
Several proposed values
        ↓
Distributed protocol
        ↓
One agreed decision
```

Consensus is commonly used to:

* Order operations
* Replicate logs
* Elect authoritative leaders
* Maintain consistent state machines
* Publish configuration changes
* Preserve decisions after the original proposer fails

Consensus protocols preserve safety under asynchronous conditions, while failure detection and eventual timing stability help provide liveness. 

### Mental model

Consensus separates:

```text
Who initiated the decision
from
Whether the decision survives
```

A value can become decided even when the original client or proposer never learns the result.

### Common mistakes

* Treating the proposer as the permanent owner of the decision.
* Assuming consensus guarantees bounded completion under complete asynchrony.
* Confusing agreement with atomic commitment.
* Assuming a failed client means its proposed value was not chosen.

### Recall questions

1. How can a proposal become decided after its proposer crashes?
2. Which part of consensus is threatened by indefinite message delay?
3. Why is consensus useful for replicated logs?

---

## 2. Consensus Properties

| Property        | Requirement                                      |
| --------------- | ------------------------------------------------ |
| **Agreement**   | Correct processes do not decide different values |
| **Validity**    | The chosen value was proposed by a participant   |
| **Termination** | Every correct process eventually decides         |



### Safety vs liveness

| Category     | Properties             |
| ------------ | ---------------------- |
| **Safety**   | Agreement and validity |
| **Liveness** | Termination            |

### Mental model

```text
Safety:
Nothing incorrect happens.

Liveness:
Something useful eventually happens.
```

A protocol may preserve agreement indefinitely while making no progress.

### Common mistakes

* Accepting a hard-coded default as valid consensus.
* Sacrificing agreement to make a quick decision.
* Declaring a protocol practical when it can wait forever.
* Treating termination as a fixed latency guarantee.

### Recall questions

1. Can a protocol preserve safety but fail termination?
2. Why does a predetermined output violate validity?
3. Which property is most affected by inaccurate failure detection?

---

# Broadcast Primitives

## 3. Best-Effort Broadcast

### Core idea

One sender independently transmits a message to every participant.

```text
Sender
 ├──▶ P1
 ├──▶ P2
 └──▶ P3
```

If the sender crashes midway:

```text
P1 receives M
P2 receives M
P3 does not receive M
```

No other participant repairs the incomplete broadcast. 

### Benefit

* Simple
* Low normal-case logic
* Appropriate when occasional loss is acceptable

### Limitation

Delivery depends entirely on the original sender remaining able to contact every recipient.

### Common mistakes

* Treating attempted transmission as cluster-wide delivery.
* Assuming all recipients observe the same message set.
* Using best-effort broadcast for replicated decisions without repair.

### Recall questions

1. What happens when the sender crashes after reaching only some recipients?
2. Which additional property must database replication normally provide?

---

## 4. Reliable Broadcast

### Core idea

Reliable broadcast ensures that if a correct process delivers a message, all correct processes eventually deliver it, even if the sender fails.

### Naive fallback

Every recipient rebroadcasts the message:

```text
Sender → P1, P2

P1 → all peers
P2 → all peers
```

This improves reliability but can generate approximately quadratic message traffic. 

### Mental model

```text
Best effort:
sender owns dissemination

Reliable broadcast:
all informed nodes help complete dissemination
```

### Trade-off

| More forwarding                 | Less forwarding              |
| ------------------------------- | ---------------------------- |
| More alternate delivery paths   | Lower message cost           |
| Better sender-failure tolerance | Greater dependency on sender |
| More duplicate messages         | Less redundancy              |

### Important limitation

Reliable broadcast agrees on the **set** of delivered messages, not necessarily their order.

### Common mistakes

* Assuming reliable delivery implies total ordering.
* Applying duplicate messages without deduplication.
* Flooding indefinitely without message identifiers or expiry.
* Ignoring the cost of all-to-all rebroadcast.

### Recall questions

1. Why can recipient rebroadcast survive sender failure?
2. Which guarantee is still missing after reliable delivery?
3. Why are stable message IDs required?

---

## 5. Atomic Broadcast

### Core idea

Atomic broadcast, or total-order multicast, provides:

1. Reliable delivery
2. The same delivery order at every correct process

```text
P1 delivers: A, B, C
P2 delivers: A, B, C
P3 delivers: A, B, C
```



### Required properties

| Property            | Meaning                                                      |
| ------------------- | ------------------------------------------------------------ |
| **Atomic delivery** | Either all correct processes deliver a message or none do    |
| **Total order**     | Every correct process delivers messages in the same sequence |

### Why it matters

If replicas start from identical state and apply identical deterministic commands in identical order:

```text
same initial state
+
same ordered commands
=
same resulting state
```

### Relationship to consensus

Atomic broadcast and consensus are closely related:

* Consensus decides one log position.
* Atomic broadcast repeatedly decides an ordered sequence of positions.

### Common mistakes

* Calling FIFO delivery atomic broadcast.
* Allowing two replicas to deliver the same messages in different orders.
* Applying a message before its position is committed.
* Assuming deterministic order compensates for nondeterministic command execution.

### Recall questions

1. How does atomic broadcast differ from reliable broadcast?
2. Why does total order support state-machine replication?
3. How can repeated consensus implement atomic broadcast?

---

# Virtual Synchrony

## 6. Group Views

### Core idea

Virtual synchrony extends ordered broadcast to changing process groups.

A **view** identifies one specific membership configuration.

```text
View 10: {A, B, C}
View 11: {A, B, D}
```

Messages belong to the view in which they were sent. 

### Receipt vs delivery

| Event        | Meaning                                                 |
| ------------ | ------------------------------------------------------- |
| **Receipt**  | One process has obtained the message                    |
| **Delivery** | The message satisfies the group-wide delivery guarantee |

A received message may remain pending until its delivery is known to be safe.

### View-change barrier

```text
Messages in view V
    ↓
must be resolved
    ↓
View changes to V + 1
```

A message sent in one view cannot be delivered as though it belonged to another view.

### Why it matters

Members of one view agree on:

* Membership
* Delivered messages
* Message order

This prevents one member from treating a message as committed while another member from the same view permanently omits it.

### Common mistakes

* Treating message receipt as atomic delivery.
* Moving an unresolved message into a new view.
* Allowing members to use different group identities.
* Updating membership without ordering the view change.

### Recall questions

1. Why are messages associated with a specific view?
2. How does a view change act as a delivery barrier?
3. Can a process receive a message that is never delivered?

---

# ZooKeeper Atomic Broadcast

## 7. ZAB Mental Model

### Core idea

ZAB is a leader-based atomic-broadcast protocol used to maintain one ordered replica history.

```text
Clients
   ↓
Leader
   ↓ ordered proposals
Followers
```

The leader:

* Receives or is forwarded writes
* Establishes proposal order
* Broadcasts proposals
* Waits for a quorum
* Commits proposals



### Epochs

Time is divided into monotonically numbered epochs.

```text
Epoch 7 → Leader A
Epoch 8 → Leader C
```

Only proposals from the established leader of the current epoch are accepted.

### Mental model

```text
Election picks a candidate.

Epoch establishment
    +
history synchronisation
    =
safe leadership
```

An election result alone is not enough to make a process authoritative.

### Common mistakes

* Accepting proposals from an older epoch.
* Treating prospective election victory as completed leadership.
* Allowing a new leader to broadcast before synchronisation.
* Assuming follower-local order can override leader order.

### Recall questions

1. Why are epoch numbers required?
2. What makes an elected candidate a safe established leader?
3. How are stale-leader proposals rejected?

---

## 8. ZAB Phases

### Phase 1: Discovery

The prospective leader:

1. Collects the latest epochs known by followers.
2. Proposes a greater epoch.
3. Learns the latest transaction observed by each follower.

After accepting the new epoch, followers reject proposals from older epochs. 

### Phase 2: Synchronisation

The leader:

* Establishes itself for the new epoch
* Reconciles follower histories
* Restores committed proposals from prior epochs
* Brings followers to one consistent prefix

Older committed proposals are delivered before new-epoch proposals. 

### Phase 3: Broadcast

```text
Leader receives client change
    ↓
Assigns ordered proposal
    ↓
Broadcasts to followers
    ↓
Waits for quorum acknowledgments
    ↓
Commits proposal
```



### Why the phases are ordered

```text
Establish authority
    ↓
repair history
    ↓
accept new work
```

Skipping synchronisation could place new proposals after inconsistent follower histories.

### Common mistakes

* Broadcasting client updates during discovery.
* Repairing followers after new commands are accepted.
* Treating follower acknowledgment as a vote against a valid proposal.
* Counting responses from the wrong epoch.

### Recall questions

1. What information is learned during discovery?
2. Why must synchronisation finish before broadcast?
3. Which event makes a broadcast proposal committed?

---

## 9. ZAB Failure and Recovery

### Leader liveness

The leader sends heartbeats.

If it cannot contact a follower quorum, it steps down and a new election begins.

Followers also start elections when they suspect the leader has failed. 

### Ordered recovery

Followers report their highest committed proposal.

A new leader can recover by selecting an up-to-date history and streaming missing proposals.

### Efficiency relationship

```text
Long-lived leader
    → no new election per command
    → local sequencing
    → fewer coordination rounds
```

ZAB’s normal broadcast requires two message rounds, while recovery can copy missing history from one sufficiently up-to-date process. 

### Common mistakes

* Continuing leadership without a quorum.
* Selecting recovery state using an uncommitted highest entry.
* Reapplying duplicate proposals without ordered IDs.
* Allowing recovery to omit committed prior-epoch proposals.

### Recall questions

1. Why must a leader step down after losing follower quorum?
2. Which follower is most useful for recovery?
3. Why does total ordering simplify catch-up?

---

# Paxos

## 10. Paxos Roles

| Role         | Responsibility                             |
| ------------ | ------------------------------------------ |
| **Proposer** | Submits a value and seeks sufficient votes |
| **Acceptor** | Promises and accepts proposals             |
| **Learner**  | Learns and stores the chosen result        |

One process may perform several roles. 

### Proposal

A proposal contains:

```text
(proposal number, value)
```

Proposal numbers are:

* Unique
* Totally ordered
* Monotonically increasing from each proposer’s perspective

A common design combines:

```text
(local counter or timestamp, proposer ID)
```

### Mental model

```text
Proposal number
    → authority generation

Value
    → candidate decision
```

A higher proposal number supersedes lower proposal authority, but it cannot arbitrarily replace a value already constrained by earlier accepted proposals.

### Common mistakes

* Reusing proposal numbers.
* Assuming the highest proposal number automatically chooses its proposer’s value.
* Treating learners as voters.
* Assuming each role requires a separate machine.

### Recall questions

1. What does a proposal number order?
2. Why must proposal IDs be unique?
3. Which role persists votes, and which role learns the result?

---

## 11. Paxos Phase 1: Prepare and Promise

### Prepare request

```text
Proposer → acceptors:
PREPARE(n)
```

The proposer contacts enough acceptors to form a phase-1 quorum. 

### Acceptor promise

An acceptor receiving `PREPARE(n)`:

1. Rejects it if it already promised a proposal greater than `n`.
2. Otherwise promises not to accept proposals lower than `n`.
3. Returns its highest previously accepted proposal, when one exists.

```text
PROMISE(n, previously_accepted?)
```



### Purpose

Phase 1 does two things:

* Establishes the proposer’s authority for the round.
* Discovers values that may already constrain the decision.

### Mental model

```text
Promise:
“I will ignore older authorities.”

Accepted-value response:
“Here is the history you must preserve.”
```

### Common mistakes

* Treating a promise as acceptance of a value.
* Forgetting to return the highest accepted proposal.
* Accepting a lower-numbered proposal after making a promise.
* Assuming an acceptor can promise only once.

### Recall questions

1. What does an acceptor promise after `PREPARE(n)`?
2. Why must it report a previously accepted value?
3. Can an acceptor later promise a greater proposal?

---

## 12. Paxos Phase 2: Accept

### Value-selection rule

After receiving a phase-1 quorum:

* No prior accepted value reported → proposer may use its own value.
* Prior accepted values reported → proposer must use the value attached to the highest-numbered accepted proposal.

```text
responses contain no value
    → choose new V

responses contain accepted proposals
    → choose value from highest accepted number
```



### Accept request

```text
Proposer → acceptors:
ACCEPT(n, V)
```

An acceptor accepts unless it has already promised a proposal number greater than `n`.

### Chosen value

A value becomes **chosen** when an acceptor quorum accepts it.

```text
Accepted by one acceptor
    → not necessarily chosen

Accepted by quorum
    → chosen
```

### Critical distinction

| State        | Meaning                                         |
| ------------ | ----------------------------------------------- |
| **Promised** | Acceptor rejected lower future proposal numbers |
| **Accepted** | One acceptor stored a proposal                  |
| **Chosen**   | A quorum accepted the value                     |
| **Learned**  | A participant discovered the chosen value       |

### Common mistakes

* Calling one acceptor’s acceptance a committed decision.
* Selecting the proposer’s new value despite a reported prior acceptance.
* Counting promises as phase-2 acceptances.
* Assuming every acceptor must respond.

### Recall questions

1. When may the proposer choose its own value?
2. When is a value actually chosen?
3. Why can one accepted value influence a later proposer without already being chosen?

---

## 13. Learners

### Core idea

Learners determine which value was chosen.

Possible notification strategies:

* Every acceptor informs every learner.
* Acceptors inform one distinguished learner.
* One learner informs the others.



### Trade-off

| Direct notification         | Distinguished learner            |
| --------------------------- | -------------------------------- |
| Faster independent learning | Fewer messages                   |
| Higher message volume       | Additional dependency            |
| No learner relay            | Learner must redistribute result |

### Common mistakes

* Learning from only one acceptance.
* Assuming the proposer must remain alive for learners to decide.
* Applying an unchosen value to replicated state.

### Recall questions

1. What evidence does a learner need?
2. Can a chosen value be learned after the proposer fails?
3. How does a distinguished learner reduce messages?

---

## 14. Paxos Quorums

### Core idea

Quorums allow progress despite failures while ensuring decision histories intersect.

For crash tolerance of `f` processes, classic majority configurations use:

```text
N = 2f + 1
Quorum = f + 1
```

 

### Intersection

Any two majorities share at least one acceptor:

```text
Q1 ∩ Q2 ≠ ∅
```

That shared acceptor can carry information about earlier accepted proposals into later rounds.

### Quorum means waiting threshold

The proposer may send requests to every acceptor but only needs enough responses to satisfy the quorum.

```text
Send to N
Wait for Q
```

### Failure tolerance

With `2f + 1` acceptors:

* Up to `f` may be unavailable.
* The remaining `f + 1` can still form a majority.

### Common mistakes

* Sending to only the minimum quorum and never updating other replicas.
* Assuming a quorum is always exactly a majority in every Paxos variant.
* Continuing without the required quorum.
* Treating quorum intersection as proof that every node knows the decision.

### Recall questions

1. Why does quorum intersection preserve prior proposal information?
2. How many failures can five acceptors tolerate?
3. Why may requests be sent to more nodes than the protocol waits for?

---

## 15. Proposer Failure

### Case: earlier acceptance is discovered

```text
P1 sends V1 to A1
P1 fails

P2 collects phase-1 responses including A1
    ↓
P2 learns V1
    ↓
P2 must propose V1
```

 

### Case: earlier acceptance is not in the new quorum

```text
P1 sends V1 only to A1
P1 fails

P2 hears from A2 and A3
    ↓
No accepted value reported
    ↓
P2 may propose V2
```

One isolated acceptance was not yet a chosen value.

### Case: old acceptor later returns

A later proposer compares accepted proposal numbers and preserves the highest-numbered relevant value, not simply the oldest value ever encountered. 

### Client uncertainty

The original client may time out while another proposer completes the value.

```text
Client sees failure
    ≠
proposal was not chosen
```

### Common mistakes

* Treating one old acceptor’s state as an already chosen value.
* Treating client timeout as abort.
* Selecting an older accepted proposal over a higher-numbered accepted proposal.
* Assuming proposer failure loses replicated acceptor state.

### Recall questions

1. When must a replacement proposer preserve an old value?
2. Why can it sometimes choose a new value despite one previous acceptance?
3. What should a client do when the proposer fails before replying?

---

## 16. Competing Proposers

### Problem

Two proposers can repeatedly pre-empt each other:

```text
P1 prepares n = 10
P2 prepares n = 11
P1 attempts accept 10 → rejected

P1 retries n = 12
P2 attempts accept 11 → rejected
```

Safety remains intact, but liveness suffers.

### Mitigation

Randomised backoff allows one proposer to pause while another completes. 

### Mental model

```text
Proposal numbers prevent incorrect decisions.

Leader stability or backoff
    prevents endless interference.
```

### Common mistakes

* Interpreting repeated rejection as a safety failure.
* Retrying immediately with synchronised timing.
* Using nonincreasing proposal numbers.
* Assuming quorum availability alone guarantees progress.

### Recall questions

1. Why can competing proposers preserve safety but prevent liveness?
2. How does random backoff help?
3. Which optimisation largely avoids repeated proposer competition?

---

# Multi-Paxos

## 17. Stable Distinguished Proposer

### Core idea

Classic Paxos performs phase 1 for every decided value.

Multi-Paxos establishes a long-lived distinguished proposer, usually called the leader.

After stable leadership is established:

```text
Phase 1 once
    ↓
Phase 2 repeatedly
    ↓
append many ordered values
```



### Benefit

* Fewer message rounds per log entry
* Higher throughput
* Stable ordering
* Less proposer competition

### Cost

* Leader bottleneck
* Leader-election pause
* Stale-leader protection
* Recovery and log synchronisation

### Mental model

| Single-decree Paxos        | Multi-Paxos                      |
| -------------------------- | -------------------------------- |
| One write-once register    | Sequence of write-once registers |
| One chosen value           | Ordered append-only log          |
| Election for each decision | Reuse elected leader             |



### Common mistakes

* Allowing the leader to alter previously chosen log entries.
* Skipping phase 1 before leadership is safely established.
* Assuming the known leader is still current.
* Treating every log slot as mutable state.

### Recall questions

1. Which phase can Multi-Paxos avoid for ordinary entries?
2. Why is the log append-only?
3. What happens when the leader fails?

---

## 18. Multi-Paxos Reads and Leases

### Read problem

A process believed to be the leader may already have been replaced.

Serving a local read from it could return stale state.

### Quorum read

A safe approach is to confirm current authority or read through quorum coordination.

### Lease optimisation

Participants promise not to accept another leader during a bounded interval.

```text
Leader lease valid
    ↓
leader may serve local read
```



### Important rule

A lease is a performance optimisation, not the fundamental safety mechanism.

It relies on bounded clock behaviour:

```text
Leader thinks lease active
+
followers think lease expired
    → stale leadership risk
```

### Trade-off

| Leader-local read         | Quorum-confirmed read          |
| ------------------------- | ------------------------------ |
| Low latency               | Stronger direct confirmation   |
| Requires valid lease      | Additional messages            |
| Sensitive to clock bounds | Higher fault exposure per read |

### Common mistakes

* Using wall-clock leases without bounded drift assumptions.
* Extending a lease without follower agreement.
* Treating lease ownership as permanent authority.
* Serving reads after quorum connectivity is lost.

### Recall questions

1. Why can reading from the last known leader be unsafe?
2. Which assumption makes leases usable?
3. Why are leases not the source of Paxos safety?

---

## 19. Log Snapshots

### Core idea

A replicated log cannot grow forever.

Periodically:

1. Apply committed entries to the state machine.
2. Create a state snapshot.
3. Persist consistent snapshot metadata.
4. Truncate the covered log prefix.



### Safety rule

```text
Snapshot includes entries ≤ K
    ↓
Snapshot becomes durable
    ↓
Log entries ≤ K may be discarded
```

Snapshot publication and log truncation must be coordinated.

### Common mistakes

* Truncating before the snapshot is durable.
* Creating a snapshot from uncommitted entries.
* Restoring a snapshot without its corresponding log position.
* Allowing replicas to compact incompatible prefixes.

### Recall questions

1. What state must a snapshot represent?
2. When may the corresponding log prefix be removed?
3. What recovery bug results from mismatched snapshot and log positions?

---

# Fast Paxos

## 20. Fast Path

### Core idea

Fast Paxos allows clients or proposers to contact acceptors directly during a fast round, removing one normal message hop.

The optimisation requires larger replica sets and quorums:

```text
To tolerate f failures:
acceptors = 3f + 1
fast quorum = 2f + 1
```



### Mental model

```text
Classic:
client → coordinator → acceptors

Fast:
client/proposer → acceptors
```

### Benefit

One fewer round trip when proposals do not collide.

### Collision

Different proposers may send different values:

```text
Some acceptors accept V1
Others accept V2
```

A coordinator must run a recovery/classic round to select one value. 

### Trade-off

| Low conflict                    | High conflict                    |
| ------------------------------- | -------------------------------- |
| Fast path saves latency         | Frequent recovery rounds         |
| Direct acceptance works         | More total messages              |
| Larger quorum may be worthwhile | Can be slower than classic Paxos |

### Common mistakes

* Using classic quorum sizes for fast rounds.
* Assuming direct submission cannot collide.
* Measuring only successful fast-path latency.
* Ignoring additional replica cost.

### Recall questions

1. Which network hop does Fast Paxos remove?
2. Why are larger quorums needed?
3. What happens after a value collision?
4. Under which workload may Fast Paxos perform poorly?

---

# Egalitarian Paxos

## 21. EPaxos Core Idea

### Core idea

EPaxos removes one permanent global leader.

Each command has a temporary leader that records:

* Commands it conflicts with
* A sequence number

 

### Interfering commands

Two commands interfere when their execution order changes the outcome.

```text
A then B ≠ B then A
    → A and B conflict
```

Nonconflicting commands can be committed independently.

### Mental model

```text
Global total order only where needed.

Independent commands
    → no dependency

Conflicting commands
    → explicit dependency
```

### Benefits

* Avoids one global leader bottleneck
* Supports geographically distributed command initiation
* Allows independent commands to progress concurrently
* Reduces unnecessary global ordering

### Costs

* Dependency metadata
* Conflict detection
* Dependency-graph execution
* Slow path on disagreement
* More complicated recovery

### Common mistakes

* Treating every command as conflicting.
* Omitting a dependency whose order affects output.
* Using sequence numbers as replacements for dependencies.
* Assuming leaderless means coordination-free.

### Recall questions

1. When do two commands interfere?
2. Why can independent commands commit without one common leader?
3. What additional metadata replaces global leader sequencing?

---

## 22. EPaxos Fast and Slow Paths

### Pre-Accept

The temporary leader sends:

```text
PRE-ACCEPT(command, dependencies, sequence)
```

Replicas compare the proposed dependencies with their local command logs.

### Fast path

```text
Fast quorum agrees on dependencies
    ↓
Command commits
```

### Slow path

```text
Replica dependency views differ
    ↓
Leader merges dependencies
    ↓
Chooses higher sequence number
    ↓
Sends ACCEPT
    ↓
Collects required acknowledgments
    ↓
Commits
```



### Execution

Committed commands form a dependency graph.

A command executes only after all of its dependencies have executed. 

### Sequence-number role

Sequence numbers:

* Break dependency cycles
* Reject stale messages
* Include configuration epoch, node-local counter and replica identity

### Common mistakes

* Executing a command before its dependencies.
* Treating fast-quorum disagreement as failure instead of entering the slow path.
* Reusing sequence identifiers across configurations.
* Committing with inconsistent dependency sets.

### Recall questions

1. What allows the fast path?
2. How does the slow path reconcile dependency disagreement?
3. Why are sequence numbers needed in addition to dependencies?
4. In what order is the dependency graph executed?

---

# Flexible Paxos

## 23. Cross-Phase Intersection

### Core idea

Paxos does not require every quorum from every phase to intersect every other quorum.

The critical condition is:

```text
Phase-1 quorum intersects every phase-2 quorum
```

For `N` acceptors:

```text
Q₁ + Q₂ > N
```

where:

* `Q₁` = prepare/election quorum
* `Q₂` = accept/replication quorum



### Example

For `N = 5`:

```text
Q₁ = 4
Q₂ = 2

4 + 2 > 5
```

Any two-node phase-2 set intersects the four-node phase-1 set.

### Trade-off

```text
Larger Q₁
    → harder leader election

Smaller Q₂
    → cheaper normal replication
```



### Why it is useful

Leader election is relatively rare.

Normal replication is frequent.

Flexible Paxos moves cost toward the rare phase to reduce frequent-path latency.

### Common mistakes

* Requiring only phase-2 quorums to intersect each other.
* Choosing `Q₁ + Q₂ ≤ N`.
* Forgetting that leader replacement becomes less available with larger `Q₁`.
* Assuming a stable leader can survive forever without re-election.

### Recall questions

1. Which quorum sets must intersect?
2. Why can phase-2 quorums be mutually disjoint?
3. What availability cost follows a large phase-1 quorum?
4. For `N = 7` and `Q₂ = 3`, what minimum `Q₁` is required?

---

# Generalised Register View of Consensus

## 24. Write-Once Registers

### Core idea

Single-decree consensus can be modelled using replicated write-once registers.

Each register is:

* Unwritten
* Written with a value
* Written with `nil`

Registers with the same index across servers form a register set. 

### Quorum states

| State         | Meaning                              |
| ------------- | ------------------------------------ |
| **Any**       | Any value may still be decided       |
| **Maybe V**   | Only `V` could eventually be decided |
| **None**      | No value can be decided here         |
| **Decided V** | `V` has been decided                 |

### Core rules

A client:

* May write only a supplied or previously read value.
* Must not allow two quorums to decide different values.
* Must not override a prior register-set decision.
* Outputs a value only after reading quorum evidence.

### Two phases

#### Phase 1

* Check the target register.
* Seal prior unwritten registers with `nil`.
* Read prior register state from a quorum.
* Choose the highest relevant nonempty value or a new input when safe.

#### Phase 2

* Write the selected value to the register set.
* Output it after quorum success.



### Why it matters

This model reframes consensus as immutable state transitions rather than interactions among named proposer and acceptor roles.

### Common mistakes

* Overwriting a write-once register.
* Inventing a value not supplied or previously read.
* Deciding from one server’s register.
* Ignoring lower register sets.

### Recall questions

1. Why are earlier unwritten registers set to `nil`?
2. Which quorum state constrains every future decision to one value?
3. How does this model correspond to proposal numbers in Paxos?

---

# Raft

## 25. Raft Mental Model

### Core idea

Raft implements consensus as a leader-managed replicated log.

```text
Client commands
    ↓
Leader log
    ↓
Follower logs
    ↓
Deterministic state machines
```

All replicas apply the same committed commands in the same order. 

### Roles

| Role          | Responsibility                          |
| ------------- | --------------------------------------- |
| **Follower**  | Responds to leaders and candidates      |
| **Candidate** | Attempts to win an election             |
| **Leader**    | Accepts commands and replicates the log |



### Terms

A **term** is a monotonically increasing leadership epoch.

```text
Term 4 → Leader A
Term 5 → Leader C
```

Messages carry their term.

A node seeing a higher term updates its own term and rejects stale authority. 

### Mental model

```text
Term:
Which leadership generation is current?

Log index:
Where does this command occur?
```

### Common mistakes

* Continuing as leader after observing a higher term.
* Comparing log indices without terms.
* Assuming one physical leader exists across every term.
* Applying uncommitted entries to externally visible state.

### Recall questions

1. What event causes a follower to become a candidate?
2. How are stale leaders identified?
3. What two identifiers locate a Raft entry?

---

## 26. Raft Leader Election

### Election trigger

A follower starts an election when it does not receive a leader heartbeat before its election timeout.

### Candidate steps

1. Increment term.
2. Vote for itself.
3. Send `RequestVote` to peers.
4. Collect a majority.
5. Become leader if successful.

```text
REQUEST-VOTE(
    candidate term,
    last log term,
    last log index
)
```

Each process votes at most once per term. 

### Log freshness rule

A follower denies its vote when its own log is more up to date than the candidate’s.

Comparison:

1. Higher last-log term wins.
2. If terms match, longer log wins.



### Why it matters

Only a candidate containing every committed entry can win a majority.

This helps ensure committed history survives leadership change.

### Split vote

Several candidates may divide the votes so no majority forms.

Raft uses randomised election timeouts to reduce repeated ties. 

### Common mistakes

* Granting several votes in one term.
* Electing a candidate with an outdated log.
* Using identical election timeout values on every node.
* Treating timeout expiry as proof of old-leader failure.

### Recall questions

1. Why does `RequestVote` include last-log information?
2. How do randomised timers resolve split votes?
3. Can two candidates each win a majority in one fixed membership and term?

---

## 27. Raft Log Replication

### Append process

1. Leader appends client command locally.
2. Leader sends `AppendEntries` to followers.
3. Followers persist matching entries.
4. Leader receives majority acknowledgments.
5. Entry becomes committed.
6. Commit position is propagated to followers.

 

### `AppendEntries` includes

* Leader term
* Previous log index
* Previous log term
* New entries
* Leader commit index

### Commit rule

```text
Entry replicated to majority
    +
leader’s commitment rule satisfied
    ↓
Entry committed
```

Committing entry `K` also commits its valid preceding log prefix.

### Ordering invariant

If two logs contain the same term and index, they contain:

* The same command at that position
* The same preceding history



### Common mistakes

* Accepting an entry despite a mismatched predecessor.
* Committing after only local append.
* Applying commands outside log order.
* Reusing one term/index pair for different commands.
* Assuming follower persistence means global commitment.

### Recall questions

1. Why is the predecessor term and index included?
2. Which acknowledgment count permits commitment?
3. What does the log-matching property guarantee?

---

## 28. Raft Retry and Deduplication

### Core idea

Leaders retry messages to slow or unavailable followers.

Because entries have stable identities, followers can detect duplicates and preserve log order. 

### Mental model

```text
Repeated transmission
    ≠
repeated logical append
```

### Client uncertainty

Raft never reports an uncommitted entry as committed.

However, a committed command may temporarily appear unresolved to a client because the response was lost or delayed.

The client must retry using application-level request deduplication when duplicate side effects matter.

### Common mistakes

* Assuming log deduplication automatically deduplicates client business operations.
* Returning failure for a command that may already be committed.
* Assigning a new logical operation ID to every retry.
* Applying an entry twice to a non-idempotent state machine.

### Recall questions

1. Why are repeated `AppendEntries` safe?
2. Why can a client still be uncertain after a timeout?
3. Which deduplication state belongs above the Raft log?

---

## 29. Raft Log Repair

### Core idea

A new leader finds the highest prefix it shares with each follower.

It then:

1. Removes conflicting uncommitted follower suffixes.
2. Sends the leader’s entries from the common point.
3. Restores one consistent log prefix.



### Mental model

```text
Follower:
[A, B, X, Y]

Leader:
[A, B, C, D]

Common prefix:
[A, B]

Repair:
remove X, Y
append C, D
```

### Important rule

Committed entries cannot be removed.

Only uncommitted conflicting suffixes may be overwritten.

### Raft guarantees

* At most one valid leader per term
* Leader appends rather than reorders its log
* Committed entries survive into future leaders
* Term/index identifiers are never reused for different entries



### Common mistakes

* Preserving an uncommitted follower suffix over the leader’s authoritative history.
* Deleting committed entries.
* Repairing from the first difference without verifying the common prefix.
* Treating every local persisted entry as committed.

### Recall questions

1. Which follower entries may be discarded?
2. Why must a new leader contain all committed entries?
3. How is the common log prefix found?

---

# Byzantine Consensus

## 30. Byzantine Fault Model

### Core idea

Crash-fault consensus assumes participants follow the protocol until they stop.

Byzantine participants may:

* Lie
* Send contradictory messages
* Forge or alter state
* Collude
* Return incorrect results
* Continue operating maliciously

Byzantine behaviour may also result from bugs, corruption or incompatible software. 

### Why it is harder

```text
Crash fault:
missing evidence

Byzantine fault:
potentially false evidence
```

Participants must cross-validate messages instead of trusting one leader or one response.

### Communication cost

Many Byzantine protocols require all-to-all communication during key phases, producing approximately quadratic message growth.

### Common mistakes

* Treating authenticated identity as proof of truthful content.
* Applying crash-fault quorum sizes to Byzantine systems.
* Assuming Byzantine behaviour must be intentional.
* Trusting one replica’s recovery state.

### Recall questions

1. Why is a malicious response harder to tolerate than silence?
2. Can cryptographic authentication prove that a value is correct?
3. Why is cross-validation required?

---

# Practical Byzantine Fault Tolerance

## 31. PBFT Fault Threshold

### Core idea

To tolerate `f` Byzantine replicas:

```text
N ≥ 3f + 1
```

No more than approximately one-third of replicas may be faulty. 

### Example

```text
f = 1
N = 4

f = 2
N = 7
```

### Why `3f + 1`?

The system must distinguish:

* Up to `f` malicious responses
* Up to `f` missing or unavailable responses
* At least `f + 1` matching correct responses

Correct responses must outnumber plausible faulty agreement.

### Views

Each PBFT view has:

* One primary
* Remaining replicas as backups

Primary selection can be derived from:

```text
primary = view_number mod N
```

### Client completion

The client waits for `f + 1` matching final responses.

At least one of those responses must come from a correct replica. 

### Common mistakes

* Using `2f + 1` total nodes for Byzantine tolerance.
* Trusting one primary response.
* Counting mismatching replies toward completion.
* Assuming every nonresponse is malicious.

### Recall questions

1. How many replicas are needed to tolerate two Byzantine faults?
2. Why are `f + 1` matching client responses meaningful?
3. How does PBFT rotate primaries?

---

## 32. PBFT Phase 1: Pre-Prepare

### Core idea

The primary assigns an ordered request identifier and broadcasts:

* View ID
* Sequence/message ID
* Client payload
* Payload digest
* Signature or authentication evidence



### Backup validation

A backup accepts only if:

* The view is current.
* The message comes from the current primary.
* The request identifier is valid.
* The calculated digest matches.
* The request has not been tampered with.

### Digest role

```text
Payload
    ↓ cryptographic hash
Digest
```

The digest provides a compact identity for later cross-validation.

### Common mistakes

* Accepting a pre-prepare from a stale view.
* Comparing only message IDs and not digests.
* Using a noncollision-resistant checksum for adversarial validation.
* Assuming a valid signature proves the operation is semantically valid.

### Recall questions

1. Which fields identify an ordered PBFT request?
2. Why is the digest later sufficient for cross-validation?
3. What must a backup verify before entering prepare?

---

## 33. PBFT Phase 2: Prepare

### Core idea

Each accepting backup broadcasts a `PREPARE` containing:

* View
* Sequence ID
* Payload digest

The full payload need not be resent.

A replica advances after receiving enough matching prepares. The text describes collecting `2f` matching prepare messages from distinct backups in addition to the corresponding pre-prepare evidence. 

### Mental model

```text
Primary says:
“Request X occupies sequence K.”

Backups cross-check:
“We observed the same X at K.”
```

### Purpose

Prepare establishes that a sufficiently large set agrees on the request identity and position.

### Common mistakes

* Counting prepare messages with different digests.
* Counting duplicate messages from one replica.
* Rebroadcasting the full payload unnecessarily.
* Advancing before the required matching evidence exists.

### Recall questions

1. What agreement is established during prepare?
2. Why are prepare messages broadcast to every replica?
3. Why must messages come from distinct replicas?

---

## 34. PBFT Phase 3: Commit

### Core idea

Prepared replicas broadcast `COMMIT`.

A replica commits after collecting:

```text
2f + 1 matching commit messages
```

possibly including its own. 

### Why two replica rounds?

| Prepare                              | Commit                                                       |
| ------------------------------------ | ------------------------------------------------------------ |
| Establishes common ordering evidence | Establishes that enough replicas know the ordering is stable |
| Cross-validates primary assignment   | Makes the decision survive view changes                      |
| Detects equivocation                 | Creates durable shared commitment knowledge                  |

### Client response

After execution, replicas respond directly to the client.

The client requires `f + 1` matching results.

### Mental model

```text
Pre-prepare:
primary proposes position

Prepare:
replicas agree on proposal identity

Commit:
replicas agree that the agreement is durable enough to execute
```

### Common mistakes

* Executing after only pre-prepare.
* Treating `2f + 1` arbitrary responses as sufficient when they disagree.
* Returning to the client from only the primary.
* Applying commits in different sequence orders.

### Recall questions

1. What evidence permits a replica to commit?
2. Why is prepare alone insufficient?
3. How does the client detect a lying replica?

---

## 35. PBFT View Changes

### Core idea

Replicas suspecting an inactive or faulty primary initiate a view change.

```text
Suspect primary
    ↓
Broadcast VIEW-CHANGE
    ↓
New primary receives sufficient evidence
    ↓
Establish new view
```

The source describes the new primary acting after receiving `2f` view-change events. 

### Why view-change evidence matters

The new primary must preserve operations already prepared or committed in the old view.

A new primary cannot simply begin with an empty history.

### Read-only optimisation

A client may send a read to all replicas and complete after receiving `2f + 1` matching responses, once relevant prior changes are committed. 

### Common mistakes

* Switching primary based on one suspicion.
* Starting a new view without preserving prior prepared requests.
* Serving reads from one potentially Byzantine replica.
* Allowing old-primary messages after view change.

### Recall questions

1. What triggers a PBFT view change?
2. Which history must the new primary preserve?
3. Why do read-only requests need several matching replies?

---

## 36. PBFT Checkpointing

### Core idea

PBFT replicas retain protocol messages for recovery, but the log cannot grow indefinitely.

Every configurable number of requests:

1. Compute a state digest.
2. Associate it with the latest executed sequence number.
3. Broadcast checkpoint evidence.
4. Collect `2f + 1` matching responses.
5. Mark the checkpoint stable.
6. Discard covered protocol messages.



### Why a digest is required

A recovering node cannot trust one replica’s state.

Matching digests provide evidence that a sufficiently large set agrees on the checkpoint state.

### Mental model

```text
State at sequence K
    +
2f + 1 matching digest evidence
    =
stable checkpoint
```

### Common mistakes

* Trusting one checkpoint provider.
* Deleting logs before checkpoint stability.
* Comparing state digests from different sequence numbers.
* Restoring bytes without validating the digest.

### Recall questions

1. Why are checkpoints more important in Byzantine recovery?
2. Which messages can be discarded after a stable checkpoint?
3. What does the state digest prove?

---

# Consensus Comparison

## 37. Protocol Mental Models

| Protocol                | Core mental model                                            |
| ----------------------- | ------------------------------------------------------------ |
| **ZAB**                 | Epoch leader broadcasts a totally ordered history            |
| **Single-decree Paxos** | Choose one value for one immutable slot                      |
| **Multi-Paxos**         | Stable proposer appends values to ordered slots              |
| **Fast Paxos**          | Direct proposer-to-acceptor fast path with larger quorums    |
| **EPaxos**              | Order only interfering commands through dependencies         |
| **Flexible Paxos**      | Require intersection across phases, not every quorum pair    |
| **Raft**                | Elected leader replicates an ordered log by terms            |
| **PBFT**                | Replicas cross-validate ordering despite malicious behaviour |

### Main trade-offs

| Design choice         | Benefit                          | Cost                                        |
| --------------------- | -------------------------------- | ------------------------------------------- |
| Stable leader         | Low normal latency               | Bottleneck and election pause               |
| Leaderless initiation | Better load distribution         | Dependency/conflict complexity              |
| Larger quorum         | Stronger alternate path          | Lower availability and more messages        |
| Flexible quorum       | Cheap frequent phase             | Expensive rare phase                        |
| Byzantine validation  | Arbitrary-fault tolerance        | More replicas and quadratic messages        |
| Direct fast path      | Fewer normal hops                | Collision recovery                          |
| Total ordering        | Simple state-machine replication | Orders independent operations unnecessarily |

---

## 38. Paxos vs Raft

| Paxos / Multi-Paxos                                 | Raft                                                |
| --------------------------------------------------- | --------------------------------------------------- |
| Described through proposers, acceptors and learners | Described through leaders, candidates and followers |
| Per-slot proposal numbers                           | Terms and log indices                               |
| Stable proposer is an optimisation in Multi-Paxos   | Leadership is central to the design                 |
| Safety follows accepted-proposal preservation       | Safety follows election and log-matching rules      |
| Many possible protocol variants                     | More prescriptive decomposition                     |
| Often difficult to explain consistently             | Designed for understandability                      |

### Shared core

Both rely on:

* Intersecting quorums
* Monotonic leadership generations
* Stable replicated logs
* Rejection of stale authority
* Majority-based progress
* Deterministic state-machine application

### Common mistake

Treating differences in terminology as fundamental differences in the agreement problem.

---

## 39. Atomic Broadcast vs Consensus

### Relationship

```text
Consensus:
agree on one value

Repeated consensus:
agree on value for slot 1
agree on value for slot 2
agree on value for slot 3
    ↓
totally ordered replicated log
    ↓
atomic broadcast
```

Atomic broadcast and consensus are equivalent under standard asynchronous crash-failure assumptions. 

### Difference in presentation

| Consensus             | Atomic broadcast           |
| --------------------- | -------------------------- |
| Decision-oriented     | Message-delivery-oriented  |
| Chooses values        | Delivers ordered events    |
| One or repeated slots | Continuous stream          |
| Agreement on outcome  | Agreement on set and order |

### Recall questions

1. How can repeated consensus create atomic broadcast?
2. How can atomic broadcast be used to solve consensus?
3. Which additional concern appears when operating many log slots continuously?

---

## 40. Consensus Rules to Retain

* Consensus requires agreement, validity and termination.
* Safety can hold while liveness temporarily fails.
* Reliable broadcast agrees on delivered messages; atomic broadcast also agrees on order.
* Leadership election is not sufficient without history synchronisation.
* Paxos phase 1 discovers constraints from earlier accepted proposals.
* Paxos phase 2 chooses a value through an acceptor quorum.
* One acceptor’s acceptance is not necessarily a chosen value.
* Quorum intersection carries prior decision evidence into later rounds.
* Multi-Paxos amortises leader establishment across many log entries.
* Leases optimise reads but depend on bounded timing assumptions.
* Fast paths improve uncontended latency but add collision-recovery cost.
* EPaxos orders conflicting commands through dependencies.
* Flexible Paxos shifts quorum cost between protocol phases.
* Raft candidates must have sufficiently up-to-date logs.
* Raft followers may discard only conflicting uncommitted suffixes.
* Byzantine tolerance requires more replicas and cross-validation.
* PBFT requires at least `3f + 1` replicas to tolerate `f` Byzantine faults.
* Stable checkpoints must be proven before protocol logs are truncated.

---

## 41. Common Conceptual Mistakes

* Saying “consensus is impossible” instead of recognising FLP’s termination limitation.
* Confusing an accepted proposal with a chosen proposal.
* Treating leader election as the complete consensus protocol.
* Assuming a majority means every replica has the latest state.
* Using local reads from a leader without confirming current authority.
* Claiming a fast path improves performance without measuring collisions.
* Applying commands before they are committed.
* Truncating consensus logs before a matching snapshot is durable.
* Allowing deterministic replicas to use uncontrolled randomness or time.
* Using crash-fault consensus in a Byzantine environment.
* Counting authenticated malicious responses as correct.
* Treating every timeout as evidence that a new leader is safe.

---

## 42. Applied Recall Questions

1. A proposer receives promises reporting accepted proposals `(4, A)` and `(7, B)`. Which value must it propose, and why?
2. One acceptor stored value `A`, but no quorum accepted it. May a later proposer choose `B`?
3. Two proposers continuously pre-empt each other with higher proposal numbers. Which property remains safe, and which property suffers?
4. Why can Multi-Paxos skip phase 1 for ordinary log entries?
5. A lease-holding leader’s clock runs slowly while follower clocks expire the lease. Which guarantee is threatened?
6. `N = 5`, `Q₁ = 3`, `Q₂ = 2`. Is this Flexible Paxos configuration safe?
7. Two EPaxos replicas report different dependencies. Which execution path is required?
8. A Raft candidate has a longer log but a lower last-log term than a follower. Should the follower vote for it?
9. A follower contains an uncommitted suffix conflicting with a new leader. Which entries may be removed?
10. Why can a client timeout even though its command became committed?
11. In PBFT with seven replicas, how many Byzantine replicas can be tolerated?
12. Why does the PBFT client wait for more than one matching result?
13. How does a stable PBFT checkpoint permit safe log deletion?
14. Why does atomic broadcast require more than reliable message delivery?
15. Which workload favours EPaxos over one global leader?

---

# Chapter 14 Design Summary

## 43. Core Mental Model

```text
Failure detector
    → suggests when leadership may have failed

Leader election
    → proposes an authority

Consensus
    → makes authority and decisions safe

Replicated log
    → records ordered decisions

State machine
    → turns decisions into database state
```

### Cause-and-effect chain

```text
Intersecting quorums
    → prior decisions cannot be forgotten

Monotonic epochs/terms
    → stale leaders can be rejected

Durable logs
    → decisions survive crashes

Deterministic execution
    → identical logs produce identical state
```

### Key design tension

```text
Central leader
    → efficient common path
    → bottleneck and failover pause

Decentralised proposal
    → better distribution
    → more conflict and dependency work
```

---

# Part II Conclusion

## 44. Local and Distributed Performance

### Core idea

Database performance depends on both:

* Node-local storage paths
* Cluster-wide communication and coordination

```text
Fast local storage
    +
slow distributed commit
    =
slow distributed database
```

Similarly:

```text
efficient consensus
    +
poor local storage engine
    =
limited node throughput
```

The subsystems must be designed and evaluated together. 

### Mental model

```text
Local engine determines:
How quickly one node performs work.

Distributed protocol determines:
How much coordination is required
and how well the cluster scales.
```

### Common mistakes

* Benchmarking storage without replication.
* Benchmarking consensus with empty operations.
* Treating network, logging and execution latency as independent.
* Optimising average local latency while ignoring distributed tail latency.
* Adding more nodes to a coordination bottleneck.

### Recall questions

1. Why can a fast storage engine still produce a slow database?
2. Which subsystem commonly limits maximum cluster scale?
3. Why should commit protocols be benchmarked with real storage work?

---

## 45. Distributed Algorithm Toolbox

| Algorithm class              | Core responsibility                        |
| ---------------------------- | ------------------------------------------ |
| **Failure detection**        | Suspect unavailable processes              |
| **Leader election**          | Select a temporary coordinator             |
| **Dissemination**            | Spread information among peers             |
| **Anti-entropy**             | Detect and repair replica divergence       |
| **Distributed transactions** | Apply multipartition operations atomically |
| **Consensus**                | Agree on durable values and ordering       |



### Relationship map

```text
Failure detection
    ↓ supports
Leader election
    ↓ supports
Consensus normal-path efficiency

Dissemination
    ↓ spreads
membership, configuration and updates

Anti-entropy
    ↓ repairs
missed or divergent replica state

Consensus
    +
atomic commitment
    ↓ enables
fault-tolerant distributed transactions
```

### Important distinctions

| Pair                          | Difference                                                   |
| ----------------------------- | ------------------------------------------------------------ |
| Failure detector vs proof     | Suspicion vs certainty                                       |
| Leader election vs consensus  | Candidate selection vs safe authority                        |
| Replication vs consensus      | Copying values vs agreeing on values/order                   |
| Consensus vs 2PC              | Choosing a durable value vs unanimous transaction commitment |
| Dissemination vs anti-entropy | Normal propagation vs later repair                           |
| Atomic broadcast vs gossip    | Total ordered delivery vs probabilistic spread               |

---

## 46. Holistic Design Questions

Before selecting a distributed database design, ask:

1. **Failure model**

   * Crash-stop, crash-recovery or Byzantine?
2. **Consistency**

   * Linearizable, serializable, causal, session or eventual?
3. **Transaction scope**

   * Single-key, single-shard or cross-shard?
4. **Data placement**

   * Range, hash or consistent-hash partitioning?
5. **Replication**

   * Leader-based, quorum-based or conflict-resolving?
6. **Availability**

   * Which operations may fail during partitions?
7. **Recovery**

   * How are logs, snapshots, hints and versions reconciled?
8. **Workload**

   * Point reads, scans, high ingest or large transactions?
9. **Cost placement**

   * Foreground coordination, background maintenance or read repair?
10. **Scale**

* How do messages, metadata and rebalancing grow with node count?

### Mental model

```text
Correctness requirements
    determine
coordination requirements

Workload characteristics
    determine
where coordination cost should be paid
```

---

## 47. Final Relationships

### Storage and replication

```text
B-Tree or LSM Tree
    → stores local ordered state

WAL
    → protects local durability

Consensus log
    → orders and replicates distributed decisions

State machine
    → applies consensus entries locally
```

### Consistency and coordination

```text
Stronger global ordering
    → more quorum communication
    → higher latency
    → lower partition availability
```

### Availability and repair

```text
Accept writes with less coordination
    → replicas may diverge
    → anti-entropy and conflict resolution required
```

### Partitioning and transactions

```text
More partitions
    → more capacity and parallelism

Transaction touches more partitions
    → more coordination and failure exposure
```

### Immutability and versioning

```text
Retain old versions
    → nonblocking snapshots and repair
    → storage and garbage-collection cost
```

### Leadership and performance

```text
Stable leader
    → efficient ordering
    → concentrated load

Distributed command leadership
    → load distribution
    → dependency and conflict complexity
```

---

## 48. Final Applied Recall Questions

1. Design a replicated key-value store that requires linearizable writes but tolerates stale analytical reads. Which protocols can be separated?
2. A system has fast local writes but poor cross-shard transaction latency. Which coordination layers should be measured?
3. A minority partition continues accepting writes under an old leader. Which authority and fencing rules are missing?
4. A leader election succeeds, but the winner lacks one committed log entry. Which election invariant failed?
5. A database uses `R + W > N` but returns stale values after partial writes. Which assumptions beyond set overlap are missing?
6. An eventually consistent store drops tombstones before a disconnected replica returns. What failure can occur?
7. A replicated state machine applies the same log but reaches different states. Which hidden source of nondeterminism should be investigated?
8. A transaction touches ten shards rather than one. How do availability and latency change?
9. Why can anti-entropy repair replication but not automatically restore violated business invariants?
10. When is blocking during a partition safer than continuing independently?
11. Which workloads justify a global leader, and which may benefit from per-partition or per-command leadership?
12. Why must recovery protocols themselves tolerate repeated crashes?
13. How do durable request IDs connect client retries with consensus logs?
14. Which costs are deferred when a system optimises foreground write latency?
15. Why should database architecture be evaluated as one interaction of storage, networking, consistency, recovery and workload rather than isolated algorithms?

END OF DATABASE INTERNALS