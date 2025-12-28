# Rate limiter

A rate limiter controls how many requests a client can make within a specific timeframe. It acts like a traffic controller for your API - allowing, for example, 100 requests per minute from a user, then rejecting excess requests with an HTTP 429 "Too Many Requests" response. Rate limiters prevent abuse, protect your servers from being overwhelmed by bursts of traffic (DoS/DDoS), and ensure fair usage across all users

Benefits:
- Prevent resource starvation caused by Denial of Service (DoS)
- Reduce cost. Limiting excess requests
- Prevent servers from being overloaded

Throttling / limiting mechanism:
- Throttling is the process of controlling the usage of the APIs by customers during a given period
- Throttling can be defined at the application level and/or API level
- When a throttle limit is crossed, the server returns HTTP status “429 - Too many requests"

## Hard parts

1. **Concurrency**: Race conditions when two requests update count at once
2. **Distributed State**: Sharing counters across different Rate Limiter nodes

## Functional requirements

Questions to ask:
1. Who/What: *What kind of rate limiter do we use? Client or Server-side?*
2. How: *How should the client be rate limited (Ex: based on IP/Key?). Should this be configured?*
3. How: *How should we rate limit? i.e Appropriate rate limiting policy / method / algorithm?*
4. Result: *What should happen when a client exceeds the rate limit? Do we inform them?*
5. Where/Scale: *Is this rate limiter used at a small startup (i.e less traffic) or a big company (i.e more traffic/storage)? If this is a large system then is it right to assume it will be distributed?*
6. Where/scale?: *Is the rate limiter a separate service or should it be implemented in application code?*

### Functional Requirements as a result of these questions

1. Server-side rate limiter ***always*** (*Why?* Client side can be spoofed)
2. Uniquely identify client based on (a) Client IP (b) User ID (c ) Client API Key, done via  configuration of *rules* for each
3. Rate limiter should be *flexible enough* to support different throttling mechanisms
4. Inform the client with a 429 too many requests header when rate limits exceed for them
5. The system must be able to handle a large number of requests (Distributed system)
6. Up to use to decide if the Rate Limiter service needs to be separate or combined

### Usually out-of-scope requirements for rate limiting

- Complex querying or analytics on rate limit data
- Long-term persistence of rate limiting data

## Non-functional requirements

Questions to ask:
1. Scale (Throughput) and volume (Storage - Overall or Bandwidth based): 
	- *What should be the rate at which Client must be limited (limit per sec or per min?)*
2. Speed (Latency/CPU/Memory):
	- *What is the overhead on a service every time we apply the rate limit check on a request?*
3. CAP trade-off (AP vs CP):
	- *Is it okay to have slight delays in enforcing the rules?*
4. Safety net (Fault Tolerance):
	- *What happens if the rate limiter crashes? Do we fail open/close*
5. Geography:
	- *Does the rate limiting need to also be limited by region? Ex: Asia/US*

### Non-Functional Requirements as a result of these questions

1. User is rate limited at 1 Million reqs/sec at 100M DAU
2. Rate limiting should take less than 10ms per request as overhead
3. We need a highly available system. Hence, a few minutes delay in enforcing new rules is okay
4. We need a highly fault tolerant system. Hence, any problems with rate limiter, we should allow the system to function (i.e Fail open)
5. Rate limiting is global (Rate limiting for user is same if he is logged in from US or from Asia)

## Core entities 

Objects and users persisted in the database or exchanged via APIs:

- Clients (A user)
- Rules
- Request

## System interface or API endpoints

System interface makes more sense since API endpoints are not restricted (i.e rate limiting happens on every request)

All 7 functional requirements are satisfied by just one interface

We need one interface, the one that rate limits:
1. **`rateLimit(clientDetail, ruleId)`**
	- Success: Allow (`2xx`)
	- Reject: `429` too many reqs
	- Response payload: `{ passed: boolean, remainingRequests: boolean, resetTimestamp: DateTime}` (We want to help client know the status, how many more reqs in a time period, and when the counter resets)

```
clientDetail {
	userId,
	clientIP,
	apiKey
}
```

## High Level Design 

