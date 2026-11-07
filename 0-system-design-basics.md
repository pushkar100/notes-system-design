# 🏛️ System Design Basics

## Table of Contents

- [🎯 Goal](#goal)
- [🖥️ The Simplest Setup (Single Server)](#the-simplest-setup)
- [📈 Handling User Growth Better](#handling-user-growth-better)
  - [Type of Database to Use: Relational vs. NoSQL](#type-of-database-to-use)
  - [Vertical vs. Horizontal Scaling](#vertical-vs-horizontal-scaling)
    - [Understanding Redundancy, SPOF, Failover & Failover Support](#understanding-redundancy-spof-failover--failover-support)
- [⚖️ Handle Traffic Growth with Load Balancers](#handle-traffic-growth-with-load-balancers)
  - [Can a Load Balancer Be a Single Point of Failure (SPOF)?](#can-a-load-balancer-be-a-single-point-of-failure-spof)
- [🔄 Handle Data Tier Growth with Database Replication](#handle-data-tier-growth-with-database-replication)
  - [Replication and Failover Challenges (and Solutions)](#replication-and-failover-challenges-and-solutions)
  - [What if the System is Write-Heavy Instead of Read-Heavy?](#what-if-the-system-is-write-heavy-instead-of-read-heavy)
- [⚡ Improve Response Times with Cache Layers](#improve-response-times-with-cache-layers)
  - [Database Cache Tier](#database-cache-tier)
  - [Static Content Cache with CDNs](#static-content-cache-with-cdns)
    - [Can a CDN Be a Single Point of Failure (SPOF)?](#can-a-cdn-be-a-single-point-of-failure-spof)
  - [Cache Considerations for Both Data and Static Assets](#cache-considerations-for-both-data-and-static-assets)
- [👤 Managing Session State in Horizontally Scaled Web Servers](#managing-session-state-in-horizontally-scaled-web-servers)
  - [What Happens if Session Data is Not Separated?](#what-happens-if-session-data-is-not-separated)
  - [The Recommended Approach: Stateless Web Tier with Shared Storage](#the-recommended-approach-stateless-web-tier-with-shared-storage)
    - [Can the Shared Session Storage Become an SPOF?](#can-the-shared-session-storage-become-an-spof)
- [🏢 Complete Set of Optimizations for One Single Data Center](#complete-set-of-optimizations-for-one-single-data-center)
- [🌐 Scaling for Users Across Multiple Geographical Areas](#scaling-for-users-across-multiple-geographical-areas)
- [📬 Decoupling Tiers with Message Queues](#decoupling-tiers-with-message-queues)
  - [Can a Message Queue Be a Single Point of Failure (SPOF)?](#can-a-message-queue-be-a-single-point-of-failure-spof)
  - [Essential Message Queue Concepts You Must Know](#essential-message-queue-concepts-you-must-know)
- [🗄️ Scaling Databases Beyond Master-Slave Replication with Sharding](#scaling-databases-beyond-master-slave-replication-with-sharding)
  - [Why Master-Slave Replication Falls Short as Traffic Grows](#why-master-slave-replication-falls-short-as-traffic-grows)
  - [What is Database Sharding?](#what-is-database-sharding)
  - [How Sharding Works](#how-sharding-works)
  - [Benefits and Challenges of Sharding](#benefits-and-challenges-of-sharding)
- [📊 Logging, Metrics, and Automation](#logging-metrics-and-automation)
- [💡 Completely Scaled Fundamental Design with Tips](#completely-scaled-fundamental-design-with-tips)

---

<a id="goal"></a>
## 🎯 Goal

> [!TIP]
> **System Design Interview Framework:** Always start by clarifying functional and non-functional requirements (traffic scale, latency SLAs, read/write ratio) before drawing architecture diagrams.

About the question:

- The question is to **_design an architecture_**
- The question maybe open-ended and the required processes unclear
- The interviewer may choose to focus on either high-level architecture or focus on one area

About the solution:

- The right strategy as well as knowledge are important to do well
- We need a step-by-step framework i.e a systematic approach to find a solution

<a id="the-simplest-setup"></a>
## 🖥️ The Simplest Setup

The simplest setup is a setup that is used for an extremely small and finite number of users. Ex: Just one user.

In this setup, we maintain one server for all services! That is, it is a 3-in-1 server

1. Web / Mobile traffic server: (Ex: JSON for an API / HTML for a page request)
   - Ex: mobile client may hit api.mysite.com (which returns JSON)
   - Ex: web client may hit www.mysite.com (which returns HTML)
2. Database server
3. Cache server

![basic server](https://i.imgur.com/2lFq310.png)

Even for a client accessing via a different platform, the same server handles the request:

```
[User (Mobile)] <== send domain name, receive IP ==> [DNS]
||
(hit IP)
||
|==> [Our web server]
```

<a id="handling-user-growth-better"></a>
## 📈 Handling User Growth Better

When the number of users starts to scale, one server handling all 3 functions (web/mobile traffic, database, and cache) is not scalable!

First step to optimization is to split the web/mobile traffic from the database server! We will talk about cache later as it has still not been implemented.

Servers needed:

1. Web/Mobile traffic server
2. Database server
   Now, these two can be scaled independently!

![Separate DB](https://i.imgur.com/Abf6F9w.png)

<a id="type-of-database-to-use"></a>
### 🗃️ Type of Database to Use

> [!TIP]
> **Default to Relational (RDBMS):** Unless you have specific requirements for massive write throughput, flexible schema, or ultra-low single-key latency, always pick a battle-tested relational database (e.g., PostgreSQL, MySQL) as your baseline.

There are two options:

1. Relational DBs (RDBMS or SQL DBs):
   - Examples: MySQL, PostgreSQL, Oracle DB
2. Non-relational or NoSQL DBs
   - Examples: Cassandra, DynamoDB, MongoDB, CouchDB

**Relational DBs**

1. Data is in the format of _tables_ and _rows_
2. Tables can be joined using SQL statements

**NoSQL DBs**

1. Can be of 4 types:
   - Key-value stores
   - Graph stores
   - Column stores
   - Document stores
2. We cannot perform joins in general i. not supported

**_Default choice must be a relational database_** because of its benefits. Only use non-relational DB for one of the four reasons:

1. Your application requires **_super low latency_**
   - _Why:_ No complex joins, query parsing, or heavy relational locks; optimized for fast single-key lookups or in-memory access.
   - _Real-world use cases:_ Real-time gaming leaderboards, high-frequency trading, ad-tech real-time bidding (<50ms end-to-end SLA), user authentication session caches.
   - _Relational vs. NoSQL Example:_
     - **Relational limitation:** Running a SQL query involves parsing the syntax, acquiring locks, scanning B-tree indexes, and resolving multi-table joins, routinely taking 15–50 ms:
       ```sql
       -- Overhead: SQL parsing, transaction logging, lock contention, and foreign key join
       SELECT u.id, s.session_data FROM users u JOIN sessions s ON u.id = s.user_id WHERE s.token = 'abc123xyz';
       ```
     - **NoSQL advantage:** In-memory key-value stores (e.g., Redis, Memcached) bypass query parsers, joins, and disk locks for deterministic sub-millisecond ($O(1)$ in $<1\text{ ms}$) lookups:
       ```python
       # Direct O(1) in-memory hash fetch (0.2 - 1 ms latency)
       session_data = redis.get(f"session:{session_token}")
       ```
2. Your data is either **_unstructured_** or **_not relational_**
   - _Why:_ Dynamic, schema-less structure easily handles changing, deeply nested, or heterogeneous data without rigid schemas or migrations.
   - _Real-world use cases:_ E-commerce product catalogs with wildly differing attributes (clothing vs. laptops vs. groceries), IoT sensor telemetry with diverse payloads, user-generated custom forms.
   - _Relational vs. NoSQL Example:_
     - **Relational limitation:** Every row must conform to a rigid predefined schema. Storing heterogeneous items requires either dozens of sparse `NULL` columns or an awkward Entity-Attribute-Value (EAV) pattern that requires multiple joins just to read one product:
       ```sql
       -- EAV anti-pattern: Multiple joins required to reconstruct one product's attributes
       SELECT p.name, a.attr_name, v.attr_value
       FROM products p
       JOIN product_values v ON p.id = v.product_id
       JOIN attributes a ON v.attr_id = a.id
       WHERE p.id = 101;
       ```
     - **NoSQL advantage:** Document stores (e.g., MongoDB, Couchbase) store self-contained documents where each record defines its own schema, with zero migration overhead or empty columns:

       ```json
       // Laptop record:
       { "id": 101, "category": "electronics", "specs": { "cpu": "M3 Max", "ram_gb": 36 } }

       // Running shoes record in the exact same collection:
       { "id": 102, "category": "apparel", "specs": { "size": 10.5, "color": "blue", "waterproof": true } }
       ```

3. You only need to **_serialize and deserialize data_**. Ex: JSON/XML/YAML
   - _Why:_ Direct document/object storage eliminates Object-Relational Mapping (ORM) overhead and impedance mismatch by aligning the database storage model directly with application objects.
   - _Real-world use cases:_ Storing complete checkout orders, user profiles, blog posts with nested comments, or application configuration blobs.
   - _Relational vs. NoSQL Example:_
     - **Impedance mismatch** is just a fancy way of saying that _your application code and your database speak two completely different languages, making it hard for them to share data_. (For RDBMS (SQL): Developers use ORMs (Object-Relational Mappers) as a translator. The ORM does the tedious work of breaking the 3D object apart into flat tables when saving, and putting it back together when reading. For NoSQL (Document DBs): Databases like MongoDB store data directly as JSON documents. Because JSON matches the nested, tree-like structure of application code perfectly, the mismatch disappears. There is no need to break the data apart.)
     - **Relational limitation (Impedance Mismatch):** An in-memory object (like an `Order` containing customer info, line items, and shipping addresses) must be shredded across multiple normalized tables with foreign keys and reassembled with multi-table `JOIN`s on read:
       ```sql
       -- Requires multi-table transaction on write and multi-table JOINs on read
       BEGIN TRANSACTION;
       INSERT INTO orders (id, customer_id, total) VALUES (42, 1001, 150.00);
       INSERT INTO order_items (order_id, sku, qty, price) VALUES (42, 'SKU-A', 2, 50.00), (42, 'SKU-B', 1, 50.00);
       INSERT INTO shipping_addresses (order_id, street, city) VALUES (42, '123 Main St', 'Austin');
       COMMIT;
       ```
     - **NoSQL advantage:** Document databases store the native JSON payload directly as a single cohesive unit, saving and retrieving in a single atomic I/O operation without ORM translation:
       ```javascript
       // Persist and fetch the native application object directly as 1 atomic document
       await db.collection("orders").insertOne({
         order_id: 42,
         customer_id: 1001,
         items: [
           { sku: "SKU-A", qty: 2, price: 50.0 },
           { sku: "SKU-B", qty: 1, price: 50.0 },
         ],
         shipping: { street: "123 Main St", city: "Austin" },
       });
       ```
4. You need to **_store massive amounts of data_**
   - _Why:_ Natively architected for horizontal scaling and automatic sharding/partitioning across distributed commodity servers using consistent hashing.
   - _Real-world use cases:_ Write-heavy, massive scale applications: chat message histories (e.g., Discord handling trillions of messages), clickstream analytics, IoT time-series logs generating terabytes/petabytes per month.
   - _Relational vs. NoSQL Example:_
     - **Relational limitation:** RDBMS engines primarily scale vertically, hitting hardware limits on a single machine. While manual application-level sharding is possible, it strips away foreign key constraints, makes cross-shard joins impossible, and complicates shard rebalancing.
     - **NoSQL advantage:** Distributed wide-column stores (e.g., Apache Cassandra, ScyllaDB, Amazon DynamoDB) partition data automatically across a peer-to-peer ring. Adding storage or write throughput simply requires adding more nodes:

       ```text
       Consistent Hashing Ring (Linear horizontal scale-out):
                    [Node 1: Tokens 0 - 2^63-1]
                         /              \
       [Node 4: Tokens -2^63 - (-2^62)]   [Node 2: Tokens 2^63 - 2^64-1]
                         \              /
                    [Node 3: Tokens -2^62 - 0]

       - Write throughput scales linearly (e.g., 4 nodes = 100k writes/sec, 40 nodes = 1M writes/sec).
       - Masterless architecture ensures zero Single Point of Failure (SPOF) at petabyte scale.
       ```

<a id="vertical-vs-horizontal-scaling"></a>
### ⚖️ Vertical vs. Horizontal Scaling

We have two servers (web/mobile traffic and database) we can scale up independently. What is the best way to do so when needed? There are two options:

1. Vertical scaling (Scale-up)
2. Horizontal scaling (Scale-out)

**Vertical scaling**

- We add more CPU and RAM power to the existing server
- Only fine when the traffic, i.e number of users, is low
- It has many drawbacks:
  1.  CPU/RAM scaling has a hard-limit. It cannot be unlimited!
  2.  It does not have **_failover_** or **_redundancy_** i.e If the single server fails, your website goes down for all users! This is also known as a **_Single Point of Failure_**

_(My note: Another issue with vertical scaling maybe **cost**. It is expensive to get more powerful servers instantiated and to keep them running)_

**Horizontal scaling**
This is the **_preferred or recommended approach_** for large scale applications!

- It allows us to scale by adding more servers into our pool of resources
- Failover and redundancy can be made possible!

<a id="understanding-redundancy-spof-failover--failover-support"></a>
#### Understanding Redundancy, SPOF, Failover & Failover Support

- **Redundancy**: Having duplicate copies of critical components (e.g., multiple web servers, duplicate databases, multiple network switches). Redundancy is the prerequisite for high availability—you cannot switch to a backup if no backup exists.
- **Single Point of Failure (SPOF)**: Any individual part of a system that, if it ceases to function, will cause the entire system to fail. A single server in vertical scaling is a classic SPOF.
- **Failover**: The process of shifting traffic and workload away from an unhealthy or failed primary component to a redundant, healthy backup component.
- **Failover Support**: The underlying automated mechanisms and infrastructure that make failover possible without human intervention:
  - _Heartbeat / Health Checks_: Continuous polling to detect node death or degradation within milliseconds.
  - _Automated Traffic Rerouting_: Load balancers or DNS automatically removing dead nodes from rotation.
  - _State & Data Synchronization_: Ensuring the standby component has replicated data ready to take over without data loss or corruption.

**Why is Failover Support required?**

1. **Hardware failure is a certainty at scale**: In large systems with hundreds or thousands of nodes, hardware crashes, network splits, and disk failures happen daily.
2. **Eliminates manual downtime**: Without automated failover support, recovering from a crashed server requires an on-call engineer to get paged, diagnose the issue, boot a new machine, and manually reroute traffic—causing minutes or hours of total outage.
3. **Preserves SLAs and Business Continuity**: Modern services promise high availability (e.g., 99.99% uptime, allowing under 53 minutes of downtime per _year_). Automated failover detects failures and reroutes requests in milliseconds, rendering server failures invisible to users.
4. **Prevents Cascading Failures**: When one server dies without failover routing, remaining servers can easily get overwhelmed by traffic spikes, leading to a total system collapse. Automated failover support manages traffic redistribution safely.

<a id="handle-traffic-growth-with-load-balancers"></a>
## ⚖️ Handle Traffic Growth with Load Balancers

> [!TIP]
> **Managed Cloud Load Balancers:** In modern cloud platforms (AWS ALB, GCP Cloud LB), redundancy and multi-AZ failover are managed automatically. You don't need to manually configure Keepalived/VIPs unless running on-premise hardware.

When simultaneous access (maybe #users or user activity) to a server increases the load on it, we will eventually reach the "load limit". Solution: Use a **_load balancer_** coupled with multiple servers behind it (i.e load balancer + horizontal web traffic server scaling)

![load balancer](https://i.imgur.com/xC9OLdD.png)

- Load balancer is the entity that interacts with the rest of the internet on behalf of your servers
- Load balancer will have a public IP
- Load balancer will use private IPs to interact with the servers handling web traffic

**Why private IP for servers?**
It is because it can be accessed only by servers in the same network and not the internet.

**Benefits of a load balancer**

1. Balances the **_load_** on servers
2. No **_failover_** issues: If one server goes offline, all the traffic will be routed to one of the remaining servers (We can also add a healthy server to the pool of servers to balance the load)
3. Improved **_availability_**: When web traffic _increases rapidly_, we only need to add more server and the load balancer will balance the traffic between them gracefully

<a id="can-a-load-balancer-be-a-single-point-of-failure-spof"></a>
### Can a Load Balancer Be a Single Point of Failure (SPOF)?

**Yes!** If you deploy only a single load balancer and it crashes, all your web servers become completely unreachable from the internet—even if every web server is healthy. The load balancer itself becomes a Single Point of Failure.

#### How is this solved in production?

To eliminate this SPOF, production architectures deploy **redundant load balancers** using one of two primary approaches:

##### 1. Active-Passive Pair with Virtual IP (VIP) (Standard On-Premise / Self-Hosted)

- **How it works:** You run two load balancer instances: **Primary (Active)** and **Secondary (Standby)**.
- **Heartbeat & Floating IP:** Both load balancers share a single public **Virtual IP (VIP)**. They constantly exchange "heartbeat" signals over the local network (using protocols like VRRP or software like Keepalived).
- **Instant Failover:** If the Active load balancer fails to send a heartbeat, the Standby detects the failure and instantly claims the Virtual IP address. Traffic shifts to the backup load balancer in seconds without changing DNS records.

```text
               Internet (Users)
                      |
                      v
             [ Virtual IP (VIP) ]
                      |
        +-------------+-------------------------+
        |                                       |
        v                                       v
 [ Active Load Balancer ] <--(Heartbeat)--> [ Standby Load Balancer ]
 (Handles 100% traffic)                      (Takes over VIP if Active fails)
        |
        +-------------------+
        |                   |
        v                   v
  [ Web Server 1 ]    [ Web Server 2 ]
```

##### 2. Active-Active with DNS Anycast or Round-Robin (High-Scale & Cloud)

- **Multiple active load balancers:** Two or more load balancers run simultaneously and share the incoming load.
- **Anycast BGP / DNS Health Checks:** Routers or DNS distribute requests across multiple load balancer IPs. If one load balancer stops responding to health checks, DNS/Anycast routing automatically excludes it from the pool.
- **Managed Cloud Load Balancers (AWS ALB, GCP Cloud LB, Azure LB):** In modern cloud environments, cloud providers handle this redundancy automatically behind a single static IP or DNS name. Under the hood, they provision multiple load balancer instances across separate physical data centers (Availability Zones) with built-in auto-scaling and zero-downtime failover.

<a id="handle-data-tier-growth-with-database-replication"></a>
## 🔄 Handle Data Tier Growth with Database Replication

Just like web traffic growth prompts scaling of web servers, the database too will need to be scaled (i.e. data tier scaling) when simultaneous queries push a single database server to its load limit.

> [!IMPORTANT]
> **Master-Slave replication is primarily designed for read-heavy systems.**
>
> **Why?** In this architecture, **all write operations still funnel into a single master database**. Adding more slave databases only expands your ability to process **read** queries in parallel—it does not scale write capacity (and in fact, having too many slaves can increase the master's replication overhead). Because typical web and mobile apps have far higher read-to-write ratios (often 10:1 to 100:1), master-slave replication is the standard default for scaling read-heavy workloads.

The recommended approach for read-heavy systems is to use **_Database replication_** with a _master-slave_ setup:

- **Master**: Only supports _write_ operations (Create, Update, Delete). It records all changes and streams them to replicas.
- **Slaves**: Continuously get copies of data from the master (replication) and only support _read_ operations (Read). As read traffic grows, you scale horizontally by adding more slaves.

![database replication](https://i.imgur.com/3WXYaeq.png)

**Advantages**

1. **Better performance**: More read queries are _processed in parallel_ due to more slaves
2. **Reliability**: If one of the database servers fail, the _data is still preserved_ (replicated across multiple locations
3. **High availability**: Website remains operational even if one database server fails since we can access data stored in another database server

**Scenarios**

1. _Only one slave and it fails_: Redirect read requests to the master temporarily until the issue is fixed or the slave is replaced.

   ```text
   Normal state:
     [ Web Server ] --(writes)--> [ Master DB ] --(replicates)--> [ Slave DB ]
     [ Web Server ] --(reads)-----------------------------------> [ Slave DB ]

   When the single slave fails:
     [ Web Server ] --(writes)-------------------------> [ Master DB ]
     [ Web Server ] --(reads redirected temporarily)---> [ Master DB ]
                                                         [ Slave DB (DEAD) ] (X)
     * Master absorbs both reads and writes until a replacement slave is provisioned.
   ```

2. _Multiple slaves and one of them fails_: Redirect read requests meant for it to the remaining healthy slave(s) until a new server replaces the faulty slave.

   ```text
     [ Web Server ] --(writes)--> [ Master DB ]
                                       |
                     +-----------------+-----------------+
                     | (replicates)                      | (broken sync)
                     v                                   v
             [ Slave 1 (Healthy) ]               [ Slave 2 (DEAD) ] (X)
                     ^
                     | (reads re-routed here)
     [ Web Server ] -+
     * Traffic to dead slave is re-routed; healthy slave handles remaining read volume.
   ```

3. _Master database goes offline_: Promote a healthy slave to become the new master. Reconfigure remaining slaves to replicate from the new master, and provision a new slave to restore redundancy.
   _Note: This is harder in production as the data in the promoted slave might not be up to date due to replication lag! Data recovery scripts or consensus tools are required to reconcile differences._

   ```text
   Phase 1: Master crashes (Writes blocked)
     [ Web Server ] --(writes fail!)--> [ Master DB (DEAD) ] (X)
                                              |
                                       (replication halts)
                                              v
                                       [ Slave 1 ]   [ Slave 2 ]

   Phase 2: Promote Slave 1 & restore topology
     [ Web Server ] --(writes)--> [ Slave 1 (PROMOTED TO MASTER) ]
                                               |
                                +--------------+--------------+
                                | (replicates)                | (replicates)
                                v                             v
                        [ Slave 2 (Replica) ]        [ New Slave 3 (Added) ]
   ```

<a id="replication-and-failover-challenges-and-solutions"></a>
### Replication and Failover Challenges (and Solutions)

While master-slave replication is the standard approach for read-heavy workloads, operating it in real-world production brings two major challenges:

#### 1. Challenges while syncing writes to slaves (Replication Lag)

When the master database executes a write, how does it send that update to its slaves?

- **Synchronous replication**: The master waits for all slaves to confirm the write before responding "Success" to the user.
  - _Challenge:_ Very slow write latency (writes are only as fast as the slowest slave). If any slave is down or suffering network jitter, all writes across the entire system hang!
- **Asynchronous replication (Default in most DBs)**: The master commits the write locally, returns "Success" to the user immediately, and streams updates to slaves in the background.
  - _Challenge:_ **Replication Lag**. There is a small time delay (milliseconds to seconds) before slaves catch up.

##### What anomalies does Replication Lag cause?

1. **"Read-Your-Own-Writes" Inconsistency**:
   - A user updates their profile picture or posts a comment (written to the master).
   - The webpage immediately refreshes and reads data from a lagging slave.
   - The user sees their _old_ profile or missing comment and thinks the update failed!
   - **(Common solution) Read-Your-Own-Writes Routing:** Route a user's read requests directly to the **Master** for data they just modified (or for a few seconds following any write), while directing generic read traffic to slaves.
2. **Monotonic Read Inconsistency**:
   - A user refreshes a page twice. Request 1 hits a fast slave (shows 5 comments). Request 2 hits a slower slave (shows only 3 comments).
   - Data appears to disappear or travel backwards in time.

```text
Replication Lag Anomaly:
  1. [User] --(Write: name="Alice")--> [ Master DB ] (Updated instantly!)
                                             |
                                  (Replication Lag: ~500ms delay)
                                             v
  2. [User] <--(Read: name="Bob"??)--- [ Slave DB ] (Still old data!)
     *User confusion: "Wait, my update didn't save?!"*
```

##### More Common Solutions:

- **Semi-Synchronous Replication:** The master waits for at least _one_ slave to acknowledge the write before returning success. This prevents data loss if the master dies without waiting for every single replica.
- **Sticky Routing (Pinning a user to one replica):** Always send the same user's read requests to the _exact same_ replica (e.g., User 12 always reads from Replica A).
  - **Pros:**
    - **No "time-travel" bugs:** Because the user reads from just one server, their timeline always moves forward. They will never see data appear on one refresh and vanish on the next.
    - **Faster reads:** That replica likely already has that user's data stored in memory (cache).
  - **Cons (Why it makes systems harder to manage):**
    - **Uneven traffic:** If several power users or bots get pinned to Replica A, it can overheat while Replica B sits mostly idle.
    - **Breaks when a server dies:** If Replica A crashes, the user has to be moved to Replica B. If Replica B is still lagging, the user will see old data anyway!
    - **Harder to add servers:** Adding a new replica reshuffles who is assigned where, temporarily breaking the consistency.
    - **Fights statelessness:** In system design, our goal is to keep servers completely interchangeable ("stateless"). Tying a user to a specific machine takes that flexibility away.

##### Why sticky routing is a bad idea in general

Sticky routing breaks the #1 rule of horizontal scaling: **servers should be disposable and interchangeable ("stateless")**. Tying users to specific machines introduces operational pain:

1. **Traffic Hotspots**: If multiple high-activity users or bots get pinned to the same replica, that server gets overloaded while others sit idle.
2. **Fragile Failover**: If that replica crashes, the user gets shifted to another replica that may lag even further behind, re-exposing the stale data bug.
3. **Autoscaling Friction**: Adding or removing replicas changes routing hashes, disrupting user affinity and causing traffic reshuffling.

```text
Problem with Sticky Routing (Traffic Skew):
               [ Users / Clients ]
              /        |        \
             v         v         v   (Active users pinned to Replica A)
   [ Replica A (99% CPU - Overloaded!) ]        [ Replica B (2% CPU - Idle) ]
```

##### What to do instead (Better alternatives with diagrams)

1. **Selective Master Reads (Read-Your-Own-Writes Window)**:
   - Check a user timestamp (e.g., `last_updated_at` stored in a cookie or JWT).
   - Route reads to the **Master** only for a short grace period (e.g., 5 seconds) after a write.
   - Once the grace period expires, route reads back to any replica since replication has safely finished.

```text
Write (t = 0s):
  [ User ] --(Write: "New Post")--> [ Master DB ] (Set cookie: write_time = t0)

Read within 5 seconds (t = 2s):
  [ User ] --(Read own post)------> [ Master DB ] (Guaranteed fresh data!)

Read after 5 seconds (t = 30s):
  [ User ] --(Read own post)------> [ Any Replica DB ] (Safely caught up!)
```

2. **Transaction ID / Log Sequence Tracking (Causal Consistency)**:
   - When the user writes, the Master returns the latest log sequence number (e.g., Log ID `#4050`).
   - For subsequent reads, the router asks: _"Which replica has already reached at least Log ID #4050?"_
   - Any replica that is up to date can serve the read; lagging replicas are skipped.

```text
Write:
  [ Master DB ] (Commits write at Log #4050) -> Returns #4050 to Client

Subsequent Read:
  [ Client (requests data with Log >= #4050) ]
        |
        +---> [ Replica 1 (At Log #4050) ] ---> SUCCESS (Serves read)
        |
        X---> [ Replica 2 (At Log #4040) ] ---> SKIPPED (Still lagging)
```

3. **External Shared Cache (e.g., Redis)**:
   - Write new data to the Master database and simultaneously update an in-memory cache (like Redis).
   - Read requests fetch directly from Redis first. Reads are served in $<1\text{ ms}$ with fresh data, completely bypassing replication lag without needing sticky database connections.

```text
Write:
  [ Web Server ] ---> Writes to [ Master DB ] & Updates [ Redis Cache ]

Read:
  [ Web Server ] ---> Reads from [ Redis Cache ] (Sub-millisecond & always fresh!)
                        (No need to hit lagging replicas)
```

##### The Key Difference at a Glance: Read-Your-Own-Writes vs. Sticky Routing

| Feature                     | **Read-Your-Own-Writes Routing**                                 | **Sticky Routing (to a Replica)**                                          |
| :-------------------------- | :--------------------------------------------------------------- | :------------------------------------------------------------------------- |
| **Where reads are sent**    | Sent to the **Master DB** (for the user's recent modifications). | Sent to one specific **Slave Replica** (e.g., Replica A).                  |
| **Problem it solves**       | **"I just made a change, but can't see it!"** (Self-consistency) | **"I refreshed twice and data went backwards in time!"** (Monotonic reads) |
| **Guarantees latest data?** | **Yes** (the Master always has the latest writes).               | **No** (that replica might still be lagging behind the Master).            |
| **Main drawback**           | Adds extra read load to the Master.                              | Causes traffic hotspots and fights server statelessness.                   |

---

#### 2. Challenges while promoting a slave to replace a dead master (Failover)

When the master crashes, promoting a standby slave to become the new master sounds simple, but introduces severe operational risks:

##### The Key Challenges:

1. **Data Loss (Replication Gap)**:
   - If the master crashes while asynchronous writes are in-flight (committed on the master, but not yet replicated to any slave), those writes are lost when a slave is promoted.
2. **The "Split-Brain" Problem (Worst Case Scenario)**:
   - If a network partition cuts communication between the master and slaves, the slaves may assume the master is dead and promote a new master.
   - Meanwhile, the old master is actually still alive and continues accepting writes from clients on its side of the network!
   - Now you have **two masters accepting conflicting writes** simultaneously, creating severe data corruption that is difficult to reconcile.

```text
                   Network Partition (Split-Brain)
                                  |
        [ Old Master ]            |            [ Promoted Slave ]
    (Still accepts writes)        |        (Also accepts writes!)
              \                   |                   /
               v                  |                  v
        [ Inconsistent Data A ]   |        [ Inconsistent Data B ]
```

##### Common Solutions:

- **Consensus & Quorum Voting (e.g., Raft, Paxos, ZooKeeper, etcd):** Failover decisions are never made by an isolated node. A strict **majority vote (Quorum, $>50\%$)** is required to declare a master dead and elect a new one. If the partitioned old master cannot see a majority of nodes, it automatically demotes itself to read-only.
- **Fencing (STONITH - "Shoot The Other Node In The Head"):** Before promoting a slave, an automated cluster orchestrator cuts power or isolates the network port of the old master to guarantee it can never accept another write.
- **Promote the Most Up-to-Date Slave:** Cluster managers (e.g., GitHub's Orchestrator, Patroni) inspect the replication log position (e.g., MySQL Binlog or Postgres LSN) across all slaves and only promote the replica with the most advanced log.
- **Virtual IP (VIP) / Service Discovery:** Web servers connect to a virtual hostname or VIP (e.g., `db-master.internal`). Once failover is complete, the VIP or internal DNS record flips to point to the new master automatically.

<a id="what-if-the-system-is-write-heavy-instead-of-read-heavy"></a>
### What if the System is Write-Heavy Instead of Read-Heavy?

Master-slave replication **only scales read operations**. Because all writes must still hit the single master database, a write-heavy system will quickly saturate the master's CPU, disk I/O, and connection limits.

#### Solutions for write-heavy systems

1. **Database Sharding (Horizontal Partitioning)**:
   - Divide data across multiple independent database servers (shards).
   - Each shard acts as a master for a subset of the data (e.g., based on User ID hash).
   - Writes are distributed evenly across shards instead of overwhelming a single master.
2. **Buffer / Asynchronous Writes with a Message Queue**:
   - Place a message queue (e.g., Kafka, RabbitMQ) in front of the database.
   - Web servers push write requests into the queue and respond immediately to users.
   - Background worker processes pull messages from the queue and write them to the database in batches, smoothing out high-volume traffic bursts.
3. **Multi-Master (Master-Master) Replication**:
   - Multiple database servers act as masters and accept writes concurrently.
   - The masters synchronize changes among each other.
   - _Trade-off:_ Complex conflict resolution is needed if two masters update the same record at the same time.
4. **Write-Optimized Storage Engines (LSM-Trees / NoSQL)**:
   - Traditional relational databases use B-Trees, which must locate and update data in-place on disk (slow random disk I/O).
   - Write-heavy systems use specialized storage engines and NoSQL architectures to maximize write throughput:
     - **How LSM-Trees support faster writes:**
       - Writes are written directly to an in-memory buffer (MemTable) and sequentially appended to a write-ahead log (WAL) on disk. Sequential disk writes are up to 100x faster than random I/O because the disk head doesn't have to seek. Data is flushed to disk in sorted batches in the background.
     - **How NoSQL DBs support faster writes:**
       - **No heavy lock contention:** They drop foreign-key constraints, cross-table ACID locks, and schema checks that slow down relational databases.
       - **Peer-to-peer write distribution:** Distributed NoSQL databases (like Cassandra or DynamoDB) allow _any_ node to accept a write and quickly record it, completely removing the single-master write bottleneck.

#### Architecture diagram

```text
                           Write-Heavy Traffic
                                   |
        +--------------------------+--------------------------+
        |                                                     |
  [ Strategy 1: Sharding ]                           [ Strategy 2: Queue Buffer ]
        |                                                     |
        +---> [ Shard 1 (Users A-M) ]                         v
        |                                            [ Message Queue ] (e.g., Kafka)
        +---> [ Shard 2 (Users N-Z) ]                         |
                                                              v
                                                     [ Batch Worker ]
                                                              |
                                                              v
                                                       [ Database ]
```

#### Common scenarios

1. _Sudden bursts of write traffic (e.g., flash sales, ticketing events)_:
   - **Solution:** Use a **message queue buffer**. Writes are queued in memory instantly, preventing the database from locking up under peak load.
2. _Continuous massive write volume exceeding single-server disk I/O (e.g., IoT telemetry, chat messages)_:
   - **Solution:** Use **database sharding** or a **write-optimized distributed NoSQL database** (like Cassandra) to distribute writes across many nodes.
3. _Writes originating from multiple geographic regions with low latency_:
   - **Solution:** Use **multi-master replication** with regional masters, resolving conflicts via rules like "Last-Write-Wins" (LWW) or assigning distinct key ranges per region.

<a id="improve-response-times-with-cache-layers"></a>
## ⚡ Improve Response Times with Cache Layers

> [!TIP]
> **The 80/20 Rule in Caching:** 80% of read requests typically target 20% of your data (the "hot set"). Sizing your in-memory cache to comfortably hold that 20% hot set will satisfy the overwhelming majority of incoming traffic.

Every time a user visits your app, fetching data from a database involves disk reads, complex queries, and network round-trips. When thousands of users request the **exact same data repeatedly** (such as trending homepage posts, a viral product, or user profile details), querying the database every single time is slow, expensive, and wasteful. It quickly exhausts database connections and disk I/O, causing the entire site to slow to a crawl.

Think of it with an everyday analogy: instead of walking all the way down to a basement storage room every time you need salt while cooking, you keep a small container right on your kitchen counter. That "kitchen counter" is a **cache**—an ultra-fast in-memory layer that serves frequently requested data in a fraction of a millisecond ($<1\text{ ms}$), shielding your primary database from redundant work and keeping your application lightning-fast.

**What is a cache?**

Temporary storage area **_in memory_** that stores either:

1. The result of expensive responses, or
2. Frequently accessed data

**Why is a cache better?**

Memory access is _faster_ than accessing secondary storage or data from a database. Hence, it is better for performance as it reduces the response time

<a id="database-cache-tier"></a>
### Database Cache Tier

Database data access is slower than reading data from a data cache. Hence:

1. Keep a separate data cache tier i.e a separate server that sits in between the web and the database servers
2. Maintain a caching strategy to fetch and update data (Ex: **_Read-through_** strategy is quite popular)

_Note: Most cache servers provide APIs to use in multiple languages making integration easier_

**When to use a cache?**
**_Use a cache when data read is more frequent than data writes!_**
Why? Cache is memory which is "_volatile_" which in turn means that if the cache server restarts, the data in memory is lost! Therefore, data should be written to a database (persistent storage) and not a cache.

![data cache tier](https://i.imgur.com/UG96SC0.png)

<a id="static-content-cache-with-cdns"></a>
### Static Content Cache with CDNs

**What is a CDN?**
_Content Delivery Networks_ is a network of _geographically dispersed servers_ used to _deliver static content_.
Static content can be images, videos, CSS, JavaScript files, etc...

**How does it work?**
When a user visits a website:

- The website will have the CDN url, instead of the origin URL, listed for a resource
  - Ex: `somecdndomain.com/path/to/image` instead of `mysite.com/path/to/image`
- The CDN server closest to the user geographically will be hit with a request to the resource
- The further the CDN server is from the user geographically, the slower the resource loads!
- Handling cache misses:
  - CDN server requests the file from the origin which is either the web server or a storage like Amazon S3
  - HTTP header Time-To-Live (TTL) descries how long the resource is cached

_CDN for dynamic content is a new concept and a lot more difficult to make work (Ex: Storing entire HTML pages)_

![cdn](https://i.imgur.com/UNpvhZR.png)

<a id="can-a-cdn-be-a-single-point-of-failure-spof"></a>
#### Can a CDN Be a Single Point of Failure (SPOF)?

**Yes.** While an individual local edge server failing is handled automatically by routing traffic to the next nearest data center, **an entire CDN provider can experience a global outage**. Major incidents (such as global BGP routing misconfigurations, bad software deployments, or expired security certificates at providers like Cloudflare or Fastly) have famously taken down dozens of major platforms simultaneously.

If an application depends 100% on a single CDN vendor for its JavaScript, CSS, and media assets, that vendor becomes a Single Point of Failure.

#### How is this handled? (Or do we just let the vendor handle it?)

In production, how you handle this depends on company size, budget, and SLA requirements:

1. **Let the Vendor Handle Internal Redundancy (Standard for 95% of applications)**:
   - For most companies, building backup CDN infrastructure is overly complex and expensive.
   - Major CDN providers (Cloudflare, AWS CloudFront, Akamai) already operate hundreds of Points of Presence (PoPs).
   - If one edge server or data center fails, **Anycast BGP routing** instantly shifts requests to the next closest healthy data center with zero downtime.
   - For standard businesses, trusting the vendor's 99.99% uptime SLA is the most cost-effective path.

2. **Client-Side Origin Fallback (Graceful Degradation)**:
   - If a CDN asset fails to load, client-side code can catch the error and fetch the asset directly from the origin server (e.g., your Amazon S3 bucket or web server):
     ```html
     <!-- If the CDN script fails to load, fallback to the origin server directly -->
     <script
       src="https://cdn.example.com/app.js"
       onerror="this.onerror=null; this.src='https://origin.example.com/app.js';"
     ></script>
     ```
   - _Trade-off:_ The origin server experiences higher load and latency during the outage, but the site remains fully operational.

3. **Multi-CDN Strategy (For High-Scale / Mission-Critical Systems)**:
   - Giant enterprises (e.g., Netflix, Meta, high-frequency e-commerce) never rely on a single CDN vendor.
   - They use a **Multi-CDN architecture** combining two independent vendors (e.g., Cloudflare + Akamai or CloudFront + Fastly).
   - A smart DNS traffic manager (e.g., AWS Route 53, NS1) monitors CDN health in real time. If Vendor A suffers an outage, DNS automatically routes 100% of user traffic to Vendor B.

```text
                                [ User ]
                                   |
                     [ Smart DNS / Traffic Router ]
                                   |
         +-------------------------+-------------------------+
         | (Normal: Active)                                  | (Standby / Secondary)
         v                                                   v
   [ CDN Vendor A (e.g. Cloudflare) ]               [ CDN Vendor B (e.g. Akamai) ]
         | (If Vendor A crashes)                             | (Takes over traffic)
         X                                                   v
                                                      [ Origin Server ]
```

<a id="cache-considerations-for-both-data-and-static-assets"></a>
### Cache Considerations for Both Data and Static Assets

> [!NOTE]
> **Cache Invalidation:** Always define sensible Time-To-Live (TTL) values so stale data automatically purges. Pair TTL with an LRU (Least Recently Used) eviction policy to prevent memory exhaustion.

1. **Expiration policy**: Not too long nor too short!
   - Too short: Repeat reloading of content from database or origin server becomes a concern
   - Too long: Data becomes stales i.e not up to date
2. **Eviction policy**: Once the cache is full, we need to replace some of the data
   - **_Least Recently Used (LRU)_** policy is the most common
3. **Consistency**: Keep the data in cache and database or server in sync! When scaling across multiple regions, this becomes challenging
4. **Mitigating failures**: A single cache server can become a **_Single Point Of Failure (SPOF)_** . Solution: Multiple cache servers across different data centers.
5. **Invalidating files**: _This pertains only to CDNs_. If a file needs to be removed before expiry, we need to "_invalidate_" it. Two options:
   1. Use a CDN API to invalidate a resource
   2. Use object versioning to get the latest version of a resource. Ex: API versioning `/v2/...`
6. **Cost**: _This pertains only to CDNs_. CDNs are usually 3rd party vendors (Ex: Cloudfront, Akamai, Cloudflare, etc) and caching things unnecessarily or replacing it unnecessarily can prove to be costly! (Check the point for expiration policy)
7. **CDN fallback**: _This pertains only to CDNs_. If there is a temporary CDN outage, clients must be able to detect it and request a resource from the origin.

<a id="managing-session-state-in-horizontally-scaled-web-servers"></a>
## 👤 Managing Session State in Horizontally Scaled Web Servers

> [!TIP]
> **The Golden Rule of Web Tiers:** Keep web servers purely stateless. When web servers have zero local session state, scaling up/down or replacing crashed nodes is completely seamless.

<a id="what-happens-if-session-data-is-not-separated"></a>
### What Happens if Session Data is Not Separated?

**What is session state?**
Session state is temporary data about an active user session—such as whether a user is logged in, their account ID, or the items currently sitting in their e-commerce shopping cart.

On a simple **single-server setup**, managing session state is trivial: the server simply saves the session in its own local memory (RAM). Since every request hits that same machine, the server always remembers the user.

However, once you **scale horizontally** and place multiple web servers behind a load balancer, keeping session data in server memory causes major problems:

1. **The "Who are you?" Problem (Random Logouts)**:
   - **Request 1 (Login):** The load balancer directs the user to **Server 1**. Server 1 authenticates the user and saves their session in its own local RAM.
   - **Request 2 (Next click / Add to cart):** The load balancer sends this next request to **Server 2**.
   - **The disaster:** Server 2's RAM has no knowledge of what happened on Server 1! To Server 2, this is an unauthenticated stranger. The user is suddenly prompted: _"Please log in again"_, and their shopping cart appears empty.

```text
The Stateful Problem (Local Server Memory):

Request 1 (Login):
  [ User ] ---> [ Load Balancer ] ---> [ Server 1 ] (Saves in local RAM: "User 42 logged in")

Request 2 (Next click):
  [ User ] ---> [ Load Balancer ] ---> [ Server 2 ] (Checks local RAM: Empty!)
                                             |
                                             v
                                  "Who are you? Please log in again!"
```

2. **The Flawed Patch: "Sticky Sessions" (Session Affinity)**:
   - A common temporary fix is forcing the load balancer to always route the same user to the same server ("sticky sessions").
   - **Why this is a bad idea in practice:**
     - **No Failover:** If Server 1 crashes or restarts for a software update, all sessions in its RAM vanish instantly. Every user pinned to Server 1 gets logged out and loses their in-progress work.
     - **Traffic Imbalance:** A few heavy users pinned to Server 1 can overload it while Server 2 and Server 3 sit idle.
     - **Autoscaling Breakdown:** You cannot easily add or remove servers on the fly when users are tethered to specific machines.

---

<a id="the-recommended-approach-stateless-web-tier-with-shared-storage"></a>
### The Recommended Approach: Stateless Web Tier with Shared Storage

To scale horizontally without friction, we must **decouple the state from the web servers**.

Instead of storing session data trapped inside individual server RAM, we move session state into a **centralized, shared data store** (typically a fast in-memory cache like **Redis** / **Memcached**, or a NoSQL database).

**How it works (in 2 simple stages):**

#### Stage 1: When the user logs in (One-time check)

1. The user sends their email and password.
2. The web server checks the **Permanent Database** to verify the password.
3. Once verified, the server creates an active session badge (token) and saves it into **Redis** (in-memory, with an auto-expiry timer like 24h).
4. The server hands this badge to the user's browser as a cookie (`session_id`).

```text
Stage 1: Login (Only time the Database is checked)

[ User ] --(1. email + password)--> [ Web Server 1 ]
                                           |
                   +-----------------------+-----------------------+
                   | (2. Check password)                           | (3. Save active badge)
                   v                                               v
        [ Permanent Database ]                          [ Shared Memory: Redis ]
        (Has users & hashed passwords)                  (Key: "sess_123" -> User 42)
                   |                                               |
                   v                                               |
        "Password is correct!"                                     |
                   |                                               |
                   +-----------------------+-----------------------+
                                           |
                                           v
[ User ] <-------(4. Return session cookie: "sess_123")------------+
```

#### Stage 2: Every click after logging in (Fast repeated checks)

1. On every subsequent click (e.g. view profile, add to cart), the browser sends its `session_id` cookie.
2. The load balancer can send the request to **any web server** (e.g. Server 2).
3. Server 2 asks **Redis**: _"Who owns badge `sess_123`?"_
4. Redis answers instantly ($<1\text{ ms}$). The **database is never touched**, keeping it free from heavy routine traffic!

```text
Stage 2: Any Subsequent Click (Database is completely bypassed!)

[ User ] --(1. Click with cookie: "sess_123")--> [ Web Server 2 ] (Any available server)
                                                        |
                                                        | (2. Instant memory lookup <1ms)
                                                        v
                                             [ Shared Memory: Redis ]
                                             ("sess_123" = User 42)
                                                        |
                                                        v
                                             "Verified! User 42 is logged in."
                                             (Notice: Database is NOT touched!)
```

#### What each store holds:

- **Database (Permanent storage on disk):**
  Stores permanent account records.
  | `user_id` | `email` | `password_hash` |
  | :--- | :--- | :--- |
  | `42` | `alice@example.com` | `$2b$12$...` |

- **Redis (Temporary cache in memory):**
  Stores active login sessions with an automatic countdown timer.
  | Key | Value | Time-To-Live (TTL) |
  | :--- | :--- | :--- |
  | `session:sess_123` | `{ "user_id": 42, "role": "admin" }` | Expires in 24 hours |

**Key Benefits of a Stateless Web Tier:**

- **Effortless Autoscaling:** Web servers become completely interchangeable ("stateless"). You can add 50 new servers during a traffic spike and shut them down afterwards without disrupting a single active user.
- **Resilient Failover:** If Server 1 crashes, the load balancer reroutes the next request to Server 2. Server 2 simply fetches the user's session from the shared store—the user never notices a thing.
- **Perfect Load Distribution:** The load balancer distributes traffic evenly across all servers without worrying about who is logged in where.

![Shared session state DB](https://i.imgur.com/j527SzM.png)

<a id="can-the-shared-session-storage-become-an-spof"></a>
#### Can the Shared Session Storage Become an SPOF?

**Yes!** If you move all user sessions into a single Redis or database instance, and that server crashes or runs out of RAM, **every single user gets kicked out at once and no one can log in**. You eliminated the web server bottleneck, but shifted the Single Point of Failure to the shared session storage!

#### How is this handled in production?

To prevent the shared session store from becoming an SPOF, production systems use three standard strategies:

##### 1. Primary-Replica with Automated Failover (e.g., Redis Sentinel / AWS ElastiCache)

- **Why use it:** Guarantees high availability. If the primary session server crashes, a standby replica is promoted to become the new primary in seconds.
- **How it works:** A cluster monitor (like Redis Sentinel or a managed cloud service) continuously pings the primary node. If the primary fails to respond, it automatically flips traffic over to the replica with zero manual intervention.

```text
               [ Web Servers (Any node) ]
                           |
                           v
        [ Primary Redis (Handles read/write) ]
                           |
                   (Async replication)
                           v
        [ Standby Replica (Backup in Sync) ]
                           ^
                           |
             [ Sentinel / Health Monitor ]
             (Auto-promotes Replica if Primary dies)
```

##### 2. Sharding & Clustering (e.g., Redis Cluster)

- **Why use it:** Handles massive user scale and limits the "blast radius" of any single machine failure.
- **How it works:** Instead of storing all sessions on one machine, sessions are split across multiple independent master shards based on `hash(session_id)`. Each master shard has its own replica.
- **Benefit:** If one shard crashes, only a small fraction of users are affected while its replica takes over, leaving all other shards completely unaffected.

```text
[ User A (sess_01) ] ---> [ Shard 1 (Master) ] ---> [ Replica 1 ]
[ User B (sess_02) ] ---> [ Shard 2 (Master) ] ---> [ Replica 2 ]
[ User C (sess_03) ] ---> [ Shard 3 (Master) ] ---> [ Replica 3 ]

* If Shard 1 fails, only Replica 1 promotes; Shards 2 & 3 run unaffected!
```

##### 3. Stateless Tokens: JSON Web Tokens (JWT)

- **Why use it:** Bypasses the session database entirely for routine page requests.
- **How it works:** Instead of storing user data in a database, session info is cryptographically signed into a **JWT** and stored in the user's browser cookie. Any web server can verify the signature mathematically using a secret key—requiring zero database lookups!

###### How a JWT bypasses Redis completely (The "Digital Passport" analogy)

Think of a standard Redis session like a **Coat-Check Ticket**: the ticket only has a random number (`#402`), so the attendant must walk back to the storage room (Redis) to look up which coat is yours.

A **JWT** is like a **Government Passport with an official holographic seal**:

1. All your details (user ID, role, expiration timestamp) are written directly onto the document itself.
2. The server stamps it with a **cryptographic digital signature** using a private secret key.
3. When you present your passport to _any_ web server, the server doesn't need to call a central database. It simply runs a quick mathematical verification on the signature using the secret key. If the signature is valid and the date hasn't expired, the server trusts the user immediately!

```text
Session ID (Redis lookup needed on every request):
  [ Browser ] --("sess_xyz123")--> [ Server ] ---> Checks [ Redis Store ] ---> "User 42"

JWT (Zero database or cache lookups):
  [ Browser ] --(Signed Token)--> [ Server ] ---> Runs local math: verify(token, SECRET)
                                                     |
                                                     v
                                      "Signature valid! You are User 42."
                                      (Zero network calls, 0ms lookup latency!)
```

###### Why is instant revocation so difficult?

Because JWTs are **self-contained and stateless**, the server has no central database row to delete!

**Real-world scenario:**

1. At 1:00 PM, Alice logs in. She receives a JWT valid for 4 hours (expires at 5:00 PM).
2. At 2:00 PM, Alice's laptop is stolen. She panics and clicks **"Log out from all devices"** on her phone.
3. **The problem:** Web servers do not check a database—they only check if the token's signature and expiration time are valid. Because the thief's token still has a valid signature and doesn't expire until 5:00 PM, **the thief can continue using Alice's account for the next 3 hours!**
4. With a Redis session store, you would simply delete Alice's session key from Redis, locking the thief out in milliseconds. With pure JWTs, you cannot "un-sign" a token that is already out in the wild.

###### Decision Matrix: Redis Sessions vs. JWTs

| Requirement         | **Server-Side Sessions (Redis)**                                                         | **Stateless JWTs**                                                             |
| :------------------ | :--------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------- |
| **Lookup Speed**    | Ultra-fast ($<1\text{ ms}$ over network).                                                | Instant ($0\text{ ms}$ network, pure CPU math).                                |
| **Server Overhead** | Requires managing a Redis cluster + RAM cost.                                            | Zero storage cost; verified using server CPU.                                  |
| **Revoking Access** | **Trivial & instant** (just delete the session key).                                     | **Very difficult** (valid until token timer expires).                          |
| **Payload Size**    | Small (tiny cookie ID, $\approx 32\text{ bytes}$).                                       | Larger (carries payload + signature, $\approx 500\text{ bytes}$).              |
| **Best Used For**   | Traditional web apps, banking, SaaS dashboards where instant logout/banning is critical. | Microservices, mobile apps, single-sign-on (SSO), high-throughput public APIs. |

###### When do we need both? (The Industry-Standard Hybrid Pattern)

To get the blistering speed of JWTs _and_ the security of instant revocation, production systems combine both using **Access Tokens + Refresh Tokens**:

1. **Access Token (Short-lived JWT, e.g. 10–15 minutes):** Sent on every regular API request. Verified locally by web servers with zero database calls.
2. **Refresh Token (Long-lived, e.g. 30 days, stored in Redis/DB):** Stored securely. When the 15-minute JWT expires, the user's browser uses the Refresh Token to request a new 15-minute JWT.

```text
The Hybrid Pattern in Action:

Normal Requests (99% of traffic - ZERO DB hits):
  [ Browser ] --(15-min JWT)--> [ Web Server ] ---> Locally verified in 0ms!

Every 15 minutes (Token renewal - checks Redis):
  [ Browser ] --(Refresh Token)--> [ Auth Server ] ---> Checks [ Redis Store ]
                                                                   |
                                                      (If NOT revoked -> Issues new 15-min JWT)
                                                      (If revoked/stolen -> BLOCKS user!)
```

**Why this is the best of both worlds:**

- **99% of user requests** fly through with $0\text{ ms}$ database latency using stateless JWT verification.
- If a device is stolen, you immediately revoke the Refresh Token in Redis. Within at most 15 minutes, the thief is permanently locked out!

<a id="complete-set-of-optimizations-for-one-single-data-center"></a>
## 🏢 Complete Set of Optimizations for One Single Data Center

**Start**: One single web server
**Optimizations**:

1. When user start to scale i.e more users:
   - Separate out the web server from the database server for independent scaling
2. More users but also more simultaneous website access:
   - Need to horizontally scale the web servers. Use a load balancer to directing traffic to multiple servers
3. More users means more data is being read than written (traditionally, this is how it is when scaling)
   - Example: Multiple users are reading, liking, commenting, etc... on a user's post
   - Scale the database to handle more read operations in parallel. Use database replication i.e 1 master DB for writes (creates, updates, deletes) and its DB replicated in many slaves that receive and service the read requests
4. Improve the database data request response times
   - Use a data tier cache that stores some of the data in-memory. Use an expiration policy as well as an eviction policy (Ex: LRU cache)
5. Improve the static resources' request response times
   - Use a CDN and take advantage of its geographic proximity to a user. Use an expiration & eviction policy
6. Manage state for horizontally scaled web servers
   - Use a separate shared database for managing only the state (Ex: user session data). Both web server and state database(s) can be scaled independently and the web servers are easy to auto-scale!

![Single data center complete diagram](https://i.imgur.com/8pQsviO.png)

<a id="scaling-for-users-across-multiple-geographical-areas"></a>
## 🌐 Scaling for Users Across Multiple Geographical Areas

> [!NOTE]
> **GeoDNS vs. Anycast Routing:** GeoDNS resolves domain names to regional IP addresses based on the user's location, while BGP Anycast routes the same public IP address to the nearest physical data center at the network routing layer.

To provide users across multiple geographical areas a better experience, i.e performance in terms of response time, we need to support the maintenance of **multiple data centers**.

We have already scaled the database previously to optimize for reads i.e having database replication using a master-slave relationship. However, this does not solve everything. The problems are:

1. There is only one master i.e Single Point of Failure (SPOF) for the writes. However, a failing master can be replaced by promoting a slave. So this problem is solved. But then, when the writes also increase to the extent that it causes whichever master DB to reach its load limit, then we hit a road-block!
2. When users are accessing the application from multiple states, countries, or continents, it does not matter if you increase the number of web servers or databases at one particular geographic location since the response time cannot improve beyond a threshold due to geographic (physical) constraints

**Benefits of multiple data centers**

1. Each data center will contain at least:
   - Horizontally scaled set of web servers
   - Database replications, and a
   - Data tier cache
2. We use a **geoDNS** service: _It is a service that allows domain names to be resolved to IP addresses based on the location of a user_
3. We can also _split the traffic_ via geoDNS. Example: `x%` to US East and `(100-x)%` to US West
4. In case of a data center failure, the traffic for it will be re-routed to another healthy data center (while the issue is resolved). Hence, there is _failover support_

_(Note: The shared session state database service will be better kept outside any of the data centers since all of them need to access it)_

**Challenges with multiple data centers**

1. In failover cases: When redirecting traffic to another center, we need to make sure it has the data from the data center that failed (i.e data sync). Common strategy: Replicate data across multiple data centers!
2. Test and deployment: Need to have automation to test, deploy, etc. at different locations
3. Traffic redirection: Need effective tools such as a geoDNS

![Multiple data centers](https://i.imgur.com/CuBnKgX.png)

<a id="decoupling-tiers-with-message-queues"></a>
## 📬 Decoupling Tiers with Message Queues

> [!TIP]
> **Asynchronous Resilience:** Message queues prevent cascading failures by buffering bursts of traffic. If downstream workers go down, tasks remain safely queued rather than dropping user requests.

Imagine a scenario where a user uploads a photo and it needs to be customized. For example, we need to crop, add a filter, compress it, and store multiple copies of different dimensions for different viewports. To do all this **_takes time_**. Another similar example includes a user action that triggers the sending of emails by our server.

- _A single server handles everything_: If we expect a web server receiving the photo to do this all this by itself then that is a lot of load and it cannot remain idle to handle other incoming requests! i.e poor response times or worse, failing to respond completely
- _A server offloads the work to a dedicated worker server_: This is better but still has bottlenecks! The operation is _synchronous_ and if the dedicated server, known as a **_worker_**, becomes unavailable, i.e goes offline, the operation fails and the data is lost too!

We need a **_buffer_**! This buffer is known as a **Message queue** service.

The basic architecture of a _Message Queue_ is that of a **producer-consumer**:

1. Web servers produce content (Ex: photo) by publishing to the queue
2. Workers consume the content (Ex: photo processing) by connecting to the queue

**Advantages of message queues**

1. It is a durable component, stored in memory, and distributes asynchronous requests
2. It supports _asynchronous_ operations:
   - A producer can publish to the queue even when the consumer is unavailable
   - A consumer can pick up tasks from the queue once it becomes available (even if the producer is unavailable)
3. The producer and the consumer can be scaled independently (Ex: More workers to process more photos)

![Message queue](https://i.imgur.com/PQPR5tQ.png)

<a id="can-a-message-queue-be-a-single-point-of-failure-spof"></a>
### Can a Message Queue Be a Single Point of Failure (SPOF)?

**Yes!** If you deploy only a single message broker instance (e.g. one standalone RabbitMQ or Kafka server) and it crashes or runs out of disk space:

- **Producers cannot send work:** Web servers attempting to publish messages will hang or fail.
- **Workers have nothing to do:** All background jobs (processing photos, sending receipts, charging accounts) stall immediately.
- **Data loss:** If messages were stored only in the broker's RAM, an abrupt crash or power cut wipes out all pending messages permanently!

#### How is a Message Queue SPOF prevented in production?

Production architectures ensure high availability for message queues using three standard techniques:

##### 1. Broker Clustering with Leader-Follower Replication (e.g. Kafka / RabbitMQ Quorum Queues)

- **How it works:** Run a cluster of multiple broker servers (typically 3 or 5 nodes).
- **Replication:** When a message is sent, the **Leader Broker** receives it and immediately replicates it to **Follower Brokers**.
- **Automated Failover:** If the Leader broker dies, the cluster consensus coordinator (e.g., Raft or KRaft) instantly promotes a synchronized Follower to be the new Leader. Producers and consumers reconnect to the new leader in seconds with zero message loss.

```text
Message Queue High Availability (Leader-Follower Cluster):

[ Producers (Web Servers) ]
             |
             v (Publishes task)
   [ Leader Broker (Node 1) ]
       |                  |
       | (Replicates)     | (Replicates)
       v                  v
 [ Follower (Node 2) ]  [ Follower (Node 3) ]
       |
       +--(If Node 1 dies, Node 2 is auto-elected NEW LEADER!)
             |
             v (Workers continue pulling tasks seamlessly)
[ Consumers (Workers) ]
```

##### 2. Disk Persistence (Write-Ahead Log / Durability)

- Messages are not just held in volatile RAM. They are appended sequentially to a persistent log on disk (e.g., disk commit log) before acknowledging the producer. If a broker machine restarts, it reloads all unacknowledged messages safely from disk.

##### 3. Cloud-Managed Multi-AZ Queues (e.g., AWS SQS / GCP Pub/Sub)

- Fully managed cloud queues remove the headache of running broker clusters yourself. Under the hood, **AWS SQS** automatically replicates every message across **three physically separate data centers (Availability Zones)**, guaranteeing 99.999999999% message durability with zero manual server maintenance.

---

<a id="essential-message-queue-concepts-you-must-know"></a>
### Essential Message Queue Concepts You Must Know

#### 1. Point-to-Point (Queue) vs. Publish-Subscribe (Pub/Sub)

- **Point-to-Point Queue (1-to-1):** Each message is pulled and processed by **only one worker**. If multiple workers listen to the queue, they compete for tasks to share the load (e.g. RabbitMQ work queue, AWS SQS).
- **Publish-Subscribe / Pub/Sub (1-to-Many):** Each message is broadcast to **every subscribed service**. Perfect when a single event needs to trigger actions across multiple distinct teams (e.g. Apache Kafka, AWS SNS).

```text
Point-to-Point (Work Queue - Exactly one worker processes each item):
  [ Task: Process Photo #101 ] ---> [ Queue ] ---> [ Worker 1 ] (Worker 2 does NOT see it)
                                                    [ Worker 2 ] (Free for next task)

Publish-Subscribe (Pub/Sub - Broadcast to all interested subscribers):
                                              +---> [ Email Service ] (Sends order email)
  [ Event: "Order Placed" ] ---> [ Topic ] ---+---> [ Inventory Service ] (Deducts stock)
                                              +---> [ Analytics Service ] (Logs metrics)
```

#### 2. The "Shock Absorber" Pattern (Traffic Spike Smoothing)

A message queue acts like an elastic buffer between fast producers and slower workers:

- **Without a queue:** 10,000 users upload photos simultaneously during a marketing sale $\rightarrow$ 10,000 heavy processing tasks hit workers at once $\rightarrow$ **Workers run out of CPU/RAM and crash**.
- **With a queue:** 10,000 tasks are safely deposited into the queue in milliseconds $\rightarrow$ **Workers pull tasks comfortably at a steady, sustainable pace (e.g. 50 tasks/second)** until the queue is empty. No crashes, no lost work!

```text
Traffic Spike without Queue:
  [ 10,000 Requests/sec ] =====> [ Workers ] ===> CRASH! (Out of memory & CPU)

Traffic Spike with Queue ("Shock Absorber"):
  [ 10,000 Requests/sec ] =====> [ Message Queue Buffer ]
                                           |
                                           | (Workers pull steadily at 50/sec)
                                           v
                                   [ Workers (Healthy) ]
```

#### 3. Delivery Guarantees & Idempotency

- **At-least-once delivery (Standard default):** The queue guarantees no message will ever be lost. However, if a worker processes a message but the network drops before it can send an acknowledgment ("ACK"), the queue redelivers the message to another worker.
- **Why Idempotency is required:** Because workers can occasionally receive a message twice, your worker logic **must be idempotent** (processing the same message twice produces the exact same result—e.g., checking a database transaction ID to avoid charging a customer's card twice).

#### 4. Dead Letter Queues (DLQ)

- What happens if a corrupted, invalid message (a "poison pill") crashes a worker every time it tries to parse it?
- Instead of retrying forever and repeatedly crashing workers, the queue counts failed retries (e.g., 3 attempts) and automatically reroutes the corrupted message to a **Dead Letter Queue (DLQ)**. This isolates the bad message for developer inspection while healthy traffic keeps flowing without interruption!

<a id="scaling-databases-beyond-master-slave-replication-with-sharding"></a>
## 🗄️ Scaling Databases Beyond Master-Slave Replication with Sharding

<a id="why-master-slave-replication-falls-short-as-traffic-grows"></a>
### Why Master-Slave Replication Falls Short as Traffic Grows

Up to this point, our primary tool for scaling the database tier has been **Master-Slave Replication**.

As covered earlier, master-slave replication works well **primarily for read-heavy workloads**:

- Adding more slave replicas increases **read throughput** linearly.
- However, **all write operations must still hit the single Master database**.

For read-heavy systems, we used Master-Slave replication logic to scale our database.

For a **Write-Heavy** system, can we still use Master-Slave?

- _Answer_: **NO!** The single master becomes an SPOF! We need to scale the master!
- _How can we scale the master too?_: We use a concept called **"Database Sharding"**
  - It is a horizontal partitioning of a database i.e splitting of the rows of table

#### Do we still need sharding if our system is still read-heavy?

A common question is: _"If our system traffic is still read-heavy with 90% reads and only 10% writes, why would master-slave replication ever fail?"_

The answer is: **Percentages do not matter when absolute traffic becomes massive.**

Consider this real-world scale scenario:

- Total traffic: **100,000 queries per second**.
- Reads (90%): **90,000 reads/sec** $\rightarrow$ Handled comfortably by adding 15–20 slave replicas.
- Writes (10%): **10,000 writes/sec** $\rightarrow$ **All 10,000 writes must hit the single Master!**

A single database server has hard physical hardware limits (disk write I/O, CPU, and transaction lock contention). A standard relational database server typically caps out at a few thousand writes per second. Once write volume passes that hardware ceiling:

1. Adding more slave replicas does **nothing** to help the master process writes.
2. In fact, having 20 slaves makes the master's job _harder_, because the master must spend CPU and network bandwidth broadcasting replication logs to all 20 replicas!

```text
The Single Master Write Bottleneck (Even in Read-Heavy Systems):

Total Traffic: 100,000 queries/sec (90% Reads, 10% Writes)

                     [ Web / Application Tier ]
                      /                      \
    10,000 Writes/sec |                      | 90,000 Reads/sec
                      v                      v
             [ MASTER DATABASE ]    [ 15-20 Slave Replicas ]
             (CRASH / OVERLOAD!)    (Handles reads comfortably)
             * Hits disk I/O cap!
             * More slaves cannot help writes.
```

#### The Solution: Database Sharding

To break through the single-master write ceiling, we must partition the data across multiple independent database servers. This is called **Database Sharding**.

- Sharding distributes **both writes and reads** across multiple database servers.
- In production, systems typically combine **Sharding + Replication**: each shard has its own Master (handling writes for that subset of data) and its own Slaves (handling reads for that subset of data).

```text
The Production Setup (Sharding Combined with Master-Slave Replicas):

                         [ Web Tier ]
                         /          \
            (Users 1 - 500k)      (Users 500k - 1M)
                   |                      |
                   v                      v
             [ Shard 1 ]            [ Shard 2 ]
             /         \            /         \
            v           v          v           v
       [ Master 1 ] [ Slave 1 ] [ Master 2 ] [ Slave 2 ]
       (Writes 1)   (Reads 1)   (Writes 2)   (Reads 2)

* Result: Write traffic is split across Master 1 and Master 2!
```

---

When a database **_(within a single data center)_** gets overloaded or approaches its capacity, it is time to scale beyond a single master-slave cluster.

There are two options to scale DBs:

1. Vertical scaling: _Not recommended_ due to the following reasons:
   1. Hardware limits on a single DB server (no unlimited CPU, RAM, etc access)
   2. Risk of SPOF
   3. Overall cost is higher (high capability systems cost a lot!)
2. Horizontal scaling: Known as **"Sharding"** (Recommended)

<a id="what-is-database-sharding"></a>
### What is Database Sharding?

- It is the practice of adding more database servers.
- It is not enough to just add more servers but we need to separate the database itself into smaller, more easily managed parts called **_"shards"_** .
- Each shard shares the **_same schema_** but the actual data in each shard is unique to the shard

<a id="how-sharding-works"></a>
### How Sharding Works

> [!TIP]
> **Picking a Sharding Key:** A good shard key distributes data evenly and keeps related queries on a single shard. Avoid low-cardinality keys (like gender or country) that create severe data skew.

1. We need a **sharding key**. For example, a user id can be a sharding key. Usually, a _column_ or a _set of columns_ is taken to be the key
2. A hash function takes in the sharding key and outputs a selection of one of the shards

It is important to find a **_sharding key that evenly distributes data_** to avoid exhaustion of one database much earlier than the others and to keep the access loads evenly distributed.

![Database sharding](https://i.imgur.com/E7tbWhj.png)

<a id="benefits-and-challenges-of-sharding"></a>
### Benefits and Challenges of Sharding

#### Benefits of sharding

1. Great technique to scale databases by routing read and write queries efficiently to the correct database

#### Challenges of sharding

It introduces complexities and new challenges:

1. **Resharding data**: When we exhaust the data possible in a shard, we need to recompute the hash function and moving data around. This is solved by the use of a **consistent hashing** technique
2. **Celebrity problem (Hotspot Key problem)**:
   - **The Issue:** Excessive access to a specific key causes a single shard to be overwhelmed while others sit mostly idle. If users like Justin Bieber or Taylor Swift have millions of followers, and queries are routed by `user_id`, all reads and writes for that celebrity slam into one server, causing 100% CPU spikes and crashes.
   - **Common Solutions:**
     - **Aggressive Caching:** Serve celebrity profiles directly from in-memory caches (Redis/Memcached) or edge CDNs, shielding the database completely.
     - **Dedicated Shard / Key Salting:** Allocate dedicated shards for celebrity accounts, or append a random salt/suffix (e.g., `user_101_0`, `user_101_1`, `...`, `user_101_N`) to distribute their posts/followers across multiple shards.

   ```text
   The Problem (Traffic Skew / Hotspot):

     [ Regular Users ] ---------> [ Shard 1 ] (5% CPU - Healthy)
                                  
     [ Millions of Fans ] ======> [ Shard 2 (Celebrity: Taylor Swift) ] ===> CRASH! (100% CPU)
                                  
     [ Regular Users ] ---------> [ Shard 3 ] (3% CPU - Healthy)


   The Solution (Caching + Key Salting):

                         [ Celebrity Traffic ]
                                   |
                +------------------+------------------+
                | (Read spikes)                       | (Write traffic)
                v                                     v
      [ Redis / Edge CDN Cache ]            [ Key Salting across Sub-shards ]
      (Absorbs 99% of read queries)         (e.g., taylor_swift_1, taylor_swift_2)
   ```
3. **Join and de-normalization (Cross-shard Joins)**:
   - **The Issue:** In a single database, joins across tables are fast and executed locally in memory. But once tables or rows are sharded across separate physical database servers, executing a relational `JOIN` across a network is extremely slow, inefficient, and often not supported by database engines.
   - **Common Solutions:**
     - **De-normalization (Most Common):** Duplicate related fields directly into the main table (e.g., storing `user_name` and `user_email` directly inside the `orders` table). This allows queries to read all needed data from a single table on a single shard without any joins.
     - **Co-locating Data by the Same Shard Key:** Shard related tables using the same partition key (e.g., shard both `orders` and `order_items` by `user_id`). This guarantees that all related records for a specific user live on the exact same physical shard, making local joins possible.
     - **Application-Level Joins:** Fetch records from Shard 1, extract foreign keys, and perform a separate batch query on Shard 2 in application code.

   ```text
   The Problem (Cross-Shard Join Across Physical Machines):

     [ Query: SELECT * FROM orders JOIN users ON ... ]
                         |
           +-------------+-------------+
           | (Network Call)            | (Network Call)
           v                           v
      [ Shard 1 (Orders) ]       [ Shard 2 (Users) ]
      (Order #101, user: 42)     (User #42: Alice)
           \                           /
            +---(Slow Network Join!)--+ 
                * Requires hauling data over the network; very slow and unscalable!


   Solution 1: De-normalization (Embed duplicate data into a single table)

   Before (Normalized across 2 shards - requires slow network join):
     [ Shard 1: orders ]                [ Shard 2: users ]
     +----+---------+---------+         +----+-------+-------------------+
     | id | user_id | item    |         | id | name  | email             |
     +----+---------+---------+         +----+-------+-------------------+
     | 1  | 42      | Laptop  |         | 42 | Alice | alice@example.com |
     +----+---------+---------+         +----+-------+-------------------+

   After (De-normalized - all data in 1 table on Shard 1):
     [ Shard 1: orders ]
     +----+---------+------------+-------------------+
     | id | item    | user_name  | user_email        |
     +----+---------+------------+-------------------+
     | 1  | Laptop  | Alice      | alice@example.com |
     +----+---------+------------+-------------------+
     * Read queries are satisfied 100% locally from Shard 1 in <1ms without any joins!


   Solution 2: Co-locating Data by the Same Shard Key (e.g., user_id)

   By sharding BOTH "orders" and "order_items" using user_id:
     - User 42 data  --> Shard 1
     - User 89 data  --> Shard 2

   Inside Shard 1 (Both tables for User 42 reside on the SAME physical server):
     +-------------------------------------------------------------+
     | [ Shard 1 ]                                                 |
     |                                                             |
     |  Table: orders                                              |
     |  +----------+---------+---------+                           |
     |  | order_id | user_id | total   |                           |
     |  +----------+---------+---------+                           |
     |  | 101      | 42      | $999.00 |                           |
     |  +----------+---------+---------+                           |
     |      |                                                      |
     |      | (Standard SQL JOIN executes locally in memory!)      |
     |      v                                                      |
     |  Table: order_items                                         |
     |  +---------+----------+---------+                           |
     |  | item_id | order_id | user_id |                           |
     |  +---------+----------+---------+                           |
     |  | 501     | 101      | 42      |                           |
     |  +---------+----------+---------+                           |
     +-------------------------------------------------------------+
     * Full database index speed, zero cross-machine network calls!
   ```

<a id="logging-metrics-and-automation"></a>
## 📊 Logging, Metrics, and Automation

> [!TIP]
> **Observability Golden Signals:** Monitor the four golden signals: **Latency** (time to service requests), **Traffic** (demand), **Errors** (rate of failures), and **Saturation** (fraction of constrained resources like CPU/RAM used).

When serving a large business, investing in tools is important!

1. **Logging**: Monitor error logs to identify problems in your system.
   - It can be either per server or aggregated to a centralized service
2. **Metrics**: Gain business insights or health status of the system
   - Host level metrics: CPU/Memory/Disk usage, etc
   - Aggregated level metrics: Performance of the entire DB tier, cache tier, etc
   - Key business metrics: Daily Active Users (DAUs), retention, revenue, etc
3. **Automation**: CI/CD as a good practice to detect problems early and easily and to automate builds to improve developer productivity

<a id="completely-scaled-fundamental-design-with-tips"></a>
## 💡 Completely Scaled Fundamental Design with Tips

**8 tips to scale a system**:

1. Keep web tier stateless
   - _Why:_ Allows servers to seamlessly auto-scale horizontally without sticky sessions, letting any server handle any request.
   - _Example:_ Store session in an external shared store (e.g. Redis) instead of local server RAM:
   ```python
   # Any server can authenticate the user from the shared store
   session = redis.get(f"session:{request.cookies['session_id']}")
   ```
2. Build redundancy at every tier
   - _Why:_ Eliminates Single Points of Failure (SPOF), ensuring high availability and seamless failover if nodes crash.
   - _Example:_ Redundant nodes with automatic failover:
   ```text
   [Primary Load Balancer] <--(heartbeat)--> [Standby Load Balancer]
              |
   [Web Server 1, Web Server 2]              [DB Master] ---> [DB Replica]
   ```
3. Scale your data tier by sharding
   - _Why:_ Distributes massive data volume and high write/read throughput across multiple database instances.
   - _Example:_ Hash-based routing to partition records across DB shards:
   ```python
   shard_id = hash(user_id) % total_shards  # e.g., user 4021 -> Shard 1
   db = db_shards[shard_id]
   user = db.query("SELECT * FROM users WHERE id = ?", user_id)
   ```
4. Split tiers into individual services
   - _Why:_ Decouples components (e.g. via message queues) so they can scale, be updated, and fail independently without blocking web traffic.
   - _Example:_ Decoupling heavy background work via a message queue:
   ```text
   [Web Server] --(push job)--> [Message Queue] ---> [Worker Pool]
   (Returns 202 Accepted instantly; workers process videos/emails asynchronously)
   ```
5. Cache data as much as you can
   - _Why:_ Avoids expensive and repetitive database queries, serving frequent reads with sub-millisecond in-memory speeds.
   - _Example:_ Cache-aside pattern:
   ```python
   user = redis.get(f"user:{user_id}")
   if not user:
       user = db.query("SELECT * FROM users WHERE id = ?", user_id)
       redis.setex(f"user:{user_id}", 3600, user)
   ```
6. Host static assets in CDN
   - _Why:_ Delivers static assets (images, JS, CSS) from edge locations close to users, cutting latency and offloading web servers.
   - _Example:_ Geographic edge caching:
   ```text
   [User in Tokyo] ---> [Tokyo Edge CDN]  (Cache hit: ~10ms latency)
                                | (Cache miss only)
                                v
                       [Origin Server in US] (~200ms latency)
   ```
7. Support multiple data centers
   - _Why:_ Reduces latency for global users via GeoDNS routing and provides cross-region disaster recovery and failover.
   - _Example:_ GeoDNS routing and regional failover:
   ```text
   [User EU] ---> [GeoDNS] ---> [EU Data Center]
   [User US] ---> [GeoDNS] ---> [US Data Center]
   (If US DC fails, GeoDNS automatically re-routes traffic to EU DC)
   ```
8. Monitor your system and use automation tools
   - _Why:_ Provides real-time visibility into system health and bottlenecks via logs/metrics, while CI/CD automates reliable deployments.
   - _Example:_ Automated observability & deployment loop:
   ```text
   [Git Push] ---> [CI/CD Tests & Canary Deploy] ---> [Live Cluster]
                                                            |
   [Prometheus / Grafana] <--- (High latency or 5xx spike) <+
            |
            v
   [Auto-rollback / Alert on PagerDuty]
   ```

![Complete design with shards and tools](https://i.imgur.com/gotzW2T.png)
