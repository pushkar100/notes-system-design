# Designing a Web Crawler

A web crawler is known as a robot or spider. It is widely used by search engines to discover new or updated content on the web. Content can be a web page, an image, a video, a PDF file, etc. A web crawler starts by collecting a few web pages and then follows links on those pages to collect new content

```
START: SEED URLs (The Roots)
--------------------------------------------------
      [ Seed URL A ]            [ Seed URL B ]
            |                          |
            | (Extract Links)          | (Extract Links)
            v                          v
LEVEL 1 (Found on Seeds)
--------------------------------------------------
    +-------+-------+                  |
    |               |                  |
[ Page A1 ]     [ Page A2 ]        [ Page B1 ]
    |                                  |
    v                                  v
LEVEL 2 (Found on Level 1)
--------------------------------------------------
    +                       +----------+----------+
    |                       |          |          |
[ Page A1.1 ]           [ Page B1.1] [Page B1.2] [Page B1.3] ... (Continues growing)
```

A crawler is used for many purposes:
1. **Search engine indexing**: This is the most common use case. A crawler collects web pages to create a local index for search engines. For example, Googlebot is the web crawler behind the Google search engine
2. **Web archiving**: This is the process of collecting information from the web to preserve data for future uses. For instance, many national libraries run crawlers to archive web sites. Notable examples are the US Library of Congress and the EU web archive
3. **Web mining**: The explosive growth of the web presents an unprecedented opportunity for data mining. Web mining helps to discover useful knowledge from the internet. For example, top financial firms use crawlers to download shareholder meetings and annual reports to learn key company initiatives
4. **Web monitoring**. The crawlers help to monitor copyright and trademark infringements over the Internet. For example, Digimarc utilizes crawlers to discover pirated works and reports

## Functional requirements

Questions to ask:
1. What?: *What is the primary use case for the crawled data?*
2. What?: *What content types need to be crawled? PDFs, images, and videos? Or just standard HTML?*
3. How?: *How do we handle web pages with duplicate content?*
4. Result?: (After crawling a page or site initially)
	- *Should we consider newly added to a site?* 
	- *Should we consider crawling edited web pages once they have been crawled initially?*
5. Where/Scale?: *Is the web crawler a separate system?* *Ans: Yes, it does not require a client or embedding, it is standalone*. **No formal FR required**

### Functional Requirements as a result of these questions

1. Search indexing is the primary use case (General-purpose search bot)
2. Only HTML content needs to be indexed and stored
3. Ignore pages with duplicate content
4. Crawl newly added pages of a website and crawl edited pages again
5. **Additional constraint**: Politeness of a crawler (Do not bombard a site/host with requests/seen as impolite or DoS attacks)

## Non-functional requirements

Questions to ask:
1. Scale (Throughput) and Volume (Overall storage or Bandwidth): 
	- *What is the rate of processing of pages target number of pages per month?*
	- *How long do we keep the data and what format do we store?*
2. Speed (Latency/CPU/Memory):
	- *How fast should we crawl pages? I know we have to be "polite". Define the rate-limiting complexity*
3. CAP trade-off (AP vs CP):
	- *Should the crawling be available or consistent?*
4. Safety net (Fault Tolerance):
	- *How should the system handle malformed HTML or infinite redirect loops?*
5. Geography:
	- *Does geography matter?*. *Ans: No, crawl based on seed urls and the hierarchy from there alone, which can be from any sites in the world*. **No formal NFR required**

### Non-Functional Requirements as a result of these questions

1. 1 billion pages in a month
2. Store the raw HTML for 5 years
3. Yes. The crawler must enforce politeness
4. A web crawler is typically an AP System (Availability)
	- *Why?*: It is better to accidentally crawl the same page twice (Consistency failure) than to stop the entire crawl just because a few servers lost connection (Availability failure)
5. The crawler should not crash or get stuck in infinite loops for malformed HTML

