## Transactions

Problem: many things can go wrong with data systems

1. Hardware/software failures
2. Network interruptions
3. Client race conditions/partial updates

Implementing fault tolerance is painful

Transactions have been the way to simplify these issues.
- Group reads and writes as a logically unit, executed as one operations
- Either entire tx succeeds, or fails. 

Transactions are an abstraction layer. Provides safety guarantees

## Slippery concept of a tx
Have advantages and limitations.

## Meaning of ACID

Atomicity - cannot be broken down into smaller parts
- refers to all or nothing change

Consistency

Isolation
Concurrently executing tx isolated from one another -> Serializability -> each tx can pretend it is the only one running because after all tx committed, result is same as if they had run serially

Durability
Once a tx is committed, never forgotten even if there is hardware fault

A tx is understood as a mechanism for grouping multiple ops on multiple objects into 1 unit of execution


## Weak Isolation Levels
Serializable isolation should make dev life easier but has a performance cost
Tradeoff is to use weaker levels of isolation which protects against some concurrency issue

"Rather than blindly relying on tools, we need to develop a good understanding of the kind of problems we have and how to prevent them"


## Read Committed
Most basic level.
Makes 2 guarantees
1. You will only read committed data (No dirty reads)
2. You will only overwrite committed data (No dirty writes)

Dirty writes prevented by row-level locks. Lock until tx committed or aborted

This approach is very slow, force everyone to wait on locks

Another solution
For every object written, old committed value and new value set by the tx that holds the write lock
While tx is ongoing, any other tx that read objects simply given old value
Only when tx completes will the old value switch

## Snapshot Isolation and Repeatable Read

Nonrepeatable read/read skew -> when reading data has not been updated

Each tx reads from a consistent snapshot of the DB -> very good for long runnign reado nly queries

### Implementing snapshot isolation
Use write locks to prevent dirty writes, but do not lock on read.
Readers never block writers, writers never block readers -> No lock contention

Readers use tx ID to decide which objects can be seen

An object  can be seen if
1. When read tx start, object has been committed
2. Object not marked for deletion when read tx start

## Indexes and snapshot isolation

one option is for index to point to all version of object + index query to filter out invisible objects.

GC will remove old object versions and corresponding indexes can be removed

Implementation details determine perf of multi-version currency control
 - B trees: append only, copy on wrte

 ## Preventing lost updates
 If two apps read and write data concurrently, one modification will be lost.

### Cursor Stability
 DBs provide atomic writes to work around this issue
Implemented by taking an ex-lock on the object when read so no other tx can read it until update has taken place = cursor stability

### Explicit Locking
If DB don't lock, app can lock. Then app can do the read-modify-write cycle

### Automatic detection of lost updates
Atomic ops + locks = prevent lost update by forcing sequential operations.

## Compare-and-set
If old value doesnt match expected old value, don't write

### Conflict resolution and replication
Since data can be replicated and modified concurrently, additional steps need to be taken to prevent lost updates

Compare and set assume there is one node with the most up to date copy. But with leaderless replication, there is no guarantees

Common approach is to create several conflicting versionso f a value and to use application code/special data structures to resolve and merge versions

### Write Skew and Phantoms
write skew
both read the same object and write at the same time, thus breaking some assumption

Options are limited with write skew
- atomic single object operations don't help, as multiple objects are involved
- automatic detection of lost updates does not help. Requires true serializable isolation
- If can't use serializable isolation level, second-best option is to explicitly lock the rows that the tx relies on

Write skew generally follow a similar pattern
1. A select query checks if a requirement is satisfied
2. Depending on 1, app decides how to proceed
3. A tx is committed which changes condition 1

This is called a phantom.

#### Materializing conflicts
If the problem is that there is no object to which we can attach the locks, could we introduce a lock onto the DB

Take a phantom and turn it into a lock conflict on a set of rows that exist in DB.
Last resort. Serializable isolation is much more preferable.

## Serializability
Serializable isolation is regarded as the strongest isolation level. Guarantees that even though tx may execute in parallel, end result in the same as if they had been execute one at a time serially.

Why isn't anyone using this? Look at options
1. Literally execute serially
2. Two phase locking
3. Optimistic concurrency control

### Execute serially
Previously, multi thread concurrency considered essential. What changed?
1. RAM became cheap enough that we can keep dataset in memory. Faster than loading from disk

2. DB designers realize that OLTP tx are short and only make a small number of reads and writes. By contrast, long-running analytic queries are read only which can be run on a consistent snapshot

Throughput limited to a single CPU core.

### Two Phase Locking 2PL
Tx are allowed to concurrently read the same object as long as no one is writing, but once someone needs to write, exclusive access needed

Writers block everyone

Blocking implemented by having a lock on each object
Lock can either be in shared or exclusive mode

1. If tx wants to read, must first acquire the lock in shared mode
    Multiple tx can hold in lock in shared mode simultaneously, but if another tx has an ex-lock on the obj, tx must wait

2. If a tx wants to write an object, it must first acquire the lock in ex mode.

3. If read, then write, can upgrade shared lock to ex lock.

4. After tx has lock, must hold lock till commit/abort. Hence 2PL.
    1 phase = get lock
    2 phase = release lock

Due to this, can have deadlock. DB automatically detects deadlocks 

#### Perf of 2PL
Shit performance. Too much overhead acquiring and releasing locks. 

### Predicate locks
Instead of belonging to one object, it belongs to all objects that matches some saerch conditions

1. If tx wants to read WHERE condition, must acquire a shared-mode predicate lock on the conditions of the query. If another tx B has an ex lock on the object matching those conditions, A must wait

2. If tx wants to write, must check if old/new value matches any predicate locks. If there is lock held, A must wait.

## Index-range locks
Predicate locks do not work well. Index-range locks are used
simplify a predicate by matching a greater set of objects.

Less precision, much lower overheads = good compromise

## Serializable Snapshot Isolation
Implementations of serializability either
1. do not perform well
2. do not scale well

Weak isolations that have good perf but prone to race conditions

SSI provides full serializability and only has a small performance penalty

### Pessimistic vs Optimistic concurrency control

2PL is pessimistic because it is based on the principle that if anything goes wrong, it's better to wait until situation is safe before doing anything. 

Serial exection is pessimistic to the extreme. Locking entire DB.

SSI is optimistic. Instead of blocking, tx continue anyway in the hopes that everything turns out fine. 

When a tx wants to commit, db checks if anything bad happened and if so, action is aborted  and has to be retried.

Performs badly under high contention because leads to high number of tx needing to abort

SSI based on SI + algo for detecting seralization conflicts

### Decisions based on an outdated premise
In order to provide serializable isolation, db must detect situations in which a tx may have acted on an outdataed premise and abort the tx in that case.

2 cases to consider
1. Detect stale MVCC object version
2. Detect writes that affect prior reads

For case 1:
DB needs to track when a tx ignores another tx writes due to MVCC visibility rules. When the tx wants to commit, db checks whether any of the ignored writes have been committed. If so, tx must be aborted

Why wait until committing?
At point of reading, db doesnt know if a write is incoming. The previous tx may abort or may still be uncommitted at the time which will lead to the read not being stale.

For case 2:
Can use a technique in 2PL, but don't block other tx
a. DB remembers what rows are being accessed.
b. When a tx writes, must look for indexes that other tx may have recently read. These txes are then notified