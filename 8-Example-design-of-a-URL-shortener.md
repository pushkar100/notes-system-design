# Example design of a URL shortener such as TinyURL or Bitly

*Problem: Design a URL shortening service*

[HelloInterview article](https://www.hellointerview.com/learn/system-design/problem-breakdowns/bitly)

## Table of Contents

- [Example design of a URL shortener such as TinyURL or Bitly](#example-design-of-a-url-shortener-such-as-tinyurl-or-bitly)
  - [Context](#context)
  - [Considerations if familiar with this question](#considerations-if-familiar-with-this-question)
  - [Building the function requirements](#building-the-function-requirements)
  - [Building the non-functional requirements](#building-the-non-functional-requirements)
    - [Building the core entities](#building-the-core-entities)
    - [Building the API endpoints](#building-the-api-endpoints)
    - [High level design](#high-level-design)
  - [Deep dive to handle quality attributes](#deep-dive-to-handle-quality-attributes)
    - [Handling creation of a unique short URL](#handling-creation-of-a-unique-short-url)
    - [Solving the second NFR of latency during redirection](#solving-the-second-nfr-of-latency-during-redirection)
    - [Solving the third NFR of scale](#solving-the-third-nfr-of-scale)
    - [Solving the last NFR of High Availability](#solving-the-last-nfr-of-high-availability)
  - [Final diagram](#final-diagram)

## Context

TinyURL is a URL shortening service that converts long URLs into short, unique links for easy sharing and tracking. Example: `https://www.example.com/very/long/path/to/resource` -> `https://tinyurl.com/abc123`

*Why it's Useful:*
- Compact Links: Perfect for platforms like Twitter and SMS
- Improved UX: Easy to share, clean, and readable
- Click Tracking: Analytics for link performance
- Branding: Custom domains for enterprises
- Cross-Channel Friendly: Works across emails, chats, and print

*How it Works:*
- User submits a long URL
- System generates a unique short key
- Stores mapping in database
- Redirects short URL to original

## Considerations if familiar with this question

These questions help uncover additional functional requirements, but they require *deep familiarity* with the problem (e.g., having built, used, or studied similar systems before)

- *Goal*: Use these questions to expand the Core Functional Requirements
- *Prerequisite*: Requires domain knowledge (i.e., you must know how the system works in the real world to ask them)

**The considerations**:
1. **Alias**: Can a user submitting a long URL also submit a custom short URL path? (Ex: `my_event_1`?)
2. **Expiry**: Can we set an expiry (Ex: default expiry) on the URL?

## Building the function requirements

Questions asked / Observations made:
- "Users can submit a long url and receive a short url"
- "Other users should be redirected to the long url when clicking on the short one"

Questions asked based on familiarity:
- "Do we need to allow setting of custom aliases for short URL paths?" "YES, optionally"
- "Do we need to allow setting of an expiry date for the URL?" "YES, optionally"

**Functional requirements (based on the questions/observations)**:
1. Create a short URL from a long URL
	- Short URL optionally supports an 'alias'
    - Short URL optionally supports an 'expiry'
2. Be redirected from the short URL to the long URL

## Building the non-functional requirements

Questions to ask for NFR:
- "Should the system have low-latency?" "YES" -- makes sense to keep redirection time short 
	- Reasoning: Maybe time to create a short URL can take longer but "redirection" latency needs to be less
	- Quantifying: A reasonable time to redirect is <200ms as that is near instant in terms of UX
- "Should the system be available or consistent?". "Available"
	- Reasoning: Consistency (strict) is not needed - OK for the system to take a minute or more to make the short URL - not a big loss/ come back in some time 
	- Ex: It takes about a minute or so to redirect. However, the system needs to be working i.e user can create short URLs and view them even if it might take a while to be redirect ready)
- "Should the system be scalable?". "YES". 
	- What kind of scale?
		- Interviewer: 100M DAU and 1B URLs stored
- "Any specific attributes of the URLs?". "YES, it makes sense to keep the short URLs unique to avoid conflict"

**Non-Functional Requirements based on the quality attributes**:
1. Low latency on redirects (~200ms)
2. High availability, eventual consistency for creating a short URL
3. Scale: 100M DAU and 1 Billion URLs' stored
4. Uniqueness of short URLs

### Building the core entities

Thinking about core entities:
- We need a **long url** (to submit)
- We need to maintain a **short url** (for the submitted long url)
-  We have a **user** who submits the long URL (and can also redirect via other short URLs)

Core entities:
1. Long url
2. Short url
3. User

### Building the API endpoints

There exists 1:1 mapping between an API endpoint and a Functional Requirement
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

### High level design

Mapping API endpoints to methods:
- Create a short URL: `createShortUrl(longUrl, alias?, expiry?)`
- Redirect to a long Url: `redirect(short)`

Corresponding server actions:
- `createShortUrl`: "Create a short URL"
- `redirect`: "Lookup the long URL"

Unusual HTTP status codes?
- Yes, needed for `redirect` response
- `301` permanent vs `302` temporary
	- Choice: `302`
	- Why? 
		- You: 
			- **`301` offers *cacheability* (browser/CDN), but** 
			- **`302`will allow us to *add analytics to our system*, *know it works since traffic passes through it* but might *add a bit more latency* than `301`**
		- Interviewer: Let's have visibility into it. Go with `302`

Data model created based on FRs:
1. `URLs` table: Start with one table
	- Add long URL
	- Add short URL or an optional Alias
	- Add expiration time (optional)
	- Add created at time (--> Needed for expiry)
	- `user_id`: Users can create URLs so map them using a foreign key

2. `Users` table
	- `id`
	- ... no other field is relevant ...

```
High Level Design:

                                     +-------------------------------+
                                     |             Server            |
                                     |                               |       /------------\
    +--------+   createShortUrl(...) | - "Create a short URL"        |      |            |
    |        | --------------------> | - "Lookup the long URL"       | <==> |  Database  |
    | Client |                       |                               |      |            |
    |        | <-------------------- |                               |       \------------/
    +--------+    redirect(...)      +-------------------------------+
        |
        |
  redirect() response:                                                       URLs table:
  302 temporary                                                              - shortURL / alias?
                                                                             - longURL
                                                                             - user_id
                                                                             - expiry?
                                                                             - creation_time

                                                                             Users table:
                                                                             - id
                                                                             - ...
```

## Deep dive to handle quality attributes

Starting with the non-trivial server action: **"Create a short URL"**

Think of questions along *speed*, *structure*, *complexity*, *time* (*to process*), and *length*

### Handling creation of a unique short URL

Which NFR does this relate to?
- The **"uniqueness"** attribute
	- You: How short / long does it need to be?
	- Interviewer: Short `5-7` digits
	- You: The conversion also needs to be fast enough

**Back of the envelope calculation**

- 1 billion urls = `10^9` urls. That is `9 + 1 = 10` characters (Ex: `1,000,000,000`)
- 10 characters > requirement of 5-7

Approaches to generate a shorter (`5-7` chars) unique, short url in a reasonably fast time:
1. ***Prefix the long URL*** (***Bad. Do not do this***!): *Not unique at all* as many prefixes can match! Too many collisions
2. ***Choose a random number and encode it***: 
	- Encoding is simply ***translating data into a different format*** so it can be safely transmitted or stored
	- It is ***reversible***. You can *always decode it back to the original data* without needing a secret password
	- Choose a random number between `0-10^9` and shrink it to lesser digits using `Base62` encoding. *Why?* `62^6 = 56Bn` which is more than sufficient to store URLs (We need only `1Bn` as per the NFR)
	- **Con**: Base62 encoding itself cannot have collisions. But, since we are generating a "random number", *there can be collisions*!

```
SCENARIO A: NO COLLISIONS (Counter)
[ DB ID: 101 ] --(Math)--> [ Base62: "1D" ]  (Always Unique)
[ DB ID: 102 ] --(Math)--> [ Base62: "1E" ]

SCENARIO B: COLLISIONS (Hashing/Random)
[ "google.com" ] --(MD5 Hash)--> [ 0xAB12... ] --(Base62)--> [ "u7x" ]
                                      ^
                                      |
[ "yahoo.com"  ] --(MD5 Hash)--> [ 0xAB12... ] --(Base62)--> [ "u7x" ]
                                      |
                          (COLLISION HAPPENED HERE)
```

**Base62 encoding**

Base62 is a method to make numbers **shorter** and **readable** by using more characters to represent them. Instead of just using 10 digits (`0-9`), we use 62 characters:
- `0-9` (10 digits)
- `a-z` (26 lowercase)
- `A-Z` (26 uppercase)

```
Input ID: 11157
           |
           v
   +-----------------+
   | DIVIDE BY 62    |
   +-----------------+
           |
   +-------+-------+
   |               |
Quotient       Remainder
  179             57  -----> [ Map to 'V' ]
   |               
   v               
 (Repeat)          
   |               
179 / 62           
   |               
   +-------+-------+
   |               |
Quotient       Remainder
   2              55  -----> [ Map to 'T' ]
   |
   v
 (Repeat)
   |
 2 / 62
   |
   +-------+-------+
   |               |
Quotient       Remainder
   0               2  -----> [ Map to '2' ]
   |
 (STOP)
 
 Result (Reversed): "2TV"
```

*Why use it in System Design?*
- **Compactness**: It **shrinks long** database **IDs** into short strings. A 10-digit ID (e.g., `1,000,000,000`) becomes just 6 characters in Base62. This is *critical for URL Shorteners* (like bit.ly) or *user-friendly IDs* (like YouTube video IDs)
- **URL Safe**: Unlike Base64, which uses *special characters* like `+` and `/` that can break URLs, Base62 uses only alphanumeric characters that work perfectly in any browser

3. ***Hash the URL and take the first 6 digits***: This too can have collisions (*will definitely have!*)

**Hashing**

Hashing is like taking a digital "fingerprint" of data. It turns any amount of input (a password, a file, a URL) into a *random-looking string of **fixed** length*

Key Rule: It is ***one-way***. You cannot turn the fingerprint back into the original person

Use cases:
- Data Integrity: To check if a file was corrupted during download (Checksums)
- Sharding/Load Balancing: To evenly distribute data across servers (Consistent Hashing uses the hash of an ID to pick a server)
- Security: Storing passwords so that even if the DB is stolen, the actual passwords aren't visible

Hashing tools:
|Algo  |Pros            |Cons |Best Use      |
|------|----------------|-----|--------------|
|MD5   |Extremely fast, short output.|Broken. Collisions are easy to find.|Non-security tasks (e.g., checksums for internal cache keys).|
|SHA-256|Highly secure, industry standard.|Slower to calculate than MD5.|Passwords, TLS Certificates, Blockchain.|

4. **Use a counter and encode it** [**CHOSEN APPROACH**]
	- Instead of a random number generator, increment a counter (that *grows linearly* from maybe `0` to `1Bn`)
	- Encode this number into `Base62`, thereby compacting the number of characters/digits
	- **Pros**: ***No collision*** since it is a *sequentially growing number*!
	- **Cons**: ***Predictable*** since the number grows sequentially. This is a security risk (Others can get all the short URLs i.e guess it)
		- ***Solution***: Use "**Bijective functions**" like **SQIDS** / **hashids**
			- Bijective functions guarantee 1:1 mapping but are still predictable
			- Sqids guarantee unpredictability by ***shuffling the mapping***

**Bijective functions**

The Simple Concept: "The Perfect Pair". A Bijective Function is a mapping where every element in `Set A` pairs up perfectly with exactly one element in `Set B`. No one is left out, and no one shares a partner

**SQIDS / Hashids**

- An open source library / tool
- It is Bijective (Reversible) because it is just math: it maps one number to one string using a specific rule
- It is "Unpredictable" (Obfuscated) because it changes the rule (shuffles the alphabet) for every single ID it processes

```
       STANDARD BASE62 (Predictable)
       Alphabet: [ 0, 1, 2, 3 ... ] (Fixed)
       
       ID: 1  ---->  "1"
       ID: 2  ---->  "2"  (Obvious Pattern)

       =======================================

       SQIDS (Obfuscated)
       
       ID: 1
         |--> (Step A) Calc Lottery Char: "X"
         |--> (Step B) Shuffle Alphabet using "X": [ 9, b, A, ... ]
         |--> (Step C) Encode "1" using New Alphabet -> "9"
         |--> Result: "X9"

       ID: 2
         |--> (Step A) Calc Lottery Char: "b"
         |--> (Step B) Shuffle Alphabet using "b": [ Q, 5, z, ... ]
         |--> (Step C) Encode "2" using New Alphabet -> "z"
         |--> Result: "bz" (Completely different!)
```

### Solving the second NFR of latency during redirection

NFR: "Low latency on redirects (~200ms)"

Redirects means:
1. Client uses the shortURL 
2. This triggers the redirect on the server
3. The server must "lookup" the shortURL and redirect (`302`) to the longURL

The "lookup" is a database read. ***We can optimize the database read***:
- Instead of a disk scan, add an index on the shortUrl (Primary key)
- If PK, it will have the lookup from memory and fetch the page & row directly (without scanning the disk)
- If not PK, we can still add a secondary index
- PK is stored as a B-Tree in PostgreSQL and is balanced: Hence, `O(n)` would get reduced to `O(log n)`

Reduce latency even further by adding an ***in-memory datastore like Redis*** (***caching service***):
- Reads: Redis is incredibly high on throughput (`~10^5` QPS)!
- We don't even have to go to the database server to fetch the data
- Redis is a "key-value" store
	- Our format: `key = shortUrl`, `value = longUrl`
- Strategy chosen:
	- **Write-through**: A caching strategy where every time you save data, you *write it to **both** the cache and the database simultaneously*
		- The Rule: The write is not considered "successful" until it is saved in both places
		- The Goal: Absolute consistency. You never lose data if the cache crashes because the database always has a copy
	- **LRU cache**: Optimize a cache-hit for reading the most recently read URLs to be saved. Ex: Redirection to an event link that a lot of people are opening
- **Pros**: Extremely fast reads
- **Cons**:
	- **Writes will be a bit slower** (Due to write-through): Okay since as per FR we want redirection to be fast
	- **Volatile**: Redis is an in-memory store (If it crashes, the cached data is lost)
		- Solution: "**Write through**" (But this makes writes slower but consistent - Okay because we want faster reads i.e redirection so a slower but consistent write is the trade-off against write availability)


*CDN caching?*
- Decision: Not using cache. (Yes, it does improve latency due to them being edge servers but that means traffic is permanently routed via CDN not us)
- Why? Same reason as using `302`. Traffic should flow through us for analytics, knowing our servers are working, etc.

###  Solving the third NFR of scale

NFR: "Scale: 1M DAU and 1B URLs' storage"

**BOTE calculations ***for traffic scale***:**
- `100M DAU = 100 x 10^6 = 10^8` DAU
- Assumption: 1 redirect per user per day
- `10^8 x 1 = 10^8` redirects per day (rpd) 
- `Average QPS = 10^8 / 10^5 = 10^3 = 1000` rps (Note: `10^5` seconds in a day)
- Assumption: Spikes can be **10x**
- Hence, load that needs to be handled is **`10,000 rps`**

A single EC2 instance handles around 5000 rps at max! Hence, we need to scale!

How to scale?
1. Vertical: No! Can be a single point of failure (SPOF)
2. Horizontal: YES! (Cheaper) -- Will need to split into a distributed architecture
	- Do we scale both reads and writes? NO! Writes can go to one service while the reads can go to another since they ***need to scale independently*** (Reads need to scale more than writes)
	- Solution: Micro-services behind an API gateway

New problem: The counter!
- All the instances carry a counter and they are not in-sync with each other!
- Solution: **A global counter using Redis**

**Redis for a global counter**

- The Concept: **Atomic Increments**
- The most efficient way to implement a global counter is using Redis's `INCR` command. Because Redis is ***single-threaded***, the `INCR` operation is **Atomic by default**
- This means if 1,000 requests hit the server at the exact same millisecond, ***Redis queues them and executes them one by one***. You do not need complex locks or race-condition handling in your application code
- Example
	- Scenario: Counting total views for a video `video_id:55`
	- Command: `INCR video_views:55`
	- Result: Redis returns the new value (e.g., `101`) immediately
- **Pros**: ***Extremely Fast*** (Memory-based), ***Thread-Safe*** (No race conditions), ***Simple*** (No "read-modify-write" logic needed).
- **Cons**: 
	1. ***Durability*** (If Redis crashes, you might lose the last few seconds of counts depending on persistence settings)
	2. ***Scale*** (A single "hot" key is limited to one Redis node's capacity) (`SPOF`)

**Solving durability in Redis**

- The Solution: Periodic Persistence
- Instead of writing to the database on every click (which is slow), we keep the high-speed counter in Redis and a *background worker updates the permanent database* (SQL/NoSQL) *periodically* (e.g., every 10 seconds)
- Redis: Handles the high-frequency increments (`INCR`)
- Database: Acts as the permanent "Save Point."
- The Trade-off: If Redis crashes, you typically lose only the data from the last "flush" interval (e.g., the last 10 seconds), not the entire history

```
 [ Users ]
     | (1) High Speed INCR
     v
+---------+         (2) Read every 10s       +----------+
|  Redis  | <------------------------------- |  Worker  |
| (1050)  |                                  |  Service |
+---------+                                  +----------+
                                                   |
                                                   | (3) Update "Save Point"
                                                   v
                                             +----------+
                                             | Database |
                                             | (1040)   |
                                             +----------+
```

**Solving scale for hot keys in Redis**

To scale writes beyond a single node, we break the single "hot" key into `N` smaller keys (e.g., `views:1`, `views:2` ... `views:10`)
- **Write (Scatter)**: When a user views the video, the app randomly picks one of these keys (e.g., `views:7`) and increments it. This spreads the write load across multiple Redis nodes in a cluster
- **Read (Gather)**: To get the total count, the app reads all `N` keys and sums their values together

**BOTE calculations ***for storage scale***:**
- Need to find size of 1Bn records x Size of each record i.e URLs table record size (avg) 
- `1Bn x (8 bytes short url + 100 bytes long url + ...)` = `~500Bn` bytes (Upper bound assumption)
- `500Bn bytes = 5 x 10^2 x 10^9 = 5 x 10^11 bytes = 500GB`
- Modern database servers support storage in the "terabytes"
- Hence, we are good. *No need to scale this database*
	- If we did need to scale the database (but, we don't need to in this case), we would have had to use ***sharding*** (by using the shortUrl as the key)

### Solving the last NFR of High Availability 

NFR: "High availability, eventual consistency for creating a short URL"

At this point, we are largely available DUE TO PREVIOUS OPTIMIZATIONS:
- Read service independent of write service
- API gateway to route traffic

What we can do *even more*:
- Add that worker service to periodically snapshot Redis and save the data to the DB
- Scale Redis clusters for the global counter too

## Final diagram

![fG6GBln.md.png](https://iili.io/fG6GBln.png)
