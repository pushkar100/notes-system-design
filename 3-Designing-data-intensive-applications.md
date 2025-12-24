# Designing data intensive applications

## Table of Contents

- [Designing data intensive applications](#designing-data-intensive-applications)
  - [Reliable, Scalable, and Maintainable Applications](#reliable-scalable-and-maintainable-applications)
    - [The Big Three Pillars](#the-big-three-pillars)
    - [Reliability](#reliability)
      - [Faults vs failures](#faults-vs-failures)
      - [Categories of faults](#categories-of-faults)
      - [Why reliability matters](#why-reliability-matters)
    - [Scalability](#scalability)
      - [Describing Load (Load Parameters)](#describing-load-load-parameters)
      - [Describing Performance](#describing-performance)
      - [Approaches for coping with Load](#approaches-for-coping-with-load)
    - [Maintainability](#maintainability)
    - [Mindset](#mindset)
  - [Data models and query languages](#data-models-and-query-languages)
    - [The Power of Abstraction](#the-power-of-abstraction)
    - [Relational model versus Document model](#relational-model-versus-document-model)
      - [One-to-Many relationships](#one-to-many-relationships)
      - [Many-to-Many](#many-to-many)
      - [Schema flexibility](#schema-flexibility)
      - [When to choose relational vs document models](#when-to-choose-relational-vs-document-models)
  - [Query languages for data](#query-languages-for-data)
    - [Declarative Query Languages (SQL)](#declarative-query-languages-sql)
    - [Imperative Query Languages (Programming Code)](#imperative-query-languages-programming-code)
    - [MapReduce Querying](#mapreduce-querying)
    - [Graph-Like Data Models](#graph-like-data-models)
      - [When to use graphs](#when-to-use-graphs)
      - [Property graph model](#property-graph-model)
        - [The Cypher Query Language](#the-cypher-query-language)
        - [Graph Queries in SQL (The Recursive Pain)](#graph-queries-in-sql-the-recursive-pain)
      - [Triple Stores or RDF](#triple-stores-or-rdf)
- [1. Define Alice](#1-define-alice)
- [2. Define Bob](#2-define-bob)
- [3. Define the Relationship (Edge)](#3-define-the-relationship-edge)
        - [SPARQL](#sparql)
      - [Summary](#summary)
    - [Datalog](#datalog)
  - [Storage and Retrieval](#storage-and-retrieval)
    - [The world's simplest database](#the-worlds-simplest-database)
      - [The Fundamental Trade-off](#the-fundamental-trade-off)
    - [Hash Indexes](#hash-indexes)
      - [The hash map data structure](#the-hash-map-data-structure)
      - [Need for segmentation and compaction](#need-for-segmentation-and-compaction)
      - [Crash recovery in hash indexes](#crash-recovery-in-hash-indexes)
      - [Pros and cons of hash indexes](#pros-and-cons-of-hash-indexes)
    - [SSTables and LSM-Trees](#sstables-and-lsm-trees)
      - [Why Sorting Helps](#why-sorting-helps)
      - [LSM-Trees](#lsm-trees)
      - [Benefits and drawbacks of LSM-Trees](#benefits-and-drawbacks-of-lsm-trees)
      - [Bloom filters](#bloom-filters)
    - [B-Trees](#b-trees)
      - [The Page Concept](#the-page-concept)
      - [The Tree Structure](#the-tree-structure)
      - [Branching factor of B-Trees](#branching-factor-of-b-trees)
      - [Page fragmentation in B-trees](#page-fragmentation-in-b-trees)
      - [Reliability of B-Trees](#reliability-of-b-trees)
      - [B-Trees vs LSM-Trees](#b-trees-vs-lsm-trees)
      - [B-Trees vs B+ Trees](#b-trees-vs-b-trees)
    - [Other indexing structures](#other-indexing-structures)
    - [Transaction processing vs analytics](#transaction-processing-vs-analytics)
      - [OLTP vs OLAP](#oltp-vs-olap)
      - [Data warehousing](#data-warehousing)
      - [Stars and snowflake schemas for analytics](#stars-and-snowflake-schemas-for-analytics)
    - [Column oriented storage](#column-oriented-storage)
      - [Column Compression](#column-compression)
      - [Sort Order in Column Stores](#sort-order-in-column-stores)
  - [Replication](#replication)
    - [Why do we replicate?](#why-do-we-replicate)
    - [Leaders and Followers](#leaders-and-followers)
    - [Synchronous vs Asynchronous Replication](#synchronous-vs-asynchronous-replication)
    - [Setting up new followers](#setting-up-new-followers)
    - [Handling Node Outages](#handling-node-outages)
    - [Implementation of Replication Logs](#implementation-of-replication-logs)
    - [Problems with lag during replication](#problems-with-lag-during-replication)
      - [Problem A: Reading Your Own Writes](#problem-a-reading-your-own-writes)
      - [Problem B: Monotonic Reads (Moving Backward in Time)](#problem-b-monotonic-reads-moving-backward-in-time)
      - [Problem C: Consistent Prefix Reads](#problem-c-consistent-prefix-reads)
    - [Multi-Leader replication](#multi-leader-replication)
      - [Use Cases for Multi-Leader replication](#use-cases-for-multi-leader-replication)
      - [Handling Write Conflicts](#handling-write-conflicts)
      - [Multi-Leader replication topologies](#multi-leader-replication-topologies)
    - [Leaderless replication](#leaderless-replication)
      - [Writing to the DB when a Node is Down](#writing-to-the-db-when-a-node-is-down)
      - [How does a dead node catch up?](#how-does-a-dead-node-catch-up)
      - [Quorums](#quorums)
      - [Limitations of Quorum Consistency](#limitations-of-quorum-consistency)
      - [Detecting Concurrent Writes](#detecting-concurrent-writes)
  - [Partitioning](#partitioning)
    - [Partitioning and replication](#partitioning-and-replication)
    - [Partitioning of Key-Value Data](#partitioning-of-key-value-data)
      - [Partitioning by Key Range](#partitioning-by-key-range)
      - [Partitioning by Hash of Key](#partitioning-by-hash-of-key)
      - [Skewed Workloads & Relieving Hot Spots](#skewed-workloads-relieving-hot-spots)
    - [Partitioning and Secondary Indexes](#partitioning-and-secondary-indexes)
      - [Document-Partitioned Index](#document-partitioned-index)
      - [Term-Partitioned Index](#term-partitioned-index)
    - [Rebalancing Partitions](#rebalancing-partitions)
    - [Automatic vs Manual Rebalancing Operations](#automatic-vs-manual-rebalancing-operations)
    - [Request Routing](#request-routing)
      - [Role of Zookeeper](#role-of-zookeeper)
    - [Parallel Query Execution](#parallel-query-execution)
  - [Transactions](#transactions)
    - [ACID standards](#acid-standards)
    - [Single-object and Multi-object operations](#single-object-and-multi-object-operations)
      - [Single-Object Writes](#single-object-writes)
      - [Do we really need Multi-Object Transactions?](#do-we-really-need-multi-object-transactions)
    - [Weak Isolation Levels](#weak-isolation-levels)
      - [Read Committed](#read-committed)
      - [Snapshot Isolation or Repeatable Read](#snapshot-isolation-or-repeatable-read)
      - [Preventing Lost Updates](#preventing-lost-updates)
      - [Write Skew and Phantoms](#write-skew-and-phantoms)
    - [Serializability](#serializability)
      - [Actual Serial Execution](#actual-serial-execution)
      - [Two-Phase Locking (2PL)](#two-phase-locking-2pl)
        - [Predicate Locks & Index-Range Locks](#predicate-locks-index-range-locks)
      - [Serializable Snapshot Isolation or SSI](#serializable-snapshot-isolation-or-ssi)
    - [Understanding phantom reads](#understanding-phantom-reads)
      - [Fixes for phantom reads](#fixes-for-phantom-reads)
  - [Consistency and Consensus](#consistency-and-consensus)
    - [Linearizability and Ordering](#linearizability-and-ordering)
      - [**Linearizability (Strong Consistency)**](#linearizability-strong-consistency)
      - [Linearizability vs Serializability](#linearizability-vs-serializability)
      - [Ordering and Causality](#ordering-and-causality)
      - [Total Order Broadcast](#total-order-broadcast)
    - [Distributed Consensus](#distributed-consensus)
      - [Atomic Commit (Two-Phase Commit / 2PC)](#atomic-commit-two-phase-commit-2pc)
      - [Distributed Consensus Algorithms (Paxos, Raft, Zab)](#distributed-consensus-algorithms-paxos-raft-zab)
      - [The FLP Result (Theoretical Limit)](#the-flp-result-theoretical-limit)


## Reliable, Scalable, and Maintainable Applications

### The Big Three Pillars

1. **Reliability**: Works correctly despite errors
2. **Scalability**: Can handle growth
3. **Maintainability**: Easy to fix/change

```
       +-----------------+
       |   DATA SYSTEM   |
       +-----------------+
      /         |         \
(Reliability)(Scalability)(Maintainability)
    |              |                 |
"Work correctly" "Handle Growth" "Easy to fix/change"
"despite errors"
```


### Reliability

**Definition**: The system should **continue to work correctly** (performing the *correct function* at the *desired level of performance*) **even in the face of adversity** (hardware/software faults, human error)

Reliability is not just "uptime," but a four-part promise. A reliable system must:
1. *Function correctly*: Do what the user expects.
2. *Tolerate mistakes*: Handle user errors (e.g., entering bad data) without crashing.
3. *Perform*: Deliver data fast enough for the use case.
4. *Secure*: Prevent unauthorized access.

#### Faults vs failures

In system design (& interviews), knowing this distinction is critical.

- **Fault**: A component of the system deviates from its spec (e.g., a hard drive crashes, a process hangs)
	- Example: In a cluster of 50 database servers, one hard drive dies spontaneously
	- Status: The system has a fault, but because you have replicas, the website is still up
- **Failure**: The system as a whole stops providing service to the user
	- Example: That single hard drive dies, but you had no backups and no replicas. Consequently, the user clicks "Save" and sees a "500 Internal Server Error" message
	- Status: The system has failed

**Goal**: ***Design systems that tolerate faults so they don't lead to failure i.e A "fault tolerant" system***

```
      [ Component A ]    [ Component B (CRASH!) ]    [ Component C ]
            |                      X                       |
            |                      |                       |
            v                      v                       v
      +------------------------------------------------------------+
      |                      SYSTEM LOGIC                          |
      |   (Detects B is dead -> Reroutes traffic to A and C)       |
      +------------------------------------------------------------+
                                   |
                                   v
                         [ USER: "It works!" ]
                        (System Failure Avoided)
```

```
        [ FAULT ]                           [ FAILURE ]
   (Internal Component)                  (User Experience)
           |                                     |
           v                                     v
   +----------------+                   +------------------+
   | Server A Dies  |                   |  "404 Not Found" |
   +-------+--------+                   |  "System Down"   |
           |                            +------------------+
           |
           v
   +------------------------+
   | FAULT TOLERANCE CHECK  |
   +------------------------+
   |
   +---> Is there a Replica?
   |     YES: Traffic moves to Server B.  -----> [ NO FAILURE ]
   |                                             (User notices nothing)
   |
   +---> NO: Traffic hits dead end.       -----> [ FAILURE ]
                                                 (User sees error)
```

#### Categories of faults

Categories of Faults
1. **Hardware Faults**: HDDs crashing, RAM becoming defective, power blackouts
	- Mitigation: Redundancy (RAID, dual power supplies, multi-AZ deployment)
2. **Software Errors**: Bugs, runaway processes, cascading failures (a bug in one node causing crashes in others)
	- Mitigation: Process isolation, crash-only design, monitoring
3. **Human Error**: Configuration errors are the #1 cause of outages
	- Mitigation: Sandbox environments, automated testing, quick rollbacks

Hardware faults example:
- It is the "Physical" Problem
- Hard disks crash, RAM becomes defective, power grids fail, and someone unplugs the wrong cable
- The Reality: Hard disks have a specific Mean Time To Failure (MTTF). If you have 10,000 disks, you will lose one every day
- The Old Way: Build special, expensive "Mainframes" with nuclear-grade backup components
- The Modern Way (Cloud): Embrace "Software Fault Tolerance." We use cheap, unreliable hardware (commodity servers) and rely on software to handle the breakage
- Solution: ***Redundancy***. RAID configurations, dual power supplies, and replicating data across different data centers (Multi-AZ)
	- RAID is a storage virtualization technology that combines multiple physical disk drives into a single logical unit to improve data performance through striping or data reliability through redundancy

Software faults example:
- The "silent" killer
- Harder to catch because they are often correlated. If a hard drive fails, it usually doesn't cause the neighboring hard drive to fail. But a software bug usually affects every server running that code at the exact same time
- Example: 
	- A "Leap Second" bug (Linux kernel panic) crashing every server at midnight
	-  A runaway process using 100% CPU
	- Cascading Failures: One node slows down, causing a backlog, which crashes the next node, toppling the whole system like dominoes
- Solution:
	- **Isolation**: Don't let one bad component crash the whole app (Bulkhead pattern)
	- **Watchdogs**: Processes that monitor health and restart dead services automatically
	- **Chaos Engineering**: Deliberately killing processes (like Netflix's Chaos Monkey) to ensure the system handles it.

Human errors example:
- The #1 cause
- Configuration errors by operators are the leading cause of outages, far outpacing hardware failures. Humans are unreliable
- The Paradox: We want to automate everything, but automation bugs can destroy data faster than humans can
- Solutions:
	- **Sandbox Environments**: Places to break things safely (Staging/QA)
	- **Canary Release**s: Roll out a config change to 1% of users. If they crash, stop. Do not roll out to 100% at once
	- **Quick Rollbacks**: A "Undo" button for configuration changes
	- **Telemetry**: Clear metrics so you know immediately when a human broke something

#### Why reliability matters

1. **Trust**: Even for a non-critical app (like a photo-sharing site), data loss destroys user trust forever
2. **Cost**: While reliability costs money to build, the lost revenue and reputation damage from an outage usually cost more

### Scalability

**Definition**: A system’s ability to cope with increased load. It is not a binary label ("X is scalable"); it is a description of how we handle growth.

Scalability is a question: "***If the system grows in a particular way, what are our options for coping with the growth***?"

**Important**: It is meaningless to say "X is scalable" without defining the Load Parameter. A system scalable for 1 million users might fail if those users suddenly start uploading 4K video instead of text.

#### Describing Load (Load Parameters)

You must define what "load" means for your specific system. It is usually measured by Load Parameters.

1. **Web server**: Requests per second (**RPS**)
2. **Database**: Ratio of Read vs. Write (e.g., 90% Read / 10% Write)
3. **Chat room**: Number of *active* users vs. *simultaneous active* users (Not total users)
4. **Cache**: Load = Hit Rate vs. Miss Rate

**The Twitter Example**

Classic System Design Case Study: Use Twitter to explain ***Fan-out***.

The Challenge: Users follow many people. When a user tweets, it must appear on millions of followers' timelines.

1. Approach A: **Pull (Load on Read)**
	- Action: When User A tweets, insert into a global tweets table
	- Read: When User B loads their timeline, query: `SELECT * FROM tweets WHERE sender_id IN (list_of_people_B_follows)`
	- Pros: ***Writes are cheap***
	- Cons: ***Reads are incredibly expensive. Complex SQL join***.
2. Approach B: **Push (Load on Write)**
	- Action: Each user has a pre-computed "Timeline Cache". When User A tweets, look up all their followers and insert the tweet ID into their cache lists
	- Read: Just read the cache `O(1)`
	- Pros: ***Reads are instant***
	- Cons: ***Write Fan-out***. If Justin Bieber (100M followers) tweets, the system must perform 100M writes instantly

**Takeaway**: System Design is about balancing these trade-offs based on your load parameters (Twitter is read-heavy)

**The Hybrid Solution**: ***Twitter uses Push for normal users (fast reads for everyone) but Pull for celebrities (to save the write buffers from exploding)***

Twitter old approach example:
```
  [User A Tweets] ---> [Global DB Table]
                          ^
                          | (Complex SQL Join)
                          |
[User B Reads] <----------+
```

Twitter "fan-out" example with numbers:
```
                 
[User A Tweets]         /--> [User B's Cache]
       |               /
       |               --> [User C's Cache]
       |             /
(System copies      /--> [User D's Cache]
 to all followers)--
                    \--> [User E's Cache] -> [User E Reads] (Instant)
```

#### Describing Performance

- **Latency**: The time waiting for a request to be handled (duration)
- **Response Time**: Latency + Network Delay + Queueing Delay (This is what the client actually sees)

**The Tyranny of the Average**
- Do not look at the Average (Mean) response time
- Averages hide outliers. If 99 requests take 1ms and 1 request takes 100s, the average looks okay, but 1 user is furious

**Tip for measuring**: The "Tail" is Everything (P95, P99) Don't use Averages (Mean). Averages hide outliers. Use ***Percentiles***.
- `P50` (**Median**): Half of users are faster than this (The "typical" user experience)
- `P95` (95th Percentile): 95% of requests are faster than this. The slowest 5% are worse
- `P99/P999` (**Tail Latency**): These are usually your VIP customers (who have the most data) and suffer the worst performance (Tail Latency Amplification).

```
Counts ^
       |           P50          P99 (The Tail)
       |            |            |
  High |      /--\  |            |
       |     /    \ |            |
       |    /      \|            |
       |___/        \____________|______
       0ms         200ms        2sec    Latency
```

**Why optimize for P99?**

- The VIP Argument: The users with the most data (slowest requests) are often your most active, highest-paying customers
- Tail Latency Amplification: If a webpage needs to call 10 microservices to render, and each service has a 1% chance of being slow, the final page has a ~10% chance of being slow. One slow service drags down the whole response

```
Service A (Fast) --\
Service B (Fast) ----> [ Aggregator ] --> User Response
Service C (SLOW) --/       (Waits for C)
                           (Response = SLOW)
```

#### Approaches for coping with Load

**Vertical Scaling (Scale Up)** vs. **Horizontal Scaling (Scale Out / Shared nothing)**
- Scale Up: Buy a bigger machine (More RAM, better CPU)
	- Pros: Simpler. No network overhead
	- Cons: Expensive. Has a hard limit (physics)
- Scale Out: Add more cheap machines
	- Pros: Infinite theoretical scale. Cheaper
	- Cons: Complex. You now have to handle distributed consistency, network faults, and data syncing

Vertical scaling:
```
      +----------------------+
      |  SUPER COMPUTER      |
      | [CPU] [CPU] [CPU]... |
      | [RAM: 10TB]          |
      +----------------------+
```
Horizontal scaling:
```
+--------+   +--------+   +--------+
| Node A |---| Node B |---| Node C |
+--------+   +--------+   +--------+
    |            |            |
  (Disk)       (Disk)       (Disk)
```

### Maintainability

**Definition**: Ideally, the majority of the cost of software is not writing it, but maintaining it over time

As a system grows, it tends to become a **"Big Ball of Mud**"—a tangled mess of dependencies that everyone is afraid to touch

1. **Operability**: Making it easy for operations teams to keep the system running
	- Needs: Good monitoring, automation, documentation, standard defaults
2. **Simplicity**: Managing complexity
	- Accidental Complexity: Complexity arising from bad tooling or implementation (bad)
		- Using a complex microservices architecture with Kafka and Kubernetes for a simple "To-Do List" app
	- Essential Complexity: Complexity inherent to the problem domain (unavoidable)
		- *Example*: Calculating US Tax Rates is complex because the tax laws are complex. You cannot code this simply because the domain is hard
	- ***Solution***: ***Abstraction***. Hiding implementation details behind clean APIs
		- i.e Abstraction Layer (API Gateway or Interface) to hide the legacy mess
		- Ex: SQL hides the hard details (disk seeks, B-Trees, data layout) behind a clean, simple API (`SELECT *`)
3. **Evolvability** (Agility): How easily can you change the system?
	- Feature flags or parallel implementations (write to old and new system, compare results) to ensure the change doesn't break existing functionality
	- *Agile / TDD*: Testing frameworks allow you to refactor without fear of breaking hidden features
	- *Loose Coupling*: Microservices (if done right) allow one team to upgrade their service without coordinating with 50 other teams
	- *Data Migration Strategies*: Being able to migrate data schemas (as discussed in the gRPC backward compatibility section) without downtime.

The goal of Agile/TDD/Refactoring patterns is to *allow systems to change requirements without breaking*

```
      [ APPLICATION CODE ]
              |
              | (Simple API Call: "Save User")
              v
      +------------------+
      | ABSTRACTION LAYER| <--- Hides the "Accidental Complexity"
      +------------------+      of handling Retries, Failover,
              |                 and Disk formats.
              v
      [ COMPLEX INTERNALS ]
```

### Mindset

The "***Composite Data System***" Mindset
- The very first page of the book challenges the definition of a "Database."
- The Old World: You had a "Database" (SQL) and an "App."
- The New World: You have a Database (Postgres), a Cache (Redis), a Search Index (Elasticsearch), and a Message Queue (Kafka)

*The Concept*: You are no longer just an "application developer." You are a **Data System Designer**. Your application code is simply the "glue" that binds these different tools into one cohesive API

Terminology:
1. **SLI (Service Level Indicator)**: The metric itself
	- Example: "The P99 latency of the GET `/home` endpoint."
2. **SLO (Service Level Objective)**: The target you aim for internally
	- Example: "We want the SLI to be < 200ms for 99.9% of requests."
	- Note: If you break this, you don't get sued, but you wake up the on-call engineer
3. **SLA (Service Level Agreement)**: The contract with the customer (with penalties)
	- Example: "If the service is down for more than 1 hour, we will refund you 10% of your bill."
	- Note: The SLA is always looser than the SLO (to give you a safety buffer)

```
       [ SLA (The Lawyer's Contract) ]
       "We promise 99.0% uptime or we pay you."
                   |
                   v
       [ SLO (The Engineer's Goal) ]
       "We aim for 99.9% uptime so we never hit the SLA."
                   |
                   v
       [ SLI (The Real Number) ]
       "Current uptime is 99.95%."
```

## Data models and query languages

This chapter argues that ***Data Modeling is the single most important decision you make*** when building a system. It is not just about where you store bits; it determines how you think about the problem

### The Power of Abstraction

Every system is built on **layers of data models** (Abstraction). Each layer hides the complexity of the layer below it:
- You (The Developer): You look at real-world things (people, invoices, products) and model them as Objects or Data Structures (JSON, Classes).
- The Database Software: Takes your objects and models them as Tables (SQL), Documents (JSON), or Graphs. It doesn't know what a "user" is; it just knows rows and columns.
- The Operating System: Takes those tables and models them as a stream of Bytes on a disk.
- The Hardware: Takes those bytes and models them as electrical pulses or magnetic spins.

The Key Takeaway: You only work at the top layer. Your choice of data model (e.g., Relational vs. Document) defines the "API" through which you see the world. ***If you choose the wrong model, your code becomes ugly, slow, and hard to maintain***

```
      [ REAL WORLD ]  (People, Money, Actions)
            |
            v
      [ APPLICATION ] (Objects, Arrays, Classes)
            |
            v
      [ DATA SYSTEM ] (Tables, Documents, Graphs) <--- CHAPTER 2 FOCUS
            |
            v
      [   HARDWARE  ] (Bytes, Electrical Pulses)
```

### Relational model versus Document model

#### One-to-Many relationships 

A One-to-Many relationship exists when a single record in one table is associated with multiple records in another table, but each of those multiple records belongs to only one "parent." A classic example is a User and their *Blog Posts*
- One User (Alice) can write 50 different blog posts, but each specific blog post is written by only one author (Alice)
- In a database, this is implemented by placing the "User ID" (Foreign Key) inside the Posts table, effectively "tagging" each post with its owner

```
   [ USER TABLE ]                      [ POSTS TABLE ]
   (The "One" Side)                    (The "Many" Side)

 +----+-------+                  +----+------------------+-----------+
 | ID | Name  |                  | ID | Title            | User_ID   |
 +----+-------+                  +----+------------------+-----------+
 | 1  | Alice | <--------------- | 10 | "My First Day"   | 1         |
 +----+-------+       |          +----+------------------+-----------+
                      |--------- | 11 | "Learning SQL"   | 1         |
                      |          +----+------------------+-----------+
                      |--------- | 12 | "Coding Tips"    | 1         |
                                 +----+------------------+-----------+
```


**The Resume Problem**

Think about a user profile. It has a unique `User_ID`, but it also has a list of Jobs, a list of Education, and a list of Contact Info
Aa "Resume" or "User Profile" to frame the argument. A profile is naturally a Hierarchical Tree. A User has a list of Jobs, and each Job has a list of responsibilities.

**The Relational Approach (SQL)**

SQL databases *hate* trees. They want ***flat lists***

To store this in a relational DB, you must perform "**Shredding**". You take the single concept of a "User" and shred it into *multiple separate tables* to satisfy the rules of **Normalization**
- **Process**: The "Shredding": To save a profile, you must "shred" the object into pieces: a User row, 5 Job rows, and 2 Education row
- **The Pain**: To reconstruct the profile, the database must scan 3 different tables and stitch them back together (`JOINs`). This is slow and complex

```
       OBJECT (Code/JSON)                 RELATIONAL (SQL)
      +------------------+              +-------+  +-------+
      | User: Alice      |              | User  |  | Jobs  |
      | - Job: Google    |   (Shreds)   |-------|  |-------|
      | - Job: Amazon    |  --------->  | Alice |  | Google|
      | - School: MIT    |              +-------+  | Amazon|
      +------------------+                         +-------+
                                                   +-------+
                                                   | School|
                                                   |-------|
                                                   |  MIT  |
                                                   +-------+
```

**The Document Model (JSON)**

Because the data is a **tree**, it fits perfectly into a **JSON document.**
- **The Fit**: The data model (JSON) looks exactly like the object model in your code (Java/JS objects)
- **The Benefit**: ***No Impedance Mismatch***. You don't need a complex translation layer (ORM) to turn your object into database rows. You just dump the object to the database
- **Performance (Locality)**: If your app usually loads the entire profile at once, this is faster. The database reads one continuous blob from the disk

> **Impedance Mismatch** refers to the difficulties encountered when translating data between two systems that conceptualize data differently, most commonly describing the friction between Object-Oriented Programming (rich objects with methods and inheritance) and Relational Databases (flat tables with rows and columns)

```
     IN MEMORY (Your Code)                  IN DATABASE (SQL)
    (Object-Oriented Model)                 (Relational Model)

       [ User Object ]
      +----------------+
      | Name: Alice    |
      | Jobs: [List] --+-----\
      +----------------+      \             [ Table: Users ]
              |                \            +----+-------+
              | (Pointer)       \           | ID | Name  |
              v                  \          | 1  | Alice |
       [ Job Object ]             \         +----+-------+
      +--------------+             \
      | Title: CEO   |              \       (The "Mismatch" Wall)
      +--------------+               \      (Requires Translation)
                                      \            ||
                                       \           ||
                                        \          \/
                                         [ Table: Jobs ]
                                         +----+---------+-------+
                                         | ID | User_ID | Title |
                                         | 55 |    1    |  CEO  |
                                         +----+---------+-------+
```

> **Locality** is the strategy of keeping things you are using (or things near them) close to you for faster access, because if you used something once, you are likely to use it again soon. The Two Types: **Temporal** (Time): "I just read this page, I will probably read it again soon." (So keep it in the cache). **Spatial** (Space): "I just read page 1, I will probably read page 2 next." (So load the next chunk of data too)

```
      RELATIONAL (Shredded)              DOCUMENT (Nested)
      "Normalization First"              "Locality First"

      [Table: Users]                     [Document: User]
      ID | Name                          {
      1  | Bill                            "id": 1,
                                           "name": "Bill",
      [Table: Jobs]                        "jobs": [
      ID | User_ID | Title                   { "title": "CEO" },
      10 | 1       | CEO                     { "title": "Founder" }
      11 | 1       | Founder               ],
                                           "education": [ ... ]
      [Table: Education]                 }
      ID | User_ID | School
      50 | 1       | Harvard
```

***Document databases are superior for one-to-many relationships*** because they ***allow you to embed related child records (like addresses or comments) directly inside the parent document***, enabling the application to ***fetch the entire tree in a single read operation without the performance penalty of complex joins***

```js
// The "Embedded" advantage: Everything is in one place.
{
  "user_id": 1,
  "name": "Alice",
  "posts": [
    { "title": "Post A", "content": "..." },  // <-- No need to look in
    { "title": "Post B", "content": "..." }   // <-- a different table!
  ]
}
```

#### Many-to-Many

**Many-to-many relationships explained**

The Scenario Imagine you are building a system to store User Resumes.
- Users: Alice and Bob
- Skills: Both of them know "Java"

This is a Many-to-Many relationship because:
- One User (Alice) has many skills (Java, SQL)
- One Skill (Java) belongs to many users (Alice, Bob)

**Drawback** of *storing text / duplicates*: If you are storing the word "Java" multiple times per user and the "Java" changes its name to "Java SE", you have to find and update it in every single user's profile

**Benefits** of *storing IDs / references*: 
- Normalization: We store the concept of "Java" in *only one place*
- Consistency: If we fix a typo in the Skill table, it *updates for everyone instantly*

**The weakness of Documents**

The Document model looks perfect for the Resume... until you need to reference data *shared* between users

**The Scenario**: Imagine you want to standardize "School Names." Instead of storing the string "Harvard" in every student's document, you want a specific Organization entity so you can update the school's logo or name in one place and have it reflect for all students.

- **Relational**: This is easy. Create an Organizations table. Users link to `Organization_ID`. You update the organization row once, and everyone sees the change. This is **Normalization**.
- **Document**: This is hard
	- *Option A*: Duplicate the data (store "Harvard" in 10,000 documents). If the name changes, you must update 10,000 documents
	- *Option B*: Simulate a Join. Store an `Org_ID` in the document, and make your application code run a second query to fetch the organization name. (*This moves complexity from the DB to your code*)

***RDBMS are superior for many-to-many relationships*** because t***hey normalize shared data into separate tables*** (linked by a junction table)***, ensuring that if a shared item like a "Skill" or "Product Category" changes, it only needs to be updated in one place***, whereas a document database would often require a massive, slow "update many" operation across thousands of documents

SQL example:
```sql
// Since users only store the ID (skill_id: 101), 
// we only update one single row in the central Skills table. 
// All 1,000,000 users see the change instantly.
// -- ONE efficient operation
UPDATE skills
SET name = 'Java 21'
WHERE id = 101;
```

Document DB example in JS:
```js
// HEAVY operation: Must find and modify 1,000,000 documents
db.users.updateMany(
  { skills: "Java" },             // Find everyone with "Java"
  { $set: { "skills.$": "Java 21" } } // Rewrite the string
);
```

Relational database data fetching example:
```
      [ Table: Schools ] <--- (Source of Truth)
      +-----+-----------+
      | ID  | Name      |
      | 99  | Harvard   |  <-- UPDATE this one row to "Harvard Univ"
      +-----+-----------+
               ^     ^
               |     | (Foreign Key References)
               |     |
      [ Table: Students ]
      +-------+-------------+
      | User  | School_ID   |
      | Alice |     99      | <--- Alice automatically sees new name
      | Bob   |     99      | <--- Bob automatically sees new name
      +-------+-------------+
```

Document models data fetching option 1: Duplication / De-normalization ("Fast reads, painful updates." The name is copied into every document. Updating requires a massive "Find & Replace" operation)
```
      [ Doc: Alice ]                  [ Doc: Bob ]
      {                               {
        "name": "Alice",                "name": "Bob",
        "school": "Harvard"             "school": "Harvard"
      }                               }
          |                                   |
          v                                   v
      (Update?)                           (Update?)
      MUST WRITE TO DISK                  MUST WRITE TO DISK

      *Result: If you have 1 million students, you must perform
               1 million writes just to change the school name.*
```

Document models data fetching option 2: App-Side Join ("Complexity moves to your code." The database acts dumb. Your application code does the heavy lifting of fetching data twice)
```
        DATABASE (Storage)                APPLICATION (Logic)
                                         "I need Alice's School"
      [ Doc: Alice ]                             |
      { "name": "Alice",                         | (1) Query User
        "school_id": 99 }  ------------------->  | Gets { school_id: 99 }
                                                 |
                                                 | (2) Logic: "Oh, I need 99"
      [ Doc: School ]                            |
      { "id": 99,                                | (3) Query School
        "name": "Harvard" } ------------------>  | Gets "Harvard"
                                                 |
                                                 v
                                          (4) Join & Return
```

**Verdict**:
- ***Document Models handle One-to-Many relationships*** (`User -> Jobs`) beautifully (Better Locality)
- **Relational Models handle Many-to-Many relationships** (`Students <-> Schools`) beautifully (Better Normalization)

#### Schema flexibility

**Read vs Write**

1. **Schema-on-Write (Relational)**
	- Analogy: ***Static Typing*** (Java/C++)
	- How it works: You must define the table structure (`CREATE TABLE`) before you insert data. The database ensures every row conforms to the rules
	- Concept: The schema is a ***Contract***. The database enforces it. You cannot insert data that breaks the rules
	- Use Case: Critical data where ***consistency is non-negotiable*** (Financial ledgers)
	- ***Pros***: Strict guarantees. You know exactly what the data looks like.
	- ***Cons***: Migrations are painful. Adding a column to a 10TB table can take hours or days and might require downtime

```
      [ RELATIONAL TABLE: USERS ]
      (Strict Columns: ID, Name)

 1. INSERT: { id:1, name:"Alice" }  -----> [ OK ]

 2. INSERT: { id:2, name:"Bob",     -----> [ ERROR! ]
              twitter:"@bob" }             (Column 'twitter' does not exist.
                                            Transaction Rejected.)

    *Solution:* You must run a Migration first.
```
```
      [ STEP 1: BEFORE ]         [ STEP 2: MIGRATION ]        [ STEP 3: AFTER ]
      +----+-------+             (Run ALTER TABLE...)         +----+-------+-------+
      | ID | Name  |             * Database Works            * | ID | Name  | Phone |
      +----+-------+             * Hard Drive I/O            * +----+-------+-------+
      | 1  | Alice |             * Potentially               * | 1  | Alice | NULL  | <--- Padded
      +----+-------+             * Locks Table               * +----+-------+-------+
      | 2  | Bob   |                                           | 2  | Bob   | NULL  | <--- Padded
      +----+-------+                                           +----+-------+-------+
                                                               | 3  | Carol | 1234  | <--- New User
                                                               +----+-------+-------+
```

2. **Schema-on-Read (Document)**
	- Analogy: ***Dynamic Typing*** (JavaScript/Python)
	- How it works: The database doesn't care. You can dump any JSON structure you want. The structure is interpreted only when your application reads the data
	- Concept: The database is a ***bucket***. ***It accepts anything***. The structure is only interpreted when you Read the data back
	- Use Case: Fast-moving apps where the ***data structure changes weekly, or dealing with "messy" external data***
	- ***Pros***: Extreme flexibility. Zero downtime migrations. Great for messy, heterogeneous data
	- ***Cons***: If you are messy, your application code becomes full of `if (user.jobs)` checks to handle old vs. new data formats

```
      [ DOCUMENT COLLECTION: USERS ]
      (No predefined structure)

 1. INSERT: { id:1, name:"Alice" }  -----> [ OK ]

 2. INSERT: { id:2, name:"Bob",     -----> [ OK ]
              twitter:"@bob" }             (Saved successfully.)

 3. INSERT: { id:3, name:"Charlie", -----> [ OK ]
              verified: true }             (Totally different structure? Fine.)
```
```
      [ COLLECTION: USERS ]
      (No structural change happens. Just insert.)

      --------------------------------------------------
      Doc 1: { "id": 1, "name": "Alice" }                 <-- Old Style
      --------------------------------------------------
      Doc 2: { "id": 2, "name": "Bob" }                   <-- Old Style
      --------------------------------------------------
      Doc 3: { "id": 3, "name": "Carol", "phone": 1234 }  <-- New Style!
      --------------------------------------------------

      *Result:* The Application Code must handle the missing field:
      if (user.phone) { ... } else { ... }
```

#### When to choose relational vs document models

**Use Document Model if:**
1. Your data is a "**Tree**" structure (One root object with sub-items) i.e **One to Many**
2. You **usually access the entire tree at once** (e.g., loading a profile page)
	- Document databases are ideal when your application's access pattern typically involves fetching an entire aggregate entity—like a parent object along with all its nested children—in a single, efficient read operation
```
    APPLICATION GOAL: "Load Full User Profile Screen"
           |
           | (One Request)
           v
+--------------------------------------------------+
| DOCUMENT DATABASE                                |
| (Data stored together as a single 'tree')        |
+--------------------------------------------------+
| Doc ID: 100                                      | --> THE ENTIRE TREE
| {                                                |     IS FETCHED AT ONCE.
|   "username": "alice_dev",                       |
|   "bio": "Full stack engineer...",               |
|   "recent_posts": [                              |
|      {"title": "Post A", "likes": 10},           |
|      {"title": "Post B", "likes": 25}            |
|   ],                                             |
|   "settings": { "theme": "dark", "notifs": on }  |
| }                                                |
+--------------------------------------------------+
           |
           | (One Response containing everything needed)
           v
   [ SCREEN RENDERED ]
```
3. Your **schema changes frequently**

**Use Relational Model if:**
1. Your data is highly interconnected (**Many-to-Many**)
2. You need **strict consistency and data integrity**
3. You often **need to analyze data in different ways** (e.g., "Find all users who went to Harvard," which is slow in Document DBs if not indexed)

| Feature | Document Model (NoSQL) | Relational Model (SQL) |
| -- | -- | -- |
| Data Structure | Trees (Hierarchy) | Tables (Flat) |
| Impedance Mismatch | Low (Matches code objects) | High (Requires translation/ORM) |
| Relationships | Great for 1-to-Many (Nesting) | Great for Many-to-Many (Joins) |
| Locality | High (Whole tree in one read) | Low (Data spread across tables) |
| Flexibility | High (Schema-on-Read) | Low (Schema-on-Write) |

## Query languages for data

### Declarative Query Languages (SQL)

**"The Manager Approach"**

In a declarative language, you tell the computer **WHAT** you want, but you *don't tell it HOW* to get it. You leave the details (indexes, sorting methods, memory management) to the database engine
- **The vibe**: You are a CEO. You say, "I want a report on sales." You don't care if the team used Excel, a calculator, or an abacus, as long as the result is right
- **The benefit**: ***The database can optimize itself***. If you add a new index later, the database automatically uses it without you rewriting your query

```SQL
SELECT * FROM animals WHERE species = 'Shark';
```

```
       User (Manager)                  Database (The Engineer)
            |                                    |
            | "Get me all Sharks."               |
            | (Doesn't care how)                 |
            | ---------------------------------> |
            |                                    | [Internal Brain]
            |                                    | "Hmm, should I scan the whole table?"
            |                                    | "No, I have an index on 'species'."
            |                                    | "I'll jump straight to 'S'."
            |                                    |
            | "Here is the list."                |
            | <--------------------------------- |
```

### Imperative Query Languages (Programming Code)

**"The Micro-Manager Approach"**

In an imperative language (like Java, Python, or the old IMS database systems), **you must tell the computer exactly HOW to achieve the goal, step-by-step**
- **The vibe**: You are a micro-manager. You say: "Go to the filing cabinet. Open drawer 3. Pull out the first folder. Check if it says 'Shark'. If yes, put it in the red box. If no, put it back. Move to the next folder..."
- **The drawback**: ***Hard to optimize***. If the database structure changes, your code breaks because your "steps" are wrong

```js
var sharks = [];
var animals = db.getAllAnimals(); // Load everything?

for (var i = 0; i < animals.length; i++) {
    if (animals[i].species === "Shark") {
        sharks.push(animals[i]);
    }
}
```

```
      DECLARATIVE (SQL)               IMPERATIVE (Code)
      "What I want"                   "How to do it"

      [ SQL Optimizer ]               [ Your Code ]
             |                             |
      (Decides Strategy)              (Hard-coded Strategy)
      (Can change silently)           (Rigid)
             |                             |
      > Uses Index X                  > Loop i=0
      > OR Uses Scan Y                > If check...
             |                             |
      Result: FAST                    Result: OFTEN SLOWER
```

### MapReduce Querying

**"The Assembly Line Approach"**

MapReduce is a ***middle ground*** used by **NoSQL databases** (like MongoDB or CouchDB). It is neither fully declarative nor fully imperative.

- How it works: You *write small snippets of imperative code (functions)*, and *the database runs them declaratively across many machines*. It allows you to **process large datasets across many machines by breaking the logic into two functions**
- **Map (Filter/Sort)**: "Go through every record and extract specific info."
- **Reduce (Aggregate)**: "Smash all those results together into one number."

Example: Counting Shark sightings per month
- Map: Extracts `{ Month: "Jan", Count: 1 }` from every log
- Shuffle: Groups all "Jan" notes together
- Reduce: Sums them up: "Jan = 5"

```
        [ RAW DATA ]
      {Jan, Shark}, {Feb, Whale}, {Jan, Shark}
             |
             v  (Map Function: "Extract Month")
             |
      [ Jan, 1 ]   [ Feb, 1 ]   [ Jan, 1 ]
             |
             v  (Shuffle: Group by Key)
             |
      [ Jan: (1, 1) ]      [ Feb: (1) ]
             |
             v  (Reduce Function: "Sum")
             |
      [ Jan: 2 ]           [ Feb: 1 ]
```

```js
// 1. MAP: Runs on EVERY document individually.
// "If you see a shark, shout 'Shark!' and hold up 1 finger."
var mapFunction = function() {
    if (this.family === "Sharks") {
        emit(this.species, 1); // Key: Species Name, Value: 1
    }
};

// 2. REDUCE: Runs on the GROUPED results from Map.
// "Take all the fingers held up for 'Great White' and sum them."
var reduceFunction = function(keySpecies, values) {
    return Array.sum(values);
};
```

**Note**: `map` and `reduce` functions **must be pure**. They must rely only on their input data and produce no side effects (like writing to a separate database or modifying a global variable)
- Why? **Distributed systems are unreliable**. A machine running a Map task might crash halfway through. The framework handles this by running the task again on a different machine.
	- If your function is ***Pure***:
		- Attempt 1 fails: No harm done
		- Attempt 2 succeeds: Output is generated
		- Result: Correct Data
	- If your function has Side Effects (***Impure***):
		- Attempt 1 writes to a DB, then crashes
		- Attempt 2 writes to the DB again
		- Result: Duplicate/Corrupted Data

```
    SCENARIO: A Map function that is NOT pure.
    It tries to send an email for every record it processes.

    [ MACHINE A (Attempt 1) ]             [ MACHINE B (Attempt 2 - Retry) ]
    (Processes Record X)                  (Processes Record X)
             |                                     |
             | 1. Sends Email!                     |
             v                                     |
    (Network Failure! Task Crashes)                |
    (Framework thinks: "Task Failed")              |
             |                                     |
             +------------------------------------>|
                                                   | 2. Framework starts Retry.
                                                   |
                                                   | 3. Sends Email (AGAIN)!
                                                   v
                                          [ TASK COMPLETE ]

    RESULT: User received 2 Emails.
    (This is why functions must be pure and side-effect free.)
```

### Graph-Like Data Models

While Document databases target One-to-Many (Hierarchy) and Relational databases handle simple Many-to-Many, **Graph models are built for Many-to-Many on steroids**

It is very useful for depicting the **Web**. If your data is all about *relationships* (Social Networks, Fraud Detection, Road Maps), "Graph-Like" models are the champion

#### When to use graphs

Standard SQL handles relationships by using Foreign Keys. This works fine if you just need to jump one step (e.g., "Who is Alice's boss?")
- But SQL fails when you need ***Recursive Relationships (variable depth)***
- The Problem: "Find the friends of friends of friends of friends"
- SQL's Struggle: You need to write a massive query with 4-5 costly JOINs, or use complex recursive logic
- Graph's Solution: ***It simply "walks" along the path***. It doesn't care if the path is 1 step or 100 steps long

**Analogy**:
- Relational: Using a map index to look up coordinates for every single city on a trip
- Graph: Just driving the car along the highway from city to city

#### Property graph model

**The Property Graph Model (e.g., Neo4j)**: This is the most popular graph model. It consists of two things:
1. **Vertices** (Nodes): The entities (e.g., Alice, Bob, Product X).
2. **Edges** (Relationships): The lines connecting them (e.g., KNOWS, PURCHASED).

The "Property" Twist: Unlike simple math graphs, in a Property Graph, the ***Edges themselves can store data***

- Node: `Alice (Age: 30)`
- Relationship: `Alice --(MARRIED_TO)--> Bob`
- Relationship Property: `MARRIED_TO has a property: since: 2010`

```
      [ Vertex: Alice ]                     [ Vertex: Bob ]
      (Type: Person)                        (Type: Person)
      (Name: Alice)                         (Name: Bob)
             |                                     ^
             |                                     |
             \                                     /
              \--------( Edge: MARRIED )----------/
                       (Since: 2010)
                       (Location: Paris)
```

##### The Cypher Query Language

Cypher (used by Neo4j) is a **declarative language** designed to look like ASCII art. It describes the Pattern you are looking for in the data.

The Syntax:
- `(Nodes)` are in parentheses.
- `[RELATIONSHIPS]` are in brackets.
- `-->` arrows show direction.

Example: "Find the names of all the people Alice is friends with"
```
MATCH
  (alice:Person {name: 'Alice'})-[:FRIENDS_WITH]->(friend)
RETURN friend.name
```

##### Graph Queries in SQL (The Recursive Pain)

If you want to find "variable depth" connections (e.g., traveling from London to Munich via any number of trains) in SQL, you have to use ***Recursive Common Table Expressions*** (`WITH RECURSIVE`)

**The Comparison**:
- **Cypher**: `(London)-[:TRAIN*]->(Munich)` (4 lines of code)
- **SQL**: A 30-line rigid query that loops over itself. It is *brittle* and *hard to optimize*

**Visualizing the Traversal**: Graph databases are fast because they use ***Index-Free Adjacency***. This means Node A physically contains a pointer to Node B. The database doesn't need to check a central index; it just jumps to the memory address of the next node.

Property graphs traversal:
```
       Start Node
         [ A ]
        /     \  (Direct Memory Pointers)
       v       v
     [ B ]-->[ C ]
             /   \
            v     v
          [ D ] [ E ] (Target)

      *The database simply hops A -> C -> E.
       No table scanning required.*
```

Comparison with SQL (Painful):
```
      GRAPH VIEW                       RELATIONAL VIEW
      ----------                       ---------------

      (Alice)                          [ Table: VERTICES ]
         |                             ID | Label  | Properties
         | (KNOWS)                     1  | Person | {"name": "Alice"}
         |                             2  | Person | {"name": "Bob"}
         v
       (Bob)                           [ Table: EDGES ]
                                       ID | Head | Tail | Label | Props
                                       10 |  1   |  2   | KNOWS | {}
```
```SQL
WITH RECURSIVE graph_path (id, depth) AS (
    -- 1. Base Case: Start at Alice
    SELECT vertex_id, 0
    FROM vertices
    WHERE vertex_id = 1

    UNION ALL

    -- 2. Recursive Step: Join Edges to the previous step
    SELECT e.tail_id, gp.depth + 1
    FROM edges e
    JOIN graph_path gp ON e.head_id = gp.id
    WHERE gp.depth < 3 -- Stop at depth 3
)
SELECT * FROM graph_path;

-- Pros: You get the reliability, ACID transactions, and tooling of an RDBMS (Postgres/MySQL) while storing graph data.
-- Cons: The SQL syntax (Recursive CTEs) is clumsy and verbose compared to Cypher ((A)-[:KNOWS*1..3]->(B)). Performance can degrade if the recursion is very deep.
```

#### Triple Stores or RDF

This is the "other" graph model, mostly used in **academia** and the **Semantic Web**. Instead of "Nodes and Edges," it stores everything as simple 3-part sentences: 
1. Subject: 
	- The Subject is the "who" or "what" the sentence is about. *It is the person or thing performing the action*
	- Example: "The **cat** is sleeping"
2. Predicate
	- The Predicate is the part of the sentence that tells you *what the subject is doing or what happens to it*. It always includes the **verb**
	- Example: "The cat **chased the mouse**"
3. Object
	- The Object is the ***noun** or **pronoun** that receives the action of the verb*. It *usually comes after the verb*
	- Example: "The cat chased **the mouse**"

Complete example: "The chef (Subject) cooked (Predicate) a delicious meal (Object)"

It feels similar to property graphs, but everything is broken down into these tiny atomic statements

- Most common data format: **Turtle format**
- Most common query language: **SPARQL**

*Turtle format example (Semantic web)*:
```
@prefix foaf: <http://xmlns.com/foaf/0.1/> .
@prefix ex:   <http://example.org/> .

# 1. Define Alice
ex:alice  rdf:type    foaf:Person .
ex:alice  foaf:name   "Alice Smith" .
ex:alice  foaf:mbox   <mailto:alice@example.org> .

# 2. Define Bob
ex:bob    rdf:type    foaf:Person .
ex:bob    foaf:name   "Bob Jones" .

# 3. Define the Relationship (Edge)
ex:alice  foaf:knows  ex:bob .
```

#####  SPARQL 

SPARQL is the standard for querying Triple Stores. It looks very similar to SQL but is designed for these triple patterns

Example: "Find the names of everyone that Alice knows"
```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX ex:   <http://example.org/>

SELECT ?friendName
WHERE {
  # 1. Start at Alice
  ex:alice foaf:knows ?friend .

  # 2. From the friend, get their name
  ?friend  foaf:name  ?friendName .
}
```
Visualizing the query:
```
      YOUR QUERY PATTERN                 THE DATA GRAPH
      ------------------                 --------------

      (ex:alice)                         (ex:alice)
          |                                  |
     [foaf:knows]                       [foaf:knows]  <-- MATCH!
          |                                  |
          v                                  v
      (?friend) <----------------------- (ex:bob)
          |                                  |
     [foaf:name]                        [foaf:name]   <-- MATCH!
          |                                  |
          v                                  v
    (?friendName) <------------------- "Bob Jones"

    RESULT: "Bob Jones"
```

***Why use Triple Stores?*** They are designed for Interoperability. If I publish data about "Cars" and you publish data about "Cars," we can merge our datasets instantly because we both used standard URLs for our predicates (e.g., http://auto-schema.org/horsepower)

#### Summary

| Concept | Explanation |
| -- | -- |
| Use Case |"Complex Many-to-Many relationships (Social, Fraud, Logistics)." |
| Property Graph | (Neo4j) Rich nodes and edges. Great for application development | 
| Cypher | Declarative query language that looks like ASCII art patterns |
| Recursive Queries | "The Achilles heel of SQL. Graphs handle ""friends of friends"" effortlessly." |
| Triple Store | (RDF) Academic/Semantic Web model. Subject-Predicate-Object |
| SPARQL | The query language for Triple Stores | 

### Datalog

Datalog or **The Grandfather**: It is an *older, logic-based language* (a subset of Prolog) that underpins many modern systems (like Datomic)

It thinks in terms of **Rules** and **Facts**.
- Fact: `friend(alice, bob)`
- Rule: `friend_of_friend(X, Z) :- friend(X, Y), friend(Y, Z)`.

It is *very powerful* but *harder for humans to read* than Cypher

## Storage and Retrieval

Until now, we have explored data models from the point of view of the end user using them. Now, let us explore how we store the data. What is the structure? What are the pros and cons of the structure? Which storage mechanism is good for transactions vs analytics?

**There a two basic types of storage engines**:
* **Log-structured storage engines**: Treats storage like a **diary**. It is *append-only*. New data (including updates and deletes) is simply written to the end of the file sequentially. It *never modifies old data in place* (Ex: LSM-Trees, RocksDB, Cassandra)
* **Page-oriented storage engines**: Treats storage like a **library**. It *divides data into fixed-size pages* (e.g., 4KB). When you update data, it *finds the specific page and overwrites the existing entry in place*.

The fundamental difference lies in how they handle ***writes*** and ***updates***

### The world's simplest database

Let us start with a simple, *append-only i.e "Log-structured"* storage engine
- Start with the simplest requirement of setting or getting the value that is stored against a particular key i.e **"key-value" pairs**. The value can be anything (JSON object, XML, etc)
- We are using the simplest representation of storage: A **text file**. The value will have to be "stringified" (serialized) and stored
- We can build it using two simple `Bash` functions

The Example: `db_set` and `db_get`
- Imagine a database that is just a ***simple text file*** (`database.txt`).
- **Writing (`db_set`)**: To save data, you simply *append it to the end of the file*. You don't delete or overwrite old data; you just add the new version to the bottom
- **Reading (`db_get`)**: To find data, you look through the file *from the very end (most recent) to the beginning* until you find the key you are looking for
- Visualizing the File: If you updated the value for key `123` three times, the file looks like this:

```
123,{"name":"Hamlet","role":"Prince"}   <-- Oldest version (ignored)
456,{"name":"Claudius","role":"King"}
123,{"name":"Hamlet","role":"Ghost"}    <-- Newer version (ignored)
123,{"name":"Hamlet","role":"Corpse"}   <-- Most recent (The Answer)
```

#### The Fundamental Trade-off

This simple database highlights the core trade-off in storage engines:
1. **Writes are incredibly fast**: Appending to a file is one of the fastest operations a computer can do (`O(1)`).
2. **Reads are incredibly slow**: To find a key, you have to scan the whole file (`O(n)`).

**Solution**: To solve the slow read problem, we need an ***Index***.

***The Golden Rule of Indexing***: An index is an additional structure derived from your data that ***speeds up reads but always slows down writes*** (because you have to update the index every time you write)

> An index is an auxiliary data structure that acts like a shortcut, allowing the database to find specific rows quickly without scanning the entire table

There are many ways to index data in a database storage. One of them is a **Hash Index**

### Hash Indexes 

Hash indexes are usually used in the the ***Log-Structured Approach***
- This is used by storage engines like ***Bitcask*** (used in *Riak*)

How it Works
- **On Disk**: You have the log file (append-only) containing the actual data.
- **In Memory (RAM)**: You keep a Hash Map (like a Python dictionary). This map tells you *exactly where in the file the latest data for each key is located* (the **"byte offset"**).

```
IN-MEMORY (RAM)                      ON-DISK (Hard Drive)
+-----------+----------------+          +----------------------------------+
|    Key    |  Byte Offset   |          |  LOG FILE (Append-Only)          |
+-----------+----------------+          +----------------------------------+
|  user:1   |      0         | -------> | 0:  user:1, {"name": "Alice"}    |
|  user:2   |      64        | -------> | 64: user:2, {"name": "Bob"}      |
|  user:3   |      128       | ---+     | 128: user:3, {"name": "Charlie"} |
+-----------+----------------+    |     | ...                              |
                                  +---> | 250: user:3, {"name": "Chuck"}   |
                                        +----------------------------------+
```

Query flow example: When you want `user:3`, the database checks RAM, sees `offset 250`, jumps instantly to `byte 250` on the disk, and reads the value

#### The hash map data structure

A Hash Map (or Hash Table) is a data structure that stores data in `Key-Value` pairs. It is designed for super-fast lookups, inserts, and deletes—typically `O(1)` (***instant***) time complexity

Think of it like a physical dictionary: You don't read every page to find "Apple"; you go directly to the 'A' section

*How It Works (The Recipe)*: To create a hash map, you need two main ingredients:
- **An Array**: A fixed-size list of "buckets" to store data.
- **A Hash Function**: A mathematical formula that *translates* a **Key** (e.g., `"User123"`) into an **Index number** (e.g., `4`).

*The Process*:
1. Input: You provide a key (`"Apple"`) and a value (`$1.00`)
2. Hash: The function calculates `Hash("Apple") = 3`
3. Store: The data is stored in Bucket #3 of the array.
4. Retrieve: To find "Apple" later, the function calculates 3 again, and you go straight to Bucket #3

*Resizing (The "Growth" Spurt)*
- Hash maps have a **Load Factor** (usually `0.75` or `75%`).
- When the array gets 75% full, the hash map automatically doubles its size.
- It then ***Rehashes*** (re-calculates indexes for) all existing items to fit the new, larger array. This is expensive but necessary to keep lookups fast

*Resolving Conflicts (Collisions)*:
1. **Separate Chaining (The "List" Method)**: Think of each bucket not as a single slot, but as a hook that holds a list. When a collision happens (e.g., "John" and "Sandra" both hash to Bucket 5), you simply attach the new item to the end of the list at that bucket. To find "Sandra" later, you go to Bucket 5 and scan through the short list until you find her. It’s like a hotel where room 5 isn't just one bed, but a hallway of multiple beds
	- Pros: Simple to implement; the table never "fills up" (lists just get longer).
	- Cons: If many keys collide, the list becomes long, slowing down lookups (`O(n)`)
	- Real-world: Used by Java's HashMap
```
 [BUCKET ARRAY]        [LINKED LISTS]
 +---+
 | 0 | --> null
 +---+
 | 1 | --> [Key:"Apple", Val:$1] --> null
 +---+
 | 2 | --> null
 +---+
 | 3 | --> [Key:"Banana", Val:$2] --> [Key:"Pear", Val:$3] --> null
 +---+
 | 4 | --> null
 +---+
```

2. **Open Addressing (The "Next Seat" Method)**: In this method, each bucket holds only one item. If a collision happens (e.g., "John" takes Bucket 5, and "Sandra" also wants Bucket 5), "Sandra" must go find the next available empty slot in the array (e.g., she checks Bucket 6, then Bucket 7). This is often called "Linear Probing." To find "Sandra" later, you start at Bucket 5; if it's not her, you keep checking the next slots until you find her or hit an empty space
	- Sub-methods: 
		1. Linear Probing: If bucket 3 is full, try 4, then 5, until you find an empty spot
		2. Quadratic Probing: If 3 is full, try 3 + 1², then 3 + 2² (skipping slots to reduce clustering)
		3. Double Hashing: Use a second hash function to calculate a new position
	- Pros: Saves memory (no pointers/lists)
	- Cons: The table can get full; performance drops drastically as it fills up
	- Real-world: Used by Python's Dictionary (optimized version)

```
  [BUCKET ARRAY]
      +---+
index | 0 |  (empty)
      +---+
index | 1 |  [Key:"Apple", Val:$1]
      +---+
index | 2 |  [Key:"Banana", Val:$2]
      +---+
index | 3 |  [Key:"Pear", Val:$3]  <- "Pear" originally hashed to index 2, but it was taken by "Banana", so it moved to 3.
      +---+
index | 4 |  (empty)
      +---+
```

#### Need for segmentation and compaction

In log-structured database, since we only ever append (never delete), the file will eventually *run out of disk space*

**Solution**: Segmentation & Compaction
- **Segmentation**: Close the file when it gets too big and start a new one
- **Compaction**: In the *background*, throw away duplicate keys, keeping only the most recent update

*Segmentation followed by compaction example:*

```
segment_1 (Old)         segment_2 (New)         Compacted Segment
---------------         ---------------         -----------------
cat: meow               dog:  woof              cat:  purr
dog: bark               cat:  purr     ===>     dog:  woof
cat: purr               frog: ribbit            frog: ribbit
                                                (Old duplicates removed)
```
*Example of segmentation only:*
```
      TIME
       |
       v

  [ DISK STORAGE ]

  +---------------------------+
  | SEGMENT 1 (Sealed)        |  <-- Oldest File
  |---------------------------|      (Read Only)
  | set key1 = "valueA"       |
  | set key2 = "valueB"       |
  | set key3 = "valueC"       |
  +---------------------------+
             |
             v
  +---------------------------+
  | SEGMENT 2 (Sealed)        |  <-- Newer File
  |---------------------------|      (Read Only)
  | set key4 = "valueD"       |
  | set key2 = "valueNew"     |  <-- Update: "key2" is now "valueNew".
  | del key1                  |      This newer entry overrides Segment 1.
  +---------------------------+
             |
             v
  +---------------------------+
  | SEGMENT 3 (Active)        |  <-- Current File
  |---------------------------|      (Writable)
  | set key5 = "valueE"       |
  | set key6 = ...            |
  |                           |  <-- New writes append here until full
  +---------------------------+
```

#### Crash recovery in hash indexes

Hash indexes (like a standard Hash Map) usually live entirely in RAM for speed. If the computer crashes (power loss), the RAM is wiped, and the index is lost

To recover, the database must rebuild the index in memory by reading the data files that are saved on the disk
1. **The "Slow" Way**: Read the entire database log file from start to finish, noting where every key is located
2. **The "Fast" Way (Snapshots)**: The database periodically *saves a copy of the RAM Index to disk ("Checkpoints").* On restart, it loads this checkpoint instantly instead of reading the whole log

Recovery Explanation (example):
To recover, the system first quickly loads the Snapshot (Step 1) to restore the majority of the index, and then scans the short Log File (Step 2) to catch up on the few recent changes (adding "Key C" and updating "Key A") that happened after the snapshot was taken
```
   [ DISK STORAGE (Persistent) ]                     [ RAM (Volatile) ]

   1. SNAPSHOT FILE                                  3. RECONSTRUCTED INDEX
   (Saved 5 mins ago)                                (Ready for queries)
   +---------------------+                           +---------------------+
   | Key A -> Offset 10  | --(A) Load Fast -->       | Key A -> Offset 90  | *Updated
   | Key B -> Offset 20  |                           | Key B -> Offset 20  |
   +---------------------+                           | Key C -> Offset 50  | *New
              |                                      +---------------------+
              |
              v
   2. LOG FILE (The Tail)
   (Recent writes only)
   +---------------------+
   | Offset 50: Key C... | --(B) Replay Log -->      (Adds Key C to RAM)
   | Offset 90: Key A... | --(B) Replay Log -->      (Updates Key A in RAM)
   +---------------------+
```

#### Pros and cons of hash indexes 

1. ✅ Pros: **fast reads**, **super fast writes**, **easy crash recovery**.
2. ❌ Cons:
	1. **RAM Limitation**: All keys must fit in RAM. If you have billions of keys, this won't work
	2. **No Range Queries**: You cannot scan "all keys between user:1 and user:5." You have to look up each one individually

### SSTables and LSM-Trees

To solve the limitations of Hash Indexes (RAM usage and range queries), we introduce **SSTables (Sorted String Tables)**.  This is the tech behind **LevelDB, RocksDB, Cassandra, and HBase**

Instead of appending data in random order, we require the file to be ***sorted by key***. We call this file an **SSTable**

#### Why Sorting Helps

1. **Sparse Index**: You *don't need every key in memory anymore*. You only need one key for every few kilobytes
	- If you are looking for handiwork, and you know handbag is at offset 1000 and handsome is at offset 2000, you know handiwork must be between them. You can jump there and scan
2. **Efficient Merging**: Merging segments becomes incredibly fast, exactly like the ***MergeSort*** algorithm. You read two files side-by-side and just pick the lowest key

```
File 1 (Sorted)     File 2 (Sorted)      Merged Output
---------------     ---------------      -------------
apple: 1            banana: 5            apple: 1
cherry: 2           date: 6       ===>   banana: 5
elderberry: 3                            cherry: 2
                                         date: 6
                                         elderberry: 3
```

**Further explanation of the mergesort-like merging**

The merge process works exactly like the "merge" step in the Merge Sort algorithm. Since both input files are already sorted, we don't need to load everything into RAM. We just read the files sequentially from start to finish using two pointers (cursors)

The merge process works exactly like the "merge" step in the Merge Sort algorithm. Since both input files are already sorted, we don't need to load everything into RAM. We just read the files sequentially from start to finish using two pointers (cursors).

The Algorithm (Step-by-Step): We place a cursor at the top of both files and compare the current keys.

*Step 1*: Compare apple vs banana
- apple comes first
- Action: Write apple: 1 to output (i.e to Merged output)
- Move: Advance File 1 cursor to cherry

*Step 2*: Compare cherry vs banana.
- banana comes first
- Action: Write banana: 5 to output
- Move: Advance File 2 cursor to date

*Step 3*: Compare cherry vs date
- cherry comes first
- Action: Write cherry: 2 to output
- Move: Advance File 1 cursor to elderberry.

*Step 4*: Compare elderberry vs date
- date comes first
- Action: Write date: 6 to output
- Move: File 2 is now empty (EOF)

*Step 5*: Cleanup
- File 2 is empty, so we simply append everything remaining in File 1
- Action: Write elderberry: 3 to output

#### LSM-Trees

***Writing sorted data to disk is slow*** (you'd have to rewrite the file). So, **LSM-Trees (Log-Structured Merge-Trees)** use a clever trick:

1. **Write to Memory (Memtable)**: When a write comes in, add it to an *in-memory tree* (like a Red-Black tree). This keeps data sorted in RAM.
2. **Flush to Disk (SSTable)**: When the Memtable gets big (e.g., several MB), write it to disk as a *new SSTable file*. Since the tree is already sorted, this *write is sequential (fast!)*.
3. **Read**: Look in the Memtable first. If not there, check the *most recent SSTable on disk*, then the next older one, then the next older one, etc.
4. **Compact**: In the *background*, merge old SSTables into new, larger ones to clean up updates and deletes

*LSM-Trees*:
```
WRITE (Key: "Cherry")
         |
         v
[ IN-MEMORY (Volatile) ]
+----------------------------+
| 1. MEMTABLE                |  <-- Writes land here.
| (Sorted Tree structure)    |      Data is sorted instantly.
| [Apple] -> [Cherry] -> ... |
+----------------------------+
         |
         | (Flush when > 4MB)
         v
[ DISK (Persistent) ] <-- Written as a new SSTable i.e new / most recent segment (Hence, fast write since no update/override)

   2. SEGMENTATION (Immutable Files)
   +-----------+    +-----------+
   | SSTable 1 |    | SSTable 2 |   <-- Old MemTables saved as files.
   | (Sorted)  |    | (Sorted)  |       They pile up over time.
   +-----------+    +-----------+
         \              /
          \            /
           \          /  (Merge Sort)
            v        v
   3. COMPACTION
   +---------------------------------------+
   | NEW MERGED SSTABLE                    |  <-- Background process merges files
   | [Apple] [Banana] [Cherry] [Date] ...  |      to discard old/deleted data.
   +---------------------------------------+
            ^
            | (Lookups use this shortcut)
   4. SPARSE INDEX
   +------------------------+
   | Key "Apple" -> Byte 0  |  <-- Does NOT store every key.
   | Key "Date"  -> Byte 64 |      Only stores the start of every block.
   +------------------------+      (e.g., 1 key per 4KB block)
```
* *SSTable (The Parts)*: These are the specific boxes labeled "SSTable 1", "SSTable 2", and "New Merged SSTable". They are the actual files stored on the hard drive
* *LSM Tree (The Whole):* This is the entire diagram. The "Tree" isn't a single file; it is the combination of the `MemTable (in RAM) + all the SSTables (on disk) + the Compaction process` that manages them.

#### Benefits and drawbacks of LSM-Trees

**Benefits**
- LSM trees offer ***massive Write Throughput*** because they convert random database updates into fast, sequential appends (like writing to a log file)

**Drawbacks**
- The downside is ***Slower Reads and Write Amplification***; looking up a single key requires checking the memory buffer and potentially multiple files on disk (SSTables) to find the latest version
- Additionally, the ***background Compaction process (merging files) burns CPU and disk bandwidth***, which can occasionally cause performance jitters

**Performance Optimization (Bloom Filters)**

To fix the slow read issue, LSM engines use **Bloom Filters**. This is a *small in-memory data structure* that acts like a gatekeeper for each SSTable on disk. Before the database touches the disk, it asks the Bloom Filter: "Does this file contain Key X?" If the answer is "No" (which is 100% accurate), the database skips that file entirely, saving huge amounts of time

Real-World Example: **Cassandra** (used by Netflix and Instagram) uses this architecture to handle millions of writes per second. It ensures that when you "Like" a post, the write is captured instantly (sequential append), even if reading that "Like" back takes a fraction of a millisecond longer.

```
        READ REQUEST (Key: "User_123")
             |
             v
   [ 1. MEMTABLE CHECK ] -> Found? (Return Value)
             | No
             v
   [ 2. BLOOM FILTER CHECK ]  <-- THE OPTIMIZATION
   | "Is User_123 in SSTable 1?" |
   +-------------+---------------+
          |              |
    Answer: "NO"   Answer: "MAYBE"
          |              |
    (Skip Disk!)    (Read Disk)
          |              |
          v              v
     Next SSTable    [ 3. READ SSTABLE 1 ]
```

#### Bloom filters

A Bloom Filter is a *space-efficient data structure stored in RAM* that tells you *if an item is present in a set*. It is **probabilistic**:
1. "No" means the item is definitely not there (100% accurate)
2. "Yes" means the item might be there (False Positive possible)

It is used to avoid expensive disk lookups for data that doesn't exist

*How it Works (Example)*

Think of an empty row of ***bits*** (switches) all set to `0`
- Add "Apple": Hash "Apple" --> positions `2` and `5`. Flip those switches to `1`
- Check "Banana": Hash "Banana" --> positions `2` and `6`
	- Since position 6 is still 0, "Banana" is *definitely not* in the set

```
1. INSERT "Apple" -> Hash -> (Pos 2, Pos 5)
         |
         v
Index:   0  1  2  3  4  5  6  7
Bits:   [0, 0, 1, 0, 0, 1, 0, 0]  <-- The Filter
               ^        ^
               |        |
      2. CHECK "Banana" -> Hash -> (Pos 2, Pos 6)
               |        |
               OK       MISS (0) -> Stop! (Definitely Not Found)
```

*Role in LSM Trees (SSTables)*

In databases like *Cassandra*, ***every SSTable (file on disk) has a corresponding Bloom Filter*** in RAM
- When you request `Get(Key)`, the database checks the Bloom Filter first
- If the filter returns "No", the database skips reading that SSTable entirely
- This prevents the system from wasting time searching through 100 files on disk for a key that doesn't exist

### B-Trees

**B-Trees** is what standard SQL databases (like **MySQL/InnoDB** and **PostgreSQL**) use. It has been the *standard* since the 1970s

#### The Page Concept

Unlike LSM-Trees (which deal with variable-length log segments), ***B-Trees break the database down into fixed-size blocks called Pages (usually 4KB)***

The ***disk*** is treated as ***an array of these pages***

To update data, ***you don't append***; you **find the page** containing the data, update it in memory, and **overwrite** that specific 4KB page on disk

#### The Tree Structure

B-Trees look like a *wide, short tree*
1. **Root Page**: The starting point. Contains a range of keys and pointers to child pages
2. **Leaf Page**: The bottom of the tree. Contains the actual data values

Example: Key lookup for "25" in a B-Tree
```
          [ Root Page: Keys 10, 50 ]
          /            |             \
   (Keys < 10)    (10 <= K < 50)    (Keys >= 50)
   Reference      Points here        Reference
                       |
                       v
           [ Child Page: Keys 20, 30 ]
          /            |             \
   (20 <= K < 30)      ...           ...
      |
      v
[ LEAF PAGE: Contains keys 20, 25, 27 and their values ]
```

#### Branching factor of B-Trees

The Branching Factor is simply the number of children (pointers to other pages) that a single node in the tree holds
- Think of it as the "***width***" of the tree at each step
- In B-Trees, because the page size is large (e.g., 4KB) and keys are small, the branching factor is typically very high (e.g., 500)

*Why It Matters (The "Flat" Tree)*
- A *high branching factor* makes the *tree short and fat*
- If a node can hold 500 pointers, you can index millions of items with a tree that is only 3 levels deep. This is crucial because `Depth = Disk Hops`
- ***A shorter tree means fewer slow disk reads to find your data***

*Example*
- Imagine a B-Tree where every node can hold references to 4 other nodes (`Branching Factor = 4`)
	- Level 1: 1 Node (Root)
	- Level 2: 4 Nodes
	- Level 3: 16 Nodes
	- Level 4: 64 Nodes
- Now, imagine a real database where the Branching Factor is 500:
	- Level 1: 1 Node
	- Level 2: 500 Nodes
	- Level 3: 250,000 Nodes (`500 x 500`)
	- Level 4: 125,000,000 Nodes (`250,000 x 500`)

**Result**: You can reach any of 125 million rows in just 4 hops.

#### Page fragmentation in B-trees

***Updating a key value example using B-Trees***

To update the price of "Apple" from $1 to $2:
- The database starts at the Root, follows the path for "A", and lands on Page 50
- It reads Page 50 into memory
- It changes "Apple: $1" to "Apple: $2" in the memory buffer
- It writes the modified Page 50 back to disk, replacing the old version

```
     ROOT NODE
         |
         | (Search "Apple")
         v
[ DISK: PAGE 50 (Before) ]        [ RAM: BUFFER (Processing) ]
+----------------------+          +-------------------------+
| Key: Apple | Val: $1 |  ------> | 1. Read Page 50         |
| Key: Berry | Val: $5 |          | 2. Change $1 -> $2      |
+----------------------+          +-------------------------+
          ^                                   |
          |                                   |
          |       3. Write Back (Overwrite)   |
          +-----------------------------------+
```

**Fragmentation**

When an update increases the size of a row (e.g., adding text) and the current fixed-size Page is already full, the database must split that page into two new pages; this often results in two pages that are only half-full, wasting significant storage space (known as **Internal Fragmentation**)

```
      STATE 1: FULL PAGE (Efficient)
      +-----------------------------+
      | [Row 1] [Row 2] ... [Row N] |  (100% Used)
      +-----------------------------+
                 |
                 | Update Row 2 (Make it huge!)
                 v
      STATE 2: SPLIT PAGES (Fragmented)
      +-----------------------------+   +-----------------------------+
      | [Row 1] [Row 2 (HUGE!)]     |   | [Row 3] ... [Row N]         |
      | (Unused Space: 40%)         |   | (Unused Space: 50%)         |
      +-----------------------------+   +-----------------------------+
             ^                                 ^
             |___ THIS IS FRAGMENTATION _______|
```

Fragmentation:
1.  **Wastes storage space**, and 
2. **Degrades performance** because the database is forced to perform *more disk I/O to read the same amount of data* scattered across partially empty pages.

#### Reliability of B-Trees

Reliability (The WAL): Since B-Trees overwrite pages on disk, a *crash* during a write could ***corrupt the page*** (Ex: writing only half of the 4KB)

*Optimization / Solution*: **Write-Ahead Log (WAL)**. Before modifying the tree, the DB writes the operation to a simple append-only log file. If the DB crashes, it replays the WAL to restore the B-Tree

```
     1. WRITE TO LOG (WAL)                 2. UPDATE B-TREE (RAM)
     (Fast, Sequential Append)             (Slow, Random Access)

   +---------------------------+          +-------------------------+
   | WAL FILE (On Disk)        |          | B-TREE PAGE (In RAM)    |
   |---------------------------|          |-------------------------|
   | Seq 1: ID:1 = 50 (SAVED!) |          | ID:1 = 50 (Dirty)       |
   +---------------------------+          +-------------------------+
                |                                      |
                |                                      |
                v                                      v
           [ SYSTEM CRASH ] -----------------> [ DATA VANISHES ]

      ---------------------------------------------------------

      3. RECOVERY (On Reboot)
      Database reads WAL --> Sees "ID:1 = 50" --> Re-applies it to B-Tree.
```

#### B-Trees vs LSM-Trees

| Feature | LSM-Trees (Log-Structured) | B-Trees (Page-Oriented) |
| -- | -- | -- |
| Write Speed | **Faster**. Writes are just appending to a file | "**Slower**. Must find the page, overwrite it, and update the WAL." |
| Read Speed | **Slower**. Might have to check Memtable + multiple SSTables | **Faster**. Usually only needs to read 1 or 2 pages (depth of tree is low) |
| Fragmentation | **Low** (compaction runs in background) | **Can be high** (empty space in pages) |
| Typical Use | "**Heavy write workloads** (Cassandra, RocksDB)" | "**General purpose / Heavy reads** (MySQL, PostgreSQL)" |

#### B-Trees vs B+ Trees

*The Core Difference*
- B-Tree: Stores actual data (rows) in every node (internal and leaf). You might find your data at the root without going deeper
- ***B+ Tree***: Stores *data only in leaf nodes*. Internal nodes are just "road signs" containing keys to guide you. Crucially, the *leaf nodes* are connected in a ***Linked List***

*Why B+ Trees are the Standard (99% of DBs)*
1. *More Keys*: Since internal nodes don't hold heavy data, they can hold many more keys (higher branching factor), making the tree shorter
2. *Fast Scans*: To get "All users with ID > 50", you just find "50" and then zip across the leaf nodes using the linked list *(no need to go up and down the tree)*

```
[ 10 | 30 ]           <-- Internal Node (KEYS ONLY)
        /     |     \
       v      v      v
[ 1..9 ] -> [10..29] -> [30..99]  <-- Leaf Nodes (DATA LIVES HERE)
                                      (Connected via Linked List)
```
More detailed diagram:
```
                     [ ROOT NODE ]
                     |  Key: 50  |
                     +-----------+
                    /             \
                   /               \
         [ INTERNAL NODE ]       [ INTERNAL NODE ]
         |   Key: 30     |       |   Key: 70     |
         +---------------+       +---------------+
           /           \           /           \
          /             \         /             \
         v               v       v               v
  +-----------+    +-----------+    +-----------+    +-----------+
  | LEAF NODE | -> | LEAF NODE | -> | LEAF NODE | -> | LEAF NODE |
  | 10 | 20   |    | 30 | 40   |    | 50 | 60   |    | 70 | 80   |
  | (Data)    |    | (Data)    |    | (Data)    |    | (Data)    |
  +-----------+    +-----------+    +-----------+    +-----------+
        ^                ^                ^                ^
        |________________|________________|________________|
           (The "Linked List" for fast range scans)
```

### Other indexing structures

1. **Inverted Index (Full-Text Search)**
- Definition: Used by search engines (*Elasticsearch*, *Lucene*), it maps individual words to the list of documents containing them (like the **index at the back of a book**)
- Example: Searching for "Best Apple" finds documents containing "Best" and intersects them with documents containing "Apple"
```
 [ WORD ]       [ DOCUMENT IDs ]
"Apple"   -->  [ Doc 1, Doc 3, Doc 5 ]
"Banana"  -->  [ Doc 2 ]
"Best"    -->  [ Doc 1, Doc 4 ]
```
- Pros: Extremely fast for keyword search and boolean queries (`AND/OR`)
- Cons: Expensive updates (modifying one document requires updating lists for every word in it)

2. **R-Tree (Spatial / Multi-dimensional)**
- Definition: A tree structure that groups nearby objects into **Bounding Boxes** (rectangles); primarily used for geospatial data (*Maps*, *Uber*, *Yelp*)
- Example: "Find all restaurants within 5 miles." The database checks high-level rectangles first, ignoring huge areas that don't overlap your search radius

```
 [ Root Box (City) ]
     |
     +--- [ Box A (North) ] ---> Contains: {Point 1, Point 2}
     |
     +--- [ Box B (South) ] ---> Contains: {Point 3, Point 4}
```
- Pros: Efficient for "nearest neighbor" and geographical range queries
- Cons: Re-balancing the tree on updates is complex and computationally heavy

3. **Bitmap Index (Low Cardinality / Analytics)**
- Definition: Uses a string of bits (`0`s and `1`s) to represent the presence of value; ideal for columns with very few unique values (e.g., Gender, Status, Color)
- Example: Filtering for `Color=Red` is just a fast bitwise operation on the "Red" array

```
Rows:      1  2  3  4  5
-------------------------
Is Red?    1  0  0  1  0  (Row 1 & 4 are Red)
Is Blue?   0  1  0  0  1
```
- Pros: extremely small size and super fast bitwise logical operations (AND/OR)
- Cons: Terrible for columns with many unique values (e.g., User IDs); locking can be an issue during updates

### Transaction processing vs analytics

#### OLTP vs OLAP

In the early days of databases, we used the same database for everything:
- The Transaction: A customer buys an item. We subtract 1 from inventory and add a record to the sales table
- The Analysis: The CEO wants to know, "Which product sold the most in February?"

The problem is that the CEO's query requires scanning millions of rows. While the database is busy crunching those numbers, the website *slows down*, and customers can't buy items

To solve this, the industry split database usage patterns into two categories:

**OLTP (Online Transaction Processing)**
- This is the **"front line"** of your application
- Who uses it? The **end-user** (via the web app)
- Access Pattern: **Random access**. Look up a **small number of records by key**
- Latency: Must be **extremely fast (milliseconds)**
- Examples: MySQL, PostgreSQL, Oracle

**OLAP (Online Analytical Processing)**
- This is the **"back office"** intelligence
- Who uses it? **Business analysts**, **Data Scientists**
- Access Pattern: **Scans over huge numbers of records**, usually *fetching only a few columns* (e.g., "Sum of price")
- Latency: Can take **seconds, minutes, or even hours**
- Examples: Redshift, Snowflake, BigQuery, ClickHouse

| Feature | OLTP (Application DB) | OLAP (Data Warehouse) |
| -- | -- | -- |
| Main Operation | Read/Write one row | Scan millions of rows |
| Data Size | Gigabytes to Terabytes | Terabytes to Petabytes |
| Bottleneck | Disk Seek (jumping around) | Disk Bandwidth (streaming data) |
| Source of Truth | The current state of the world | History of events |

#### Data warehousing

An OLTP database is optimized for high availability and low latency transactions. You don't want a massive analysis query to lock your tables and stop users from logging in

So, companies build a **Data Warehouse**. This is a *separate database* that contains a *read-only copy of the data* from all the various OLTP systems (Billing DB, User DB, Inventory DB)

**The ETL Process**: Getting data out of the OLTP systems and into the Warehouse is called ***Extract-Transform-Load*** (ETL)

```
  [ OLTP DB 1 ]        [ OLTP DB 2 ]         [ OLTP DB 3 ]
 (User Profiles)      (Billing System)      (Inventory App)
       |                     |                     |
       +----------+----------+---------------------+
                  |
             PERIODIC EXTRACT (ETL)
       (Clean up messy data, format it)
                  |
                  v
       +-------------------------+
       |     DATA WAREHOUSE      |  <-- Optimized for Analytics
       | (Redshift / Snowflake)  |
       +-------------------------+
                  |
         [ Analyst Queries ]
      "Show me avg spend by age"
```

#### Stars and snowflake schemas for analytics

In OLTP (application code), we usually normalize data heavily to avoid duplication. In Analytics, we don't care about duplication; we care about **easy querying** (i.e ***data is usually denormalized and duplicated in OLAP***)

**The Star Schema**

The standard for analytics is the Star Schema.
- **Fact Table (Center)**: This table is HUGE. It captures "events." Every time something happens (a view, a click, a purchase), a row is added here
- **Dimension Tables (Points of the Star)**: These are smaller tables that describe the *"who, what, where"* of the event

***Why is it called a Star?*** Because when you visualize it (like above), the Fact table is in the middle surrounded by Dimension tables

```
          [ dim_product ] <-------+
          | product_id  |         |
          | sku, name   |         |
          | category    |         |
          +-------------+         |
                                  |
                           [ fact_sales  ]
                           | date_key    | ---> [ dim_date ]
                           | product_id  |      | date_key, year |
                           | store_id    |      | month, holiday |
                           | customer_id |
                           | quantity    |
                           | price_sold  |
                           +-------------+
                                  |
          [ dim_store ] <---------+
          | store_id    |
          | city, state |
          | manager     |
          +-------------+
```

**Benefit of using a star schema**

The primary benefit of the Star Schema in OLAP (Online Analytical Processing) is **Faster Query Performance due to simplicity**

By separating data into a central Fact Table (metrics/numbers) and surrounding Dimension Tables (context/attributes), it **minimizes the number of "Joins" required to answer a question** 

Unlike normalized schemas (Snowflake), the dimension tables here are denormalized (flat), meaning the database can connect the data in a ***single hop***

**Snowflake Schema**

A variation where the dimension tables are broken down further (e.g., `dim_product links` to a `dim_brand` table). It's "more normalized" but harder for analysts to query, so Star Schema is usually preferred

### Column oriented storage

In a typical OLTP database (MySQL/Postgres), storage is **Row-Oriented**. If you have a table with 50 columns, all 50 columns for `User:1` are stored next to each other on the disk

```
Row 1: | ID:100 | Date:2024-01-01 | Product:Apple  | Price:$5 | ... (46 more cols) |
Row 2: | ID:101 | Date:2024-01-01 | Product:Banana | Price:$3 | ... (46 more cols) |
Row 3: | ID:102 | Date:2024-01-02 | Product:Cherry | Price:$9 | ... (46 more cols) |
```

**The Problem with Row-Oriented for Analytics**
- Imagine you want to calculate the Average Price: `SELECT AVG(Price)` FROM sales.
- In a Row-Store, the database has to load Row 1, skip over the ID, Date, and Product just to read the Price. Then load Row 2, skip everything to get the Price
- You are *loading huge amounts of useless data into memory just to discard it*

**The Solution: Column-Oriented Storage**
- In a ***Column-Store*** (like `Redshift` or `Parquet` files), we *store all the values from one column together*

```
File 1 (IDs):      | 100, 101, 102, 103... |
File 2 (Dates):    | 2024-01-01, 2024-01-01, 2024-01-02... |
File 3 (Products): | Apple, Banana, Cherry... |
File 4 (Prices):   | $5, $3, $9... |
```

**The Benefit**: To calculate `AVG(Price)`, the database only reads `File 4`. It ignores the other files completely. If your table has 100 columns and you only need 5, you do 95% less I/O!

**How to read a row in column oriented storage?**: To read a single row (e.g., Row 10), the database must go to the 10th position in every separate column file and stitch those values together. This is why ***column stores are slow for retrieving single records;*** finding one "row" requires doing ***random disk seeks in many different files*** (one for Name, one for Age, one for Date, etc.) instead of just reading one block like in a row-store

```
WANT: Row 3 (User C)

      [ Col: ID ]    [ Col: Name ]    [ Col: Age ]
      1. User A      1. Alice         1. 25
      2. User B      2. Bob           2. 30
----> 3. User C      3. Charlie       3. 35   <-- Fetch Index 3 from ALL
      4. User D      4. Dave          4. 40       files to build the row.
```

#### Column Compression

Column storage isn't just about *skipping data*; it's about **squeezing data**. Because a single column often contains ***repetitive data*** (e.g., "Country" column has "USA" listed millions of times), it *compresses extremely well*!

**Bitmap Encoding**: If a column has low cardinality (few unique values, like "Currency"), we can replace the data with simple bits

Example: A column color with values Red, Red, Blue, Red, Blue
```
Row Approach:
0: Red
1: Red
2: Blue
3: Red
4: Blue

Bitmap Approach: Instead of storing strings, we store one bitmap per distinct value:
Value "Red":  1, 1, 0, 1, 0  (Is it Red? Yes, Yes, No, Yes, No)
Value "Blue": 0, 0, 1, 0, 1  (Is it Blue? No, No, Yes, No, Yes)
```

Bitmap Encoding *reduces the data size massively* and *allows for crazy fast logical operations* (bitwise `AND/OR`)

**Benefits of Column-Oriented Storage**
- Faster Analytics: Instead of reading entire rows (which include data you don't need), the database reads only the specific columns required for the query (e.g., "Sum of Revenue"). This drastically reduces Disk I/O
- Vectorized Processing: Modern CPUs can process a column of data (an array of integers) in batches using SIMD (Single Instruction, Multiple Data) instructions, making calculations lightning fast

**Benefits of Compression (in Column Stores)**
- High Compression Ratios: Shrink files by 10x or more.
- Faster Reads: Smaller files mean less data to move from Disk to RAM. The CPU cost to unzip the data is far smaller than the time saved by not waiting for the disk

#### Sort Order in Column Stores

In a column store, you ***can't sort every column individually***, because you'd lose the link between `Row 1: Apple` and `Row 1: $5`. However, you can choose a **primary sort order** for the data
- If you sort the data by `Date`, then in the `Date` column, all the `2024-01-01`s will be grouped together. This makes compression even better (**Run-Length Encoding)**

Run-Length Encoding Example: 
- Raw Data (Sorted by Date): `2024-01-01`, `2024-01-01`, `2024-01-01`, `2024-01-02`, `2024-01-02`
- Compressed: `2024-01-01: 3` (There are 3 of these) `2024-01-02: 2` (There are 2 of these)
- This allows the database to *scan billions of rows in seconds*

## Replication

### Why do we replicate?

Replication means ***keeping a copy of the same data on multiple machines (nodes) that are connected via a network***

Why would you want to duplicate data?
1. **High Availability (Reliability**): If one machine goes down (crashes, power cut), the others can keep the system running
2. **Latency (Performance)**: If you have users in London and New York, you can put a replica in London and one in New York so users can access data locally
3. **Scalability (Read Throughput)**: If you have too many read requests for one machine to handle, you can add more replicas to share the load

**The Core Difficulty**: If data never changed, replication would be easy (just copy the files once). The difficulty lies in handling changes (writes) and ensuring all replicas agree on the data

### Leaders and Followers

The most common solution for handling writes is the **Leader-Based Replication** (also known as **Master-Slave** or **Active/passive** replication)

**How it Works**:
- One replica is designated the Leader
	- ***All Writes must go to the Leader***
- The other replicas are Followers
	- Whenever the Leader writes new data locally, it ***sends the data change (replication log) to all Followers***
- Reads can happen from the Leader or any Follower

```
     [ Client ]
          |       ^ 
(1) Write Request | (3) Ack (Success)
          |       |
          v       |
    +--------------+  (2) Replication Log   +-------------+
    |   LEADER     | ---------------------> |  FOLLOWER   |
    +--------------+                        +-------------+
```

### Synchronous vs Asynchronous Replication

When the Leader sends data to a Follower, *does it wait for the Follower to say "I got it"?*

1. **Synchronous Replication**: The Leader waits for the Follower to confirm the write before telling the client "Success."
	- Pros: ***Guaranteed durability***. If the Leader dies, the Follower definitely has the data
	- Cons: If the ***Follower crashes*** or the network is slow, the ***Leader freezes*** and cannot accept writes

2. **Asynchronous Replication (Most Common)**: The Leader sends the message to the Follower but immediately tells the client "Success." *It doesn't wait*
	- Pros: **Fast**. The Leader keeps working even if Followers are slow or dead
	- Cons: If the ***Leader crashes before the data reaches the Follower, that data is lost forever***

*Note: In real systems, usually 1 follower is synchronous, and the rest are asynchronous. This is called **"Semi-Synchronous"***

*Synchronous replication*:
```
       CLIENT              LEADER (Master)          FOLLOWER (Slave)
         |                       |                        |
      1. |----- Write "A" ------>|                        |
         |                       |                        |
         |                       | 2. Send "A" ---------->|
         |      (WAITING)        |                        |
         |                       |                        | 3. Write "A" to Disk
         |                       |                        |
         |                       |<-------- ACK ----------| 4. Confirm
         |                       |                        |
      5. |<---- "Success!" ------|                        |
         |                       |                        |
```
*Asynchronous replication*:
```
       CLIENT              LEADER (Master)          FOLLOWER (Slave)
         |                       |                        |
      1. |----- Write "A" ------>|                        |
         |                       |                        |
      2. |<---- "Success!" ------|                        |
         |                       |                        |
         |                       | 3. Send "A" ---------->| (Background)
         |                       |                        |
         |                       |                        | 4. Write "A" to Disk
```

### Setting up new followers

You *cannot just copy the database file while the system is running* (the file is constantly changing). 

The Process:
- Take a consistent **Snapshot** of the Leader's database *at a specific point in time*
- Copy the snapshot to the new Follower node
- The Follower connects to the Leader and asks for all changes that happened after the snapshot was taken
- Once the Follower catches up, it is now in sync

```
       LEADER (Active)                       NEW FOLLOWER (Empty)
      (Current Log Offset: 500)             (Starts from Scratch)
         |                                         |
         | 1. Create Snapshot (at Offset 100)      |
         | [BACKUP FILE] ------------------------->| 2. Load Snapshot
         |                                         | (Follower is now at Offset 100)
         |                                         |
         |           (Normal writes continue...    |
         |            Leader is now at 550)        |
         |                                         |
         |<---------- 3. REQUEST UPDATE -----------|
         | "Give me changes from Offset 101"       |
         |                                         |
         |---------- 4. REPLICATION LOG ---------->|
         | (Sends changes 101...550)               |
         |                                         |
         |                                         | 5. Apply Log
         |                                         | (Follower reaches Offset 550)
         |                                         |
      [ IN SYNC ] <--------------------------- [ IN SYNC ]
```

### Handling Node Outages

Handle two types:
1. **Follower Failure (Catch-up Recovery)**: If a Follower crashes and restarts:
	- It looks at its own log to see the last transaction it processed
	- It connects to the Leader and requests all changes since that point

```
      LEADER (Active)                       FOLLOWER (Crashed)
      (Log: 50, 51, 52, 53)                 (Disk Log stops at: 50)
         |                                         X
         | (Writes 51, 52, 53...)                  X (Down for 5 mins)
         |                                         X
         |                                         |
         |                                         | (RESTART)
         |<--------- "I have up to 50" ------------| 1. Handshake
         |                                         |
         |---------- "Here is 51, 52, 53" -------->| 2. Catch Up
         |                                         |
         |                                         | (Write to Disk)
      [ IN SYNC ] <--------------------------- [ IN SYNC ]
```

2. **Leader Failure (Failover)**: If the Leader dies, chaos ensues. We need a Failover:
	- *Detection*: Usually a timeout. Nodes "ping" each other; if the Leader is silent for 30 seconds, it is presumed dead.
	- *Election*: The remaining Followers vote to pick a new Leader (usually the one with the most up-to-date data)
	- *Reconfiguration*: The system tells all clients to send writes to the new Leader

```
      OLD LEADER (A)           FOLLOWER (B)            CLIENTS
         |                        |                       |
         X (CRASH!)               |                       |
         X                        |? (Where did A go?)    | (Writes fail)
         X                        |                       |
         X                        | 1. ELECTION WON       |
         X                        | (B becomes Leader)    |
         X                        |                       |
      (Offline)                   | <---- Writes ---------| 2. New Routing
                                  |                       |
      (Reboots later)             |                       |
      "I am boss!" -> (NO!) ----> |                       |
      (Demoted to Follower)       |                       |
```

**The "Split Brain" Problem:** Sometimes, the old Leader wasn't dead, just slow (network lag). It comes back and thinks it is still the Leader. Now you have two Leaders accepting conflicting writes. *The system must ensure the old Leader steps down (often by "fencing" or shutting it down)*

### Implementation of Replication Logs

**How exactly does the Leader send changes?**

1. **Statement-based (`SQL`)**:
	- Ex: Leader sends `INSERT INTO users VALUES (NOW(), RAND())`
	- Problem: `NOW()` and `RAND()` generate different values on the Follower. This causes data divergence. ***Generally avoided***
2. **Write-Ahead Log (WAL) Shipping**:
	- Leader sends the ***low-level byte changes*** (e.g., "Bytes 50-60 in disk block 10 changed to X")
	- Problem: ***Tied to the storage engine version***. You can't replicate from Postgres 12 to Postgres 13 easily
3. **Logical (Row-based) Log**:
	- Leader sends a description of the row change: Log: `For table 'users', ID 50: set name='Alice'`
	- **Best approach: Decoupled from storage engine internals**
		- A *standard, engine-agnostic description* of the row change

```
      CLIENT QUERY
      "UPDATE users SET login = NOW() WHERE id = 1"
               |
               v
      [ LEADER DATABASE ]
      1. Execute SQL -> Calculate NOW() = "12:00:00"
      2. Identify Row ID: 1
               |
               v
      [ LOGICAL LOG ENTRY ]  <-- The Replicated Format
      +-----------------------------------------+
      | Type: UPDATE                            |
      | Table: users                            |
      | Key: {id: 1}                            |
      | Changes: {login: "12:00:00"}            |
      +-----------------------------------------+
               |
               v
      [ FOLLOWER DATABASE ]
      1. Read Log -> Find user with ID 1
      2. Set login to "12:00:00" (Exact Match)
```

### Problems with lag during replication

**`Lag = delay`**

In ***Asynchronous replication*** (the most common setup), the Follower is always a few steps behind the Leader. This delay is called **Replication Lag**

Usually, lag is small (<1s). But during high load, it can grow to minutes. This leads to **Eventual Consistency**: if you stop writing, eventually all followers will match. But in the meantime, *weird things happen*

#### Problem A: Reading Your Own Writes

Scenario:
- User updates their profile (Write -> Leader)
- User refreshes the page immediately (Read -> Slow Follower)
- The Follower hasn't received the update yet
- User sees their old profile and panics, thinking the update failed
- **Solution**: ***Read-After-Write Consistency*** 
	- We need a *guarantee that a user will always see their own updates*
	- *Technique 1*: If the user is *reading their own profile, always read from the Leader*. If reading other people's profiles, read from Followers
	- *Technique 2*: Track the *timestamp of the last update by the user*. If a Follower's data is older than that timestamp, forward the read to the Leader (or wait)

```
     USER (Client)
         |
         | 1. UPDATE Profile ("My Name is Bob")
         |--------------------------------------> [ LEADER NODE ]
         |                                             |
         |                                             | (Replication Lag: 1s)
         | 2. READ Profile                             |
         |    (Load Balancer routes to Follower)       | 3. Sync (Pending...)
         v                                             v
   [ FOLLOWER NODE ] <---------------------------- ( X )
   (Still has "My Name is Alice")
         |
         | 4. RETURNS "Alice"
         |-------------------> User: "Why didn't my update save?!"
```
```
        USER (Client)
      [ Last Write: 10:00:05 ]  <-- Client remembers write time/cookie
             |
             | 1. READ Profile (Time: 10:00:06)
             v
      [ ROUTING LOGIC / API ]
      "Did this user write recently?"
             |
      +------+-------+
      | YES (< 1min) |  ---------------->  [ LEADER NODE ]
      +--------------+                     (Has "Bob" - SUCCESS)
             |
             | NO (> 1min)
             v
      +--------------+
      |  Safe to use |  ---------------->  [ FOLLOWER NODE ]
      |   Follower   |                     (Eventually Consistent)
      +--------------+
```

#### Problem B: Monotonic Reads (Moving Backward in Time)

Scenario:
- User reads a comment. The request hits Follower A (fast), which has the comment. User sees the comment
- User refreshes. The request hits Follower B (slow), which doesn't have the comment yet
- To the user, the comment just vanished. They *effectively moved backward in time*
- **Solution**: **Monotonic Reads**
	- *Technique*: Ensure that *a specific user always reads from the same replica*
		- We can ***hash** the User ID to pick a replica* (e.g., `User 123` always goes to Follower A)

```
User updates: "Hello!"
   |
   +--> Leader (Has "Hello!")
          |
          +--> Follower A (Fast) -> Has "Hello!"
          +--> Follower B (Slow) -> Empty
...
User Reads (1): Hits Follower A -> Sees "Hello!" (Happy)
User Reads (2): Hits Follower B -> Sees Empty    (Confused - "It disappeared!")
```
```
        USER
         |
         | (Hash: UserID % Nodes = 1)
         v
   [ LOAD BALANCER ]
      |
      | (Always routes to Node 1)
      v
[ FOLLOWER 1 ]       [ FOLLOWER 2 ]
(Has "Hello")   (Lagging...)
      ^
      |
      +-----> USER (Always sees "Hello") 
      
      (Even with refreshes, it reads from FOLLOWER 1 and hence, 
      data does not 'disappear')
```

#### Problem C: Consistent Prefix Reads

This issue *mostly happens in **sharded (partitioned) databases*** where *data is **split** across different **independent** groups of nodes*

Scenario:
- Mr. A asks: "How is the weather?" (Partition 1)
- Mr. B replies: "It is sunny." (Partition 2)
- Because of network lag, the replication for Partition 2 arrives at the observer before Partition 1
- Observer sees:
	- Mr. B: "It is sunny."
	- Mr. A: "How is the weather?"

This violates **causality**. The answer arrived before the question!

> Causality ensures that if Event A influences Event B, the entire system sees A happen before B; the "Effect" must never appear before the "Cause"

```
      USER A              USER B               USER C (Observer)
      (Send Q)            (Send Ans)           (Reads Feed)
         |                    |                      |
      1. "How are you?" ----> |                      |
         |                    |                      |
         |                 2. "Good!" -------------> | Sees: "Good!" (Wait, what?)
         |                                           |
         |-----------------------------------------> | Sees: "How are you?"
                                                     |
                                                     (CONFUSED: Answer came first)
```

**Solution**: **Consistent Prefix Reads**

*Technique*: If a ***sequence of writes is causally related*** (the reply depends on the question), they must be ***written to the same partition***. This ensures they are replicated in the correct order

```
      [ REPLICA NODE ]
      (Receives data out of order from Leader/Network)

      1. RECEIVED: [ Msg #2: "Answer" ]
         (Metadata: "I depend on Msg #1")
             |
             v
      [ HOLDING BUFFER ]  <-- STOP! Msg #1 is missing.
      | [ Msg #2 (Pending) ]  (User sees NOTHING yet)
      |
      | ... (Waiting) ...
      |
      2. RECEIVED: [ Msg #1: "Question" ]
             |
             v
      [ PROCESSOR ]
      1. Commit Msg #1 (Order restored)
      2. Release Msg #2 (Dependency met)
             |
             v
      [ READABLE STORAGE ]
      +------------------------+
      | 1. "Question?"         |  <-- Observer sees correct order
      | 2. "Answer!"           |      (Or sees both appear at once)
      +------------------------+
```

### Multi-Leader replication

**Multi-Leader Replication** (also called **Master-Master** or **Active/active** replication): Instead of one leader, you have *two or more leaders*. *Each leader accepts writes, and then they forward those writes to each other*

- In the previous "Leader-Follower" model, there was only one boss (Leader). If you wanted to write data, you had to talk to that one specific server.
- The Problem: *If that one leader goes down, or if you are on the other side of the world from it, you can't write (or writing is very slow)*. We need to elect a new leader and this takes time -- ***failover mechanism delay*** (or worse, data is lost)

```
     [ Client A ]                      [ Client B ]
           |                                 |
           v                                 v
    +-------------+                   +-------------+
    |  LEADER  1  | <---------------> |  LEADER  2  |
    +-------------+   (Replication)   +-------------+
```
- Client A writes to Leader 1
- Client B writes to Leader 2
- Leader 1 and Leader 2 swap notes in the background to sync up

#### Use Cases for Multi-Leader replication
 
**When do we actually use this?**

You *rarely* use Multi-Leader replication within a *single* datacenter (it's too messy). It is usually used in three specific scenarios:

1. **Multi-Datacenter Operation**
	- Imagine you have a database in London and another in New York
	- Single Leader: All writes from London must travel across the ocean to New York (Leader). This is slow
	- Multi-Leader: Users in London write to the London Leader (fast). Users in New York write to the NY Leader (fast). The two datacenters sync asynchronously

```
      Datacenter: London            Datacenter: New York
    +--------------------+        +--------------------+
    | [User] -> [Leader] | <----> | [Leader] <- [User] |
    +--------------------+        +--------------------+
         (Fast Write)                 (Fast Write)
```

2. **Clients with Offline Operation**
	- Think of the Calendar app on your phone
	- You can add an event while you are on an airplane (Offline)
	- Your phone acts as a local "Leader" (it accepts writes)
		- Every device (laptop, phone) acts as a "Leader"
	- When you get internet back, your phone (Leader 1) syncs with the Google Server (Leader 2)
	- This is technically multi-leader replication
```
      [ USER'S LAPTOP ]                   [ SERVER / CLOUD ]
      (Leader A - Local DB)               (Leader B - Central DB)
             |                                    |
   1. OFFLINE MODE (Airplane)                     |
      User: Edit Doc "A -> B"                     |
      Action: Save to Local DB                    |
      Status: SUCCESS (Queue: 1)                  |
             |                                    |
             X  (Network Cut)                     X
             |                                    |
   2. ONLINE MODE (WiFi On)                       |
             |                                    |
      [ ASYNC REPLICATION ] ------------------->  |
      "Here are my offline edits"                 |
             |                                    | 3. MERGE CHANGES
             |                                    | (Server applies "A -> B")
             |                                    |
             | <--------------------------------- |
      "Here are edits you missed"                 |
             |                                    |
         [ SYNCED ]                           [ SYNCED ]
```

3. **Collaborative Editing**
	- Tools like Google Docs
	- When you type, your browser applies changes instantly
	- Your colleague types, their browser applies changes instantly
	- Both browsers act as leaders and sync changes to merge the document
	- Algorithms & Tools
		1. **CRDT (Conflict-Free Replicated Data Type)**:
			- Concept: Data structures that can always be merged without conflicts
			- Example: RGA (Replicated Growable Array) or Yjs (a popular library)
			- Pros: Works offline, decentralized (multi-leader)
			- Cons: ***Slightly higher memory usage*** (storing those IDs)
			- Summary: Keeps the algorithm simple (just merge IDs) but makes the data complex (every character needs a unique ID)
		2. **Operational Transformation (OT)**:
			- Concept: The "Google Docs" way. A central server modifies (transforms) User B's operation: "User A added 1 character, so I will change User B's 'Insert at 3' to 'Insert at 4'."
			- Pros: ***Memory efficient***
			- Cons: ***Extremely complex to implement without a central server*** (Requires a smart server to constantly adjust indices)

**CRDT Deep Dive**: 
- ***The Problem: "Index Shifting"***
	- In a multi-leader system (like two users editing a doc offline), using simple array indices (e.g., "Insert at Index 2") fails
	- If User A adds text at the start of the document, all the indices shift right
	- If User B simultaneously adds text at the end (using the old indices), their text lands in the wrong spot because they didn't know the indices changed
- ***The Solution: CRDTs (Conflict-Free Replicated Data Types)***
	- Instead of using array indices (which change), **CRDTs assign a Unique, Immutable ID (often a fractional number or a decimal string) to every character**
	- Old Way (Indices): "Insert 'X' at Index 5." (Fails if Index 5 is now Index 6)
	- CRDT Way (Unique IDs): "Insert 'X' between ID 0.5 and ID 0.6." (Always works, no matter how much text surrounds it)

```
Scenario: Two users edit "CAT". User A types "H" (Chat). User B types "S" (Cats).

INITIAL STATE: [ "C", "A", "T" ]
Indices:         0    1    2

      USER A (Leader 1)                   USER B (Leader 2)
      ACTION: Insert "H" at Index 1       ACTION: Insert "S" at Index 3
      RESULT: "C H A T"                   RESULT: "C A T S"
              (Indices shift!)

               | (Sync Changes)                    | (Sync Changes)
               v                                   v
      RECEIVES: "Insert S at Index 3"     RECEIVES: "Insert H at Index 1"

      PROBLEM: Index 3 is now "T"         PROBLEM: Index 1 is fine here,
      (because "H" pushed it).            but imagine if B deleted "C"!
      (The "S" will now push "T")        (The "H" will now push "A" -- we're fine)
			
      RESULT: "C H A S T" (WRONG!)        RESULT: "C H A T S" (Correct)
               ^
               |__ "S" landed in the middle!
                   Expected: "CHATS"
```
```
The CRDT Solution (Fractional Indexing)
CRDTs give "A" the ID 1.0 and "T" the ID 2.0.

      INITIAL STATE: [ "C" (1.0), "A" (2.0), "T" (3.0) ]

      USER A                              USER B
      Insert "H" between C & A.           Insert "S" after T.
      Math: (1.0 + 2.0) / 2 = 1.5         Math: (3.0 + 1.0) = 4.0

      OP: Insert "H" with ID 1.5          OP: Insert "S" with ID 4.0
               |                                   |
               | (Sync Changes - Order by ID)      |
               v                                   v
      SORTED LIST (Merged):
      1.0 -> "C"
      1.5 -> "H"  (User A's insert)
      2.0 -> "A"
      3.0 -> "T"
      4.0 -> "S"  (User B's insert)

      FINAL RESULT: "CHATS" (Correct!)
```

**Operational Transformation (OT) Deep Dive**
- Operational Transformation (OT) is the algorithm used by **Google Docs.** It relies on a **central server** to act as a ***traffic cop.***
- When two users edit at the same time, the ***server doesn't just apply their commands blindly***
	- It **calculates the difference caused by the first user** and **adjusts (transforms) the second user's command** so it still makes sense in the new version of the document
	- The Example: "CAT" --> "CHATS"
	- Scenario: User A wants to turn "CAT" into "CHAT" (Insert "H" at index 1)
		- User B wants to turn "CAT" into "CATS" (Insert "S" at index 3)
		- The Conflict: If User A goes first, the string becomes "CHAT" (4 letters)
		- If we then run User B's raw command ("Insert S at 3") on this new string, the "S" lands in the wrong spot because the letters shifted
		- *The Transformation:The server sees User A added 1 letter. It tells User B's operation: "Hey, everything shifted right by 1. You wanted Index 3, but now you must use Index 4."*
```
              INITIAL STATE: [ C, A, T ]
                       0  1  2

      [ USER A ]                     [ USER B ]
      Op A: Insert "H" @ 1           Op B: Insert "S" @ 3
            |                              |
            v                              v
      [ SERVER (The Arbitrator) ] <--------+
      1. RECEIVE & EXECUTE Op A
         String becomes: "C H A T"
                          0 1 2 3

      2. TRANSFORM Op B (The Magic)
         Server Logic: "Op A added 1 character before Index 3."
         Calculation:  New Index = Old Index (3) + 1 = 4

         New Command:  Op B' -> Insert "S" @ 4

      3. EXECUTE Op B'
         String becomes: "C H A T S" (Correct!)
                          0 1 2 3 4
```

#### Handling Write Conflicts

This is the **biggest headache in Multi-Leader replication**. In a Single-Leader system, if two users try to change the same record at the same time, the Leader just queues them up: "User A goes first, then User B."

```
Title is "Zero"

User A -> Leader 1:         User B -> Leader 2:
"Set Title = 'A'"           "Set Title = 'B'"
       |                           |
    (Success)                   (Success)
       |                           |
       +-------> Conflict! <-------+
     (Leader 1 tells Leader 2 "It's A")
     (Leader 2 tells Leader 1 "It's B")
```

*Conflict Resolution Strategies*: Since the conflict already happened, we must resolve it.
1. **Last Write Wins (LWW)**
	- Give every write a *timestamp*
	- If `"Title=A"` happened at `12:00:01` and `"Title=B"` happened at `12:00:02`, B wins. *A is discarded silently*
	- Pros: **Simple**
	- Cons: **Data loss** (User A's work just vanishes).
2. **On-Read Resolution (Merge Values)**
	- Keep both values. The field becomes `Title = ["A", "B"]`
	- The next time a user reads this record, the app shows both and asks the user: "Which one is correct?" (Like a Git merge conflict)
3. **Custom Logic (Code)**
	- You write a snippet of code to handle it automatically
	- Example: If the conflict is in a "Cart" application, and User A adds "Pear" and User B adds "Apple", the resolution logic is Add Both (Result: Pear + Apple)

#### Multi-Leader replication topologies

*That is: How do the leaders talk to each other (to swap / share each other's data)?*

1. **All-to-All**
	- Every leader talks to every other leader
	- Pros: Robust. If one link fails, messages can go around another way
	- Cons: Messages can arrive out of order (Concept: Causality)
		- You might get an "Update record" message before the "Create record" message
2. **Circular (Ring)**
	- Leader 1 -> Leader 2 -> Leader 3 -> Leader 1
	- Pros: Simple to manage
	- Cons: Single Point of Failure. If Leader 2 dies, Leader 1 cannot talk to Leader 3. The loop is broken
3. **Star**
	- One central Root Leader sends to all other Leaders
	- Cons: Also has a single point of failure (the central node)

```
    All-to-All                 Circular           Star
   
    A ----- B                  A ---> B             B
    | \   / |                  ^      |             |
    |   X   |                  |      v           A-C-D
    | /   \ |                  D <--- C             |
    C ----- D                                       E

"X" meaning:
Wires cross, no node here
```

### Leaderless replication

This architecture became famous because of **Amazon's internal Dynamo system**. It is now used by databases like **Cassandra, Riak**, and **Voldemort**

**What is Leaderless Replication?**

In previous models, you had a "Boss" (Leader). If the Boss was down, you couldn't write. In *Leaderless Replication, there is no boss.* ***All replicas are equal***

- **The Rule**: The client sends the write request to all replicas (or a *coordinator node* does it).
- **The Success Condition**: The client doesn't wait for everyone to say "OK." ***It just waits for a specific number of them to say OK*** (e.g., 2 out of 3)

#### Writing to the DB when a Node is Down

Imagine you have 3 Replicas. *Replica 3 crashes*. ***In a Leader system, writes would fail***. In a **Leaderless system, writes proceed!**

The Scenario:
- Client sends `"Set Key=A"` to Replica 1, 2, and 3.
	- Replica 1 says "OK"
	- Replica 2 says "OK"
	- Replica 3 is dead (Time out)
- The Client received 2 "OKs". It considers the write *Successful*

```
       [ Client ]
      /    |     \
     /     |      \  (Write "A")
    v      v       x
[Rep 1]  [Rep 2]  [Rep 3]
  OK       OK      (Dead)
   \      /
    \    /
   (Client sees 2 OKs -> "Success!")
```

#### How does a dead node catch up?

When Replica 3 comes back online, it has old data. *It missed the write*. We fix this in two ways:
1. **Read Repair (Lazy)**:
	- A client reads data from *all nodes*
	- It sees Rep 1 has Version 2 and Rep 3 has Version 1
	- The client realizes Rep 3 is stale, *(1) sends the new data to Rep 3*, and *(2) returns the result to the user*

```
STEP 1: PARALLEL READS
Client asks replicas for data. One node is stale (v1).

     [Node A: v2 (Fresh)]      [Node B: v1 (Stale)]
            ^                        ^
            | (Read ok)              | (Read ok)
            |                        |
+-----------+------------------------+-----------+
|                    CLIENT                      |
+------------------------------------------------+


STEP 2: DETECT & REPAIR
Client sees v2 is newer than v1. It serves v2 to the
user and silently updates Node B.

     [Node A: v2]              [Node B: v2 (NOW FIXED)]
                                     ^
                                     | (Repair Write v2)
                                     |
+-----------+------------------------+-----------+
|                    CLIENT                      |
| (Takes v2, returns to user app)                |
+------------------------------------------------+
```

2. **Anti-Entropy Process (Active)**:
	- A ***background process*** constantly scans all replicas looking for differences and copies missing data
```
STEP 1: BACKGROUND GOSSIP (Merkle Tree Exchange)
Nodes compare "Summaries" (Hashes) of their data ranges
to find differences without sending all the actual data.

     [Node A: Full Data]        [Node B: Missing Data]
            |                            |
            | 1. Compare Hashes          |
            | <------------------------> |
            | "Range 0-50: MATCH"        |
            | "Range 51-100: MISMATCH!"  |
            |                            |


STEP 2: SYNC DIFFERENCE
Node A sends only the specific missing pieces found
in the mismatched range.

     [Node A]                   [Node B: UPDATING]
            |                            ^
            | 2. Send missing keys       |
            | -------------------------> |
            |                            |
```

#### Quorums

***How do we ensure we don't read old data***? We use **math**.

- `N`: Total number of replicas (e.g., 3)
- `W`: Write Quorum (How many nodes must confirm a write for it to be valid)
- `R`: Read Quorum (How many nodes we must query when reading)

Quorum formula: **`W + R > N`**

Logic: ***If your Write group and your Read group overlap by at least one node, you are guaranteed to see the latest data***

Example:
- `N=3, W=2, R=2`
- `W + R = 2 + 2 = 4`
- W + R > N i.e `4 > 3` (**Safe**). We write to 2 nodes. We read from 2 nodes. At least one of the nodes we read from must have been in the group we wrote to
```
Nodes: [ 1 ] [ 2 ] [ 3 ]

Write to (W=2):  [ 1 ] [ 2 ]
Read from (R=2):       [ 2 ] [ 3 ]

Intersection: Node [ 2 ] has the new data!
```

#### Limitations of Quorum Consistency

Even if `W + R > N`, **edge cases can break strict consistency**

1. **Sloppy Quorums**: You might be writing to the "wrong" nodes, so the overlap doesn't exist
	- What if the network cuts the client off from the correct nodes? Imagine you need to write to Nodes A, B, and C. But you can't reach A or B
	- **Strict Quorum**: The database says "Error: Cannot reach Quorum." (System stops working)
	- **Sloppy Quorum (High Availability)**: The database says: "I can't reach A or B. *I will write the data to Node D instead, just for safekeeping*."
		- Node D is not the "home" of this data, but it holds it temporarily
		- This ***keeps the write throughput high***
		- **Hinted Handoff**: Once Node A and B come back online, Node D hands the data back to them and deletes its local copy
		- *Analogy*: You want to leave a key for your friend at their house (Home Node). They aren't home. You leave the key with their Neighbor (Sloppy Quorum). When your friend returns, the neighbor walks over and gives them the key (Hinted Handoff)

```
STEP 1: SLOPPY QUORUM (Substitution)
Goal: Write to "Home" Nodes A, B, C.
Problem: Node C is DOWN.
Solution: Write to neighbor D instead (temporarily).

      [Node A]     [Node B]     [Node C (DOWN X)]    [Node D]
         ^            ^                                 ^
         |            |                                 |
         | (Write)    | (Write)                         | (Temp Write)
         |            |                                 | "Hold this for C"
+--------+------------+---------------------------------+--------+
|                            CLIENT                              |
+----------------------------------------------------------------+


STEP 2: HINTED HANDOFF (Restoration)
Node C comes back online. Node D automatically forwards the
data back to C and then deletes its temporary copy.

                   [Node C (ONLINE!)] <-------------- [Node D]
                           ^                           |
                           |   "Here is the data       |
                           |    I held for you"        |
                           |                           |
                  (Data is restored locally)
```
Salient points:
- **Why use this?** You **prioritize Write Availability** above all else. You want your system to accept writes even if half the cluster is on fire (e.g., Amazon Shopping Cart)
- **The Risk**: It **increases the chance of Stale Reads**. If a reader asks Node C for data before Node D has finished the handoff, Node C will say "I don't have it"
- **Term to drop**: "This moves us from a CP system (Consistent Partition-tolerant) to an **AP system** (Available Partition-tolerant) temporarily"

2. **Concurrent Writes**: If two people *write at the exact same time*, it's *unclear* which one happened first (Clock skew)
3. **Incomplete Writes**: A write succeeds on Node 1 but fails on Node 2 (network cut). The client is told "Write Failed." However, a subsequent read might hit Node 1 and see the value that was supposed to have failed

#### Detecting Concurrent Writes

In *Leaderless systems*, multiple clients can write to the same key simultaneously. This creates conflicts

**The Problem**: Last Write Wins (**LWW**)
- If Client A writes "Blue" and Client B writes "Red" at the same time:
	- We can look at the clock. "Red" came 1 millisecond later
	- We overwrite "Blue" with "Red."
	- Result: "Blue" is lost forever. This is acceptable for caching, but bad for banking

**The Solution**: **Version Vectors / Vector clocks** (The "Happens-Before" Relationship)
- To avoid losing data, we don't overwrite. We treat the data as *siblings*
- Scenario: The Shopping Cart
	- Cart is empty. (`Version: 0`)
	- Client A adds "Milk". (`Version: 1`) -> `[Milk]`
	- Client B adds "Eggs" concurrently. They didn't know about the Milk
	- The system detects a fork. ***It saves both***
	- **Merge**: Next time the client reads the cart, they get both items. ***The application code must merge them (Union: Milk + Eggs)***
	- The client writes back the merged result with a *new Version ID*
	- **Version vectors**: Don't overwrite concurrent writes; keep both versions (siblings) and let the app merge them later
```
Key: Cart_123
Value:
  - sibling 1: [Milk] (Version 1a)
  - sibling 2: [Eggs] (Version 1b)
```
```
          [ Empty ]
          /         \
   (Client A)     (Client B)
  Adds Milk       Adds Eggs
      |               |
   [ Milk ]        [ Eggs ]  <-- CONFLICT (Siblings)
      \               /
       \             /
        \           /
      [ Milk, Eggs ]  <-- MERGED (New Version)
```

**Use cases for version vectors**: 
- **Amazon Shopping Carts**: Used to preserve items added to a cart from different devices at the same time. Instead of overwriting (losing items), the system keeps both versions so the application can merge them (Union operation).
- **Collaborative Editing (Offline Mode)**: Used when two users edit the same document field while offline. Vector clocks flag this as a conflict, forcing the UI to show both versions for the user to manually resolve, rather than silently losing one edit.
- **Distributed Counters (e.g., Likes/Views)**: Used when high-velocity updates hit different database nodes simultaneously. Vector clocks allow the system to recognize multiple independent increments and sum them up later for an accurate total.
- **Social Network Graphs**: Used when two users accept each other's friend requests simultaneously on different nodes. The resolution is to merge the edge creation so the friendship exists, rather than resetting the state.

## Partitioning

For ***very large datasets***, ***replicating data isn't enough***. If you have 100 TB of data, but your machine only has 10 TB of disk space, you cannot store the whole database on one machine!

You need to *split the data into chunks*. This is called **Partitioning** (also known as **Sharding**)
- The Goal: ***Scalability***
- The Idea: If you have 10 partitions and 10 nodes (computers), each node handles 1/10th of the read and write load. If you need more capacity, just add more nodes...

### Partitioning and replication

Partitioning is *usually combined with* Replication
- Partitioning splits data up (Node A has A-M, Node B has N-Z)
- Replication makes copies for safety

So, *a single node might act as the Leader for Partition 1*, but *act as a Follower for Partition 2*

```
Node 1          Node 2          Node 3
-------         -------         -------
P1 (Leader)     P2 (Leader)     P3 (Leader)
P2 (Follower)   P3 (Follower)   P1 (Follower)
```
- If Node 1 dies, we lose the Leader for P1, but Node 3 has a copy and can take over

### Partitioning of Key-Value Data

How do we decide which record goes to which node? We want to avoid **Skew**.
1. **Skew**: When one partition has far more data or load than others.
2. **Hot Spot**: A partition with disproportionately high load (e.g., if one node handles 90% of requests, the other 9 nodes sit idle). (Ex: Celebrity profile or celebrity tweets)

We have two main strategies to avoid this:
- Partitioning by Key Range
- Partitioning by Hash of Key

#### Partitioning by Key Range

Assign a **continuous range of keys to each partition**, like *volumes of a paper encyclopedia*.

Example:
- Partition 1: Authors A - G
- Partition 2: Authors H - O
- Partition 3: Authors P - Z

```
[ A, B, C ... G ]  --->  Node 1
[ H, I, J ... O ]  --->  Node 2
[ P, Q, R ... Z ]  --->  Node 3
```

✅ **Pros**:
- ***Efficient Range Scans***. If you want "All authors between B and E", you only query Node 1.

❌ **Cons**: 
- ***High risk of Hot Spots***. Ex: If the *timestamp* is the key, and you partition by range, all writes for today go to the same node (the "Today" partition), while yesterday's partitions sit idle
	- Since time only moves forward, 100% of your new incoming traffic (writes) will have a timestamp starting with today's date. Yesterday's and earlier partitions do not receive any writes ever moving forward (Bad distribution!)

#### Partitioning by Hash of Key

Instead of using the key directly (e.g., `"User123"`), you ***run the key through a hash function (like MD5)*** to get a "random-looking" number

Example:
- `"User123"` -> `Hash: 0x5A...` -> Goes to Partition 1
- `"User124"` -> `Hash: 0xB2...` -> Goes to Partition 2

```
Key: "Apple"  --hash--> 10  --> Node A (Range 0-30)
Key: "Banana" --hash--> 85  --> Node C (Range 60-90)
Key: "Cherry" --hash--> 42  --> Node B (Range 31-59)
```

✅ **Pros**: 
- ***Even distribution***. Data is scattered randomly, eliminating hot spots caused by sequential keys

❌ **Cons**:
- ***You lose Range Scans***. Since adjacent keys ("Apple", "Apricot") are hashed to totally different numbers, you can't ask for "Everything starting with A" efficiently. You have to query all nodes

#### Skewed Workloads & Relieving Hot Spots

Even with hashing, you can have a **"Celebrity Problem" (Extreme Skew)**

*Scenario*: Justin Bieber has 100 million followers on Twitter. When he tweets, millions of people fetch that one specific tweet (Key: `tweet_id`) 

Even with hashing, that one key lives on one node. That node melts!

```
=============================================================================
   THE CELEBRITY PROBLEM (EXTREME PARTITION SKEW)
=============================================================================

SCENARIO: Millions of users trying to "Like" Justin Bieber's new tweet
          at the exact same second.

INCOMING TRAFFIC FLOOD:
[Like JB_Tweet] [Like JB_Tweet] [Like JB_Tweet] [Like JB_Tweet] ... (x 10 Million)
       |
       | (Load Balancer hashes TweetID to find which node owns it)
       V

+-----------------------------+-----------------------------+-----------------------------+
|         NODE 1              |         NODE 2              |         NODE 3              |
| Ownership: Tweets A-M       | Ownership: Tweets N-Z       | Ownership: Tweets 0-9       |
|                             | (Holds JB's TweetID)        |                             |
+-----------------------------+-----------------------------+-----------------------------+
|                             |                             |                             |
| TRAFFIC:                    | TRAFFIC:                    | TRAFFIC:                    |
| -> [Like mom's tweet]       | ===>[Like JB_Tweet]         | -> [Like neighbor's tweet]  |
|                             | ===>[Like JB_Tweet]         |                             |
|                             | ===>[Like JB_Tweet]         |                             |
|                             | ===>[Like JB_Tweet]         |                             |
|                             | ===> ... (x10 Million)      |                             |
|                             |                             |                             |
V                             V                             V
STATUS: IDLE (2% CPU)         STATUS: MELTDOWN (100% CPU)   STATUS: IDLE (2% CPU)
(Wasted resources)            (System Bottleneck)           (Wasted resources)
```

**Solution**: Add a ***random number*** to the end/beginning of the key for **hot items**
- Key: `bieber_tweet` -> becomes `bieber_tweet_1`, `bieber_tweet_2` ... `bieber_tweet_100`
- Now the data is spread across 100 different nodes
- **Benefits**:
	- ***Massive Write Throughput*** (You are no longer limited by the CPU of a single node)
	- ***No Hotspots***: The "Celebrity" data looks just like normal data to the database (No meltdown)
- **Trade-off**: When you read, you have to read from all 100 keys and merge the data
	- ***The "Scatter-Gather" Read Penalty***: To find out how many likes Justin Bieber actually has, you can no longer just `GET JB_Tweet`. You must now *query all possible keys* (`JB_Tweet_1... JB_Tweet_100`) and sum them up in your application code
		- As a consequence, you have ***pagination nightmares***
	- ***Increased Latency***: Your read operation is now as slow as the slowest node in the cluster. If just one of those 100 partitions is lagging, the user sees a loading spinner

### Partitioning and Secondary Indexes

So far, we only discussed looking up data by Primary Key (`ID`). What if we want to search by color (e.g., `WHERE color = 'Red'`)? This requires a **Secondary Index**

There are two ways to partition these indexes:
1. Document-Partitioned Index (Local Index)
2. Term-Partitioned Index (Global Index)

#### Document-Partitioned Index

Each partition manages its own index for *only the data it holds*
- If Node 1 holds cars with IDs `0-100`, it builds an index for "Red" cars only within IDs `0-100`

```
     [ Client Request: "Find Red Cars" ]
             /        |        \
            /         |         \
      (Query)      (Query)      (Query)
        v             v            v
    [ Node 1 ]    [ Node 2 ]   [ Node 3 ]
    (Local Index) (Local Index) (Local Index)
        |             |            |
   "Found 2"      "Found 0"    "Found 5"
        \             |            /
         \            |           /
          v           v          v
       [ Result: 7 Red Cars Total ]
```
- **Pros**: ***Writes are fast*** (only update one node)
- **Cons**: ***Reads are slow*** (***Scatter/Gather***). You must query every partition to find all results

#### Term-Partitioned Index

We build *one giant global index* for "Color", but we *partition that index itself*
- Node A holds the index for colors `A-M` (Aqua to Magenta)
- Node B holds the index for colors `N-Z` (Navy to Yellow)

Pros and cons:
- **Pros**: ***Reads are fast***. If you want "Red", you know exactly which node holds the "R" index. You only ask that one node
- **Cons**: ***Writes are slow and complex***. If you save a "Red Car" on Node 1, you might have to update the index on Node B. This requires a **Distributed Transaction**

```
GLOBAL (TERM-PARTITIONED) INDEX SCHEMA
======================================

Read Flow (FAST): Client knows "Red" starts with 'R', goes straight to Node B.
Write Flow (SLOW): Writing "Red Car" on Data Node 1 requires updating Index Node B.

        READ PATH ("Find Red")
                 |
                 v
+----------------+----------------+  +----------------+----------------+
| INDEX NODE A (Terms A-M)        |  | INDEX NODE B (Terms N-Z)        |
| "Blue" -> [Node 2]              |  | "Red" -> [Node 1]               |
+---------------------------------+  +----------------^----------------+
                                                      |
          (Physical Network Separator)                | (CROSS-NODE
                                                      |  INDEX UPDATE!)
                                                      |
+---------------------------------+  +----------------+----------------+
| DATA NODE 1                     |  | DATA NODE 2                     |
| Document: {id:1, color:"Red"}   |  | Document: {id:2, color:"Blue"}  |
+----------------^----------------+  +---------------------------------+
                 |
                 |
        WRITE PATH ("Save Red Car")
```

### Rebalancing Partitions

As your database grows, you need to add more nodes. *Moving data from old nodes to new nodes is called **Rebalancing***

**What NOT to do**: `Hash % N` If you assign keys using `Hash(key) % NodeCount`:
- If NodeCount changes from 10 to 11, nearly every key will change its assigned node (e.g., `12 % 10 = 2`, but `12 % 11 = 1`).
- This causes ***massive, unnecessary*** data transfer

Better Strategies:
1. **Fixed Number of Partitions**
	- ***Create many more partitions than nodes*** (e.g., 1000 partitions for 10 nodes)
	- Each node owns ~100 partitions
	- *Adding a node*: The new node steals a few partitions from the existing nodes. ***The partitions themselves never change size or keys***; they just *move houses*

***Pro***: Rebalancing is fast and simple because you move entire existing buckets between nodes rather than recalculating individual keys

***Con***: You must guess the correct number of partitions upfront; if you guess too low, re-partitioning the entire live database later is incredibly difficult

```
Node 1: [P1] [P2] [P3]
Node 2: [P4] [P5] [P6]

*Add Node 3* -> Node 3 takes P3 from Node 1 and P6 from Node 2.

Node 1: [P1] [P2]
Node 2: [P4] [P5]
Node 3: [P3] [P6]
```

2. **Dynamic Partitioning**
- Used by ***HBase***/***RethinkDB***
- Start with **1 big partition**
- When it gets *bigger than* 10GB, **split it in half**
- If it gets small, merge it
- ***Benefit***: Adapts to data volume automatically


***Pro***: The system automatically adapts to data volume, scaling effortlessly from a small startup dataset to petabytes *without manual config*

**Con**: Sudden massive writes (bulk loading) can cause "split storms," where the database stalls because it is too busy splitting partitions to actually write data

```
STAGE 1: STARTING OUT (One big bucket)
+--------------------------------------------------+
| Partition A (Range: A-Z)                         |
| Status: 20% Full [##........]                    |
+--------------------------------------------------+

          (Time passes, data is added...)
                    |
                    V

STAGE 2: THRESHOLD REACHED
+--------------------------------------------------+
| Partition A (Range: A-Z)                         |
| Status: 100% Full [##########] -> TOO BIG! SPLIT!|
+--------------------------------------------------+
                    |
                    V

STAGE 3: THE DYNAMIC SPLIT
(Partition A splits at the median key, e.g., 'M')

+-----------------------+    +-----------------------+
| New Part A (Range A-M)|    | New Part B (Range N-Z)|
| Status: 50% [#####..] |    | Status: 50% [#####..] |
+-----------------------+    +-----------------------+
```

###  Automatic vs Manual Rebalancing Operations

Should the database automatically move data when a node is added?
1. **Fully Automatic**: *Convenient, but dangerous*. If the network is flaky, the system might think a node is dead, start moving terabytes of data, clog the network, and cause more failures (cascading failure).
2. **Manual (or Human-Approved)**: The system suggests a rebalance, but an administrator must click "Confirm". *This is usually safer*

### Request Routing

When a client wants to read `"Key: Foo"`, how does it know which IP address to connect to? This is the **Service Discovery** problem (This is a general distributed system problem and NOT limited to partitioned i.e sharded databases)

Three approaches:
1. **Round-Robin (Load Balancer) / Routing Tier**: Client talks to a Load Balancer. The LB knows the partition map and routes the request
2. **Partition-Aware Client**: The client library has the partition map loaded. It connects directly to the right node
3. **Request Forwarding**: Client connects to any random node. If that node doesn't have the data, it forwards the request to the correct node

Routing Tier / Load Balancer illustration:
```
 [ Client ]
    |
    | (1) Request
    v
 [ Routing Tier ] (Has the Map)
    |
    | (2) "Oh, User:123 is on Node B"
    +----------------------------------> [ Node B ]
                                            |
    <---------------------------------------+
    | (3) Returns Data
 [ Client ]
```
Request forwarding illustration:
```
 [ Client ]
     |
     | (1) "Get User:123"
     v
 [ Node A ] ------------------------> [ Node B ]
 (I don't have it,                      (I have Partition 7!)
  but I know Node B does)               (2) Returns Data
          <-----------------------
     |
     v (3) Returns Data
 [ Client ]
```

#### Role of Zookeeper

If partitions are constantly moving, how do the Routing Tier or the Client know the new addresses? If Node B dies and Partition 7 moves to Node C, how does everyone find out?

We need a highly reliable **"Source of Truth"** to keep track of the changes. This is where **ZooKeeper** comes in.

What is ZooKeeper?
- ZooKeeper is *a separate software service used for Distributed Coordination*. In the context of partitioning, it acts like a **Cluster Manager** or a **Dynamic Directory**
- It maintains the authoritative mapping of: `Partition 7 -> 192.168.1.50 (Node B)`

How it works (The Workflow)
1. *Registration*: When a database node starts up, it registers itself with ZooKeeper. (`"Hi, I am Node B, my IP is 1.2.3.4"`).
2. *Tracking*: ZooKeeper keeps track of which partitions are on which node.
3. *Subscription*: The Routing Tier (or the Client) subscribes to ZooKeeper. They say, "Tell me if anything changes."
4. *Updates*: If a rebalance happens (Partition 7 moves from Node B to Node C):
5. The database updates ZooKeeper
6. ZooKeeper pushes a notification to the Routing Tier
7. The Routing Tier updates its internal map

Why use ZooKeeper?
- Implementing ***a distributed system where everyone agrees on "who holds what data" is incredibly difficult*** (*Consensus Problem)*.
- If you try to write this logic yourself, you will likely introduce bugs where clients send data to the wrong node.
- ***ZooKeeper handles the hard part (Consensus/Coordination)*** so the database developers don't have to build it from scratch.
- Databases that use ZooKeeper (or similar) for this: **HBase, SolrCloud, Kafka, Helix**
- Databases that DON'T: Cassandra and Riak use a "Gossip Protocol" (nodes whisper to each other to figure out the map) instead of a central authority like ZooKeeper

```
        [ ZooKeeper ]
        (The Manager)
             |
             | (1) "Update: Partition 1 is on Node A"
             |     (Metadata only, no heavy data)
             v
    [ Routing Tier ] <--------------------- [ Client ]
    (The Receptionist)                       (2) "Read Key X"
    (Local Map: P1 = A)
             |
             | (3) Forward Request
             |
             v
       [ Node A ]              [ Node B ]
     (Has Partition 1)       (Has Partition 2)
```

### Parallel Query Execution

For **OLAP** (Analytics) / Data Warehouse workloads, ***queries often touch all partitions*** ("Count all users"). The ***Massively Parallel Processing (MPP)*** query optimizer breaks the complex query into stages

## Transactions 

In a data system, many bad things can happen
- The database software crashes
- The power goes out
- The disk gets full halfway through a write
- The network cuts off, etc....

*The Solution*: The **Transaction**. A transaction is *a way to group several reads and writes into one logical unit* 
- The database gives you a guarantee: either all of them happen (Commit), or none of them happen (Abort/Rollback)
- *Benefit*: You ***don't have to worry about partial failure*** (e.g., money left one account but didn't arrive in the other). If it fails, you can safely retry

```
      WITHOUT TRANSACTION            WITH TRANSACTION
      -------------------            ----------------
1. Deduct $100 from Alice (OK)    [ START TRANSACTION ]
2. System Crashes! (Power Fail)   1. Deduct $100 from Alice (OK)
3. Add $100 to Bob (Never ran)    2. System Crashes! (Power Fail)
                                  [ ROLLBACK HAPPENS ON RESTART ]
   Result: Money vanished.           Result: Alice still has her $100.
```

### ACID standards

To ensure transactions are ***safe***, databases follow the **ACID** standards

1. **Atomicity (The Abort Button)**: It is NOT about speed or multi-threading; it guarantees *abortability*—an "all-or-nothing" safety net. If a transaction fails midway, the database rolls back all changes to prevent partial corruption, ensuring the system remains clean as if the attempt never happened..
```
Step 1: Write A  --> Done.
Step 2: Write B  --> Error! (Disk Full)
Step 3: Write C  --> Skipped.

Action: AUTO-UNDO Step 1.
Final State: Database looks like nothing ever happened.
```

2. **Consistency (The Valid State)**: It ensures the database *transitions from one "valid" state to another, preserving all invariants* (e.g., `"Credits = Debits"`). However, DDIA notes this is actually *an application property*, not a database one; the database guarantees Atomicity and Isolation, but if you write logic that violates your own business rules, the database cannot prevent the inconsistency

```
Rule: Account Balance cannot be negative.

Current: $50
Transaction: "Withdraw $100"

Outcome:
   Application Code -> Checks rule -> ABORTS transaction.
   (Consistency is preserved by the code logic).
```

3. **Isolation (I am Alone)**: It *handles concurrency to prevent race conditions* (like overwriting data). It guarantees that even if transactions execute simultaneously, the *final result is identical to if they had run sequentially (Serially)*. This ensures no transaction ever sees the invalid, half-written state of another
```
SCENARIO: T1 is transferring $10 from Account A to B.
Assumption: Initial A balance = Initial B balance = $100
T2 tries to read the total bank balance mid-transfer.

      Transaction 1 (T1) Running
+-----------------------------------+
| 1. Take $10 from A (A now = $90)  | <- Temporary
| 2. Give $10 to B   (B now = $110) |    Messy
| 3. COMMIT                         |    State
+-----------------------------------+
            ^
            | <=== ISOLATION BARRIER ===> |
            | (Blocks access to temp state)
            |
      Transaction 2 (T2) Reads Total
      Result: T2 either waits for T1 to finish, or sees
      the old snapshot. It NEVER sees the messy state
      where $10 briefly vanished.
```

4. **Durability (Written in Stone)**: It guarantees that *once a transaction commits*, the data is *permanently saved to non-volatile storage* (like a **Write-Ahead Log**) and will survive even an immediate power failure, accepting the necessary performance penalty of slow disk writes to ensure this safety
```
1. Client says "Commit"
2. Database writes to Disk Log (fsync)
3. Disk confirms "Saved"
4. Database tells Client "Success"

*Power Fails Here* -> Data is safe on disk.
```

### Single-object and Multi-object operations

Many operations in an application require *changing several different pieces of data* (objects, rows, or documents) at the same time. To keep the database clean, these changes must happen in an "all-or-nothing" (**Atomic**) way and in **isolation** from other users

If ***we don't have multi-object transactions***, users might see *weird, half-finished data*

Example: The Email Alert (The Sync Problem)
- Imagine a system where you manage an unread email count
- You insert a new email into the Emails table
- You update the UnreadCount table (increment by 1)
- If Step 1 succeeds but Step 2 fails, the user sees the email but the badge count is wrong
```
Table: EMAILS               Table: UNREAD_COUNT
+------------------+        +------------------+
| ID | Subject     |        | User | Count     |
+------------------+        +------------------+
| 1  | "Hello!"    |        | Alice| 5         |
| 2  | "Meeting"   | <-----+|      |           |
+------------------+       || +----------------+
      ^                    ||
      | (Step 1: OK)       |X (Step 2: Failed!)
      |
   [ Insert Email ]     [ Update Count ]
```

**Dirty reads**

Violating Isolation (Dirty Reads): If you don't group these updates into a transaction, another user might read the database in the middle of your updates
- Scenario: You act as the database
- Action: You update the email list, then pause for 1 second, then update the counter
- The User: Reads the database during that 1-second pause. They see the new email, but the old counter. This is *inconsistent*

```
Time   Writer (Transaction)           Reader (User)
 |          |                               |
 v     1. Write Email (Done)                |
       [ PAUSE / LAG ] -----------------> 2. Read Email List (Sees New Email)
                                          3. Read Counter (Sees OLD Count!)
       4. Update Counter (Done)             |
                                          (Confused: "I have mail but counter says 0?")
```

**Solution**: A **Multi-Object Transaction** ensures the Reader sees either both old versions or both new versions, never a mix

#### Single-Object Writes

Before worrying about multiple tables, we must ensure one single row is safe. If you are writing a massive 20KB JSON document to a database, and the power fails after 10KB, you are left with a corrupted, unreadable JSON file

Storage engines provide **Single-Object Atomicity**
- The Guarantee: Even if the crash happens mid-write, the database uses *checksums* or *logs* to discard the partial garbage on reboot.
- Compare-and-Set (CAS): A lightweight tool for *concurrency*. "Only update this value if it hasn't changed since I last looked."

```
Object: { "name": "Alice", "bio": "A very long string..." }

Write Operation:
[ "Ali... (Power Cut) ]

Database on Restart:
1. Detects partial write (Checksum mismatch).
2. Rolls back to previous version.
3. Object is safe.
```

#### Do we really need Multi-Object Transactions? 

While some NoSQL stores dropped them for speed (claiming smart modeling is enough), DDIA argues they remain essential to prevent data corruption in two key areas:
1. **Secondary Indexes**: Updating a field (e.g., City) requires atomically updating the record and the search index. Without transactions, search results (the index) become permanently out of sync with the actual data
2. **Denormalization**: If a field (like a Username) is copied across 1,000 comments for performance, a name change requires a transaction to update all 1,000 copies simultaneously, or readers will see inconsistent data

### Weak Isolation Levels

Most databases do not run in strict "Serializable" mode (processing one transaction at a time) because it is too slow. Instead, they use ***Weak Isolation Levels, which allow some concurrency bugs in exchange for speed***

#### Read Committed

*Read Committed* is the default setting in many databases (PostgreSQL, Oracle, SQL Server). It makes two guarantees:
1. **No Dirty Reads**: You will never see data that has been written but not yet committed
	- Scenario: Alice updates a value but hasn't hit "Save" (Commit) yet. Bob reads that value
	- Without Protection: Bob sees the temporary value. If Alice aborts, Bob saw data that never existed
	- With Read Committed: The database shows Bob the old value until Alice officially commits
```
Value = 10

Alice (Writer)               Bob (Reader)
      |                            |
  Set Value=20                     |
  (Not Committed)                  |
      |                            |
      | -------------------------> Read Value? -> Returns 10 (Old)
      |                            |
    COMMIT                         |
      | -------------------------> Read Value? -> Returns 20 (New)
```
2. **No Dirty Writes**: You will never overwrite data that someone else is currently busy writing
	- Mechanism: ***Row-level locking***. If Alice wants to update Object A, she locks it. If Bob tries to update Object A, he must wait until Alice finishes
```
  [Alice Transaction]
         |
         | 1. Grabs Lock & Writes
         v
+====================+
|  [ROW A LOCKED]    | <--- Mechanism
| (Alice is busy...) |
+====================+
         ^
         | X (BLOCKED! Wait for lock)
         |
   [Bob Transaction]
   2. Tries to write same Row A
```

#### Snapshot Isolation or Repeatable Read

*Read Committed* has a major flaw: **Read Skew** (also called **Non-Repeatable Read**)

> A **Non-Repeatable Read** occurs when a **single transaction reads the same record twice** but gets different values (e.g., $500 then $600) because another transaction updated and committed that data in between the two reads

The Scenario (The Bank Audit):
- You have $500 in Account A and $500 in Account B. (Total: $1000)
- ***You run a report that reads Account A first ($500)***
- Suddenly, a transfer of $100 moves from B to A. (A=$600, B=$400)
- You read Account B ($400)
- ***Your Report: A($500) + B($400) = $900. Money vanished!***

```
 [READER: REPORT]              [WRITER: TRANSFER]
            |                               |
1. Reads A: $500                            |
   (Sum so far: $500)                       |
            |                               |
            |                       2. Moves $100 (B -> A)
            |                       (A=$600, B=$400)
            |                       3. COMMITS
            |                               |
4. Reads B: $400                            |
   (Sum: $500 + $400)                       |
            |                               |
            v                               v
   RESULT: $900 (ERROR!)           ACTUAL TOTAL: $1000
   ($100 vanished)
```

**The Solution**: **Snapshot Isolation**. The database takes a virtual "photo" of the database at the start of your transaction. Even if data changes later, ***you continue to see the data exactly as it was when you started***
- Implementation: **MVCC (Multi-Version Concurrency Control)**. The database keeps multiple versions of the same object in memory (Version 1, Version 2, Version 3)

```
Object: Account A (Ver 1: $500)

Time   Reader (You)                Writer (Transfer)
 |     [Start Snapshot 10]                 |
 v     Read A -> Sees Ver 1 ($500)         |
       (Snapshot locked to Ver 1)          |
                   |                       |
                   |                   Update A to $600 (Ver 2)
                   |                   COMMIT (Ver 2 is live)
                   |
       Read A again -> Still sees Ver 1 ($500)
       (Ignores Ver 2 because it's new)
```

#### Preventing Lost Updates

This problem happens when two people do a ***Read-Modify-Write cycle at the same time***

The Scenario: Two users try to increment a "Like" counter (currently 42)
- Alice reads 42
- Bob reads 42
- Alice writes 43
- Bob writes 43. Result: 43. (Should be 44). Bob's write "clobbered" Alice's write

```
Counter = 42

   Alice                     Bob
     |                        |
  Read (42)                Read (42)
  Calc (42+1)              Calc (42+1)
  Write (43) --\      /--  Write (43)
                \    /
                 \  /
             Value = 43
       (Alice's update is LOST)
```


**Solutions**:
1. **Atomic Write**: Use database instructions that do it in one step `UPDATE counters SET value = value + 1 WHERE key = 'likes'`
2. **Explicit Locking**: The application explicitly locks the row (Bob must wait)
3. **Compare-and-Set (CAS)**: `UPDATE counters SET value = 43 WHERE key = 'likes' AND value = 42` (If the value changed to 43 while Bob was thinking, this update fails)

#### Write Skew and Phantoms

Write Skew is a *subtle problem* that ***Snapshot Isolation does not fix***. It happens when ***two transactions read different objects*** but ***violate a rule that applies to the relationship between them***

The Scenario (Doctors on Call):
- Rule: At least one doctor must be on call
- Current: Alice and Bob are both on call
- Action: Both feel sick and try to leave at the same time
	- (Concurrently) Alice checks: "Is anyone else on call? Yes, Bob." -> Alice leaves
	- (Concurrently) Bob checks: "Is anyone else on call? Yes, Alice." -> Bob leaves.
- Result: Zero doctors.

This is **Write Skew**. They didn't write to the same object (Alice updated Alice, Bob updated Bob), so "Lost Update" protection didn't catch it

**Phantoms**: The *root cause* here is a Phantom. **A phantom is when a write in one transaction changes the result of a search query in another**
- Bob checked `SELECT * FROM doctors WHERE on_call = true`
- Alice changed the result of that query after Bob checked it, but before Bob committed

```
DB State: {Alice: On, Bob: On}

      Alice Tx                    Bob Tx
         |                           |
   Check: Is Bob On? (Yes)           |
         |                    Check: Is Alice On? (Yes)
   Update: Alice = Off               |
         |                    Update: Bob = Off
       Commit                      Commit

Final State: {Alice: Off, Bob: Off} -> DISASTER.
```

**The Fix**: To prevent Write Skew, you usually need **Serializable isolation (strictest)** or you must **explicitly lock the rows you are checking (FOR UPDATE)**

### Serializability

Serializability is the **strongest isolation level**. ***It guarantees that even if transactions run in parallel, the final result is exactly the same as if they had run one by one, strictly in order***

Discussed below are the main ways to achieve this

#### Actual Serial Execution

The simplest way to avoid concurrency bugs is to **remove concurrency entirely**. Instead of multi-threading, we ***execute transactions sequentially on a single thread***
- Why didn't we do this before? RAM used to be small. We needed to read from disk (slow), so we let other threads work while waiting for the disk.
- Why it works now: RAM is cheap. If the whole dataset fits in memory, reading is instant. We don't need to wait.
- Examples: Redis, VoltDB

***Encapsulating Transactions (Stored Procedures)***

In this model, you cannot have a transaction that pauses to ask the user "What next?". That would stop the entire database. You must bundle the entire logic (Read A, Calculate, Write B) into a Stored Procedure and submit it at once

- Pros: No locks, no coordination overhead. Extremely fast.
- Cons: Limited by a single CPU core. If one transaction is slow, everything halts

```
Interactive (Traditional)        Serial (Stored Proc)
-------------------------        --------------------
Client: "Get Balance"            Client sends code:
   | (Network delay)                {
DB: "$100"                            val = get_bal()
   | (User thinks...)                 val -= 10
Client: "Set to $90"                  set_bal(val)
   | (Network delay)                }
DB: "Done"                       
                                 DB executes continuously (No waiting)
```

#### Two-Phase Locking (2PL)

This is the traditional **"Pessimistic" approach** used by strong SQL databases (like SQL Server or MySQL in Serializable mode).

The Rule:
1. Readers block Writers. (If I am reading it, you can't change it).
2. Writers block Readers. (If I am changing it, you can't read it).

**Note**: ***In Snapshot Isolation, Readers never blocked Writers. In 2PL, they do***

***Shared vs. Exclusive Locks***
1. Shared Lock (Reader): Many people can hold this. "We are all reading."
```
 [Tx1 Read] --(S)--> [ DATA ] <--(S)-- [Tx2 Read]
(RESULT: OK (Sharing allowed))
```
2. Exclusive Lock (Writer): Only one person can hold this. "I am changing this". *Everyone else is blocked*
```
 [Tx1 Write] ==(X)==> [ DATA ]
                        X
 [Tx2 Read/Write] ----->(BLOCKED!)
RESULT: WAITING (Must wait for Tx1 to release X)

Note: Tx stands for 'transaction'
``` 
3. Upgrade: If you are reading (Shared) and want to write, you must upgrade to Exclusive.

It is called **Two-Phase Locking** because a ***transaction*** is forced to operate in two distinct stages:
- ***Expanding Phase***: It acquires locks but cannot release any.
- ***Shrinking Phase***: *Once it releases a single lock, it cannot acquire any new ones* (Why? It is the rule -- once you release the first lock, you can only release more locks but never acquire any until the transaction completes)

This strict rule (you can't unlock and then relock) ***ensures valid isolation by creating a "locked point" where the transaction holds everything it needs at once***

##### Predicate Locks & Index-Range Locks

How do we solve the Phantom problem (writing a new row that affects someone else's search)?
- Predicate Lock: "Lock all rows where `room_type = 'Suite'`" (Ideally correct, but very slow to implement).
- Index-Range Lock (Next-Key Locking): A simplified approximation. We attach a lock to the Index entry

If you search for "Suites", the DB locks the index entry for "Suite". If someone tries to insert a new Suite, they must update the Index. They hit the lock and are blocked.

Cons: **2PL is slow**. ***Deadlocks (A waits for B, B waits for A) are frequent***

#### Serializable Snapshot Isolation or SSI

This is the modern, "Optimistic" approach (used by PostgreSQL). It ***combines the non-blocking speed of Snapshot Isolation with the safety of Serializability***.

The Philosophy:
- 2PL says: "Wait! Don't do that, it might be dangerous." (Pessimistic).
- SSI says: "Go ahead! Do whatever you want. I'll check at the end if you broke anything." (Optimistic)

How it works
- Transactions run freely using Snapshot Isolation. When a transaction wants to Commit, the database checks two things:
	1. Did I read a stale version? (Did someone update the data while I was looking at the old snapshot?)
	2. Did I rely on a premise that is no longer true? (Phantom detection)

If a conflict is detected, the transaction is ***Aborted*** and must ***retry***

```
     Tx A (Reader)               Tx B (Writer)
           |                            |
    Read Key X (Ver 1)                  |
           |                      Write Key X (Ver 2)
    Thinking...                   COMMIT (Success)
           |                            |
    Write Key Y based on X              |
    Try to COMMIT                       |
           |                            |
    DB Check: "Hold on..."              |
    "The X you read (Ver 1)             |
     is now Ver 2!"                     |
           |                            |
    ABORT! (Retry needed)               |
```
- *Pros*: ***Reads don't block writes***. Great when conflicts are rare
- *Cons*: If contention is high (many people updating the same object), ***many transactions will abort/retry, wasting CPU***

### Understanding phantom reads

Phantom Reads happen when *a transaction **runs the same search query** (e.g., `WHERE age > 18`) **twice**, but the ***set of matching rows changes*** because another transaction added or removed data in between*

**Scenario 1: The New Arrival (Insert)**: T1 searches a range. T2 inserts a new row that fits that range. T1 runs the search again and sees a "ghost" row that wasn't there before
```
       [Tx1: READER]                   [Tx2: WRITER]
            |                               |
1. SELECT * WHERE age > 10                  |
   Result: [Bob]                            |
            |                               |
            |                 2. INSERT "Alice" (age 12)
            |                 3. COMMIT
            |                               |
2. SELECT * WHERE age > 10                  |
   Result: [Bob, Alice!]                    |
            ^                               |
        (Phantom appears!)
```

**Scenario 2: The Disappearance (Delete)**: T1 sees a row in the first search. T2 deletes it. T1 runs the search again, and the row has vanished
```
       [Tx1: READER]                   [Tx2: WRITER]
            |                               |
3. SELECT * WHERE active=1                  |
   Result: [A, B]                           |
            |                               |
            |                 2. DELETE row B
            |                 3. COMMIT
            |                               |
4. SELECT * WHERE active=1                  |
   Result: [A]                              |
            ^                               |
      (Phantom vanished!)
```

**Scenario 3: The Shape Shifter (Update)**: T2 updates an existing row so that it now matches T1's search criteria (or no longer matches). It enters or leaves the "range" mid-transaction
```
       [Tx1: READER]                   [Tx2: WRITER]
            |                               |
5. SELECT * WHERE score > 90                |
   Result: (Empty)                          |
            |                               |
            |                 2. UPDATE Charlie
            |                    SET score = 95
            |                 3. COMMIT
            |                               |
6. SELECT * WHERE score > 90                |
   Result: [Charlie!]                       |
            ^                               |
   (Phantom sneaked in via update)
```

#### Fixes for phantom reads

1. **Serializable Isolation**: Switch the database to `SERIALIZABLE` level, forcing transactions to execute sequentially and effectively preventing any concurrency anomalies
	- The transaction doesn't just lock the rows it found; it locks the search criteria itself!
```
QUERY: "SELECT * FROM Cars WHERE Color = 'Red'"

      [Tx1: READER]                     [Tx2: WRITER]
             |                                 |
             v                                 |
   +=============================+             |
   |  PREDICATE LOCK             |             |
   |  Scope: Color='Red'         |             |
   |                             |             v
   |  [Row 1: Red Truck]         |      [TRY INSERT 'Red Car']
   |  [Row 2: Red Bike ]         |             |
   |  [  EMPTY SPACE   ] <-------+----(BLOCKED! STOP!)
   +=============================+             |
             |                                 |
             |                                 |
        Tx1 COMMITS                            |
             |                                 |
             +-----------------------> Lock Released.
                                       Tx2 can now Insert.
```
2. **Index-Range Locking (Next-Key Locking)**: The database automatically locks the "gaps" in the index between existing rows, physically blocking other transactions from inserting new data into that range
```
QUERY: SELECT * FROM Users WHERE ID BETWEEN 10 AND 20
EXISTING DATA: Only IDs 10 and 20 exist.

      [Tx1: READS RANGE]                [Tx2: TRIES INSERT 15]
              |                                   |
              v                                   v
    +----+----------+----+               +--------+--------+
    | 10 | GAP LOCK | 20 |               | INSERT ID: 15   |
    +----+----+-----+----+               +--------+--------+
              |                                   |
              |                                   |
              +<----------(BLOCKED!)--------------+
              |
      "The empty space between 10 and 20
       is locked. Come back later."
```
3. **Materializing Conflicts**: Manually create a table of placeholder rows (e.g., `TimeSlots`) and lock the relevant row to artificiality create a conflict when no actual data exists to lock
```
SCENARIO: Two users try to book "Room A" at 12:00.
(The 'Bookings' table is empty, so there are no rows to lock!)

      [PRE-REQ: MATERIALIZED TABLE]
      Table: TimeSlots (Rows: 11:00, 12:00, 13:00 created in advance)

      [Tx1: BOOKING]                  [Tx2: BOOKING]
            |                               |
            v                               v
  1. LOCK row "12:00"              2. TRY LOCK row "12:00"
     (SELECT FOR UPDATE)                    |
            |                               |
    +-------+-------+                       |
    | Slot: 12:00   |<-------(BLOCKED!)-----+
    | [LOCKED Tx1]  |
    +---------------+
            |
  3. Safe to Insert into
     'Bookings' table
```

## Consistency and Consensus

### Linearizability and Ordering

In a distributed system, ***data is spread across many nodes*** (computers).
- The Ideal: The user should not know this. They should feel like they are **talking to one single super-computer**
- The Reality: The network is slow, clocks drift, and nodes die

**CONSISTENCY** is the ***attempt to maintain this illusion***

#### **Linearizability (Strong Consistency)**
	- This is the ***strongest guarantee*** a system can provide
	- *Definition*: The system behaves as if there is **only one copy of the data**, and **every operation is instantaneous**
	- Once a write completes, all subsequent reads (by anyone, anywhere) must see that new value
	- You are not allowed to serve "stale" (old) data, ever!

Scenario: The World Cup Final
- Imagine a score updates from "0-0" to "1-0"
- Alice refreshes her phone (connected to Replica A). She sees "1-0". She yells "Goal!"
- Bob hears Alice, refreshes his phone (connected to Replica B)
- **Without Linearizability**: Replica B might be lagging. Bob sees "0-0". He thinks Alice is hallucinating. This violates the "Single Copy" illusion because Bob effectively traveled backward in time
- **With Linearizability**: If Alice saw "1-0", Replica B must also show "1-0" (or wait until it can)

Linearizability violation:
```
Value: 0

Proponent (Referee)  [ Write 1 ]---------------------> (Success)
                        |
Alice (Reader)          +-------> [ Read: 1 ]
                                    (Time passes...)
Bob (Reader)                           +----------> [ Read: 0 ]
                                                      ^
                                                      |
                                         VIOLATION! (Stale Read)
                                   Bob saw '0' AFTER Alice saw '1'.
```

The Cost of Linearizability (Why doesn't everyone use this?)
- ***Performance***: To guarantee no one sees old data, you usually have to talk to all replicas (or a majority quorum) for every read. This is **slow**
- ***Availability***: If the network between data centers is cut, you cannot process requests safely. **You must choose: CP or AP (CAP Theorem)**
	- **CP (Consistent):** "I can't reach the other nodes, so I will lock the database until I can." (System Down)
	- **AP (Available)**: "I can't reach the others, so I'll serve you old data." (Linearizability Broken)

#### Linearizability vs Serializability 

**Linearizability (Recency Guarantee)**: A property of a single object (like a register) that guarantees "freshness." Once a write successfully completes, every subsequent read (in wall-clock time) must return that new value. ***It makes a distributed system look like a single computer***
```
SCENARIO: Write completes at 10:00. Read starts at 10:01.

      [CLIENT A: WRITER]                [CLIENT B: READER]
             |                                  |
   1. WRITE(x=5) STARTS                         |
             |                                  |
             |                                  |
   2. WRITE FINISHES (Ack)                      |
      (Time: 10:00)                             |
             |                                  |
    -------------------------------------------------------
    The "New Reality" is set. Any read past this line MUST see 5.
    -------------------------------------------------------
             |                                  |
             |                         3. READ(x) STARTS
             |                            (Time: 10:01)
             |                                  |
             |                         4. RESULT: 5 (Required!)
                                          (If it sees 0, it's
                                           NOT Linearizable)
```

**Serializability (Isolation Guarantee)**: A property of transactions (groups of operations) that guarantees the final database state is valid. It ensures concurrent transactions produce the same result as if they had executed one after another (serially), but ***it does not guarantee they run in the exact real-time order they arrived***
```
SCENARIO: Two transactions overlap perfectly in time.
Target: The DB must pick a winner and play them sequentially.

      [Tx1: Transfer $10]           [Tx2: Calculate Sum]
             |                               |
      (Starts 10:00)                  (Starts 10:00)
             |                               |
             v                               v
    +=================+             +=================+
    |  Step A...      |             |  Step X...      |
    |  Step B...      |             |  Step Y...      |
    +=================+             +=================+
             |                               |
             |                               |
      [ THE DATABASE "MAGIC" (SERIALIZER) ]
      "I will force these to run as: Tx2 -> Tx1"
             |
             v
      RESULT STATE:
      1. Tx2 runs (Sees old balance)
      2. Tx1 runs (Updates balance)
      
      (This is VALID Serializability, even though
       Tx1 might have physically finished first!)
```

| Feature | Linearizability | Serializability |
| -- | -- | -- |
| Focus | Single Object (Read/Write) | Transactions (Groups of ops) |
| Constraint | Real-time (Wall clock) | Logical Order (Database correctness) |
| Goal | Freshness (Don't serve stale data) | Correctness (Don't corrupt data) | 

#### Ordering and Causality

If Linearizability is too expensive, can we accept something weaker? Often, we don't need *exact* real-time updates. We just need the Order to make sense

**Causality**: If Event A causes Event B, then everyone must see A before B. *Example*: You cannot see the "Reply" before you see the "Question."

**The Problem with Physical Time (Wall Clocks)**
- You might think: "Just attach a timestamp to every message!"
- Problem: *Computer clocks are unreliable*. One server might think it's 12:00:00 while another thinks it's 12:00:05. *You cannot trust timestamps for ordering*

**The Solution**: **Lamport Timestamps** (Logical Clocks)
- Instead of using the time of day, we use a simple Counter. Every node keeps a ***counter*** (1, 2, 3...)
- The Algorithm:
	- Every time a node does something, it increments its *counter*
	- When sending a message, attach the counter value
	- Crucial Rule: When you receive a message with a counter higher than yours, set your counter to `Message_Counter + 1`

```
(Node A)              (Node B)
Time Counter: 1      Time Counter: 10
   |                     |
   | (Write A)           |
   | Time: 2             |
   +-------------------->|
       (Msg: "Hello",    |
        Timestamp: 2)    |
                         | (Receives 2. My clock is 10.)
                         | (Rule: Max(2, 10) + 1 = 11)
                         |
                         +---> (Write B)
                               Timestamp: 12
```
* *Result*: Even though Node A was "behind," Node B's new event (12) is correctly ordered after Node A's event (2)

#### Total Order Broadcast

Lamport timestamps tell us the order, but they have a *flaw*: **Uniqueness**
- Two nodes can both generate "Timestamp 5" at the same time. You need a tie-breaker (like Node ID), but you can't know for sure if "Timestamp 5" is the final winner until you check every other node
- To solve "First come, first served" (e.g., claiming a username), we need **Total Order Broadcast**
- Think of this as a "Log" or a "Queue."
- Delivery: If Message A is delivered before Message B on one node, it must be delivered in that order on all nodes
- Reliability: If a message is delivered to one node, it must be delivered to all.

Analogy: A whatsapp group. Everyone sees the messages in the exact same order. You can't see "Reply" before "Message"

### Distributed Consensus

Consensus Goal: ***Get several nodes to agree on one single value*** (e.g., "Who is the leader?", "Did this transaction commit?")

#### Atomic Commit (Two-Phase Commit / 2PC)

Used when you need to execute a transaction across multiple nodes (Shards).

Example: Transfer $100 from Bank A (Partition 1) to Bank B (Partition 2). Either both must happen, or neither.

The Process (The Marriage Ceremony Analogy): There is a ***Coordinator*** (Priest) and ***Participants*** (Couple).

Phase 1 (Prepare):
- Coordinator asks: "Can you commit?"
- Participants check disk space, constraints, locks.
- The Point of No Return: If a participant says "YES", they promise to commit. They cannot back out later.

Phase 2 (Commit):
- If everyone said `YES` -> Coordinator sends `"COMMIT"` (to every node)
- If anyone said `NO` -> Coordinator sends `"ABORT"` (to every node)

Example: The commit path
```
     [COORDINATOR]                  [NODE A]       [NODE B]
            |                           |              |
    PHASE 1:|--(Prepare to Commit?)---->|              |
    (Vote)  |--(Prepare to Commit?)------------------->|
            |                           |              |
            |<---------(YES)------------|              |
            |<---------(YES)---------------------------|
            |                           |              |
    PHASE 2:|--(COMMIT!)--------------->|              |
    (Exec)  |--(COMMIT!)------------------------------>|
            |                           |              |
            |                    (Writes Saved)  (Writes Saved)
```

Example: The abort path
```
     [COORDINATOR]                  [NODE A]       [NODE B]
            |                           |              |
    PHASE 1:|--(Prepare to Commit?)---->|              |
    (Vote)  |--(Prepare to Commit?)------------------->|
            |                           |              |
            |<---------(YES)------------|              |
            |<---------(NO!)---------------------------|
            |                           |              |
    PHASE 2:|--(ABORT!)---------------->|              |
    (Exec)  |--(ABORT!)------------------------------->|
            |                           |              |
            |                     (Rollback)     (Rollback)
            |                    "Nothing happened"
```

**Flaw**: A 2PC Failure in case the Coordinator crashes after sending "Commit" to A but before sending it to B.... Node A has committed. Node B is stuck. It promised to listen to the Coordinator, but the Coordinator is dead. It holds the lock forever
```
   [ Coordinator ]             [ Node A ]      [ Node B ]
          |                        |               |
   (1) "Prepare?" ---------------->|               |
          |--------------------------------------->|
          |                        |               |
          |<-------------------- "YES"             |
          |<------------------------------------ "YES"
          |                        |               |
   (2) "Commit!" ----------------->|               |
      *CRASH* |               |
     (Dies here)               (Commits)           |
                                               (WAITING...)
```

***2PL vs 2PC***: 
- 2PL (Two-Phase Locking) is a ***mechanism for Isolation (Concurrency)*** used to ***prevent race conditions on a single node*** by **acquiring locks before processing and releasing them afterward** ("Growing" then "Shrinking")
- 2PC (Two-Phase Commit) is a ***protocol for Atomicity (Consensus)*** used ***to ensure multiple nodes finish a transaction together*** by ***first voting to proceed*** and ***then executing the save*** ("Prepare" then "Commit")

#### Distributed Consensus Algorithms (Paxos, Raft, Zab)

To solve the blocking problem of 2PC, we use algorithms like **Paxos** or **Raft** (used in `etcd`) or **Zab** (used in ZooKeeper)

Use cases of Distributed Consensus Algorithms:
1. *Leader Election* (Who is the Boss?): Automatically picking one "Leader" node to handle writes. If the leader dies, the group votes for a new one ***instantly***
2. *Distributed Locking* (Single Access): Ensuring a specific task (like a daily payment job) runs on exactly one machine, preventing duplicate processing
3. *Cluster Metadata* (The "Truth"): Storing small, critical configuration data (like "Node A owns Partition 1") that must be 100% consistent across the entire system

How they are ***better*** than 2PC:
- **Majority Rules (Quorum)**: You don't need everyone to agree! You just need a majority (**`N/2 + 1`**)
	- Outcome: You  **only need 51% agreement!**
	- If one node is dead, the system keeps working
	- *Leader Election*: If the leader dies, the nodes automatically vote for a new one

```
SCENARIO: 3 Nodes Total. Majority needed = 2.

   [LEADER A] --(Proposal)--> [NODE B] --(ACK!)--+
       |                                         |
       +-------> [NODE C]                        v
                 (DEAD/OFFLINE)               COMMIT!
                                              (2/3 agreed.)
                                              (System stays UP!)
```

Leader election: ***Epochs*** (***Terms***). These algorithms divide time into "Epochs" (or Terms)
- Epoch 1: Leader is Node A
- Epoch 2: Leader is Node B

Every time a leader is thought to be dead, a vote starts, the Epoch number increases, and a new leader is chosen

```
TIME FLOW ------------------------------------------>

   +----------------+      +--------------+      +----------------+
   |    EPOCH 1     |      |   ELECTION   |      |    EPOCH 2     |
   |   Leader: A    | ---> |   (A Died!)  | ---> |   Leader: B    |
   | (Everyone obeys|      |   (Voting..) |      | (Everyone obeys|
   |      A)        |      |              |      |      B)        |
   +----------------+      +--------------+      +----------------+
```

(**Note**: Implementing *Paxos/Raft* is incredibly **hard**. Most developers shouldn't do it. Instead, we use **Consensus Services** like **ZooKeeper** or **etcd**). ***These services also help with partition/service discovery!***

**Paxos vs Raft**:
1. **Paxos (The Complex Academic Original)**
	- Paxos doesn't enforce a strict single leader initially
	- ***The key to Paxos is that a Proposer must first "reserve" a ticket number (Phase 1) before it is allowed to suggest a value (Phase 2)***
	- Any node can act as a proposer, leading to a complex, two-phase "dialogue" for every single decision, which can result in dueling proposers restarting the process
```
      [PROPOSER]                     [ACCEPTORS (Quorum)]
           |                                |
           |          PHASE 1: PREPARE      |
           |    "I want to propose with     |
           |     Ticket #100. OK?"          |
         1.|------------------------------->|
           |                                |
           |          PHASE 1b: PROMISE     |
           |    "OK, I promise to ignore    |
           |     anyone with Ticket < 100"  |
         2.|<-------------------------------|
           |                                |
   (Check: Did I get a majority of Promises?)
           |                                |
           |          PHASE 2: ACCEPT       |
           |    "Great. Since you promised, |
           |     please write Value: 'X'"   |
         3.|------------------------------->|
           |                                |
           |          PHASE 2b: ACCEPTED    |
           |    "Written. Value 'X' is      |
           |     locked for Ticket #100."   |
         4.|<-------------------------------|
           |                                |
        [CONSENSUS REACHED] -> Notify Learners
```

2. **Raft (Designed for Understandability)**
	- Raft decomposes consensus into two distinct, sequential steps.
	- First, **elect a strong leader**, and 
	- Second, that leader manages log replication in a strict, linear flow. This structure makes it much easier to implement and understand
		- Raft uses a Strong Leader. ***The Followers do not vote on every transaction; they simply do what the Leader tells them, provided the Leader is still valid (current Term)***

```
     LEADER                FOLLOWER 1           FOLLOWER 2
   (Log State)             (Log State)          (Log State)
   +---------+             +---------+          +---------+
   | Index 1 |             | Index 1 |          | Index 1 |
   +----|----+             +----|----+          +----|----+
        |                       |                    |
1. New: | [X=9]                 |                    |
        v                       |                    |
   +---------+            2. AppendEntries RPC       |
   | Index 2 |----------------->|                    |
   | [X=9]   |-------------------------------------->|
   +----|----+                  v                    v
        |                  +---------+          +---------+
        |                  | Index 2 |          | Index 2 |
        |                  | [X=9]   |          | [X=9]   |
        |                  +----|----+          +----|----+
        |                       |                    |
        |<-------(ACK)----------|                    |
        |<-------(ACK)-------------------------------|
        v
 3. COMMIT! (Majority agreed)
    (Safe to tell Client "Done")
```


| Feature | Paxos | Raft |
| -- | -- | -- |
| Philosophy | Peer-to-Peer: Any node can lead at any time | Strong Leader: One node rules; others obey |
| Complexity | Hard: Complex, subtle, difficult to implement | Easy: Designed specifically for understandability |
| Log Order | Flexible: Logs can have "holes" / out-of-order commits | Strict: Must be sequential (1, 2, 3...) |
| Liveness | Can "livelock" (dueling leaders) without optimization | System stops if Leader dies (until re-election) |
| Examples | Google Spanner, Cassandra (LWT) | Kubernetes (etcd), Consul, Kafka |

#### The FLP Result (Theoretical Limit)

- Theory: In a purely asynchronous system (where messages can be delayed forever), you cannot mathematically guarantee consensus if even one node might crash.
- Reality: We cheat. We use Timeouts. If a node doesn't reply in 5 seconds, we assume it's dead. This allows consensus to work in the real world

