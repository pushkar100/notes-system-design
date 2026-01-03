# Unique ID generator in a distributed system

Every modern distributed system relies on unique identifiers. From databases and message queues to payment processors and social platforms, unique IDs are the glue that holds everything together. Without them, it’s impossible to track entities, process requests reliably, or ensure data consistency across multiple servers

## Hard parts

1. **Clock synchronisation**: If different machines rely on timestamps, are they all synchronised to avoid conflicts?
2. **Section length tuning**: For low concurrency
3. **High Availability**: Mission critical system

## Functional requirements

Questions to ask:
1. What?: *What are the constraints on the ID?* *Ex: uniqueness?*
2. What?: *What type of characters should the ID contain?*
3. How?: *How long should the ID be?*
4. Result?: *What happens when a unique ID is generated?* *Passed to client*. **No formal FR required**
5. Where/Scale?: *Does the Unique ID generator need to be a separate system?*

### Functional Requirements as a result of these questions

1. ID must be ***unique*** and ***time sortable*** (Ex: sort by date)
2. ID must contain only numeric values
3. ID must fit into 64-bits (i.e A number represented by a combination of 64 1s and 0s)
4. ID generator may or may not be a separate system (Up to the designer)

## Non-functional requirements

Questions to ask:
1. Scale (Throughput) and Volume (Overall storage or Bandwidth): 
	- *What is the scale of this ID generation?* 
2. Speed (Latency/CPU/Memory):
	- *What is the speed at which a ID must be generated and returned?*
3. CAP trade-off (AP vs CP):
	- *Do we need the system to be consistent or highly available?*
4. Safety net (Fault Tolerance):
	- *How fault tolerant does the system need to be?*
5. Geography:
	- *Does it matter if a client uses it from a particular region? Ex: Asia*. *Ans: Just a distributed system*. **No formal NFR required**

### Non-Functional Requirements as a result of these questions

1. Ability to generate over 10,000 IDs per second
2. IDs must be generated and returned  within milliseconds (10ms)
3. ID generation is mission-critical (every entity needs an ID). Hence, system should be highly available
4. Account for faults in a highly available distributed system

## Core entities 

Objects and users persisted in the database or exchanged via APIs:
- ID

## System interface or API endpoints

System interface or API? A system interface makes sense since a client will generally use a library to integrate with a key-value store to perform CRUD on key-value pairs

1. **`generateID() --> Returns an ID (if successful) (else, error)`**

## High Level Design 

1. Starting with MVP: In this case, it is a client-server model (i.e `client-server-database`)
2. Go through *Functional Requirements* one by one and see if it is fulfilled. If not, modify design  (***Note: Need not go in order***)

```
 [client] --> [server]
```
- Server to generate ID
- A database is not required since the clients store the unique ID, not the generation service
	- *Note: We might need it in case we figure out that persistence is needed to fulfil any requirement*

*FR#1*: ID must be ***unique*** and ***time sortable***
- Single server ID generation: ***Does not work!***
	- Auto-increment works for a single system (non-distributed)
	- The same feature cannot guarantee consistent increment / prevent collisions in a multi-server distributed system!
- **We need a distributed ID generation system!** 
	- There are 4 common ways to generate IDs. We should compare them and select one:

**A) Multi-Master Replication**
- Multi-Master Replication for ID generation is a setup where multiple servers are active at the same time, and each one can generate and issue a unique ID independently. There is no single "Leader" that everyone must wait for
- How it works (The "Step" Strategy): To prevent two servers from creating the same ID (a collision), each server is assigned a starting point (Offset) and increases by a fixed amount (Step) equal to the total number of servers
- Pros
	1. *High Scalability*: You can generate IDs much faster because the load is split across 3 servers
	2. *Fault Tolerance*: If "Server 2" crashes, Server 1 and Server 3 continue working perfectly. The system never stops
- Cons
	1. *Not Time-Ordered*: ID 6 (from Server 3) might be generated before ID 4 (from Server 1) if Server 3 is faster. You cannot trust the ID order to represent time
	2. *Hard to Scale Out*: If you add a 4th server, you have to change the "Step" logic on all existing servers, which is complex and risky
