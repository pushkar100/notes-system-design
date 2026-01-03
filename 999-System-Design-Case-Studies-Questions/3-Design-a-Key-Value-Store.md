# Distributed Key-Value Store

A key-value store, also referred to as a key-value database, is a non-relational database. Each unique identifier is stored as a key with its associated value. This data pairing is known as a “key-value” pair.

In a key-value pair, the key must be unique, and the value associated with the key can be accessed through the key. Keys can be plain text or hashed values (Takes `O(1)` to retrieve a value). For performance reasons, a short key works better. What do keys look like? Here are a few examples:

- Plain text key: `“last_logged_in_at”`
- Hashed key: `253DDEC4`

**Why are hashed keys more performant?** Keys are often stored in *RAM (memory)* for speed. *Shorter keys take up less space* and are faster for the CPU to process, making the system *cheaper* and *faster*
- If you have 100 million records, and your key is 1 KB long, you need 100 GB of RAM just to store the keys (ignoring the values)
- If you shorten that key to 10 bytes, you only need 1 GB of RAM
- ***Hashing Speed*** (***CPU***): Every time you read or write, the database must run a hash algorithm on the key. Hashing a very long string takes more CPU cycles than hashing a short one. High-throughput systems need to save every microsecond
- ***Network Overhead***: If you are doing 100,000 requests per second, sending a massive key string in every request packet saturates the network bandwidth unnecessarily

The value in a key-value pair can be strings, lists, objects, etc. The value is usually treated as an opaque object in key-value stores, such as Amazon dynamo, Memcached, Redis, etc

**Why NoSQL?** A distributed Key-Value (KV) store is classified as NoSQL because it fundamentally abandons the "Relational" model (rows, columns, and tables linked by relationships) in favor of a ***simpler, highly scalable model designed for distributed systems***.

Also, a relational database stores data as **tables** i.e known structure but a key-value pair usually contains a ***blob*** as the value i.e *opaque* (Database does not know the structure of the value - can be a list, image, set, string, etc)

In short: A *Relational Database Management System (RDBMS) prioritizes **structure and consistency***, whereas a *Distributed KV Store prioritizes **scale** and **partition tolerance***

(Scaling a relational database is harder as it breaks ACID properties while NoSQL databases are built for scale and they are not strictly ACID compliant as a reason)

**Real-world use cases**
- *Caching*: Store computed results (like rendered pages or API responses) for fast retrieval
- *Session management*: Maintain user session data in distributed web apps
- *Configuration data*: Store dynamic settings for micro-services
- *Leaderboards or counters*: Keep track of rankings or statistics

## Hard parts

1. **Data Partitioning**: How to split 100TB of data across 1,000 nodes evenly?
2. **Replication & Consistency**: How to ensure data safety without sacrificing write speed? (The CAP Trade-off)
3. **Conflict Resolution**: If Node A and Node B receive different updates for "Key1" at the same time, who wins?
4. **Node Failures**: Detecting and managing churn (nodes dying/joining constantly)

## Functional requirements

Questions to ask:
1. Who/What: 
	- *Who accesses this key-value store? A client. Could be any! ***No formal FR required***
	- *What type of values can be set for a key?*
2. What/How: *What type of operations must it support?* *How do we set and get values?*
3. Result: *What happens when a key is set or read?* *Ans: Value returned*. ***No formal FR required***
4. Where/Scale: *Where is the key-value store going to be? Within a system or separate?*

### Functional Requirements as a result of these questions

1. Value can be anything i.e a blob
2. Support setting a key-value pair and fetching the value using a key
3. It is a separate system that clients can use / integrate with

## Non-functional requirements

Questions to ask:
1. Scale (Throughput) and Volume (Overall storage or Bandwidth): 
	- *What is the scale at which this KV system is going to work at?*
	- *Storage size? Are we storing TBs or PBs?* 
2. Speed (Latency/CPU/Memory):
	- *What is the speed at which a key must be fetched? Are we prioritizing key read or write?*
3. CAP trade-off (AP vs CP):
	- *Do we need the system to be consistent or highly available?*
4. Safety net (Fault Tolerance):
	- *How fault tolerant does it need to be in terms of (a) network partitions and (b) node failures?*