## Core entities 

1. URL
2. Page (HTML document)
3. Domain (We need to track metadata about the host to be "polite." We don't just crawl URLs; we crawl Domains)
4. Links (within crawled pages)

## System interface or API endpoints

System interface or API? We need an interface since it is a **Data Processing Pipeline**

High-level interfaces (Not exactly detailed for every component)
1. **`submitSeed(list<string> urls, string jobType) --> Provides a Job ID`** to inject initial seed URLs
2. **`getCrawlStatus(jobId) --> { "pages_crawled": 1.2M, "queue_size": 50k, "errors": 120 }`** to get detailed monitoring of the process

## High Level Design

1. Starting with MVP: Generally start with the  `client-server(database)` model
2. Go through *Functional Requirements* one by one and see if it is fulfilled. If not, modify design  (***Note: Need not go in order***)

Note: A web crawler is not a simple client-server model based system. It is a data engineering pipeline with various components. The minimum or basic components are listed below:
- **A) Seed URLs**: We feed the initial list into the system.
- **B) Frontier**: The "Manager" decides which URL is next for processing (can be based on priority)
- **C) HTML Downloader**: Fetches the actual page from the internet. Interacts with the DNS to convert the page link to an IP address
- **D) Parser and validator**: We scan the downloaded page to find other links (e.g., finding a link to bbc.com on cnn.com) and we also validate the content
- **E) Content storage**: We save the page content immediately
- **F) URL filter**: Filters out malicious / blacklisted links processed from the page (Security)
- **G) Deduplicator for seen content**: We check, "Have we already crawled bbc.com?" (Nearly 30% of sites are redundant)
- **H) URL storage**: Store the URLs processed in a database
```
                               (Start: submitSeed)
                                  |
                        +---------v----------+
                        |      Seed URLs     |
                        +---------+----------+
                                  | (1) Add to Queue
                                  v
+---------------------------------+---------------------------------+
|                           URL FRONTIER                            | <-- (getCrawlStatus) -- [ monitoring system ] 
|                  (The Queue & Scheduler)                          |
+---------+---------------------------------------------------------+
          |                                               ^
          | (2) Get Next URL                              | (7) Add New
          v                                               |     Links
+---------+----------+                                    |
|   HTML Downloader  |                                    |
|   (Fetch Webpage)  | <-- Address to IP -> [ DNS ]       |
+---------+----------+                                    |
          |                                               |
          | (3) Raw HTML                                  |
          v                                               |
+---------+----------+     (4) Save HTML        +---------+---------+
|   Content Parser   | -----------------------> |  Content Seen?    | --> [Content storage (DB / S3)]
|  + Link Extractor  |                          +-------------------+
+---------+----------+                          
          |
          | (5) Extracted Links
          v
+-------------+------------+
|        URL filter        |
| (Remove malicious sites) |
+-------------+------------+
          |
          v
+---------+----------+
|    Deduplicator    | (6) Filter out
|     (Seen URL?)    |     duplicates
+--------------------+
          |
          v
		[URL storage]
```

*FR#1: Search indexing is the primary use case (General-purpose search bot)*
- The system takes care of crawling URLs from seed and in turn, discovery. It saves the contents which can be indexed later

*FR#2: Only HTML content needs to be indexed and stored*
- Our content parser fetches the HTML only and stores that sanitized content (No other format)
- We can use an available parser or HTML to text converter, etc (Not compicated / out-of-scope)

*FR#3: Ignore pages with duplicate content*
- We have a component called "Content Seen?" responsible for detecting duplicate content saved in different page links. If duplicate: discard, else: save.
- Problem: To browse the entire page content and comparing it with pages we have is time consuming i.e *extremely slow*. Hence, not feasible!
	- Solution: **Compare only the hashes or checksums**
	- To check for duplicate content efficiently, we don't compare the text (which is slow and heavy). Instead, we create a ***digital fingerprint*** (Hash) of the content
	- The Process:
		1. **Hash It**: Pass the page content through an algorithm (like MD5 or SHA-256)
		2. **Shrink It**: This turns massive HTML into a tiny, fixed string (e.g., a5x9...)
		3. **Compare It**: You only compare this tiny string against your database of "seen strings." If it matches, the content is a duplicate
	- *Why this works*: Comparing two 32-character strings is millions of times faster than comparing two full paragraphs of text!