- ***We cannot use this approach*** because of the cons (We need time ordering and servers most likely will be added / removed)
```
    [ Client A ]        [ Client B ]        [ Client C ]
         |                   |                   |
         v                   v                   v
+----------------+  +----------------+  +----------------+
|    Server 1    |  |    Server 2    |  |    Server 3    |
| (Start: 1)     |  | (Start: 2)     |  | (Start: 3)     |
| (Step: +3)     |  | (Step: +3)     |  | (Step: +3)     |
+----------------+  +----------------+  +----------------+
         |                   |                   |
    Gen: 1              Gen: 2              Gen: 3
    Gen: 4              Gen: 5              Gen: 6
    Gen: 7              Gen: 8              Gen: 9
    Gen: 10             Gen: 11             Gen: 12
         |                   |                   |
         v                   v                   v
   Unique IDs!         Unique IDs!         Unique IDs!
```

**B) UUID** (Unique User ID)
- A UUID (Universally Unique Identifier) is a standard method for generating IDs that are effectively unique across the entire world, without needing a central server to manage them. It is a 128-bit number, usually displayed as a 36-character string (e.g., `550e8400-e29b-...`)
- How it works (The "Local" Strategy)
	- Unlike other methods where you ask a server for an ID, with UUID, the client generates the ID itself
	- It uses a combination of the current time, the machine's unique network address (MAC), and random noise to ensure uniqueness
- Pros
	1. *Zero Latency*: No network call is needed. The ID is created instantly in memory
	2. *No Single Point of Failure*: Since there is no "ID Generator Server," nothing can crash or bottleneck the system
	3. *Offline Capable*: A mobile phone can generate valid IDs while in Airplane mode and sync them later
- Cons:
	1. *Too Long*: 128 bits is huge. It takes up significant space in databases and indexes compared to a simple 64-bit integer
		- What if you take the 64 bit prefix of a 128 bit UUID? 
			- It does not work because 128 bits guarantees extremely low collision probability *(For all practical purposes, the probability is effectively 0%, as the chance of a collision is astronomically low, i.e roughly one in `2.7 x 10^18`, even after generating a billion UUIDs)* 
			- BUT with reduced bits, collisions will be more frequent and we will have to account for it
	2. *Unreadable*: They are hard for humans to speak or type (550e8400...). May contain non-numeric digits as well
	3. *Not Sortable*: Most UUIDs are random. You cannot sort them by time to see which record is "newest"
- Due to the cons (longer than 64 bits, non-numeric characters, and not time sortable), ***we cannot use this approach either***
```
    [ Web Server A ]      [ Mobile App ]       [ Microservice ]
           |                    |                     |
   (Generates Locally)  (Generates Locally)   (Generates Locally)
           |                    |                     |
           v                    v                     v
   "a1b2-c3d4..."        "99z8-x7y6..."       "f5e4-d3c2..."
```

**C) Ticket Server**
- A Ticket Server is a centralized "counter" system. It is a dedicated database or server whose only job is to hand out sequential unique numbers (1, 2, 3...) to other servers upon request. It acts like the "take a number" machine at a deli counter
- How it works (The "Central Authority")
	- Instead of generating an ID locally, your web servers connect to this central Ticket Server and ask, "Give me the next number." 
	- The Ticket Server uses a database feature (like SQL `AUTO_INCREMENT` or `REPLACE INTO`) to guarantee the number is unique and sequential
	- Pros
		1. *Numeric & Sortable*: You get nice, short integers (e.g., 1001, 1002) that are easy to store and easy to sort by time
		2. *Simple Implementation*: It relies on standard database features that developers already know well
	- Cons
		1. ***Single Point of Failure***: If the Ticket Server goes down, nobody in your entire system can create new data. Your app effectively freezes
		2. *Latency*: Every time you need an ID, you must make a network call to the Ticket Server. This is slower than generating it locally (like UUID)
		3. *Bottleneck*: As your traffic grows, this single server gets hammered by requests and becomes the performance limit