5. Geography:
	- *Does it matter if a client uses it from a particular region? Ex: Asia*

### Non-Functional Requirements as a result of these questions

1. Millions of requests per second (Ex: 10M reqs / sec) and TBs of storage
	- Note: Whenever there are high frequency of requests, we need to take care of 
		- 1. The "celebrity" / "hot key" problem
		- 2. If millions of writes, 
2. Respond in single digit milliseconds (Ex: 5ms). Optimise key reads (assuming more people read the value than write it) i.e ***Read heavy***
3. System must be available but with a tunable consistency
4. Extremely fault tolerant in case of network partitions and node failures (Reliable) 
5. Global KV store: Region should not matter - all users should be serviced the same

## Core entities 

Objects and users persisted in the database or exchanged via APIs:
- Key
- Value

## System interface or API endpoints

System interface or API? A system interface makes sense since a client will generally use a library to integrate with a key-value store to perform CRUD on key-value pairs

1. **`put(key, value) --> returns success/failure`**
2. **`get(key) --> returns the value`**

## High Level Design 

1. Starting with MVP: In this case, it is not client-server but a `client-kvstore` model
2. Go through *Functional Requirements* one by one and see if it is fulfilled. If not, modify design  (***Note: Need not go in order***)

Step 1:
```
 [client] -> [KV store system]
```

Step 2:
- *FR#1*: Yes, we can store anything in the system as a value
- *FR#2*: Apply `put` and `get` methods to the system
- *FR#3*: Already represented as a separate system
```
 [client] <--- get(key), put(key, value) ---> [KV store system]
```

## Deep dive

1. Starting with constructed High Level Design:
2. Go through *Non-Functional Requirements* one by one and see if it is fulfilled. If not, modify design (***Note: Need not go in order***)
3. Also check some unique constraints of the problem you might have missed (scan or read the problem again) and apply solutions for them

*NFR#1*: Millions of requests per second (Ex: 10M reqs / sec) 
- Problem: Insane frequency of writes and reads
	- We need to scale! Horizontal is better (cheaper, reliable). We can have different nodes for the system (**Replication** for a distributed system)
	- How to handle high writes?
		- Leader based? 
			- All the writes go to just one node (overloading, bottleneck). Not too scalable
			- If write goes down there will be (*Check the Availability NFR*):
				1. Panic: Delay in electing a new leader (Not available immediately)
			- Summary of cons: 
				- Leader failure requires failover handling
				- Asynchronous replication may lead to temporary inconsistency
		- **Leader-less replication**: Choose this!
			- Pros: Any node can accept writes, Zero downtime when a node goes down (Ex: no master)
			- Cons: **Write conflicts** will have to be dealt with
	- How to handle high reads?
		- Replication should solve it
	- How to handle high scale in general? That is, make sure one node is not overloaded (Always written to or read from, based on the key)
		- Hash the keys in such a way that it is consistently hashed on a ring (**Consistent Hashing**) so that even if nodes are added or removed, we minimise data movement
			- Problem: `Hash(key) % N` breaks when N changes (nodes join/leave)
			- Solution: Consistent Hashing (The Ring)
			- Treat the hash space (e.g., `0` to `2^160`) as a circle
			- Nodes are placed on the circle
			- A Key is hashed to a point; we walk clockwise to find the first node (Owner)
			- Optimization (Virtual Nodes): One physical server = 100 virtual positions on the ring. This balances load if one server is more powerful than others