```
		PAGE A (New)                     PAGE B (Already in DB)
  [ "Hello World..." ]              [ "Hello World..." ]
          |                                  |
          v                                  v
    +-----------+                      +-----------+
    | MD5 HASH  |                      | MD5 HASH  |
    +-----------+                      +-----------+
          |                                  |
          v                                  v
   checksum: "8f7e2"                  checksum: "8f7e2"
          |                                  ^
          |                                  |
          +---------> [ COMPARING ] <--------+
                          |
                          v
                     IT'S A MATCH!
             (Content is Duplicate -> Discard)
```

*FR#4: Crawl newly added pages of a website and crawl edited pages again*
- **Problem 1**: Prevent re-crawling of *seen* pages i.e seen URLs
	- Solution: Check the URL in our database using our "URL Seen?" component
	- If seen, do not recrawl, do not add to the URL database
- **Problem 2**: How do we maintain ***freshness*** of an indexed web page?
	- We have a page indexed but its content has changed after that (Stale data)
	- *Solution*: ***Recrawl pages periodically based on update history*** (OR) ***re-crawl important URLs*** more frequently (Ex: Apple product page is more important than a blog post about the same product)
		- Achieved by sending these pages to the *"URL frontier"* component

*FR#5: Politeness of a crawler*
- We need to adhere to the **`Robots.txt`** of a website to see what is allowed to be crawled and what is not!
- Adding
	- (1) A ***delay*** between crawls of pages of the same host i.e Prevent DoSing it / overloading
	- (2) *Make sure to not only crawl from one host, too frequently, again overloading - Discussed more in the deep dive!*

```
    [ CRAWLER ]                       [ ROBOTS.TXT FILE ]
         |                           +-----------------------+
         | "Can I enter?"            | User-agent: *         |  <-- (Rule for Everyone)
         |-------------------------> | Disallow: /private    |  <-- (Do not enter here)
         |                           | Allow: /public        |  <-- (You can enter here)
         |                           +-----------------------+
         |                                      |
         v                                      v
   DECISION MATRIX:
   1. /public/index.html  -----------------> [ CRAWL IT ]
   2. /private/salary.pdf -----------------> [ IGNORE IT ]
```

## Deep dive

1. Starting with constructed High Level Design:
2. Go through *Non-Functional Requirements* one by one and see if it is fulfilled. If not, modify design (***Note: Need not go in order***)
3. Also check some unique constraints of the problem you might have missed (scan or read the problem again) and apply solutions for them

*NFR#3: The crawler must enforce politeness* (**Politeness**)
- How do we not overload or DoS a host?
- We need a handling mechanism inside the *URL frontier*
- Factors to consider:
	- **1) Algorithm to use**
	- **2) Politeness strategy** (***Delay*** between tasks for same host & Worker threads for each host leading to ***parallelization***)
	- **3) Prioritisation** of more important sites or pages (Ex: Apple product page more important than random article about Apple product) - Can be used for ***freshness** of a page (Feed from URL storage / service using the DB)

**Algorithm to use**
- Use **BFS** over DFS
- Reasons:
	1. ***Politeness***: DFS "hammers" a single server by requesting page after page rapidly. BFS spreads requests across many different servers/domains
	2. ***Priority***: The most important information (Homepages, Categories) is usually shallow (near the seed). DFS wastes time on deep, low-value pages
	3. ***Traps***: DFS can get stuck in infinite loops (e.g., infinite calendar dates). BFS naturally limits depth