- ***The single point of failure is enough to discard this approach in a mission-critical, highly available system***
```
   [ Web Server A ]      [ Web Server B ]
           |                     |
           | "Need ID!"          | "Need ID!"
           v                     v
    +-----------------------------------+
    |          TICKET SERVER            |
    |      (Central Database)           |
    |                                   |
    |   Current Counter: 10,402         |
    |   Action: +1                      |
    +-----------------------------------+
           |                     |
           v                     v
       Returns:              Returns:
       "10,403"              "10,404"
```

**D) Twitter Snowflake**
- Twitter Snowflake is a clever ID generation approach that creates 64-bit unique IDs (integers) that are roughly sorted by time. It combines the best of both worlds: it is distributed like UUID (fast, scalable) but sortable and readable like a Ticket Server
- How it works (The "Bit Layout" Strategy)
	- Instead of a random string, Snowflake constructs an ID by stitching together different pieces of information into binary bits
	- It doesn't rely on a central server for the number itself, but uses the Time and the Server's ID to ensure uniqueness
	- The 64-bit Formula:
		1. Time (41 bits): Milliseconds since a custom epoch (start date). This makes the IDs grow over time (Sortable)
		2. Machine ID (10 bits): Identifies which server generated the ID. This prevents collisions between different servers. For 10 bits, we can have 2^10 machines i.e 1024 machines! High scale!
		3. Sequence (12 bits): A local counter that resets every millisecond. This allows one server to generate 4096 IDs per millisecond
- Pros
	1. ***Time-Sorted:*** Because the "Time" bits are at the beginning, larger IDs = newer data. You can sort by ID to sort by creation time (Convert timestamp to UTC)
	2. ***High Performance***: It generates IDs locally in memory (no network call needed). It can handle over 4 million IDs per second per server
	3. ***Compact***: It fits into 64 bits (8 bytes), making it half the size of a UUID and much faster to index in a database
```
     [  Sign  ]   [    Timestamp (41 bits)    ]   [ Machine ID ]   [ Sequence ]
         |                   |                          |               |
         v                   v                          v               v
       (0)        (Time since Nov 4, 2010)       (Server #52)     (Counter)
      +---+-----------------------------------------+------------+--------------+
      | 0 | 001010111101010100101010101111010101010 | 0000110100 | 000000000001 |
      +---+-----------------------------------------+------------+--------------+
                                         |
                                         v
                         Decimal ID: 1288834974657
```
- **It is the perfect approach for us (Our choice)**

*FR#2: ID must contain only numeric values*
- This is satisfied by the Twitter Snowflake approach which contains only numeric data

*FR#3: ID must fit into 64-bits (i.e A number represented by a combination of 64 1s and 0s)*
- This is also satisfied by the Twitter Snowflake approach which contains exactly 64 bits

*FR#4: ID generator may or may not be a separate system (Up to the designer)*
- We will have it as a separate system (So that we can scale it i.e add nodes, load balance, add fault tolerance mechanisms, etc)

In the following whiteboarding:
1. **"ZooKeeper"/"etcd"** acts as a central coordinator that assigns a unique Machine ID (0–1023) to every server at startup. This prevents ID collisions in the Snowflake algorithm by guaranteeing that no two servers ever claim the same worker number
2. The **ID Generator Cluster**: The Factory Workers. It represents a cluster of many servers/nodes. It serves as the execution layer, where multiple servers generate unique IDs using the formula `[Time] + [Machine ID] + [Sequence]`. This distributed approach enables the system to scale horizontally, processing high traffic volumes (e.g., 10,000+ IDs/s) that would overwhelm a single server.
```
                                  +-------------------+
                                  |   Orchestrator    |
                                  | (ZooKeeper/Etcd)  |
                                  +---------+---------+
                                            |
                                 (Assigns Worker IDs: 1, 2, 3...)
                                            |
      +-------------+              +--------v--------+
      | Client App  |              |                 |
      | (Web/Mobile)|              |   ID Generator  |
      +------+------+              |     Cluster     |
             |                     |                 |
             | generateID()        +--------+--------+
             |                              |
             v                              |
      +------+------+              +--------v--------+
      |     Load    +------------->|  Node 1 (ID:1)  |
      |   Balancer  +----+         |  (Snowflake)    |
      +-------------+    |         +-----------------+
                         |
                         |         +-----------------+
                         +-------->|  Node 2 (ID:2)  |
                         |         |  (Snowflake)    |
                         |         +-----------------+
                         |
                         |         +-----------------+
                         +-------->|  Node N (ID:N)  |
                                   |  (Snowflake)    |
                                   +-----------------+
```