1. Starting with MVP: Client-Server model (`client - server - database`)
2. Go through *Functional Requirements* one by one and see if it is fulfilled. If not, modify design  (***Note: Need not go in order***)

```
 [client] -> [Server] -> [database]
```

*FR 1: Server side RL ✅*

*FR 4: Inform the client with a 429 -- Our interface already sends that ✅*
- Also add some helpful headers (Ex: `X-RateLimit-Limit`: The rate limit ceiling for that request (e.g., "100"), `X-RateLimit-Remaining`: Number of requests left in the current window (e.g., "0"), `X-RateLimit-Reset`)

*FR 5: Work on a large distributed system* AND *FR 6 Where should the rate limiter be?*:
1. Bad approach: On each service! Why? 
	- Service needs to no only handle biz logic but also need to do this
	- Unnecessary resource utilization on application servers
	- Not a global approach (Bad for distributed systems): Servers don't know limits on each other. Hence, difficult to rate limit globally
2. Good approach: A separate service. Why?
	- Separation of concerns
	- Can be scaled independently of application services
	- Cons: 
		- A new point of failure
		- Latency: Every service needs to make a call to this new rate limiting service
3. **[Chosen]** Great solution - **API Gateway (Load balancer)**:
	- All requests pass through the API Gateway which is already doing infra related work (Authentication, Request Translation, etc)
	- Hence, it can be a *process* within the API gateway
	- API Gateways are extensively used in distributed systems (to route requests based on L4/L7)
	- Challenge: It cannot see the business logic of the request

*FR 2: Uniquely identify clients in order to configure rate limiting*:
- Since we have chosen an API gateway, it cannot /should not see the business logic of the request but can inspect the L4/L7 loads
- *Chosen mechanism*: All client identification and response etc should be through *HTTP Headers*
	- `Authorization` header
	- `X-API-Key`
	- IP Address
- Where should the ***rules*** reside?
	- A ***separate service***. Can be lightweight e.g Redis
	- Why? It is configurable! Hence, better to have it as a managed service that the API Gateway can update the rules from every now and then
	- Use a ***push based strategy*** (i.e API gateway is informed via a webhook when rules changes)
		- Why? *Less delay* in updating the rules on the Gateway (and it need not keep a tab by polling)

*FR 3: Different throttling mechanisms*
- We can have a different throttling mechanism configured per ID/IP/Key
  - We maintain a **counter** for each (Ex: count for `user_id_123 = 40`)
- Mechanisms:
	1. ***Token bucket algorithm*** (Good for bursts or sudden spikes)
	2. ***Leaky bucket algorithm*** (Good for a consistent stream of request)
	3. ***Fixed window counter*** (Not more than X requests in a time frame. *Con*: Edge case of getting *2x reqs* at the edges of a window)
	4. ***Sliding window log*** (Very accurate. Checks requests from current till start of window. *Con*: *Memory inefficent* as it needs to keep track of every request in the last window timeframe)
	5. **Sliding window counter**: *Great approximation* and *memory efficient*  ✅ (Fixed windows, saves number of requests in the last window, when req arrives, checks the amount of time in current window and takes a weighted average of the previous window i.e `number of reqs = (100 - % of current window) * #reqs in prev window + #reqs so far in current window`))

```
                                       +-------------------+
                                       |   Rules Cache     |
                                       | (e.g., "100/min") |
                                       +---------+---------+
                                                 ^
                                                 | (Read Config)
                                                 |
        +--------+                     +---------+---------+
        |        |   (1) Request       |                   |
        | Client +-------------------> |    API GATEWAY    |
        |        |   (w/ Headers)      |   (Middleware)    |
        |        | <-------------------+                   |
        +--------+   (4) Response      +----+----+----+----+
                     (w/ Headers)           |    |    ^
                                            |    |    |
                   (2) Incr & Check         |    |    |
                       (Sliding Window)     |    |    |
                       +--------------------+    |    |
                       |                         |    |
                       v                         |    |
             +---------+---------+               |    |
             |  Redis Cluster    |               |    |
             | (Distrib. Counts) |               |    |
             +-------------------+               |    |
                                                 |    |
                       (3a) IF LIMIT EXCEEDED:   |    |
                            Return 429 to Client |    |
                            (Short-circuit)      |    |
                                                 |    |
                       (3b) IF ALLOWED:          |    |
                            Forward Request      |    |
                                                 v    |
                                       +---------+----+----+
                                       |                   |
                                       |    App Service    |
                                       |   (Order/User)    |
                                       |                   |
                                       +-------------------+
```

