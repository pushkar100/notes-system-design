# Software Architecture of Large Scale Systems

[Course link](https://www.udemy.com/course/software-architecture-design-of-modern-large-scale-systems)

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

