# Quick Revision: Consistent Hashing

This is one of the most critical concepts for "Distributed Data" systems (Databases, Caches). If you are designing a system that stores data across multiple nodes, you almost certainly need this

## Table of Contents

- [Quick Revision: Consistent Hashing](#quick-revision-consistent-hashing)
  - [Why not just use Modulo?](#why-not-just-use-modulo)
  - [The Solution: Consistent Hashing](#the-solution-consistent-hashing)
  - [The Hotspots Gotcha & data skew](#the-hotspots-gotcha-data-skew)
    - [The Fix with Virtual Nodes](#the-fix-with-virtual-nodes)
  - [Comparing hashing techniques](#comparing-hashing-techniques)
  - [Trade-offs of consistent hashing](#trade-offs-of-consistent-hashing)
  - [Real-world use case for consistent hashing](#real-world-use-case-for-consistent-hashing)
  - [Consistent hashing and the CAP theorem](#consistent-hashing-and-the-cap-theorem)

## Why not just use Modulo?

Before understanding Consistent Hashing, you must understand the problem with the *naive* approach: **Modulo Hashing**

Imagine you have 4 cache servers. You need to decide where to store the key `user_123`. The simple math is: `server_index = hash(key) % 4`

The Scenario:
- We have 4 servers: Node A, B, C, D
- We want to store 4 keys
```
[Key 1] -> Hash(10) % 4 = 2 -> Node C
[Key 2] -> Hash(11) % 4 = 3 -> Node D
[Key 3] -> Hash(12) % 4 = 0 -> Node A
[Key 4] -> Hash(13) % 4 = 1 -> Node B
```

**The Failure**: What happens if Node C *crashes* (Scale down) or we *add* Node E (Scale up)? The divisor changes from 4 to 3 or 5

- `Hash(10) % 5` might now be `0` (Node A)
- Suddenly, almost every key maps to a new server!

**Consequence**: 
- A **"Cache Avalanche/Stampede"** OR A **massive data migration in a database**: All your cache values / data becomes invalid instantly. The database gets hammered. The system dies

Mathematically, when `N` changes to `N+1` nearly 100% of your keys change their target server. This means your entire cache cluster becomes "Cold" instantly. The database is suddenly hit with traffic for every single key in your system at once

```
 [ Server A ]   [ Server B ]   [ Server C ]
    |              |              |
 (Holds keys)   (Holds keys)   (Holds keys)
  0,3,6...       1,4,7...       2,5,8...

        CRITICAL EVENT: Add [ Server D ]

Result: Key 4 now belongs to A? Key 5 to D?
        Key 0 is fine, but Key 6 moves to B?
        
CHAOS: ~75% of all keys must move to a new server.
```

## The Solution: Consistent Hashing 

Consistent Hashing solves this by ensuring that when a server is added or removed, *only a small fraction of keys **(`1/N`)** need to move*. *The rest stay where they are*

**Core Concept**: **The Hash Ring**. Imagine the entire range of possible hash values (e.g., `0` to `2^32-1`) is ***bent into a circle***
- Hash the Servers: We place the servers on this ring based on their IP or ID
- Hash the Keys: We place the data keys on the *same* ring
- **The Rule**: To find which server stores a key, ***go clockwise from the key's position until you find a server***

Step-by-Step visualization:
- Initial State (3 Servers)
- We place Servers A, B, and C on the ring
- We place Keys 1, 2, 3
- Key 1 is at 2 o'clock: Move Clockwise --> Hits Server B (4 o'clock)
- Key 2 is at 6 o'clock. Move Clockwise --> Hits Server C (8 o'clock)
- Key 3 is at 10 o'clock. Move Clockwise --> Hits Server A (12 o'clock)

```
             [Server A] (0) 
              /          \
          (k3)            (k1)
          /                  \
    [Server C]             [Server B]
          \                  /
           -------(k2)-------
```

**Result**: ***Even distribution***


**Handling Failure (Removing Server B)**

- Server B crashes. We remove it from the ring
- Key 1 (2 o'clock) looks for Server B, doesn't find it. It keeps going clockwise
- It hits Server C (8 o'clock)
- Key 2 and Key 3 are *unaffected*

```
       (No massive reshuffle!)
               [Server A]
              /          \
          (k3)            (k1 maps to C now)
          /                  \
    [Server C]             (Server B Gone)
          \                  /
           -------(k2)-------
```

Result: ***Only the keys belonging to the failed node moved***

**Scaling Up (Adding Server D)** 

- We add Server D at 3 o'clock
- Key 1 (2 o'clock) moves clockwise. It now hits Server D (3 o'clock) instead of Server B
- Other keys stay put

```
           [Server A]
          (12 o'clock)
          /            \
         /              * Key 1 (2 o'clock)
        /                \
   [Server C]             \ (Travels Clockwise)
   (8 o'clock)             \
       ^                    v
       |               [Server D] (3 o'clock)
       |               (NEW OWNER)
       |                    |
        \                   |
         \                  v
          -------------- [Server B]
                         (4 o'clock)
 ```

Result: Only key 1 moves, others stay put (unaffected)

## The Hotspots Gotcha & data skew

In the simple ring above, it's very *unlikely* that 3 servers will split the ring into *perfectly even* `33.33%` chunks.

**Problem**: Server A might end up controlling 10% of the ring, while Server B controls 80%. Server B will be overloaded ("**Hotspot**")

> **Hotspot definition**: In consistent hashing, a hotspot happens when the ***random placement of servers on the ring is uneven***, causing *one server to cover a huge "range*" (e.g., 80% of the ring) and *handle the vast majority of traffic while others sit idle*
```
 [Server A]
      (12 o'clock)
        |
    10% | Traffic
        v
      [Server B]
      (1:00 o'clock)
        |
        |
        |
        |  <-- HUGE GAP (80% of Ring)
        |      All this traffic hits the
        |      next server clockwise...
        |      which is [Server C]!
        |
        v
      [Server C]
      (11:00 o'clock)
```

> **Data skew** occurs when the *data itself is unevenly distributed*, not because of the server ranges, but because *a specific Shard Key (like `user_id`) has disproportionately more data or traffic than others*

**Example** (The "**Celebrity Problem**"): If you shard your database by `user_id`, the shard containing "Justin Bieber" (`User ID: 1`) will receive *millions* of comments/likes, while the shard containing "User 99" receives almost zero. *The server hosting Bieber will crash, even if the number of users per server is equal*

```
SHARDING STRATEGY: By User_ID

[ Server 1 ]       [ Server 2 ]       [ Server 3 ]
(Users A-J)        (Users K-R)        (Users S-Z)
    |                  |                  |
    |                  |                  |
   [A]                [K]                [S]
   [B]                [L]                [T]
   [JustinBieber]     [M]                [U]
   (10M Requests)      |                  |
    |                  |                  |
    v                  v                  v
  CRASHED!           Idle               Idle
```

### The Fix with Virtual Nodes

**VNodes**: Instead of placing "Server A" on the ring once, we map it to 100s of positions on the ring (e.g., `A_0`, `A_1` ... `A_99`). We do this for *every* server!

Effect: The ring becomes a **mix of randomly interleaved nodes:** `A -> B -> C -> A -> C -> B...`

Benefit:
1. **Balance**: Statistically, every server gets a roughly equal slice of the total ring.
2. **Recovery**: If Server A crashes, its load (the keys for `A_0...A_99`) is ***split evenly*** among B and C, rather than dumping it all on just one neighbor

```
        Simple Ring        vs.        VNode Ring

    [A]---------------[B]      [A1]---[B1]---[C2]---[A2]
     |                 |        |                     |
     |      UNEVEN     |        |   INTERLEAVED       |
     |                 |        |   (Balanced)        |
    [C]----------------+       [B2]---[A3]---[C1]---[B3]
```

## Comparing hashing techniques

|Feature|Modulo Hashing (% N)|Consistent Hashing|
|-------|--------------------|------------------|
|Logic  |Hash(key) % ServerCount|Find first server clockwise on ring.|
|Adding Server|Catastrophic. Almost all keys move.|Minimal. Only keys for new server move.|
|Removing Server|Catastrophic.       |Minimal. Only keys from failed server move.|
|Data Balance|Naturally balanced (if Hash is good).|Needs Virtual Nodes to be balanced.|
|Complexity|Extremely Simple.   |Moderate (Requires binary search to find node).|

## Trade-offs of consistent hashing

Pros:
1. **Elasticity**: You can add/remove nodes without downtime
2. **Failure Isolation**: If a node dies, only its immediate neighbors (on the ring) feel the pressure
3. **Hotspot Mitigation**: With VNodes, a powerful server can be assigned more virtual nodes (e.g., 200) than a weak server (e.g., 50), balancing load by capacity

Cons:
1. **Implementation Complexity**: Harder to write than % N. (You need a sorted map structure like a Red-Black Tree to perform the "find next clockwise" search efficiently).
2. **Cascading Failures**: If one node dies and its load shifts to the next node, that node might become overloaded and die too. (***Fixed by VNodes***)

## Real-world use case for consistent hashing

Whenever we have **Distributed Data** where **Partitioning** is involved, consistent hashing is used

1. **Distributed Caches (Redis Cluster / Memcached)**
	- To decide which RAM stick stores which key
2. **Distributed Key-Value Stores (DynamoDB / Cassandra)**
	- These databases use Consistent Hashing (often called "The Token Ring") to determine Partition placement
3. **Distributed Rate Limiters**
	- If you have 50 rate limiter servers holding counters (`user_123: 5 reqs`), you need to know which server holds `user_123`'s counter.
4. **Chat Applications (Discord/Slack)**
	- Mapping a "Server/Guild" to a specific "Voice Service Node." If a node restarts, you don't want to disconnect everyone, just the people on that specific node

## Consistent hashing and the CAP theorem

Consistent Hashing is the mechanism that essentially enables the **"P" (Partition Tolerance)** in CAP.

It provides the map for where requests should go when the network is partitioned (i.e., when a server dies or is unreachable). The "Tie-in" to CAP happens based on what the new target server does when it receives the traffic:

- **The AP Route (Availability)**: The new server accepts the write immediately, even if it doesn't have the old history yet. (Used by Dynamo, Cassandra)
- **The CP Route (Consistency):** The new server blocks/rejects the request until it can verify it has the latest data. (Used by HBase, MongoDB)

Example:
- Partition Event: Server A dies. Consistent Hashing routes traffic to Server B
- AP Choice: Server B says, "I'll accept this write now!" (System stays UP, but B might have missing data i.e inconsistency)
- CP Choice: Server B says, "Error: I am not the leader yet," or waits to sync. (System goes DOWN for that key i.e unavailability, but data is safe

```
    [ Network Partition / Node Failure ]
                      |
        Consistent Hashing re-routes
           traffic to [ Server B ]
                      |
          +-----------+-----------+
          |                       |
   (Option 1: AP)           (Option 2: CP)
"I accept the write!"    "Reject! I'm syncing."
          |                       |
   [ High Availability ]    [ Strong Consistency ]
   (Cassandra/Dynamo)          (HBase/Redis)
```