## Deep dive

1. Starting with constructed High Level Design:
2. Go through *Non-Functional Requirements* one by one and see if it is fulfilled. If not, modify design (***Note: Need not go in order***)
3. Also check some unique constraints of the problem you might have missed (scan or read the problem again) and apply solutions for them

*NFR #1: User is rate limited at 2 reqs/sec at 100M DAU*
- B.O.T.E calculations
	- Throughput: `QPR = 1M req/sec`
		- Let us say this is average. Peak is `3x` or **3M req/sec**
- Normal RDBMS/NoSQL: ~1000-5000 req/sec
- Redis: ~100,000 req/sec
	- We will use **Redis** to handle this load -- Extremely fast!
		- `INCR` and `EXPIRE`
			- `INCR`: It increases the stored counter by 1
			- `EXPIRE`: It sets a timeout for the counter. If the timeout expires, the counter is automatically deleted
	- However, we will need a **Redis Cluster (10-20 instances)**
		- Also useful for the High-Availability aspect below
- 100M DAU: Each user may have rate limiting data associated
	- Let's say we need User id, IP, key (~10B) to be stored
	- `10Bytes * 100M` = `10^9 B` = `1GB` storage per day
	- How many years? Assume 1 month - More than enough since rate limting is at a very minute timeframe level (seconds, minutes, days) *(Discuss with interviewer)*
	- `1 * 20 * 1GB` = `30 x 1GB` = `30GB`
	- ***Redis clusters can easily handle 500GB***: (However, if we needed to scale the storage, we would have used ***sharding*** along with ***consistent hashing***. *Note*: Redis has *preconfigured **slots*** for consistent hashing (~16k slots) for faster hashing)

*NFR #2: Rate limiting should take less than 10ms per request as overhead*
- Redis works in-memory: `<100ns` just for an operation. Assume 1000 operations just to rate limit, it should still be `10^5 x 10^-9 s` = `10^-4 s` = `0.1ms` and network travel time should be still in a few msecs. We might just be good
- How can we speed up Redis?? 
	1. Add **TCP Connection pooling** since requests to it will be highly frequent so no point opening and closing connections repeatedly
	2. **Deploy API Gateways across different geographies** so that rate limiting happens closer to the edges themselves

*NFR #3: We need a highly available system. Hence, a few minutes delay in enforcing new rules is okay*
- For rules, a push based approach works fine
	- The same Redis service that does the rate limit check can be used to push rules to cache in API gateway. Slight delay is okay as per FR
- Again, since it is Redis, we will need a ***Redis cluster***
- Cluster management for *High Availability*:
	- **Replication** is needed. Redis does it using the ***Master-Slave (Leader based replication)***
		- You run a Primary node and one or more Replicas
		- Writes: Only goes to the Primary i.e **Counter** is written to the master only! (Counter can be read by others though) 
		- Reads: Can be distributed across Replicas to handle heavy traffic
		- If Primary dies, a Replica is promoted to keep the system alive
	
*NFR #4: We need a highly fault tolerant system. Hence, any problems with rate limiter, we should allow the system to function (i.e Fail open)*
- Since we allow "fail open", we can bypass the rate limiter, in case of a failure, from the API gateway
- Monitoring and recovery: 
	- We'll want to track Redis health metrics like CPU usage, memory consumption, and network connectivity across all shards. 
	- We also need application-level monitoring of rate limiting success rates, response latencies, and alerts that trigger when we enter fail-open mode. The goal is detecting and responding to issues quickly enough that users don't experience degraded service

*Rate limiting is global (Rate limiting for user is same if he is logged in from US or from Asia)*
- One API Gateway: Already globally rate limited
- Multiple API Gateways (Ex: One per region)
	- All of them have to talk to the Redis Cluster
	- This cluster is global (Since we do not rate limit per region as per requirement)
		- How do we increase availability? Perhaps asynchronously poll the Redis cluster for remaining requests in a window (Say, every 20 seconds) and store it. Since it is fail open, some let throughs should be fine