```
     BFS (The Wide Net)                 DFS (The Deep Dive)
      "Scan Level by Level"             "Follow One Path Forever"

       [ SEED URL ]                      [ SEED URL ]
       /    |     \                        |
      /     |      \                       v
   (1) -- (2) -- (3)                     (1)
    |      |      |                        |
    v      v      v                        v
   (4)    (5)    (6)                     (2)
                                           |
   Result: Even coverage across            v
   many domains (Polite).                (3)
                                           |
                                           v
                                         (4)... (Trapped in deep path)

                                  Result: Hammers one domain (Impolite).
```

**Politeness**
- The Politeness architecture ensures that a crawler doesn't "hammer" a single server with hundreds of concurrent requests (which would look like a DDoS attack).
- Solution: ***one queue per host*** mechanism of the **URL Frontier**
	1. *Queue Router (The Sorter)*: It looks at the incoming URL's domain (e.g., wikipedia.org)
	2. *Mapping Table (The Directory)*: It tells the Router which specific queue is assigned to that domain (e.g., "Wikipedia goes to Queue #5")
	3. *FIFO Queues (One Queue = One Host)*: We create a separate queue for every single website we find
		- *Key Rule: Queue #5 contains only Wikipedia URLs. Queue #6 contains only CNN URLs*
	4. *Queue Selector (The Scheduler)*: It acts as a rotary switch. It picks a queue, grabs one URL, assigns it to a worker, and then waits (enforces a delay) before returning to that same queue
- *Why this works*: Because Queue #1 is purely Wikipedia, the Queue Selector can easily say: "I just took a job from Queue #1, so I am going to lock Queue #1 for 3 seconds." Meanwhile, it keeps working on Queue #2 and Queue #3. This maximizes speed without overloading any single specific server.
```
 [ Incoming URLs ]
       |
       v
 [ QUEUE ROUTER ] <---- (Checks Mapping Table: "Wiki -> Q1", "CNN -> Q2")
       |
       +-------------------------+
       |                         |
       v                         v
 [ Queue #1 ]              [ Queue #2 ] ... (Hundreds of Queues)
 (Only Wiki)               (Only CNN)
       |                         |
       +-----------+-------------+
                   |
                   v
          [ QUEUE SELECTOR ]  <-- (Round Robin + Time Delays)
                   |
                   | "Okay, Q1 is ready. Q2 must wait 2s."
                   v
          [ WORKER THREAD ]
                   |
                   v
          (Downloads Page)
```

**Priority**
- ***Prioritizer***: It analyzes the input URL (based on PageRank, update frequency, etc.) and assigns a priority level (e.g., High, Medium, Low).
	- *Front Queues* (F1 to Fn): These are storage buckets
		- F1: Stores High Priority URLs
		- Fn: Stores Low Priority URLs
	- *Front Queue Selector*: This acts as a biased scheduler. It does **not** treat queues equally. It might pull 70 URLs from F1 for every 10 it pulls from Fn
	- *Back Queue Router*: Once a URL is picked by the Front Selector, this router catches it and sends it to the "Politeness" system (the specific queue for that website)
```
     [ Incoming URLs ]
             |
             v
      [ PRIORITIZER ]  <-- (Judges URL: "CNN is High", "Bob's Blog is Low")
             |
    +--------+--------+
    |                 |
    v                 v
[ Queue F1 ]      [ Queue Fn ]  <-- (Front Queues store by Importance)
 (High Pri)        (Low Pri)
    |                 |
    +--------+--------+
             |
             v
   [ FRONT QUEUE SELECTOR ]  <-- (Biased Logic: Picks F1 70% of the time)
             |
             v
    [ BACK QUEUE ROUTER ]    <-- (Hand-off to Politeness System)
             |
             v
     (To Host-Specific Queues)
```


