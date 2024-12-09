# System design

- [Goal](#goal)
- [The simplest setup](#the-simplest-setup)
- [Handling user growth better](#handling-user-growth-better)
   * [Type of database to use ](#type-of-database-to-use)
   * [Vertical vs horizontal scaling](#vertical-vs-horizontal-scaling)
- [Handle traffic growth with load balancers](#handle-traffic-growth-with-load-balancers)
- [Handle data tier growth with database replication ](#handle-data-tier-growth-with-database-replication)
- [Improve the load or response times with cache layers](#improve-the-load-or-response-times-with-cache-layers)
   * [Database cache tier](#database-cache-tier)
   * [Static content cache with CDNs](#static-content-cache-with-cdns)
   * [Cache considerations for both data and static assets](#cache-considerations-for-both-data-and-static-assets)
- [Managing session state in horizontally scaled web servers](#managing-session-state-in-horizontally-scaled-web-servers)
- [Complete set of optimizations for one single data center](#complete-set-of-optimizations-for-one-single-data-center)
- [Scaling for users across multiple geographical areas](#scaling-for-users-across-multiple-geographical-areas)
- [Scaling systems independently by decoupling them with a message queue](#scaling-systems-independently-by-decoupling-them-with-a-message-queue)
- [Scaling databases beyond the master slave replication with sharding](#scaling-databases-beyond-the-master-slave-replication-with-sharding)
- [Logging, metrics, and automation](#logging-metrics-and-automation)
- [Completely scaled fundamental design with tips](#completely-scaled-fundamental-design-with-tips)

## Goal

About the question:
* The question is to ***design an architecture***
* The question maybe open-ended and the required processes unclear
* The interviewer may choose to focus on either high-level architecture or focus on one area

About the solution:
* The right strategy as well as knowledge are important to do well
* We need a step-by-step framework i.e a systematic approach to find a solution

## The simplest setup

The simplest setup is a setup that is used for an extremely small and finite number of users. Ex: Just one user.

In this setup, we maintain one server for all services! That is, it is a 3-in-1 server
1. Web / Mobile traffic server: (Ex: JSON for an API / HTML for a page request)
	* Ex: mobile client may hit api.mysite.com (which returns JSON)
	* Ex: web client may hit www.mysite.com (which returns HTML)
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

## Handling user growth better

When the number of users starts to scale, one server handling all 3 functions (web/mobile traffic, database, and cache) is not scalable!

First step to optimization is to split the web/mobile traffic from the database server! We will talk about cache later as it has still not been implemented.

Servers needed:
1. Web/Mobile traffic server
2. Database server
Now, these two can be scaled independently!

![Separate DB](https://i.imgur.com/Abf6F9w.png)

### Type of database to use 

There are two options:
1. Relational DBs (RDBMS or SQL DBs):
	* Examples: MySQL, PostgreSQL, Oracle DB
2. Non-relational or NoSQL DBs
	* Examples: Cassandra, DynamoDB, MongoDB, CouchDB

**Relational DBs**

1. Data is in the format of *tables* and *rows*
2. Tables can be joined using SQL statements

**NoSQL DBs**

1. Can be of 4 types: 
	* Key-value stores
	* Graph stores
	* Column stores
	* Document stores
2. We cannot perform joins in general i. not supported

***Default choice must be a relational database*** because of its benefits. Only use non-relational DB for one of the four reasons:

1. Your application requires ***super low latency***
2. Your data is either ***unstructured*** or ***not relational***
3. You only need to ***serialize and deserialize data***. Ex: JSON/XML/YAML
4. You need to ***store massive amounts of data***

### Vertical vs horizontal scaling

We have two servers (web/mobile traffic and database) we can scale up independently. What is the best way to do so when needed? There are two options:
1. Vertical scaling (Scale-up)
2. Horizontal scaling (Scale-out)

**Vertical scaling**
* We add more CPU and RAM power to the existing server
* Only fine when the traffic, i.e number of users, is low
* It has many drawbacks:
	1. CPU/RAM scaling has a hard-limit. It cannot be unlimited!
	2. It does not have ***failover*** or ***redundancy*** i.e If the single server fails, your website goes down for all users! This is also known as a ***Single Point of Failure***

*(My note: Another issue with vertical scaling maybe **cost**. It is expensive to get more powerful servers instantiated and to keep them running)*

**Horizontal scaling**
This is the ***preferred or recommended approach*** for large scale applications!
* It allows us to scale by adding more servers into our pool of resources
* Failover and redundancy can be made possible!

## Handle traffic growth with load balancers

When simultaneous access (maybe #users or user activity) to a server increases the load on it, we will eventually reach the "load limit". Solution: Use a ***load balancer*** coupled with multiple servers behind it (i.e load balancer + horizontal web traffic server scaling)

![load balancer](https://i.imgur.com/xC9OLdD.png)

* Load balancer is the entity that interacts with the rest of the internet on behalf of your servers
* Load balancer will have a public IP
* Load balancer will use private IPs to interact with the servers handling web traffic

**Why private IP for servers?**
It is because it can be accessed only by servers in the same network and not the internet.

**Benefits of a load balancer**
1. Balances the ***load*** on servers
2. No ***failover*** issues: If one server goes offline, all the traffic will be routed to one of the remaining servers (We can also add a healthy server to the pool of servers to balance the load)
3. Improved ***availability***: When web traffic *increases rapidly*, we only need to add more server and the load balancer will balance the traffic between them gracefully

## Handle data tier growth with database replication 

Just like web traffic growth prompts scaling of web servers, the database too will need to be scaled i.e data tier scaling. Why? It is because even database servers can reach their load limit when the simultaneous access increases (i.e many reads and writes)

The recommended approach is to use ***Database replication*** with a *master-slave* approach
* Master: Only supports *write* operations (Create, Update, Delete)
* Slaves: Gets copies of the data from the master (i.e replication) and only supports *read* operations (Read)

Reason: Most systems have more reads than writes. Hence, we want to **optimize for more reads by having more slaves** to read the data from

![database replication](https://i.imgur.com/3WXYaeq.png)

**Advantages**

1. **Better performance**: More read queries are *processed in parallel* due to more slaves
2. **Reliability**: If one of the database servers fail, the *data is still preserved* (replicated across multiple locations
3. **High availability**: Website remains operational even if one database server fails since we can access data stored in another database server

**Scenarios**

1. *Only one slave and it fails*: Redirect read requests to the master temporarily until the issue is fixed or the slave is replaced
2. *Multiple slaves and one of them fails*: Redirect read requests meant for it to one of the other slaves until a new database server replaces the old, faulty slave.
3. *Master database goes offline*: Promote a slave to be the new master. Add a new server to replace the slave that became the master (for replication purposes). *Note: This is harder in production as the data in the promoted slave might not be up to date! We need to run data recovery scripts - other complex techniques exist but it is too early to discuss them/too difficult!*

## Improve the load or response times with cache layers

**What is a cache?**

Temporary storage area ***in memory*** that stores either:
1. The result of expensive responses, or 
2. Frequently accessed data

**Why is a cache better?**

Memory access is *faster* than accessing secondary storage or data from a database. Hence, it is better for performance as it reduces the response time

### Database cache tier

Database data access is slower than reading data from a data cache. Hence:
1. Keep a separate data cache tier i.e a separate server that sits in between the web and the database servers
2. Maintain a caching strategy to fetch and update data (Ex: ***Read-through*** strategy is quite popular)

*Note: Most cache servers provide APIs to use in multiple languages making integration easier*

**When to use a cache?**
***Use a cache when data read is more frequent than data writes!***
Why? Cache is memory which is "*volatile*" which in turn means that if the cache server restarts, the data in memory is lost! Therefore, data should be written to a database (persistent storage) and not a cache.

![data cache tier](https://i.imgur.com/UG96SC0.png)

### Static content cache with CDNs

**What is a CDN?**
*Content Delivery Networks* is a network of *geographically dispersed servers* used to *deliver static content*.
Static content can be images, videos, CSS, JavaScript files, etc...

**How does it work?**
When a user visits a website:
* The website will have the CDN url, instead of the origin URL, listed for a resource
	* Ex: `somecdndomain.com/path/to/image` instead of `mysite.com/path/to/image`
* The CDN server closest to the user geographically will be hit with a request to the resource
* The further the CDN server is from the user geographically, the slower the resource loads!
* Handling cache misses:
	* CDN server requests the file from the origin which is either the web server or a storage like Amazon S3
	* HTTP header Time-To-Live (TTL) descries how long the resource is cached

*CDN for dynamic content is a new concept and a lot more difficult to make work (Ex: Storing entire HTML pages)*

![cdn](https://i.imgur.com/UNpvhZR.png)

### Cache considerations for both data and static assets

1. **Expiration policy**: Not too long nor too short! 
	* Too short: Repeat reloading of content from database or origin server becomes a concern
	* Too long: Data becomes stales i.e not up to date
2. **Eviction policy**: Once the cache is full, we need to replace some of the data
	* ***Least Recently Used (LRU)*** policy is the most common
3. **Consistency**: Keep the data in cache and database or server in sync! When scaling across multiple regions, this becomes challenging
4. **Mitigating failures**: A single cache server can become a ***Single Point Of Failure (SPOF)*** . Solution: Multiple cache servers across different data centers.
5. **Invalidating files**: *This pertains only to CDNs*. If a file needs to be removed before expiry, we need to "*invalidate*" it. Two options:
	1. Use a CDN API to invalidate a resource
	2. Use object versioning to get the latest version of a resource. Ex: API versioning `/v2/...`
6. **Cost**: *This pertains only to CDNs*. CDNs are usually 3rd party vendors (Ex: Cloudfront, Akamai, Cloudflare, etc) and caching things unnecessarily or replacing it unnecessarily can prove to be costly! (Check the point for expiration policy)
7. **CDN fallback**: *This pertains only to CDNs*. If there is a temporary CDN outage, clients must be able to detect it and request a resource from the origin.

## Managing session state in horizontally scaled web servers

When increasing the number of servers horizontally, we need to also **move the state** (Ex: user session data) out of the web tier.

The **recommended approach is to go stateless** by storing state (Ex: session data) in a **separate persistent storage** such as an RDBMS / NoSQL DB server. The benefit? Each web server in the cluster can access state data from the state database(s). This makes the web servers (i.e servers handling web/mobile traffic) *stateless!* 

**Stateful architecture**

1. A web server (i.e servers handling web/mobile traffic) remembers state from one request to another
2. **Drawback**: Same client needs to be routed to the same server every time i.e a sticky session using a sticky routing which is *overhead*. Adding or removing servers becomes challenging with this approach. Hence, not recommended

**Stateless architecture**

1. A web server does not need to remember state
2. A request can be routed to any server at any point in time
3. When a request comes, a web server will fetch the state from a database containing it (Ex: RDBMS/NoSQL/Memcache)
4. **Advantage**: A simpler, more robust, and scalable system. ***Note: The web servers can be "auto-scaled" w.r.t traffic since the state is managed in DB. Auto-scaling is not possible or is challenging if the servers are stateful*** 

![Shared session state DB](https://i.imgur.com/j527SzM.png)

## Complete set of optimizations for one single data center

**Start**: One single web server
**Optimizations**:
1. When user start to scale i.e more users: 
	* Separate out the web server from the database server for independent scaling 
2. More users but also more simultaneous website access:
	* Need to horizontally scale the web servers. Use a load balancer to directing traffic to multiple servers
3. More users means more data is being read than written (traditionally, this is how it is when scaling)
	* Example: Multiple users are reading, liking, commenting, etc... on a user's post
	* Scale the database to handle more read operations in parallel. Use database replication i.e 1 master DB for writes (creates, updates, deletes) and its DB replicated in many slaves that receive and service the read requests
4. Improve the database data request response times
	* Use a data tier cache that stores some of the data in-memory. Use an expiration policy as well as an eviction policy (Ex: LRU cache)
5. Improve the static resources' request response times
	* Use a CDN and take advantage of its geographic proximity to a user. Use an expiration & eviction policy
6. Manage state for horizontally scaled web servers
	* Use a separate shared database for managing only the state (Ex: user session data). Both web server and state database(s) can be scaled independently and the web servers are easy to auto-scale!

![Single data center complete diagram](https://i.imgur.com/8pQsviO.png)

## Scaling for users across multiple geographical areas

To provide users across multiple geographical areas a better experience, i.e performance in terms of response time, we need to support the maintenance of **multiple data centers**.

We have already scaled the database previously to optimize for reads i.e having database replication using a master-slave relationship. However, this does not solve everything. The problems are:
1. There is only one master i.e Single Point of Failure (SPOF) for the writes. However, a failing master can be replaced by promoting a slave. So this problem is solved. But then, when the writes also increase to the extent that it causes whichever master DB to reach its load limit, then we hit a road-block!
2. When users are accessing the application from multiple states, countries, or continents, it does not matter if you increase the number of web servers or databases at one particular geographic location since the response time cannot improve beyond a threshold due to geographic (physical) constraints

**Benefits of multiple data centers**

1. Each data center will contain at least:
	* Horizontally scaled set of web servers
	* Database replications, and a
	* Data tier cache
2. We use a **geoDNS** service: *It is a service that allows domain names to be resolved to IP addresses based on the location of a user*
3. We can also *split the traffic* via geoDNS. Example: `x%` to US East and `(100-x)%` to US West
4. In case of a data center failure, the traffic for it will be re-routed to another healthy data center (while the issue is resolved). Hence, there is *failover support*

*(Note: The shared session state database service will be better kept outside any of the data centers since all of them need to access it)*

**Challenges with multiple data centers**

1. In failover cases: When redirecting traffic to another center, we need to make sure it has the data from the data center that failed (i.e data sync). Common strategy: Replicate data across multiple data centers!
2. Test and deployment: Need to have automation to test, deploy, etc. at different locations
3. Traffic redirection: Need effective tools such as a geoDNS

![Multiple data centers](https://i.imgur.com/CuBnKgX.png)

## Scaling systems independently by decoupling them with a message queue

Imagine a scenario where a user uploads a photo and it needs to be customized. For example, we need to crop, add a filter, compress it, and store multiple copies of different dimensions for different viewports. To do all this ***takes time***. Another similar example includes a user action that triggers the sending of emails by our server.

* *A single server handles everything*: If we expect a web server receiving the photo to do this all this by itself then that is a lot of load and it cannot remain idle to handle other incoming requests! i.e poor response times or worse, failing to respond completely
* *A server offloads the work to a dedicated worker server*: This is better but still has bottlenecks! The operation is *synchronous* and if the dedicated server, known as a ***worker***, becomes unavailable, i.e goes offline, the operation fails and the data is lost too!

We need a ***buffer***! This buffer is known as a **Message queue** service. 

The basic architecture of a *Message Queue* is that of a **producer-consumer**:

1. Web servers produce content (Ex: photo) by publishing to the queue
2. Workers consume the content (Ex: photo processing) by connecting to the queue

**Advantages of message queues**

1. It is a durable component, stored in memory, and distributes asynchronous requests
2. It supports *asynchronous* operations: 
	* A producer can publish to the queue even when the consumer is unavailable
	* A consumer can pick up tasks from the queue once it becomes available (even if the producer is unavailable)
3. The producer and the consumer can be scaled independently (Ex: More workers to process more photos)

![Message queue](https://i.imgur.com/PQPR5tQ.png)

## Scaling databases beyond the master slave replication with sharding

When a database ***(within a single data center)*** get overloaded or near, it might be the right time to scale them beyond the master-slaves replication approach.

There are two options to scale DBs:
1. Vertical scaling: *Not recommended* due to the following reasons:
	1. Hardware limits on a single DB server (no unlimited CPU, RAM, etc access)
	2. Risk of SPOF
	3. Overall cost is higher (high capability systems cost a lot!)
2. Horizontal scaling: Known as **"Sharding"** (Recommended)

**What is database sharding?**

* It is the practice of adding more database servers.
* It is not enough to just add more servers but we need to separate the database itself into smaller, more easily managed parts called ***"shards"*** . 
* Each shard shares the ***same schema*** but the actual data in each shard is unique to the shard

**How sharding works**

1. We need a **sharding key**. For example, a user id can be a sharding key. Usually, a *column* or a *set of columns* is taken to be the key
2. A hash function takes in the sharding key and outputs a selection of one of the shards

It is important to find a ***sharding key that evenly distributes data*** to avoid exhaustion of one database much earlier than the others and to keep the access loads evenly distributed.

![Database sharding](https://i.imgur.com/E7tbWhj.png)

**What are the benefits of sharding?**

1. Great technique to scale databases by routing read and write queries efficiently to the correct database

**What are the challenges of sharding?**

It introduces complexities and new challenges:

1. **Resharding data**: When we exhaust the data possible in a shard, we need to recompute the hash function and moving data around. This is solved by the use of a **consistent hashing** technique
2. **Celebrity problem**: Also known as hotspot key problem. Excessive access to a specific shard could cause server overload (Ex: celebrity searches for Lady Gaga, Taylor Swift, etc all end up on the same shard). We need to identify these hotpsots and separate them into their own shards and further partition them if required
3. **Join and de-normalization**: It is hard to perform joins across a database that has been sharded across multiple servers. Common workaround: De-normalize the database so that queries can be performed in a single table!

## Logging, metrics, and automation

When serving a large business, investing in tools is important!

1. **Logging**: Monitor error logs to identify problems in your system.
	* It can be either per server or aggregated to a centralized service
2. **Metrics**: Gain business insights or health status of the system
	* Host level metrics: CPU/Memory/Disk usage, etc
	* Aggregated level metrics: Performance of the entire DB tier, cache tier, etc
	* Key business metrics: Daily Active Users (DAUs), retention, revenue, etc
3. **Automation**: CI/CD as a good practice to detect problems early and easily and to automate builds to improve developer productivity

## Completely scaled fundamental design with tips

**8 tips to scale a system**:

1. Keep web tier stateless
2. Build redundancy at every tier
3. Scale your data tier by sharding
4. Split tiers into individual services
5. Cache data as much as you can
6. Host static assets in CDN
7. Support multiple data centers
8. Monitor your system and use automation tools

![Complete design with shards and tools](https://i.imgur.com/gotzW2T.png)