## Deep dive

1. Starting with constructed High Level Design:
2. Go through *Non-Functional Requirements* one by one and see if it is fulfilled. If not, modify design (***Note: Need not go in order***)
3. Also check some unique constraints of the problem you might have missed (scan or read the problem again) and apply solutions for them

*NFR#1: Ability to generate over 10,000 IDs per second*
- 41 bits for timestamp means that it can have a max value of 2^41 -1 ms = ~69 years since epoch
	- If we move the epoch (start date, originally at 1971) to sometime now, we get 69 years of ID generation which is timestamp unique
- Sequence numbers: 12 bits which is 4096 combinations. This is the combination for each millisecond
	- Therefore, we can have 4096 x 1000 = ~4 million IDs generated per second

*NFR#4: Account for faults in a highly available distributed system*
- Add more nodes to the cluster. Scale-out
- We do not need sharding i.e Leader or leaderless replicaton since each node independently generates the IDs (No read/write replication issues)
- Hence, only add more nodes to scale-out to ensure H.A

*NFR#2: Ensuring Low Latency (<10ms)*
- To achieve this, we must *remove the two slowest things* in computing: 
	1. **Disk I/O** and 
	2. **Network I/O**
- Solution for Disk I/O: **In-Memory "Bit-Banging"**.The generation process (`timestamp << 22 | node_id << 12 | sequence`) is pure math. It uses CPU bitwise operations, which take ***nanoseconds***, not milliseconds
- Solution for Network I/O: **Zero Coordination** (The "Shared Nothing" Architecture): Once a node starts up, it never talks to the database, Zookeeper, or other nodes to generate an ID. It generates IDs in isolation.
	- Contrast: A "Ticket Server" (database approach) requires a network round-trip and a disk lock for every single ID. That takes 5-20ms. Snowflake takes <1ms
- Result: The only latency is the HTTP round-trip from the client to the Load Balancer and back. The actual processing time is effectively zero

Note on **In-memory "Bit banging"**:
- Instead of asking a database "Give me the next ID" (which requires a slow trip to the hard drive/network), the server "bangs" the bits together itself—combining the current time, its own badge number, and a counter—instantly creating a unique ID in nanoseconds
- Why it works: You are trading coordination (checking with a central disk) for calculation (doing math locally). Since every server has a unique "Badge Number" (Machine ID), they can all "bang bits" simultaneously without ever colliding
```
METHOD 1: THE DISK TRIP (Slow)
[Server] ------------------------> [Database/Disk]
                                         |
                                   (Seek & Read)
                                         |
[Server] <------------------------ (Return ID)
Result: ~5-10 milliseconds (Eternity for a CPU)


METHOD 2: IN-MEMORY BIT-BANGING (Fast)
[Server]
   |
   +-- [CPU]: "Shift Timestamp left 22 bits" (<<)
   +-- [CPU]: "OR with Machine ID" (|)
   +-- [CPU]: "OR with Sequence" (|)
   |
   V
[Unique ID Created]
Result: ~10 nanoseconds (1,000,000x faster)
```
```
      RAW DATA                     THE BIT-BANGING PROCESS (In CPU Register)
      (Base 10)                    (64-bit Binary Representation)

1. TIME (Millis)    [1 1 0...]
   Shift Left 22    ----------->  [1 1 0 1 . . . 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
   (<< 22)                        |<- Timestamp (41 bits) ->| (Empty Space created for others)


2. WORKER ID (#5)                             [1 0 1]
   Shift Left 12    ----------->  [0 0 0 0 . . . 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0]
   (<< 12)                                                  |<- ID ->|   (Empty Space)


3. SEQUENCE (#0)                                                       [0 0 0 0 0 0 0 0 0 0 0 0]
   No Shift         ----------->  [0 0 0 0 . . . 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
                                                                         |<-- Sequence -->|

                                  =========================================================
   BITWISE OR (|)   ----------->  [1 1 0 1 . . . 0 0 0 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 0 0 0]
   (Combines All)                 |_________________________|____________|__________________|
                                          Timestamp           Worker ID       Sequence
                                        (Sortable)           (Unique)       (Collision)

   RESULT: One unique 64-bit integer generated in nanoseconds.
```