*NFR#1: 1 billion pages in a month* AND *NFR#2: Store the raw HTML for 5 years*
- **B.O.T.E Calculations** requited
- Throughput:
	- 1B pages / 30 days / 24 hours / 3600 sec ≈ ~400 pages per second. 
	- Peak traffic might be 2x, so design for ~800 - **1,000 QPS**
	- Generally, any database Relational or NoSQL can handle this! In-memory stores like Redis or ***hDFS (Ex: Hadoop) can handle more than this too***. Make a choice based on the storage requirements
- Massive Storage Capacity
	- 1B pages x 100KB (avg size) = 100 x 100 x 1000 B per month = **100 TB per month**
	- 100TB x 12 months x 5 years = **6 Petabytes (PB)**
	- Implication: You cannot use a standard database (*Relational* ~10TB, *NoSQL* ~10GB per partition - need to have way too many partitions to solve for 100TB)
		- You need a ***distributed file system like S3, HDFS, or GFS*** for the content store
- *Note*: Storage volume for "Frontier" and "URL store" will not be as massive as the content store. We can choose a relational DB/NoSQL store with memtable/index in front of them and add a caching layer if required

*NFR#4: A web crawler is typically an AP System (Availability)*
- if a worker cannot verify if a URL has already been visited (e.g., the "Seen URLs" cache is down), it defaults to crawling the page anyway to ensure the system keeps moving, relying on the underlying database to silently discard the duplicate data later (eventual consistency)
```
      [ Worker ]
          |
          v
 [ Dedup Service (DOWN) ]   <-- Network Partition happens
          |
          v
   "CRAWL ANYWAY"           <-- Choose Availability (Don't stop)
          |
          v
 [ Database / Storage ]     <-- Solves Consistency later (Idempotent write)
```

*NFR#5: Fault tolerance: The crawler should not crash or get stuck in infinite loops for malformed HTML*
- **Handling failures**: Have timeout if page does not respind 
- **Handling node failures**
	- Use ***replication*** (Mostly need leaderless/active-active solutions) per replica,
	- Use ***Sharding*** for the content store (Huge data) if that is allowed in a HDFS setup

## Further optimisations

**Optimizing the HTML downloader**
- Once the workers pick up URLs from the URL frontier and pass it off to the HTML downloader, we can optimize it by doing the following:
	1. Only crawl pages allowed as per the **`robots.txt`** file for a host
	2. Scale out crawls by using a ***distributed system and a load balancer***: Multiple nodes (leaderless/active-active)
	3. **Cache the DNS resolver**: DNS resolution involves network calls which can be a performance bottleneck if we do it repeatedly. Instead, use a cache for hosts that have been recently accessed or are currently being crawled. Update it periodically from the DNS server

**Robustness**
- Consistent hashing
- Save crawl states and data if HTML content parser & validator fails midways (Do not have to restart)
- Exception handling: HTML downloader must add retries and fail gracefully

**Extensibility**
- Create the parser & validator service in such a way that future media (Images, PDF files, etc) can be indexed and stored

**Use Server-Side Rendering (SSR) for Dynamic webpages (SPAs)**
- Cannot crawl HTML of SPAs
- Wait for JS to load which in-turn loads the page content
- Perform SSR (Server-side rendering) to wait until JS executes and loads all the page data -> Then parse and index it!

**Avoid Spider traps**
- *Trap*: A spider trap is a website structure that generates infinite unique URLs (like an endless calendar), causing the crawler to get stuck in an endless loop
- *Solution*: We solve this by enforcing strict limits, such as a maximum URL character length or a maximum folder depth, to force the crawler to stop following the path
```
THE CALENDAR TRAP
[ Start: /jan-01 ]
      |
      v
  [ /jan-02 ]
      |
      v
  [ /jan-03 ]
      |
      v
   ... (Infinite "Next Day" links) ...
      |
      v
 [ /year-3000 ]  <-- Crawler is stuck here forever.
```
