
# Software Architecture of Large Scale Systems

[Course link](https://www.udemy.com/course/software-architecture-design-of-modern-large-scale-systems)

## Table of Contents

- [Software Architecture of Large Scale Systems](#software-architecture-of-large-scale-systems)
  - [Table of Contents](#table-of-contents)
  - [Definition of software architecture](#definition-of-software-architecture)
  - [Levels of abstraction](#levels-of-abstraction)
  - [Advantages of distributed services](#advantages-of-distributed-services)
  - [Software architecture position in the SDLC](#software-architecture-position-in-the-sdlc)
  - [Challenges of software architecture](#challenges-of-software-architecture)
  - [Gathering requirements](#gathering-requirements)
    - [Importance of gathering requirements](#importance-of-gathering-requirements)
    - [Types of requirements](#types-of-requirements)
      - [Functional requirements](#functional-requirements)
      - [Non-Functional requirements](#non-functional-requirements)
      - [Limitations and boundaries](#limitations-and-boundaries)
    - [Right way to gather functional requirements](#right-way-to-gather-functional-requirements)
      - [Formal way of gathering requirements:](#formal-way-of-gathering-requirements)
      - [Steps to gather requirements:](#steps-to-gather-requirements)
    - [More on non-functional requirements](#more-on-non-functional-requirements)
      - [Non-functional requirements need to be proven](#non-functional-requirements-need-to-be-proven)
      - [Last consideration is feasibility](#last-consideration-is-feasibility)
    - [System constraints](#system-constraints)
      - [Technical constraints](#technical-constraints)
      - [Business constraints](#business-constraints)
      - [Legal/Regulatory constraints](#legalregulatory-constraints)
      - [Accepting vs pushing back on constraints](#accepting-vs-pushing-back-on-constraints)
  - [Performance](#performance)
    - [Response time](#response-time)
    - [Measuring response time distribution](#measuring-response-time-distribution)
      - [Tail latency in response times](#tail-latency-in-response-times)
    - [Throughput](#throughput)
    - [Identifying performance degradation](#identifying-performance-degradation)
  - [Scalability](#scalability)
    - [Understanding load](#understanding-load)
    - [Intuition for a scalable design](#intuition-for-a-scalable-design)
    - [Note on linear scalability](#note-on-linear-scalability)
    - [Types of scalability](#types-of-scalability)
      - [Vertical scalability](#vertical-scalability)
      - [Horizontal scalability](#horizontal-scalability)
      - [Pros and cons of each scalability choice](#pros-and-cons-of-each-scalability-choice)
      - [Team / Organisational scalability](#team-organisational-scalability)
  - [Availability](#availability)
    - [Formula of availability](#formula-of-availability)
    - [Alternative formula of availability](#alternative-formula-of-availability)
    - [Myth of complete availability](#myth-of-complete-availability)
    - [The 9s of availability](#the-9s-of-availability)
    - [Fault tolerance](#fault-tolerance)
      - [Failure prevention](#failure-prevention)
      - [Types of redundancy](#types-of-redundancy)
      - [Strategies for redundancy](#strategies-for-redundancy)
      - [Failure detection](#failure-detection)
      - [Failure recovery](#failure-recovery)
    - [SLAs SLOs and SLIs](#slas-slos-and-slis)
      - [SLA or Service Level Agreement](#sla-or-service-level-agreement)
      - [SLOs or Service Level Objectives](#slos-or-service-level-objectives)
      - [SLIs or Service Level Indicators](#slis-or-service-level-indicators)
      - [Important considerations](#important-considerations)
  - [APIs](#apis)
    - [Categories of APIs](#categories-of-apis)
    - [API benefits](#api-benefits)
    - [API best practices](#api-best-practices)
  - [Remote Procedure Calls](#remote-procedure-calls)
    - [RESTful  APIs](#restful-apis)
      - [REST API quality attributes (Non-functional requirements)](#rest-api-quality-attributes-non-functional-requirements)
      - [REST API endpoints](#rest-api-endpoints)
      - [Best practices for resources](#best-practices-for-resources)
      - [CRUD operations via HTTP verbs](#crud-operations-via-http-verbs)
      - [REST API step by step process](#rest-api-step-by-step-process)
    - [Why is gRPC popular?](#why-is-grpc-popular)
  - [Architectural blocks of large scale systems](#architectural-blocks-of-large-scale-systems)
    - [Load balancer](#load-balancer)
      - [Benefit of using a load balancer](#benefit-of-using-a-load-balancer)
      - [Quality attributes of a load balancer](#quality-attributes-of-a-load-balancer)
      - [Types of load balancers](#types-of-load-balancers)
      - [DNS load balancer](#dns-load-balancer)
      - [Hardware and software load balancers](#hardware-and-software-load-balancers)
      - [Global Server Load Balancing](#global-server-load-balancing)
      - [Common load balancer tools](#common-load-balancer-tools)
    - [Message brokers](#message-brokers)
      - [Quality attributes of message brokers](#quality-attributes-of-message-brokers)
      - [Popular message broker tools](#popular-message-broker-tools)
    - [API gateways](#api-gateways)
      - [Popular API gateways](#popular-api-gateways)
      - [API Gateway vs Load Balancer](#api-gateway-vs-load-balancer)
    - [Content Delivery Networks](#content-delivery-networks)
      - [Popular CDN providers](#popular-cdn-providers)
  - [Data storage at global scale](#data-storage-at-global-scale)
    - [Relational databases](#relational-databases)
      - [Database schema](#database-schema)
      - [Structured Query Language](#structured-query-language)
      - [Rise of Relational databases](#rise-of-relational-databases)
      - [Advantages of Relational databases](#advantages-of-relational-databases)
      - [Properties of  Relational databases](#properties-of-relational-databases)
      - [Drawbacks of Relational databases](#drawbacks-of-relational-databases)
    - [Non-relational databases](#non-relational-databases)
    - [Key-Value Stores](#key-value-stores)
  - [Document Stores](#document-stores)
    - [Graph Databases](#graph-databases)
    - [Database optimization techniques](#database-optimization-techniques)
      - [Technique 1 - Database Indexing](#technique-1-database-indexing)
      - [Technique 2 - Database Replication](#technique-2-database-replication)
      - [Technique 3 - Database Sharding or Partitioning](#technique-3-database-sharding-or-partitioning)
    - [CAP theorem](#cap-theorem)
    - [Scalable unstructured data storage](#scalable-unstructured-data-storage)
      - [Storage solution 1 - Distributed File System or DFS](#storage-solution-1-distributed-file-system-or-dfs)
      - [Storage solution 2 - Object stores](#storage-solution-2-object-stores)
  - [Software architectural patterns](#software-architectural-patterns)
    - [Three-Tier Architecture](#three-tier-architecture)
      - [The Core Concept](#the-core-concept)
      - [The "Monolith" Connection](#the-monolith-connection)
      - [Pros and cons](#pros-and-cons)
    - [Microservices architecture](#microservices-architecture)
      - [The Architecture (The Layers)](#the-architecture-the-layers)
      - [Why is it Needed?](#why-is-it-needed)
      - [Pros and cons](#pros-and-cons)
      - [One database per microservice](#one-database-per-microservice)
      - [Real World Use Case](#real-world-use-case)
      - [Summary](#summary)
    - [Event-Driven Architecture](#event-driven-architecture)
      - [Why is it needed?](#why-is-it-needed)
      - [Why do Microservices use it?](#why-do-microservices-use-it)
    - [Event-Driven Patterns](#event-driven-patterns)
      - [The SAGA Pattern or the Undo button](#the-saga-pattern-or-the-undo-button)
      - [CQRS or Command Query Responsibility Segregation](#cqrs-or-command-query-responsibility-segregation)
    - [Message delivery semantics and guarantees](#message-delivery-semantics-and-guarantees)
    - [Strategy for processing infinite streams of events](#strategy-for-processing-infinite-streams-of-events)
      - [Tumbling Window](#tumbling-window)
- [Count events every 10 seconds](#count-events-every-10-seconds)
      - [Hopping Window](#hopping-window)
- [Window Size: 10s, Hop: 5s](#window-size-10s-hop-5s)
      - [Sliding Window](#sliding-window)
- [Alert if > 5 events in last 60 seconds](#alert-if-5-events-in-last-60-seconds)
      - [Session Window](#session-window)
      - [Comparison](#comparison)
    - [Observability in Microservices architecture](#observability-in-microservices-architecture)
      - [The Three Pillars](#the-three-pillars)
    - [Migrating from a monolith to a microservice architecture](#migrating-from-a-monolith-to-a-microservice-architecture)
      - [The Strategy: The "Strangler Fig" Pattern](#the-strategy-the-strangler-fig-pattern)
      - [How to Decompose? (Finding Boundaries)](#how-to-decompose-finding-boundaries)
      - [Handling the Database (The Hardest Part)](#handling-the-database-the-hardest-part)
      - [Steps & Tips for Migration](#steps-tips-for-migration)
    - [Microservices principles and best practices](#microservices-principles-and-best-practices)
  - [Big Data](#big-data)
    - [Processing Strategies: Batch vs. Stream](#processing-strategies-batch-vs-stream)
      - [Batch Processing (The "Laundry" Method)](#batch-processing-the-laundry-method)
      - [Stream Processing (The "Dishwasher" Method)](#stream-processing-the-dishwasher-method)
    - [The Lambda Architecture](#the-lambda-architecture)
      - [The Three Layers](#the-three-layers)
    - [OLTP vs OLAP](#oltp-vs-olap)
      - [Why separate them? The Big Data Strategy](#why-separate-them-the-big-data-strategy)
      - [Row vs. Columnar Storage - The Secret Sauce](#row-vs-columnar-storage-the-secret-sauce)
      - [When to use what?](#when-to-use-what)
    - [Map-Reduce for Big Data processing](#map-reduce-for-big-data-processing)
  - [Reliability, Error Handling, and Recoverability patterns](#reliability-error-handling-and-recoverability-patterns)
    - [Throttling & Rate Limiting (The Bouncer)](#throttling-rate-limiting-the-bouncer)
    - [Retry Pattern (The "Try Again")](#retry-pattern-the-try-again)
    - [Circuit Breaker Pattern (The Fuse)](#circuit-breaker-pattern-the-fuse)
    - [Dead Letter Queue (DLQ) (The Graveyard)](#dead-letter-queue-dlq-the-graveyard)
    - [Transactional Outbox Pattern](#transactional-outbox-pattern)
  - [Deployment patterns](#deployment-patterns)
  - [Other patterns](#other-patterns)

## Definition of software architecture

> Software Architecture of a system is a high-level description of the **system's structure**, its **different components**, and **how those components communicate with each other** to fulfil the system's requirements and constraints.

* High-level: 
	* It means it is an abstraction showing the important components
	* Hides implementation details
	* Programming languages or technologies are not a part of Software architecture but implementation
	* Decisions about technologies should be delayed to the very end of our design

* Components are "black boxes" defined by their *behaviour* and *APIs*
	* They themselves maybe complex systems described by their own architecture diagrams (recursive when needed)

* System's requirements
	* What is **should do**

* System constraints
	* What it **should not do**

## Levels of abstraction

We can talk about software architecture in terms of:
1. Classes / Structs (Low-level)
2. Modules / packages / libraries
3. Services (processes / group of processes) (High-level)

**Services** are what we focus on when explaining the *architecture of large-scale systems* i.e at a high-level. 

Services are potentially placed in a **distributed** network

## Advantages of distributed services

1. Handle a large amount of requests
2. Process and store very large amounts of data
3. Serve many users every day

Example applications: Ride sharing, Video on Demand (VOD), online video games, investment services and banks, etc.

## Software architecture position in the SDLC

Software development lifecycle consists of the following states (loop):
1. Design
2. Implementation
3. Testing
4. Deployment
```
Design -> Implementation -> Testing -> Deployment
^                                         |
|_________________________________________|
```

Software architecture sits *in-between* **Design** and **Implementation**:
* ***Software architecture is the output of the Design phase***
* ***Software architecture is the input to the Implementation phase***

## Challenges of software architecture

* We **CANNOT** prove software architecture is:
1. Correctness
2. Optimal

* We **CAN**:
1. Follow a methodical process
2. Use proven architectural patterns
3. Follow best practices

## Gathering requirements

* **Requirements**: Formal description of what we need to build
	* These are generally not limited to a small function but the entire application
	* These also may come from non-technical stakeholders like business clients, PMs, etc
	* Therefore, they have a high-level of ambiguity
	* It is *our responsibility to convert these into **technical requirements*** by asking *clarifying questions* -- and believe it or not, this transformation is part of the solution!
		* Why is it part of the solution? It greatly narrows down what we need to design and build!

Example requirements from the builders of a ride-hailing app: 
"Allow people to join drivers on the same route, who are willing to take passengers for a fee"

Clarifying questions:
1. Is it real time or advanced reservation?
2. Is the UX on mobile or desktop or both?
3. Is payment through the app or direct?

(**Note**: Asking these questions and narrowing down the design is important for system design interviews)

### Importance of gathering requirements

We need to gather the requirements correctly early on since:
* Large scale systems cannot be easily changed
* Many engineers are involved
* Effort to implement can take many months
* It may include hardware and software costs (Ex: licenses for a vendor tool)
* Contracts include financial obligations (SLAs, SLOs, etc)
* If project is not delivered on time and matching requirements, it could lead to a loss of reputation and brand

### Types of requirements

1. **Features of the system (`Functional requirements`)**
2. **Quality attributes (`Non-Functional requirements`)**
3. **System constraints (`Limitations and boundaries`)**

The above three requirements are also known as **"Architectural drivers"** (Why? They essentially "drive" our architecture from an infinite set of possibilities towards one solution that satisfies all our client's needs)

#### Functional requirements

These describe the **system behaviour** i.e "What the system must do" (not how, how soon, etc)
* The system is a "black box function" with user inputs & external events as input and outcomes as the result. Hence, these requirements are called "functional" requirements (Black box means we do not know its internal properties and details)

```
User Inputs ----\
                  \
                   ---> [ SYSTEM ] ---> Result / Outcome
                  /
Events ----------/
```

**Note**: Functional requirements **DO NOT** dictate the architecture of our system since the system is a "black box". Any architecture could practically realise a requirement. Hence, job of software architects is much harder than gathering functional requirements (Hence, necessary but not sufficient)

*Example 1*:
"When a rider logs into the service mobile app, *the system must* display a map with nearby drivers within a 5 mile radius"
* User input: "When a rider logs into the service mobile app"
* Result / outcome: "Display a map with nearby drivers within a 5 mile radius"

*Example 2*:
"When a ride is completed, *the system will* charge the rider's credit card and credit the driver, minus service fees"
* External event: "When a ride is completed"
* Result / outcome: "Charge the rider's credit card and credit the driver, minus service fees"

#### Non-Functional requirements

Known as "Quality attributes". These requirements tell **"the properties the system must have"**

Examples:
1. Scalability 
2. Availability
3. Reliability
4. Security
5. Performance
6. ...

Useful mnemonic to remember the acronym of the above points:
**S**ome **A**cts **R**equire **S**erious **P**lanning

**Note**: Non-Functional requirements **DO ** dictate the architecture of our system (Unlike functional requirements)

```
User Inputs ----\
                  \
                   ---> [ SYSTEM (Quality attributes) ] ---> Result / Outcome
                  /
Events ----------/
```

#### Limitations and boundaries

Known as "System constraints". Examples include:

1. Time constraints -- strict deadlines
2. Financial constraints -- Limited budget
3. Staffing constraints -- small number of engineers

System constraints *force us* to make **tradeoffs / sacrifices** in our architecture that we would not make otherwise

### Right way to gather functional requirements

Naive way is to ask client to list all requirements hoping he will not forget any detail. This does not work for a large scale system as it is too complex

#### Formal way of gathering requirements:
1. **Use-cases**: Situation / scenario in which our system is used
2. **User flows**: A step-by-step graphical representation of each of our use case

#### Steps to gather requirements:
1. Identify all **actors** / **users** in the system
2. Capture and describe all possible **use-cases** / **scenarios**
3. **User flow** - Expand each use-case through flow of **events**
	- Each event contains
		- **Action** (Usually an API call to/from actors/users)
		- **Data** (Usually the data sent as part of the API call)

*Example 1:* "Allow people to join drivers on a route, who are willing to take passengers for a fee"

Actors: 
- Driver 
- Passenger

Use cases:
- Rider first time registration
- Driver registration
- Rider login
- Driver login
- Successful match and ride
- Unsuccessful ride

Graphical representation of a "user flow:" ***Sequence diagrams*** (Shows interactions between actors and objects. It is part of the Unified Modelling Language (UML))

"User flow" diagram benefit is that we can take the diagram and ***identify our future API with it*** (Data flowing between those interactions is our API payload)

User flow example for "Successful match and ride" use case:
```
Passenger        App              Driver
    |             |                 |
    |             |<--------------  |  
    |             |  Ready to pick up
    |             |                 |
    |------------>|                 |
    | Look for    |                 |
    | driver      |                 |
    |             |                 |
    |             |----->|          |
    |             | Match driver    |
    |             |<----|           |
    |             |                 |
    |<------------|                 |
    | Driver is   |                 |
    | found       |                 |
    |             |---------------> |
    |             | Passenger found |
    |             |                 |
    |             |     <---- Ride  |
    |             |        started  |
    |             |                 |
```

```
Passenger        App                Driver
    |             |                   |
    |             |<------------------|
    |             |   Finish ride     |
    |             |                   |
    |             |--------->|        |
    |             | Charge passenger  |
    |             |<---------|        |
    |             |                   |
    |<------------|                   |
    | Show ride   |                   |
    |  receipt    |                   |
    |             |                   |
    |             |--------->|        |
    |             | Take service fee  |
    |             |<---------|        |
    |             |                   |
    |             |--------->|        |
    |             |  Pay driver       |
    |             |<---------|        |
    |             |                   |
    |             |------------------>|
    |             | Show driver       |
    |             |   earnings        |
    |             |                   |
```

### More on non-functional requirements

Quality attributes (Non-functional attributes) are the main reason why a system needs redesign, not the functional requirements. Reasons could be that system is not fast, scalable, easy to maintain, and so on

Quality attributes describe:
1. The **qualities** of the functional requirements
2. The **overall properties** of the system

*In simple terms*: They provide a **quality measure** on how well our system performs on a **particular dimension**

Example using a requirement: 
* Statement: "When a user clicks on a search button after they typed in a particular search keyword, the user will be provided with a list of products that closely match the search keyword within at most 100 milliseconds"
* Functional requirements: ""When a user clicks on a search button after they typed in a particular search keyword, the user will be provided with a list of products that closely match the search keyword"
* ***Non-functional requirement / quality attribute: "within at most 100 milliseconds"***
* **Dimension = Performance quality**

Example using an overall property:
* "The online store must be available to user requests at least 99.9% of the time"
* ***Non-functional requirement / quality attribute: "at least 99.9% of the time"***
* **Dimension = Availability**

Example using an overall property:
* "Development team can deploy a new version of the online store at least twice a week"
* ***Non-functional requirement / quality attribute: "at least twice a week"***
* **Dimension = Deployability**

#### Non-functional requirements need to be proven

Quality attributes need to be proven. How to prove? They must be:
1. **Measurable**
2. **Testable**

If this is not possible, there is no way to know how *well* or *poorly* our system performs on that dimension.

Bad example:
* "When a user clicks on the buy button, a purchase confirmation screen should be *quickly* shown to the user"
* Here, *"quickly"* does not represent something measurable nor testable. Is 200ms quick? What about 100ms, 2000ms? Not clear.

**Note**: 
* There is **NO** perfect system architecture
* Why? Certain quality attributes ***contradict*** each other
* It is impossible to achieve certain quality attributes' combinations
* Hence, software architecture is all about **trade-offs** -- making the right ones in order to achieve success

Example:
* Requirement: Fast login
* Requirement: Secure login with username, password, SSL for encryption, etc
* It is difficult to make login fast while also waiting for the user to input details and network delays causes by the extra SSL layer

#### Last consideration is feasibility

Sometimes the clients ask for something that is impossible to implement. It is our job to convey this to the client and with valid reasons (Ex: Unrealistic low latency expectations / 100% available system, even during maintenance / very high storage growth / full protection against hackers)

### System constraints 

With the requirements and trade-offs, we design a system. Without constraints to the design, we have infinite degrees of freedom to design it how we like.

Constraints limit our degrees of freedom: "A system constraint is essentially a **decision** that was either **already fully or partially made** for us, **restricting our degrees of freedom**"

Constraint is NOT ALWAYS a bad thing! Look at it as a decision that was made instead of a choice that was taken away

System constraints are known as the *"Pillars of software architecture"* since they provide us with a solid starting point for our architecture

Three types of constraints:
1. Technical constraints
2. Business constraints
3. Legal/Regulatory constraints

#### Technical constraints

These might seem to belong to implementation but these in fact affect architecture too as our decisions need to take technical constraints and its effects into account

Examples:
1. Locked to a particular vendor
2. Locked to a particular programming language - Difficult to find engineers / to change overall stack (?)
3. Having to use a particular database technology
4. Having to support multiples browsers or OSes
5. If the network is on premise then a lot of the cloud based tools maybe unavailable to us

#### Business constraints

Examples:
1. Limited budget
2. Strict deadline
3. Start up wanting to iterate quickly v/s a big company building things slower
4. Use of 3rd part APIs due to business contracts (Ex: 3rd party billing or a bank API)

#### Legal/Regulatory constraints

Examples:
1. HIPAA regulations in the US for storing medical records
2. GDPR for privacy in the Europe region

#### Accepting vs pushing back on constraints

Do not accept constraints lightly!

There should be a distinction v/s :
1. Real constraints 
	- Ex: Legal or compliance constraints
	- No scope for push back
2. Self-imposed / Internal constraints
	- Ex: Questioning a vendor lock-in? Do we need the lock-in? How about in-house or other vendors (Opportunity to explore others)?
	- Scope to question these, for sure!

Constraints when existing should be integrated in a ***loosely coupled*** way. That is, if we need to change things in the future such as changing a vendor, our architecture should support that with minimal effort and changes.

## Performance

Performance is one of the most important quality attributes i.e non-functional requirement

There are two ways to measure performance:
1. Response time
2. Throughput

### Response time

***Response time between a client sending a request and receiving a response***. It is also referred to as "end to end latency"

`Response time = processing time + waiting time`

* Processing time is the time spent by the system doing the actual work such as running the coding logic, fetching information from the datastore, etc
* Waiting time (also known as "latency") is a combination of (a) Network delay i.e response in transit, and (b) Time the request is waiting in a queue prior to processing!

**Measuring response time correctly**

We must take into account the "waiting time" as well. Ex: Consider two requests, A & B, arriving at the same time. While A is being serviced, B is waiting. Once A is done, B is picked up. Let us say that both took 10ms to process. If we measure only the processing time (incorrectly) then the average response time is `(10 + 10) / 2 = 10` which is wrong. The correct calculation after including the 10ms waiting time for B would be `(10 + 20) / 2 = 15`

```
Time →
Request A:
A arrives ----[ Processing A (10ms) ]------------------------------>
             0ms ------------------ 10ms

Request B:
B arrives ---(     Waiting 10ms     )--[ Processing B (10ms) ]-------->
             0ms ------------------ 10ms ------------------ 10ms

Response Times:
- A = 10ms
- B = 10ms waiting + 10ms processing = 20ms

Incorrect average (processing only):
(10 + 10) / 2 = 10ms

Correct average (including waiting):
(10 + 20) / 2 = 15ms
```

### Measuring response time distribution

It is important to measure response times correctly
We do not benefit from simple arithmetic like average, median, etc as they don't capture the full picture for all types of requests

Steps:
1. Draw a histogram for #requests vs time (Ex: 3 requests, each taking 10ms would mean that a line with unit size 3 at point 10ms would be present)
2. From this Histogram, draw a percentile distribution graph

Correct way to measure the response time distribution: Use a **percentile distribution** as  the metric which essentially means: "The `x`th percentile is the value __below__ which `x%` of the values can be found"

Example:
```
Response Time (ms)
^
|                                  *
|                             *
|                        *
|                   *
|              *
|        *
|   *
| *
+--------------------------------------------------> Percentile (%)
  0     50     75     90     95     99    99.9   100

Sample Points:
  P50   = 120ms
  P75   = 180ms
  P90   = 240ms
  P95   = 300ms
  P99   = 480ms
  P99.9 = 900ms
  Max   = 1200ms
```

Interpretation example:
In the above graph, `75%` of requests have a response time of `180ms` or less (`P75`)

#### Tail latency in response times

Tail latency is *"The small percentage of response times from a system that take the longest time in comparison to the rest of the values"*. ***Shorter the tail the better***

When you take the average response time of all requests, there sometimes is "tail" that extends well below the average. These requests took a lot more time than the average. It can be visualised in the percentile distribution line like so:

```
Response Time (ms)
^
|                           /
|                          /
|                         /
|                        /
|-----------------------/----------------------  ← Inclined tail latency line
|                      /
| ___        /\       /
|/----\-----/--\-----/----------------------  ← Average response time (flat)
|      ----/    \---/
+----------------------------------------------------> Percentile (%)
 0   50   75   90   95   99  99.9  100

Explanation:
- The **horizontal line** represents the *average latency* staying constant.
- The **inclined line** shows how **latency increases sharply** in the tail (P95–P99.9+).
```

### Throughput

Throughput is the measure of how many requests are being serviced at a given point in time

Two ways to measure:
1. Amount of *work performed* by our system per unit of time
	- Measured in **Tasks/second** (Ex: QPS or queries per second is a common metric for APIs/DBs)
2. Amount of *data processed* by our system per unit of time
	- Measured in bits/second, bytes/second, megabits/second, and so on

Ex: A distributed log service needs to have high throughput in order to receive many requests simultaneously and add the logs to the system as fast as possible

**Note**: Throughput is important when we need to "keep up with the load". Whenever we are dealing with processing large amounts of data especially when collected from multiple sources (clients) then throughput is clearly more important than response time. Think of analytics, logs, data engineering, monitoring systems, etc. It is okay to send the response late but it's important to service very many requests at a given time to gather all the data

### Identifying performance degradation

Performance degradation can be measured by mapping either the (A) response time or (B) throughput against the load (Load can be requests per second, for example)

Whenever there is a *sudden increase" in the response time / "sudden drop" in throughput, we know that that point is the point of stress for our system (Called ***degradation point***) i.e our system is not designed for load beyond that point & performs poorly

```
Response Time (ms)
^
|                     
|                    * * ← sudden drop (perf degradation)
|                  *    *
|                *       *
|             *           *
|         *                *
|     *                     *
|  *                         *
| *                           *
|*                            *
|                              *
|                               *
+-------------------------------------------> Load
   Low           Medium         High
```

## Scalability

"Scalability is the measure of the system's ability to handle a growing amount of work, in an easy and cost efficient way, by adding resources to the system"

### Understanding load

Load is the stress on a system. It can be defined in terms of queries / requests per unit of time. The important aspect to note is that *load is never really constant.* It keeps fluctuating.

Imagine a search spike on google.com for a keyword relating to an event that has just gone viral. There is going to be a spike i.e increase in load for google's servers during that time. As another example, users might be using an office digital service so the load on that service might be more during the day than night.

```
Load
^
|               *        * * *         * * *
|            *     *    *     *     * *      *
|  * *     *        *  *       * * *          *
|      *  *           *
|        *                                   
|
+--------------------------------------------------------> Time
```

### Intuition for a scalable design

A system is scalable if provided more resources and time, it can handle more load easily and efficiently

Consider two designs, A and B, for a system. Both are provided more resources (Ex: More compute, storage, etc) and time (to add these resources and the changes to make to support integration of the resources). Assume that A performs better on the response time vs load graph than B (Ex: Degradation point arrives much later). In that case, design A is said to be *more scalable* than design B!

### Note on linear scalability

The ideal scenario for scalability is if we simply continue to add resources then the amount of work done grows linearly too. This is wrong in practice! ***Linear scalability is not possible due to physical constraints.*** There will also be a diminishing return after a point (Ex: a parabolic curve than a straight line

```
Ideal scalability scenario:

Amount of work
^
|                 *
|              *
|           *
|        *
|      *
|    *
|  *
+---------------------------------------------> Resources
```

```
Practical scalability scenario:
(The smaller the downward bend, the better the system's scalability score)

Amount of work
^
|                  
|              
|               *  *  *  *  *
|           *                  *
|        *                        *
|     *
|  *
+---------------------------------------------> Resources
```

### Types of scalability

3 types:
1. Vertical 
2. Horizontal 
3. Team / Organisational

#### Vertical scalability

Definition: Adding resources or upgrading the resources on a single computer, to allow our system to handle higher traffic or load

Also known as **"scale-up"**: 
* We add more compute power, more storage, etc to the existing system's resource
* Due to physical constraints on compute & storage. Infinitely scaling up is not possible
* Thresholds on resources can be reached pretty quickly in the real world

```
          Vertical Scaling (Scaling Up)
          --------------------------------

                     +------------------+
                     |     Server       |
                     |   (Baseline)     |
                     +---------+--------+
                               |
                               |  Add more CPU / RAM / Disk
                               v
                     +------------------+
                     |     Server       |
                     |   (Upgraded)     |
                     |  More Resources  |
                     +---------+--------+
                               |
                               |  Further upgrade
                               v
                     +--------------------------+
                     |        Server            |
                     |    (High Capacity)       |
                     |  Even More Resources     |
                     +--------------------------+
```

#### Horizontal scalability

Definition: Adding more resources in the form of new instances running on different machines, to allow our system to handle higher traffic or load

Also known as **"scale-out"**: 
* We add more computers (Which contain the compute or storage resources)
* Less physical constraints compared to scaling-up
* Communication between the computers is via the network

```
          Horizontal Scaling (Scaling Out)
          --------------------------------

                     +------------------+
                     |     Server 1     |
                     |  (Initial Load)  |
                     +---------+--------+
                               |
                     Add more servers
                               |
        -------------------------------------------------
        |                       |                       |
        v                       v                       v
+---------------+      +---------------+       +---------------+
|   Server 1    |      |   Server 2    |       |   Server 3    |
| (Same spec)   |      | (Same spec)   |       | (Same spec)   |
+-------+-------+      +-------+-------+       +-------+-------+
        \____________________|_________________________/
                             |
                  Load distributed across servers
```

#### Pros and cons of each scalability choice

*Vertical scaling:*
1. Pros
	* Any application can benefit from it (i.e take any application and place the whole app in another system)
	* No code changes are required
	* The migration between different machines is very easy (Ex: Move whole app down to a smaller compute device to save on costs when traffic is low)
2. Cons
	* The scope of the upgrade is limited (due to constraints of the physical world & cost)
	* We are **locked in a centralised system** which **CANNOT PROVIDE**:
		* ***High Availability***
		* ***Fault tolerance***

*Horizontal scaling:*
1. Pros
	* No limit on scalability
	* It's easy to add or remove machines 
	* If designed correctly, **WE GET**:
		* ***High Availability***
		* ***Fault tolerance***
2. Cons
	* Initial code changes maybe required
	* Increased complexity, coordination overhead

Hence, **Horizontal scalability is better in the long run!**

#### Team / Organisational scalability

This type of availability focuses on adding engineers (i.e hiring) as the resource to get more work done in terms of features, testing, etc. It is a bit different than vertical and horizontal scaling in that it focuses on people and processes than compute, storage, network and so on

This scalability too is not linear and receives diminishing returns for the following reasons:
* Many crowded meetings
* Code merge conflicts
* Biz complexity - longer ramp up time
* Testing is harder and slower
* Releases become very risky 
* ...

One way to make team scalability work is to:
1. Split features or domains into services
2. Each service has its own stack, resources and so on
3. Services can interact via the network with each other to fetch data
4. Now each team can scale independently and even deploy independently

This way of scaling relates to having a "service oriented architecture" (Refer to the "Architecture patterns" section)

## Availability

1. **Inconvenience**: Imagine users going to a website but it does not load. Or, they go to checkout but get an error during payment
2. Think about ***outages* affecting B2B** relationships. any AWS or Cloudflare outages on services like DNS or S3 affects hundreds of thousands of other services that rely on them
3. Some of the discruptions could affect **mission critical systems** like ATC or hospital networks.

The above are examples of unavailability where users cannot use our system. In such cases:
* Our revenue goes down
* Our users go to our competitors

Availability: **"The fraction of time or probability that our service is operationally functional and accessible to the user"**

### Formula of availability

* `Uptime = Time that our service is operationally functional and accessible to the user`
* `Downtime = Time that our system is unavailable to the user`
* `Availability (%) = Uptime / (entire time our system is running)`

**`Availability = Uptime / (Uptime + Downtime)`**

Sometimes availability and uptime are used interchangeably

### Alternative formula of availability

A more statistical formula

There is a formula based on MTBF and MTTR:
1. **MTBF (Mean Time Between Failures)**: Represents the average time between failures
	- This is mostly useful for detecting hardware faults, lifespan of devices, and not so much cloud-related services or APIs
2. **MTTR (Mean Time To Recovery)**: Average time it takes to detect and recover from failure
	- This represents the average downtime of the system

**`Availability = MTBF / (MTBF + MTTR)`**

^ The main benefit of this way of calculating availability for companies is that if we can bring MTTR to zero i.e detect and recover from failures instantly, then we can achieve 100% availability since you can let failures happen as long as you recover really fast (`MTBF / (MTBF + 0)`

### Myth of complete availability

*100% availability is a myth*. There will always some disruption (Ex: maintenance, upgrade, infra issue, natural calamity, etc). But, the goal is to move towards 100% availability even if we cannot achieve it

What is a good availability?
* There is no fixed value
* Cloud companies provide `99.9%` uptime and onwards. This value has generally come to be accepted as high availability

```
+---------------+----------------------+------------------------+-------------------------+
| Uptime %      | Daily Downtime       | Monthly Downtime (30d) | Yearly Downtime (365d)  |
+---------------+----------------------+------------------------+-------------------------+
| 90%           | 2.4 hours            | 72 hours               | 36.5 days               |
| 95%           | 1.2 hours            | 36 hours               | 18.25 days              |
| 99%           | 14.4 minutes         | 7.2 hours              | 3.65 days               |
| 99.9%         | 1.44 minutes         | 43.2 minutes           | 8.76 hours              |
| 99.99%        | 8.64 seconds         | 4.32 minutes           | 52.56 minutes           |
+---------------+----------------------+------------------------+-------------------------+
```

### The 9s of availability

Availability is also measured as the number of 9's in the uptime `%`

We count the 9s even before the decimal point:
* 99.9%: 3 nines (Good / High)
* 99.99%: 4 nines
* 99.999%: 5 nines (Excellent!)
* ...

### Fault tolerance

What prevents us from achieving "High Availability (HA)"?
1. Human errors (pushing faulty config to prod, wrong command/script, deploying untested code) 
2. Software errors (Code errors, out of memory & null pointer exceptions, segmentation fault, etc)
3. Hardware errors (power outages, shelf life of hardware devices, N/W failures: Calamity, congestion, or Infra issues)

***Failures are inevitable!***

Best way to achieve High Availability: **Fault tolerance**

Definition: "Fault tolerance enables our system to remain operational and available to our users ***despite*** failures within one or multiple of its components"

When failure occurs, a fault-tolerant system will:
1. Continue operating at the same or a reduced level of performance
2. Prevent the system from becoming unavailable

**Three major fault tolerance tactics**

1. Failure prevention
2. Failure detection and isolation
3. Failure recovery

#### Failure prevention

Failure prevent means ***eliminating any single point of failure (SPOF)*** in the system. Examples of single points of failure:
* *One server* where we're running our application 
* Storing all our data in *one instance* of our database that runs on a *single instance*

This is the reason why ***scaling-up (vertical scaling) is not fault tolerant***!

Best way to eliminate a SPOF: "Replication" and "redundancy"
* This is the reason why ***scaling-out (horizontal scaling) can be fault tolerant if designed well***!
* Ex: If one server is down, redirect the requests to other servers
* Ex: If one database is down, redirect the request to one of the replicated databases

#### Types of redundancy

Two types:
1. **Spatial**: Running replicas of our application on different computers
2. **Time**: Repeating the same operation/request multiple times until we succeed / give up (Essentially retrying)

#### Strategies for redundancy

There are two main strategies (for maintaining replicas):
1. Active-Active architecture
2. Active-Passive architecture

**Active-Active architecture**

Requests go to all the replicas. This means they are forced to always be in-sync with each other. In turn, this means that whenever a replica goes down, other replicas can take in a request immediately!

**Note**: This does not mean that every single request goes to every single replica. The load is distributed. For example, reads might be distributed amongst the replicas. Writes may go to a single replica but the synchronisation of data between the replicas need to happen immediately after a write

Advantages:
1. Spread the load among all replicas
2. Identical to horizontal scalability
3. Allows more traffic
4. Better performance

Disadvantages:
1. All the replicas are taking requests (Bandwidth, cost, etc)
2. Additional coordination required to keep active replicas in sync

```
                         +--------------------+
User Requests  --------> |   Load Balancer    |
                         +--------------------+
                           |        |        |
                           v        v        v
                     +---------+ +---------+ +---------+
                     |  DB #1  | |  DB #2  | |  DB #3  |
                     | Active  | | Active  | | Active  |
                     +----+----+ +----+----+ +----+----+
                          |           |           |
                          |           |           |
                          +-----------+-----------+
                                      |
                              Replication Layer
                                      |
                   ------------------------------------------------
                   |      All updates are synchronized             |
                   ------------------------------------------------

Notes:
- Requests are **distributed** so each goes to **one** DB node.
- All DB nodes are **active** and can handle reads or writes.
- Writes accepted by any node are **replicated** to the others.
```

**Active-Passive architecture**

There is primary instance (or primary replica) which takes in all the requests. Other replicas follow the primary by taking periodic snapshots of it (if these are databases). When the primary replica goes down, there is a process by which another primary replica is chosen from the set of remaining replicas. This change involves some effort and time and is not as immediate as Active-Active architecture

(**Note**: Active-Passive BY DEFAULT is not scalable due to the single point of failure (SPOF) problem. There are ways to solve this)

Disadvantages:
* We lose the ability to scale our system since all requests go to one single machine (No fault prevention)

Advantages:
* Implementation is easier (than active-active)
* There is a clear leader with up-to-date data
* Rest of the replicas are just followers

```
                         +--------------------+
User Requests  --------> |   Load Balancer    |
                         +--------------------+
                                    |
                                    v
                           +----------------+
                           |   Primary DB   |
                           |    (Active)    |
                           +--------+-------+
                                    |
                         Replication (Periodic snapshots taken i.e async process)
                                    |
          ---------------------------------------------------------
          |                         |                             |
          v                         v                             v
+----------------+        +----------------+             +----------------+
|  Standby DB 1  |        |  Standby DB 2  |             |  Standby DB 3  |
|    (Passive)   |        |    (Passive)   |             |    (Passive)   |
+--------+-------+        +--------+-------+             +--------+-------+
         ^                         ^                              ^
         |                         |                              |
         ---------------------------  Failover (if primary fails) --------------------------

Notes:
- Only the **primary** handles live traffic.
- Multiple **standby nodes** stay updated through async replication (periodic snapshots).
- If the primary fails, *one* standby becomes active.
```

#### Failure detection

Failure detection is when an instance goes down and you need to know about it in order to take corrective actions such as redirecting traffic while the instance is repaired or replaced

How can we detect failure? Add a separate ***monitoring service***

How can the monitoring service detect an instance failure? Expect each instance to receive/send periodic signals (***pings and heartbeats***) to the monitoring service. 

If a ping/heartbeat does not arrive in time? There are two possibilities:
1. False positive: Network issue / long garbage collection (not really instance problem). This is okay! We are fine with it
2. False negatives: This is not okay as it means that servers may have crashed or the monitoring system did not detect that

Monitoring system functions:
1. Exchange of messages via pings and heartbeats
2. Collect data about number of errors each host gets in a minute *(High error rate? Interpretation: **Failure** in host)*
3. Collect information about time taken for each host to respond *(Too long? Interpretation: Host is **slow**)*

#### Failure recovery

Actions taken after failure:
1. Stop sending traffic to the affected / failed server
2. Restart host to make the failure go away
3. Rollback - Going back to a version that was stable and correct

Rollback:
* Quite common in databases
* In databases: It means rolling back to the last correct state of data that does not violate DB constraints
* In server: It means rolling back to the last stable version of the software i.e previous version

### SLAs SLOs and SLIs

#### SLA or Service Level Agreement

It is:
* A **legal contract** that represents the quality of our service
	* Covers availability, performance, data durability, time to respond to failures, and so on
* States penalties if contract is breached. Includes
	* Full/partial refunds
	* Subscription / license extension (Ex: If free trial user cannot use system, extend the free trial)
	* Service credits
* SLAs exist for:
	* **External paying users** (*Always*)
	* **External free users** (*Sometimes* - but better to not keep an SLA if service is entirely free)
	* Internal users (*Occasionally*. **Why**? Because another team using our service might rely on our SLAs to provide their own SLA to an external client)

#### SLOs or Service Level Objectives

These are:
1. **Individual goals** set for our system
2. Represent a **target value** or **target range** our service needs to meet

Ex: 3 nines (99.9%) availability or 24-48 hours of response time in case of failure.

**Note**: SLOs are d*eeply tied* to Non-functional requirements (Quality attributes) since without them these goals cannot be set. Additionally, it is another reason why quality attributes need to be measurable and testable!

Therefore, SLO include:
1. Quality attributes of the system
2. Other objectives

**Link between SLOs and an SLA:**
1. SLA is a legal contract formed based on the SLOs (i.e SLAs cover multiple SLOs)
2. SLOs must still be provided even if an SLA is missing. Why? Users need to know what to expect from our system!

#### SLIs or Service Level Indicators

These are the actual numbers that are received and compared against our SLOs
* Measured using a monitoring service
* Calculated from our logs

These help make sure that we are compliant with our SLOs which in turn means we are compliant with our SLA. (This is why quality attributes are important)

#### Important considerations

SLAs are crafted by business or legal architects

Developers have more control over SLOs! Here are the considerations to leave a *budget for error*:
1. Don't create an SLO for every SLI: Only add SLOs for things that our users care about the most!
2. Promise fewer SLOs: Why? Because it hard to prioritise many quality attributes
3. Set realistic SLOs: Commit to lower availability (Set internal bar high but externally, commit to less!). Saves cost and incorporates unexpected issues

* [AWS SLAs](https://aws.amazon.com/legal/service-level-agreements/?aws-sla-cards.sort-by=item.additionalFields.serviceNameLower&aws-sla-cards.sort-order=asc&awsf.tech-category-filter=*all)
* [GPC SLAs](https://cloud.google.com/terms/sla)
* [Github SLA](https://github.com/customer-terms/github-online-services-sla)
* [Atlassian SLA](https://support.atlassian.com/subscriptions-and-billing/docs/service-level-agreement-for-atlassian-cloud-products/)

## APIs

Once we have all requirements, the system is a "black box" with:
1. Behaviour (i.e input/output), and
2. An interface

What is an interface? It is a **contract** between the engineers who implement the system and the client application that uses it. Since the application is interfacing with the system, it has come to be known as **Application Programming Interface (API)**

APIs are usually accessed *remotely via the network* in large-scale systems

Who calls our APIs?
1. Frontend client apps (external)
2. Backend systems (still external)
3. Internal systems (within the company - in order to take advantage of existing system's capabilities)

### Categories of APIs

**Public APIs**
* Exposed to general public i.e any one can access (Ex: some weather APIs)
* *Good practice*: Register the client before they can start using it (Better security i.e `how` and `who` is using it is known)

**Private APIs**
* Exposed internally within the company
* Allow other teams and orgs to take advantage of the system and provide even bigger value to the company
* System is not exposed outside the company (i.e company network)

**Partner APIs**
* Similar to Public APIs but exposed only to those companies having a business relationship with us
* Example: Think SaaS client, business tie-up to use our resources, etc
* The basis is a customer agreement after buying our product

### API benefits

1. Clients can immediately enhance their biz by using our APIs (Biz value)
2. Clients need not know internal implementation of the system (Black box, API contract is supreme!)
3. Client can integrate with us immediately (No wait time, no manual intervention or support)
4. **Important**: Makes it easier to design and architect the system. Why?:
	- API defines the endpoints to the different routes that the user can take to our system

### API best practices

There are 6 best practices:
1. **Complete encapsulation** (i.e decoupled from internal implementation)
	- Abstract away internal details completely from the user/client
	- If internal details are required to use the API, its purpose is defeated
	- We should be able to change implementation without changing the contract (**Important!**)
2. **Make the API easy to use, understand, and hard to misuse**. **How?**:
	1. Make sure there is <u>only one way</u> to get either a certain data or task performed
	2. Use <u>descriptive names</u> for actions/resources
	3. Expose <u>only the information</u> the users *needs*
	4. Keep things <u>consistent</u> across the APIs
3. Keeping the operations **idempotent**! (Important)
	* What is Idempotency? **An operation doesn't have *any additional effect* on the result if it is performed *more than once***
	* Examples of idempotent operations:
		* Updated the address to a new one (As long as the new value does not change, the result is the same)
	* Examples of non-idempotent operations:
		* Adding $100 to a bank account. If retried, this would increase the balance by more than desired. Better, i.e idempotent, way is to change the bank account value to a fixed number

Why does Idempotency matter? It matters because when a client sends a request and something goes wrong, it could be because of one of the following reasons:
* Request packet lost in transit
* Server crash
* Response packet lost in transit

Since it does not know the reason, a common practice is to retry the operation i.e resend the request. Hence, Idempotency is important!

4. **Paginate your API**
	* If response contains a large dataset, client cannot handle so much data and it results in a poor experience
	* Examples: Email page is flooded with all the emails since forever, receive millions of product results on a product search page at once, and so on
	* Pagination benefits:
		* Client requests small amounts of data 
			* Specify the maximum amount of results to be returned in one called known as the **limit**
			* Specify an **offset** within the overall dataset (Used to request for the "next segment")

Therefore, "limit" and "offset" values in the API request are necessary for pagination to be made possible

5. **Asynchronous APIs for time consuming operations**
	* Some operations take a lot of time to compute or process
	* Nothing meaningful can be shared with the client until the result is ready
	* Ex: Report generation tasks, big data analytics on log files, compression of media files, etc
	* Best approach:
		* Client receives an immediate response *without having to wait* for the final result
		* Response includes information to track the progress and status of the operation
		* Using the tracking API, it can eventually receive the final result

6. **Versioning our API**:
	 * Required since we want to make changes to our internal implementation without changing the API
	 * Needed when we need to make non-backward compatible changes to the API
		 * Maintain two versions of the API (One new, one for the old clients)
		 * Gradually deprecate the older API over time

## Remote Procedure Calls

**RPCs** are methods of the system that we invoke on the client like as though it is just another local function/method. This concept is known as ***"location transparency"***

It is useful if you want to abstract away the transport layer details. Development using RPC methods is very easy.

RPC calls might be slower than regular methods owing to the network and debugging is not the same as debugging a local method

```
+--------+                         +--------+
| Client |                         | Server |
+--------+                         +--------+
     |      Request: Call X(args)       |
     | --------------------------------> 
     |                                  |
     |                                  |  Execute subroutine X
     |                                  |  ---------------------
     |                                  |     do work...
     |                                  |     compute result...
     |                                  |  ---------------------
     |                                  |
     |   Response: Result of X          |
     <---------------------------------- 
```

**Unique features of RPC:**
* Supports multiple programming languages
* Applications written in different languages can talk to each other using RPC. (How? Explained below)

**Working of RPC**:
1. There exists a *contract* known as an **Interface Description Language (IDL)**
	- Essentially describes the exposed methods, arguments, response and corresponding data types 
	- IDL structure varies between implementations
2. An RPC code generation tool creates stubs on:
	- *Client*: Stub creates the methods the client can invoke
	- *Server*: Stub creates the methods that invoke the actual methods on the server for the logic
3. Code generation also *adds the data types as **classes / structs** / etc* (whatever the language supports as schema) when creating the stub. This is highly useful for type-safety and auto-completion!
4. Client request goes through the following steps:
	1. Call Function (local stub) just like you would a regular one (Ex: `user = getUser(id)`)
	2. Stub packs args into request message (**Data serialisation** or Marshalling)
	3. Send request over network (Transport layer)
	4. Server stub receives and unpacks request  (**Data deserialisation** or Marshalling)
	5. Server executes the actual function/method
	6. Packs result into response message (**Data serialisation** or Marshalling)
	7. Return response to client stub (Transport layer)
	8. Client stub unpacks response (**Data deserialisation** or Marshalling) and returns result as if it were a local call (Ex: `user = { name: ..}`)

Example IDL:
```
struct User {
    string name;
    int age;
};

interface UserService {
    User getUser(int id);
    bool updateUser(User u);
};
```

RPC illustration:
```
+-------------------+                     +--------------------+
|   Client Process  |                     |   Server Process   |
+-------------------+                     +--------------------+
          |                                          |
          | 1. Call Function (local stub)            |
          |----------------------------------------->|
          |                                          |
          | 2. Stub packs args into request message  |
          |                                          |
          | 3. Send request over network             |
          |----------------------------------------->|
          |                                          |
          |                           4. Server stub receives
          |                              and unpacks request   |
          |                                          |
          |                           5. Executes actual
          |                              function/method       |
          |                                          |
          |                           6. Packs result into
          |                              response message      |
          |                                          |
          |<-----------------------------------------|
          |        7. Return response to client stub |
          |                                          |
          | 8. Stub unpacks response and returns     |
          |    result as if it were a local call     |
          |                                          |
```

RPC is an old concept but over time the implementation, efficiency, and frameworks have been changing. As a developer:
* Pick a framework
* Define API and relevant data types using IDL

**Benefits of RPC**
* Clients & Servers can pick their own programming language
	* This leads to decoupling of the client from the server as long as the contract remains set
* Remote methods can be called like normal, local ones (Ease of use)
* ***Details of communication (protocol, payload, etc) are abstracted away from the client***

**Drawbacks of RPC**
* Client never knows how long the remote operation is going to take
	* Can be mitigated by providing asynchronous methods (Ex: think promises in JS)
* Since the call is made over a network, if the API is designed badly then debugging is confusing when things go wrong
	* This can be mitigated *partially* by providing *idempotent* operations

**When to use RPCs?**

RPCs are good for communication between *two backend systems* (Why? Abstracts away network details and complexities)
* Between different components i.e services of the same system
* Between APIs of two different systems i.e backend APIs of two different companies

**When not to use RPCs?**
1. Frameworks on the *frontend clients* do NOT usually support RPCs (Security?)
2. When we want to *take direct advantage of HTTP* and the benefits it provides (Headers, verbs, cookies, browser or CDN caching, etc)
3. When we focus on ***data*** rather than actions 
	* Think **CRUD** operations which are needed in very data-centric applications

(**Note**: RPC focuses on actions)

**Popular RPC frameworks**
1. **gRPC**: It uses `HTTP/2` as its transport protocol and Protocol Buffers as its Interface Description Language. Created by Google
2. **Thrift**: It is a lightweight, language-independent software stack for point-to-point RPC. Created by Meta
3. **Java Remote Method Invocation (RMI)**: RPC framework that allows one Java virtual machine to invoke methods on an object running in another Java virtual machine. RMI uses Java as the Interface Description Language.

### RESTful  APIs

REST is a *new style of API* published as dissertation way back in 2000

It is mainly useful for *client facing* & *data centric APIs*. Example: Fetching user details. Here, the user is the resource and we fetch a "*representation*" of the user (It need not be the exact user object in server code)

**What is REST?**
* Stands for **REpresentational State Transfer**
* It is a set of (1) Architectural constraints and (2) Best practices for defining APIs on the web

**What is REST not?**
* It is not a standard nor a protocol (Ex: You may theoretically use any protocol like HTTP, UDP, etc and any payload format)

**Goal of REST**: Create APIs that are *easy* for our clients to use and understand!
* Easy means easy to implement high availability, scalability, and performance

**What is a RESTful API?**
* Simply an API that adheres to the REST constraints
* It follows something known as *HATEOAS* (Hypermedia As The Engine Of the Application State)
	* Basically means we receive links to other related resources when asking for the details of one of them (Ex: such as the parent resource)
	* This is how we can navigate through the modern web, actually!
	* Ex: A `user` resource contains links to get details of `friends` and `posts`

**RPC vs REST**
| RPC | REST |
|--|--|
| Relies on methods | It is resource-oriented |
| API expansion through more methods | API expansion through more resource endpoints |
| New methods must be in IDL (static) | New endpoints can be added whenever (dynamic) |
| Methods are the abstractions | Named resources are the abstraction |
| Unlimited actions (add new methods) | Actions are restricted by the HTTP verbs (GET, PUT,..) |

***REST essentially manipulates resources via a small number of methods***

**REST in practice**
1. **HTTP** is the *more commonly used* application layer protocol to implement REST (By extension, TCP becomes the most commonly used transport layer protocol)
2. **JSON** is the most commonly used format for data payloads due to its easy to understand structure and the fact that most programming languages can understand and decode it

#### REST API quality attributes (Non-functional requirements)

1. **Server is stateless** 
	- No session information is maintained about the client
	- Each request should be served in isolation without any information about previous requests
	- *Why is this useful? The request can be routed to any server in a distributed environment.* 
2. **Cacheability**
	- A nearer CDN or edge server should be able to service some of the requests without hitting the actual server
	- Good for performance and UX
	- Server has to explicitly define whether a response is cacheable or not

**Food for thought**
> If we use session cookies instead of passing in an authorisation header in a request for user authentication then is the API still RESTful?

**Yes** — an API *can still be RESTful* even if it uses session cookies instead of an Authorisation header. REST does not mandate how authentication is passed. REST only requires:
* Statelessness (each request must contain everything needed to process it)
* Uniform interface
* Resource-based URLs
* Appropriate use of methods (`GET/POST/PUT/DELETE`)

Using cookies does not break REST, **as long as:**
* ✔ The session cookie is sent automatically with every request
	* Browsers do this → each request still carries the authentication context.
* ✔ The server does NOT store session state
	* Instead it uses stateless cookies like:
		* Signed cookies
		* Encrypted cookies
		* JWTs stored in cookies

This keeps the system REST-compliant.

✘ ***It becomes non-RESTful only when the server stores persistent session state***
* Traditional server-side sessions (where user state lives in memory/Redis) break REST’s statelessness rule.

**TL;DR**
* RESTful if: cookie is stateless (JWT/signed cookie)
* Not RESTful if: server stores session state for that cookie

*So: session cookies are fine; session state is what breaks REST.*

#### REST API endpoints

Each resource is <u>named</u> and <u>addressed</u> using a **Uniform Resource Identifier (URI)**. (Think of it as a the *path* to the resource on the API excluding the host & port details)

Resources have a *hierarchy* represented by **`/`** and each resource can be one of two types:
1. **Simple resource**: Has a *state* and can contain *one or more sub-resources*
	* Example: http://best-movies.com/movies/movie-01
	* Example: http://best-movies.com/movies/movie-01/actors/john-smith
	* Example: http://best-movies.com/movies/movie-01/actors/3114 (Parameter: actor id)
	* A resource can be expressed in a *variety of ways* (images, link to movie stream, object, HTML page, binary blob, JS executable, etc)
2. **Collection resource**: Contains a *list* of resources of the *same type*
	* Example: http://best-movies.com/movies
	* Example: http://best-movies.com/movies/movie-01/directors

Hierarchy: *Some resources are independent while some are strictly based on another resource*. Ex: `/movies` is independent while `/movies/{movie-id}/reviews` is not since reviews can only exist for a movie!

#### Best practices for resources

1. Name resources using *nouns*
	- Why? Makes a clear distinction from the actions we are going take on those resource
	- We will use HTTP verbs on those resources to define the actions
2. Make a distinction between collection resources and simple resource:
	- Collections must be in *plural* and simple in *singular*
3. Provide *clear* and *meaningful names*
	- Easy to use and understand
	- Avoid typos or inconsistent naming
	- Do not use generic names (Ex: entities, object, item, instance, etc) as they are not clear / specific enough
4. Resource URIs should be *unique* and *URL friendly*
	- There should only be one way to manipulate a resource via a URI
	- URL friendly means we should use characters supported by the web URLs

#### CRUD operations via HTTP verbs

"HTTP methods or verbs" are predefined constructs in the HTTP protocol and they help us define the action to take on a resource in REST

REST APIs are very good at manipulating data i.e a resource:
1. Create a resource  **(`POST`)**
2. Delete a resource **(`DELETE`)**
3. Update a resource **(`PUT`)** or `PATCH`
4. Read a resource **(`GET`)**

In some situations, we define additional custom methods i.e verbs (maybe not very RESTful but it is added as an exception so it is okay)

Note
1. `GET` is considered a "safe action"  as it reads but does not manipulate data
2. `GET` responses are considered "cacheable by default"
3. `POST` responses "can be made cacheable"
4. `GET`, `DELETE`, `PUT` are considered **idempotent** as re-applying them does not change the result (Unless your API handler is written badly!)
5. Additional information in the payload can be in JSON (popular) or XML/SOAP, etc

#### REST API step by step process

1. **Identify entities** (Ex: movies, directors, actors, reviews, users, ...)
2. **Map entities to URIs** (Ex: `/users`, `/users/{user-id}`, `/movies`, `/movies
3. **Define resource representations**
	- Ex: 
	```
	movies [
		{ name: ..., id: ... },
		{ name: ..., id: ... },
		{ name: ..., id: ... },
	]

	movie {
		name: ...,
		id: ...,
		description: ...,
		published: ...,
		reviews: [ review_id_1, review_id_2, ...],
		directors: [ director_id_1, director_id_2, ...],
		...
	}
	```
4. **Assign HTTP methods to operations, i.e actions, on resources** 
	- Ex: GET `/movies/movie-001` to read details of a movie
	- Repeat on all resources

### Why is gRPC popular?

gRPC is popular because:
1. It’s much faster than REST thanks to HTTP/2 and compact binary serialization (protobuf)
	- HTTP/2 allows multiplexing i.e many requests in parallel on one TCP connection
2. It provides strong type safety and autogenerated client/server code from `.proto` files
3. It supports real-time communication with built-in streaming (server, client, and bidirectional)
	- REST is request/response only
4. It works well in micro-service architectures and across multiple programming languages
5. It includes production-ready features like deadlines, retries, and authentication

**What are protocol buffers that gRPC uses?**

Protocol Buffers (Protobufs) are a binary, schema-based data serialization format created by Google
- You define your data structures in a `.proto` file (like an IDL)
- Code generators produce strongly typed classes in any language
- Data is encoded in a compact binary format → smaller + faster than JSON
- Used heavily in gRPC for efficient communication between services

In short: `Protobufs = tiny, fast, typed, binary JSON with a schema`.

**When should you use gRPC over REST?**

Use gRPC over REST when you need:
- High performance and low latency (e.g., microservice-to-microservice calls).
- Strongly typed contracts with autogenerated clients in multiple languages.
- Streaming (server, client, or bidirectional).
- Efficient communication over slow networks or between backend services.
- Consistent internal APIs in large, polyglot backend systems.

**Use REST when you need easy debugging, browser access, or public-facing APIs.**

Use REST instead of gRPC when you need:
- Easy consumption from browsers (*REST works natively over HTTP/1.1; gRPC does not*).
- ***Public-facing APIs*** where simplicity and broad compatibility matter.
- ***Human-readable payloads*** (JSON) for debugging, logging, and testing.
- ***Caching*** via standard HTTP semantics (`GET`, `POST`, `PUT`, `DELETE`).
- When you rely on passing information via HTTP headers and cookies along with the payload
- *Loose coupling and simple request/response patterns without streaming*.

*In short*: **choose REST when accessibility, simplicity, and compatibility are more important than raw performance.**

## Architectural blocks of large scale systems

### Load balancer

A load balancer is a component / service that "*balances the load among a group of servers*" in our system

**Case: Without a load balancer** (drawbacks of direct communication in a scale-out distributed system):
1. Client has to know the IP addresses of all the servers in our system
2. Client has to know the number of servers 

The system is no more a black box for the client. If one of the servers goes down then there is no way for the client to know immediately

```
                 ❌ NO LOAD BALANCER ❌
     -------------------------------------------------
     Client must:
       - Know ALL server IPs
       - Know HOW MANY servers exist
       - Handle failures itself
     -------------------------------------------------

                    +----------------+
                    |     Client     |
                    +----------------+
                     /       |       \
                    /        |        \
                   v         v         v
        +---------------+  +---------------+  +---------------+
        |   Server A    |  |   Server B    |  |   Server C    |
        +---------------+  +---------------+  +---------------+
              ^                   ^                   ^
              |                   |                   |
              |<--- client must manually track ------>|  
                    all servers + their health  
```

#### Benefit of using a load balancer

A load balancer sits in between our servers and the client and it:
1. Balances the load among servers
2. Abstracts away the details of servers from the client (Client need not know about IPs & #servers)

Our application basically becomes a *single* server for the client!

```
                   ✔ WITH LOAD BALANCER ✔
        -------------------------------------------------
        Client does NOT need to know:
          - Server IPs
          - Number of servers
          - Which server is healthy
        System becomes a black box to the client
        -------------------------------------------------

                          +----------------+
                          |     Client     |
                          +----------------+
                                   |
                                   v
                         +---------------------+
                         |    Load Balancer    |
                         +---------------------+
                           /        |        \
                          /         |         \
                         v          v          v
                +---------------+ +---------------+ +---------------+
                |   Server A    | |   Server B    | |   Server C    |
                +---------------+ +---------------+ +---------------+
```

#### Quality attributes of a load balancer

There are **4** quality attributes that a load balancer boosts:
1. **Scalability** 
	* An LB adds additional servers when load increases
	* An LB removes servers when load decreases (Saving costs)
	* *Auto-scaling*: Many Cloud-based LBs provide auto-scaling *policies* that are based on req/sec, bandwidth, etc (Intelligent scaling)
2. **Availability**
	* Load balancers can contain health monitoring service within it (Ex: pings and heartbeats to the servers)
	* If the health of a server is not good, it can stop routing requests to it and re-route to a healthy server until the issue is fixed
	* This keeps the system available to the user 
3. **Performance**
	* An LB does *add a tiny bit of latency* since it introduces another (set of) hop(s) to reach a server
	* However, this is offset by the extra load that it can take and distribute among different servers
	* Hence, compared to a single server, adding an LB + multiple servers distributed behind it is actually a *net positive in terms of performance* (Ex: N servers => N times more requests handled) 
4. **Maintenance**
	* We can use a LB to stop traffic to a server while it is being repaired
	* In this way, we can continue to support our SLA by distributing load among other servers instead of the one that needs to be serviced i.e no downtime

#### Types of load balancers

1. DNS load balancer
2. Hardware load balancer
3. Software load balancer
4. GSLB

#### DNS load balancer

DNS is a service that maps a domain name (Ex: `google.com`) to a single or a set of IP address (Ex: `234.1.23.44`, ...) 

* Used by N/W routers to route requests
* DNS is the "phonebook of the internet"

DNS load balancer essentially:
1. Knows the IP addresses of each of our servers
2. Returns the set of IP addresses to the client (which usually picks the first one)
3. When returning IP addresses, it rotates them using a *round-robin fashion* in order to distribute the load among the servers

**Benefits:**
* Simple
* Cheap (Comes by default when you purchase a domain name for your application and configure the IP addresses)

**Drawbacks**:
* DNS *does not* monitor the health of the system (It has no health monitoring system in place)
	* DNS will not know about a down server to not route requests to
* DNS *takes longer to identify a changed IP address for a server* 
	* It depends on a preconfigured TTL for a server before identifying that the IP is no longer in use. During this time, the routing might happen to a non-existent server 
	* On top of it, the IP addresses might be cached on the client computer - DNS has no way to invalidate this!
* The load balancing strategy is ALWAYS a simple round-robin
	* Does not take into account power of individual servers (some may have more compute than others)
	* DOes not take into account that some servers may be more overloaded than the others
* ***Important***: Client gets the direct IP of our servers (security issue - can lead to direct attacks on a single server, implementation detail unnecessarily shared)

```
                       DNS: "Phonebook of the Internet"

                 +----------------------+
  Domain Name -->|      DNS Server      |--> Returns IPs in
  google.com     +----------------------+    round-robin order
                         |    |    |
                         v    v    v
                [IP A] [IP B] [IP C]  (No health checks, no overload checks)


                           Client Request Flow
                           --------------------

                +---------+
                | Client  |
                +---------+
                     |
        DNS Lookup: "google.com" ?
                     |
                     v
         +-------------------------+
         |   DNS Load Balancer    |  (Round-robin rotation)
         +-------------------------+
               |         |         |
        returns IP A  returns IP B  returns IP C
          (this time)   (next)       (next)


Client → Directly Contacts Server (Security Drawback)
-----------------------------------------------------

       +---------+       +-------------+
       | Client  | ----> |  Server A   |
       +---------+       +-------------+
           |                 ^
           |-----------------|
             (Client may hit a dead server—DNS won't know)


                Summary of DNS LB Behavior
                --------------------------
     ✔ Simple  
     ✔ Cheap  
     ✔ Built-in with domain hosting  

     ✘ No health monitoring  
     ✘ Slow to update IP changes (TTL + client caches)  
     ✘ Always round-robin (no load/power awareness)  
     ✘ Exposes server IPs → security risk  
     ✘ Client may contact overloaded / dead servers  
```

#### Hardware and software load balancers

These types of LBs are better than a simple DNS. They pretty have the same LB function but with the following differences:
1. Hardware LB: Runs on a dedicated devices at the hardware level (routers, switches)
2. Software LBs: Runs as an application, i.e service, on any general-purpose computer

**Working/benefits:**
* ***All communication with servers from client is done by a load balancer***
	* Client *need not know* IP addresses or count of servers (Similar to DNS load balancer)
	* Security benefit!
* ***Contains health monitoring system*** i.e sends periodic health checks to the server
	* This was not possible with a simple DNS LB
	* Can actively re-route / redirect the request to a healthy server (big plus!)
* H/W and S/W LBs can ***balance the load based on hardware constraints, current load*** on each server and number of open connections(since we have the health pings that can fetch such metrics from the server)
	* Again, this was not possible with a simple DNS LB

```
                Hardware & Software Load Balancers
                ----------------------------------

      Hardware LB = Dedicated device (router/switch level)
      Software LB = Application/service running on a normal machine


                       Client → Load Balancer → Servers
                       --------------------------------


                    +---------+
                    | Client  |
                    +---------+
                         |
                         v
                +--------------------+
                |  HW / SW Load Bal  |
                |    (Smart LB)      |
                +--------------------+
                 |        |        |
                 v        v        v
           +---------+ +---------+ +---------+
           |Server A| |Server B| |Server C|
           +---------+ +---------+ +---------+


Key Behavior (Better than DNS LB):
----------------------------------

✔ Client never sees server IPs  
✔ LB handles **all** communication to servers  
✔ LB knows server health via periodic **health checks**  
    - Probes servers regularly  
    - Removes unhealthy servers immediately  

✔ LB supports **intelligent routing**:
    - Based on CPU load  
    - Based on memory usage  
    - Based on open connections  
    - Based on real-time metrics  

✔ LB can **reroute** requests instantly if a server goes down  
    (DNS LB cannot do this)

✘ Not tied to round-robin only — can use:
    - Least connections  
    - Weighted routing  
    - Resource-aware routing  
    - Sticky sessions  


High-Level Flow:
----------------

Client → Load Balancer  
Load Balancer → Selects healthiest / least-loaded server  
Load Balancer → Sends request to that server  
```

**Another major benefit: Internal set of load balancers**
* We can use H/W and S/W LBs ***internally as well*** 
* That is, we can separate out different groups of component, place them behind a load balancer, and scale each of them independently since the load balancer for each group takes care of distributing the load
* One load balancer for the entire system can route requests to load balancers of sub-systems (each sub-system now has different levels of distribution and scaling) 
* Ex: A load balancer each for frontend related servers, payment related ones, product catalog ones, etc

```
            Internal Load Balancing (Tiered Load Balancers)
            ------------------------------------------------
        We can place different components behind their own LBs
        and scale each independently.

                        +---------+
                        | Client  |
                        +---------+
                             |
                             v
                   +---------------------+
                   |  Main Load Balancer |
                   +---------------------+
                     /         |         \
                    /          |          \
                   v           v           v

      +----------------+  +----------------+  +----------------+
      | Frontend  LB   |  | Payments   LB  |  | Catalog   LB  |
      +----------------+  +----------------+  +----------------+
          |      |            |         |               |        |
          v      v            v         v               v        v
    +--------+ +--------+  +--------+ +--------+  +--------+ +--------+
    | FE S1 |   | FE S2 |   | Pay S1| | Pay S2|    | Cat S1|  | Cat S2|
    +--------+ +--------+  +--------+ +--------+  +--------+ +--------+


Key Benefits:
-------------
✔ Each subsystem has its **own LB**  
    - Frontend LB  
    - Payments LB  
    - Catalog LB  
    - etc.

✔ Each subsystem can be **scaled independently**  
    - Add/remove frontend servers without touching payments  
    - Add more payment servers without affecting catalog, etc.

✔ Main LB distributes high-level requests  
    → then hands off to subsystem LBs for fine-grained load distribution  

✔ Allows deep layering of load balancing:
        Main LB → Subsystem LB → Servers

✔ Enables clean separation of components  
    (microservices or tiered architectures)
```

**Colocating a hardware or software load balancer**

We do not have this freedom with a DNS load balancer. However, we do with a H/W or S/W load balancer.

Why is it important to place an LB close to our servers?
* ***Latency!*** It takes a long time for requests to reach our servers from an LB if it is not geographically co-located with them!
* This in turn affects UX on the client side

```

[client] --> [load balancer] ---- far away, takes long ---> geo-area[  [ server 1], [server 2] ]

[client] --> geo-area[ [load balancer] -- close , fast --> [ server 1], [server 2] ]

```


**Note**: 
* We *still need DNS resolution* to take place for a request to reach the load balancer
* So, a client still needs to initially interact with a DNS server somewhere to fetch the IP address. However, this time the client gets just one IP address, that of our load balancer
* If we have scaled our load balancers too then multiple IPs are returned, one for each LB

#### Global Server Load Balancing

**GSLB** is a *hybrid* between:
1. A DNS service
2. A hardware or software load balancer

A **GSLB**:
* Contains ***a health monitoring system***
	* Hence, can actively know when servers are down and re-route immediately
	* Can re-route to another data-centre in case of a disaster in one of them because of active routing capability
* ***Is aware of the location*** of our servers (i.e location of each of our "data centres")
	* Can intelligently route based on geo-location of the client w.r.t the servers

We generally *also have a load balancer at each geo-location* of a group of servers, i.e data centre region, in order to distribute load within them. Again, this is useful for taking more specific decisions and security such as client not being share an IP other than a local LB's IP or a set of local LBs' IPs if the LBs too were scaled.

GSLB can be configured based on:
* Current traffic
* Geo-location
* Best estimated response 
* Network bandwidth
* ...

```
                   GSLB (Hybrid: DNS + Smart Load Balancer)
                   -----------------------------------------

                          +--------+
                          |Client  |
                          +--------+
                               |
                         DNS Query: "myapp.com"
                               |
                               v
                       +----------------+
                       |     GSLB       |
                       | (Geo + Health) |
                       +----------------+
                       /        |        \
                      /         |         \
                     v          v          v
         Routes to nearest healthy region (i.e nearest data center) based on:
            - Geo-location of client
            - Server health (up/down)


     +-----------------+    +-----------------+    +-----------------+
     | Region A        |    | Region B        |    | Region C        |
     | Local LB -----> |    | Local LB -----> |    | Local LB -----> |
     | Servers         |    | Servers         |    | Servers         |
     +-----------------+    +-----------------+    +-----------------+

(Note: Each region can also have multiple local LBs if the scalability aspect demands it)

Key Points:
-----------
✔ GSLB = DNS-style global entry + smart LB logic  
✔ Performs **health checks** on regions  
✔ Knows **server locations**  
✔ Sends client to **nearest healthy region**  
✔ Each region still uses its own **local LB** to distribute load  
✔ Client never sees actual server IPs  
```

#### Common load balancer tools

1. **HAProxy**: HAProxy is a free and open-source, reliable, high performance TCP/HTTP load balancer
2. **Nginx**: NGINX is a free, open-source, high-performance HTTP server and reverse proxy (load balancer)
3. Cloud based:
	1. **AWS ELB (Elastic Load Balancing)**
	2. GCP Cloud Load Balancing
	3. Azure Load Balancer
4. GSLB Solutions:
	1. **Amazon Route 53**
	2. GCP Cloud DNS
	3. Azure Traffic Manager

### Message brokers

**Synchronous communication:**
- Both client and server have an open connection
- Request is sent and response is awaited immediately

Drawbacks of synchronous communication for *long processing tasks*:
1. Both client and servers apps need to remain healthy and maintain connection (Gets complex when a service task takes a long time to complete)
```
         +---------+
         | Client  |
         +---------+
              |
              |  HTTP Request
              v
    +----------------------+
    |  Frontend Service    |
    +----------------------+
              |
              |  Internal API Call (If this takes long, HTTP response is delayed!)
              v
    +------------------------+
    | Fulfillment Service    |
    +------------------------+

Note: It may take even longer if fulfillment service needs to do many things:
Billing, notifications, etc

If the app crashes, it is even worse. 
Client may need to retry and wait again for a long time - without guarantee
```

**Asynchronous communication with message brokers:**

**Message broker**: A software architectural building block that <u>uses the queue data to store messages</u> between senders and receivers

Message broker is used inside our system and *not exposed externally* (unlike an API)

Message broker basic capabilties:
1. Storing / temporarily buffering the messages
2. Message routing
3. Req/Response transformation
4. Req/Response validation
5. Load balancing (too!)

**Best benefit** of message brokers: **Entirely decouples** sender from receiver which is the basic building block of asynchronous communication
* *The receiver may not even be up & running when the sender sends a request that goes to the queue!*
* The client is usually given an immediate confirmation that the event was published -- and some additional data / link perhaps to track the status, etc. However, there is no immediate response to the request

```
         +---------+
         | Client  |
         +---------+
              |
              v
    +----------------------+
    |  Frontend Service    |
    +----------------------+
                    ^  |
                    |  Enqueue
                    |  v
        +----------------------------------------+
        | [msg1] | [msg2] | [msg3] | [   ] | ... |
        |          Message Queue                 |
        +----------------------------------------+
                           |
                           |  Dequeue / Consume
                           v
    +------------------------+
    | Fulfillment Service    |
    +------------------------+
```

**We can also have *multiple message brokers* to decouple the services from each other** (independent scaling!)

```
         +---------+
         | Client  |
         +---------+
              |
              v
    +----------------------+
    |  Frontend Service    |
    +----------------------+
              |
              |  Enqueue
              v
        +----------------------------------------+
        | [req1] | [req2] | [req3] | [   ] | ... |
        |      Queue: To Reservation             |
        +----------------------------------------+
                              |
                              |  Dequeue
                              v
    +------------------------+
    |  Reservation Service   |
    +------------------------+
              |
              |  Enqueue
              v
        +----------------------------------------+
        | [res1] | [res2] | [res3] | [   ] | ... |
        |       Queue: To Billing                |
        +----------------------------------------+
                              |
                              |  Dequeue
                              v
    +------------------------+
    |    Billing Service     |
    +------------------------+
              |
              |  Enqueue
              v
        +----------------------------------------+
        | [bill1] | [bill2] | [bill3] | [   ] | ... |
        |     Queue: To Notification             |
        +----------------------------------------+
                              |
                              |  Dequeue
                              v
    +------------------------+
    | Notification Service   |
    +------------------------+
```

**Message broker benefit (Pub/sub)**:
1. Message brokers offer the **publish/subscribe model** where 
	- Multiple services can publish to a channel
	- Multiple services can subscribe to a channel
		- They get notified when a new event is published
		- Easy to add and remove subscribers to and from a channel

Ex: A message broker queue for a topic/channel that the fulfilment service is listening to can also be subscribed to by a billing service / notification service. This can be done ***without any architecture changes***! Simply subscribe to the channel from the newly added service! **(Imp!)**

```
         +---------+
         | Client  |
         +---------+
              |
              v
    +----------------------+
    |  Frontend Service    |
    +----------------------+
              |
              |  Enqueue Event
              v
        +-----------------------------------------------+
        | [evt1] | [evt2] | [evt3] | [evt4] |  ...     |
        |              Central Event Queue              |
        +-----------------------------------------------+
              |                 |                  |
              |                 |                  |
     Dequeue / Consume   Dequeue / Consume   Dequeue / Consume
              |                 |                  |
              v                 v                  v
    +----------------+  +----------------+  +--------------------+
    | Fulfillment    |  |   Billing     |  |   Notification     |
    |   Service      |  |   Service     |  |     Service        |
    +----------------+  +----------------+  +--------------------+
```

#### Quality attributes of message brokers

1. **Fault tolerance** is the most important quality attribute of message brokers!
	- How? Allows different services to communicate with each other even if some of them are unavailable at the moment
	- Message brokers *prevent messages from being los*t (Holds them in the queue i.e persistent)
2. **Availability** increases (as a result of fault tolerance): Users will eventually be able to get their request processed and responded to when the receiver is able to get to it. Until then it is present in the queue
3. **Scalability** increases (as a result of fault tolerance): The system can *handle high spikes in traffic* owing to the fact that a message queue can buffer requests processed by our services (Ex: If an online store is running a promotion then more users will order during that window, but with a message broker, it can still handle the spike)

*Note*: **System takes a hit on performance** when message brokers are added (This is due to latency that is created and that is still okay for most systems)

#### Popular message broker tools

1. **Apache Kafka**
2. **RabbitMQ**
3. Cloud based:
	1. **Amazon Simple Queue Service (SQS)**
	2. **Google Pub/Sub** & Cloud Tasks
	3. Microsoft Azure Service Bus, Event Grid, ...

### API gateways

A single service (Here, a service may still refer to a group of servers but we are referring to a single feature domain, such as "billing" or "product catalog") when entrusted with multiple responsibilities finds it difficult and complex. Ex: A single service doing user profile management, video storage, security (Auth), serving frontend, and so on

To reduce complexity, we *split* a service into different smaller ones, typically based on responsibility (SRP). One each for:
* Serving frontend (Frontend service)
* User management (User service)
* Video service
* Comment service
* ...

This solves the responsibility problem and makes the services modular **BUT**:
1. The client's workload increases: 
	- Talks to multiple services (an earlier single API is now multiple APIs)
	- Ex: It needs to talk to three services (frontend, video, and comments) to load a single video on demand page like YouTube
2. Each service needs to implement its own security mechanisms (Ex: Auth)

Introducing the **API Gateway**:
* An API gateway sits in the middle of the client and our different services (middleman between client and services)
	* Client needs to make a call to just one service (to the one exposed by the gateway) (Similar to a load balancer)
* *Note*: Each service maybe a set of servers with application logic geared towards a particular feature
* API gateway is built on an architecture pattern called ***"API composition"***
	* Compose all different APIs of our different services into a single one that our client can call
	* Ex: Combine `/billing` and `/order` endpoints into one `/api` endpoint (Maybe `/api/v1/fulfillment`)

**Benefits of an API gateway**:
1. ***Seamless internal modifications / refactoring***
	- Ex: Adding a different service to serve mobile frontend to mobile clients (as opposed to desktop clients) is seamless as we add the service and make changes to the API gateway without the client needing to change its API calls
2. ***Consolidating all security, authorisation, and authentication in a single place***
	- A malicious user can be stopped at the API gateway itself (before reaching our services)
	- We can *authorise users for certain features* based on permissions (Ex: some users can edit a video while other cannot)
	- ***Rate-limiting***: This is a big one! We can prevent Denial of Service (DOS) attacks from malicious users by rate limiting them at the API gateway before reaching our services
3. **Request routing**: 
	* API gateways increase **performance**. How?
		* *All security measures are in one place* (Each service need not re-implement it)
		* Saves the user from making multiple requests to services i.e *One API request to the gateway* which then has the logic to make multiple requests to the services internally
4. **Static content and response caching**
	* If we cache responses for static content, we can immediately respond with it to the client without reaching our servers. This improves performance while fetching static content
5. **Monitoring and alerting**
	* Since *all requests pass through one single endpoint*, we can have monitoring and alerting system in one place, which give us a holistic view of the health of our entire system (Ex: Identify the service that is down or overloaded)
6. **Protocol translation**
	* Many of our services may work on different protocols & standards (Ex: HTTP1.0, ProtoBuf)
	* Many of our clients too may work on different protocols (Ex: REST API, SOAP, Thrift, etc) as they might be reluctant to support our protocols
	* The *API gateway is responsible for translating the protocols* so that client and service communication is unaffected due to differing protocols

```
                     +---------+
                     | Client  |
                     +---------+
                          |
                          | REST / HTTP
                          v
                +------------------------+
                |      API Gateway       |
                |  (Protocol Translation)|
                +------------------------+
                  |            |            \
      translates  |            |             \ translates
   HTTP → gRPC    |            |              \ HTTP → AMQP
                  |            |               \
                  v            v                v
      +----------------+ +----------------+  +----------------+
      |  Service A     | |  Service B     |  | Message Queue  |
      |   (gRPC)       | |   (gRPC)       |  |   (AMQP)       |
      +----------------+ +----------------+  +----------------+
```

**Main purposes of an API gateway**
1. API composition
2. Request routing (Note: This is not the same as routing in load balancers. That was only related to balancing load between instances or sub-systems. This is about *performance*: identifying the service that needs to be invoked after security aspects, protocol translation, etc have been taken care of -- all done in ONE PLACE, the API gateway!)

**Note**:
* API gateway DOES NOT try to distribute the load like a load balancer!
* API gateway routes requests only based on the API (Maps an API call to a service and this service might be behind a load balancer to distribute the load amongst its servers) 

**What should an API gateway not contain?**

An API gateway should not contain **business logic**
* It bloats the gateway with unmanageable code
* It becomes a single service that does all the work -- defeating the initial purpose of splitting a single service into many smaller ones

**Important consideration: Single Point of Failure (SPOF)**

Since all the traffic goes through a single gateway, it can lead to SPOF, and the system cannot be called as fault tolerant anymore.

If SPOF is a concern with increasing traffic, it is better to also ***scale an API gateway*** i.e have ***multiple API gateways in parallel***. This is similar to how we scaled a load balancer that happened to become a SPOF

```
               ┌────────────────────────────┐
               │          CLIENT            │
               └──────────────┬─────────────┘
                              |
                              v
                     ┌─────────────────┐
                     │      DNS        │
                     └───────┬─────────┘
                             |
                             v
            ┌────────────────────────────────────────┐
            │         MULTIPLE API GATEWAYS          │
            │   (Scaled horizontally to avoid SPOF)  │
            └────────────────────────────────────────┘
            |                                        |
            |     ┌──────────────┐   ┌──────────────┐|
            |     │ API GW #1    │   │ API GW #2     │
            |_____└──────┬───────┘___└──────┬────────┘
                         \                 /
                          \               /
                           v             v
                         ┌─────────────────┐
                         │   LOAD BALANCER │
                         └────────┬────────┘
                                  |
                                  v
                       ┌──────────────────────┐
                       │      SERVICES        │
                       └──────────────────────┘
```

**Note**: 
1. Do not put a load balancer in front of the API gateway. By putting a load balancer in front of API Gateways creates the exact SPOF you were trying to avoid.
2. Be very careful in pushing changes to the config in an API gateway. Any problem will affect security, translation, routing, and so many more functions of the system!
3. Do not try to bypass an API gateway
	- Making changes to a service interacting with clients via only a gateway is safe
	- All the safety that comes with a gateway vanishes when a services interacts directly with a client

#### Popular API gateways

1. Amazon API gateway
2. GCP API gateway
3. Azure API management
4. Netflix Zuul (free and open-source application gateway written in Java)

#### API Gateway vs Load Balancer

> A load balancer forwards requests from a client/sender to a physical machine/computer. Its role is to do what its name suggests, to balance the load on service across several instances. A load balancer can operate on a TCP (level-4) or HTTP (level-7) layer, but it's unaware of the concept of an API

> An API Gateway, on the other hand, forwards API calls from the client to different services.
Its purpose is not load-balancing. Instead, it forwards the requests, based on their API, to the service responsible for handling this API.

> Once a request arrives at a service, it will most likely hit its "load balancer" first. And the load balancer will forward the request to an actual computer running an instance of that service.

*Purpose*
* Load Balancer → 1. *Distribute traffic* across multiple servers to improve availability and scale.
* API Gateway → 1. *Manage APIs* and 2. *act as a single entry point for multiple backend services*.

*Layer of operation*
* Load Balancer → Works at L4 (TCP) or L7 (HTTP).
* API Gateway → Works at L7 only (application layer).

*Functionality*

- Load Balancer
	- Distributes traffic
	- Health checks
	- Failover
	- Simple routing (round-robin, least connections, etc.)

- API Gateway
	- Request routing by API paths
	- Authentication & authorization
	- Rate limiting & throttling
	- Request/response transformation
	- Caching
	- Logging, metrics, observability
	- Versioning + routing to microservices

*Client awareness*
- Load Balancer → Clients talk to a single endpoint but only for traffic distribution.
- API Gateway → Clients send all API calls through a single managed entry point for policy enforcement + aggregation.

*Use Case*

- Use a Load Balancer when:
	- You need to distribute traffic among identical servers
	- You want high availability for one backend service

- Use an API Gateway when:
	- You have multiple microservices
	- You need authentication, rate limiting, API keys, logging, etc.
	- You want to expose a unified API to clients

**TL;DR**

`Load Balancer = traffic distributor`
`API Gateway = traffic distributor + security + API management + smart routing`

**You can also use both together:**
`Client → API Gateway → Load Balancer → Servers.`

```
                           +---------+
                           | Client  |
                           +---------+
                                |
                                | 1. DNS Lookup
                                v
                        +----------------+
                        |      DNS       |
                        +----------------+
                                |
                                | 2. Returns API Gateway IP
                                v
                      +----------------------+
                      |     API Gateway      | (AUTHENTICATION)
                      +----------------------+
                                |
                                | 3. Route by API path
                                +-----------------------------+
                                |                             |
                                v                             v

                  (Service A LB)                    (Service B LB)
           +-------------------------+        +-------------------------+
           |    Load Balancer A      |        |    Load Balancer B      |
           +-------------------------+        +-------------------------+
               /            \                    /             \
              /              \                  /               \
             v                v                v                 v
     +-------------+   +-------------+  +-------------+   +-------------+
     | Service A1  |   | Service A2  |  | Service B1  |   | Service B2  |
     +-------------+   +-------------+  +-------------+   +-------------+
```

### Content Delivery Networks

Even with distributed systems placed globally over the entire world, request response times might still take a lot of time to reach the user. The reasons are:
1. Geography still plays a part even if there are distributed data centres
2. A request / response may go through many hops (think routers, switching ISPs) before reaching the destination

Google analytics study:
- More than 50% users leave a website if it takes more than 3 seconds to load (slow loading)
- Users perceive changes within 100ms as instant responses
- A good response time is within 1 second

Example:
```
- User in brazil
- Data centre in US

Time for a request to travel between Brazil and US: 200ms

Time to establish TCP connection (3-way handshake)
1. SYN: 200ms
2. SYN-ACK: 200ms
3. ACK: 200ms

Page load time:
4. Request for HTML page: 200ms
5. Response: 200ms

Assuming HTML page has 10 images to load from the same server
If images are loaded at once from the client (i.e all request sent at same time)
6. Images load time: 10 images x 200ms ~= 2000ms

Total time = 3200 ms
```

**CDN motivation**

Data centres can be closer to the user but that is not the bottleneck for a client. It is the *static content* served from these data centres that are many in number and could be large in size too! These include:
1. Images
2. HTML pages
3. JavaScript
4. CSS files
5. Videos

**CDN**:
* A globally distributed network of servers
* Located in strategic places
* Main purpose: Speeding up delivery of content to the end-users

**CDN working**
* They provide service by <U>caching</u> content on their "edge servers" which are located at different "Points of Presence"
* Edge servers: *Physically closer to the user* and *strategically located in terms of infra*

**CDN benefit**
* Allows transfer of content much faster to the end-user

**CDN quality attributes**
* **Performance**: Faster page loads, for example
* **High Availability**: Any issue or slowness is not noticeable (Since CDN still serves content if server is down)
* **Security**: Protection against Distributed Direct Denial of Service attacks (*DDOS*) as the resource is served from the edge server and not the system data centre

**Additional CDN techniques**
* Use faster and optimised storage devices
* Reduce bandwidth by compressing content delivered using algorithms such as:
	* gzip
	* brotli
	* JS file minification

**CDN strategies**

1. ***Pull strategy***
	* Client always hits the CDN edge first
	* If the CDN does not have the asset → pulls from origin
	* CDN caches the asset
	* Next clients get the content directly from the edge → fast
	* How long an asset is stored on the CDN is determined by a predefined value in config (TTL)
	* Past the TTL for an asset, it is invalidated i.e considered expired and maybe purged 
	* *Note*: If all assets were pulled at same time & have same TTL then you might see spikes in caches misses (failures) as all assets expired at the same time!

**First time load** of an asset is ***always slow in pull strategy*** on a CDN as it does not have it yet! Illustration:
```
                        ┌────────────────┐
                        │     CLIENT     │
                        └───────┬────────┘
                                |
                                v
                        ┌────────────────┐
                        │      CDN       │
                        │ (POP / Edge)   │
                        └───────┬────────┘
                        Cache Miss? ── Yes ───────────────┐
                                |                          |
                                v                          |
                        ┌────────────────┐                 |
                        │  ORIGIN SERVER │ <──────────────┘
                        │ (Your Backend) │   Fetches content
                        └───────┬────────┘
                                |
                                |  Sends asset (HTML, JS, images, video…)
                                v
                        ┌────────────────┐
                        │      CDN       │
                        │  Stores in     │
                        │     Cache      │
                        └───────┬────────┘
                                |
                                v
                        ┌────────────────┐
                        │     CLIENT     │
                        │ Gets response  │
                        └────────────────┘
```

2. ***Push strategy***
	* Origin → CDN: You push files to the CDN proactively (deploy time)
	* Client → CDN: Client fetches content that is already present on the CDN
	* CDN → Client: CDN returns the asset immediately (no cache miss)
	* *Note*: Reduces burden on system to maintain High Availability (HA) since CDN always serves the content 
	* *Push strategy is a problem if our content changes frequently*: Means we have to purge an redeploy content on CDN very often!

```
                 (1) Push / Upload static assets
┌────────────┐                                          ┌────────────┐
│  ORIGIN    │ ---------------------------------------> │    CDN     │
│  SERVER    │         (preloaded into CDN)             │ (Edge Cache│
└────────────┘                                          └─────┬──────┘
                                                              |
                                                              | (2) Client requests asset
                                                              v
                                                        ┌────────────┐
                                                        │   CLIENT   │
                                                        │  Browser   │
                                                        └─────┬──────┘
                                                              |
                                                              | (3) CDN serves directly
                                                              v
                                                        ┌────────────┐
                                                        │    CDN     │
                                                        │ (Edge Cache│
                                                        └────────────┘
```

**Push vs Pull**

| Aspect                   | **CDN Pull Strategy**                                    | **CDN Push Strategy**                                            |
| ------------------------ | -------------------------------------------------------- | ---------------------------------------------------------------- |
| How content reaches CDN  | Fetched **on-demand** from origin when first requested   | **Uploaded proactively** to CDN before users request it          |
| Initial request behavior | First request is slow (miss → fetch → cache)             | Served fast because content is preloaded                         |
| Deployment complexity    | Very low — no need to push assets                        | Higher — requires build/deploy pipeline to push assets           |
| Cache control            | TTL-based automatic cache population                     | Developer has full control over which assets exist in CDN        |
| **Ideal for**                | **Frequently updated or dynamic content**                    | **Large, static, versioned assets (videos, game bundles, firmware)** |
| Origin server load       | May get sudden spikes during cache misses or expirations | Very low — CDN serves everything without needing origin          |
| Risk                     | Cache miss → slower initial load                         | Missing file if not pushed explicitly                            |
| Best use case            | Websites, blogs, SaaS apps                               | Media delivery, gaming assets, app bundles, global distribution  |

#### Popular CDN providers

1. Fastly
2. Cloudflare
3. Akamai
4. Amazon CloudFront
5. GCP CDN
6. Azure CDN

## Data storage at global scale

### Relational databases

Data is modelled as a Table i.e Rows and Columns

The columns are predetermined and each of them have:
1. A name
2. A type
3. A set of constraints (Optionally)

A row in the table is identified uniquely by what is known as a **Primary Key** (PK)
- PK can be a single column value, or
- A set of columns (Composite)

```
Table Name: Users

+------------+----------------+---------------+-------------------+
|  Columns   |                |               |                   | --> (Fields/Attributes)
| (Header)   |   user_id      |   username    |      email        |
+------------+----------------+---------------+-------------------|
|            |      1         |    alice      |   alice@mail.com  | --> Row 1 (Record)
|    Rows    +----------------+---------------+-------------------+
| (Data)     |      2         |    bob        |   bob@mail.com    | --> Row 2 (Record)
|            +----------------+---------------+-------------------+
|            |      3         |    charlie    |   charlie@mail.com| --> Row 3 (Record)
+------------+----------------+---------------+-------------------+
```

#### Database schema

A relational database has a **Schema**
- It gives the database a ***structure*** ahead of time
- Gives us knowledge of what each column must have
- We can apply a *robust* query language like SQL to *analyze* and *update* the data

#### Structured Query Language

We can perform queries on relational databases using SQL (Structured Query Language)
- Different DBs have their own SQL implementation
- However, ***common SQL operations are standardized*** across the various relational DBs

#### Rise of Relational databases

Relational databases *eliminate the need for data duplication.* This is done by a process called ***normalization***. It became a proven way to hold data i.e store once, reference multiple times (This is usually done via something called a "Foreign Key" / FK)

Data storage is growing much more than ever before! -- Relational DBs save on these costs!

**Note**: We can perform a **`JOIN`** operation to get data from different tables using the Foreign keys
```
   TABLE A: USERS                     TABLE B: ORDERS
   +----+----------+                  +----------+---------+-------+
   | ID |   Name   |                  | Order_ID | User_ID | Price |
   +----+----------+                  +----------+---------+-------+
   | 1  |  Alice   | <--- MATCH --->  |   101    |    1    |  $50  |
   | 2  |   Bob    |       (X)        |   102    |    1    |  $20  |
   | 3  |  Charlie |                  |   103    |    2    |  $80  |
   +----+----------+                  +----------+---------+-------+
       (Charlie has no                  (Matches Alice & Bob only)
        orders, so he
        is excluded)

                    ||
                    ||  (JOIN Operation)
                    \/

             RESULT TABLE
   +----------+----------+-------+
   |   Name   | Order_ID | Price |
   +----------+----------+-------+
   |  Alice   |   101    |  $50  |
   |  Alice   |   102    |  $20  |
   |   Bob    |   103    |  $80  |
   +----------+----------+-------+
```

#### Advantages of Relational databases

1. Ability to form complex and flexible queries
2. Efficient storage
3. Natural structure of data for humans
4. ACID transaction guarantees

#### Properties of  Relational databases

**Transaction**: A transaction is a sequence of database operations that should appear as a single operation to an external observer i.e A transaction should appear as a whole or not have happened at all. No in-between! --> This is "Atomicity" of the ACID transactions

*ACID transactions*
1. **Atomicity**: "All or Nothing". If any part of the transaction fails, the entire operation is cancelled (rolled back)
```
       Transaction: Transfer $100
      /                          \
 [Debit User A]             [Credit User B]
    (Success)                  (FAIL!)
        \                         /
         \                       /
          -----> ROLLBACK <-----
          (Undo Debit on A. Result: No change)
```
2. **Consistency**: "Follow the Rules". The database must always remain in a valid state (e.g., no negative balances, data types match)
```
Rule: "Balance cannot be < 0"

   Start: Balance $50
          |
   Attempt: Withdraw $100
          |
      [ DB CHECK ]
          |
      "VIOLATION!" -> REJECT Transaction
          |
   End: Balance $50 (State remains valid)
```
3. **Isolation**: "Don't Interfere". Multiple transactions running at the same time should not see each other's half-finished work
```
User 1 (Buying last ticket)     User 2 (Buying last ticket)
               |                                 |
        [Read: 1 Left]                    [Read: 1 Left]
               |                                 |
        [Write: 0 Left]                   [Write: 0 Left]
               |                                 |
       (Locked by User 1)           (BLOCKED - Must Wait)
               |                                 |
           [COMMIT]                              |
                                         (Unblocked: Reads 0)
                                         (Fail: Sold Out)
```
4. **Durability**: "Saved means Saved". Once a transaction says "Success," the data is safe on the hard drive, even if the power goes out 1ms later
```
  Client -> "COMMIT"
        |
        v
 [Write to Disk Log]  <-- The "Checkpoint"
        |
      (Crash!)
        |
    (System Restart)
        |
 [Read Disk Log] -> Replay data -> Data is Safe.
```

#### Drawbacks of Relational databases

1. Rigid schema or structure
	- Schema needs to be defined ahead of its time
	- Change of schema would require *downtime for maintenance* which is not ideal
		- Need to *plan ahead* our schema in advance so as to *not change it* as much as possible
2. Hard to maintain or *scale*
3. Slower read operations

### Non-relational databases

Relatively new concept
- Popular in the 200s
- **Solves** the *disadvantages* of Relational databases

**Benefits**
1. It allows ***logical grouping of data without forcing all the records to have the same schema*** (Flexible schema)
2. ***Support more data structures*** (JSON/XML, lists, ...) than just the tables (Like Relational databases force you to)
	- This is more intuitive for programmers since it aligns closely with the data structures they use in their respective programming languages
	- ***Eliminates the need for ORMs*** to translate the business logic in a programminc language into the data modelling language

**Drawbacks**
1. We ***lose the ability to easily analyze records*** since they have flexible schemas
2. ***Analyzing multiple groups of records becomes very hard***! Why? There are *no* JOINs
3. ***ACID transaction are rarely supported***

### Key-Value Stores

**Concept**: The simplest NoSQL type, functioning like a ***giant dictionary or hash map***. Every item is stored as a specific *key* associated with a *value* (which can be a string, number, or binary object). ***You can only retrieve data if you know the exact key***
```
+-------+       +-------------------------+
|  KEY  | ----> |         VALUE           |
+-------+       +-------------------------+
| "u123"| ----> | {"name": "Sid", "age":25} |
| "sess"| ----> | "A8d9-31kL-99zX"        |
+-------+       +-------------------------+
```
- **Pros**: 
	- ***Extremely fast lookups*** (`O(1)`)
	- Simple data model
	- Scales horizontally easily
- **Cons**: 
	- ***No complex queries*** (cannot "search" by value content)
	- No relationships
- **When to Use**:
	- User sessions
	- Caching (Redis  - It is an **in-memory** key-value store)
	- Shopping carts
	- User preferences
- **CAP Tie-in**: **Usually AP (Available & Partition Tolerant)**. Systems like DynamoDB or Cassandra allow eventual consistency to ensure the system is always up, even if nodes disconnect

## Document Stores

**Concept**: Stores data in *semi-structured formats* like JSON or XML. Unlike rows in SQL, ***each document can have a unique structure (schema-less)***, and ***documents can contain nested structures (lists, sub-documents)***
```
Collection: Users
+--------------------------------+
| _id: 101                       |
| name: "Alice"                  |
| contacts: [email, phone]       | <--- Nested Array
+--------------------------------+
      |
+--------------------------------+
| _id: 102                       |
| name: "Bob"                    |
| address: {city: "NY", zip:99}  | <--- Nested Object
+--------------------------------+
```
- **Pros**: 
	- ***Flexible schema*** (good for agile dev)
	- Data is self-contained
	- Intuitive for developers (maps to code objects)
- **Cons**: 
	- ***Queries can be slower than key-value***
	- ***Data duplication is common to avoid joins***
- **When to Use**:
	- Content management
	- User profiles
	- Catalogs
	- Blogging platforms (MongoDB)
- **CAP Tie-in**: **Often CP (Consistent & Partition Tolerant)**. MongoDB, for example, defaults to strong consistency (you always read the latest write), sacrificing availability during a primary node failure

### Graph Databases

**Concept**: Designed to treat relationships as first-class citizens. Data is stored as Nodes (entities like People) connected by Edges (relationships like `"FRIEND_OF"`)
```
   (Alice) --[FRIEND_OF]--> (Bob)
      ^                       |
      |                       |
   [OWNS]                  [WORKS_AT]
      |                       |
      v                       v
   (Car)                  (Google)
```
- **Pros**: 
	- ***Blazing fast for deep relationship queries*** (e.g., "friends of friends")
	- Intuitive for connected data
- **Cons**: 
	- ***Difficult to scale horizontally*** (sharding a graph is hard)
	- Steep learning curve for query languages (Cypher, Gremlin)
- **When to Use**: 
	- Social networks
	- Recommendation engines
	- Fraud detection rings
	- Knowledge graphs
- **CAP Tie-in**: Typically CP (Consistent & Partition Tolerant) or CA (Consistent & Available). Graph databases like Neo4j prioritize data integrity (ACID compliance) over partition tolerance, making them harder to cluster across loose networks

### Database optimization techniques

#### Technique 1 - Database Indexing

A **database index** is a helper table, created from a particular column or group of columns

- ***Speeds up retrieval operations***
- ***Locates the desired records in a sub-linear time***
- Without indexing, these operations may:
	- Full table scan
	- Take a long time for large tables

```
      [ INDEX TABLE (B-Tree) ]                 [ ACTUAL DATA TABLE ]
      (Sorted Helper Structure)                (Random Storage on Disk)

      +---------+-----------+                   +-----+-------------+
      | Key(ID) | Pointer   |                   | ID  | Name        |
      +=========+===========+                   +=====+=============+
      | 005     | -> Row 2  |------------------>| 005 | Bob         |
      +---------+-----------+                   +-----+-------------+
      | 102     | -> Row 1  |------------------>| 102 | Alice       |
      +---------+-----------+                   +-----+-------------+
      | 550     | -> Row 500|------+            | ... | ...         |
      +---------+-----------+       |           +-----+-------------+
      | 999     | -> Row 3  |       +---------->| 550 | Dave        |
      +---------+-----------+                   +-----+-------------+
                                                | 999 | Carl        |
      ^                                         +-----+-------------+
      |
      SEARCH STRATEGY (Binary Search):
      1. Start at middle. Is 550 > 102? Yes. (Ignore top half)
      2. Jump to 550. Found!
      3. Follow Pointer to Row 500.

      Performance: O(log N) (Sub-linear)
      Impact: Retrieval is instant; no bottleneck.
```

**Drawbacks of a full table scan**: 
- These operations *if performed frequently on large tables* can:
	- Become a performance bottleneck
	- Impact our user's experience

```
      [ DATABASE TABLE (Unsorted Heap) ]
      (The database must read every row from top to bottom)

      Row 1:   [ ID: 102 | Name: Alice ]  <-- (Is this 550? No.)
      Row 2:   [ ID: 005 | Name: Bob   ]  <-- (Is this 550? No.)
      Row 3:   [ ID: 999 | Name: Carl  ]  <-- (Is this 550? No.)
       ...           ...
      Row 500: [ ID: 550 | Name: Dave  ]  <-- (Found it! After 500 checks)
       ...           ...
      Row 1000:[ ID: 888 | Name: Eve   ]

      Performance: O(N) (Linear)
      Impact: High CPU/IO usage. Slows down the entire system.
```

**Index table data structures**:
1. Hash map
2. B-Tree (A self-balanced tree)
3. Memtable (Used in LST-Trees with SSTables)

**Composite indexes**: Instead of making an index from one column, we can make index using more than one column too! This helps include more columns in the index, making reads faster. Caveat: If `(A,B,C)` is your composite index, you can only perform an index scan for `(A,B,C)`, `(A,B)`, or `(A)` (*prefixcontaining searches only*)

**Indexing trade-offs**
- Reads are fast!
- Writes are slow
- Additional space is required to store the indexes

**Note**: Indexing is also used extensively in non-relational databases such as "Document stores"

#### Technique 2 - Database Replication

A *single* database is a problem while scaling: A **single point of failure (SPOF)**

```
    [ Application Servers ]
             |   |
             |   | (Read/Write)
             v   v
      +-----------------+
      |  Single DB      |  <-- "I am overloaded!"
      |  (Instance A)   |      "I just crashed!"
      +-----------------+
              |
              X  <-- (Total System Failure)
```

**Database replication** solves this! There are two types
- Master-slave / Active-passive: There is one leader accepting writes while replicas perform reads. When leader dies, a new one is elected from among the replicas
- Leaderless / Active-active: There is no leader. All accept writes and reads. Conflicts are resolved using various techniques such as using a consensus (quorum) 
```
             [ Application Servers ]
             /          |          \
    (Write) /           | (Read)    \ (Read)
           v            v            v
  +-----------+    +-----------+    +------------+
  |  MASTER   |    |  SLAVE 1  |    |  SLAVE 2   |
  | (Writes)  |--->| (Read Only)|   | (Read Only)|
  +-----------+    +-----------+    +------------+
        |                ^                ^
        |                |                |
        +----------------+----------------+
             (Data Replication Stream)
        "Copy this new row to Slaves"
```

**Drawbacks of database replication**:
- *Higher complexity* when it comes to operations like write, delete, update
- It is not a trivial task to make sure *concurrent modifications to same records*:
	- Do not conflict with each other
	- Provide guarantees in terms of consistency and correctness

```
TIME      ACTION
 |
 v  1. WRITE (Update User: "Bob")
    [ App ] --------------------> [ MASTER DB ]
       |                            |  (Current: "Bob")
       |                            |
       |                            | (Network Delay / Lag)
       |                            v
    2. READ (Get User)            [ SLAVE DB ]
    [ App ] -------------------->   (Current: "Alice") <--- STALE DATA!
       |                               ^
       |                               |
       |                          (Update arrives 100ms later)
       |                               |
       v                          [ SLAVE DB ]
                                    (Now: "Bob")
```

Support for replication:
- **Non-Relational databases**: Replication is supported **Out-of-the-Box!**
- **Relational databases**: Replication support varies among different implementations

*Note: Replication turns our database into a distributed system!*

#### Technique 3 - Database Sharding or Partitioning

**Database sharding**: We split our data to be stored in different instances of the databases. Each instance is typically stored on a different computer

```
      [ HUGE DATASET (1 TB) ]
      (Too big for one machine!)
                |
                |  (SHARDING / PARTITIONING)
                v
      +-------------------+           +-------------------+
      |    SHARD  #1      |           |    SHARD  #2      |
      |   (Server A)      |           |   (Server B)      |
      |                   |           |                   |
      |  Users: [A - M]   |           |  Users: [N - Z]   |
      |  (500 GB Data)    |           |  (500 GB Data)    |
      +-------------------+           +-------------------+
```

**Why shard?** (Very difficult for one instance to store massively large amounts of data)
- We can *scale our database* to store more data
- Different queries can be performed in *parallel*
- We get both:
	- Better performance
	- Higher scalability

***Parallelism example***:
```
 [ User Query: "Alice" ]         [ User Query: "Zack" ]
             |                               |
             |                               |
             v                               v
    +-----------------+             +-----------------+
    |   SHARD #1      |             |   SHARD #2      |
    | (Processing...) |             | (Processing...) |
    +-----------------+             +-----------------+
             ^                               ^
             |                               |
      ( CPU / RAM )                   ( CPU / RAM )
      Resource Pool A                 Resource Pool B
```

Support for sharding:
- **Non-Relational databases**: A *first class feature* in non-relational databases since records are decoupled from each other (since more denormalized, nested, and with more duplication). It is natural to implement sharding on these systems
- **Relational databases**: Replication support varies among different implementations. Since `JOIN`s are difficult when a database is sharded i.e fetching related data from distributed servers, it is more challenging to implement

```
            [ Application Server ]
          (Needs to join "Alice" + "Orders")
                /           \
               / (1. Get ID) \ (2. Find Orders?) <--- COMPLEXITY!
              v               v
    +-----------------+   +-----------------+
    |    SHARD #1     |   |    SHARD #2     |
    |  (Holds Alice)  |   | (Holds Orders?) |
    +-----------------+   +-----------------+

    Problem: The database engine cannot do a simple "JOIN"
    because the data is on a physically different hard drive
    over the network. The App must manually stitch it together.
```

*Note: Sharding/Partitioning turns our database into a distributed system (similar to replication)!*

### CAP theorem

*CAP theorem*: In the presence of a network partition, a distributed database cannot guarantee both **Consistency** and **Availability** and has to ***choose only one of them***

We can only provide both consistency and availability when:
- There is no network partition
- Our replicas can freely communicate with each other

`C` = Consistency

`A` = Availability

`P` = Partition tolerance

```
        CONSISTENCY (C)
            /   \
           /     \
          /       \
    (CA) /         \ (CP)
        /           \ 
       /             \ 
      /               \
     /_________________\
AVAILABILITY (A)     PARTITION TOLERANCE (P)
           (AP)
```

**Consistency**: Every read request receives the **most recent write** OR an **error**!
- This means that all clients see the same value at the same time regardless of which database instance the client is talking to

**Availability**: Every request receives the **a non-error value**!
- Clients may get *different versions* of a particular record
- All requests return successfully with *a value*

**Partition tolerance**: System continues to operate despite an arbitrary number of messages being lost or delayed by the network between different computers

**CAP theorem interpretation**
1. `CP`: No availability
2. `AP`: No consistency

**Note**: **`CA`** is **not possible** since Partition tolerance can never be guaranteed in a distributed environment (A cable may break, humans may unplug devices, servers may crash, hardware faults, etc... cannot stop them from happening now and then)
- We can have a centrailzed database to provide Partition tolerance but it does not scale and hence, impractical when talking about large-scale systems

**When should we choose consistency, CP?**
- Imagine an online ordering store. Multiple users should see the same inventory count for a product. If one of them buys the last one, for example, another should not be able to place an order for the same item when the stock is empty for it

```
System chooses to error out rather than show wrong data.

     [Client Write v2]                 [Client Read?]
             |                                |
             v                                v
      +-------------+                 +-------------+
      |   Node 1    |    Network      |   Node 2    |
      | (Data: v2)  | --X-Broken-X--  | (Data: v1)  |
      +-------------+                 +-------------+
             |                                |
    (Updates cannot reach Node 2)             |
                                              v
                                       "ERROR / TIMEOUT"
                                     (Sacrifice Availability
                                      to ensure Consistency)
```

**When should we choose availability, AP?**
- Imagine counting the likes for a post. Many users might like a post but that does not mean we should not display the post until the most recent like has been shared with all instances. It is okay to display the post and show like count as 100 instead of 103 (most recent count)

```
System chooses to answer immediately, even if the data is old.

     [Client Write v2]                 [Client Read?]
             |                                |
             v                                v
      +-------------+                 +-------------+
      |   Node 1    |    Network      |   Node 2    |
      | (Data: v2)  | --X-Broken-X--  | (Data: v1)  |
      +-------------+                 +-------------+
             |                                |
    (Updates cannot reach Node 2)             |
                                              v
                                         "Data is v1"
                                      (Sacrifice Consistency
                                       to ensure Availability)
```

### Scalable unstructured data storage

**Unstructured data**: Data that does NOT follow a particular ***structure, schema, or model***
- Ex: Audio & video files, PDFs, images
- Known as a **Blob** (Binary Large Object)

**Where to store unstructured data?**
- Some DBs allow storing Blobs
- However, neither Relational nor Non-Relational DBs are optimized for storing Blobs
	- Additionally, they have a size limit for data e.g a record  (~1MB)

**Use cases for unstructured data**
1. Video/audio i.e entertainment or media files (compressed and stored)
2. Web assets that need to load when a webpage is opened (Ex: logo, article image, CSS/JS bundles)
3. **Database backup files** - these are large, so they are saved or archived in a *binary* format to save space (decompressed when accessed)
4. Sensor data from IoT devices (Machine learning and surveillance)

Blobs:
- Very big
- Datasets are also very big

```
   [ Video.mp4 ]  \
   [ Image.png ]   \     +-----------------------+
   [ Report.pdf ]   ---> |  BLOB (Binary Chunk)  |
   [ Backup.zip ]  /     | 101010110010101011011 |
   [ IoT_Log.txt ]/      +-----------------------+
                            ^
                            |
           "No strict Schema, Rows, or Columns"
```

#### Storage solution 1 - Distributed File System or DFS

Instead of using a single storage device, we use a *network of storage devices* connected to each other and provide:
- Replication
- Strong / Eventual consistency
- Autohealing

Benefits:
1. No need for special API
2. Modify files easily (Ex: modify a doc, append logs to a file)
3. **Very efficient** and **high performance I/O operations**

Limitations:
- ***Number*** of files is **limited** (**a scalability issue**)
- No easy access through Web APIs (REST+HTTP). Ex: Not easy to display a file on a web app through a web server, it needs special abstractions

```
 [Root Directory /]
             |
    +--------+--------+
    |                 |
 [Logs/]           [Media/]  <-- Hierarchical Folders
    |                 |
  sys.log           movie.mkv
    |                 |
    v                 v
 +------+         +-------+
 |Block1|         |Block A|
 +------+         +-------+
    |                 |
    v                 v
[Server 1]        [Server 2] ... [Server N]
 (Blocks distributed & Replicated across network)
```

#### Storage solution 2 - Object stores

A **scalable solution** for storing blobs/unstructured data

**Benefits:**
1. **Linear scalability**
2. **No limit on the number of files** we can create
3. **Very high limit on a single object size** (~5-10TB)
4. Provides HTTP + REST **API**
5. Supports **versioning** out-of-the-box

**Object store abstractions:**
- Objects are not stored in a hierarchical file structure
- They are stored in a *flat* structure called **buckets** (No limit on storage within a bucket except for our budget)

**Object contains:**
- An object ID
- The content
- Metadata (Information about the content. Ex: file size, format, etc)
- **Access control list (ACL)**: **Permissions that specify who can read/write or override the object permission**

Popular object stores (Cloud):
- Google Cloud Storage (GCS on GCP)
- Amazon S3
- Azure Blob
- Alibaba OSS

Usually, the top tiers in these tools provide **High Availability (HA)**

Note: Open source tools exist to store Objects on-premise if Cloud solutions' costs are too expensive!

**Scalability**:
Object stores use **data replication *under the hood***! (High Scale)

**Drawbacks:**
- Objects are **immutable**
	- We can ***only replace*** an existing object with a new version
	- Has ***negative performance implications*** (Cannot store log files and append data to them)
- ***No easy file-system like access*** that DFSes provide
	- Access through SDK / API
- ***Lower I/O performance*** compared to DFS

```
    [Client App / Browser]
             |
             v  (HTTP/REST API: PUT / GET / DELETE)
             |
   +-----------------------+
   |   Object Storage      |
   |      (Gateway)        |
   +-----------------------+
             |
   +---------+-----------+------------------+
   | Bucket: "Videos"    | Bucket: "Backups"|
   +---------+-----------+------------------+
             |                    |
    +-----------------+           v
    | Object ID: "v1" |     +------------------+
    |-----------------|     | Object ID: "b99" |
    | [METADATA]      |     |------------------|
    | Size: 5GB       |     | [DATA BLOB]      |
    | Type: .mp4      |     | 101010...        |
    | ACL: Public     |     +------------------+
    |-----------------|
    | [DATA BLOB]     |
    | 1101001...      |
    +-----------------+
```

```
FEATURE          |   DFS (File System)        |   OBJECT STORE (S3/GCS)
-----------------|----------------------------|--------------------------
Structure        |  Hierarchy (Tree/Folders)  |  Flat (Buckets & IDs)
Access Method    |  OS Mount / File Driver    |  REST API (HTTP)
Modifiable?      |  YES (Edit/Append)         |  NO (Write Once, Read Many)
Scalability      |  High (Petabytes)          |  Massive (Exabytes)
Best Use Case    |  Big Data Processing (Logs)|  Media hosting, Backups,
                 |  active working files.     |  Static Web Assets.
```

## Software architectural patterns

Patterns are used because they save time since they have been formed after other folks in the industry faced similar issues and came up with them

Patterns also prevent us from making our architecture a "Big ball of mud" (Messy, tightly coupled, difficult to understand, every server talks to every other server, and so on)

Patterns provide a vocabulary and helps other engineers follow the system

### Three-Tier Architecture

Three-Tier Architecture: The most common way to build web applications

#### The Core Concept

Instead of writing all your code in one giant file, you ***split your application into three distinct layers, each with a specific job***

Think of a Restaurant:
- Waiter (**Presentation Tier**): Takes your order and shows you the food. (Doesn't cook)
- Kitchen (**Application Tier**): Receives the order, cooks the food, follows recipes. (The brains)
- Pantry (**Data Tier**): Stores the raw ingredients (vegetables, meat). (The storage)

**The Three Layers**
- **Tier 1: Presentation Layer (The Frontend)**
	- What it is: The user interface. It’s what the user touches and sees on their screen
	- Tech: HTML, CSS, React, Mobile Apps
	- Job: Collects inputs (clicks, text) and displays outputs (images, data). It is "dumb"—it doesn't know how to calculate taxes, it just shows the final number
- **Tier 2: Application Layer (The Backend / Logic)**
	- What it is: The brains of the operation
	- Tech: Java (Spring), Python (Django), Node.js
	- Job: It processes data. If you try to log in, this layer checks if your password matches. It calculates prices, runs rules, and talks to the database
- **Tier 3: Data Layer (The Database)**
	- What it is: The long-term memory
	- Tech: MySQL, PostgreSQL, MongoDB
	- Job: Stores and retrieves data. It doesn't care why you need the data, it just gives it to the Application Layer

```
(1) CLICK LOGIN
          |
          v
+-----------------------------+
|  TIER 1: PRESENTATION       |  <-- "Client"
|  (Browser / Mobile App)     |      (React / iOS)
+-----------------------------+
          |
          | (2) Send HTTP Request (username/pass)
          v
+-----------------------------+
|  TIER 2: APPLICATION        |  <-- "Server"
|  (The Logic / API)          |      (Java / Python)
|                             |
|  * Checks logic             |
|  * Hashes password          |
+-----------------------------+
          |
          | (3) Query: SELECT * FROM users...
          v
+-----------------------------+
|  TIER 3: DATA               |  <-- "Database"
|  (Storage)                  |      (SQL / NoSQL)
+-----------------------------+
```

#### The "Monolith" Connection

In the early stages of a startup, these three layers often live inside a Monolith!

What is a Monolith here? It means that while the code is separated into layers (folders for UI, Logic, DB scripts), the ***Application Layer (as per the OSI model) runs as a single giant program on one server***

```
      USER
       |
       v
+-----------------------+
|     THE SERVER        |
| +-------------------+ |
| |   Presentation    | |  <-- HTML is generated here
| +-------------------+ |
|         |             |
| +-------------------+ |
| |   Application     | |  <-- All logic in one process
| +-------------------+ |
+-----------------------+
           |
           v
+-----------------------+
|     DATABASE          |
+-----------------------+
```

#### Pros and cons

**Pros**
1. Simplicity: Easy to develop and test locally. You just run one app
2. Security: The database is hidden behind the Application layer. Users can't touch the DB directly
3. Performance: Since the Logic and UI often live on the same server in a monolith, communication is fast

**Drawbacks**
1. **Scalability**: If the "Logic" part gets too heavy, you have to duplicate the entire server to handle more traffic
2. **Rigidity**: If you want to change the Application tech (e.g., Java to Python), you have to rewrite the whole middle tier
3. **Single Point of Failure**: In a Monolith, if one line of code crashes the Logic layer, the whole website goes down

### Microservices architecture

In software, **Microservices** means *breaking one big application into many small, independent mini-applications that talk to each other*

Imagine a Swiss Army Knife (The Monolith). It has a knife, a corkscrew, and scissors all glued together. If you want to sharpen the knife, you have to take the whole tool to the shop. If the scissors break, you might have to throw the whole thing away.

Now imagine a Toolbox (Microservices). You have a separate hammer, a separate screwdriver, and a separate wrench. If the hammer breaks, you just buy a new hammer. The screwdriver still works. If you need a better wrench, you upgrade just the wrench

#### The Architecture (The Layers)

In a microservices world, we don't just have one server. *We have a **swarm** of services*

The Flow:
1. **Client**: The user's phone or browser
2. **API Gateway**: The "Receptionist." It takes all requests and routes them to the right service
3. **Microservices**: The workers. Each does one specific job
4. **Databases**: Crucial difference! Each service has its own database. Service A cannot touch Service B's database directly
```
      (User App)
           |
           v
+-----------------------+
|     API GATEWAY       |  <-- The Entry Point
| (Routes traffic)      |
+-----------------------+
      /    |    \
     /     |     \  (HTTP / gRPC calls)
    v      v      v
+------+ +------+ +------+
| USER | | CART | | SHIP |  <-- The Microservices
| SERV | | SERV | | SERV |
+------+ +------+ +------+
   |        |        |
   v        v        v
(UserDB) (CartDB) (ShipDB)  <-- Separate Databases
```

#### Why is it Needed?

The Monolith (the old way) has a major flaw: ***Coupling***. In a monolith, if the "Shipping" feature has a bug that crashes the server, the "Login" feature also goes down. *The whole site dies*

**Microservices** are needed for **Decoupling**.
- **Scale**: If millions of people are browsing products but only 5 people are buying, you can create 100 copies of the Catalog Service and keep only 1 Payment Service. In a monolith, you'd have to clone the whole giant thing
- **Speed**: One team can work on the Cart while another works on Payments. They don't block each other

#### Pros and cons

**Pros**
- **Scalability**: You can scale only the specific services that are hot or busy, rather than the entire application
- **Resilience**: The system prevents total failure. If one service (e.g., "Reviews") crashes, other parts (e.g., "Checkout") continue to work
- **Tech Freedom**: Different teams can use different technology stacks (e.g., Java for Search, Python for AI) without needing to agree on a single language

**Drawbacks**
- **Complexity**: Managing the infrastructure becomes difficult as you move from one big server to many small servers
- **Network Latency**: Communication between services happens over the network (HTTP/gRPC), which is significantly slower than internal function calls in a monolith
- **Data Consistency**: Because databases are often separated, ensuring data stays in sync across services is difficult (e.g., handling scenarios where Payment succeeds but Shipping fails)

#### One database per microservice

The Core Reason: **Decoupling**
- The #1 rule of microservices is ***independence***. If Service A and Service B share the same database, they are married. If Team A changes a table name (e.g., `user_id` to `uuid`), Team B's code breaks. Both teams must constantly hold meetings to agree on every database change. By giving each service its own database, Team A can change their schema whenever they want without breaking Team B

```
    [Service A]      [Service B]
        |                |
        |                |
    +------------------------+
    |    SHARED DATABASE     |  <-- Single Point of Failure
    | (Table A)    (Table B) |      & Bottleneck
    +------------------------+
```

```
   [Service A]                   [Service B]
        |                             |
        v                             v
 +-------------+               +-------------+
 |   DB  (A)   |               |   DB  (B)   |
 |   (SQL)     |               |  (NoSQL)    |
 +-------------+               +-------------+
```

**"Polyglot Persistence"** (The Right Tool for the Job)
- Since the databases are separate, they don't even have to be the same type of database
- Payment Service: Needs strict transactions --> Uses MySQL (SQL)
- Catalog Service: Needs flexible product attributes --> Uses MongoDB (NoSQL)
- Social Service: Needs friend connections --> Uses Neo4j (Graph)

In a shared monolith database, everyone is forced to use the same technology (usually SQL), even if it's bad for their specific use case

**Scalability & Fault Isolation**
- ***Scaling***: If the "Order Service" is getting hammered with millions of writes, you can upgrade just that database to a bigger server. You don't need to pay to upgrade the "User Database" which might be idle
- ***Isolation***: If a bad query crashes the "Order Database," the "User Service" stays alive. In a shared DB, one bad query can take down the entire company

#### Real World Use Case

E-Commerce (e.g., Amazon)
- When you load a product page on Amazon, you aren't talking to one server. You are hitting ~100 microservices at once
- - Single page load exmaple:
```
Request: "Show me the iPhone 15 Page"
        |
        v
   [API Gateway]
        |
   +----+--------------------------+-----------------------+
   |                               |                       |
   v                               v                       v
[Inventory Service]         [Pricing Service]       [Review Service]
"Do we have stock?"         "What is the cost?"     "Fetch 5 stars"
   |                               |                       |
   v                               v                       v
(Stock DB: Yes, 5 left)     (Price DB: $999)        (NoSQL DB: Great phone!)
```

#### Summary

- Monolith: Easy to build, hard to scale
- Microservices: Hard to build, easy to scale

Rule of Thumb: Don't start with microservices. Start with a monolith, and break it up only when your team grows too big to manage one code base

### Event-Driven Architecture

In a traditional system (***Request-Response***), when Service A wants Service B to do something, it calls it directly and waits on the phone until B is done

**In Event-Driven Architecture**, Service A doesn't tell anyone what to do. Instead, it just shouts, "Hey! Something happened!" (an Event) and goes back to work. It doesn't care who listens

Analogy:
- Request-Response: You call a friend and ask, "Please come to my party." You wait for them to answer "Yes/No"
- Event-Driven: You send an invite. You don't wait. Your friend sees the invite (Event) whenever they check their mail and decides to come

There are three main parts to this system:
1. **Producer (Emitter)**: The service where the action happens. It creates the *event*
2. **Message Broker**: The "Middleman" or "Post Office." It *receives the event and holds it safe*
3. **Consumer**: The service that *waits for events*. When it sees one, it *wakes up and does its job*

```
 [ PRODUCER ]              [ MESSAGE BROKER ]             [ CONSUMER ]
  (Order Service)           (RabbitMQ / Kafka)            (Shipping Service)
       |                            |                            |
       | 1. "Order Placed!"         |                            |
       | (The Event)                |                            |
       +--------------------------> |                            |
       |                            |                            |
       |                            | 2. "Hey, new event!"       |
       |                            +--------------------------> |
       |                            |                            | 3. Process Logic
       |                            |                            |    (Ship Item)
```

#### Why is it needed?

To understand why we need it, let's look at what happens without it.

**Scenario A: The Bad Way (Direct HTTP Calls)**
- The Order Service calls the Shipping Service directly
- Problem 1: If Shipping is down, the Order fails. The user sees an error
- Problem 2: The Order Service has to wait (latency)
```
 [Order Service] --(Wait...)--> [Shipping Service (CRASHED!)]
      |
      v
   "ERROR 500: Cannot order"
```

**Scenario B: The Good Way (Event-Driven)**
- The Order Service drops a message in the Broker and immediately tells the user "Success!"
- Benefit: If Shipping is down, the message just sits in the Broker (Queue) waiting. When Shipping comes back online 1 hour later, it processes the message. **No data is lost**
```
 [Order Service] --(Drop Message)--> [ Broker ]           [Shipping Service (CRASHED)]
      |                               |                       (Offline)
      v                               v
"Success! Order Placed"          [Message Saved]
                                      |
                               (Later, when online)
                                      |
                                      +----------------> [Shipping Service (ONLINE)]
```

#### Why do Microservices use it?

Microservices architectures use EDA primarily for **Decoupling** and **Buffering**

1. **Decoupling (Independence)**. In a large company like Uber or Netflix, the team building the "Signup Service" shouldn't need to know about the "Welcome Email Service.
	- With EDA: The Signup Service just says "New User!"
	- Any team can add a new Consumer (e.g., "Analytics Service" or "Coupon Service") to listen to that event without touching the Signup Service code.

2. **Handling Spikes (Traffic Buffering)**. Imagine a flash sale (Black Friday). 1 million users order in 1 second
	- Without Broker: The Shipping Service explodes and crashes under the load
	- With Broker: The ***Broker acts like a Buffer*** (***Dam***). It catches all 1 million requests. The Shipping Service processes them at its own pace (e.g., 500 per second) until the queue is empty

```
 HUGE TRAFFIC              BUFFER                 SAFE PROCESSING
      |||                      |                         |
      vvv                      |                         v
[ 1M Orders ]  ----->  [ MESSAGE BROKER ]  ----->  [ Shipping ]
                       [ |||||||||||||| ]          (Takes 1 at a time)
                       [ |||||||||||||| ]
```

### Event-Driven Patterns

#### The SAGA Pattern or the Undo button

**The Problem**: In a Microservices world, a single "transaction" (like booking a trip) involves multiple services (Flight, Hotel, Car). If the Flight books successfully, but the Hotel is full, you can't just "cancel" the database transaction because they are different databases. You have a partial data mess

**The Solution (Saga)**: Break the long transaction into a chain of small local transactions. If one fails, you run a "Compensating Transaction" (an undo step) to reverse the previous steps.

Happy path:
```
[Order Service] ---> [Payment Service] ---> [Shipping Service]
   (Success)            (Success)              (Success)
```
The Failure Path (Rollback) Imagine Shipping fails. The Saga runs backwards to clean up.
```
1. [Order Service] Created Order #101.
        |
        v
2. [Payment Service] Charged $50.
        |
        v
3. [Shipping Service] ERROR! Out of Stock!
        |
        v
   (TRIGGER SAGA ROLLBACK)
        |
        v
4. [Payment Service] **REFUND** $50. (Compensating Action)
        |
        v
5. [Order Service] **CANCEL** Order #101. (Compensating Action)
```

The Saga Pattern is a ***way to manage data consistency across microservices in distributed transactions***. Instead of one big lock on a database, you break the transaction into smaller, local steps. If one step fails, you run "compensating transactions" (undo steps) to fix the data

There are two ways to coordinate these steps: **Choreography** and **Orchestration**
1. **Choreography** (**Decentralized**): In Choreography, there is *no* central leader. Each microservice knows what to do and reacts to events published by other services. It is like a dance troupe where every dancer knows their part and reacts to the music or the person next to them.
	- How it works:
		- Service A does its work and emits an event (e.g., "Order Created")
		- Service B listens for that event, does its work, and emits a new event
		- Service C listens for that, and so on
```
      (Start)
         |
    [Order Service]
         |
         | "OrderCreated" Event
         v
    [Payment Service]
         |
         | "PaymentProcessed" Event
         v
   [Inventory Service]
         |
         | "StockReserved" Event
         v
      (Finish)
```
2. **Orchestration**: In Orchestration, there is a ***central coordinator*** (the Orchestrator or Saga Manager). This manager tells every service what to do and when. It is like an orchestra conductor waving the baton to tell the violins when to play
	- How it works:
		- The Orchestrator receives a request
		- It sends a command to Service A ("Create Order"). Service A replies "Done"
		- It sends a command to Service B ("Process Payment"). Service B replies "Done"
		- If Service B fails, the Orchestrator sends a command to Service A ("Undo Order")

|Feature              |Choreography                                                                |Orchestration                                                                                                       |
|---------------------|----------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
|Team Size            |Small, autonomous teams.                                                    |Larger teams needing strict control.                                                                                |
|Complexity           |Low (2-4 services).                                                         |High (4+ services).                                                                                                 |
|Visibility           |Low (Need log aggregation).                                                 |High (Dashboard visible).                                                                                           |
|Maintenance          |Harder as system grows.                                                     |Easier to maintain logic in one place.                                                                              |

Case Study: When would you choose one over the other?
- Scenario: An E-commerce Checkout System
	- Start with Choreography if: You are a startup with only 3 services: Order, Payment, and Email
	- Why? It is overkill to build a dedicated Orchestrator for a linear flow (Order -> Pay -> Email)
	- Choreography is faster to build and deploy
	- Switch to Orchestration if: 
		- Your business grows. Now, a checkout involves: Order, Payment, Inventory, Shipping, Fraud Check, Loyalty Points, and Referral Bonuses
		- The Problem with Choreography here: If the "Fraud Check" fails, who tells "Loyalty Points" to rollback? Who tells "Inventory" to restock? In choreography, every service has to listen to "FraudFailed" events, creating a massive, tangled web of logic
		- The Solution: You introduce an Order Orchestrator. It calls the services step-by-step. If Fraud Check fails at step 5, the Orchestrator simply runs the "Compensate" command for steps 4, 3, 2, and 1

*Summary Rule of Thumb*: Use ***Choreography for simple, linear flows where visibility isn't critical***. Use ***Orchestration for complex flows where you need centralized control, easy rollbacks, and clear status tracking***

#### CQRS or Command Query Responsibility Segregation

**The Problem**: In a traditional database, the same model is used for writing and reading. ***Writes need complex validation*** (normalized data). ***Reads need fast joins and dashboards*** (denormalized data). **Using one database for both creates a bottleneck**

**The Solution (CQRS)**: Split the system into *two* parts:
1. **Command Side (Write)**: Optimized for updates/inserts. Uses a normalized database (e.g., PostgreSQL)
2. **Query Side (Read)**: Optimized for fast viewing. Uses a NoSQL DB or Cache (e.g., Elasticsearch, Redis)
3. **Sync**: Events update the Read side ***asynchronously***

```
      [ USER / CLIENT ]
        /             \
       / (Writes)      \ (Reads)
      v                 v
+-------------+    +-------------+
| COMMAND API |    |  QUERY API  |
+-------------+    +-------------+
      |                   ^
      v                   |
 [ WRITE DB  ]       [ READ DB   ]
 (Postgres)          (ElasticSearch)
      |                   ^
      |                   |
      +----(Events)-------+
      "Item Created"
      "Item Updated"
```

**Event Sourcing (The "Ledger")**

**The Problem**: In a normal database (CRUD), you only store the ***current state***

Table: `User: { Name: "Bob", Balance: $100 }`. If Bob had $50 yesterday, that information is lost forever. You don't know how he got to $100

**The Solution (Event Sourcing)**: Don't store the current state. ***Store the history of events that led to the state***. To find the current balance, you ***replay*** (add up) the events

Traditional DB:
```
| ID | Name | Balance |
|----|------|---------|
| 1  | Bob  | $100    |  <-- You only see this.
```
Event Sourced DB (The Log):
```
Event Stream:
1. [UserCreated]  Name: Bob, Balance: $0
2. [Deposited]    Amount: $50
3. [Deposited]    Amount: $60
4. [Withdrawn]    Amount: $10
-----------------------------------------
Current State (Calculated): $0 + 50 + 60 - 10 = $100
```

|Pattern|Best For...     |
|-------|----------------|
|Saga   |Complex flows across microservices (e.g., Booking a vacation: Flight + Hotel + Car)|
|CQRS   |Systems with high read traffic and complex search requirements (e.g., Amazon Product Search vs. Amazon Inventory Admin)|
|Event Sourcing|Systems requiring strict audit trails or "undo/replay" capability (e.g., Banking, Accounting, Legal systems)|

### Message delivery semantics and guarantees

These are the "promises" a messaging system (like Kafka or RabbitMQ) makes about whether your message will arrive safely

1. ***At-Most-Once (Fire and Forget)***
	- The Promise: "I will try to send the message once. If it gets lost, too bad. I won't try again."
	- How it works: The Producer sends a message and immediately moves on. It doesn't wait for a confirmation receipt
	- Risk: ***Data loss***
	- Performance: ***Highest*** (no waiting)
	- Use Case:
		- ***IoT Sensor Data***: If you miss one temperature reading out of 10,000 sent every second, it doesn't matter
		- ***Log Files***: Losing one line of a debug log is usually acceptable
```
 [ Producer ]                      [ Broker ]
     |                                 |
     | --(Msg: "Update View Count")--> X (Lost in network!)
     |                                 |
     | (Does not retry)                |
     v
(Continues working...)
```
	
2. ***At-Least-Once (The Standard)***
	- The Promise: "I guarantee the message will arrive. However, in rare cases, it might arrive twice"
	- How it works: The Producer sends a message and waits for an ACK (Acknowledgement)
		- If the ACK is received -> Success
		- If the ACK is lost (network failure), the Producer thinks the message failed and sends it again
	- Risk: ***Duplicate messages***. (The ***consumer must be Idempotent to handle this***—meaning processing the same message twice doesn't break anything)
	- Performance: ***Medium*** (Retries cost time)
	- Use Case:
		- ***Most Systems***: Payment processing, Order creation. ***It is better to process an order twice (and filter the duplicate later) than to lose the order entirely***
```
 [ Producer ]                         [ Broker ]
     |                                    |
     | --(Msg: "Charge $10")------------> | (Saved!)
     |                                    |
     |             (Network Cut!)         |
     | X <-------(ACK: "Got it")--------- |
     |                                    |
     | (Didn't get ACK, waiting...)       |
     | (Timeout! Retrying...)             |
     |                                    |
     | --(Msg: "Charge $10")------------> | (Saved Again!)
     v                                    v
                                   [ "Charge $10", "Charge $10" ]
                                   (Duplicate Created)
```
3. **Exactly-Once (The Holy Grail)**
	- The Promise: "The message will arrive exactly one time. No loss, no duplicates"
	- How it works: This is ***very hard to achieve***. It usually requires a ***transactional coordination*** between the Producer and the Broker (like "Transaction ID" checks)
	- The system tracks unique IDs. If it sees `ID #101` again, it discards it automatically
	- Risk: ***Complexity and slowness***
	- Performance: ***Lowest*** (Heavy overhead)
	- Use Case:
		- ***Financial Trading***: You cannot buy a stock twice just because of a network glitch
		- ***Voting Systems***: Every vote must count exactly once
```
 [ Producer ]                         [ Broker ]
     |                                    |
     | --(ID: 101, Msg: "Vote A")-------> | (Checks ID Log...)
     |                                    | "New ID. Save it."
     |                                    |
     | <---------(ACK lost)-------------- X
     |                                    |
     | --(ID: 101, Msg: "Vote A")-------> | (Checks ID Log...)
     |        (Retry)                     | "Saw 101 already. Ignore."
     v                                    v
                                    [ "Vote A" ] (Only 1 stored)
```

|Guarantee|Reliability     |Duplicates?|Performance|Best For...                         |
|---------|----------------|-----------|-----------|------------------------------------|
|At-Most-Once|Low             |No         |Fast       |Analytics, Logging, Sensors         |
|At-Least-Once|High            |Yes        |Medium     |Payments, Orders (Needs Idempotency)|
|Exactly-Once|Very High       |No         |Slow       |Stock Trading, Accounting           |

### Strategy for processing infinite streams of events

**The Problem: Infinite Data** You cannot "sum up" a stream that never ends. You can't say "Count all clicks" because new clicks keep coming forever. 

**Solution**: You *split* the stream into *chunks* called **Windows**

#### Tumbling Window

- **Concept**: ***Fixed-size chunks that do not overlap***. As soon as one finishes, the next one starts
- Analogy: A train. As soon as one car ends, the next one begins. You can only be in one car
- Use Case: "Count requests per minute." (`00:00-00:01`, `00:01-00:02`)

```
Stream:  [e1, e2]   [e3, e4]   [e5]
            |          |         |
Windows: |00-10s|   |10-20s|  |20-30s|
         +------+   +------+  +------+
```

```python
# Count events every 10 seconds
def tumbling_window(stream):
    bucket = []
    window_start = time.now()
    
    for event in stream:
        if time.now() - window_start >= 10_seconds:
            print(f"Count: {len(bucket)}") # Process Window
            bucket = []                     # Clear Bucket
            window_start = time.now()       # Start New
        bucket.append(event)
```

#### Hopping Window

**Concept**: ***Fixed-size chunks that overlap***. The window "hops" forward by a smaller step than its size

Analogy: You are analyzing the "last 1 hour" of data, but you update the report every "5 minutes"

Result: ***A single event can belong to multiple windows***

Use Case: "Show me the moving average of stock price for the last 10 minutes, updated every 1 minute"

```
Events:      e1    e2    e3    e4
Time:   0s---5s---10s---15s---20s
             |     |
W1:     [0---------10]
W2:          [5---------15]
W3:                [10--------20]
```

```python
# Window Size: 10s, Hop: 5s
windows = []

def process_event(event):
    current_time = event.timestamp
    
    # Add event to ALL active windows it belongs to
    # (In reality, optimized systems don't duplicate data like this)
    for w in windows:
        if w.start <= current_time < w.end:
            w.add(event)
            
    # Create new windows periodically
    if current_time % 5 == 0:
        windows.append(Window(start=current_time, end=current_time + 10))
```

#### Sliding Window

**Concept**: The ***window slides continuously*** (or specifically triggers on every event). It ***looks backwards*** from the *current* event

Difference from Hopping:
- Hopping jumps in fixed steps (e.g., every 5 mins)
- ***Sliding moves smoothly with the data***

Use Case: "Alert me immediately if there are more than 5 errors in the last minute." (You don't wait for the minute to end; you check on every error)

```
Events:       e1.......e2.......e3 (Current)
              |        |        |
Lookback:     [<-- 1 min -------]
              (From e3 backwards)
```
```python
# Alert if > 5 events in last 60 seconds
event_queue = deque() # Double-ended queue

def on_new_event(event):
    now = time.time()
    event_queue.append(now)
    
    # Remove events older than 60s from the left
    while event_queue and (now - event_queue[0] > 60):
        event_queue.popleft()
        
    if len(event_queue) > 5:
        trigger_alert()
```        

#### Session Window

**Concept**: **Dynamic size**. The ***window stays open as long as events are coming***. It ***closes only after a gap of inactivity (timeout)***

Analogy: A user browsing a website. They click, click, click (Session 1). Then they leave for lunch. They come back 1 hour later and click (Start of Session 2)

Use Case: User behavior analysis ("How long did they play the game?")

```
Events:   [e1, e2, e3]                    [e4, e5]
Time:     10:00.......10:05..............11:00....11:02
               |                              |
               v                              v
Window 1: [10:00-10:05]               Window 2: [11:00-11:02]
(Closed because no events             (New session started
 for >30 mins)                         after long gap)
```
```python
last_event_time = 0
current_session = []
TIMEOUT = 30 * 60 # 30 mins

def process_event(event):
    global last_event_time, current_session
    
    if (event.time - last_event_time) > TIMEOUT:
        # Gap detected! Close old session
        save_session(current_session)
        current_session = [] # Start new
        
    current_session.append(event)
    last_event_time = event.time
```    

#### Comparison

```
     Low Cost                                       High Cost
     (Efficient)                                   (Expensive)
         |                                              |
    [ Tumbling ] -------- [ Hopping ] ------------- [ Sliding ]
         |                                              |
    (Misses patterns                            (Catches everything,
     at boundaries)                              eats RAM/CPU)
```

|Window Type|The Good (Pros) |The Bad (Cons)|Real World Example|
|-----------|----------------|--------------|------------------|
|Tumbling   |**Simplest**. Fast and easy to build. No double-counting.|**Blind Spots**. If an event happens right on the cut-off line, you might miss the pattern.|"Hourly Report" (09:00-10:00)|
|Hopping    |**Smoother**. Catches patterns that happen on the edge of windows.|**Wasteful**. Processes the same data twice (or more). Uses more CPU.|"Trending Now" (Last 1 hr, updated every 5m)|
|Sliding    |**Fastest**. Alerts you immediately when a threshold is hit.|**Expensive**. Needs huge memory to track every single event timestamp.|"Fraud Alert" (5 failures in any 10s)|
|Session    |**Human**. Tracks actual behavior (starts/stops) instead of arbitrary time.|**Unpredictable**. Hard to code because you never know when a session will end.|"User Analytics" (How long was a user online?)|


### Observability in Microservices architecture

**What is Observability?**

If Monitoring tells you "The system is dead," Observability tells you "Why it died"

- In a Monolith, you just look at one log file
- In Microservices, a single user request hits 20 different services. If the user says "It's slow," you need to know *which of those 20 services is the bottleneck*. You can't just check 20 different log files manually

#### The Three Pillars

To make a system observable, you need three types of data:
1. **Logs**: "What happened?" (Events)
2. **Metrics**: "Is it healthy?" (Numbers)
3. **Tracing**: "Where did the request go?" (The Path)

Pillar 1: Distributed Logging (The "Event Diary")
- The Problem: You have 50 services. If Service A errors out, you don't want to SSH into Server A to read a text file
- The ***Solution***: All services send their logs to a ***Central Log Aggregator*** (like ELK Stack - Elasticsearch, Logstash, Kibana). You search one dashboard to see logs from all servers
- ***Key***: You need a **Correlation ID**. This is a random ID (e.g., `req-123`) attached to the user's request at the start. Every service adds this ID to its log. This *lets you filter the central dashboard*: `show logs where id="req-123"`
```
[Service A] --(Log: "User login")--> \
                                      \
[Service B] --(Log: "DB Error")------> [ Log Collector ] ---> [ Dashboard ]
                                      /  (Logstash)           (Kibana)
[Service C] --(Log: "Payment OK")--> /
```

Pillar 2: Metrics (The "Health Score")
- The Problem: Logs are text. They are hard to count. You need to know trends like "Are errors increasing?" or "Is CPU high?"
- The **Solution**: ***Services calculate numbers (aggregates) over time*** and ***send them to a Time-Series Database (like Prometheus)***
- Examples: "Requests per second", "CPU usage", "Memory usage"
- Use Case: You set an Alert: "If CPU > 90% for 5 minutes, page the on-call engineer."
```
 [ Service A ]
 (Memory: 40%)  <------- (Pull) ----- [ Prometheus ]
 (CPU: 10%   )                        (Stores history)
                                           |
                                           v
                                     [ Grafana ]
                                     (Pretty Charts)
```

Pillar 3: Distributed Tracing (The "Map")
- The Problem: The user says "Checkout is slow"
	- Service A took 10ms
	- Service B took 10ms
	- Service C took 5000ms (The culprit!)
- Metrics tell you "It is slow"
- Tracing tells you Service C is the reason
- The ***Solution***: ***A Trace visualizes the lifespan of a request as it hops between microservices***. It looks like a waterfall chart
- Tools: Jaeger, Zipkin
```
Request: "Buy Item" (Total: 5120ms)
|
|-- [API Gateway] (10ms)
    |
    |-- [Auth Service] (50ms)
    |
    |-- [Order Service] (50ms)
        |
        |-- [Payment Service] (10ms)
        |
        |-- [Inventory Service] (5000ms) <--- THE BOTTLENECK!
```

### Migrating from a monolith to a microservice architecture

**The Goal: Moving Without Breaking**

Migrating to microservices is like renovating a house while you are still living in it. You cannot just "tear it down" and build a new one because the business still needs to run. You must do it piece by piece

**The "Big Bang" Mistake**: *Never try to rewrite the whole system from scratch at once*. It almost always fails because the old system keeps changing while you write the new one

#### The Strategy: The "Strangler Fig" Pattern

The most famous migration pattern is the "Strangler Fig"
- **Concept**: A strangler fig tree grows around an old tree. Eventually, the old tree dies, and the new tree stands alone
- **In Software**: You ***build new features as microservices and slowly peel old features out of the monolith one by one***

**The Strangler Process**

***Phase 1: The Monolith (Start) All traffic hits the Big App***
```
     [ User ]
         |
         v
    [ Monolith ]
   (Auth, Order, User)
         |
    [ Big DB ]
```

***Phase 2: Add an API Gateway (The Interceptor)*** 
- Put a "Front Door" (Gateway) before the monolith. This lets you control traffic
```
     [ User ]
         |
         v
  [ API Gateway ]
         |
         v
    [ Monolith ]
```

***Phase 3: Strangle One Service (e.g., User Profile)***
- Build a new "User Service." 
- Tell the Gateway to send /users requests to the new service, and everything else to the Monolith
```
     [ User ]
         |
         v
  [ API Gateway ]
     /       \
    / (Old)   \ (/users)
   v           v
[ Monolith ] [ User Service ] (New!)
(Auth, Order)       |
     |           [ User DB ]
  [ Big DB ]
```

***Phase 4: Repeat Until Done***
- Keep moving pieces until the Monolith is empty (or very small)

#### How to Decompose? (Finding Boundaries)

How do you decide what piece to break off first? You look for Seams.

**Approach A**: **By Business Capability (Vertical Slice)**. Group things by what they do for the business
- Shipping
- Billing
- Inventory

**Approach B**: **By Subdomain (DDD - Domain Driven Design)**.Group things by the "language" they speak
- ***Core Domain***: The unique money-maker (e.g., The matching algorithm for Uber)
- ***Supporting Domain***: Important but generic (e.g., The notification system)

**Tip**: 
- Start with ***Edge Services (things with few dependencies)***
	- Good Start: "Notification Service" (Just sends emails, doesn't need much data)
	- Bad Start: "Core Pricing Engine" (Deeply tangled in everything)

#### Handling the Database (The Hardest Part)

Code is easy to move. ***Data is hard***. You cannot rip the data out easily because the Monolith still needs it

*Pattern*: **Dual Write (The Transition)** 
- When you move the "User" table to a new service, the Monolith might still try to read it
	- Step 1: The Monolith writes to ***both*** the Old Table and the New Service
	- Step 2: Move the "Read" traffic to the New Service
	- Step 3: Stop writing to the Old Table

```
    [ Monolith ]
       /    \
      /      \ (Sync Call)
     v        v
[ Old DB ]  [ User Service ] -> [ New User DB ]
```

#### Steps & Tips for Migration

- Stop the Bleeding: Stop adding new code to the Monolith. Any new feature must be a microservice
- Decouple the UI: If your frontend is baked into the backend (e.g., JSP/Thymeleaf), separate it into a Single Page App (React/Angular) first
- Prioritize:
	- High Change Frequency: Move parts that change often (so you can deploy them fast)
	- Resource Hogs: Move parts that use too much RAM/CPU (e.g., Image processing) so they don't crash the main app
	- Accept Data Duplication: It is okay to have user_id and user_email stored in the Order Service's DB and the User Service's DB. Do not try to make one perfect normalized DB across the world

*Summary*
1. Do: Use the Strangler Fig pattern
2. Do: Put an API Gateway in front early
3. Don't: Share databases
4. Don't: Rewrite from scratch

### Microservices principles and best practices

**The DRY Principle (Don't Repeat Yourself) vs. Shared Libraries**

The Standard Rule: In coding, we are taught "Don't Repeat Yourself." If you write the same code twice, put it in a shared library.

**The Microservices Exception**: ***In Microservices, DRY can be dangerous***. If Service A and Service B both use a shared library common-utils.jar, they are now "married"

- If Service A needs to change common-utils to add a feature, it might break Service B
- You end up in ***Dependency Hell***, where you cannot deploy Service A because Service B isn't ready for the library update
- Best Practice: ***Duplication is better than Coupling***. It is okay to copy-paste a small Date Helper function into two different services if it keeps them independent
- *When to use Shared Libraries: Only for **infrastructure concerns** that rarely change (e.g., Logging clients, Auth tokens), never for business logic*

```
// Bad!

[ Service A ]       [ Service B ]
      \               /
       \             /
      [ SHARED-LIB.JAR ]
      (Contains Logic)
             |
    "I changed the Lib!"
    (Both Services Break)
```
```
// Good!

  [ Service A ]       [ Service B ]
(Has DateUtil.java) (Has DateUtil.java)
      |                   |
  (Independent)       (Independent)
```

**Databases: The "Shared Nothing" Architecture**

Principle: **Database-per-Service**. We discussed this before, but the principle here is Data Sovereignty. A service must be the only thing that can touch its data

Why?
- **Schema Evolution**: If the "Shipping Team" wants to rename a column, they shouldn't need to email the entire company to ask for permission
- **Performance**: A bad query in the "Reporting Service" shouldn't crash the "Checkout Service"

Best Practice: If Service A needs data from Service B, it must ***ask via API*** (or listen for Events). It cannot run a SQL query on B's database

```
   [ TEAM A ]                 [ TEAM B ]
       |                          |
       v                          v
 +-------------+            +-------------+
 |  Service A  | --(API)--> |  Service B  |
 +-------------+            +-------------+
       |   (Allowed)              |
       v                          v
 [ DB A (Private) ]         [ DB B (Private) ]
       ^                          ^
       |                          |
       X (FORBIDDEN!) ------------+
```

**Structured Autonomy (Team Organization)**

Principle: **Conway's Law**. ***"Software structure reflects the organization structure"***

If you have a separate "DBA Team," "Frontend Team," and "Backend Team," you will build a Monolith. In Microservices, you ***organize teams vertically (Cross-Functional Teams)***

Structured Autonomy:
- You Build It, You Run It: The team that writes the code is also the team that gets woken up at 3 AM if it crashes. This forces them to write better code
- Freedom of Choice: The team chooses the best tool for the job (e.g., Python vs. Java), provided they maintain it themselves

```
// Horizontally sliced:

 [   UI Team   ] (Handover) -> [ Backend Team ] (Handover) -> [ DB Team ]
      |                            |                            |
      v                            v                            v
(Slow releases, lots of meetings, "It's not my fault" culture)
```
```
// Vertically sliced (Microservices):

     [ PRODUCT TEAM A ]            [ PRODUCT TEAM B ]
 (Product + UI + Dev + Ops)    (Product + UI + Dev + Ops)
           |                              |
           v                              v
   [ SERVICE A (Full Stack) ]    [ SERVICE B (Full Stack) ]
```

**API Management (The "Contract")**

Principle: The Interface is King. Since services talk over the network, ***your API (REST/gRPC) is a strict contract. You cannot break it***

Best Practices:
- **Versioning**: Never change an existing endpoint. Create a new version (`/v1/user -> /v2/user`)
- **API Gateway**: Use a Gateway to handle "Cross-Cutting Concerns" like Authentication, Rate Limiting, and SSL termination. Do not write this logic inside every microservice
- **Consumer-Driven Contracts**: Before you change your API, run tests provided by the people using your API to ensure you don't break them

```
     [ External Clients ]
              |
              v
     +------------------+
     |   API GATEWAY    | <--- Handles Auth, Rate Limits, Routing
     +------------------+
          /       \
         /         \  (Clean Internal Traffic)
        v           v
  [ Service A ]  [ Service B ]
  (Focuses only   (Focuses only
   on logic)       on logic)
```

## Big Data

**What is "Big Data"?**

In System Design, "Big Data" doesn't just mean "a lot of files." It refers to data that **breaks the 3 V's rule**, meaning a normal database (like MySQL on one computer) crashes if it tries to handle it
1. **Volume**: Too big to fit on one hard drive (Petabytes)
2. **Velocity**: Coming in too fast to write to disk (Millions of events/sec)
3. **Variety**: It's messy (JSON, CSV, Videos, Logs all mixed together)

### Processing Strategies: Batch vs. Stream

To handle this data, we have two main approaches

#### Batch Processing (The "Laundry" Method)

You ***wait for data to pile up, then process it all at once***!
- Analogy: You wait for the laundry basket to fill up before running the washing machine
- Tech: Hadoop MapReduce, Spark
- Pros: ***Very accurate***, can process huge history
- Cons: ***Slow***. You get the answer hours later

```
Data -> [ Wait... ] -> [ Wait... ] -> [ PROCESS HUGE CHUNK ] -> Result
       (09:00 AM)     (12:00 PM)          (End of Day)
```

#### Stream Processing (The "Dishwasher" Method)

You ***process every piece of data the moment it arrives***.
- Analogy: You wash a dirty plate immediately after dinner
- Tech: Apache Kafka, Flink, Storm
- Pros: ***Instant results (Real-time)***
- Cons: ***Hard to look back at history***; complex to handle late data

```
Data -> [ Process ] -> Result
Data -> [ Process ] -> Result
Data -> [ Process ] -> Result
(Continuous Flow)
```

### The Lambda Architecture

The Problem:
- Batch is accurate but slow
- Stream is fast but sometimes approximate (or hard to re-calculate history)

**The Solution (Lambda Architecture)**: ***Why not do both?*** The Lambda Architecture is a *design pattern* that splits data into two paths: ***a Hot Path (Speed)*** and ***a Cold Path (Batch)***.

#### The Three Layers

1. Batch Layer (The Cold Path):
	- Stores the "Master Dataset" (immutable history)
	- Pre-computes comprehensive views every few hours
	- Goal: Accuracy & Completeness
2. Speed Layer (The Hot Path):
	- Processes *only the recent data* (e.g., the last hour) *that the Batch Layer hasn't touched yet*
	- Goal: Low Latency
3. ***Serving Layer (The Merge)***: When a user asks a question, ***this layer combines the answer from the Batch Layer (History) and the Speed Layer (Real-time)***

**`Final Answer = Batch View + Real-time View`**

```
                    [ DATA SOURCE ]
                           |
            +--------------+--------------+
            |                             |
            v                             v
    1. [ BATCH LAYER ]            2. [ SPEED LAYER ]
    (Hadoop / S3)                 (Kafka / Storm)
    "Process all history"         "Process last 1 hour"
            |                             |
            v                             v
    [ Batch Views ]               [ Real-time Views ]
    (Database A)                  (Database B)
            |                             |
            +-------------+---------------+
                          |
                          v
                 3. [ SERVING LAYER ]
                 "Query: Batch + Speed"
                          |
                          v
                      [ USER ]
```

Example: ***YouTube View Counter***
- Imagine you are building the "View Count" for a YouTube video
- Speed Layer:
	- User clicks video -> `Counter +1` immediately
	- The user sees "105 views" instantly
	- Risk: Might double-count if the network is glitchy, but it's fast
- Batch Layer:
	- Every night, it takes all the raw logs of the day
	- It filters out bots, spam clicks, and duplicate reloads
	- It calculates the perfectly accurate count
- ***Serving Layer*** (Next Morning):
	- ***The system overwrites the "Fast Estimate" with the "Batch Accurate Count"***
	- The view count might jump from 10,500 (Speed estimate) down to 10,200 (Batch corrected)

**Pros and cons of Lambda**

Pros	
- **Resilient**: If the Real-time layer crashes, the Batch layer fixes the data later
- **Accurate**: You get the speed of streaming + the accuracy of batching

Cons	
- **Complexity**: You have to write code twice (once for Batch, once for Stream). This violates the DRY principle
- **Maintenance**: You have to manage two complex distributed systems

### OLTP vs OLAP

The Analogy: The ***Cashier vs The Manager***

OLTP is the Cashier
- Job: Handle one customer at a time, very fast
- Task: "Sell one coffee." "Update one inventory item."
- Goal: Don't keep the customer waiting

OLAP is the Manager
- Job: Sit in the back office and look at all receipts from the last 10 years
- Task: "Which coffee sold the best in winter 2023?"
- Goal: Find trends to make better decisions

#### Why separate them? The Big Data Strategy

In a Big Data architecture, ***you never run analytics on your OLTP database***

The Golden Rule: **Performance Isolation**. If a Data Scientist runs a massive query ("Sum all orders for 10 years") on your live MySQL database (OLTP), the CPU hits 100%. Suddenly, users cannot log in or buy products. *The site freezes*

The Strategy: ***Move data from the fast transactional system (OLTP) to a dedicated big data system (OLAP) via a pipeline***

```
       [ USERS ]
           |
           v (Small, fast writes)
 +----------------------+
 |      OLTP DB         |  <-- "Source of Truth"
 | (MySQL / Postgres)   |      (Optimized for writing rows)
 +----------------------+
           |
           |  ETL Pipeline (Extract, Transform, Load)
           |  (Runs every night or streams via Kafka)
           v
 +----------------------+
 |   OLAP / Data Lake   |  <-- "Big Data Storage"
 | (Snowflake / HDFS)   |      (Optimized for reading columns)
 +----------------------+
           |
           v (Huge, slow reads)
    [ DATA ANALYST ]
    (Tableau / BI Tools)
```

#### Row vs. Columnar Storage - The Secret Sauce


The physical storage strategy ***changes drastically*** between OLTP and Big Data OLAP

**OLTP uses Row Storage**:
- It stores data line-by-line
- Why? It's fast to "Write a new User". You just append one line

*OLTP (Row Store) on Disk:*
| ID | Name | Age |
| --- | --- | --- |
| 1 | Alice | 25 | | 2 | Bob | 30 |

```
[1, Alice, 25]  [2, Bob, 30] ...
```
*(Good for fetching Alice's FULL profile)*

**OLAP uses Columnar Storage**:
- It stores data column-by-column
- Why? 
	- ***Big Data queries usually ask for specific fields*** (e.g., "Average Age")
	- Columnar storage lets the DB read only the "Age" column and ignore "Name," "Address," "Password," etc. This scans terabytes of data 100x faster

| ID | Name | Age |
| --- | --- | --- |
| 1 | Alice | 25 | | 2 | Bob | 30 |

```
[1, 2]  [Alice, Bob]  [25, 30] ...
```
*(Good for calculating Average Age - just read the last block!)*

***Table comparison***

|Feature|OLTP (Transaction)|OLAP (Big Data Analysis)|
|-------|------------------|------------------------|
|Primary Task|Recording transactions (`INSERT`/`UPDATE`)|Analyzing patterns (`SELECT`/`SUM`)|
|Data Source|Original source (The App)|Copy of data (From ETL)|
|Volume |Gigabytes (Current data).|Petabytes (All history)|
|Speed  |Milliseconds     |Seconds to Minutes     |
|Storage|Row-oriented     |Column-oriented (Parquet, ORC)|
|Strategy|ACID Compliant (Must be safe) |Eventual Consistency is okay |


- Batch is traditionally associated with OLAP (Historical Analytics)
- Stream is used for both (Real-time Analytics and Event-Driven Systems)
- OLTP is almost always Real-Time / Synchronous

#### When to use what?

- "Design a Bank Backend": You need OLTP (PostgreSQL/Oracle). ACID compliance is non-negotiable
- "Design a YouTube Analytics Dashboard": You need OLAP (BigQuery/ClickHouse). You are aggregating billions of views
- "Design the whole system": You use OLTP to catch the user's click, and an ETL pipeline to move that click into an OLAP warehouse for the monthly report

### Map-Reduce for Big Data processing

"MapReduce" ***splits a huge task into tiny tasks***, **runs them in parallel on many computers**, and ***combines the results***

Google Search Indexing: ***MapReduce was invented by Google to process the entire internet***
- **Map**: Read a webpage and list all words (keywords)
- **Shuffle**: Group all pages that contain the word "Python"
- **Reduce**: Create the index list: `"Python" -> [Page A, Page B, Page C]`

The Concept: Counting a Giant Piggy Bank
- Imagine you have a truck filled with coins. You want to know the total value
- One Person: It will take weeks
- *MapReduce*: You dump the coins on a table and ask 3 friends to help

**Step 1: SPLIT (Divide the Work)**
- You pour a small pile of coins in front of each friend
```
   [ TRUCKLOAD OF COINS ]
           |
   ---------------------
   |         |         |
[Pile 1]  [Pile 2]  [Pile 3]
(Friend A) (Friend B) (Friend C)
```

**Step 2: MAP (Identify & Tag)**
- Each friend looks at their pile and shouts out what they see, one by one. They don't do math yet; they just identify the coin
- Friend A sees a Quarter -> Writes `(Quarter, 1)`
- Friend A sees a Penny -> `Writes (Penny, 1)`
```
   Friend A        Friend B        Friend C
   --------        --------        --------
 (Quarter, 1)    (Dime, 1)       (Quarter, 1)
 (Penny, 1)      (Quarter, 1)    (Penny, 1)
 (Quarter, 1)    (Penny, 1)      (Dime, 1)
```

**Step 3: SHUFFLE (Sort & Group)**
- This is the most important part. You (the System) run around and put all the "Quarter" slips in one bucket, all "Penny" slips in another
```
     |             |            |
     |             |            | (Sorting...)
     v             v            v

 [ Quarter Bucket ]   [ Penny Bucket ]   [ Dime Bucket ]
    (Quarter, 1)         (Penny, 1)         (Dime, 1)
    (Quarter, 1)         (Penny, 1)         (Dime, 1)
    (Quarter, 1)         (Penny, 1)
    (Quarter, 1)
```

**Step 4: REDUCE (Count the Piles)**
- Now, you ask a "Counter" (Reducer) to sum up each bucket
```
 [ Quarter Reducer ]  [ Penny Reducer ]  [ Dime Reducer ]
        |                    |                  |
   1+1+1+1 = 4          1+1+1 = 3          1+1 = 2
        |                    |                  |
        v                    v                  v
  Total: $1.00         Total: $0.03       Total: $0.20
```

|Phase |Analogy         |What the Computer Does|
|------|----------------|----------------------|
|Split |Pouring piles of coins.|Dividing a 1TB file into 100MB blocks.|
|Map   |Identifying "This is a Quarter."|Parsing data and outputting (Key, Value).|
|Shuffle|Putting Quarters in the Quarter bucket.|Moving data across the network so keys stay together.|
|Reduce|Counting the stack of Quarters.|Summing up the values for each key.|

**Why is this useful for System Design?**
1. **Scalability**: If the file gets 2x bigger, you just add 2x more computers (Workers). The logic doesn't change
2. **Fault Tolerance**: If "Worker 2" crashes while counting, the Master Node notices and simply gives "Worker 2's" book to "Worker 4". The whole job doesn't fail!

## Reliability, Error Handling, and Recoverability patterns

### Throttling & Rate Limiting (The Bouncer)

What is it? Control how many requests a user or service can make in a specific time
- **Rate Limiting**: ***Capping a specific user*** (e.g., "You can only tweet 100 times an hour")
- **Throttling**: ***Slowing down everyone*** to save the server from crashing under load

Why? To prevent one angry user or a bot from eating up all the server's resources, causing a denial of service for everyone else

```
     [ User A ]  [ User B ]  [ User C ]
          |           |           |
          v           v           v
      +-----------------------------------+
      |      RATE LIMITER / GATEWAY       |
      |   "User A: OK"   "User C: STOP!"  |
      +-----------------------------------+
              |           |
              v           v
      [     Backend Service      ]
```

### Retry Pattern (The "Try Again")

What is it? When a request fails, don't give up immediately. Wait a bit and try again
- Crucial Logic: **Use Exponential Backoff**
	- Don't retry instantly (`0s`). Wait `1s`, then `2s`, then `4s`, then `8s`. *This gives the failing server time to recover*
		- Why? Networks are glitchy. Sometimes a failure is just a 1-millisecond blip

```
 [ Service A ] --(1. Request)--> [ Service B (Busy!) ]
      |                                ^
      | (Fail! Wait 1s...)             |
      |                                |
      +-----(2. Retry)-----------------+
      |                                |
      | (Fail! Wait 2s...)             |
      |                                |
      +-----(3. Retry)-----------------+
                                       |
                                    (Success!)
```

### Circuit Breaker Pattern (The Fuse)

What is it? ***If a service fails repeatedly (e.g., 10 times in a row), stop calling it entirely***. Just ***return an error immediately***

Why different from Retry? 
- Retry is for *glitches*
- Circuit Breaker is for ***outages***
- If Service B is dead, retrying 1,000 times just wastes time and makes things worse

The 3 States:
- **Closed (Normal)**: Traffic flows freely
- **Open (Broken)**: Traffic is blocked. *Immediate error*
- **Half-Open (Test)**: Let 1 request through to *see* if the service is fixed

```
 [ Service A ] ----> [ Circuit Breaker ] -X-> [ Service B (DEAD) ]
                          |
    (Detects 5 failures)  |
                          v
                  [ SWITCH OPENS ]
                          |
[ Service A ] ----> [ Circuit Breaker ]
                          |
                          v
                 "Error: Service Unavailable"
           (Service B is not even touched)
```

### Dead Letter Queue (DLQ) (The Graveyard)

What is it? A ***special queue where we put messages that failed to process even after retries***. Instead of deleting the message (data loss) or blocking the whole queue (clogging), we move the "poison" message aside to look at later

Why? So ***engineers can debug why that specific order failed without stopping the rest of the business***

```
       [ Message Queue ]
       |   |   |   |   |
       v   v   v   v   v
    +---------------------+
    |      Consumer       | <--- (Processing...)
    +---------------------+
              |
        (Order #99 Fails 5x!)
              |
              v
    +---------------------+
    |  DEAD LETTER QUEUE  |  <-- "Graveyard"
    | [ Order #99 ]       |
    +---------------------+
              ^
              |
      (Engineer inspects later)
```

|Pattern|Role            |Analogy|
|-------|----------------|-------|
|Rate Limiting|**Prevention**. Stops the system from getting overloaded.|The Club Bouncer.|
|Retry  |**Recovery**. Handles temporary network glitches.|Reloading a webpage.|
|Circuit Breaker|**Protection**. Stops the system from wasting time on dead services.|Electrical Fuse.|
|DLQ    |**Analysis**. Saves failed data so it isn't lost.|The "Undeliverable Mail" bin.|

### Transactional Outbox Pattern

The ***gold standard for reliability*** in *microservices*

**The Problem: The "Dual Write" Danger**

In an Event-Driven system, when a user creates an order, you need to do two things:
1. Save the order to your Database
2. Publish an "OrderCreated" event to the Message Broker (Kafka/RabbitMQ) so the Shipping Service knows about it

The Trap: If you do these as two separate steps, one might fail

Scenario A: ***Database succeeds, Broker fails***
- You have the order in your DB
- You never sent the event
- Result: The user paid, but the item never ships

Scenario B: ***Broker succeeds, Database fails***
- You sent the event "Order Created!"
- The DB transaction rolls back (maybe a constraint error)
- Result: The Shipping Service ships a free item that doesn't exist in your records

```
      [ User ]
         |
         v
    [ Order Service ]
         |
    +----+----------------+
    |                     |
    v (1. Write)          v (2. Publish)
 [ Database ]        [ Kafka ]
 (Success)           (CRASH!)
    ^                     ^
    |_____________________|
      NOT ATOMIC! (Inconsistent State)
```

**The Solution: Transactional Outbox**

The Trick: Instead of sending the message to Kafka directly, you ***save the message into your own database first***

***Since you are writing to the same database***, you can ***wrap both*** the "`Order Insert`" and the "`Event Insert`" in a **Single ACID Transaction**. It is impossible for one to succeed and the other to fail

***Step 1: The Local Transaction***: When the user orders, you insert two rows into the DB in one go

Table: Orders
| ID | User | Item | 
| :--- | :--- | :--- | 
| 101 | Bob | Laptop |

Table: Outbox
| ID | Event_Type | Payload | Status | 
| :--- | :--- | :--- | :--- | 
| 99 | OrderCreated | {"id":101...} | **PENDING** |

Safe write:
```
     [ User ]
         |
         v
    [ Order Service: ---(Start Transaction)--- ]
         |                                     |
         v                                     v
   INSERT INTO Orders...               INSERT INTO Outbox...
         |                                     |
         +------------------+------------------+
                            |
                   (COMMIT TRANSACTION)
                            |
                    [ SQL Database ]
            (Both exist, or neither exists.)
```

***Step 2: The Relay (The Sweeper)***: Now the data is safe in your DB. You ***need a separate background process (The Relay) to pick it up and send it to Kafka***
- Read: The Relay polls the Outbox table for "PENDING" events
- Publish: It pushes them to Kafka
- Delete/Mark: Once Kafka says "Received" (ACK), the Relay deletes the row from the Outbox

```
    [ SQL Database ]                  [ Message Broker ]
    |  (Outbox Table)                       (Kafka)
    |  [Event 99]                              ^
    +-------+                                  |
            | (1. Poll)                        |
            v                                  |
    [ Message Relay ] -------------------------+
    (Background Worker)      (2. Publish)
            |
            | (3. Mark as Sent)
            v
    [ SQL Database ]
    (Update Outbox set Status='SENT')
```

**Why is this reliable?**

- **Atomicity**: The event is guaranteed to be saved if the order is saved
- **Retry**: If the Relay crashes or Kafka is down, the event sits safely in the Outbox table. The Relay will just pick it up again when it restarts
- **At-Least-Once Delivery**: The Relay might send the message, crash, and restart before marking it as "SENT." It will send it again. This creates a duplicate.
	- *Requirement*: Your consumers must be **Idempotent** (handle duplicates gracefully)

## Deployment patterns 

Here are explanations for each pattern:

**Rolling Deployment**

You update your instances incrementally rather than all at once. You deploy the new version to a small batch of servers (e.g., `20%`), wait for them to stabilize, and then move to the next batch.
- Goal: **Zero downtime**. If errors occur, only a small percentage of capacity is affected
- Trade-off: ***The old and new versions must coexist briefly, requiring backward compatibility***

**Blue-Green Deployment**

You maintain two identical environments: **Blue (Live)** and **Green (Idle)**. You ***deploy the new version to the Green environment and test it fully***. Once ready, you switch the Load Balancer to route all traffic to Green instantly
- Goal: **Instant rollout and instant rollback** (just switch the router back to Blue)
- Trade-off: Requires **double the infrastructure resources (cost)**

**Canary Release vs A/B Testing**

These look similar (splitting traffic) but have different goals
- **Canary Release (Safety)**: You ***route a tiny percentage of traffic*** (e.g., `1%`) to the new version to catch crashes or bugs early. If the "canary" survives, you roll out to the rest. It is about ***risk mitigation***
- **A/B Testing (Business)**: You route 50% of users to Version A and 50% to Version B to see which one performs better (e.g., higher conversion rate). It is about ***user behavior***

**Chaos Engineering**

This is the ***practice of intentionally breaking parts of your system in production*** (e.g., killing a server, severing a database connection) to ***ensure the system can recover automatically without human intervention***
- Goal: Build **confidence** in the system's **resilience** (e.g., Netflix's Chaos Monkey)

## Other patterns

**Materialized View Pattern**

Instead of running a **complex query** (joins, aggregations) every time a user requests data, the system **pre-computes** the result and **stores it in a physical table** (the "Materialized View")
- The application reads directly from this pre-computed table
- Standard View: A virtual table that runs the query on-the-fly (high CPU)
- ***Materialized View***: *A physical table containing the cached result of the query (high Storage)*

```
 [ User Request ]
      |
      v (Read)
[ Materialized View ]  <---- (Fast, O(1) Lookup)
|  ID  |  Total    |
| 101  |  $500.00  |
+------+-----------+
      ^
      | (Async Update / Batch Job)
      |
[ Primary Database ]
(Complex Joins & Sums happen here)
```
- Pros: ***Extremely fast read performance***; offloads complex processing from the transactional database
- Cons: ***Data is eventually consistent (stale)*** until the view is refreshed

**Sidecar & Ambassador Patterns**

These patterns involve ***deploying a secondary container/process alongside the main application container within the same host or pod***

A. **Sidecar Pattern**
- Concept: ***Extends the functionality of the main application without modifying its code***. The sidecar ***shares the same lifecycle and resources*** (disk/network) as the main app.\
- Use Cases: Log shipping, configuration synchronization, monitoring agent
```
Host / Pod
+----------------------------+
|  [ Main Service ]          |
|  (Business Logic)          |
|       | writes logs        |
|       v                    |
|  [ Sidecar Agent ]         |
|  (Reads logs -> Sends to   |
|   Central Log Server)      |
+----------------------------+
```
B. **Ambassador Pattern**
- Concept: A ***specific type of sidecar that acts as a proxy for outbound network connections***. The main application connects to "localhost," and the Ambassador handles the actual network routing, security, and resiliency
- Use Cases: Adding mTLS encryption, circuit breaking, or service discovery lookups outside the application code
```
Host / Pod
+----------------------------+
|  [ Main Service ]          |
|       | (HTTP localhost)   |
|       v                    |
|  [ Ambassador Proxy ]      |
|  (Adds Retry logic, SSL)   |
+-------+--------------------+
        | (Secure HTTPS)
        v
 [ External Service ]
```

*Pros and cons of Sidecar and Ambassador patterns*

*Sidecar Pattern*: 
- Why it’s needed: To ***decouple "infrastructure" logic from "business" logic***. You shouldn't have to write SSL or Logging code in Java, Python, and Go separately
- Problem Solved: ***Cross-cutting concerns***. It handles logging, metrics (Prometheus), SSL termination, and configuration reloading without touching the main application code
- Trade-off: ***Resource Overhead***. Every pod now needs extra CPU/Memory for the sidecar, even if the app is idle

*Ambassador Pattern*
- Why it’s needed: The ***application shouldn't care about the complexity of the external world*** (e.g., "Which database shard do I call?")
- Problem Solved: ***Connectivity Complexity***. The app simply talks to localhost, and the Ambassador handles smart routing, retries, circuit breaking, or translating protocols (e.g., HTTP to gRPC)
- Trade-off: ***Latency***. Every request makes an extra local network hop before leaving the server

**Backends for Frontends (BFF) Pattern**

Instead of having a single, general-purpose API Backend that serves all clients, you ***create a dedicated backend service for each specific frontend interface*** (e.g., Mobile, Web, 3rd Party)

Each BFF is tailored to the specific needs of its client (e.g., formatting data, stripping unused fields to save bandwidth)

```
 [ Mobile App ]                 [ Desktop Web ]
      |                              |
      v                              v
[ Mobile BFF ]                 [ Web BFF ]
(Optimized: JSON,              (Optimized: HTML/Full Data,
 small payload)                 aggregates multiple calls)
      |                              |
      +-------------+----------------+
                    |
                    v
          [ Downstream Microservices ]
          (User, Cart, Inventory)
```

- Pros: ***Optimized performance for specific clients*** (no over-fetching); ***allows frontend teams to iterate independently***
- Cons: ***Code duplication across different BFFs***; ***increases the number of services to manage***

*Hybrid architecture of API Gateway + BFF*

- API Gateway (The "Bouncer"): Handles cross-cutting infrastructure concerns (SSL termination, DDoS protection, Rate Limiting, Global Authentication). It sits at the very edge
- BFF (The "Concierge"): Handles logic (Aggregating data, formatting JSON, removing fields). ***It sits behind the Gateway***

Why combine them? If you only use BFFs without a Gateway in front, you have to implement SSL certificates, Rate Limiting logic, and Firewall rules inside every single BFF

```
        [ Mobile App ]        [ Web App ]
             |                   |
             v                   v
    +-----------------------------------------+
    |           GLOBAL API GATEWAY            |
    | (SSL, DDoS, Rate Limiting, Auth Check)  | <--- "The Shield"
    +-----------------------------------------+
             |                   |
             v                   v
    +-----------------+  +-----------------+
    |   MOBILE BFF    |  |    WEB BFF      | <--- "The Shapers"
    | (Aggregates)    |  | (Aggregates)    |
    +-----------------+  +-----------------+
             \             /
              \           /
               v         v
         [ Microservices Cluster ]
```

*BFF vs GraphQL*

- BFF Approach (Fixed Menus): The Mobile App calls a specific "Mobile API." The server has hard-coded logic to send back only the name
- GraphQL Approach (Buffet): Both apps call the ***same*** server. The Mobile App just asks for less

BFF example
```
        [ Mobile App ]                         [ Web Browser ]
            |                                      |
            | GET /mobile/user/1                   | GET /web/user/1
            v                                      v
    +-------------------+                  +-------------------+
    |    MOBILE BFF     |                  |     WEB BFF       |
    | (Code: return     |                  | (Code: return     |
    |  { name } only)   |                  |  { name, bio,     |
    +-------------------+                  |    friends[] })   |
            |                              +-------------------+
            v                                      |
      { "name": "Ali" }                            v
                                         { "name": "Ali",
                                           "bio": "Dev",
                                           "friends": [...] }
```

GraphQL example:
```
       [ Mobile App ]                         [ Web Browser ]
            |                                      |
            | POST /graphql                        | POST /graphql
            | Query: { name }                      | Query: { name, bio, friends }
            v                                      v
    +----------------------------------------------------------+
    |                     GRAPHQL SERVER                       |
    |          (One endpoint to rule them all)                 |
    +----------------------------------------------------------+
            |                                      |
            v                                      v
      { "name": "Ali" }                  { "name": "Ali",
                                           "bio": "Dev",
                                           "friends": [...] }
```

- *Use BFF if*: You want to keep your backend logic simple (REST) but need to optimize performance for specific devices
- *Use GraphQL if*: You have many different clients with constantly changing data needs, and you want to avoid writing new API endpoints for every little UI change