*NFR#3: Ensuring High Availability (Mission Critical)*
- To ensure the system never stops producing IDs, we rely on **Redundancy** and **Decoupling**
	1. **Active-Active Cluster**: You run multiple ID Generator nodes (e.g., 5 nodes) behind a Load Balancer. All of them are active!
	2. **Fault Isolation**: Since nodes don't share data, if Node 1 crashes, Node 2 doesn't care. It keeps working. The Load Balancer simply detects Node 1 is dead and routes traffic to Nodes 2-5
- The "Zookeeper Paradox": You might ask, ***"What if Zookeeper (the HR Manager) dies?"***
	- The Safety Net: *The ID generators only need Zookeeper at startup to get their Machine ID*. Once they have it, they cache it locally
	- Result: If Zookeeper goes down, the existing ID generators keep working perfectly. You just can't add new servers until Zookeeper is fixed. This makes the system incredibly resilient during outages

```
  +-----------------------------------------------------------------------+
  |                   INSIDE A SINGLE ID GENERATOR NODE                   |
  |                                                                       |
  |   INPUTS:                                                             |
  |   1. System Clock (ms)     2. Machine ID (Static)    3. Sequence      |
  |      (e.g., 1709...)          (Cached in RAM)        (Atomic Int)     |
  |           |                         |                      |          |
  |           v                         v                      v          |
  |   +----------------+       +----------------+      +----------------+ |
  |   | Shift << 22    |       | Shift << 12    |      | No Shift       | |
  |   | (Make space)   |       | (Middle bits)  |      | (Last 12 bits) | |
  |   +-------+--------+       +-------+--------+      +-------+--------+ |
  |           |                        |                       |          |
  |           |                        v                       |          |
  |           +--------------------->( OR )<-------------------+          |
  |                                    |                                  |
  |                                    v                                  |
  |                           [ FINAL 64-BIT ID ]                         |
  |                                                                       |
  |   PERFORMANCE:                                                        |
  |   - CPU Cycles: ~50                                                   |
  |   - Time:       ~10-20 Nanoseconds                                    |
  |   - I/O:        Zero (No Disk, No Network)                            |
  +-----------------------------------------------------------------------+
```

```
SCENARIO: Zookeeper Outage Handling

  PHASE 1: STARTUP (Strict Dependency)
  ------------------------------------
      [ Zookeeper Cluster ]
             |
             | "Assign: You are Machine ID #5"
             v
      [ ID Gen Node A ]
      (State: Starting...) -> (Stores ID:5 in RAM) -> (State: Ready)


  PHASE 2: RUNTIME (Decoupled Independence)
  -----------------------------------------
      [ Zookeeper Cluster ]
           ( CRASHED! )
           (    X     ) <-------+
                                |
                                | (Node A does NOT check ZK)
                                | (Node A continues using RAM)
                                |
      [ ID Gen Node A ] <-------+
      ( RAM: ID=5     )
             ^
             | generateID()
             |
      [ Client Request ]
      (Result: SUCCESS)
```

## How does snowflake approach prevent collisions?

- UUID relies on "Luck" i.e Probability
- ***Snowflake relies on "Orchestration" i.e Certainty***.

