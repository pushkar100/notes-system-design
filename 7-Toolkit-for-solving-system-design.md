# Toolkit for Solving System Design Problems

The **HelloInterview.com** approach

## Table of Contents

- [Toolkit for Solving System Design Problems](#toolkit-for-solving-system-design-problems)
  - [The 6 step approach a problem](#the-6-step-approach-a-problem)
  - [Gathering Functional Requirements](#gathering-functional-requirements)
    - [Tip for a question you are familiar with](#tip-for-a-question-you-are-familiar-with)
  - [Gathering Non-Functional Requirements](#gathering-non-functional-requirements)
  - [Defining our core entities](#defining-our-core-entities)
  - [Designing our API](#designing-our-api)
  - [High Level Design to fulfil FRs](#high-level-design-to-fulfil-frs)
  - [Deep Dive to scale our design](#deep-dive-to-scale-our-design)
  - [Important HTTP verbs and status codes for API endpoints](#important-http-verbs-and-status-codes-for-api-endpoints)
  - [Data for Back Of The Envelope estimations](#data-for-back-of-the-envelope-estimations)
    - [QPS and storage capacities for different services](#qps-and-storage-capacities-for-different-services)
    - [Storage calculation example](#storage-calculation-example)
    - [QPS calculation example](#qps-calculation-example)

## The 6 step approach a problem

The **6-Step** System Design Process in ***35-45 minute interview***

1. **Understanding the requirements**
	- Collect the ***Functional Requirements*** (Define the core features), ***Non-Functional Requirements*** (Consider performance, scalability, reliability, security), and  ***Constraints*** (Account for time, budget, regulatory, and technical limitations)
2. **Identify core entities**
	- These are the entities that are going to be persisted and exchanged in our system (Ex: Names of the tables or collections you will have in your data model)
3. **Identify the APIs**
	- The contract between your client and your users and your backend service
4. **Identify the data flow**
	- Only applicable to *infra-heavy* systems (Ex: Design a rate limiter, a message queue, etc)
5. **High-level design**
	- This is the step where we draw the boxes and arrows in order to get an outline of our system. *Primary goal: Satisfy the **functional requirements** of **step 1***
	- At this point, the system is ***simple*** (***Not able to scale yet** or do anything fancy!*)
6. **Deep dives**
	- Evolve the high-level design in a way such that it *satisfies the **non-functional requirements** of ***step 1*** (**Scalability, reliability, performance**, etc)*
	- ***It is better to do the "Back of the Envelope" estimations during this phase than earlier!*** 
		- *Why*? It is because we do not need those numbers if we do not want to scale our system yet! (Acknowledge to the interviewer earlier that this needs to be done but you are parking it until this phase)

```
+-----------------------------------------------------------+ <---+
|  1. UNDERSTAND REQUIREMENTS                               |     |
|  • Functional (Core Features)                             |     |
|  • Non-Functional (Scale, Reliability, Speed)             |     |
|  • Constraints (Budget, Time, Tech Stack)                 |     |
+-----------------------------------------------------------+     |
                            |                                     |
                            v                                     |
+-----------------------------------------------------------+     |
|  2. IDENTIFY CORE ENTITIES                                |     |
|  • The "Nouns" of your system                             |     |
|  • Data Model (SQL Tables / NoSQL Collections)            |     |
+-----------------------------------------------------------+     |
                            |                                     |
                            v                                     |
+-----------------------------------------------------------+     |
|  3. IDENTIFY APIS                                         |     |
|  • The Contract (Inputs/Outputs)                          |     |
|  • Client <---> Backend Communication                     |     |
+-----------------------------------------------------------+     |
                            |                                     |
                            v                                     |
. - - - - - - - - - - - - - - - - - - - - - - - - - - - - - .     |
:  4. IDENTIFY DATA FLOW (Optional)                         :     |
:  • Only for Infra-Heavy components (e.g., Rate Limiter)   :     |
:  • Map how data moves through the logic                   :     |
' - - - - - - - - - - - - - - - - - - - - - - - - - - - - - '     |
                            |                                     |
                            v                                     |
+-----------------------------------------------------------+     |
|  5. HIGH-LEVEL DESIGN                                     | ----+
|  • "Boxes and Arrows" Diagram                             |     |
|  • Goal: Satisfy FUNCTIONAL Requirements                  |     |
|  • State: Simple, working, but not scalable               |     |
+-----------------------------------------------------------+     |
                            |                                     |
                            v                                     |
+-----------------------------------------------------------+     |
|  6. DEEP DIVES                                            | ----+
|  • Evolve the Architecture (Shard, Cache, LB)             |
|  • Goal: Satisfy NON-FUNCTIONAL Requirements              |
+-----------------------------------------------------------+
```

## Gathering Functional Requirements

YOU DO NOT NEED A LONG LIST OF 'ALL' FEATURES. ***YOU ONLY NEED AROUND 3 CORE FEATURES*** TO FOCUS ON IN THE ~45 MINS INTERVIEW

***Basic type of questions to ask***: **`"The [Actor] should be able to / can do [Action]"`**

**Actors can be**:
1. Your *primary user* (Ex: Passenger in a ride hailing app)
2. Your *secondary user* (Ex: Driver in a ride hailing app)
3. Your *system* (Ex: The ride hailing app)

**Examples**: *Does the problem statement or interviewer agree with / tell you that*:
- "The User should be able to upload 4K videos"
- "The System should be able to show a feed of trending posts"
- "The Admin should be able to ban users for violating rules"

### Tip for a question you are familiar with

If the question is something you are ***familiar with***, you can build the questions to ask based on the problem statement itself (**your insight**)
- Ex: "Should we allow custom aliases to be set for a short URL?" in a URL shortener system design question
- Ex: "Should we allow expiry for a URL once set?" in a URL shortener system design question

(**Note**: If it is a ***question you are NOT familiar with***, pepper your interviewer with ***a lot of clarifying questions*** like "Can/Should the user do [action]?")

***Pro Tip: The "Out of Scope" Check***: Always ask one final question to save yourself from over-engineering. Ex: "What are we NOT building right now?" ("No video editing tools," "No direct messaging for v1", ...)

## Gathering Non-Functional Requirements

Consider CAP theorem, environment constraints, scalability, latency, durability, security, fault tolerance, compliance, etc.

Solution: ***Think about which of the following are important or relevant to the system to be designed and add only those***

**Tip**: ***Ask "System should have/be ..."*** questions to *each* of these quality attributes and check if it makes sense! 
- Ex: "System should have low latency" --> true? why? why not? ... and make a choice

**Quality attributes**:
1. **Performance (Latency)**: ***Is it fast enough?*** (Never use "average" as it hides outliers. Always use Percentiles like `p95`, `p99`)
	- Ex: "The home feed must render within 200ms for 99% of users (p99)"
	- Ex: "Typeahead suggestions must appear within 50ms"
	- Ex: "The system must acknowledge the upload in <100ms, but the actual video processing can take up to 5 minutes"
2. **Availability**: ***Does it stay up***?, ***Does it require redundancy***?
	- Ex: "Must have ZERO downtime during failure"
	- Ex: "Can accept 5 minutes of downtime"
3. **Consistency**: ***Is data accurate everywhere***?
	- *Strong consistency*: Ex: Banking systems, Ticketing systems (preventing double booking). "If User A posts a comment, User B must see it immediately upon refreshing. We cannot serve stale data"
	- *Eventual consistency*: "If User A changes their profile picture, it is acceptable for User B to see the old picture for up to 1 minute"
4. **Scalability**: ***Can we handle growth?***
	- Decides if you need *Sharding* (Database) or *Horizontal Scaling* (App Servers)

**We can only have Availability or Consistency but not both** (**CAP Theorem**)
- Partition is a *given* (i.e 100%) in a distributed system
- We must therefore make the decision between Availability and Consistency for our system based on what is more important (Discussions with Interviewer, reasonable expectations that lead to the choice)

**Quantify**
- Whenever possible, ***try to quantify your non-functional requirements***. *This will help later in your design! (Back of the Envelope calculations)*
- Ex: "low latency" could be "low latency search results in under 500ms"

**Last step**

Identify attributes *specific* to the question:
- Ex: **uniqueness** in a URL shortener system design question

## Defining our core entities

Think *"objects"* and *"users"*

These are the entities that are:
1. ***Persisted in our database*** (stored), AND
2. ***Exchanged via the APIs*** 

**Example**: short url, long url, user (the one creating them)

**Note**: DO NOT list the entire data model just yet! (Ex: No `id` or `user_id` and other attributes, just the names)

## Designing our API

The ***contract*** between our clients / users and our system

*Steps*: 
1. Check each functional requirement
2. Create an *action* (endpoint) to fulfil that requirement
3. Use only the 
	1. ***Appropriate HTTP verb*** (Ex: `GET`, `POST`)
	2. ***URL Path***
	3. ***Response attribute (or a larger response body)*** 
	4. ***Payload / body*** (With just enough data to wholly cover the FRs). ***Omitted for `GET` requests*** 

**Note**: Use REST API semantics (Ex: nouns for resources and plurals for object creation) but *not all the descriptions of RESTful* 

Example:
```
// Create a short URL
POST /urls -> shortUrl
{
    longUrl,
    alias?,
    expiry?
}

// Redirect to a long Url
GET {shortUrl} -> Redirect to the original longUrl
```

## High Level Design to fulfil FRs

*Steps*:
1. Start with a monolith (Simple client-server architecture): A ***client***, a ***monolith server***, a ***database***
2. The API endpoints are implemented as "methods" on arrows from  client (Ex: `getUser()`)
	- If there are any specific / unusual HTTP response statuses (Ex: `301` , `302`) for a request, it is a good idea to have a discussion i.e pros and cons with the interviewer and settle on one!
3. When addressing an endpoint, *identify the action on the server*. These form the *core biz logic*
	- Ex: If `getUser` then action on server is "Lookup the user"
	- Ex: If `createShortUrl()` then action on server is "Create short URL from long URL"
	- *Note*: Usually, it's one of two types of actions - ***read*** or ***write***
4. Maintain a data model based on the reads and writes to the database
	- Do not split it into too many tables (Keep it *denormalized*)
		- Okay to have a `user_id` pointing to a `User` table from a `Products` table
		- Not okay to split `Users` (*denormalized*) into `Users`, `UserAddresses`, `UserProfiles`, etc). This would lead to *LLD and wastes valuable time* 
	- Do not add additional attributes not relevant to the question (Ex: `User` table need not contain `user_address` unless it is needed for the question. *Tip*: Indicate additional fields as `...`)

**During this phase**:
- Feel ***free to discuss choices*** further *based on FRs* in this phase
- Identify the ***"black boxes"*** to be discussed next
	- Ex: If "Create short URL from long URL" is the action item on the server and it is *not trivial*, mention that you will address this soon (i.e design a function) and do so eventually!

Once you cover all the API endpoints, it means all our FRs are covered, and we can move on to the last phase (Deep dive)

## Deep Dive to scale our design

Here we go into the Non-functional requirements of our design and try to solve them!

**Important**: While solving these, you should do **'Back of the Envelope' estimations** here! (**Required / A must!**)

*Steps*:
1. Check the ***black boxes*** ("non-trivial") of our server actions in the HLD and map them to an existing NFR. ***Spend time solving them!***
	- Ex: "Create a short URL" is a server action. It may relate to a uniqueness quality attribute (NFR).
	- Discuss it further (needs to be fast, short, unique) and find approaches / algorithms that solve for it!
	- **Note: Think of questions along *speed*, *structure*, *complexity*, *time* (*to process*), and *length***
2. Check the remaining NFRs and based on *distributed* and *microservices* concepts solve for scale, reliability, availability, and so on

**NOTE**: When solving for scale, move from **Left-to-Right** i.e Start from the client, move to the server, then cache (Redis?), and finally the database! At each step, try to scale that component (**Easy process, methodical**)

Finally, ***after considering all FRs and NFRs and proposed solutions that handle them along with their trade-offs***, you have successfully **completed** your system design interview! Answer any follow-up questions that come

## Important HTTP verbs and status codes for API endpoints

**HTTP verbs**:
|Verb  |Usage           |Idempotent?|System Design Note (The "Why")                                                                                               |
|------|----------------|-----------|-----------------------------------------------------------------------------------------------------------------------------|
|GET   |Read data       |YES        |**Cacheability**. Easily cached by CDNs & Browsers                                                           |
|POST  |Create new data |NO         |Used for non-idempotent actions (e.g., Checkout) Risk: If the network fails, retrying might charge the user twice!        |
|PUT   |Update (***Replace***)|YES        |Replaces the entire entity. If you send `{name: "A"}` to a User profile, it wipes out the email field if you didn't include it |
|PATCH |Update (***Partial***)|Maybe      |Updates fields. Generally treated as idempotent, but technically increment counter `PATCH` isn't                              |
|DELETE|Delete data     |YES        |Deleting a deleted item still results in "Item is gone."                                                                     |

**HTTP statuses**:
|Code  |Name            |Meaning|System Design Context & Usage                                                                                                |
|------|----------------|-------|-----------------------------------------------------------------------------------------------------------------------------|
|200   |OK              |Success|Standard synchronous response                                                                                               |
|201   |Created         |Resource created|Result of a successful `POST`. Synchronous                                                                                    |
|**202**   |**Accepted**        |**Async processing**|Crucial: **Request queued (e.g., Kafka) but not finished**. Used for heavy jobs (video processing) to keep API latency low      |
|**301**   |**Moved Permanently**     |**New URL forever**|**Caching: Browser/CDN caches this indefinitely**. Used for HTTP → HTTPS or domain migrations                                   |
|**302**   |**Found**           |**Temporary Redirect**|**Routing: Client checks back next time**. Used for A/B testing or temporary maintenance routing                                |
|400   |Bad Request     |Client error|Malformed JSON or validation error                                                                                          |
|**401**   |**Unauthorized**    |**AuthN Failed**|**"Who are you?"** (Missing/Expired Token)                                                                                      |
|**403**   |**Forbidden**       |**AuthZ Failed**|**"I know you, but no."** (Valid User, insufficient permissions/scopes)                                                         |
|404   |Not Found       |Missing resource|Standard error                                                                                                              |
|**409**   |**Conflict**        |**State Conflict**|**Concurrency: DB constraint violation (e.g., duplicate username)**. Used in Optimistic Locking failures                        |
|**429**   |**Too Many Req**    |**Throttled**|**Protection: The Rate Limiter rejected the request**                                                                           |
|500   |Internal Error  |Server Crash|Unhandled exception in application code                                                                                     |
|502   |Bad Gateway     |Upstream Error|Infra: Load Balancer (Nginx/ALB) cannot reach the Backend. Usually means Backend is down/restarting                         |
|503   |Service Unavailable|Overloaded|Server is alive but too busy or in maintenance mode. Circuit Breaker might be open                                          |

## Data for Back Of The Envelope estimations

1. **Power of 10 hacks** (forget Power of 2 which is difficult to remember)

|Unit  |Value|Math Tip                                                                                                                     |
|------|-----|-----------------------------------------------------------------------------------------------------------------------------|
|1 KB (KiloByte) |1,000 (`10^3`B)|Thousand                                                                                                                     |
|1 MB (MegaByte) |1,000,000 (`10^6`B)|Million                                                                                                                      |
|1 GB (GigaByte) |1 Billion (`10^9`B)|Billion                                                                                                                      |
|1 TB (TeraByte) |1 Trillion (`10^12`B)|Trillion                                                                                                                    |
|1 PB (PetaByte) |1 Quadrillion (`10^15`B) |(Rarely used, but good to know)                                                                                              |

2. **The seconds conversion** (Forget actual calculator conversion)

| #Days | #Seconds |
| -- | -- |
| 1 Day | ~ **10^5** seconds. Use this for ***Daily Active Users (DAU)*** to ***QPS (queries per second) math*** | 
| 1 Month | ~ **2.5 x 10^6** seconds |
| 1 Year | ~ **3 x 10^7** seconds |

3. **Single Machine Constraints (The "Safe Limits")**: When asking *"Do I need distributed cache or just local memory?"*, use these conservative limits for a single modern server

|Resource|Constraint Limit|Usage in Design|
|--------|----------------|---------------|
|RAM (Memory on a single computer, like an EC2 instance)    |64 GB - 128 GB  |Can I store the User ID lookup table entirely in RAM? (100M users * 8 bytes = ~800MB. YES).|
|Network Bandwidth|1 Gbps          |Simple math: 1 Gbps≈100 MB/s.|
|Web Server QPS|5k - 10k QPS    |Can 1 server handle 1M DAU? (106/105 sec = 10 QPS. YES).|
|SQL DB (Write)|1k - 2k QPS     |Writes are heavy (ACID). If you need 50k Write QPS, you MUST shard.|
|Redis (Read)|100k QPS        |Redis is single-threaded but incredibly fast because it's in-memory.|

4. **Latency numbers**

|Operation|Time (Approx)   |Power of 10 (ns)|Human Scale Analogy|
|---------|----------------|----------------|-------------------|
|L1/L2 Cache|~1 ns           |100 ns          |1 Heartbeat        |
|RAM (Main Memory)|~100 ns         |102 ns          |Brushing teeth     |
|SSD (Flash)|~100 μs         |105 ns          |1 Day              |
|Network (Intra-DC)|~500 μs         |5×105 ns        |5 Days             |
|HDD (Disk Seek)|~10 ms          |107 ns          |4 Months           |
|Network (Cross-Region)|~150 ms         |1.5×108 ns      |5 Years            |

5. **UX Latency Constraints**
	- Map these to your Non-Functional Requirements

|Latency|User Perception |System Requirement|
|-------|----------------|------------------|
|< 100 ms|Instant         |Required for typing/gaming/search suggestions|
|< 1 s  |Uninterrupted   |Required for page loads/app transitions|
|> 10 s |Abandonment     |User closes the tab. *Needs async processing (Job Queue)*|

### QPS and storage capacities for different services

|Component|Throughput Limit (QPS/IOPS)|Storage/Size Limit (Per Node/Unit)|The "Why" (Constraint)|
|---------|---------------------------|----------------------------------|----------------------|
|EC2 (App Server)|1,000 - 5,000 req/s (`10^3`)          |64-128 GB (RAM)                       |CPU bound for logic; RAM bound for local state.|
|RDS (SQL DB)|1,000 Write/s (`10^3`)        |10 TB (`10^13`)                      |ACID transactions limit writes. ***Maintenance becomes painful >10TB***|
|NoSQL (DynamoDB)|1,000 Write/s (`10^3`)        |10 GB (`10^10`)                      |Per Partition Key. If one user exceeds this, you have a "Hot Partition."|
|S3 (Blob Store)|3,500 Write/s (`3.5×10^3`)    |5 TB (File Size) (`5 x 10^12`)                  |Per Prefix (Folder). Total storage is infinite, but single file is capped.|
|Redis (Cache)|100,000 req/s (`10^5`)        |100 GB (`10^11`)                     |In-memory speed. Cost/RAM is the storage bottleneck.|
|EBS (Disk)|10,000 IOPS (`10^4`)          |16 TB (`1.6 x 10^13`)                            |Standard SSD speed/size limits per volume.|

### Storage calculation example

- Example: "Calculate Storage for Chat App"
- Prompt: "Design WhatsApp. 1 Billion DAU. Each sends 10 messages. Save for 1 year."
- Traffic:`10^9` Users x `10` msgs = `10^10 msgs/day`
- Size per Msg: Assume 100 Bytes (text)
- Daily Storage = `10^10 x 100B` = `10^12B` = `1TB/day`
- 1 Year Storage: `1 TB x 365 = 365TB`
- Conclusion: *This fits easily into a standard Blob Store (S3), but requires sharding/partitioning metadata if using a SQL D*

### QPS calculation example

Formula: **`Average QPS = Total Requests per Day / Seconds in a Day`**

Note: There are ~`10^5` seconds in a day

- Numerator (`10^6`): 1 Million Daily Active Users (assuming conservatively 1 request per user)
- Denominator (`10^5`): 100,000 seconds in a day
- Result: `1,000,000 / 100,000 = 10 QPS`