```
                                GLOBAL TRAFFIC
                                      ||
             +------------------------++-------------------------+
             |                        ||                         |
      [Region: US]              [Region: Asia]            [Region: EU]
             |                        |                          |
             v                        v                          v
 +------------------------+  +------------------------+  +------------------------+
 |     US API GATEWAY     |  |    ASIA API GATEWAY    |  |     EU API GATEWAY     |
 |                        |  |                        |  |                        |
 |  +------------------+  |  |  +------------------+  |  |  +------------------+  |
 |  |  Rules Cache     | <------+  Rules Cache     | <------+  Rules Cache     |  |
 |  | (Local / Pushed) |  |  |  | (Local / Pushed) |  |  |  | (Local / Pushed) |  |
 |  +--------^---------+  |  |  +--------^---------+  |  |  +--------^---------+  |
 |           |            |  |           |            |  |           |            |
 |  +--------+---------+  |  |  +--------+---------+  |  |  +--------+---------+  |
 |  |  Circuit Breaker |  |  |  |  Circuit Breaker |  |  |  |  Circuit Breaker |  |
 |  | (Fail Open Logic)|  |  |  | (Fail Open Logic)|  |  |  | (Fail Open Logic)|  |
 |  +--------+---------+  |  |  +--------+---------+  |  |  +--------+---------+  |
 |           |            |  |           |            |  |           |            |
 +-----------+------------+  +-----------+------------+  +-----------+------------+
             |                           |                           |
             | (TCP Conn Pool)           | (TCP Conn Pool)           | (TCP Conn Pool)
             |                           |                           |
             +-------------+      +------+      +--------------------+
                           |      |             |
                           v      v             v
                  +-----------------------------------------+
                  |         GLOBAL REDIS CLUSTER            |
                  |     (Consistent Hashing / Slots)        |
                  +-----------------------------------------+
                  |                                         |
        +---------+---------+                     +---------+---------+
        |     SHARD 1       |                     |     SHARD N       |
        |   (Users A-M)     |                     |   (Users N-Z)     |
        +-------------------+                     +-------------------+
        |                   |                     |                   |
  +-----v------+      +-----v------+        +-----v------+      +-----v------+
  |  MASTER    |----->|   SLAVE    |        |  MASTER    |----->|   SLAVE    |
  | (Writes)   | Sync | (Read/Fail)|        | (Writes)   | Sync | (Read/Fail)|
  +------------+      +------------+        +------------+      +------------+
```

### Wondering about unique constraints to the question

**If Rate Limiter i.e Redis was SHARDED:** Since rate limiting can be done on user IDs, **How do we handle "hot keys"?**
- Example: A particular user ID bombards requests over and over. Yes, we block him but how do we not let one Redis shard take the heat until the user is blocked? (`user id -> consistently hashed -> hits Redis shard A only -> Shard is locked/hardware wears out over time`)
- *Approach 1*: Use random prefix/suffix attachment. Ex: `<user_id>_1`, `<user_id_2` so that different shards store the limits
	- Drawback: We need to read the total / accurate requests for that user by collecting requests so far from different shards. That is okay if the system needs to be highly available but not consistent (Few seconds delay in knowing limit has been reached, only for this hot key, is okay)
- *Approach 2*: Reduce the limits per user or particularly for that malicious user. Limit is reached much sooner and user is blocked even before the shard is hit

**Hard part: How can we handle concurrenmt writes to Redis by two immediate requests? i.e Race Condition**
- We need either (1) Locking mechanism or (2) Conflict free write/read method
- Luckily, Redis can be used with a tool called **LuaScript** that guarantees a atomicity for a series of operations (Transaction)
  - We can also use **sorted sets data structure** in Redis
 
## Practical rate limiting

- https://newsletter.systemdesign.one/p/rate-limiter
- https://medium.com/@linz07m/ubers-go-rate-limiter-a-leaky-bucket-implementation-20639db69982