*NFR#2*: *Respond in single digit milliseconds & is read-heavy*
- Write heavy workloads? Use LSMTrees (SSTables) with Memtables (in memory) and perhaps Bloom filters (Insanely fast writes, poor read speed especially for range queries
	- Storage Engine: LSM Trees (Log Structured Merge Trees)
		- Problem: Random disk writes are slow
		- Solution: Convert random writes to sequential writes
		- Write to WAL (Append-only log for crash recovery)
		- Write to MemTable (In-memory sorted structure)
		- When MemTable is full -> Flush to SSTable (Immutable file on disk)
		- Compact SSTables periodically
- However, ours is a **ready-heavy workload** :
	- **Solution**: Use a B-tree index based database (Indexing improves read speed, especially for range queries)
		- Index is stored in memory (A B-Tree data structure)
		- However, we can go one step ahead and have an in-memory cache (LRU? Choose a cache strategy) with snapshots. Use periodic snapshots to sync with the database (persistence)
			- What if node crashes? KV pairs in memory are lost but whatever was saved until snapshot can be restored or is durable (Availability)

*NFR#3* and *NFR#4*: *System must be available but with a tunable consistency* and *Extremely fault tolerant in case of network partitions and node failures (Reliable)*
- High availability: We are already decided on a **leaderless replication**
- **Consistency**: Since it needs to be ***tunable***, leaderless replication makes sense. Using it we can employ a **quorum**
- Fault tolerance: 
	1. **Failure detection**
		- Problem: Node A is down. How do we know?
		- Solution: **Gossip protocols**. Periodically, every node in the cluster selects a few random peers and exchanges information about itself and other nodes it knows about
		- Instead of broadcasting updates to the entire network at once—which would overwhelm the network bandwidth—this decentralized, peer-to-peer sharing ensures that updates (like a "heartbeat" status, a node failure, or a schema change) propagate exponentially across the system
		- This method is highly fault-tolerant and scalable because it doesn't rely on a central master to distribute state; even if several nodes fail, the information eventually flows around them to reach every healthy node, ensuring the entire cluster reaches a consistent view (convergence) very quickly
	2. **Handling Failures of a node: Hinted Handoff & Gossip**
		- Problem: Node A is down. Writes to it fail
		- Solution: Hinted Handoff (**Sloppy Quorum**)
		- The ***Coordinator*** (one node is chosen to be it) sends Node A's data to Node B (temporarily) with a note: "This belongs to A
			- When A comes back online, B pushes the data back
		- **Gossip Protocol**: Nodes randomly talk to each other every second ("I'm alive", "Node A is dead") to maintain the cluster state map
		3. **Write conflicts**
			- Since the replication is leaderless, how do we handle write conflicts since any node can accept writes now?
			- Solution: **Vector Clocks**
				- Problem: Two users update Key X on different nodes at the same time
				- Solution: Vector Clocks
				- Attach a counter for each node involved: `{NodeA: 1, NodeB: 2}`.
					- If versions are ancestors (e.g., `{A:1}` vs `{A:2}`), overwrite
					- If versions conflict (e.g., `{A:1}` vs `{B:1}`), return both to client and ask client to resolve (merge)

Additional fault tolerance considerations:
1. Recovery of a node: Use WAL + snapshot 
2. **Permanent node failure**:
	- There is a concept of **Merkle Tree/Hash**
		- A Merkle Hash (or Merkle Tree) is a verification structure where large data is broken into chunks, hashed, and then those hashes are paired and hashed again repeatedly until a single Root Hash remains
		- This Root Hash acts as a unique fingerprint for the entire dataset; if even a single byte of data at the bottom changes, that change ripples upward through the branches, altering the Root Hash instantly
		- This allows systems to efficiently check if two large databases are identical just by comparing their top Root Hash, *rather than scanning every single record*
```
       [ Root Hash ]        <-- 1. Compare this first. If it matches, STOP.
        /         \                (Data is identical)
    [Hash AB]    [Hash CD]
     /    \       /    \
  [Hash A][Hash B][Hash C][Hash D]  <-- 2. If Root differs, follow the
    |       |       |       |              path down to find the specific
  [Data A][Data B][Data C][Data D]         block that changed.
```
- Merkle Trees act as a high-speed "difference detector" for synchronization
- When a new node replaces a failed one, it compares its Merkle Tree with a healthy replica's tree to instantly identify exactly which data ranges are missing
- This allows the system to transfer only the specific missing data blocks rather than copying the entire database, drastically reducing recovery time and network usage

*NFR#5: Global KV store: Region should not matter - all users should be serviced the same*
- Along with replication within a data centre, create replicas globally. Nothing much else to do

HL architecture and consistent hashing:
```
                                 [ Client ]
                                     |
                                     | 1. put("Key1", val)
                                     v
                        +-------------------------+
                        |  Node A (Coordinator)   | <--- Client connects here
                        +-------------------------+      (Becomes Coordinator)
                               /           \
                           (Gossip)      (Route)
                             /               \
            +---(Virtual Node B1)----(Virtual Node C1)---+
           /                                              \
      +--------+                                      +--------+
      | Node B | <--- 3. Replica 1                    | Node C |
      +--------+      (Stores "Key1")                 +--------+
          |                                               |
  (Ring Logic: Hash(Key1) lands here)                     |
          |                                               |
      +--------+                                      +--------+
      | Node E |                                      | Node D | <--- 4. Replica 2
      +--------+                                      +--------+      (Stores "Key1")
           \                                              /
            +---(Virtual Node A1)----(Virtual Node B2)---+
```

Write path: Leaderless quorum
```
SCENARIO: N=3 (Replicas), W=2 (Write Quorum)

      [ Client ]
          |
          | 1. Write Request
          v
  +----------------------+
  | Node A (Coordinator) | ----------------------+
  +----------------------+                       |
      |             |                            |
      | 2. Fwd      | 2. Fwd                     | 2. Fwd
      v             v                            v
 +--------+    +--------+                    +--------+
 | Node B |    | Node C |                    | Node D |
 | (Repl) |    | (Repl) |                    | (Repl) |
 +--------+    +--------+                    +--------+
      |             |                            |
      | 3. Ack      X (Slow/Fail)                | 3. Ack
      |             X                            |
      +------------>|                            |
                    |<---------------------------+
          4. Coordinator receives 2 Acks
          (Met W=2 threshold)
                    |
          5. Success to Client
```
Not writing directly to memory within a node for the write path (performance):
```
                                  [ Client ]
                                      |
                                      | 1. put(key, value)
                                      v
                         +--------------------------+
                         |     Coordinator Node     |
                         +------------+-------------+
                                      |
              +-----------------------+-----------------------+
              |                                               |
              | 2. Append (Disk I/O)                          | 3. Write (RAM)
              v                                               v
    +-------------------+                           +-------------------+
    |    Commit Log     |                           |     MemTable      |
    | (Write Ahead Log) |                           | (In-Memory Map)   |
    +-------------------+                           +-------------------+
              |                                               |
              +-----------------------+-----------------------+
                                      |
                                      | 4. Return Success
                                      v
                                  [ Client ]
```

Node Internal Design: Read-Heavy Optimization
```
  +---------------------------------------------------------------+
  |                       SINGLE NODE ARCHITECTURE                |
  |                                                               |
  |   RAM (Fast Access)                                           |
  |   +--------------------------+    +-----------------------+   |
  |   |      LRU Cache           |    |    B-Tree Index       |   |
  |   | (Hot Keys/Values)        |    | (Map Key -> Disk loc) |   |
  |   +--------------------------+    +-----------------------+   |
  |                ^                               |              |
  |                | (Read Miss)                   | (Lookup)     |
  |                v                               v              |
  |   DISK (Persistence)                                          |
  |   +--------------------------+    +-----------------------+   |
  |   | Write Ahead Log (WAL)    |    |   Data Snapshots      |   |
  |   | (Append-only / Recovery) |    |   (Persistence)       |   |
  |   +--------------------------+    +-----------------------+   |
  |                                                               |
  +---------------------------------------------------------------+
```

Read path: More complex because the data could be in RAM or in one of many files on the disk. The system uses a Bloom Filter to avoid searching every file unnecessarily.
```
                                 [ Client ]
                                      |
                                      | 1. get(key)
                                      v
                         +--------------------------+
                         |     Coordinator Node     |
                         +------------+-------------+
                                      |
        +-----------------------------+
        |                             |
        | 2. Check Memory?            |
        v                             |
  +-----------+                       |
  | MemTable  |--[Found?]--> (Return Value)
  +-----------+                       |
        | (Not Found)                 |
        v                             |
        +-----------------------------+
        |                             |
        | 3. Check Disk?              |
        v                             |
  +--------------+                    |
  | Bloom Filter |                    |
  +--------------+                    |
        | (Maybe in SSTable 2?)       |
        v                             v
  +--------------+            +--------------+
  |  SSTable 1   |            |  SSTable 2   | <--- 4. Fetch from Disk
  +--------------+            +--------------+
                                      |
                                      | 5. Return Value
                                      v
                                  [ Client ]
```
 
Failure Handling: Hinted Handoff
```
SCENARIO: Node C is down. Node A handles the write temporarily.

     1. Write(Key1)
        (Belongs to C)
  [Coordinator A] ------------------X  [ Node C (DOWN) ]
        |
        | Node C is unresponsive!
        |
        v
  [ Local Buffer @ Node A ]
  | Key: Key1             |
  | Val: ...              |
  | Target: Node C        | <--- "Hint" attached
  +-----------------------+
        |
        | (Time passes... Gossip detects Node C is UP)
        v
  [ Handover Process ] -----------------> [ Node C (UP) ]
     (Replay Writes)                       (Store Key1) 
```

Conflict resolution: Vector clocks
```
Event: User 1 writes "Red"       Event: User 2 writes "Blue"
   Node: A                          Node: B
   Clock: [A:1]                     Clock: [B:1]

                  (Replication/Gossip)
            --------------------------------
            |                              |
            v                              v
      [ Node A ]                     [ Node B ]
   Has: "Red", [A:1]              Has: "Blue", [B:1]
   Rx: "Blue", [B:1]              Rx: "Red", [A:1]

            |                              |
            v                              v
      CONFLICT DETECTED! ([A:1] and [B:1] are not ancestors)
            |
            v
      [ Client Read ]
            |
            |<--- Returns Both Values: ["Red", "Blue"]
            |     Context: Vector Clock [A:1, B:1]
            |
      [ Client Logic ]
            | Resolves conflict (e.g. "Purple")
            v
      [ Client Write ]
      Value: "Purple"
      Clock: [A:1, B:1, C:1] (New Version overwrites previous)
```

Node Internal Design: Write-Heavy Optimization
```
   Incoming Write ("Key1", Val)
             |
             |----------------------------------+
             v                                  |
   +----------------------+                     | 1. Append (Fast Sequential I/O)
   |  Write Ahead Log     | <-------------------+    for Crash Recovery
   |      (WAL)           |
   +----------------------+
             |
             | 2. Insert (In-Memory)
             v
   +-------------------------------------------------------------+
   |  RAM (Memory)                                               |
   |                                                             |
   |  +-----------------------+      +------------------------+  |
   |  |      MemTable         |      |     Bloom Filter       |  |
   |  |  (Sorted Skip List    |      | (Quick check: "Does    |  |
   |  |   or Red-Black Tree)  |      |  key exist on disk?")  |  |
   |  +-----------------------+      +------------------------+  |
   |             |                                ^              |
   +-------------|--------------------------------|--------------+
                 | 3. FLUSH (When Full)           |
                 |    (Immutable Write)           | Used during Read
                 v                                | to skip files
   +----------------------------------------------|--------------+
   |  DISK (Storage)                              |              |
   |                                                             |
   |  Level 0:  [SSTable]  [SSTable]  [SSTable] --+              |
   |                 \         |         /                       |
   |                  \        |        /                        |
   |                   v       v       v                         |
   |                4. COMPACTION (Background Merge)             |
   |                   (Removes old/deleted keys)                |
   |                           |                                 |
   |                           v                                 |
   |  Level 1:           [   SSTable   ]                         |
   |                                                             |
   +-------------------------------------------------------------+
```

## Practical key-value stores

1. Amazon dynamo
2. Memcached
3. Redis

## Revision on tunable consistency

Problem: Balancing speed vs. accuracy. Solution: **Tunable Quorum**.
- N: Number of replicas (e.g., 3)
- W: Write Quorum (How many must confirm write)
- R: Read Quorum (How many must respond to read)

If R + W > N, you have strong consistency!
- Common Config: N=3, W=1, R=1 (Fastest, risk of stale data)
- Common Config: N=3, W=2, R=2 (Balanced)

How to configure N, W, and R to fit our use cases? Here are some of the possible setups:
- If R = 1 and W = N, the system is optimized for a fast read
- If W = 1 and R = N, the system is optimized for fast write
- If W + R > N, strong consistency is guaranteed (Usually N = 3, W = R = 2)
- If W + R <= N, strong consistency is not guaranteed
