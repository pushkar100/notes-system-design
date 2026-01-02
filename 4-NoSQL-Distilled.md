# NoSQL Distilled

## Table of Contents

- [NoSQL Distilled](#nosql-distilled)
  - [Overview](#overview)
  - [Why NoSQL](#why-nosql)
    - [Value of relational databases](#value-of-relational-databases)
    - [Application vs integration databases](#application-vs-integration-databases)
    - [Impedance mismatch drawback of RDBMS](#impedance-mismatch-drawback-of-rdbms)
    - [The Attack of the Clusters](#the-attack-of-the-clusters)
    - [Emergence of NoSQL](#emergence-of-nosql)
  - [Aggregate data models](#aggregate-data-models)
    - [Relational vs Aggregate data models](#relational-vs-aggregate-data-models)
    - [How to choose aggregate boundaries](#how-to-choose-aggregate-boundaries)
    - [Key-Value and Document Data Models](#key-value-and-document-data-models)
    - [Column-Family Stores](#column-family-stores)
    - [Graph Data Models](#graph-data-models)
  - [Additional NoSQL concepts](#additional-nosql-concepts)
    - [Schemaless Database](#schemaless-database)
    - [Materialized Views](#materialized-views)
    - [Modeling for Data Access](#modeling-for-data-access)
  - [Distribution Models](#distribution-models)
  - [Consistency](#consistency)
  - [Version Stamps](#version-stamps)
  - [Key-Value stores](#key-value-stores)
    - [What is a Key-Value Store](#what-is-a-key-value-store)
    - [Key-Value Store Features](#key-value-store-features)
- [Simple Pseudo-code](#simple-pseudo-code)
    - [Suitable Use Cases](#suitable-use-cases)
    - [When NOT to Use](#when-not-to-use)
  - [Document databases](#document-databases)
    - [What is a Document Database](#what-is-a-document-database)
    - [Features](#features)
    - [Suitable Use Cases](#suitable-use-cases)
    - [When NOT to Use](#when-not-to-use)
  - [Column-Family Stores](#column-family-stores)
    - [What is a Column-Family Store](#what-is-a-column-family-store)
    - [Features](#features)
    - [Suitable Use Cases](#suitable-use-cases)
    - [When NOT to Use](#when-not-to-use)
  - [Graph databases](#graph-databases)
    - [Implementation](#implementation)
    - [Features](#features)
    - [Suitable Use Cases](#suitable-use-cases)
    - [When NOT to Use](#when-not-to-use)
  - [Schema Migrations](#schema-migrations)
    - [Schema Changes](#schema-changes)
    - [Schema Changes in RDBMS](#schema-changes-in-rdbms)
    - [Schema Changes in NoSQL](#schema-changes-in-nosql)
    - [Schema Gremlins (The Danger)](#schema-gremlins-the-danger)
    - [Key Points for System Design](#key-points-for-system-design)
  - [Polyglot persistence](#polyglot-persistence)
    - [Disparate Data Storage Needs](#disparate-data-storage-needs)
    - [Polyglot Data Store Usage](#polyglot-data-store-usage)
    - [Service Usage over Direct Data Store Usage](#service-usage-over-direct-data-store-usage)
    - [Expanding for Better Functionality](#expanding-for-better-functionality)
    - [Choosing the Right Technology](#choosing-the-right-technology)
    - [Enterprise Concerns & Deployment Complexity](#enterprise-concerns-deployment-complexity)
    - [Key Points](#key-points)

## Overview

The Four Main Families of NoSQL 
1. **Key-Value Stores**: (e.g., Riak, Redis) Simple, fast lookups.
2. **Document Databases**: (e.g., MongoDB) Storing data as JSON/XML documents
3. **Column-Family Stores**: (e.g., Cassandra) Optimized for large data sets and high write throughput
4. **Graph Databases**: (e.g., Neo4j) Optimized for traversing relationships between data points

**Note**: DynamoDB is primarily a Key-Value store, but it supports Document (JSON) data structures, making it a *hybrid*. It is Schema-less for everything except the Primary Key. You must define the Key upfront, but the rest of the data (Attributes) can be anything

## Why NoSQL

Why should we use NoSQL when we already have relational databases?

### Value of relational databases

For a long time, **RDBMS (like Oracle, SQL Server, MySQL) was the default choice** for every project. Why?
1. ***Persistence***: They provided a standard way to save data so it survives after a power loss.
2. ***Concurrency***: They handled many users reading/writing at the same time using Transactions.
3. ***Integration***: They acted as a shared integration point for different applications 
3. ***Standardization***: SQL is a standard language. If you know SQL, you can work on almost any project

Keyword: **ACID Transactions**. A set of properties (Atomicity, Consistency, Isolation, Durability) that guarantees data validity. If you transfer money, ACID ensures the money leaves one account and enters the other without getting lost in the middle, even if the server crashes

### Application vs integration databases

1. **Integration Databases (The Old Way)**
- Multiple different applications (e.g., the Accounting App and the Shipping App) all read and write directly to one giant shared database
- *Pro*: All data is in one place
- *Con*: It is very hard to change the database structure (schema) because changing it for the Shipping App might break the Accounting App
```
 [ Accounting App ]      [ Shipping App ]
        |                      |
        +----------+-----------+
                   |
           ( Shared Database )
```
2. **Application Databases (The Modern Way)**
- Each application (or microservice) has its own private database. If the Accounting App needs shipping data, it asks the Shipping App ***via an API*** (web service); it never touches the Shipping DB directly
- *Pro*: ***You can choose the best database for the job*** (e.g., SQL for accounting, NoSQL for shipping logs)
- *Con*: You have to ***manage data syncing between apps yourself***
```
 [ Accounting App ]      [ Shipping App ]
        |                      |
 ( Accounting DB )       ( Shipping DB )
        ^                      ^
        |__ (Talk via API) ____|
```

### Impedance mismatch drawback of RDBMS

A major frustration developers had with RDBMS: the **Impedance Mismatch**

> Keyword: Impedance Mismatch; It is the difference between the relational model (tables/rows) and the in-memory data structures (objects/graphs) used by application code

- The Problem: In **code (Java, C#, Python), we use Objects** (lists, nested structures, maps). **In RDBMS, we use Tables (rows and columns)**
- The Pain: **Translating a complex object (like a User with a list of Addresses and a set of Roles) into flat tables requires complex code or "Mapping" frameworks (like Hibernate)**. It feels *unnatural* and *slows down development*

JSON and XML do not have "Impedance Mismatch" because their data structure (Hierarchical Trees) mirrors exactly how modern programming languages store objects in memory.

The Core Conflict:
- Code (Java/Python): Uses Objects (Trees). A "User" contains a list of "Addresses"
- RDBMS (SQL): Uses Tables (Flat Lists). You must chop the "User" into pieces to store it (Normalization) and stitch it back together to read it (Joins)
- JSON/NoSQL: Uses Documents (Trees). You store the "User" and their "Addresses" together in one big nested blob, just like in your code

```
      [APPLICATION CODE]            [RDBMS / SQL]                   [JSON / NoSQL]
      (Memory: A Tree)              (Disk: Flat Grids)              (Disk: A Tree)

      +-----------------+           Table: USERS                 +------------------+
      | User: Alice     |           +----+-------+               | {                |
      |                 |           | ID | Name  |               |   "name": "Alice"|
      | - Address 1     |   Vs.     | 1  | Alice |   Vs.         |   "addresses": [ |
      | - Address 2     |           +----+-------+               |     "Addr 1",    |
      +-----------------+                 +                      |     "Addr 2"     |
                                    Table: ADDRESSES             |   ]              |
      (Natural Fit!)                +----+-------+               | }                |
            ^                       | ID | Addr  |               +------------------+
            |                       | 1  | Addr1 |                        ^
            |                       | 1  | Addr2 |                        |
            |                       +----+-------+                        |
            |                                                             |
            +====================== MATCH ================================+
                               (No Translation Needed)
```

General (other) drawbacks of RDBMS:
1. **Hard to Scale Out**: They are designed for Vertical Scaling (bigger machine), not Horizontal Scaling (more machines). Distributing data (Sharding) is complex and manual
2. **Rigid Schema:** You *must define the structure (columns/types) upfront*. Changing it later (migrations) on a live database is slow and risky
3. **Impedance Mismatch**: They *don't map well to modern Object-Oriented code* (requires **ORMs** like Hibernate) or unstructured data (JSON/XML).
4. **Performance at Scale**: *Joins become extremely slow as table sizes grow* into the billions of rows

### The Attack of the Clusters

This is the main driver for NoSQL. As web traffic exploded (think Google/Amazon scale), single servers couldn't handle the load

**Scaling Up vs. Scaling Out**
- *Scaling Up (Vertical)*: Buying a bigger, more expensive server with more RAM and CPU. ***RDBMS loves this***
- *Scaling Out (Horizontal)*: Buying lots of cheap, small servers and connecting them ("Clustering"). ***NoSQL loves this***

```
SCALING UP:             SCALING OUT (Clusters):
+-------------+         +---+  +---+  +---+
|  SUPER      |         |Small| |Small| |Small|
|  SERVER     |         | Svr | | Svr | | Svr |
+-------------+         +---+  +---+  +---+
(Expensive limits)      (Cheaper, infinite scale)
```

**Why RDBMS struggles with Clusters?**

*Relational databases are designed to **run on a single machine***. When you try to spread a relational database across 100 servers (Sharding), you lose many benefits:
1. ***Joins are slow***: Joining tables stored on different servers requires massive network traffic.
2. ***Transactions are hard***: Keeping data consistent across 100 servers is very complex and slow.

> Keyword: **Sharding**. Splitting a database into smaller chunks (shards) and storing them across multiple servers

### Emergence of NoSQL

"NoSQL" is a loose term (often meaning **"Not Only SQL"**) for the new wave of databases created to *solve the **clustering problem** and the **impedance mismatch***

Characteristics of NoSQL:
1. **Schema-less**: You don't need to define tables/columns upfront. You can store data flexibly
2. **Cluster-Ready**: Designed from *day one* to run on many servers (Scaling Out)
3. **Open Source**: Most major NoSQL DBs are open source
4. **Aggregate-Oriented**: They often store data the way the application sees it (e.g., *storing a User and their Addresses as one "Document" rather than splitting them into two tables*), ***solving the Impedance Mismatch***

| Feature | Relational (SQL) | NoSQL |
| -- | -- | -- |
| Scaling | Scale Up (Vertical) | Scale Out (Horizontal) |
| Data Model | Tables & Relations | "Documents, Key-Value, Graphs" |
| Structure | Strict Schema | Flexible / Schemaless |
| Best For | "Complex queries, Transacions" | "High scale, Rapid changes" |


## Aggregate data models

### Relational vs Aggregate data models

1. **The Relational Way (Shredding)**: In a relational database (SQL), data is "shredded" into small pieces. To save a "Customer," you might split their data across three tables. To read it back, you must `JOIN` them
	- *Con*: ***Joins are slow***, especially across clusters.
	- *Pro*: Very flexible. You ***can query data from any angle*** i.e RDBMS is good for complex queries

2. **The Aggregate Way**: We *treat a collection of related objects as a single unit—an Aggregate*. We store the Customer, their Addresses, and their Preferences all in one "bucket"
	- *Pro*: 
		- ***Fast to read/write the whole object (just one lookup)***
		- ***Great for clusters (the whole aggregate lives on one server)***
	- *Con*: ***Harder to query deep inside the data*** (e.g., "Find all users who have an address in Boston" is harder if addresses are buried inside a blob)

> Keyword: **Aggregate**. A collection of related objects that we wish to treat as a unit. In NoSQL, this is usually the unit of storage and retrieval

```
RELATIONAL (SQL)             AGGREGATE (NoSQL)
+-------------+              +------------------------+
| Customer ID |              | Customer Aggregate     |
+-------------+      /-----> |------------------------|
       |             |       | ID: 101                |
       v             |       | Name: "Alice"          |
+-------------+      |       |                        |
| Address ID  | <----+       | Addresses: [           |
+-------------+              |   {City: "NY",...},    |
                             |   {City: "LA",...}     |
                             | ]                      |
                             +------------------------+
```

### How to choose aggregate boundaries

How to Choose (The 3 Rules)
1. **Rule of Togetherness (Reads)**: If you almost always read data points together (e.g., User and ProfileSettings), put them in the same document
2. **Rule of Atomicity (Writes)**: *NoSQL usually only guarantees ACID transactions within a single document*. If you need to update two things perfectly in sync (e.g., Order and OrderLineItems), they must be in the same aggregate
3. **Rule of Unbounded Growth (Size)**: *If a list can grow infinitely (e.g., a popular user's Followers or Comments), do not embed it*. Keep it separate to *prevent the document from hitting size limits* (e.g., MongoDB's 16MB limit)

```
        GOOD AGGREGATE                    BAD AGGREGATE
    (Bounded, Atomic Unit)          (Unbounded, High Contention)

   +-----------------------+        +--------------------------+
   | Doc: Order_101        |        | Doc: Product_X           |
   | {                     |        | {                        |
   |   "User": "Alice",    |        |   "Name": "TV",          |
   |   "Items": [          |        |   "Reviews": [           |
   |     {"id": 1, "qty":2}|        |     "Great!",            |
   |     {"id": 5, "qty":1}|        |     "Bad!",              |
   |   ]                   |        |     ... (10,000 more)    | <--- DANGER!
   | }                     |        |   ]                      |      (Document too big)
   +-----------------------+        | }                        |
                                    +--------------------------+
   RESULT: Efficient Reads.         RESULT: Slow writes & Memory
                                            Overflows.
```

### Key-Value and Document Data Models

These are the two main forms of Aggregate-Oriented databases

1. **Key-Value Model** (e.g., Riak, Redis)
	- The simplest NoSQL model. It acts like a ***giant Hash Map*** or ***Dictionary***
	- The Key: The ID (e.g., `"session_123"`)
	- The Value: An *opaque blob* (black box). The ***database does not know or care what is inside the value***. It just stores the bytes
	- How you use it: *You can only get data by knowing the Key*. You cannot run a query like "Give me all blobs where city = NY"

```
 [ KEY ]   ->   [ VALUE (Opaque) ]
 "101"     ->   [ <blob>0101101100...</blob> ]
```

2. **Document Model** (e.g., MongoDB)
	- Similar to Key-Value, but the **Value is transparent** (usually JSON or XML)
	- The Key: The ID
	- The Document: The ***database understands the structure of the data inside***
	- The Difference: Because the DB sees the structure, ***you can query internal fields*** (e.g., "Find documents where city is 'NY'")

```
 [ KEY ]   ->   [ DOCUMENT (Visible Structure) ]
 "101"     ->   { "name": "Alice", "city": "NY" }
```

**Note**: Key-Value and Document models are ***aggregate-oriented*** because they allow you to store, retrieve, and lock an entire complex object (like a User with all their Orders) as a single atomic unit, rather than forcing you to disassemble it into normalized rows across multiple tables

```
    [AGGREGATE-ORIENTED]           [RELATIONAL (Not Aggregate)]
    
    KEY: "User:101"                Table: Users (Row 101)
         |                         Table: Orders (Rows 5, 6, 7)
         V                         Table: Items (Rows A, B, C...)
    { User + Orders + Items }
    (Stored/Read together)         (Stored/Read separately)
```

### Column-Family Stores

Column-Family Stores (e.g., Cassandra, HBase): This is the ***most complex model***, often called *"BigTable" style*. It looks like a table, but it acts like a *two-level nested map*

**Structure**:
1. **Row Key**: Identifies the aggregate (like a Primary Key)
2. **Column Family**: A group of columns that are physically stored together
3. **Column**: The actual data field

**The concept**: It acts like a "Row Key" pointing to a group of "Column Families." Unlike SQL, each row can have different columns inside those families, and each family is stored in a separate file on disk
```
Row Key: "User_101"
   |
   +-- Family: "Profile" (Stored together)
   |      |-- Column: "Name" -> "Alice"
   |      |-- Column: "Email" -> "a@test.com"
   |
   +-- Family: "History" (Stored together)
          |-- Column: "Order_1" -> "Book"
          |-- Column: "Order_2" -> "Pen"
```

> Keyword: **Column Family**. A container for groups of columns. All data in a column family is usually stored in the same file on disk.

**Why use Families?**: It groups related data for performance. You might put frequently accessed data (Profile Info) in one Family and rarely accessed data (Logs) in another. When you read the Profile, the DB doesn't waste time loading the Logs

**Sparse Rows**: Unlike SQL, a row doesn't have to have all columns. Row 1 can have 3 columns; Row 2 can have 10,000 columns

```
LOGICAL VIEW (What it looks like to you)
+---------+----------+----------------+----------------+
| ROW KEY | "Name"   | "Email"        | "Twitter"      |
+---------+----------+----------------+----------------+
| User:01 | "Alice"  | "a@ex.com"     |      --        | <- Gap
| User:02 | "Bob"    |       --       | "@Bobby"       | <- Gap
| User:03 | "Carl"   |       --       |      --        | <- Huge Gap
+---------+----------+----------------+----------------+

PHYSICAL STORAGE (What is actually saved on disk)
(Notice: The "Gaps" consume ZERO space)

Row: User:01
  => Name:    "Alice"
  => Email:   "a@ex.com"

Row: User:02
  => Name:    "Bob"
  => Twitter: "@Bobby"

Row: User:03
  => Name:    "Carl"
```

**Note**: Column-Family stores are ***aggregate-oriented*** because the Row Key acts as the root identifier for a massive, flexible structure (the Row), ensuring that all associated columns—even if they number in the millions—are physically stored together and retrieved as a single logical unit

```
 [AGGREGATE UNIT: THE ROW]
    
    Row Key: "User:101"
       |
       +--> {Family:Info} -> [Name, Email, Age]
       +--> {Family:Posts}-> [Post_1, Post_2, ... Post_99]
       
    (The Database treats "User:101" as the single handle for this entire blob.)
```

### Graph Data Models

Graph Data Models are **NOT** Aggregate-Oriented! They are rather ***relationship-oriented*** (similar to RDBMS) , as they deconstruct data into small individual nodes and edges to prioritize the complex connections between things rather than storing them as large, self-contained units
```
    [AGGREGATE STORE]                [GRAPH STORE]
    (Bounded "Blobs")                (Unbounded Web)

    +------------------+             (Node: Alice)
    | Doc: Alice       |                   |
    | Friends: [Bob]   |            [REL: FRIEND]
    +------------------+                   |
                                           v
    +------------------+             (Node: Bob)
    | Doc: Bob         |                   |
    | Car: Tesla       |            [REL: DRIVES]
    +------------------+                   |
                                           v
    (Data is trapped inside)         (Node: Car)
    (distinct boxes)                 (Data flows everywhere)
```

## Additional NoSQL concepts

### Schemaless Database

Concept: It doesn't mean there is *no* schema; it means ***the database doesn't enforce one***. The schema is implicit in your application code rather than explicit in the database engine. You can store data with different fields in the same bucket (bucket/table/collection), and it is up to your code to handle the differences. This ***allows you to change data structures without downtime*** (no `ALTER TABLE` commands)

Example: A "User" bucket can have old records with just a Name, and new records with Name and Email
```
 [ Database Bucket: "Users" ]
      |
      +---> Record 1: { "id": 1, "name": "Alice" }  <-- (Old Format)
      |
      +---> Record 2: { "id": 2, "name": "Bob", "email": "b@x.com" } <-- (New Format)

(The Database accepts both. Your CODE decides how to read them.)
```

**Buckets**

In a NoSQL datastore (specifically `Key-Value` stores like Riak, Redis, or Couchbase), a **Bucket** is a ***logical container used to group data items together***

Think of it like:
- A Table in a Relational Database (SQL)
- A Folder in a file system

*It acts as a **namespace** to keep keys organized*
- You might have one bucket named `Users` and 
- Another named `Products`

This allows you to have a key called `ID_101` in both buckets without them conflicting

Structure: `Database -> Bucket -> Key -> Value`

### Materialized Views

Concept: A materialized view is ***a pre-calculated answer to a query, stored on disk as if it were a real table***. Instead of running a slow, complex calculation every time someone asks a question, you calculate it once, save the result, and just read the saved result. ***It trades freshness for speed*** (the view might be slightly out of date).

Example: Instead of summing up 1 million sales every time a manager opens the dashboard, you sum them up once an hour and save the total in a DailySales table
```
 [ SOURCE DATA ]             [ CALCULATION ]             [ MATERIALIZED VIEW ]
 (Huge, Slow)                (Complex logic)             (Tiny, Fast)

 +-------------+             (Sum, Join, Sort)           +------------------+
 | Sales Data  | --------------------------------------> | Total: $50,000   |
 | (1M Rows)   |             (Done once/hour)            +------------------+
 +-------------+                                                  ^
                                                                  |
                                                          [User Reads This]
                                                          (Instant access!)
```

> In RDBMS, a materialized view is a database-managed feature that automatically caches query results, whereas in NoSQL, it typically refers to a manual design pattern where the application itself must create and update separate, pre-structured data aggregates to optimize specific read patterns

1. **In RDBMS (The "Magic" Way)**
	- In a relational database (like Oracle or PostgreSQL), a ***materialized view is a feature built into the database engine***
	- You say: "Create a view that sums up sales by month."
	- The DB does: It runs the query, saves the result to disk, and (depending on settings) automatically updates that result whenever you add a new sale
	- Trade-off: ***It is easy for you, but it puts a heavy load on the database server (CPU/RAM)***
2. **In NoSQL (The "Manual" Way)**
	- In the context of NoSQL Distilled, ***a materialized view is often a Design Pattern (something you build), not just a button you click***. Because NoSQL databases are optimized for speed, they rarely do expensive "magic" background calculations for you
	- You do: ***You write code in your application*** (or use a stream processor). "When a user places an order, write the order to the Orders table AND update the TotalSales counter in the Analytics table."
	- The DB does: Just stores what you told it to store
	- Trade-off: ***It is harder to code*** (you have to ensure both writes happen), but it makes reading the data incredibly fast because the answer is already calculated

### Modeling for Data Access

Concept: ***In Relational databases, you model data based on what it is*** (**Entities: Users, Orders, Products**). ***In NoSQL, you model data based on how it is read*** (**Access Patterns**). You design your structures to perfectly match the specific questions your application asks, even if it means duplicating data.

Example: If your app frequently displays "User + Last 5 Orders", you don't keep them separate. You embed the 5 orders directly inside the User object so you can grab everything in one single read
```
Query Needed: "Get User Profile and Recent Orders"

STRATEGY A: Relational (Bad for NoSQL)   STRATEGY B: Access Oriented (Good)
   +------+   +--------+                     +-----------------------+
   | User |---| Orders |                     | User Doc              |
   +------+   +--------+                     | {                     |
       (Requires JOIN)                       |   "Name": "Alice",    |
                                             |   "Orders": [A, B, C] | <--- Embedded!
                                             | }                     |
                                             +-----------------------+
                                             (One single read operation)
```

## Distribution Models 

Distribution is h***ow data is spread across multiple machines***. There are two main axes: **Sharding** (horizontal scaling) and **Replication** (availability)

**A. Sharding (Horizontal Partitioning)**
- Concept: Distributing different data across multiple servers. Each node is responsible for a unique subset of data
- Goal: ***Scalability***. It allows you to handle more data and traffic by adding more machines
- Trade-off: If one node goes down, that specific subset of data is unavailable

**B. Replication (Master-Slave vs. Peer-to-Peer)**
- ***Master-Slave***: One node is the "Master" (handles writes); others are "Slaves" (read-only replicas)
	- Best for: ***Read-heavy workloads***
	- Risk: ***Master is a single point of failure for writes***
- ***Peer-to-Peer*** (***Leaderless***): Every node can handle reads and writes
	- Best for: ***High availability*** and ***write-heavy workloads***
	- Risk: ***Dealing with write conflicts (concurrency)***

```
 [MASTER-SLAVE]                  [PEER-TO-PEER]
     (W)                             (W/R)
      |                               / \
      v                              /   \
 (Master Node)                 (Node A)---(Node B)
    /    \                        \       /
 (R)      (R)                      (Node C)
(Slave)  (Slave)                    (W/R)
```

##  Consistency

NoSQL moves away from strict ACID consistency to handle the **CAP Theorem**

**A. Update Consistency**
- *Write-Write Conflict*: Two people update the same record at the exact same time.
- Solution: Pessimistic locking (lock the record) or Optimistic locking (check version stamps)

```
     [ USER A ]              [ DATA RECORD ]             [ USER B ]
         |                         |                         |
         |      (1) START UPDATE   |                         |
         |------------------------>|                         |
         |                         |      (2) START UPDATE   |
         |                         |<------------------------|
         |                         |                         |
         V                         V                         V

   [ OPTION 1: PESSIMISTIC ]            [ OPTION 2: OPTIMISTIC ]
   "Lock it so no one else             "Let everyone try, but 
    can touch it."                      check the version."
             |                                     |
    (A) Locks Record  <--[LOCKED]-->      (A) Reads Version 1
    (B) User B must WAIT or FAIL          (B) User B Reads Version 1
    (C) User A updates & unlocks          (C) User A Saves (Success!)
                                          (D) DB bumps to Version 2
                                          (E) User B Saves -> [REJECTED]
                                              "Version mismatch!"
```

**B. Read Consistency**
- *Logical Consistency*: Ensuring you see the data you just wrote (***Read-Your-Own-Writes***)
- *Replication Lag*: In a Master-Slave setup, a user might write to the Master but read from a Slave before the data has synced, seeing "stale" data

```
The "Stale Read" Problem (Replication Lag)
This diagram shows the classic scenario where 
a user updates their profile but immediately 
sees the old version because the "Slave" hasn't caught up yet!

     [ USER ]              [ MASTER DB ]             [ SLAVE DB ]
        |                        |                        |
        |   (1) UPDATE NAME      |                        |
        |       "Alice" -> "Bob" |                        |
        |----------------------->|                        |
        |                        |----(2) SYNC DATA------>|
        |      [SUCCESS!]        |      (Takes 500ms)     |
        |<-----------------------|                        |
        |                        |                        |
        |   (3) READ NAME        |                        |
        |------------------------------------------------>|
        |                        |                        |
        |   (4) RETURN "Alice"   |                        |
        |<-----------------------|------------------------|
        |    (Wait, what?!)      |                        |
```

**C. Quorums (Strict Consistency)**
- To ensure consistency in *Peer-to-Peer systems*, use `W + R > N`. `N` = Number of replicas. `W` = Number of nodes that must confirm a write. `R` = Number of nodes that must respond to a read

```
SCENARIO: N=3. We set W=2 and R=2.
(2+2 > 3) -> This guarantees the reader will see at least 
one node with the latest write.

   [Node A] (Old)
   [Node B] (New) <--- Read overlaps here!
   [Node C] (New)
```

```
In this example, we have 3 replicas (N=3). 
We require 2 nodes to confirm a write (W=2) 
and 2 nodes to respond to a read (R=2). 
Because 2 + 2 > 3, consistency is guaranteed!

       [ WRITE ACTION ]                    [ READ ACTION ]
        (Set Value=B)                       (Get Value?)
              |                                   |
      ________|________                   ________|________
     |        |        |                 |        |        |
  [Node 1] [Node 2] [Node 3]          [Node 1] [Node 2] [Node 3]
  |  (B)  | |  (B)  | | (A) |          |  (B)  | |  (B)  | | (A) |
  |_______| |_______| |_____|          |_______| |_______| |_____|
      \        /                                \        /
       \______/                                  \______/
          |                                         |
    W=2 (Success)                             R=2 (Returns B)
                                         (Node 2 has the latest!)
```
* In the diagram above, Node 2 is the overlap. It was part of the write group AND the read group, ensuring the reader sees the "B" written by the writer

## Version Stamps

In distributed systems, we ***can't rely on a single central clock***. We use Version Stamps to track changes and detect conflicts

**A. Optimistic Concurrency Control (OCC)**
- Instead of locking a row, the application reads a version number (`V1`). When it saves, it says: "Update this only if the version is still `V1`." If it's now `V2`, the update fails (Conflict!)
- *Conflict Detection*: The system doesn't stop people from editing at the same time; it just detects when two people tried to change the same thing and makes the "late" person try again

```
     [ USER A ]              [ DATABASE ]              [ USER B ]
         |                        |                         |
    (Reads V1)                    |                    (Reads V1)
         |                        |                         |
    (Saves V1)                    |                         |
         |----(V1 matches!)------>|                         |
         |      SUCCESS!          |                         |
         |      (Now V2)          |                         |
         |                        |        (Saves V1)       |
         |                        |<---(V1 doesn't match!)--|
         |                        |         FAILURE!        |

Read: Everyone grabs the current version number (e.g., V1).
Compare: When you save, the DB asks: 
	"Is the version still what you thought it was?"
Update:
	* If Yes: The save works and the version increments to V2.
	* If No: Someone else beat you to it. You must refresh and try again
```

**B. Vector Clocks**
- The Problem: In Peer-to-Peer, two nodes might accept different writes for the same key simultaneously
- The Solution: A ***Vector Clock is a list of (node, counter) pairs***.
- Example: [A:3, B:1] means Node A has seen 3 changes and Node B has seen 1.
- Conflict Detection: If two stamps have values that aren't strictly "greater than" the other, a divergence has occurred, and the application must resolve it (e.g., "Last Write Wins" or manual merge)

```
     [ START: V1 ]
         [A:1]
        /     \
       /       \
 [ NODE A ]  [ NODE B ]
 (Update)    (Update)
    |           |
 [A:2]       [A:1, B:1]
    \         /
     \       /
   [ CONFLICT! ]


Initial State: [A:1] — Node A created the data.
Node A's Update: It bumps its own counter. New state: [A:2].
Node B's Update: 
	It hasn't seen A's second update yet. 
	It bumps its own counter. New state: [A:1, B:1].

The Conflict: 
	When the system tries to merge them, 
	it sees that [A:2] is not "greater than" [A:1, B:1] 
	(because B's version has a B value that A is missing). 
	This triggers a Divergence.
```

| Concept | Use it when... |
| -- | -- |
| Sharding | Your data is too big for one disk or traffic is too high for one CPU. | 
Quorum (W+R>N) | You need tunable consistency (choosing between speed and "fresh" data). | 
| Read-Your-Writes | You want to prevent a user from thinking their post was deleted after they just hit "Save." | 
| Vector Clocks | You have a leaderless system (like DynamoDB or Riak) and need to handle merge conflicts. | 

Using vector clocks, it is easy to tell that a version X is an ancestor (i.e. no conflict) of version Y if the version counters for **each participant in the vector clock of Y is greater than or equal to the ones in version X**. For example, the vector clock `D([s0, 1], [s1, 1])]` is an ancestor of `D([s0, 1], [s1, 2])`. Therefore, no conflict is recorded

Similarly, you can tell that a version X is a sibling (i.e., a conflict exists) of Y if there is **any participant in Y's vector clock who has a counter that is less than its corresponding counter in X**. For example, the following two vector clocks indicate there is a conflict: `D([s0, 1], [s1, 2])` and D([s0, 2], [s1, 1])`.

Even though vector clocks can resolve conflicts, there are two notable downsides. First, vector clocks add complexity to the client because it needs to implement conflict resolution logic.

Second, the [server: version] pairs in the vector clock could grow rapidly. To fix this problem, we set a threshold for the length, and if it exceeds the limit, the oldest pairs are removed. This can lead to inefficiencies in reconciliation because the descendant relationship cannot be determined accurately. However, based on Dynamo paper [4], Amazon has not yet encountered this problem in production; therefore, it is probably an acceptable solution for most companies.



## Key-Value stores

### What is a Key-Value Store

A Key-Value (KV) store is the simplest form of NoSQL database. It acts like a ***giant Distributed Hash Map*** (or Dictionary).
- The Key: A unique identifier (e.g., `user:123`)
- The Value: An "opaque" blob of data. The database doesn't care what is inside the value—it could be text, a JSON object, an image, or a video
- Access: ***You can only query by the Key***. You cannot ask the database to "find all users where age > 25" because the database cannot see inside the "blob"

```
       KEY (Index)                 VALUE (Opaque Blob)
    +--------------+            +-----------------------+
    | session:99   | ---------> | { "uid": 1, "exp": 3} |
    +--------------+            +-----------------------+
    | user:img:01  | ---------> | [BINARY IMAGE DATA]   |
    +--------------+            +-----------------------+
    | prefs:101    | ---------> | "theme=dark;lang=en"  |
    +--------------+            +-----------------------+
```

### Key-Value Store Features

1. *Consistency*: Usually ***only consistent for operations on a single key***.
2. *Transactions*: Most KV stores ***do not support multi-key transactions***. If you update Key A and Key B, there is no guarantee both happen together.
3. *Querying*: Limited to `GET`, `PUT`, and `DELETE` ***by Key***. To search by value, you would have to pull all data into your application and filter it yourself (very slow).
4. *Scalability*: Extremely easy to scale via Sharding. Since keys are independent, you just hash the key and send it to a specific server

```shell
# Simple Pseudo-code
db.put("user:42", '{"name": "Bob", "location": "NY"}') # Store
user_data = db.get("user:42")                          # Retrieve
```

### Suitable Use Cases

1. **Session Stores**: Every web request carries a Session ID. A KV store can retrieve the user's session data in milliseconds
2. **User Profiles / Preferences**: When you know the UserID and just need to grab their settings
3. **Shopping Carts**: The CartID is the key; the list of items is the value
4. **Caching**: Storing the results of expensive SQL queries or web pages to speed up response times (e.g., Redis/Memcached)

```
       [ APP / WEB SERVER ]
               |
      (1) "Give me data for KEY: X"
               |
               v
      +-------------------------+
      |      KV DATA STORE      |
      |   (Redis / Memcached)   |
      +-------------------------+
      |  [ KEY ]  |  [ VALUE ]  |
      |-----------|-------------|
      | SESS_99   | {User_Data} | <-- Session Store
      | USER_101  | {Prefs: EN} | <-- User Profile
      | CART_505  | [Item1, 2]  | <-- Shopping Cart
      | SQL_CACHE | [Raw_Rows]  | <-- Query Caching
      +-------------------------+
               |
      (2) "Here is the Value"
               |
               v
      [ Instant Response ]
```

### When NOT to Use

1. **Relationships between Data**: If you need to link "Users" to "Orders" to "Products," a KV store is painful because it ***doesn't support Joins***
2. **Multi-Key Transactions**: If you need to update five different records at once and ensure they all succeed or fail together
3. **Querying by Data**: If your UI requires ***complex filters*** (e.g., "Find all blue shirts under $20")
4. **Operations on Sets**: If you frequently need to update just one field inside a large object. In a basic KV store, you have to `GET` the whole blob, change one field, and `PUT` the whole thing back

```
    [ THE "BLIND" BOX PROBLEM ]
    
    (1) NO RELATIONSHIPS (JOINS)
    [User_1] --?--> [Order_A]
    "I can't link these. You have to
     find the ID and look it up yourself."
    
    (2) NO COMPLEX QUERIES
    [ ? ] <--- "Find Blue Shirts < $20"
    "I don't know what's inside the boxes.
     I only know the names on the outside."
    
    (3) NO PARTIAL UPDATES
    [ Old Blob ] --(1. GET)--> [ App ]
                                 | (2. Change 1 item)
    [ New Blob ] <---(3. PUT)-- [     ]
    "To change one word, you have to 
     rewrite the entire book."
```

| Feature | Description |
| -- | -- |
| Data Model | Pair of Key and Value |
| Scalability | High (Horizontal via Sharding) |
| Performance | Extremely Fast (Low latency) |
| Complexity | Lowest of all NoSQL types |
| Examples | Redis, Riak, Amazon DynamoDB, Memcached |

## Document databases

### What is a Document Database

A Document Database ***stores data as documents (usually JSON, BSON, or XML)***. Unlike Key-Value stores, ***the "Value" is not opaque***; it is ***a structured object that the database can inspect, index, and query***

- Self-Describing: Each document contains ***both the data and the labels*** for that data
- *Hierarchical*: Documents can contain ***nested*** lists and other documents (Aggregates)

```
COLLECTION: "Orders"
      |
      +---> Document: {
      |       "_id": "ORD-99",
      |       "customer": "Alice",
      |       "items": [
      |          {"item": "Book", "price": 20},
      |          {"item": "Pen",  "price": 2}
      |       ],
      |       "total": 22
      |     }
```

### Features

1. **Content Awareness**: Because the DB understands the structure, you ***can query specific fields*** (e.g., `db.orders.find({"customer": "Alice"})`).
2. **Indexing**: You can create indexes on nested fields (like the "price" inside the "items" array) to make searches faster
3. **Schema Flexibility**: ***Every document in a collection can have different fields***. You can add a `"discount_code"` field to new orders without affecting old ones
4. **Aggregate-Oriented**: ***Related data is stored together***. Instead of joining a Users table and an Addresses table, you store the address inside the user document

```js
// MongoDB style query to find expensive items in any order
db.orders.find({ "items.price": { $gt: 100 } });
```
Schema flexibility:
```
    [ "PRODUCTS" COLLECTION ]
                 |
      +--------------------------+
      | {                        |
      |   "id": "item_1",        |
      |   "type": "T-Shirt",     | <--- Product A
      |   "size": "Large",       |
      |   "color": "Red"         |
      | }                        |
      +--------------------------+
      | {                        |
      |   "id": "item_2",        |
      |   "type": "Hard Drive",  | <--- Product B (Different fields!)
      |   "capacity": "2TB",     |
      |   "interface": "USB-C"   |
      | }                        |
      +--------------------------+
                 |
        [ QUERY ENGINE ] 
     "Find all 'USB-C' items" 
                 |
        ( Returns Item 2 )
```

### Suitable Use Cases

1. **Content Management Systems (CMS)**: Different types of content (articles, videos, images) have *different metadata but need to live in the same "Posts" collection*
2. **E-commerce Platforms**: Products have wildly different attributes (a T-shirt has "Size," but a Hard Drive has "Capacity")
3. **Web Analytics / Real-time Logging**: You can capture varying event data from different sources into a single stream
4. **User Profiles**: When user data is complex and hierarchical (e.g., social media profiles with interests, education, and work history)

### When NOT to Use

1. **Complex Multi-Document Joins**: If your data is ***highly normalized and you constantly need to link many different documents together***, the "Join-on-the-fly" performance is poor compared to RDBMS
2. **Strict Consistency across multiple documents**: While most Document DBs are ACID-compliant at the single document level, they ***may struggle with transactions involving many different collections***
3. **Constantly Changing Aggregate Boundaries**: If you frequently need to pull data apart and re-group it in ways that cross your document boundaries, a Relational or Graph model is better

```
     (1) THE JOIN NIGHTMARE             (2) THE TRANSACTION GAP
     [ Orders ]   [ Products ]          [ Inventory ]   [ Accounting ]
         |             |                      |               |
         +----(JOIN)---+                      +----(XACT)-----+
                |                             "Update both or neither!"
      "Slow! The app must do                 "Hard to guarantee across
       the heavy lifting."                    separate collections."

     (3) THE FIXED BOUNDARY PROBLEM
     [   AGGREGATE A   ]          [   AGGREGATE B   ]
     { User + Address  }          { Order + Items   }
               \                  /
                \__( New Query )_/
             "I need a report linking 
              Address to Items only!"
     "Painful. You have to 'break' your aggregates
      to get the data you need."
```

| Feature | Document Database | Key-Value Database |
| -- | -- | -- |
| Visibility | Database sees and indexes fields | Database sees a "blob" (opaque) |
| Querying | Can query by internal attributes | Can only query by ID (Key) |
| Storage | JSON, XML, BSON | Strings, Binary, JSON |
| Examples | MongoDB, CouchDB | Redis, Riak |

## Column-Family Stores

### What is a Column-Family Store

Think of it as a **two-dimensional Key-Value map**. Instead of a simple `"Key -> Blob"` (Key-Value) or `"Key -> Document"` (Document), it uses a **Row Key** to map to multiple **Column Families**
- **Column**: The smallest unit of data (Name, Value, and a Timestamp)
- **Column Family**: A logical grouping of related columns (e.g., "ProfileData"). These are usually stored together on disk
- **Super Columns**: A column that contains other columns (nested)

```
Row Key: "User:123"
 ├─ Column Family: [Info]
 │   ├─ Name: "Alice"
 │   └─ City: "London"
 └─ Column Family: [Activity]
     ├─ LastLogin: "2023-10-01"
     └─ Points: "550"
```

### Features

1. **Sparse Storage**: If a row doesn't have data for a specific column, it takes up ***zero space***. ***You don't store "`NULL`s" like in SQL***
2. **High Write Throughput**: They are optimized for writes (using LSM-Trees), making them ***much faster than RDBMS for heavy ingestion***
3. **Physical Separation**: ***Different Column Families are stored in separate files***. If you only need "Name," the database doesn't have to touch the "Activity" files
4. **Versioning**: ***Every cell has a timestamp***. You can store multiple versions of the same column (e.g., the last 3 addresses of a user)

```SQL
-- Creating a table with a Column Family structure
CREATE TABLE users (
  user_id int PRIMARY KEY,
  name text,
  email text,
  city text
) WITH COMPACT STORAGE;
```

### Suitable Use Cases

1. **Event Logging**: Capturing millions of small events per second (e.g., website clicks, sensor data)
2. **Content Management / Blogging**: Where rows might have very different attributes (one post has tags, another has a thumbnail, another has neither)
3. **Expiring Data**: Because of the timestamps and "Time To Live" (TTL) features, they are great for data that should disappear after a while
4. **Big Data Analytics**: When you have billions of rows but only need to read a few specific columns (the "Columnar" nature makes this fast)

```
The primary strength of Column-Family stores is their ability to 
handle sparse data (rows with different columns) and high-volume ingestion.

      [ COLUMN-FAMILY STRUCTURE ]
      
      Row Key: "Post_101"
      +-------------------------------------------------------+
      | Family: Metadata    | Family: Content                 |
      |---------------------|---------------------------------|
      | author: "Alice"     | body: "Hello World..."          |
      | date: "2023-10-01"  | tags: ["NoSQL", "Tech"]         |
      +-------------------------------------------------------+
      
      Row Key: "Post_102" (Sparse - Different columns!)
      +-------------------------------------------------------+
      | Family: Metadata    | Family: Content                 |
      |---------------------|---------------------------------|
      | author: "Bob"       | body: "Check this video!"       |
      |                     | thumbnail: "video_thumb.jpg"    |
      +-------------------------------------------------------+
```

### When NOT to Use

1. **Early Stage Prototyping**: These databases require you to ***know your query patterns upfront***. If you don't know how you'll need to read the data, you’ll struggle
2. **ACID Transactions**: They typically ***do not support multi-row transactions***
3. **Complex Joins**: Like most NoSQL, ***they don't do joins***. You must denormalize your data (duplicate it) into separate tables to satisfy different queries

```
     (1) THE "KNOW THY QUERY" PROBLEM
     [ App Design ] ----> [ Table Schema ]
          ^                      |
          |                      v
     "Wait, I want to     "Sorry! You didn't design
      query by Date now!"  the table for that. 
                           Start over."

     (2) THE DENORMALIZATION TAX
     [ Table by UserID ]      [ Table by Date ]
     { User: Alice     }      { Date: Oct 1st   }
     { Data: "Hi"      }      { Data: "Hi"      }
             \                  /
              \____( Update )__/
           "To change one post, you must 
            update it in 5 different places!"
```

## Graph databases

Graph databases are presented as the ***"exception" to the aggregate-oriented rule***. While other NoSQL types group data into "blobs," ***Graphs focus on the connections between data points***

### Implementation 

Graphs consist of three core elements:
1. **Nodes** (Vertices): The entities (e.g., People, Cities, Products)
2. **Edges** (Relationships): The lines connecting nodes. They are first-class citizens and can have their own properties
3. **Properties**: Key-value pairs attached to both nodes and edges

```
(Node: Alice) --[Edge: LIVES_IN {since: 2010}]--> (Node: London)
        |                                             ^
  [Edge: FRIENDS_WITH]                          [Edge: LOCATED_IN]
        |                                             |
        v                                             |
  (Node: Bob)   --[Edge: BORN_IN]---------------------+
```

### Features

1. Relationship Focus: Traversing relationships is extremely fast (O(1) or constant time) regardless of the total data size.
2. Schema-less: You can add new relationship types or node properties on the fly without migrating a schema.
3. No Joins: Unlike SQL, which computes relationships at query time using expensive joins, Graphs store the connections physically on disk.
4. Graph Algorithms: Native support for pathfinding (Dijkstra’s), centrality (influencer detection), and clustering.

### Suitable Use Cases

1. Social Networks: Finding "friends of friends" or common interests.
2. Recommendation Engines: "People who bought this also bought that" or "You might know..."
3. Fraud Detection: Detecting suspicious patterns/loops in financial transactions or insurance claims.
4. Network/IT Operations: Mapping dependencies between servers, routers, and applications to find single points of failure.

### When NOT to Use

1. Bulk Updates: If you need to update a single attribute on every single record (e.g., "Increase all prices by 10%"), a Relational or Column-family store is better.
2. Partitioning/Sharding: Graphs are notoriously hard to shard across multiple servers because a single query might need to "hop" across many nodes, leading to massive network latency.
3. Simple Key-Value lookups: If you only need to fetch a user profile by ID, the overhead of a Graph engine is unnecessary.

## Schema Migrations

### Schema Changes

Every successful application eventually needs to change its data. Whether you are adding a new feature (e.g., "Add Social Media links to User Profiles") or fixing a design flaw, you have to ***transition*** from **Version A** to **Version B**
- The Problem: You have millions of existing records in the "Version A" format
- The Goal: ***Ensure the application doesn't break while transitioning*** to "Version B"

### Schema Changes in RDBMS

In a Relational Database, the ***schema is explicit***. The ***database engine "owns" the structure***
- Process: ***You use Migrations (scripts)***. You run an `ALTER TABLE` command to add columns or change types
- The Downside:
	1. *Downtime*: On large tables, `ALTER TABLE` can lock the database for hours
	2. *All-or-Nothing*: Every single row must conform to the new schema immediately. There is no "middle ground"

```
 [V1 Database] ----(Migration Script)----> [V2 Database]
 (Row 1: V1)       "ALTER TABLE..."        (Row 1: V2)
 (Row 2: V1)       [LOCKS TABLE]           (Row 2: V2)
 (Row 3: V1)                               (Row 3: V2)
       ^                                         ^
 App V1 works                              App V2 works
```

### Schema Changes in NoSQL

In NoSQL, the ***schema is implicit***. The ***application code "owns" the structure***. This allows for ***Incremental Migrations***, where different versions of data *coexist*

**A. Incremental Migration (Lazy Migration)**
- Instead of updating all 1 million rows at once, you update them ***only when they are accessed***
- Read: App reads a document
- Check: "Is this Version 1 or Version 2?"
- Update: If V1, the app converts it to V2 in memory
- Save: The app saves the V2 version back to the DB

```
 [DATABASE]                     [APPLICATION CODE]
| Row 1 (V1) | --(Read)--> | if record.v == 1:       |
| Row 2 (V2) |             |    record.new_field = X |
| Row 3 (V1) |             |    record.v = 2         |
                           | save(record) <--(Write)-|
                                 
Result: Over time, the database "heals" itself to V2.
```

### Schema Gremlins (The Danger)

Because NoSQL allows different versions to live together, you can run into "***Gremlins***"—old code trying to read new data, or new code encountering ancient data it doesn't know how to handle. ***You must use Feature Toggles or Conditional Logic in your code***

Data Snippet (Handling Versions in Code):
```python
def get_user_profile(user_id):
    doc = db.users.find(user_id)
    
    # Handle missing fields from older versions (Implicit Schema)
    if "twitter_handle" not in doc:
        doc["twitter_handle"] = "Not Provided"
        
    return doc
```

### Key Points for System Design

1. **Move Complexity to Code**: NoSQL doesn't eliminate migrations; it shifts the burden from the Database Administrator (DBA) to the Developer.
2. **Version Everything**: Always include a version field in your NoSQL documents. It makes migrations much easier to manage
3. **Read-Write Asymmetry**: You can often read multiple versions, but you should only write the latest version
4. **Scrubbing Scripts**: Sometimes, "Lazy Migration" isn't enough (e.g., you need to calculate a new field for a report). In this case, you write a background script (a "scrubber") that slowly crawls the DB and updates records during low-traffic hours

## Polyglot persistence

**Polyglot Persistence**: the idea that any large enterprise should use different data storage technologies for different problems, rather than trying to force everything into a single RDBMS

### Disparate Data Storage Needs

In the past, we tried to make the Relational Database (RDBMS) do everything. However, modern apps have diverse needs that one model can't satisfy perfectly:
- Search: Needs inverted indexes (Elasticsearch)
- Social Connections: Needs graph traversals (Neo4j)
- High-Volume Caching: Needs simple key-value lookups (Redis)
- Financial Records: Needs strict ACID transactions (PostgreSQL)

**The Concept**: Use the right tool for the job

### Polyglot Data Store Usage

A single application can communicate with multiple databases simultaneously

Example: An E-commerce App
- Product Catalog: Document Database (flexible attributes)
- Shopping Cart: Key-Value Store (fast, ephemeral)
- Recommendations: Graph Database (relationships)
- Sales Ledger: Relational DB (financial integrity)

```
        [ USER INTERFACE ]
              |
      [ APPLICATION LOGIC ]
        /      |        \
       /       |         \
 [Redis]   [MongoDB]   [Neo4j]   [Postgres]
 (Cart)    (Catalog)   (Social)   (Orders)
```

### Service Usage over Direct Data Store Usage

If every microservice talks directly to every database, you get "**Integration Spaghetti**"
- The Better Way: Wrap each data store in a Service

Other parts of the app call the RecommendationService via API rather than querying the Graph DB directly. This encapsulates the data model and allows you to swap databases later without breaking the whole system

```
      [ External App Parts ]
      (e.g., UI, Billing, Shipping)
                |
                | (1) Simple API Request
                | "Get recommendations for User 101"
                v
      +-------------------------+
      |  RecommendationService  | <--- Encapsulation Layer
      +-------------------------+
                |
                | (2) Complex Graph Query
                | "MATCH (u:User)-[:LIKES]->(m:Movie)..."
                v
         (( Graph DB ))
         (e.g., Neo4j)
```
- Encapsulation: The "External App Parts" don't need to know how to write Graph queries (Cypher/Gremlin); they only need to know how to call a standard web API
- Flexibility: If you decide to swap the Graph DB for a Document DB (like MongoDB) later, you only have to rewrite the code inside the RecommendationService. The rest of the app never even notices the change
- Security: Only one service has the credentials and access to touch the raw data, reducing the "attack surface" of your database

### Expanding for Better Functionality

Polyglot persistence isn't just about "storing" data; it's about better features. By adding a specialized store, you enable features that were previously impossible or too slow

Example: Adding a Spatial Database (like `PostGIS`) suddenly allows you to offer "Find a store near me" features with high performance

In this model, the Spatial DB handles the complex math of geography, while your Relational DB continues to handle what it does best (like transactions and user profiles):
```
       [ User Request ]
             |
             | "Find stores within 5 miles of me"
             v
    +----------------------+
    |   Application Code   |
    +----------------------+
      /                  \
     /                    \
    v                      v
[ Relational DB ]      [ Spatial DB (PostGIS) ]
(User Profiles,        (Coordinates, Polygons, 
 Order History)         Store Locations)
      |                      |
      |                      |-- (Fast Calculation) --+
      |                                               |
      +-----------------------------------------------+
                             |
                             v
                    [ Result to User ]
                 "Found 3 stores near you!"
```

### Choosing the Right Technology

Fowler and Sadalage suggest picking based on these 3 metrics / factors:
1. **Data Model**: Does the data look like a tree (Document), a graph (Graph), or a map (KV)?
2. **Query Patterns**: Do you need to find things by ID, or by complex relationships?
3. **Consistency Needs**: Do you need strict ACID or is "Eventual Consistency" okay?

```
   [ 1. DATA MODEL ]                [ 2. QUERY PATTERNS ]            [ 3. CONSISTENCY ]
   "What does it look like?"         "How do we find it?"           "How accurate now?"
  
   (  Nested Tree / JSON ) ------> ( Find by ID or Field ) ------> [ DOCUMENT DB ]
             |                               |                    (e.g., MongoDB)
             |                               |
   ( Simple Key-Value Pair ) ----> ( Lookup by Primary Key ) ----> [ KEY-VALUE STORE ]
             |                               |                    (e.g., Redis)
             |                               |
   ( Complex Relationships ) ----> ( Traverse the Network ) -----> [ GRAPH DB ]
             |                               |                    (e.g., Neo4j)
             |                               |
   ( Giant Sparse Table ) --------> ( High-Volume Writes ) ------> [ COLUMN-FAMILY ]
                                                                  (e.g., Cassandra)
```

```
   [ 1. DATA MODEL ]                [ 2. QUERY PATTERNS ]            [ 3. CONSISTENCY ]
   "What does it look like?"         "How do we find it?"           "How accurate now?"
  
   ( Normalized / Tabular ) ------> ( Complex Joins/Reporting ) ----> [ RDBMS (SQL) ]
             |                               |                    (e.g., PostgreSQL)
             |                               |
   ( Fixed Schema / Rows ) --------> ( Ad-hoc Multi-table ) --------> [ STICK TO SQL ]
                                         Queries
```

### Enterprise Concerns & Deployment Complexity

Polyglot persistence ***isn't free***. It adds Organizational Debt:
1. **Knowledge Gap**: Your DBAs now need to know 4 different systems instead of 1
2. **Backup/Recovery**: You can't just take one "snapshot." You have to coordinate backups across different types of engines
3. **Deployment**: Your CI/CD pipeline becomes more complex as you manage multiple clusters

```
  LOW COMPLEXITY                HIGH COMPLEXITY
  (Standard RDBMS)              (Polyglot)
  
  [App] -> [SQL]                [App] -> [SQL]
                                     |-> [NoSQL A]
                                     |-> [NoSQL B]
  (One backup script)           (Three backup scripts, 
                                 three monitor alerts,
                                 three security patches)
```

### Key Points

1. **Encapsulation**: Always wrap your data stores in ***services*** to hide the polyglot complexity from the rest of the business logic
2. **Don't Over-Engineer**: Just because you can use 5 databases doesn't mean you should. ***Start with one and add others only when the performance or functional gain is significant***
3. **Reporting**: Polyglot persistence makes reporting hard because data is scattered. You often need a ***Data Lake or ETL process to pull everything into one place for the business analysts***

| Aspect | Single RDBMS | Polyglot Persistence |
| -- | -- | -- |
| Development | Easier (One language: SQL) | Harder (Multiple APIs/Models) |
| Performance | Compromised for some tasks | Optimized for every task |
| Maintenance | Simple | Complex / Expensive |
| Scalability | Vertical (Hard) | Horizontal (Easier) |