While the chance of a UUID collision is astronomically low, it is theoretically possible. With Snowflake, collisions are mathematically impossible—provided the system is configured correctly

Here is how the 3 parts of the Snowflake ID work together to make a collision impossible:
1. The "Time" Guard (Timestamp)
	- Scenario: You generate an ID now. I generate one tomorrow
	- Protection: Since our timestamps are different, the resulting binary bits will be different
	- Result: No Collision
2. The "Space" Guard (Machine ID)
	- Scenario: You and I both generate an ID at the exact same millisecond
	- Protection: This is where Zookeeper comes in. You are Worker #1. I am Worker #2. Even though the time bits are identical, the middle 10 bits (Machine ID) will differ
	- Result: No Collision
3. The "Speed" Guard (Sequence Number)
	- Scenario: You generate two requests on Worker #1 at the exact same millisecond
	- Protection: The local counter kicks in
	- Request A gets Sequence 0
	- Request B gets Sequence 1
	- Result: No Collision

The Only Way Snowflake Fails (The "Gotchas")
- Since Snowflake is deterministic, it only breaks if the **rules of reality are broken** by your infrastructure:
	1. *Duplicate Machine IDs*: If you accidentally manually configure two servers as Worker #1, and they generate IDs at the same millisecond -> Collision! (This is why we use Zookeeper to assign them)
	2. *Clock Rollback (NTP Drift)*: If the server's clock glitches and jumps back 5 seconds in time, it might generate an ID "from the past" that it has already issued -> Collision! (This is why the algorithm throws an error if `current_time < last_time`)

## Solving the hard parts

1. **Clock Synchronisation (Handling Drift)**
- The Fix: Use ***NTP (Network Time Protocol)*** to keep servers synchronized.
- The Safety Mechanism: If a server detects its clock has moved backwards (drifting into the past), it must refuse to generate IDs and wait until the clock catches up to the last generated timestamp. This prevents creating duplicate IDs for the "same" time twice

2. **Section Length Tuning (Bit Allocation)**
- The Fix: Customize the Bit Layout
- Low Concurrency: If you don't need 4,096 IDs per millisecond (12 bits), reduce the Sequence to 8 bits (256 IDs/ms)
- The Benefit: You can give those 4 saved bits to the Timestamp section. This extends the lifespan of your system from ~69 years to over 100+ years before running out of numbers

3. **High Availability (Mission Critical)**
- The Fix: Active-Active Architecture
- How: Run multiple ID Generator nodes behind a Load Balancer. Because these nodes do not share a database (Shared-Nothing), they are independent
- Result: If Node 1 crashes, the Load Balancer instantly routes traffic to Node 2. The system never stops

## Redis 

We can also use Redis key-value pair to generate unique IDs. Redis stores data in memory, so this approach offers better performance than the database

**How?** Redis generates unique IDs by functioning as a centralized, high-speed counter stored entirely in RAM, utilizing its atomic `INCR` (increment) command. Because Redis is ***single-threaded***, it processes incoming requests one by one, effectively lining them up; even if ten different servers request an ID at the exact same microsecond, Redis executes them sequentially (e.g., returning 101, then 102, then 103). 
- This *ensures absolute uniqueness and massive throughput (millions of IDs per second) without the slow disk I/O or complex locking mechanisms required by traditional SQL databases*

**The Catch (Persistence)**: Since Redis lives in RAM, if the power goes out, you lose your counter. You must enable **AOF** (Append Only File) or **RDB** (Snapshots) in Redis to save the number to disk periodically so you don't start over from 0 after a restart

```
     [ Web Server A ]      [ Web Server B ]
           |                     |
           | 1. "INCR id"        | 1. "INCR id"
           v                     v
    +---------------------------------------+
    |                REDIS                  |
    |  (Single Threaded "Traffic Cop")      |
    |                                       |
    |  Current Value: 500                   |
    |   - Request A comes first -> 501      |
    |   - Request B comes next  -> 502      |
    +---------------------------------------+
           |                     |
           | 2. Returns:         | 2. Returns:
           v                     v
         "501"                 "502"
```
