# Networking Concepts and Large Scale Networks Basics

## Table of Contents

- [Networking Concepts and Large Scale Networks Basics](#networking-concepts-and-large-scale-networks-basics)
  - [Networking layers](#networking-layers)
    - [OSI Model](#osi-model)
    - [TCP/IP model](#tcpip-model)
  - [Transport layer protocols and issues](#transport-layer-protocols-and-issues)
    - [TCP — Transmission Control Protocol](#tcp-transmission-control-protocol)
      - [TCP 3-way handshake](#tcp-3-way-handshake)
      - [TCP 4-way handshake](#tcp-4-way-handshake)
    - [Head-of-Line HOL Blocking](#head-of-line-hol-blocking)
    - [User Datagram Protocol](#user-datagram-protocol)
    - [Head-of-Line blocking in HTTP 2 vs HTTP 3](#head-of-line-blocking-in-http-2-vs-http-3)
  - [Transport Layer Security](#transport-layer-security)
    - [Understanding  TLS](#understanding-tls)
      - [TLS certificate or cert](#tls-certificate-or-cert)
      - [TLS 1.2 vs TLS 1.3](#tls-12-vs-tls-13)
      - [Drawbacks of TLS](#drawbacks-of-tls)
      - [TLS optimizations](#tls-optimizations)
      - [TLS Pros & Cons](#tls-pros-cons)
      - [TLS termination](#tls-termination)
  - [Application layer and protocols](#application-layer-and-protocols)
    - [Hypertext Transfer Protocol](#hypertext-transfer-protocol)
      - [HTTP 1.1](#http-11)
      - [HTTP 2](#http-2)
      - [HOL differences between HTTP 1 and HTTP 2](#hol-differences-between-http-1-and-http-2)
      - [HTTP 3 and QUIC](#http-3-and-quic)
    - [Domain sharding](#domain-sharding)
    - [HTTPS](#https)
    - [HTTP optimisations](#http-optimisations)
    - [HTTP header basics](#http-header-basics)
      - [Common ***request*** headers](#common-request-headers)
      - [Common ***response*** headers](#common-response-headers)
      - [Headers for ***caching***](#headers-for-caching)
      - [Headers for ***security***](#headers-for-security)
  - [Server side caching vs  CDN caching](#server-side-caching-vs-cdn-caching)
    - [Server side caching](#server-side-caching)
    - [CDNs](#cdns)
      - [Cache invalidation strategies](#cache-invalidation-strategies)
      - [Asset compression in the cache](#asset-compression-in-the-cache)
  - [Polling](#polling)
    - [Short Polling (The "Nagging" Approach)](#short-polling-the-nagging-approach)
    - [Long polling](#long-polling)
    - [Comparison of polling with other mechanisms](#comparison-of-polling-with-other-mechanisms)
  - [Server push](#server-push)
  - [Server-Sent Events](#server-sent-events)
  - [WebSockets](#websockets)
    - [Websocket handshake process](#websocket-handshake-process)
    - [Websockets and load balancers](#websockets-and-load-balancers)
    - [Challenges scaling websockets in a distributed system](#challenges-scaling-websockets-in-a-distributed-system)
    - [Server push vs SSE vs Web sockets](#server-push-vs-sse-vs-web-sockets)
  - [WebRTC](#webrtc)
    - [How WebRTC works](#how-webrtc-works)
  - [MAC address](#mac-address)
  - [IP address](#ip-address)
    - [IPv4 vs IPv6](#ipv4-vs-ipv6)
    - [Public vs private IPs](#public-vs-private-ips)
  - [NAT](#nat)
  - [Subnetting](#subnetting)
  - [CIDR](#cidr)
  - [DNS](#dns)
    - [Types of DNS and DNS components](#types-of-dns-and-dns-components)
    - [DNS request flow and hierarchy](#dns-request-flow-and-hierarchy)
    - [Types of DNS requests](#types-of-dns-requests)
    - [DNS caching](#dns-caching)
    - [DNS in large scale systems](#dns-in-large-scale-systems)
  - [Load balancers](#load-balancers)
    - [Choosing a load balancing strategy](#choosing-a-load-balancing-strategy)
    - [L4 vs L7 load balancers](#l4-vs-l7-load-balancers)
    - [Consistent hashing](#consistent-hashing)
    - [Consistent hashing in load balancers](#consistent-hashing-in-load-balancers)
  - [API gateways](#api-gateways)
    - [API gateways vs Load balancers](#api-gateways-vs-load-balancers)
  - [Proxy and reverse proxy](#proxy-and-reverse-proxy)
    - [Forward proxy or proxy](#forward-proxy-or-proxy)
    - [Reverse proxy](#reverse-proxy)
    - [Reverse proxy vs Load balancer](#reverse-proxy-vs-load-balancer)
    - [Reverse proxy vs API gateway](#reverse-proxy-vs-api-gateway)
  - [Cryptography](#cryptography)
    - [Types of cryptography](#types-of-cryptography)
    - [RSA](#rsa)
  - [Service mesh](#service-mesh)
    - [Service mesh vs Load Balancer vs API Gateway](#service-mesh-vs-load-balancer-vs-api-gateway)
  - [Client Server model](#client-server-model)
    - [3 components of the client server model](#3-components-of-the-client-server-model)
    - [Steps of a client server interaction](#steps-of-a-client-server-interaction)
    - [3 types of communication in the client server model](#3-types-of-communication-in-the-client-server-model)
    - [Request response cycle](#request-response-cycle)
    - [Stateless vs stateful servers](#stateless-vs-stateful-servers)
    - [Caching helps performance of client server models](#caching-helps-performance-of-client-server-models)
    - [How LBs work in a client-server model](#how-lbs-work-in-a-client-server-model)
    - [Scaling a client server model for high traffic scale](#scaling-a-client-server-model-for-high-traffic-scale)
    - [Security challenges in client server model](#security-challenges-in-client-server-model)
  - [Beyond REST APIs](#beyond-rest-apis)
    - [Benefits and drawbacks of classic REST APIs](#benefits-and-drawbacks-of-classic-rest-apis)
    - [GraphQL](#graphql)
      - [Benefits](#benefits)
      - [How it works](#how-it-works)
      - [Core Components](#core-components)
      - [Data Flow](#data-flow)
      - [Responsibilities](#responsibilities)
      - [Popular tools](#popular-tools)
      - [Drawbacks of GraphQL](#drawbacks-of-graphql)
      - [GraphQL use cases](#graphql-use-cases)
      - [Common System Design Interview Questions](#common-system-design-interview-questions)
    - [gRPC](#grpc)
      - [Benefits](#benefits)
      - [Drawbacks](#drawbacks)
      - [Use cases of gRPC](#use-cases-of-grpc)
      - [gRPC working in detail](#grpc-working-in-detail)
      - [Diagrams for easy understanding](#diagrams-for-easy-understanding)
      - [Protocol buffers explained](#protocol-buffers-explained)

## Networking layers

### OSI Model 

**OSI model is 7 layers**

1. Application – Where user-facing network applications live (HTTP, FTP, SMTP, DNS).
2. Presentation – Translates, encrypts, and compresses data so both sides understand it (TLS, encoding).
3. Session – Manages sessions: setup, keep-alive, checkpoints, and teardown.
4. Transport – End-to-end delivery with reliability, ordering, and flow control (TCP/UDP).
5. Network – Routes packets across networks using logical addressing (IP, routing).
6. Data Link – Reliable delivery between two directly connected nodes using MAC addresses (Ethernet, ARP).
7. Physical – Sends raw bits over the wire: voltages, cables, radio signals.

```
7. Application   → HTTP, HTTPS, FTP
6. Presentation  → Encryption, Compression
5. Session       → Session mgmt, reconnect
4. Transport     → TCP, UDP
3. Network       → IP, routing
2. Data Link     → Ethernet, MAC
1. Physical      → Wires, signals
```

*Mnemonic*: **“All People Seem To Need Data Processing”**

```
Ultra short mapping (high level)
L7: Apps talk
L6: Data format & encryption
L5: Session management
L4: Reliable delivery
L3: Routing
L2: Local delivery (MAC)
L1: Bits on wire
```

### TCP/IP model

TCP/IP is an alternative model to the OSI model and is only **4 layers**

* OSI: Conceptual
* TCP/IP: Practical / Real-world implemented

> “OSI is a conceptual model, TCP/IP is what the internet actually uses.”

One-line mapping to OSI:
1. `Application (TCP/IP) = OSI Application + Presentation + Session`
2. `Transport = OSI Transport`
3. `Internet = OSI Network`
4. `Network Access = OSI Data Link + Physical`

```
Application  → HTTP, HTTPS, DNS
Transport    → TCP, UDP
Internet     → IP
Link         → Ethernet, WiFi
```

## Transport layer protocols and issues

Transport layer protocols provide end-to-end communication between processes, handling reliability, ordering, flow control, and congestion (e.g., TCP) or fast best-effort delivery (e.g., UDP).

> They move data between processes on two hosts, deciding ***how reliable*** and ***how fast*** the delivery should be.

<!-- TOC --><a name="tcp-transmission-control-protocol"></a>
### TCP — Transmission Control Protocol

Core idea: **Reliable, ordered, congestion-aware communication**

**Use TCP when**:
1. You cannot lose data
2. Order matters

**Example use cases** (TCP at the transport layer will support the below application later protocols):
* HTTP/1, HTTP/2 traffic
* HTTPS traffic
* Database connections
* File transfers (FTP?)

**TCP guarantees**:
* ✅ Reliable
* ✅ Ordered
* ✅ Congestion controlled: TCP congestion control dynamically adjusts the sending rate based on network feedback (loss, delay, ACKs) to avoid overwhelming the network and causing collapse

**Drawbacks**:
* ❌ Slower than UDP (*3 reasons*: Higher latency, HOL blocking, Connection setup cost)

**TCP/IP (Quick clarity)**
* IP → Addressing + routing
* TCP → Reliability + order

IP doesn’t care if packets arrive. TCP does.

**Questions on TCP**:

> Q: Why is TCP slower than UDP?
A: TCP adds reliability, ordering, congestion control, and handshake overhead.

> Q: Why does HTTP/2 still use TCP?
A: To reuse existing infrastructure and reliability, while fixing HOL at application level.

**Enterprise Tools using TCP**
1. Nginx
2. Envoy
3. HAProxy
4. Databases (Postgres, MySQL)
5. REST APIs

#### TCP 3-way handshake

TCP 3 way handshake is used for **Opening a connection**
1. Client → Server : SYN
2. Server → Client : SYN + ACK
3. Client → Server : ACK

```
Client                Server
  | ---- SYN -------> |
  | <--- SYN+ACK ---- |
  | ---- ACK -------> |
  | === Connected === |
```

Explanation:
> SYN – “I want to start a connection, and here’s my initial sequence number.”
Why: Lets the server know a new connection is requested and establishes sequence numbering.

> SYN-ACK – “I got your request, here’s my sequence number, and I acknowledge yours.”
Why: Confirms the server is reachable and both sides agree on initial sequence numbers.

> ACK – “I acknowledge your sequence number; connection is ready.”
Why: Final confirmation so both sides know the connection is fully established and reliable.

The sequence numbers are unique to each client:
* Client chooses its own Initial Sequence Number (ISN)
* Server chooses its own Initial Sequence Number (ISN)
* They exchange and acknowledge each other’s ISNs during the handshake

Ex:
```
Client → Server: SYN, seq = x
Server → Client: SYN-ACK, seq = y, ack = x + 1
Client → Server: ACK, ack = y + 1
```

**Why they must be different?**
1. TCP is full-duplex (data flows independently in both directions)
2. Each direction needs its own sequence space for ordering, retransmissions, and loss detection
3. Prevents confusion between old and new connections (especially after crashes)

> A 2-way handshake can’t confirm bidirectional readiness or protect against delayed/duplicate packets, leading to half-open or bogus connections. The third ACK is what proves “I heard you hear me."

#### TCP 4-way handshake

TCP 4-way handshake is used for **Terminating a connection**

TCP 4-way handshake (connection termination):
> FIN – “I’m done sending data.”
Sender closes its write side.

> ACK – “I acknowledge your FIN.”
Receiver confirms and may still send data.

> FIN – “I’m also done sending.”
Receiver now closes its write side.

> ACK – “I acknowledge your FIN; connection fully closed.”

Additionally, `TIME_WAIT` exists to absorb delayed packets and prevent old connections from corrupting new ones

```
Client                                Server
  |                                      |
  | -------- FIN (seq = x) ----------->  |  1. Client done sending
  |                                      |
  | <------- ACK (ack = x+1) ----------  |  2. Server acknowledges
  |                                      |
  | <------- FIN (seq = y) ------------  |  3. Server done sending
  |                                      |
  | -------- ACK (ack = y+1) ----------> |  4. Client acknowledges
  |                                      |
  |          TIME_WAIT                   |
  |------------------------------------- |
```

**Why termination needs 4 steps?**
* TCP is full-duplex — each direction closes independently
* One side may finish sending earlier than the other
* Clean shutdown without data loss

> TCP uses a 4-way handshake for teardown because each side must independently close its send channel.

### Head-of-Line HOL Blocking 

**HOL occurs in TCP connections**

*Problem*: TCP delivers data in **order**.

If packet #2 is lost:
```
Packet 1 → OK
Packet 2 → LOST ❌
Packet 3 → WAITING ❌ (even if received)
```

So *everything waits.*

This:
1. Hurts the HTTP/1.1 protocol (One TCP connection per request -- entire connection is blocked)
2. Leads to lossy networks

**The fix for H.O.L blocking**: ***You can’t fix Head-of-Line (HoL) blocking inside TCP — it’s a fundamental property of in-order, reliable delivery.*** 

**Alternative fixes**:
* Use multiple TCP connections: Parallel connections reduce the blast radius of one lost packet (HTTP/1.1 browsers used this trick)
* Application-level multiplexing: Move stream management above TCP so one stalled stream doesn’t block others (HTTP/2 tried this, but still suffers from TCP-level HoL)
* Use QUIC (UDP-based) ✅ real solution: Runs over UDP, has multiple independent streams, packet loss in one stream doesn’t block others (HTTP/3)
* Reduce packet loss (mitigation, not cure)

### User Datagram Protocol

Core idea: **Fast, connectionless, best-effort delivery**

No:
- Handshake
- Retries
- Ordering

Just fire-and-forget.

**UDP characteristics**
- ❌ No guarantee of delivery
- ❌ No order
- ❌ No congestion control
- ✅ Very low latency

**Where UDP is used**
| Use Case	| Why UDP |
| -- | -- |
| Video streaming |	Losing a frame is okay |
| Online gaming	| Latency > reliability |
| VoIP	| Real-time |
| DNS	| Small, fast |
| QUIC / HTTP/3	| Custom reliability |

**UDP flow**
```
Client ---> Packet ---> Server
Client ---> Packet ---> Server
(no ACKs)
```

**Examples of UDP usage**
1. QUIC (used in HTTP/3)
2. WebRTC (Cloudflare, Google)
3. DNS servers
4. Media servers

> “UDP pushes complexity to the application layer.”

**UDP pros**
* Very fast
* No handshake
* Great for real-time

**UDP cons**
* Unreliable
* Harder to implement correctly
* Firewalls sometimes block UDP

**Questions**

> Q: Why not use UDP everywhere?
A: Most applications need reliability and ordering, which UDP doesn’t provide.

> Q: Why is QUIC built on UDP?
A: UDP allows "custom reliability" without TCP’s HOL blocking.

**TCP vs UDP comparison**
| Feature     | TCP      | UDP           |
| ----------- | -------- | ------------- |
| Reliability | Yes      | No            |
| Order       | Yes      | No            |
| Latency     | Higher   | Lower         |
| Handshake   | Yes      | No            |
| Use cases   | APIs, DB | Video, gaming |


### Head-of-Line blocking in HTTP 2 vs HTTP 3

- **HTTP/2**
	- Multiplexes many streams over a single TCP connection
	- Fixes application-layer HoL (one slow request doesn’t block others at HTTP level)
	- ❌ Still suffers from TCP HoL
	- TCP enforces in-order delivery

> One lost packet blocks all streams behind it

Result: Loss in one stream pauses every stream

- **HTTP/3**
	- Runs on QUIC (over UDP)
	- Each stream has independent ordering and retransmission
	- ❌ No TCP → no transport-layer HoL

Result: Loss in one stream blocks only that stream

**"Streams" explained**
> A stream is a logical, independent, bidirectional sequence of bytes representing one request–response pair, multiplexed over a single connection.

Key points:
- Multiple streams share one connection
- Each stream has its own ID and ordering
- Streams are interleaved, not sent end-to-end

Think of streams like ***multiple conversations happening over one phone call***.

**H.O.L blocking in TCP connections**
```
Single TCP Connection
┌───────────────────────────────────────────┐
│ Stream 1 │ Stream 2 │ Stream 3 │ Stream 4 │
└───────────────────────────────────────────┘
          ↓ multiplexed into TCP packets
```
```
TCP Packet Order →
[P1][P2][P3][P4][P5]

P2 = lost ❌

TCP behavior:
- P3, P4, P5 arrive
- TCP MUST wait for P2
- Delivery to app is blocked
```
```
Stream 1: ❌ blocked
Stream 2: ❌ blocked
Stream 3: ❌ blocked
Stream 4: ❌ blocked
```

**No H.O.L blocking in UDP connection**
```
QUIC Connection (UDP)
┌───────────────────────────────────────────┐
│ Stream 1 │ Stream 2 │ Stream 3 │ Stream 4 │
└───────────────────────────────────────────┘
        ↓ independently packetized
```
```
Packets →
[S1-P1][S2-P1][S3-P1][S1-P2❌][S2-P2][S3-P2]
```
```
Stream 1: ⏳ blocked (waiting for retransmit)
Stream 2: ✅ continues
Stream 3: ✅ continues
```

## Transport Layer Security

TLS stands for Transport Layer Security

### Understanding  TLS

**Problem without TLS:**
```
Client ---- HTTP ----> Server
          (plain text)
```

Anyone *in between (middleman)* can:
- Read data
- Modify data
- Impersonate server

**How solves TLS solves issues**

| Property       | Meaning                            |
| -------------- | ---------------------------------- |
| Encryption     | No one can read data               |
| Integrity      | Data not modified                  |
| Authentication | You’re talking to the right server |


**TLS vs SSL**:

* SSL = Old, deprecated
* TLS = Modern replacement

Current versions:
- TLS 1.2 (still common)
- TLS 1.3 (best, fastest)

**Where does TLS sit in the network layer stack?**

TLS runs above TCP to secure application data, effectively sitting *between Transport and Application layers*.
```
Application → HTTP
Security    → TLS
Transport   → TCP (usually)
Network     → IP
```
`HTTPS = HTTP over TLS over TCP`

It sits more closely to the Presentation layer in OSI terms, not the Session layer. Why? ✅ TLS secures the data, it doesn’t manage session state like OSI L5.

**How TLS works**:

TLS Handshake (Simple Version)

**Goal**: Both client and server agree on a *shared secret* to encrypt communication and confirm the server’s identity.

Steps:
> 1. ClientHello – Client says: “Hi! I want to talk securely.” Offers supported TLS versions and encryption methods.

> 2. ServerHello + Certificate – Server replies: “Hi! Let’s use this version and cipher.” Sends its certificate to prove identity.

> 3. Key Exchange – Client sends info to establish a shared session key (can be via RSA, Diffie-Hellman, etc.)

> Rest of the steps: ChangeCipherSpec – Both sides switch to encrypted communication. Finished – Both confirm handshake worked.

From now on, all application data is encrypted.

**TLS handshake steps**
1. Verify server identity 
2. Agree on encryption keys 
3. Start secure communication

```
Client → Server  : Hello + supported crypto
Server → Client  : Certificate + public key
Client → Server  : Pre-master secret (encrypted)
Both sides       : Derive session keys
Secure data flow starts 🔒
```
```
Client                         Server
  | --- ClientHello ----------> |
  | <--- ServerHello + Cert --- |
  | --- Key Exchange ---------> |
  | === Encrypted Data ======= |
```

**Important detail (but simple)**
* Asymmetric crypto → used only during handshake
* Symmetric crypto → used for actual data (faster)

**Assymetric crypto meaning**

Asymmetric cryptography uses a public/private key pair to enable secure encryption and digital signatures without sharing a secret key.
- A public key (anyone can see) and a private key (kept secret) is kept — so that:
	- Encrypt with public key → only private key can decrypt
	- Sign with private key → anyone can verify with public key

Simple analogy: `public key = mailbox slot`, `private key = only you can open it.`

**Use of public and private keys**

Example in TLS:
- Server (owner) generates a private key and keeps it secret.
- Server provides the public key to clients via its certificate.
- Clients use the public key to encrypt data; only the server’s private key can decrypt it.

> The owner (server) generates and keeps the private key secret, and shares the public key with anyone who wants to communicate securely.

#### TLS certificate or cert

> A TLS certificate ***proves*** the server’s identity and ***provides the public key*** needed to establish a secure encrypted session.

What a TLS Certificate Is: A TLS certificate is a digital identity card for a server (or sometimes a client) that proves it is who it claims to be.

**Key points**
1. Authenticates the server – Ensures you are really talking to the legitimate website/server.
2. Contains a public key – Used by the client to set up encryption for the session.
3. Signed by a trusted Certificate Authority (CA) – The CA vouches that the server’s identity is valid.
4. Optional info – Domain name, organization, expiration date, etc.

**Analogy**
- Think of it like a passport for websites:
- Shows your identity (domain)
- Verified by a trusted authority (CA)
- Lets others communicate securely with you

If invalid:🚨 “Your connection is not private”

**Trusted authority**

A trusted authority (CA) is an entity that *verifies and vouches for a server’s identity*, allowing clients to trust its TLS certificate.

Here are some well-known **Certificate Authorities (CAs)**
- GlobalSign
- DigiCert (includes Symantec, Thawte, GeoTrust)
- Let's Encrypt (free, automated)
- Comodo / Sectigo
- Entrust

These are entities browsers and operating systems trust to verify website identities

#### TLS 1.2 vs TLS 1.3
| Feature       | TLS 1.2 | TLS 1.3 |
| ------------- | ------- | ------- |
| Handshake RTT | 2       | 1       |
| Security      | Good    | Better  |
| Legacy crypto | Yes     | Removed |
| Performance   | Slower  | Faster  |

> “TLS 1.3 reduces handshake latency by one round trip.”

#### Drawbacks of TLS

- TLS adds:
	- Handshake latency
	- CPU cost (encryption)
- Bad for:
	- Mobile
	- High-QPS APIs

#### TLS optimizations

1. 1️⃣ TLS Session Resumption
	- Reuse previous session keys.
	- Client reconnects → skips full handshake (faster)
	- Methods: **Session IDs** (server remembers) or **Session tickets** (client stores)
2. 2️⃣ TLS 1.3 "0-RTT"
	- Client sends data with first packet.
	- ⚠ Risk: replay attacks
	- Use only for idempotent requests.
3. 3️⃣ Hardware Acceleration
	- TLS termination at load balancer
	- Dedicated crypto hardware (Faster encryption/decryption, key generation)

> “0-RTT trades some security for latency.”

**Problem with the "O-RTT" optimisation**

A replay attack is when an attacker records a valid message and sends it again later to trick the server.

> A replay attack resends a valid message to cause unintended actions; TLS 0‑RTT improves latency but risks replay, so it’s used only for safe, repeatable requests.

Example / analogy:
```
You send: “Transfer ₹10,000”
Attacker captures it
Attacker resends the same encrypted message
Server thinks it’s a new request → money sent again ❌

👉 The message is valid, just replayed.
```

What is 0‑RTT in TLS?
- 0‑RTT lets a client send data immediately when reconnecting, without waiting for the handshake.

Why it exists?
- Saves one network round-trip
- Makes websites load faster

Why or how 0‑RTT enables replay attacks?
- Because the server hasn’t fully verified freshness yet
- An attacker can replay the early (0‑RTT) data

```
Client ---- 0-RTT request ----> Server
        (attacker records this)
Attacker ---- replay same request ----> Server
👉 Server may accept it twice.
```

How systems stay safe (key idea)
1. Only use 0‑RTT for ***idempotent*** requests
(safe to repeat, like `GET`, not POST /pay)
2. (Or) *disable* 0‑RTT for sensitive operations

Prevention of replay attack:
- In **regular (non-0‑RTT) TLS,** a replay attack is practically impossible because of fresh session keys and handshake validation. This is known as ***"Fully verified freshness"***. Example
- `Regular TLS = “Every conversation gets a new padlock and key.”`
- `0‑RTT TLS = “Client can send messages before new padlock is fully checked,”` which is why replay becomes possible.

In TLS context, “fully verified freshness” means the server has completely checked that the session, keys, and messages are new and not reused before accepting any application data.

Broken down:
- Session keys are new – generated during the handshake; old keys cannot decrypt messages.
- Nonces/random numbers are fresh – every handshake uses new random values to prevent reuse.
- Sequence numbers are in order – ensures no previous message is accepted again.
- Handshake completed successfully – both client and server have authenticated each other and agreed on encryption parameters.

#### TLS Pros & Cons

Pros
- Secure
- Standardized
- Trusted by browsers

Cons
- Latency
- CPU cost
- Operational complexity

**Questions**

> Q: Why not encrypt at application layer instead of TLS? A: TLS is standardized, well-audited, and hardware-accelerated.

> Q: Why is TLS handshake expensive? A: Asymmetric cryptography and certificate verification.

> Q: Where would you terminate TLS in a large system? A: At CDN or load balancer, depending on security requirements.

**Enterprise Tools for TLS**
- Cloudflare
- AWS ALB / NLB
- Nginx
- Envoy
- Istio
- Let’s Encrypt

#### TLS termination

Why TLS is terminated at a **CDN** or **Load Balancer**

1. Performance / Offloading
	- TLS handshake and encryption/decryption are CPU-intensive.
	- Terminating at the CDN/LB offloads work from origin servers.
2. Caching & Content Delivery
	- CDNs need to inspect and cache content.
	- If traffic is encrypted end-to-end to the origin, the CDN cannot cache content efficiently.
3. Centralized certificate management
	- Easier to manage TLS certificates at CDN/LB rather than on every backend server.
4. Flexibility / Scalability
	- Backend servers can focus on application logic without worrying about TLS.

***Makes horizontal scaling simpler.***

```
       Client
         |
     HTTPS request
         |
         v
+---------------------+
|  CDN / Load Balancer|  <-- TLS Termination happens here
|  (Decrypts HTTPS,   |
|   caches content)   |
+---------------------+
         |
     HTTP request
   (or HTTPS inside)
         |
         v
   Origin Server
+---------------------+
|  Application Logic  |
+---------------------+
```

## Application layer and protocols

### Hypertext Transfer Protocol

HTTP is:
1. ***Stateless***: Each request is independent.
2. Client-server: Browser (client) requests → server responds.
3. Text-based: Easy to debug.
4. ***Runs over TCP***, often with TLS (HTTPS).

```
CLIENT REQUESTS:
GET /index.html HTTP/1.1
Host: example.com
```
```
SERVER RESPONDS:
HTTP/1.1 200 OK
Content-Type: text/html
...
```

**Note**: 
- HTTP port used is `80`
- HTTPS port used is `443`

**HTTP 1.1 vs 2 vs 3**

* HTTP/1.1: sequential requests, many TCP connections
* HTTP/2: multiplexing streams over one TCP connection, still TCP HoL
* HTTP/3: multiplexed streams over QUIC/UDP, eliminates HoL and reduces latency

| Version  | Key Feature                                       | Pros                           | Cons                              |
| -------- | ------------------------------------------------- | ------------------------------ | --------------------------------- |
| HTTP/1.0 | One request per TCP connection                    | Simple                         | Slow, high latency                |
| HTTP/1.1 | Persistent connections, pipelining                | Fewer TCP handshakes           | HOL blocking in pipelining        |
| HTTP/2   | Multiplexing, header compression, server push     | Single connection, reduced HOL | Needs TLS in browsers             |
| HTTP/3   | QUIC (UDP-based), 0-RTT, multiplexing without HOL | Faster on lossy networks       | Newer, less mature in some stacks |

Multiplexing:
> Multiplexing means sending multiple independent streams of data simultaneously over a single connection.

Streaming:
> A stream is an independent, bidirectional sequence of data within a connection, ***representing a single request–response flow***.

A stream is one conversation or task inside a bigger connection, so many streams can happen at the same time without waiting for each other.

Analogy: Think of a single TCP/QUIC connection as a highway, and each stream as a car traveling on it — multiple cars (streams) can go together.

**Note**:
- A stream does not send all its data in one go.
- Data is broken into frames/chunks
- Each chunk is tagged with a stream ID
- Chunks from different streams are interleaved in multiplexing
- Mental model:
	- Connection = highway
	- Stream = car
	- Chunks = pieces of luggage
	- Cars can enter, exit, and re-enter the highway carrying luggage in parts 🚗📦

> Streams send data in chunks that can be interleaved, so a stream can pause, let others send data, and then resume later.

<!-- TOC --><a name="http-11"></a>
#### HTTP 1.1

Analogy: Each request is a separate single-lane road. Cars (requests) wait for the previous car to finish before the next can go (unless you open multiple TCP connections).

**Characteristics**
- Important: ***TCP connection per request (or pipelining with limitations)***
- No true multiplexing
- Head-of-Line blocking is severe at TCP layer

```
Client              Server
  |                    |
  |--- Req1 ---------> |
  |<-- Resp1 --------- |
  |--- Req2 ---------> |
  |<-- Resp2 --------- |
  |--- Req3 ---------> |
  |<-- Resp3 --------- |
```

**Problems**
1. Sequential; slow for many small resources
2. Multiple TCP connections needed → overhead

#### HTTP 2

Analogy: One multi-lane highway over a single TCP connection.
- Many cars (requests) can go at the same time.
- Each car has a stream ID.
- ***Single TCP connection***
- ***Multiplexed streams over one connection***
- ***Application-layer HoL fixed, but TCP-level HoL remains:***
	- In HTTP/2, multiple streams can be sent simultaneously over one TCP connection (application-layer HoL fixed), but if a TCP packet is lost, all streams are blocked until it’s retransmitted (TCP-level HoL remains)

```
Single TCP Connection
┌───────────────────────────────┐
│ Stream1 │ Stream2 │ Stream3 │
└───────────────────────────────┘
          ↓ all go over TCP
```
```
Packets → [S1-P1][S2-P1][S1-P2 lost][S3-P1][S2-P2]
TCP in-order delivery blocks S3 and S2 until S1-P2 retransmit
```

Key idea:
- Multiple requests multiplexed
- Loss of a single TCP packet blocks all streams → still some HoL

#### HOL differences between HTTP 1 and HTTP 2

**HTTP/1.1 HoL (application-level HoL)**
- Requests are handled one after another per TCP connection
- Response order must match request order

Example: A page needs index.html, style.css, app.js
```
Client            →→→         Server
Req: index.html  ------>
(wait...)
Resp: index.html <------

Req: style.css   ------>
(wait...)
Resp: style.css  <------

Req: app.js      ------>
(wait...)
Resp: app.js     <------
```

Problem
- If index.html is slow:
	- style.css and app.js must wait
		- Even if server has them ready, they’re blocked by order

*👉 This is application-layer HoL*


**What HTTP/2 fixes**
- Multiple requests are multiplexed
- Responses can be interleaved

```
Single TCP connection
Streams: S1(html), S2(css), S3(js)

Packets sent:
[S1][S2][S3][S1][S2][S3]
```

Where HoL still appears
- TCP enforces in-order delivery.

Example with packet loss
```
Packets on wire:
[P1][P2][P3][P4][P5]

P2 = lost ❌
```

TCP behavior:
```
P3, P4, P5 arrive
TCP waits for P2

App gets nothing until P2 is retransmitted
```

Result
- HTML stream ❌ blocked
- CSS stream  ❌ blocked
- JS stream   ❌ blocked

*👉 This is TCP-level HoL*

| Aspect             | HTTP/1.1            | HTTP/2             |
| ------------------ | ------------------- | ------------------ |
| Multiplexing       | ❌ No                | ✅ Yes              |
| App-layer HoL      | ❌ Yes               | ✅ Fixed            |
| TCP-layer HoL      | ❌ Yes               | ❌ Still exists     |
| Packet loss impact | Blocks next request | Blocks all streams |

#### HTTP 3 and QUIC

Analogy: Multi-lane highway with independent lanes, each lane has its own toll booth and traffic light. Packet loss in one lane doesn’t block others.

Characteristics
- ***Runs over QUIC (UDP)***
- ***Multiplexed streams*** with  ***independent ordering***
- ***No transport-layer HoL***

```
QUIC Connection (UDP)
┌───────────────────────────────┐
│ Stream1 │ Stream2 │ Stream3 │
└───────────────────────────────┘
Packets sent independently per stream
Packet loss in Stream1 does NOT block Stream2/3
```

**What is QUIC?**

QUIC is a modern transport protocol built on **UDP** that ***provides TCP + TLS features*** but ***without TCP’s head-of-line blocking***.

Benefits:
1. Reliable delivery
2. Built-in TLS 1.3
3. Multiple independent streams

```
Client
  |
  |  HTTP/3 Streams
  |  S1  S2  S3
  v
======== QUIC (UDP) ============
(independent stream delivery)
  ❌ Loss in S1 → only S1 waits
  |
Server
```

In very simple terms:
1. UDP gives a fast, empty pipe
2. QUIC adds:
	- reliable delivery (retries lost packets)
	- encryption (TLS 1.3 built in)
	- multiple independent streams (no head-of-line blocking)

**QUIC and UDP tango**

QUIC is a transport protocol implemented on top of UDP, ***even though both are considered Layer 4 conceptually.***

How this works (simple mental model)
- OSI layers are conceptual, not physical rules
- OSI doesn’t mean “only one protocol per layer”
- You can build a transport protocol inside another transport protocol

```
App (HTTP/3)
     |
   QUIC   ← real transport logic
     |
   UDP    ← simple packet carrier
     |
   IP
```

**Why QUIC didn’t replace UDP directly?**
- TCP & UDP are in the ***OS kernel***
- Changing them requires OS updates
- UDP is widely allowed through firewalls
- ***QUIC in user space → faster innovation***

### Domain sharding

Domain Sharding is a ***concept, not a protocol***: 
- Splitting requests across multiple subdomains to increase parallel downloads.
- Example: `assets1.example.com`, `assets2.example.com`
- Old hack from HTTP/1.1 era because only ~6 connections per domain.

**Drawback today**
- With HTTP/2 multiplexing → domain sharding can hurt performance.

**Note**: Do not domain shard in today's day and age (HTTP 2+ world)

### HTTPS

HTTPS = TLS + HTTP interplay

- ***Browsers force HTTP/2 over TLS (except local or special cases).***
- `TLS handshake + TCP handshake cost → mitigated by connection reuse, HSTS, 0-RTT.`

Clean explanation:
1. HTTP is an application protocol that defines how clients and servers request and send data.
2. TLS is a security protocol that encrypts and authenticates communication.
3. HTTPS is simply HTTP running over TLS, providing secure, encrypted communication.

In one line: **`HTTPS = HTTP + TLS (secure HTTP)`**

✅ **Pros of HTTPS**
- Security: Encrypts data, preventing eavesdropping and tampering
- Authentication: Verifies server identity via certificates
- Trust & SEO: Required by browsers; improves user trust and search ranking
- Modern features: Enables HTTP/2, HTTP/3, service workers, etc.

❌ **Cons of HTTPS**
- Slight performance overhead: TLS handshake and encryption (mostly negligible today)
- Operational complexity: Certificate issuance, renewal, and management
- TLS termination concerns: Encryption often ends at CDN/LB, not always end-to-end

**HTTPS: Mandatory vs Optional** 

**Mandatory when**:
- Login, payments, personal or sensitive data
- Authentication (cookies, tokens)
- Public internet or modern browser features

**Optional (rare cases)**:
- Public, read-only content
- Trusted internal networks

*Rule of thumb: If it’s on the internet or involves users → use HTTPS.*

### HTTP optimisations

**4**	major optimisations for HTTP network:

| Technique                              | Purpose                      |
| -------------------------------------- | ---------------------------- |
| Client & Browser Caching               | Avoid repeated downloads     |
| CDN (Cloudflare, Akamai, Fastly)       | Serve content closer to user |
| Server Caching (Redis, Varnish)        | Reduce DB / backend hits     |
| Asset Compression (gzip, brotli, zstd) | Reduce transfer size         |

**Questions**

> Q: Why does HTTP/2 improve performance over HTTP/1.1?
A: Multiplexing, header compression, persistent connection reduce latency and HOL blocking.

> Q: Should you use domain sharding today?
A: No, it can reduce HTTP/2/3 multiplexing benefits.

> Q: Why HTTP/3 uses UDP?
A: To avoid TCP HOL blocking and allow 0-RTT connections.

> Q: Difference between HTTP and HTTPS?
A: HTTPS = HTTP over TLS (encrypted, authenticated, integrity-protected).

### HTTP header basics

> HTTP headers carry metadata that control authentication, content type, caching, and security for requests and responses.

Analogy: 
```
Envelope  → Headers (metadata)
Letter    → Body (actual data)
```

<!-- TOC --><a name="common-request-headers"></a>
#### Common ***request*** headers

| Header        | Purpose                              |
| ------------- | ------------------------------------ |
| Host          | Which domain is being requested      |
| Authorization | Auth info (Bearer token, Basic auth) |
| Content-Type  | Format of request body               |
| Accept        | What response formats client accepts |
| User-Agent    | Client identity (browser/app)        |
| Cookie        | Sends stored session data            |


<!-- TOC --><a name="common-response-headers"></a>
#### Common ***response*** headers

| Header         | Purpose                            |
| -------------- | ---------------------------------- |
| Content-Type   | Format of response body            |
| Content-Length | Size of response                   |
| Set-Cookie     | Sets cookies on client             |
| Cache-Control  | Caching rules                      |
| Location       | Redirect target                    |
| Server         | Server info (often hidden in prod) |


<!-- TOC --><a name="headers-for-caching"></a>
#### Headers for ***caching***

Example:
```
Cache-Control: max-age=3600
ETag: "v1.2"
If-None-Match: "v1.2"
```

1. **`Cache-Control`** (most important): Controls if and how long a response can be cached.
```
Cache-Control: max-age=3600      # cache for 1 hour
Cache-Control: no-cache          # must revalidate before use
Cache-Control: no-store          # don’t cache at all
Cache-Control: public            # can be cached by CDNs
Cache-Control: private           # only browser cache
```
> "revalidate": Revalidate ensures stale cached data is verified with the server before being reused.

2. **`ETag`**: A version identifier for the response
```
SERVER SENDS:
ETag: "v3.1"

CLIENT LATER SENDS:
If-None-Match: "v3.1"
```
An ETag (Entity Tag) is a unique identifier for a specific version of a resource. The server sends it with a response, and the client stores it; on the next request, the client includes it using If-None-Match to ask if the resource has changed. If the ETag matches, the server returns 304 Not Modified, saving bandwidth; if not, it sends the updated content.

Server response:
- `304 Not Modified` → use cache
- `200 OK` → send new data

*Etag vs last modified: ETag more precise, better for cache validation*

3. `Last-Modified` (Legacy but, still in use): Timestamp of last change.
```
Last-Modified: Wed, 13 Dec 2025 10:00:00 GMT
If-Modified-Since: Wed, 13 Dec 2025 10:00:00 GMT
```
Older than ETag, but still common.

4. `Expires` (legacy)
```
Expires: Thu, 14 Dec 2025 10:00:00 GMT
```
*Replaced by Cache-Control: max-age.*

<!-- TOC --><a name="headers-for-security"></a>
#### Headers for ***security***

| Header                    | Why                  |
| ------------------------- | -------------------- |
| Authorization             | Secure access        |
| Strict-Transport-Security | Force HTTPS          |
| Content-Security-Policy   | Prevent XSS          |
| X-Frame-Options           | Prevent clickjacking |

1. `Authorization` – Tells the server who the client is or what it can access.
	- Example: `Authorization: Bearer abc123`
	- Bearer token = a secret token proving the client has permission.
2. `Strict-Transport-Security (HSTS)` – Forces the browser to always use HTTPS for the site, preventing downgrade attacks.
	- Example: `Strict-Transport-Security: max-age=31536000; includeSubDomains`
	- `max-age` = how long the browser should enforce HTTPS (in seconds).
	- `includeSubDomains` = applies this rule to all subdomains too.
3. `ontent-Security-Policy (CSP)` – Controls what content (scripts, images, etc.) the browser is allowed to load, preventing attacks like XSS (cross-site scripting).
	 - Example: `Content-Security-Policy: default-src 'self'; script-src 'self' cdn.example.com`
	 - `'self'` = only allow content from the same origin/domain.
	 - `cdn.example.com` = explicitly allow scripts from this CDN.
4. `X-Frame-Options` – Prevents your website from being embedded in an iframe, which protects against clickjacking attacks.
	 - Example: `X-Frame-Options: DENY`
	 - `DENY` = don’t allow any site to embed this page.
	 - Another option: `SAMEORIGIN` allows only your own site to embed it.

**"Clickjacking"**:

Clickjacking is a type of attack where a malicious website tricks a user into clicking something they didn’t intend, usually by hiding or overlaying elements.

Simple example:
- You think you’re clicking a “Play” button on a video.
- But there’s an invisible button on top that actually clicks “Delete account” or approves a transaction.

How `X-Frame-Options` helps
- `X-Frame-Options: DENY` prevents your site from being loaded inside an iframe, stopping attackers from overlaying it for clickjacking.

> Clickjacking tricks users into clicking hidden or disguised elements, often to perform actions without their consent.

## Server side caching vs  CDN caching

### Server side caching

**Server side caching**: Provides a lot of benefits in terms of ***reducing latency***!

| Type               | Example Tools              | Notes                                     |
| ------------------ | -------------------------- | ----------------------------------------- |
| In-memory          | Redis, Memcached           | Very fast, small TTL                      |
| HTTP reverse proxy | Varnish, Nginx             | Can cache full pages or partial responses |
| Application-level  | Django cache, Spring cache | Cache objects, queries                    |

### CDNs

`CDNs = Content Delivery Networks`

**Problem without CDNs**
- High latency (*Even with server side caching*, the request still needs to reach a "geographically distant" server)
- Overloaded origin servers (Every request reaches the origin servers)
- Bandwidth constraints and slow load times

**CDNs** (edge servers): Globally distributed network of servers working together to deliver content
1. Servers close to users
2. Serve static assets (images, JS, CSS) or even dynamic pages
3. Reduce latency, improve availability

```
Browser
   |
   | Request → CDN Edge
   |          ↳ Cache HIT → return content
   |          ↳ Cache MISS → fetch from Origin Server
   |
Origin Server
```

**Popular CDNs**
- Cloudflare
- Akamai
- Fastly
- AWS CloudFront

***Tip: CDNs also handle TLS termination for HTTPS.***

**Benefits of CDNs**
- Reduce latency
- Enhance content availability
- Improve load handling
- Security

#### Cache invalidation strategies

*Cache invalidation* is the process of removing or updating data in the cache when the original data in the database changes. It is famously difficult because you must ensure users don't see "stale" (old) data while maintaining high performance

***Write strategies***:
1. ***Write-Through (The "Safety First" Strategy):*** The application writes data to both the cache and the database at the same time. The write is only considered "success" when both are updated
	- Pros: Data is always consistent. Nothing is ever stale.
	- Cons: Write latency is higher (you have to wait for two writes).
	- Best For: Critical data that cannot be stale (e.g., financial balances, game scores)
```
(1) Write Request
User -----------------> +-------------+
                        | Application |
                        +-------------+
                               |
          +--------------------+--------------------+
          |                                         |
          | (2a) Update directly                    | (2b) Update directly
          v                                         v
    +-----------+                             +------------+
    |   Cache   | <----(Wait for both)------> |  Database  |
    +-----------+                             +------------+
          |                                         |
          +--------------------+--------------------+
                               |
                        (3) Acknowledgment
User <-----------------(Once 2a & 2b are finished)
```
2. ***Write-Around (The "Don't Clutter" Strategy):*** The application writes data directly to the database, skipping the cache entirely. The cache is only populated when someone actively reads that data (***this pairs well with the "Cache-Aside" read strategy***)
	- Pros: Keeps the cache clean. Prevents **"cache flooding**" where you write data that is never read again (e.g., archival logs)
	- Cons: The very first person to read the new data will have a slower experience (`Cache Miss --> Read DB`)
	- Best For: Data written once and read rarely, or big data blobs (logs, backup files)
```
(1) Write Request
User -----------------> +-------------+
                        | Application |
                        +-------------+
                               |
                               | [ Cache is bypassed / ignored ]
                               |
                               | (2) Write directly
                               v
                        +------------+
                        |  Database  |
                        +------------+
                               |
      (3) Acknowledgment       |
User <-------------------------+
```
3. ***Write-Back (The "Speed Demon" Strategy):*** The application writes data only to the cache and immediately confirms "success" to the user. A background process (asynchronously) syncs that data to the database later
	- Pros: Extremely fast writes. Good for write-heavy workloads.
	- Cons: High Risk. If the cache server crashes before syncing, that data is lost forever
	- Best For: Non-critical heavy writes (e.g., analytics counters, video view counts, "likes" on a post)
```
(1) Write Request
User -----------------> +-------------+
                        | Application |
                        +-------------+
                               |
                               | (2) Fast Update & Mark "Dirty"
                               v
      (3) FAST Ack      +-----------+
User <----------------- |   Cache   |
                        +-----------+
                               |
                               | (4) Asynchronous Sync
                               |     (Done later in background)
                               v
                        +------------+
                        |  Database  |
                        +------------+
```

***Special Mention***: ***TTL i.e Time-To-Live*** This is a ***passive strategy*** used alongside the ones above. You simply set a **timer** on every cache item (e.g., "expire in 5 minutes")
- How it works: Regardless of what happens, the data is deleted after time T
- Use Case: The ultimate safety net. If your invalidation code fails, the data will eventually fix itself when the timer runs out.

***A Note on "Delete vs. Update" in cache invalidation***:

When you change data in the DB, should you *Update* the cache value or *Delete* it? Guideline: **Prefer delete**.

Why? If you try to update the cache, *two simultaneous updates might race each other,* leaving you with a mess. If you just delete the key, the next read will simply fetch the fresh, correct version from the DB.

***Read strategies***:
1. ***Cache-Aside (Lazy Loading):*** This is the most common approach where the **Application is in charge**. The application first checks the cache; if the data is missing ("cache miss"), the application itself must query the database, return the result to the user, and then write that data back to the cache for next time. It is called "lazy" because data is only loaded into the cache when someone actually asks for it
```
(1) Get Data?
User ----------------> +-------------+
                       | Application | <---(2) Check Cache (Miss)----> +-------+
                       +-------------+                                 | Cache |
                              |                                        +-------+
                              | (3) Read from DB
                              v
                       +------------+
                       |  Database  |
                       +------------+
                              |
                              | (4) App gets data, returns to User,
                              |      AND updates Cache manually.
                              v
                       (Back to App -> Cache)
```
2. ***Read-Through:*** In this strategy, **the Cache is in charge**. The application treats the cache as the main data store and never talks to the database directly. If the cache doesn't have the data, the cache software/library (using a configured plugin) automatically connects to the database, fetches the data, saves it, and then hands it to the application.
```
(1) Get Data?             (2) Miss? Cache fetches from DB automatically
User ----------------> +-------+ -------------------------------------> +--------+
                       | Cache |                                        |   DB   |
                       +-------+ <------------------------------------- +--------+
                           |            (3) DB returns data to Cache,
                           |                Cache saves it, then sends to User.
                           v
                     (4) Data to User
```

***Common cache read and write strategy pairs***:
- `Cache-Aside + Write-Around` (The "Efficient" Pair)
	- How it works: You read data only when needed (Cache-Aside), and you write data directly to the database, skipping the cache (Write-Around)
	- Why: It prevents **"Cache Pollution"**. You don't fill your cache with data that was just written but might never be read again (e.g., archival logs). The cache stays reserved for "hot" data that is actively requested.
- `Read-Through + Write-Through` (The "Consistent" Pair)
	- How it works: The application treats the cache as the main data store. The cache is responsible for fetching missing data (Read-Through) and saving new data (Write-Through) to the database.
	- Why: It guarantees **"Data Consistency"**. Because the cache handles all movement in and out of the database, your application never sees stale data. It also simplifies your code—the app logic never touches the DB directly.
- `Read-Through + Write-Back` (The "High-Performance" Pair)
	- How it works: The application reads/writes everything to the cache. The cache lazily syncs updates to the DB later (Write-Back)
	- Why: It offers Maximum Speed. Both reads and writes happen at memory speed. You accept the risk of data loss (if the cache crashes before syncing) in exchange for the lowest possible latency.

#### Asset compression in the cache

Why? **Reduce payload size → faster downloads**

| Algorithm | Browser / Tool Support | Compression Ratio                    |
| --------- | ---------------------- | ------------------------------------ |
| gzip      | universal              | 2-3x                                 |
| Brotli    | modern browsers        | 3-5x (better than gzip)              |
| zstd      | server-side / CDN      | Very high compression, CPU efficient |

Example: Nginx config for brotli
```
brotli on;
brotli_static on;
brotli_types text/plain text/css application/javascript;
```

**Combining Techniques**

- `Browser caching` + `CDN` + `compression` + `HTTP/2` → huge performance boost
- Avoid unnecessary domain sharding
- Use persistent connections and multiplexing
- Optimize image/video assets (webp, AVIF, streaming protocols)

**Questions**

> Q: How would you speed up a global web app?
A: Use CDN for static assets, Browser caching with ETag / Cache-Control, Compress assets (gzip/brotli), HTTP/2 or 3 for multiplexing, Server-side caching (Redis, Varnish)

> Q: Difference between ETag and Last-Modified?
A: ETag = hash/version identifier, Last-Modified = timestamp. ETag more precise, better for cache validation

> Q: Why not cache everything forever?
A: Dynamic content changes → must have TTL or invalidation strategy.

## Polling

Polling is a technique where the client (like a web browser) repeatedly asks the server for new data. Since HTTP is a "request-response" protocol, the server cannot push data to the client on its own; the client must ask for it

### Short Polling (The "Nagging" Approach)

- Short polling is like a child in the backseat constantly asking, "Are we there yet?" 
- The ***client sends a request at fixed intervals*** (e.g., every 5 seconds). 
- The server responds immediately, usually with "No new data" (empty response) or "Yes, here is the data." 
- This is simple to implement but wasteful; it consumes bandwidth and server resources even when nothing is happening, creating unnecessary traffic.
- Use Case: Real-time Dashboards (like a sports score tracker or stock ticker) where data changes constantly, so the "waste" of asking repeatedly is minimal because there is almost always something new to report

### Long polling 

- Long Polling (The "Hanging" Approach): Long polling is like saying, "Tell me when we get there," and then waiting for an answer
- The client sends a request, but the server does not respond immediately. Instead, the ***server holds the connection open*** and "hangs" until new data actually becomes available
- Once data arrives, the server sends the response, closing the connection
- The client immediately opens a new request to wait for the next update. This reduces empty requests but keeps server connections tied up for longer
- Use Case: Chat Applications (like WhatsApp Web) or Notification Systems. You want the message the instant it arrives, but you don't want to spam the server every second when no one is talking

```
SHORT POLLING                        LONG POLLING
  (Fixed Interval Checks)             (Wait for Data, then Reply)

Client             Server            Client             Server
  |                  |                 |                  |
  |--- Request 1 --->| (Check)         |--- Request 1 --->| (Holds...)
  |<-- "Nothing" ----|                 |                  |
  |                  |                 |                  |
  |    (Wait 5s)     |                 |                  |
  |                  |                 |                  |
  |--- Request 2 --->| (Check)         |                  | * New Data!
  |<-- "Nothing" ----|                 |                  |
  |                  |                 |<---- "Data" -----|
  |    (Wait 5s)     |                 |                  |
  |                  |                 |--- Request 2 --->| (Holds...)
  |                  | * New Data!     |                  |
  |--- Request 3 --->| (Check)         |                  |
  |<---- "Data" -----|                 |                  |
  |                  |                 |                  |
```

**Long vs short polling**

Short Polling:
- Speed: **Slow**. You always wait for the next "check" interval to see new data.
- Cost: **High CPU & Bandwidth**. Wastes resources answering "No new data" thousands of times.
- Best When: **Data changes constantly** (e.g., every second), so every check is useful.

Long Polling:
- Speed: **Fast**. You get data the instant it arrives.
- Cost: **High Memory**. The server must hold thousands of idle connections open, eating up RAM.
- Best When: **Data changes sporadically** (e.g., chat), so you don't want to spam requests.

Summary: Short Polling is **"Chatty"** (wasteful network calls). Long Polling is **"Heavy"** (wasteful server memory).

**Long polling vs Websockets: When to use what?**
- Use *websockets*:
	- **High frequency, bi-directional data exchange**
	- **Low latency is critical** (Ex: gaming, chat)
	- Ex: Slack uses websockets for chat
	- Ex: Stock exchanges required websockets for real time data
- Use *long-polling*:
	- **Websockets** are **not supported** or are **overkill**
	- **Periodic updates** are sufficient (Ex: Notifications)
	- Ex: Twitter uses long polling for notifications
	- Ex: IoT devices using long polling for intermittent updates

### Comparison of polling with other mechanisms

| Feature | Short Polling | Long Polling | WebSockets | Server-Sent Events (SSE) | HTTP/2 Server Push |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Communication Direction** | **Unidirectional**<br>(Client requests, Server responds) | **Unidirectional**<br>(Client requests, Server waits) | **Bidirectional**<br>(Full-duplex; both send at will) | **Unidirectional**<br>(Server streams to Client) | **Unidirectional**<br>(Server sends resources to Client) |
| **Initiator** | **Client** initiates every check. | **Client** initiates; Server holds connection. | **Client** initiates handshake; then either party sends. | **Client** initiates connection; Server pushes updates. | **Server** predicts need and pushes alongside response. |
| **Connection Persistence** | **None**<br>(New connection every request) | **Short/Medium**<br>(Closes after data received; re-opens) | **Persistent**<br>(Single connection stays open) | **Persistent**<br>(Single connection stays open) | **Transient**<br>(Tied to a specific request cycle) |
| **Data Format** | Standard HTTP (JSON, XML) | Standard HTTP (JSON, XML) | Binary or Text (Custom protocols) | Text only (UTF-8) | Binary (HTTP Frames) |
| **Resource Efficiency** | **Low**<br>(High overhead; many empty responses) | **Medium**<br>(Reduces empty responses, but frequent re-connects) | **High**<br>(Minimal overhead after handshake) | **High**<br>(Standard HTTP compression) | **High**<br>(Uses idle bandwidth) |
| **Primary Use Case** | Legacy systems or rarely changing data. | Simple notifications where WebSocket is overkill. | High-frequency apps (Games, Chat). | Live feeds (News, Stock Tickers). | Pre-loading web assets (CSS/JS). |

<!-- TOC --><a name="server-push"></a>
## Server push

It is an *HTTP/2 feature*

**Concept**: Server proactively sends resources to the client before requested

**Benefit**: Reduces round-trips for critical assets

```
Client → Server : GET /index.html
Server → Client : HTML + CSS + JS (pushed)
```

**Pros**
- Faster page load
- Fewer round trips

**Cons**
- Can push unnecessary data → waste bandwidth
- Hard to control caching

**Enterprise Tools**
- Nginx, Apache HTTP/2
- CDNs (Cloudflare, Fastly)

## Server-Sent Events

SSE stands for Server-Sent Events

**Concept**: Browser opens one-way, long-lived connection

**How it works**: Server pushes text/event-stream updates

**Use case:** live notifications, stock tickers

```
Browser → Server : GET /events (HTTP)
Server → Browser : event1
Server → Browser : event2
```

**Pros**
- Simple (HTTP-based)
- Auto-reconnect built-in
- Works with existing proxies

**Cons**
- One-way only
- Less control than WebSocket
- Limited browser support on older versions

**Enterprise Tools**
- Node.js `EventSource` API
- Spring `SSEEmitter`
- Django `Channels`

## WebSockets

**Concept**: Full-duplex, persistent TCP connection

**Benefit**: Low-latency, real-time communication

```
Client ↔ WebSocket Server
   ↕
Bidirectional messages
```

**Use Cases**
- Chat apps
- Multiplayer games
- Collaborative editing

**Pros**
- Full-duplex
- Low-latency
- Works over TCP

**Cons**
- More complex to scale
- Needs connection management
- Firewalls/proxies sometimes block

**Enterprise Tools**
- `Socket.IO` (Node.js)
- `SignalR` (.NET)
- AWS `AppSync` / API Gateway WebSockets

```
Client                               Server
  |                                   |
  | --- HTTP Upgrade Request --------> |   (Upgrade from HTTP to WebSocket)
  |                                   |
  | <--- HTTP 101 Switching Protocols-|   (Handshake accepted)
  |                                   |
  | <==============================> |   (Full-duplex communication begins)
  |   Messages can flow both ways     |
  |                                   |
```

**Key points**
- Starts as a normal HTTP request (Upgrade: websocket)
- Server responds with `101` Switching Protocols
- After handshake, bi-directional messages flow ***without*** HTTP overhead

### Websocket handshake process

1. Client starts with normal HTTP (Browser/app sends an HTTP request to the server but later it asks to upgrade the connection)
2. Client requests an upgrade. Sends headers like `Upgrade: websocket`, `Connection: Upgrade`, `Sec-WebSocket-Key (random value)`
3. Server agrees (or rejects): If server supports WebSockets, responds with `HTTP 101 Switching` Protocols. Sends back `Sec-WebSocket-Accept`
4. Connection is upgraded: HTTP is now gone. A persistent, full-duplex WebSocket connection is established
5. Real-time data starts flowing: Client and server can now send messages anytime. *No request–response cycle anymore*

```
Client (Browser/App)                         Server
       |                                        |
       |  --- HTTP PHASE ---                    |
       |                                        |
       | (1) Starts as normal HTTP Request      |
       | (2) Asks to "Upgrade" with Headers     |
       | -------------------------------------> |
       |     GET / HTTP/1.1                     |
       |     Host: server.com                   |
       |     Connection: Upgrade                |
       |     Upgrade: websocket                 |
       |     Sec-WebSocket-Key: [Random String] |
       |                                        |
       |                                        |
       | (3) Server Agrees (Switching Protos)   |
       | <------------------------------------- |
       |     HTTP/1.1 101 Switching Protocols   |
       |     Connection: Upgrade                |
       |     Upgrade: websocket                 |
       |     Sec-WebSocket-Accept: [Hashed Key] |
       |                                        |
       |                                        |
========================================================
  (4) CONNECTION UPGRADED! (HTTP protocol ends here)
      Persistent, Full-Duplex TCP connection opens
========================================================
       |                                        |
       |  --- WEBSOCKET PHASE ---               |
       |                                        |
       | (5) Real-time Data Flow starts         |
       |     (No request-response needed)       |
       |                                        |
       | <---------- [Message Data] ----------> |
       |                                        |
       | ----------- [Client Message] --------> |
       |                                        |
       | <---------- [Server Message] --------- |
       |                                        |
       | <---------- [Server Message] --------- |
       |                                        |
       v                                        v
```

### Websockets and load balancers

*Do websockets need load balancers?* **Yes**, absolutely—but they are *much harder to load balance than standard HTTP traffic*

The Problem: WebSockets are "Sticky"
- Standard HTTP requests are *stateless*. You can send Request #1 to Server A and Request #2 to Server B, and it doesn't matter.

*WebSockets are **stateful***: Once a client creates a connection with Server A, that connection stays open. If the Load Balancer sends a packet from that client to Server B, the connection breaks because Server B knows nothing about the handshake.

The Solution: **Sticky Sessions** (Session Affinity)
- To make WebSockets work with a Load Balancer, you must enable Sticky Sessions. This forces the Load Balancer to remember: "This client (IP `1.2.3.4`) is talking to Server A. Do not send them anywhere else."

```
+----------------+
| Client Browser |
+----------------+
       |
       | (1) Initial Handshake Request (HTTP Upgrade)
       |
       v
+-------------------------------------------------------------+
|                      LOAD BALANCER                          |
|-------------------------------------------------------------|
| [ Logic: New connection. Select available server (e.g., S1).|
|   **Action: Create Sticky Record (Client IP <-> Server 1)**]|
+-------------------------------------------------------------+
       |
       | (2) Forward Handshake only to selected server
       |
       v                                 (Other servers ignored)
+------------+                           +------------+
|  Server 1  |                           |  Server 2  |
+------------+                           +------------+
       |
       | (3) Server 1 agrees (HTTP 101 Switching Protocols)
       |
       v
+-------------------------------------------------------------+
|                      LOAD BALANCER                          |
|-------------------------------------------------------------|
| [ Logic: Pass response back to client. ]                    |
+-------------------------------------------------------------+
       |
       v
+----------------+
| Client Browser |
+----------------+

===============================================================
   *** CONNECTION UPGRADED: PERSISTENT TUNNEL ESTABLISHED ***
===============================================================

+----------------+
| Client Browser |
+----------------+
       |
       | (4) Real-time WebSocket Data Frame sent
       |
       v
+-------------------------------------------------------------+
|                      LOAD BALANCER                          |
|-------------------------------------------------------------|
| [ **Sticky Logic:** Check record. Client IP maps to Server 1.]|
| [ Action: Route data straight through persistent pipe to S1]|
+-------------------------------------------------------------+
       |
       | (5) Data travels mapped pipe (Bypassing re-balancing)
       |
       v
+------------+                           +------------+
|  Server 1  | <=== (Tunnel Locked) ===> |  Server 2  | (X)
+------------+                           +------------+
```

### Challenges scaling websockets in a distributed system

Here are the four main headaches of scaling WebSockets:
1. **Memory Hog (Stateful Connections):** Unlike HTTP requests which finish and leave, WebSocket connections stay open forever. A server runs out of RAM and File Descriptors (connection slots) long before it runs out of CPU
```
        HTTP SERVER (Stateless)               WEBSOCKET SERVER (Stateful)
     "Rent a room, leave immediately"       "Move in and stay forever"

     Time 1: 100 Users Request              Time 1: 100 Users Connect
     [Memory: ||||......] (In Use)          [Memory: ||||......] (In Use)
             |                                      |
             v                                      v
     Time 2: Requests Finished              Time 2: 100 New Users Connect
     (Connections Closed)                   (Old ones STAY open!)
     [Memory: ..........] (Empty)           [Memory: ||||||||..] (Filling up)
             |                                      |
             v                                      v
     Time 3: 100 New Requests               Time 3: 100 New Users Connect
     [Memory: ||||......] (In Use)          [Memory: ||||||||||] (FULL!)
             |                                      |
             v                                      v
     RESULT: Stable Memory Usage            RESULT: CRASH (Out of RAM/Ports)
```

2. **Server Isolation (The "Chat" Problem):** User A is on Server 1. User B is on Server 2. They cannot talk to each other directly. You are forced to build a complex "backend bus" (like Redis Pub/Sub) just to pass messages between servers
```
User A (Say "Hi")          User B (Waiting)
            |                          ^
            v                          |
    +--------------+            +--------------+
    |   Server 1   |            |   Server 2   |
    +--------------+            +--------------+
            |                          ^
            | (Message Stuck here)     |
            X                          |
  (Servers don't share memory!)        |
            |                          |
            |      THE FIX: REDIS      |
            v                          |
    +------------------------------------------+
    |           Redis Pub/Sub Layer            |
    | (Server 1 publishes -> Server 2 picks up)|
    +------------------------------------------+
```

3. **Sticky Load Balancing:** You cannot route packets randomly. The Load Balancer must remember exactly which server holds a user's connection. If it sends a packet to the wrong server, the connection breaks (See "sticky sessions" explained in the above section for diagram)

4. **The Reconnection Storm:** If a server with 20,000 users crashes, all 20,000 try to reconnect instantly. This massive spike (Thundering Herd) acts like a DDoS attack and often crashes the surviving servers too
```
    [ Server 1 ]                [ Server 2 ]
      (CRASHES!)                   (Alive)
          |                           |
          v                           |
  10,000 Users Disconnect             |
          |                           |
          | (ALL RETRY AT ONCE)       |
          v                           |
  +-----------------------+           |
  |     LOAD BALANCER     | <---------+
  +-----------------------+
          |
          | (Dumps 10k users onto Server 2 instantly)
          v
     [ Server 2 ]
     (OVERLOADED -> CRASHES!)
```
```
      [ NORMAL STATE ]                  [ SERVER A CRASHES ]
                                    (50k Users Disconnected!)
      +-------------+                
      | Load Balncr |                +-------------+
      +-------------+                | Load Balncr | <=== (PANIC!!)
       /     |     \                 +-------------+
      /      |      \                 /     |     \
  [Srv A] [Srv B] [Srv C]          (X)   [Srv B] [Srv C]
  (50k)   (50k)   (50k)           DEAD    (50k)   (50k)
                                            ^       ^
                                            |       |
                                     +25k Re-connects EACH!
                                     (Instant Spike causes crash)
```

### Server push vs SSE vs Web sockets

| Concept                       | Use Case                                                  | Example                                     | Pros                                                        | Cons                                                                                     |
| ----------------------------- | --------------------------------------------------------- | ------------------------------------------- | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Server Push (HTTP/2 Push)** | Server proactively sends resources before client requests | HTTP/2 pushes CSS/JS when HTML is requested | Reduces page load time; leverages HTTP/2                    | Hard to control; may send unnecessary resources; not suitable for dynamic real-time data |
| **SSE (Server-Sent Events)**  | One-way real-time updates from server to client           | Live news feed, stock tickers               | Simple to implement; built-in reconnection; works over HTTP | Only server → client; no binary support; limited browser support in some cases           |
| **WebSocket (WS)**            | Full-duplex, low-latency communication                    | Chat apps, multiplayer games                | Bi-directional; low overhead; supports binary data          | More complex; connection management; firewall/port issues                                |

**More comparisons**
| Feature     | SSE             | WebSocket             | Server Push      |
| ----------- | --------------- | --------------------- | ---------------- |
| Direction   | Server → Client | Bidirectional         | Server → Client  |
| Protocol    | HTTP            | TCP                   | HTTP/2           |
| Use Case    | Notifications   | Chat, games           | Preload assets   |
| Complexity  | Low             | Medium-High           | Medium           |
| Scalability | Moderate        | Needs connection mgmt | Depends on cache |


## WebRTC

**Concept**: Peer-to-peer,  real-time communication

**Peer-to-Peer (P2P) in WebRTC**
- Meaning: Two clients (browsers or apps) communicate directly with each other without sending all data through a central server.
- Why it matters: Reduces latency and server load.
- Example: Video call between Alice and Bob — their browsers send audio/video directly to each other.

**Real-Time in WebRTC**
- Meaning: Data (audio, video, or messages) is sent immediately as it’s generated, with minimal delay.
- Why it matters: Enables live communication, like calls, screen sharing, or gaming.
- Example: You hear someone speak as they speak, not seconds later.

**Advantage**: Supports audio, video, data channels

**Use Cases**
- Video conferencing (Zoom, Google Meet)
- Live streaming
- Peer-to-peer file transfer

### How WebRTC works

- Uses ICE, STUN, TURN to traverse NATs
- Most clients are behind NATs (Network Address Translators) or firewalls.
	- NAT changes private IPs to public IPs.
	- Without help, two browsers can’t easily find each other to do P2P.

Steps:
1. **STUN** (Session Traversal Utilities for NAT)
	- Lets a client discover its public IP and port as seen from the internet.
	- Example: “I’m behind NAT, my public IP is 203.0.113.5, port 54321”
2. **ICE** (Interactive Connectivity Establishment)
	- Framework that finds the best way for two peers to connect.
	- Tries multiple options (direct, relayed) to establish connection.
3. **TURN** (Traversal Using Relays around NAT)
	- If direct P2P fails, relay traffic via a server.
	- Slower, but ensures connection works even with strict NAT/firewall.

```
Peer A               Peer B
  |                      |
  |----STUN request---->  (discovers public IPs)
  |<---STUN response---- |
  |                      |
  |-----ICE checks------> (tries direct paths)
  |                      |
  |<----ICE response-----|
  |                      |
  |----TURN relay if needed----> TURN Server
  |<----------------------------|
  |                     |
  |----Direct P2P / relayed media--->
```

`STUN = find your public address, TURN = relay when blocked, ICE = pick the best route.`

> WebRTC uses ICE to find the best path, STUN to discover public addresses, and TURN to relay data when direct peer-to-peer connections are blocked.

## MAC address

MAC Address stands for **Media Access Control Address**

- **Purpose**: A unique hardware identifier for a network device’s interface (like Wi-Fi or Ethernet).
- **How it works**: Every network card has a fixed 48-bit address, usually written as `AA:BB:CC:DD:EE:FF.`
- Used for local network communication (LAN), ***not for routing across the internet***.

**Analogy**
- Think of a MAC address as your device’s permanent name tag in your house or office.
- Even if you change rooms (IP address), the name tag stays the same, so people know exactly which device to talk to.

**Key points**
- Unique per device interface
- Works within the local network
- Assigned by the manufacturer (can sometimes be spoofed)

## IP address

**Concept**
- An IP address is like a home address for your computer/device on a network.
- It tells other computers where to send data.
- Without it, the internet would be like a city with no street numbers.

### IPv4 vs IPv6

**IPv4 (most common today)**
- Format: 4 numbers (`0–255`) separated by dots
- Example: `192.168.1.10`
- Size: 32-bit → ~4.3 billion addresses
- Problem: Running out of addresses
- Use Case: Almost all existing internet devices

Structure (simplified):
```
Network part | Host part
192.168.1    | 10
```

“Think of `192.168.1` as street name, `10` as house number”

**IPv6 (future-proof)**
- Format: 8 groups of 4 hex digits separated by colons
- Example: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Size: 128-bit → practically infinite addresses
- Benefits:
	- Eliminates need for NAT (Network Address Translation)
		- A NAT lets multiple private devices share a single public IP by translating addresses and ports for internet communication.
	- Built-in security (IPSec)
	- Easier routing

Shortcut: ***IPv6 lets every device have a unique public address globally.***

### Public vs private IPs

**Public IP**
- Visible on the internet
- Assigned by your ISP
- Example: `103.21.45.67`

**Analogy**: Your house number that anyone in the world can send mail to.

**Private IP**
- Only visible inside your network (LAN)
- Not routable on the public internet
- Common ranges (IPv4):
```
10.0.0.0 – 10.255.255.255
172.16.0.0 – 172.31.255.255
192.168.0.0 – 192.168.255.255
```

Example: `192.168.1.10`

**Analogy**: Apartment number inside a building; the mailman needs the building address (public IP) to reach it.

**How They Work Together**
- Device has private IP: `192.168.1.10`
- Router has public IP: `103.21.45.67`
- When you access a website:
	- Router translates private → public IP using NAT
	- Website sees only 103.21.45.67
	- Response comes back → router sends to correct private IP

**Benefits of Private IPs**
1. Conserves public IP addresses (Especially v4)
2. Enhances security (Private IPs are not known publicly)
3. Enables NATs (Network Address Translation - usually configured on your router) to allow multiple private IPs to share a public IP

```
[Your PC 192.168.1.10] 
           |
           v
[Router NAT: 103.21.45.67]  --> Internet
           |
           v
[Website 142.250.190.78]
```

**Additional benefits of IP addresses in general (All types of IPs)**
1. Load balancing for scalability: It enables load balancers to distribute traffic by using the IP addreess
2. Security based on IP: Firewall rules, VPNs, etc
3. Microservices and container communication: They can talk to each other using internal private IPs

## NAT

NAT (**Network Address Translator)**

**Purpose**: Lets many devices in a private network share a single public IP to access the internet.

**How it works**
- Devices have private IPs (like `192.168.x.x`) that aren’t visible on the internet.
- NAT translates private IP + port → public IP + port when sending packets out.

Important: ***Incoming replies are mapped back to the correct private device.***

**Analogy**
- Your house has many rooms (devices) but only one mailbox (public IP).
- NAT is the mailman who ensures letters from the internet get to the correct room.

**NAT vs other tools**:
NAT translates IPs, Load Balancer spreads traffic, API Gateway manages and secures API requests.

| Component     | Purpose                            | Scope            | Key Point                                             |
| ------------- | ---------------------------------- | ---------------- | ----------------------------------------------------- |
| NAT           | Maps private IPs to public IPs     | Network (L3/4)   | Enables multiple devices to share one IP              |
| Load Balancer | Distributes traffic across servers | L4/L7            | Improves performance, availability, may terminate TLS |
| API Gateway   | Manages and routes API requests    | Application (L7) | Handles auth, rate limiting, logging, transformations |

## Subnetting

Subnetting **splits** a large network into smaller networks (subnets).

**Use case**
Helps organize IPs, improve security, and reduce broadcast traffic.

**Analogy**
- A big apartment complex (network) is divided into blocks (subnets).
- Each block has its own range of apartment numbers (IP addresses).

Example
```
Network: 192.168.1.0/24 (256 addresses, from 192.168.1.0 → 192.168.1.255)

Subnet it into 2 smaller networks: /25 (128 addresses each)

Original /24 network: 192.168.1.0 - 192.168.1.255

Subnet 1 (/25): 192.168.1.0 - 192.168.1.127
Subnet 2 (/25): 192.168.1.128 - 192.168.1.255
```

**When to Subnet**
- You have a big network and want to divide it into smaller networks for organization, security, or traffic management.
- Example: Your `192.168.1.0/24` network for an office can be split into `/26` subnets for different departments.

## CIDR

CIDR stands **Classless Inter-Domain Routing**. CIDR is a **notation** to specify IP ranges using a suffix `/n` instead of old classful addresses.

`/n` = number of bits used for the network part of the address.

Example
- `192.168.1.0/24` → first 24 bits are network, last 8 bits are host
- `/25` → first 25 bits network, 7 bits host → 128 addresses

```
IP:       192.168.1.0
Binary:   11000000.10101000.00000001.00000000
Network:  first 24 bits /24
Host:     last 8 bits

Dividing into /25:

Subnet 1: 11000000.10101000.00000001.0xxxxxxx (0-127)
Subnet 2: 11000000.10101000.00000001.1xxxxxxx (128-255)
```

**When to use CIDR**
- You want to define or communicate a network range efficiently.
- CIDR is the notation that tells devices how many bits are for the network vs hosts.
- Example: Instead of saying “Network class C with 256 addresses,” you say 192.168.1.0/24.

> Subnetting divides networks into smaller parts; CIDR specifies network size using a /n suffix.

| Concept    | Meaning                                                 | Example                                                                  | Key Point                                                 |
| ---------- | ------------------------------------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------- |
| **Subnet** | A smaller network created by splitting a larger network | `192.168.1.0/24` → two subnets `/25`: 192.168.1.0-127, 192.168.1.128-255 | Focuses on **dividing networks into smaller parts**       |
| **CIDR**   | Notation to represent network + host bits flexibly      | `192.168.1.0/24` or `/25`                                                | Focuses on **representing network size** in a compact way |

Simple way to remember
- `Subnetting = “split the cake”`
- `CIDR = “label the pieces”`

## DNS

**Definition**
- DNS is like the internet’s phonebook.
- It ***translates human-friendly domain names*** (e.g., example.com) into ***IP addresses*** (e.g., `93.184.216.34`) so computers can communicate.

Example
- You type www.google.com in your browser.
- Browser asks DNS: “What is the IP for www.google.com?”
- DNS replies: `142.250.190.68`
- Browser connects to that IP

```
Client (Browser)
   |
   | --- DNS Query: "www.google.com?" ---->
   |
DNS Resolver
   | 
   | --- Root Server -----------------> (find .com TLD server)
   | --- TLD Server -----------------> (find google.com NS)
   | --- Authoritative NS ---------> (return IP 142.250.190.68)
   |
   <--- DNS Response: 142.250.190.68 ----
   |
Client connects to 142.250.190.68
```

**Why DNS matters**
- Converts human-friendly names → machine-friendly IPs
- Enables scalable and distributed internet addressing
- Supports email, web, and app services

### Types of DNS and DNS components

| Component                         | Description                                             | Example                                      |
| --------------------------------- | ------------------------------------------------------- | -------------------------------------------- |
| **TLD (Top-Level Domain)**        | The last part of a domain                               | `.com`, `.org`, `.in`                        |
| **Second-Level Domain (SLD)**     | Usually the organization or brand                       | `google` in `google.com`                     |
| **Subdomain**                     | Optional prefix for services                            | `www`, `mail`, `maps`                        |
| **Authoritative Nameserver (NS)** | Server that knows the exact IP of a domain              | `ns1.google.com`                             |
| **Root Server**                   | The top of DNS hierarchy                                | Points to TLD servers                        |
| **Recursive Resolver**            | Resolver that queries the chain on behalf of the client | Provided by ISP or public DNS like `8.8.8.8` |


**Recursive resolver**
- A recursive resolver is a DNS server that finds the IP address for a client by querying the DNS hierarchy on its behalf.
- A server that does the “legwork” of finding an IP for a domain on behalf of your device.
- It queries root → TLD → authoritative nameservers until it gets the answer, then returns it to the client.
- Usually provided by your ISP or a public DNS like `8.8.8.8` (Google DNS) or `1.1.1.1`(Cloudflare)
- Analogy:
	- You ask the resolver: “Where does www.example.com live?”
	- It runs around the neighborhood (root, TLD, authoritative servers) and tells you the exact address.

**Root server**
- A root server is the top-level DNS server that directs queries to the correct TLD servers.
- The top-most DNS server in the hierarchy that knows where all TLD (Top-Level Domain) servers are.
- It doesn’t know the final IP of a website, but it directs queries to the correct TLD server.
- There are ***13 logical root server addresses (A–M) globally***, each with many physical instances.
- Analogy:
	- Think of the root server as a city directory: it doesn’t know individual house addresses, but tells you which street to go to for each neighborhood (TLD).

**Authoritative name server**
- An authoritative nameserver is the DNS server that provides the final, definitive IP address or record for a domain.
- The DNS server that has the definitive answer for a domain’s IP address.
- When queried, it responds with the actual IP or other DNS records for that domain.
- Example: For example.com, ns1.example.com is often the authoritative NS.
- Analogy:
	- Think of it as the owner of the house: it knows exactly which apartment number corresponds to the person you’re looking for.

### DNS request flow and hierarchy

```
Client (Browser)
   |
   |--- Request IP for www.example.com ---> Recursive Resolver
   |
   |--- Resolver asks Root Server ---> Points to .com TLD Server
   |
   |--- Resolver asks TLD Server ---> Points to Authoritative NS for example.com
   |
   |--- Resolver asks Authoritative NS ---> Returns IP: 93.184.216.34
   |
   <--- Resolver sends IP back to Client ---
   |
Client connects to 93.184.216.34
```

```
          Root Server
           (. )
            |
       -------------
       |     |     |
     .com   .org   .in   <- TLD Servers
       | 
  Authoritative NS for example.com
       |
     www.example.com → 93.184.216.34
```

**Scenario**

Client wants to visit: www.example.com

**Step 1**: Check Local Cache
- If request originates from the browser, it checks its cache for a lookup
- If browser did not find a match in its cache (or we did not use a browser), the **client OS** checks its local DNS cache for the domain.
- If cached → return IP immediately.
- If not cached → proceed to recursive resolver.

**Step 2**: Recursive Resolver
- The recursive resolver (ISP or public DNS) handles the query.
- It too checks its own cache for the IP. If not found, it checks higher up the hierarchy
	- Its job: find the IP by asking the DNS hierarchy.

**Step 3**: Query Root Server (13 of them exist globally)
- Resolver asks a root server:
- “Where can I find .com domains?”
- Root server doesn’t know the IP, but points resolver to the TLD server for .com.
- *Component used*: Root Server
- *DNS type*: Indirect, top-level referral

**Step 4**: Query TLD Server
- Resolver asks TLD server: “Where is example.com?”
- TLD server responds with the authoritative NS for example.com.
- *Component used*: TLD server
- *DNS type*: NS record (Nameserver for the domain)

**Step 5**: Query Authoritative Nameserver
- Resolver asks authoritative NS: “What is the IP for www.example.com?”
- Authoritative NS responds with the `A` record (or `AAAA` for IPv6): `www.example.com → 93.184.216.34`
- *Component used*: Authoritative NS
- *DNS type*: `A/AAAA` record (actual IP), may include `CNAME` if www is an alias

**Step 6:** Return IP to Client
- Recursive resolver ***caches*** the response (based on TTL) for future requests.
- Sends the IP back to the client.
- Client connects to `93.184.216.34`.
	- *Response is usually cached* at ISP (recursive resolver, your OS, and your browser (if it applies))

<!-- TOC --><a name="types-of-dns-requests"></a>
### Types of DNS requests

| Type            | Purpose                      | Example                                            |
| --------------- | ---------------------------- | -------------------------------------------------- |
| **A record**    | IPv4 address                 | `example.com → 93.184.216.34`                      |
| **AAAA record** | IPv6 address                 | `example.com → 2606:2800:220:1:248:1893:25c8:1946` |
| **CNAME**       | Alias to another domain      | `www.example.com → example.com`                    |
| **MX**          | Mail server                  | `example.com → mail.example.com`                   |
| **NS**          | Nameserver                   | `example.com → ns1.example.com`                    |
| **PTR**         | Reverse lookup (IP → domain) | `93.184.216.34 → example.com`                      |

### DNS caching

DNS caching is the process of temporarily storing the IP address of a recently visited domain (like saving a contact in your phone) so that your computer doesn't have to ask the global network where to find it every single time you load a page.

**DNS caching helps to**
1. Reduce latency i.e less time to know the IP address and hit server
2. Reduce load on DNS servers themselves i.e fewer requests going through them

**Where does caching occur?** It occurs in 3 places:
1. *Browser* (Browser can cache the previous IP lookups)
2. *OS cache* (`etc/hosts` file on MacOS, Windows DNS cache) i.e OS too can cache lookups at the system level
3. *Recursive resolver* (@ ISP): Your ISP too will usually maintain a lookup table as a cache that the recursive resolver will use

**When does the cache get invalidated?**
- **Time-To-Live (TTL)**: This attribute to the cached lookup determines when the cache should be invalidated so that fresh request do actually hit the DNS. It helps avoid stale lookups. Ex: If server IP has changed but we still use the old one indefinitely, it is a problem!

### DNS in large scale systems

**DNS ensures "high availability"**
  1. DNS can act as a *load balancer (a primitive one though)*
  2. Anycast DNS: Instead of a single DNS, we use multiple geographically distributed DNSes. Why? Ensures users get the response from the closest DNS -- also the fastest!
2. DNS failover strategies: Use a primary and a secondary DNS (Fault tolerance?)
3. DNS routing to CDNs: DNS can route requests to the nearest CDN for faster loading -- mainly for static assets that need to load quickly and they don't change frequently

**DNS security risks**:
  - **DNS poisoning**: This is hacking the "address book" of the internet so that when a user types a legitimate website name (like https://www.google.com/search?q=google.com), they are secretly redirected to a fake malicious site (**Analogy**: It’s like a prankster breaking into the phone company’s database and swapping your bank's phone number with their own; when you dial the bank, you unknowingly talk to the scammer). **Solution**: The most effective solution is to implement **DNSSEC** (Domain Name System Security Extensions), which adds cryptographic signatures to DNS records so that computers can verify the data actually came from the authoritative source and was not faked. 
  - **Cache poisoning**: This involves tricking a system (like a browser or server) into storing a fake or malicious file, which it then automatically serves to other users thinking it is the real version (Analogy: It’s like someone slipping a fake answer key into a teacher's desk; the teacher (the cache) confidently hands out the wrong answers to every student who asks, spreading the error automatically). **Solution**: configure your cache to treat requests with different inputs (like headers) as completely different pages so that the previous page's cache is not affected
  - **DDOS attacks (Distributed Denial of Service)**: This is an attack where thousands of hijacked computers flood a target server with junk traffic to crash it or make it unusable for real people (Analogy: It’s like a group of 500 people crowding into a small coffee shop and refusing to buy anything, making it impossible for actual customers to enter or get served). **Note**: **CDNs** help avoid DDOS attacks on DNS and your company servers

## Load balancers

A Load Balancer is a device or service that **distributes incoming network or application traffic across multiple servers** to *improve availability, reliability, and performance*.

A Load Balancer provides **High Availability** and **Failover**

Works at **Layer 4 (TCP/UDP)** or **Layer 7 (HTTP/HTTPS)**.

```
          Clients
         /   |   \
        /    |    \
       v     v     v
   +-------------------+
   |   Load Balancer   |
   +-------------------+
       |     |     |
       v     v     v
   Server1 Server2 Server3
```

- Clients send requests to the LB, which forwards them to the least busy or appropriate server.
- LB can also perform health checks and SSL termination.
- Analogy:
	- LB = Reception desk in a hotel
	- Guests (clients) arrive
	- Reception (LB) decides which room (server) to send them to
	- Ensures no room is overcrowded and broken rooms are skipped

**Examples: LBs based on deployment**
1. ***Hardware based***: F5 BIG-IP, Citrix ADC
2. ***Cloud based***: AWS ELB, Google Cloud Load Balancing, Azure Load Balancer
3. ***Software based***: NGINX, HAProxy

**Pros**
- High availability → if a server fails, LB redirects traffic (Handles failures gracefully)
- Scalability → add/remove servers without affecting clients
- Health checks → ensures traffic only goes to healthy servers
- SSL/TLS termination → reduces load on backend servers ***(Note: This can also be a con in terms of security! Communication beyond the LB and to the servers is not TLS encrypted)***

**Cons**
- Single LB failure → if not redundant, it becomes a bottleneck
- Added latency → traffic goes through an extra layer
- Complexity → configuration and monitoring required

**When to Use**
- Web apps with many concurrent users
- Microservices architectures needing traffic distribution
- High availability or fault tolerance requirements

**When Not to Use**
- Small apps with very few users
- Simple static content sites where direct server access is fine

**Load balancing strategies**
1. **Static LB strategies**
	1. **Round Robin**: Distributes requests sequentially to each server
	2. **Least connections**: Directs traffic to the server with the least connections
	3. **IP Hashing**: Routes requests based on client IP
2. **Dynamic LB strategies**:
	1. **Least Response Time**: Request sent to server with the least response time
	2. **Adaptive load balancing**: Use real-time monitoring to make decisions
	3. **Weighted load balancing**: Assigns different weights to servers based capacity

### Choosing a load balancing strategy

Key questions to ask: 
- Are my servers equal?
- Do my users need to stay put?
- How long do requests take?

Here is a simple guide to choosing the right strategy (algorithm).
1. ***The "Default" Choice: Round Robin***
- How it works: Goes down the list 1-by-1 (Server A, then B, then C, repeat).
- Best for: Stateless applications where all servers have the same specs (CPU/RAM).
- Why: It is simple, foolproof, and requires almost no processing power.

2. ***If Servers Are Unequal: Weighted Round Robin***
- How it works: You assign a "weight" (score) to servers. A powerful server gets 5 requests for every 1 request a weak server gets.
- Best for: Mixed hardware (e.g., you have one brand new powerful server and two older, slower ones).
- Why: It prevents the old servers from crashing while the new one sits idle.

3. ***If Requests vary in Length: Least Connections***
- How it works: Sends the next user to the server with the fewest active users right now.
- Best for: Long-lived connections (e.g., WebSocket chats, video streaming, or heavy database queries).
- Why: In Round Robin, Server A might get stuck with 5 "heavy" users while Server B has 5 "fast" users. "Least Connections" balances the actual load, not just the request count.

4. ***If You Need "Memory": IP Hash / Sticky Sessions***
- How it works: The LB remembers the user's IP address (or cookie) and always sends them to the exact same server.
- Best for: Shopping carts or apps that store data locally on the server RAM rather than a shared database.
- Why: If a user adds an item to their cart on Server A, and the next request goes to Server B, their cart will appear empty.

### L4 vs L7 load balancers

- **L4** (Transport Layer): Makes decisions based on **numbers** (IP address and Port). It does not look at the message content.
	- How it works: It looks at the TCP/UDP protocol info.
	- Speed: Very fast because it does not decrypt or inspect the data
	- Key Behavior: Once a connection is established, it just passes packets back and forth without analyzing them
- **L7** (Application Layer): Makes decisions based on **content** (URL, Cookies, Headers). It reads the message to understand what the user wants
	- How it works: It looks at the HTTP/HTTPS headers, URLs, and cookies.
	- Speed: Slower than L4 (more processing required), but much "smarter."
	- Key Behavior: It can terminate SSL (decrypt the traffic) to read the request, then re-encrypt it to send it to the server.

| Feature |	L4 Load Balancer |	L7 Load Balancer |
| -- | -- | -- |
| OSI Layer |	Transport Layer (Layer 4)	| Application Layer (Layer 7) |
| Visibility |	IP Address & Port |	URL, Headers, Cookies, Data |
| Complexity |	Low (Simple routing) |	High (Intelligent routing) |
| Performance	| High (Less processing) |	Lower (More CPU intensive) |
| Decryption |	No (Usually just passes encrypted data) |	Yes (SSL Termination) |

***Practical example***: You move to microservices from a monolith and now you need to route to different microservices based on the route (Ex: `/cart`). For this, you need to move from an L4 to L7 load balancer!

### Consistent hashing

**What is Consistent Hashing?**
- It is a ***distributed hashing scheme*** that maps data to servers using a ***circular data structure*** (a "Ring")
- Its main feature is that ***adding or removing a server does not force you to reorganize all data***—only a small fraction moves!

**The Problem It Solves**
- It solves the "Rehashing Storm" seen in traditional hashing (`hash(key) % N`)
- Traditional: If you change the number of servers (`N`), the math changes for every key
- 100% of data moves, crashing the system
- Consistent: Only the "neighbors" of the new server are affected. ~90% of data stays where it is.

**How It Works (The Math)**
- Imagine a circle representing values `0` to `360`.
- **Servers**: Hash server IPs to place them on the circle.
- **Keys**: Hash data keys to place them on the circle.
- **The Rule**: A key belongs to the first server found moving Clockwise on the ring

```
Think of the hash space as a path starting at the top (0) 
and going clockwise around the square.

The Rule: A Key belongs to the first Server it hits while moving clockwise.

Server A owns everything in the top-left quadrant. 
Server B owns the top-right, and so on.

START (0) >----------------[ Server A ]----------------+
|                                                      |
|   (Range owned by A)            (Range owned by B)   |
|                                                      |
|                                          * Key_1     |
|                                            |         |
|                                            v         |
[ Server D ] <---------------------------[ Server B ]  |
|                                                      |
|   (Range owned by D)            (Range owned by C)   |
|                                                      |
|   * Key_2                                            |
|     |                                                |
|     v                                                |
+------------------------[ Server C ]------------------< END


Key_1: Lands between A and B. It moves clockwise and hits Server B.
Key_2: Lands between C and D. It moves clockwise and hits Server D.
```
```
We are zooming in on the path between Server A 
and Server B from the previous diagram.

We add a new Server E in the middle of that path.

BEFORE adding Server E: Server B owned the entire path. 
Both Key_X and Key_Y would have gone to Server B.

AFTER adding Server E:

          [ Server A ] (Start of range)
              |
              |
    (Region 1)|  * Key_X (Starts here)
              |    |
              |    v (Moves Clockwise)
              |
NEW ----> [ Server E ] (INTERCEPTION!)
NODE          |    ^
              |    | (Key_X now parks here)
              |
    (Region 2)|  * Key_Y (Starts here, past E)
              |    |
              |    v (Moves Clockwise)
              |
         [ Server B ] (Original owner)
                   ^
                   | (Key_Y still parks here)


The Interception (Key_X): 
Because Server E was placed before Key_X's original destination, 
Server E "intercepts" it. Data moves.

The No-Op (Key_Y): Because Key_Y is located after the new Server E, 
its path to Server B is unchanged. Data stays put.

Efficiency: Only the data in "Region 1" had to move. 
"Region 2" remains untouched.
```

To "move data" for a new region:
1. Identify the Successor: Find the node directly clockwise of the new server.
2. Filter: The successor identifies which of its keys now fall into the new server's range.
3. Migrate: Only those specific keys are copied over network to the new server.

Use cases of consistent hashing:
- ***Distributed Caches***: (Redis Cluster, Memcached) To add cache nodes without flushing the whole memory.
- ***NoSQL Databases***: (Cassandra, DynamoDB) To partition Terabytes of data across thousands of nodes.
- ***Load Balancers***: To ensure "Sticky Sessions" (User A always maps to Server A) even when the server fleet scales up/down.

### Consistent hashing in load balancers

In the context of a Load Balancer (LB), Consistent Hashing is used to solve one specific problem: **Sticky Sessions** (also called Session Affinity)

```
     [ User A (IP: 1.2) ]       [ User B (IP: 5.6) ]
               |                          |
               v                          v
      +-------------------------------------------+
      |             LOAD BALANCER                 |
      | Logic: Hash(Source_IP) -> Map to Ring     |
      +-------------------------------------------+
               |                          |
               | (Consistently            | (Consistently
               |  routes to A)            |  routes to B)
               v                          v
      +------------------+       +------------------+
      |    SERVER A      |       |    SERVER B      |
      | (Has User A's    |       | (Has User B's    |
      |  Cart in RAM)    |       |  Cart in RAM)    |
      +------------------+       +------------------+
```

## API gateways

An API Gateway is a server that acts as a ***single entry point for clients to access multiple backend services or microservices.***

- It handles routing, authentication, rate limiting, caching, logging, and request/response transformations.
- Works at **Application Layer (L7)**.

```
          Clients
         /   |   \
        v    v    v
   +-------------------+
   |   API Gateway     |
   +-------------------+
   | Auth | Rate Limit |
   | Cache| Logging    |
   +-------------------+
      |     |     |
      v     v     v
   Service1 Service2 Service3
```

**Analogy**
- API Gateway = Reception desk in a large office building
- Visitors (clients) arrive
- Reception checks ID (auth), enforces rules (rate limits), and directs them to the correct department (service).
- Reduces chaos and ensures security and logging

**Examples**
- Cloud: AWS API Gateway, Azure API Management, GCP Apigee
- Open-source: Kong, Tyk, Traefik, NGINX as API gateway

**Pros**
- Centralized security and authentication
	- Supports *OAuth*, *JWT*, *API keys*
	- DDOS protection 
	- Bot protection
	- TLS termination: similar to a load balancer
- Simplifies client interaction with many microservices
	- Combines multiple client requests to different servers into just one to the gateway
	- The gateway takes care of making those multiple calls to the servers internally and fetching the data
- Rate limiting: Restricts number of calls from a user/IP per second/minute
- Throttling: Controls traffic during peak loads to avoid system crashes
- Caching
- Logging & monitoring for observability and performance
- Can transform requests/responses to match service requirements (**Request transformation**)
	- Ex: Client might send a payload using gRPC but a server accepts only REST over HTTP. API Gateway will do this transformation, both for the request as well as in the opposite direction i.e for the response

**Cons**
- Single point of failure if not redundant
- Added latency since all traffic passes through it
- Extra complexity in configuration and maintenance

**When to Use**
- Microservices architecture with multiple APIs
- Multi-client APIs (Ex: web, mobile, IoT)
- Need centralized auth, rate limiting, logging, or request transformation
- Public APIs that need security and monitoring

**When Not to Use**
- Small, monolithic apps (Ex: Low traffic)
- Direct, simple client-server setups with minimal security or routing requirements (Ex: internal only services)

**Note**: Similar to Load Balancers, an ***API Gateway too can act as a reverse proxy to servers***

### API gateways vs Load balancers

Load balancers focus on distributing traffic across servers, while API gateways focus on managing, securing, and routing API requests.

The "Acid Test" Difference:

> If you are only routing traffic based on `/video` vs. `/images`, you are doing L7 Load Balancing. If you are asking, "Does this user have a valid API key?" or "Has this user made too many requests?", you are using an API Gateway.

| Feature                   | API Gateway                                                         | Load Balancer                                                              | Example Use Case                                             |
| ------------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------ |
| **Purpose**               | Central entry point for APIs; manages, secures, and routes requests | Distributes traffic across servers to improve availability and performance | API Gateway: AWS API Gateway; LB: AWS ELB                    |
| **Layer**                 | Application Layer (L7)                                              | Layer 4 (TCP/UDP) or Layer 7 (HTTP/HTTPS)                                  | NGINX, HAProxy                                               |
| **Routing**               | Routes based on URL paths, headers, methods                         | Routes based on IP, TCP/UDP, or HTTP host/path                             | `/api/user` → UserService vs any request → least busy server |
| **Security**              | Handles auth, tokens, rate limiting, logging                        | Usually does TLS termination, may do basic access control                  | OAuth2 token check vs SSL termination                        |
| **Transforms / Features** | Can modify requests/responses, caching, protocol translation        | Mostly pass-through or simple load distribution                            | Convert JSON → XML or route HTTP → gRPC                      |
| **When to use**           | Microservices, public APIs, need security & monitoring              | Distribute traffic across multiple servers, ensure high availability       | Microservice architecture vs high-traffic web app            |

## Proxy and reverse proxy

**Proxy**: It is an *intermediary server* between server and client

| Type                      | Definition                                                                                                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Proxy (Forward Proxy)** | A server that acts on behalf of **clients**, forwarding requests to the internet and returning responses. Clients know and configure the proxy.                           |
| **Reverse Proxy**         | A server that acts on behalf of **servers**, receiving requests from clients and forwarding them to backend servers. Clients usually don’t know the reverse proxy exists. |

*Tip*: *The direction of flow is from the client (start) to the server (end)*

**Usefulness: Why do we need them?**
- Proxy in general: Security, caching, traffic control
- **Forward proxy** :
	- *Privacy / Anonymity*: Hides user IP (Ex: ***VPN is a forward proxy***)
	- *Content filtering* (Access control): Restrict certain websites (Ex: your company blocks you from visiting NSFW sites)
	- *Bypass geo-restrictions*: Access region-blocked content (Ex: ***VPN*** again)
	- *Caching*: Reduce redundant requests, speed up browsing
- **Reverse proxy**
	- *Load balancing*: Distribute the load among server (Ex: ***Nginx, Cloudflare, AWS LB***). Do not overload any single one
	- *Security and DDOS protection*: Hide backend servers, block attacks
	- *SSL termination*: Offload TLS termination to proxy instead of individual servers
	- *Caching and compression*: Again, reduces load on servers and saves bandwidth

### Forward proxy or proxy

```
           +-----------------+
           |     Client      |
           +-----------------+
                    |
                    | Request (to a website)
                    v
           +-----------------+
           | Forward Proxy   |
           +-----------------+
                    |
                    | Forwards request
                    v
           +-----------------+
           | Internet Server |
           +-----------------+
                    |
                    | Response
                    v
           +-----------------+
           | Forward Proxy   |
           +-----------------+
                    |
                    | Sends response
                    v
           +-----------------+
           |     Client      |
           +-----------------+
```

**Flow explained**
1. Client sends request to the proxy instead of directly to the internet.
2. Forward Proxy forwards the request to the target server.
3. Server responds to the proxy.
4. Proxy sends the response back to the client.

### Reverse proxy

```
           +-----------------+
           |     Client      |
           +-----------------+
                    |
                    | Request (to a website)
                    v
           +-----------------+
           | Reverse Proxy   |
           +-----------------+
                    |
       -----------------------------
       |             |             |
       v             v             v
   Server1        Server2       Server3
                    |
                    | Response
                    v
           +-----------------+
           | Reverse Proxy   |
           +-----------------+
                    |
                    | Sends response
                    v
           +-----------------+
           |     Client      |
           +-----------------+
```

**Flow explained**
1. Client sends request to the reverse proxy (they usually don’t know about backend servers).
2. Reverse Proxy forwards the request to one of the backend servers (load balancing possible).
3. Server responds → Reverse Proxy receives the response.
4. Reverse Proxy sends the response back to the client.

**Analogy**

| Type              | Analogy                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Forward Proxy** | You ask a **friend to buy something from a store** for you. The store sees the friend’s ID, not yours.                          |
| **Reverse Proxy** | Clients visit a **hotel reception** that decides which room (server) handles the request. Clients don’t see the rooms directly. |

**Examples**

| Type          | Example                                               |
| ------------- | ----------------------------------------------------- |
| Forward Proxy | Corporate proxy for internet access, VPN, Squid Proxy |
| Reverse Proxy | NGINX, HAProxy, AWS CloudFront, Cloudflare            |

**Pros and cons**

| Type          | Pros                                                                                          | Cons                                                                                             |
| ------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Forward Proxy | - Can filter/block content<br>- Hides client IP<br>- Caching for faster access                | - Needs client configuration<br>- Can be a single point of failure                               |
| Reverse Proxy | - Load balancing<br>- SSL termination<br>- Caching and compression<br>- Hides backend servers | - Adds latency<br>- Single point of failure if not redundant<br>- Extra configuration complexity |

**When to use / not use**

| Type          | When to Use                                                                            | When Not to Use                                               |
| ------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| Forward Proxy | Controlling/monitoring client traffic, caching web content, bypassing geo-restrictions | Small networks, clients don’t need centralized access control |
| Reverse Proxy | Load balancing, securing backend servers, caching, SSL termination                     | Single-server apps with minimal traffic, simple setups        |


### Reverse proxy vs Load balancer

*They are **quite similar and can be implemented by the same tool / cloud service** but ***it need not be the same conceptually***!*

A reverse proxy primarily secures, manages, and routes requests to servers, while a load balancer primarily distributes traffic across servers for availability and performance.

| Feature                 | Reverse Proxy                                                                                   | Load Balancer                                                                           | Example Use Case                                                                                        |
| ----------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Purpose**             | Acts on behalf of servers, forwards client requests, can add security, SSL termination, caching | Distributes traffic across multiple servers to ensure high availability and performance | Reverse Proxy: NGINX in front of web servers; LB: AWS ELB distributing requests to multiple app servers |
| **Layer**               | Application Layer (L7)                                                                          | Layer 4 (TCP/UDP) or Layer 7 (HTTP/HTTPS)                                               | NGINX, HAProxy                                                                                          |
| **Routing**             | Can route based on URL path, host headers, or other application info                            | Can route based on IP, TCP/UDP, or HTTP host/path                                       | `/api/user` → UserService (Reverse Proxy) vs Any request → least busy server (LB)                       |
| **Security**            | Can handle SSL/TLS termination, hide backend servers, enforce auth                              | Usually handles TLS termination; not primarily for security                             | Reverse Proxy hides real server IPs                                                                     |
| **Additional Features** | Caching, compression, request/response transformation                                           | Health checks, sticky sessions                                                          | Reverse Proxy can modify requests; LB mainly balances load                                              |
| **When to use**         | Securing backend servers, centralizing routing, adding caching                                  | Distributing traffic for scalability and fault tolerance                                | Microservices with exposed APIs (Reverse Proxy) vs Web app needing high availability (LB)               |

### Reverse proxy vs API gateway

Reverse Proxy is mainly about protecting and routing to servers, while an API Gateway is about managing, securing, and transforming API requests for clients.

| Type              | Acts on behalf of         | Main job                                                                           |
| ----------------- | ------------------------- | ---------------------------------------------------------------------------------- |
| **Reverse Proxy** | The **server**            | Protects backend servers, routes requests, can do caching or SSL termination       |
| **API Gateway**   | The **API/microservices** | Manages all API traffic, handles security, routing, rate limiting, transformations |

- Reverse Proxy: NGINX sits in front of Server1, Server2, Server3 → forwards HTTP requests.
- API Gateway: AWS API Gateway sits in front of User Service, Payment Service, Order Service → handles authentication, rate limiting, routing, and request/response changes.

**Tip to remember**
- Reverse Proxy = Server-focused (routes & protects servers)
- API Gateway = API-focused (manages traffic, security, and transformations for clients)

Example
```
                    Clients
                 /     |     \
                v      v      v
          +-----------------------+
          |    Reverse Proxy      |
          |  (protects servers)   |
          +-----------------------+
                   |
           -------------------
           |                 |
        Web Server1       Web Server2
           |
           v
     +-----------------------+
     |    API Gateway        |
     | (manages APIs & sec) |
     +-----------------------+
       |      |       |
       v      v       v
   User Service  Payment Service  Order Service
```

**Flow Explained**
- Clients send requests to the Reverse Proxy first.
- RP protects backend web servers, hides their IPs, may handle SSL termination.
- Requests needing microservice APIs are routed through the API Gateway.
- API Gateway handles authentication, rate limiting, request/response transformation, logging, etc.
- API Gateway forwards requests to the correct microservice.
- Responses flow back the same path to the client.

**Memory Tip**
- `Reverse Proxy = server-focused, first line of defense`
- `API Gateway = API-focused, smart traffic manager for microservices`

## Cryptography

**`Cryptography = the science of keeping information secret, safe, and trustworthy`**

**Main goals**
- Confidentiality → only intended people can read it
- Integrity → data is not tampered
- Authentication → prove identity of sender/receiver
- Non-repudiation → sender cannot deny sending

**Analogy**: Sending a locked box with a secret message inside. Only someone with the correct key can open it.

### Types of cryptography

| Type           | How it Works                                  | Example Use  |
| -------------- | --------------------------------------------- | ------------ |
| **Symmetric**  | Same key to encrypt and decrypt               | AES, DES     |
| **Asymmetric** | Public key encrypts, private key decrypts     | RSA, ECC     |
| **Hashing**    | Converts data into fixed-size digest; one-way | SHA-256, MD5 |

### RSA

**Definition**
- RSA is an asymmetric encryption algorithm.
- Uses a public key (everyone can see) to encrypt, and a private key (kept secret) to decrypt.

```
Sender:   Message → Encrypt with Public Key → Ciphertext → Send
Receiver: Ciphertext → Decrypt with Private Key → Original Message
```

**Analogy**:
- Imagine a mailbox with a public slot (public key).
- Anyone can drop a letter in (encrypt), but only the owner has the key to open the mailbox (private key) and read the letters.

**Key Points**
- RSA is asymmetric: public key ≠ private key
- Usually used to secure small data or encryption keys, not large files. Why?
	1. Computational Cost
		- RSA uses very large numbers (hundreds or thousands of bits) and complex math (modular exponentiation).
		- Encrypting large files directly would be very slow and resource-intensive.
	2. Practical Approach
		- Instead of encrypting the full file, RSA is used to encrypt a small symmetric key (like an AES key).
		- Then the large file is encrypted with AES (fast symmetric crypto).
		- This combines speed of AES with security of RSA.

**Analogy**
- RSA = a tiny, very strong padlock
- AES = a big, fast padlock

***Easy & performant***: Use RSA to secure the tiny key, AES to secure the big file

## Service mesh

**Definition**

> A Service Mesh is a dedicated infrastructure layer for handling service-to-service communication. It uses a sidecar proxy to abstract network complexity (like retries, security, and observability) away from the application code. It's great for complex microservices, but overkill for simple apps.

**The Problem: The "Microservices Headache"**
Imagine you are building a simple e-commerce app. You have a Cart Service and a Payment Service.

- In a Monolith (Old way): The Cart just calls a function payment.process(). It’s instant and reliable because it happens in the same memory space.
- In Microservices (New way): The Cart has to make an HTTP request over a network to the Payment Service.

**Problems**
1. Here is where the headache starts. Networks are unreliable.
2. What if the Payment Service is down? You need code to retry.
3. What if the Payment Service is slow? You need code for a timeout.
4. How do you ensure the data is safe? You need code for encryption (SSL/TLS).
5. How do you know who called whom? You need code for logging.

**The Nightmare**: If you have 50 microservices written in different languages (Java, Python, Go), you have to write this "retry/timeout/security" logic 50 times. If you want to change how retries work, you have to update and redeploy all 50 services.

**The Solution: The "Personal Assistant" (Service Mesh)**

A Service Mesh solves this by saying: "Hey developer, stop writing networking code. Focus on business logic. I will handle the network for you."

It does this using a pattern called the **Sidecar**.

- Imagine every Microservice is a CEO.
- The CEO (Service) shouldn't be dialing numbers, waiting on hold, or checking security badges.
- Instead, every CEO gets a Personal Assistant (Sidecar Proxy) sitting at the desk right next to them.

**The Workflow**
- Cart Service (CEO) wants to talk to Payment Service.
- It doesn't call Payment directly. It whispers to its Assistant (Sidecar): "Send this data to Payment."
- The Assistant handles the hard stuff: It encrypts the data, finds the best route, and retries if the line is busy.
- It talks to Payment's Assistant.
- Payment's Assistant decrypts the data and hands it to the Payment Service (CEO).

The CEOs (your code) think the network is perfect. The Assistants (Service Mesh) do the heavy lifting.

**How it looks**

Here is the flow without a Service Mesh vs. with one.

Without Service Mesh (The "Spaghetti" Way) Your code is bloated with network logic.

```
 [ Cart Service ]
(Code: Business Logic + Retries + Timeout + Encryption + Monitoring)
      |
      |  <-- "Network Call" (Direct)
      v
[ Payment Service ]
```

With Service Mesh (The Clean Way) Your code is clean. The "Proxy" handles the mess.
```
       ----------------- Pod A -----------------
      |                                         |
      |  [ Cart Service ]                       |
      |   (Only Business Logic)                 |
      |          |                              |
      |          v (Localhost - very fast)      |
      |  [ Proxy / Assistant ]                  |  <-- "Data Plane"
      |          | (Envoy)                      |
       ----------|------------------------------
                 |
                 |  <-- Encrypted Network Call (mTLS)
                 |      (Retries & Logging happen here automatically)
                 v
       ----------|------------------------------
      |  [ Proxy / Assistant ]                  |
      |          | (Envoy)                      |
      |          v                              |
      |  [ Payment Service ]                    |
      |   (Only Business Logic)                 |
      |                                         |
       ----------------- Pod B -----------------
```

**The three superpowers (Why use a service mesh?)**

1. ***Traffic Control (Connect)***
	- You can control traffic without changing code.
	- Ex: Canary Release: You launch Payment v2. You tell the mesh (the Assistants): "Send 99% of traffic to v1, and only 1% to v2." If v2 crashes, only 1% of users are affected. The Assistants handle this splitting instantly.
2. ***Security (Secure)***
	- mTLS (Mutual TLS): Usually, setting up encryption (HTTPS) inside a microservice is annoying.
	- The Mesh does it automatically. The Cart Assistant encrypts data; the Payment Assistant decrypts it. The services themselves don't even know it happened. This is critical for Zero Trust security.
3. ***Observability (Observe)***
	- Since every request goes through the Assistants, they track everything.
	- You get a dashboard that shows: "Cart called Payment 500 times. 2 calls failed. Average time was 20ms." You get this for free, without writing a single line of logging code.

**The "Control Plane" vs. "Data Plane"**

1. Data Plane (The Assistants): The proxies (like Envoy) sitting next to services. They do the actual moving of data.
2. Control Plane (The HR Department): The central server (like Istio) that gives instructions to the Assistants.

Example: You (the admin) tell the Control Plane: "Enforce a 5-second timeout on all payments." The Control Plane pushes this rule to all the Assistants (Data Plane).

**The Downside**

Question: If Service Meshes are so great, why doesn't everyone use them?

1. ***Complexity***: You are doubling the number of moving parts. Instead of 50 services, you now have 50 services + 50 proxies + a Control Plane. Debugging can be hard.
2. ***Latency***: Every request now makes two little extra hops (Service -> Proxy -> Network -> Proxy -> Service). It adds a tiny bit of slowness (milliseconds), but for high-frequency trading, this is bad.

### Service mesh vs Load Balancer vs API Gateway

**The Analogy**
- Load Balancer (LB): The Revolving Door.
	- Its only job is to get people (requests) inside evenly so no single door gets jammed. It doesn't care who you are or what you want; it just balances the flow.
- API Gateway: The Front Desk / Receptionist
	- It stops you after the door. It checks your ID (Auth), asks who you are here to see (Routing), and gives you a visitor badge. It handles the "North-South" traffic (people entering the building).
- Service Mesh: The Internal Hallway Monitors / Intercoms.

Once you pass the receptionist, you are inside. Now, how does the Accounting Dept talk to the HR Dept? That is the Service Mesh. It handles the "East-West" traffic (employees talking to employees) to ensure they aren't shouting or leaking secrets.

```
(External User)
            |
            v
+-----------------------+
|    Load Balancer      |  <-- 1. Distributes traffic to available servers
+-----------+-----------+
            |
            v
+-----------------------+
|     API Gateway       |  <-- 2. Checks Auth, Rate Limits, Routes to Service
+-----------+-----------+       (North-South Traffic)
            |
            | (Request enters the internal network)
            v
+-----------------------+             +-----------------------+
|  [ Service A ]        |             |  [ Service B ]        |
|  (Microservice)       |             |  (Microservice)       |
|      +                |             |      +                |
|  [ Sidecar Proxy ]<---|------------>|  [ Sidecar Proxy ]    |
+-----------------------+    (mTLS)   +-----------------------+
            ^
            |
     3. SERVICE MESH
     (Handles retries, security, and logging for internal calls)
     (East-West Traffic)
```

**Does a Service Mesh "live inside" these LB or API gateway tools?**
Short Answer: **No**. They are ***neighbors***, not roommates.

**Detailed Answer**: They are distinct layers, but they are often built using the same underlying technology.

**The Shared Engine**: A popular tool called **Envoy** is a high-performance proxy.
- It is often used inside an API Gateway to route external traffic.
- It is also used as the sidecar in a Service Mesh to route internal traffic.
- So, while a Service Mesh doesn't "live inside" an API Gateway, they might both be "wearing the same uniform" (using Envoy under the hood).

**Exceptions**: Some modern tools try to blur the lines. For example, **Istio** (a service mesh) has an "Ingress Gateway" feature that acts like a basic API Gateway, allowing you to use one tool for both. However, in a strict system design sense, treat them as separate boxes.

## Client Server model

**Client Server model**: A model where *clients request services* and *servers provide them*. It is the foundation of modern web, DB, & application architectures

**Why is it important?**
- Enables ***efficient resource management***: 
	- *Meaning*: Instead of requiring every user's device (the Client) to have a massive hard drive, a powerful CPU, and expensive software licenses, you put all that power into one centralized machine (the Server). The clients can be lightweight and cheap because they are just "remote controls" for the heavy lifting happening on the server.
	- *Analogy*: You don't need to buy a Ferrari for every employee; you just buy a bus.
- Used in:
	1. Web browsing (Browser is client, application server serves responses)
	2. Emails (Email app is client, email server serves emails)
	3. APIs
	4. Databases (SQL client is the client, DB server is the server)
	5. Cloud services

### 3 components of the client server model

1. **Client**: User facing. Sends the requests. Ex: browser, mobile app, API consumer
2. **Server**: System processing requests and returning responses. Ex: Web or DB servers, mail server
3. **Network**: The medium of communication between client and server. Ex: Internet, LAN, WiFi, 5g

### Steps of a client server interaction

(Using the Analogy: Ordering Pizza)
1. **Connect**: You dial the pizza place (Establish connection).
2. **Request**: "I want a large pepperoni" (Send data).
3. **Process**: The kitchen bakes the pizza (Server logic/Database lookup).
4. **Response**: The delivery driver hands you the box (Send data back).
5. **Disconnect**: You hang up the phone -- not practical to wait till delivery but this is just for explanation (Close connection)

```
CLIENT (Browser)                        SERVER (Backend)
     +----------------+                      +-------------------+
     |                |                      |                   |
  1. |  Open Connect  |--------------------->| (Accept Conn)     |
     |                |                      |                   |
  2. |   "GET /home"  |----(Request)-------->|                   |
     |                |                      |  [ 3. Processing ]|
     |                |                      |  [ DB Lookup  ]   |
  4. |                |<---(Response)--------|                   |
     |   Render Page  |      (200 OK)        |                   |
     |                |                      |                   |
  5. |  Close Connect |--------------------->| (Close)           |
     +----------------+                      +-------------------+
```

### 3 types of communication in the client server model

**Type A: Synchronous (Blocking)**
- What it is: The "Telephone Call." You ask a question and you hold the line until you get an answer. You cannot do anything else while waiting.
- Protocol: Standard `HTTP/REST`.
- Use Case: Logging in, searching for a product, loading a webpage.
- Pros/Cons: Simple to build, but if the server is slow, the user freezes.

```
Client: "Here is my password. Is it right?"
   |
   | (Client freezes/waits...)
   |
Server: "Yes, logged in."
Client: "Okay, now I can move on."
```

**Type B: Asynchronous (Non-Blocking)**
- What it is: The "Email" or "Text Message." You send a request and immediately go back to work. You don't wait. The server will ping you later (or you check back later) when it's done.
- Protocol: `Message Queues` (RabbitMQ, Kafka) or `HTTP with callbacks`.
- Use Case: Generating a heavy PDF report, sending a welcome email, video processing.
- Pros/Cons: Great for user experience (no freezing), but harder to build (need to handle "callbacks").

```
Client: "Please generate the yearly report (this takes 10 mins)."
Server: "Received. I'll email you when it's done."
Client: (Immediately shows "Job Started" and lets user browse other pages)
...
(10 mins later)
Server: "Report is ready."
```

**Type C: Bidirectional / Streaming**
- What it is: The "Walkie-Talkie" or "Open Line." The connection stays open permanently. Both sides can shout data at each other whenever they want.
- Protocol: `WebSockets` / `FTP sessions`.
- Use Case: Chat apps (***WhatsApp***), Live Stock Tickers, Multiplayer Games.
- Pros/Cons: Real-time and fast, but keeps a connection open on the server (resource heavy).

```
Client: [Connects via WebSocket]
   |
   | (Connection stays open indefinitely)
   |
Server: "New message from Mom!"
Client: "Typing reply..."
Server: "New message from Dad!"
```

**Note**: The usual protocols used in client-server (typical but not mandatory)
- The Protocol (The Language): `HTTP` (Uses Verbs (GET, POST, PUT, DELETE) and Status Codes (200 OK, 404 Missing, 500 Error))
- The Transport (The Road): Usually `TCP/IP`. This ensures reliability (Analogy: It guarantees the pizza arrives in one piece, not scattered slices)
- The Payload (The Package): Usually `JSON` (JavaScript Object Notation). It's the standard format for data. Example: `{"user": "John", "id": 123}`

### Request response cycle

Request-Response cycle explained as a "Data Retrieval Mission"

The 4-Step Cycle
1. The ***Lookup*** (Where are you?): Before talking, the Client (Browser) needs the Server's IP address. It asks a DNS server: "What is the IP for google.com?" The DNS replies: "It is 142.250.1.1."

2. The ***Handshake*** (Are you listening?): The Client knocks on the Server's door (port 80 or 443) to open a connection. This is the TCP Handshake (SYN, SYN-ACK, ACK). It ensures the server is ready to talk before we send any data.

3. The ***Request*** (The "Ask"): The Client sends a specific command using HTTP. It includes:
	- Method: What I want to do (GET a page, POST a form).
	- Path: Specific file (/index.html).
	- Headers: Metadata (I am using Chrome; I accept JSON).

4. The ***Response*** (The "Answer"): The Server processes the logic (queries the database), then sends back:
	- Status Code: Did it work? (200 OK, 404 Not Found)
	- Body: The actual content (HTML, JSON data, Image)

```
CLIENT (Browser)                               SERVER (Backend)
             |                                             |
   (User types URL)                                        |
             |                                             |
   1. DNS LOOKUP                                           |
   "Where is example.com?" ------------------------------> | (DNS Server)
   <------------------------------- "It is 93.184.216.34"  |
             |                                             |
             |                                             |
   2. TCP HANDSHAKE (Connect)                              |
   "Hello? Can we talk?"  -------------------------------> |
                          <------------------------------- "Yes, I am ready."
   "Great, here I come."  -------------------------------> |
             |                                             |
             |                                             |
   3. HTTP REQUEST                                         |
   "GET /users/123 HTTP/1.1" ----------------------------> |
   (Headers: Accept JSON)                                  | [ PROCESSING ]
                                                           | - Check Auth
                                                           | - Query Database
                                                           | - Format Data
                                                           |
   4. HTTP RESPONSE                                        |
                          <------------------------------- "HTTP/1.1 200 OK"
   (Render Data to User)                                   | (Body: {"id": 123...})
             |                                             |
             v                                             v
```

### Stateless vs stateful servers

**Stateless Architecture**

In a Stateless model, the server does not retain any session information or context about the client between requests. This is a "Shared Nothing" architecture.

Mechanism: Every HTTP request is completely independent. The client must include all necessary context (authentication tokens, user preferences, pagination cursors) in the payload or headers of every single request.

Server Logic: The server processes the request based solely on the information provided in that specific request package. It does not look up local memory variables to see "what this user did last."

State Location: The state is offloaded to the Client (e.g., holding a JWT) or a centralized, external database (e.g., Redis), never on the application server's local RAM.

```
 [ Client ] --(Req 1: Auth Token)---> [ Load Balancer ] ---> [ Server A ]
                                                                  |
                                                            (Validates Token)
                                                                  |
 [ Client ] <--(Resp 1: 200 OK )----- [ Load Balancer ] <---------|


 [ Client ] --(Req 2: Auth Token)---> [ Load Balancer ] ---> [ Server B ]
                                                                  |
                                                            (Validates Token)
                                                                  |
 [ Client ] <--(Resp 2: Data )------- [ Load Balancer ] <---------|
```

**Stateful Architecture**

In a Stateful model, the server retains client context (session data) in its local memory or disk during the lifespan of a session.

Mechanism: The client establishes a session (often via a handshake). The server generates a Session ID and stores the user's data (e.g., is_logged_in=true, cart_items=[...]) in its own RAM. The client only sends the Session ID in subsequent requests.

Server Logic: Upon receiving a request, the server uses the Session ID to look up the user's context in its local memory.

State Location: The state lives on the specific Server Instance that handled the initial login.

```
 [ Client ] --(Login Request)-------> [ Load Balancer ] ---> [ Server A ]
                                                                  |
                                                          (Writes to RAM: "Session_99 = Bob")
                                                                  |
 [ Client ] <--(Set-Cookie: ID_99)--- [ Load Balancer ] <---------|


 [ Client ] --(Req 2: Cookie ID_99)-> [ Load Balancer ] ---> [ Server A ]
                                                                  |
                                                          (Reads RAM: "ID_99 is Bob")
                                                                  |
 [ Client ] <--(Resp 2: Data )------- [ Load Balancer ] <---------|
```

**HTTP Specifics**

HTTP is inherently Stateless. The TCP connection may be kept alive (Keep-Alive), but the HTTP application layer treats each request as an isolated transaction.

***To implement stateful behavior over stateless HTTP, we use Cookies or Headers.***

Stateful Implementation: The server sets a Set-Cookie: session_id=xyz header. The browser automatically appends this cookie to future requests. The server maps session_id to local memory.

Stateless Implementation (Modern Standard): The client stores a JWT (JSON Web Token). This token contains the data itself (payload: `{"user_id": 123, "role": "admin"}`) and is cryptographically signed. The server validates the signature. No lookup in local server memory is required.

**Benefits of stateless architecture**

- ***Horizontal Scaling (Elasticity)***
	- Stateless: Highly elastic. You can spin up 100 new server instances instantly. The Load Balancer can use a simple Round Robin algorithm because any server can handle any request.
	- Stateful: Difficult to scale. You cannot simply direct a user to a new server, because the new server doesn't have that user's session data in RAM. You must implement Sticky Sessions (Session Affinity) at the Load Balancer level, ensuring traffic from IP X always goes to Server A.

- ***Fault Tolerance***
	- Stateless: If Server A crashes, the Load Balancer retries the request against Server B. The user notices nothing.
	- Stateful: If Server A crashes, all session data stored in its RAM is lost. All users connected to Server A are logged out or lose their progress.

**Benefits of stateful architecture**

- Personalization
- Seamless user experiences

However, the drawbacks far outweigh the benefits for a *client-server model that needs to scale and be fault tolerant*

**Does using cookies make the server stateful?**

*Using a cookie does **not automatically** make a server stateful.* A cookie is just a transport mechanism (like an envelope) for moving data from the client to the server.

Whether the system is Stateful or Stateless depends on what you put inside that cookie.

> If the cookie is a **Map** (telling you where to look on the server), it is **Stateful**

> If the cookie is a **Backpack** (carrying the data itself), it is **Stateless**.

Here are the two scenarios you must distinguish in an interview:

***Scenario 1: The "Reference" Cookie (Stateful)***

- Content: The cookie contains only a random ID (e.g., `session_id=99`).
- Server Logic: The server receives ID 99. It must pause and search its local memory (RAM) to figure out: "Who is 99? Oh, it's Bob."
- Verdict: Stateful. The server relies on its own memory to interpret the cookie.

```
 [ Cookie: "ID=99" ]  ----->  [ Server ]
                                |
                             (Looks in RAM: "99 = Bob")
```

***Scenario 2: The "Self-Contained" Cookie (Stateless)***

- Content: The cookie contains the actual data, cryptographically signed (e.g., a JWT: `user=Bob; role=admin; signature=xyz`).
- Server Logic: The server receives the data. It validates the signature mathematically. It does not need to look up anything in memory. It trusts the cookie itself.
- Verdict: Stateless. The server is purely processing the input without needing local history.

```
 [ Cookie: "User=Bob" ]  ----->  [ Server ]
                                  |
                               (Validates Signature: "OK, Hi Bob")
                               (No RAM lookup needed)
```

### Caching helps performance of client server models

Caching improves performance by inserting a high-speed storage layer between the client and the primary data source (origin server), effectively **short-circuiting** the request lifecycle

Caching improves performance by inserting a high-speed storage layer between the client and the primary data source (origin server), effectively short-circuiting the request lifecycle.

The Mechanics: "Hit" vs. "Miss"
- Cache Hit: The client requests data, and the cache already has a copy. The data is returned instantly. Result: Zero interaction with the origin server.
- Cache Miss: The cache does not have the data. The request travels to the origin server, the response is fetched, stored in the cache for future use, and then sent to the client.

**Three Key Performance Gains**
1. ***Reduced Latency*** (Speed): Reading from memory (Cache/RAM) is nanoseconds; reading from a disk/database (Origin) is milliseconds. If the cache is on the client (Browser Cache) or a CDN (Edge Server), the physical distance the data travels is significantly shorter.
2. ***Reduced Server Load*** (Scale): By intercepting read requests, the cache acts as a "shock absorber." This prevents the origin server's CPU and Database from being overwhelmed by repetitive queries (e.g., thousands of users asking for the same "Top 10 Products" list).
3. ***Bandwidth Optimization*** (Cost): Fewer repeated large payloads travel across the network, reducing congestion and egress costs.

```
       [ CLIENT ]
           |
           v
  +------------------+
  |  CACHE LAYER     |  <-- Checks: "Do I have this Data?"
  | (Redis / CDN)    |
  +--------+---------+
           |
           | (Yes: CACHE HIT)
           | --------------------------------> Return Data Instantly (FAST)
           |
           | (No: CACHE MISS)
           v
  +------------------+
  |  ORIGIN SERVER   |  <-- Expensive Operations occur here
  | (App + Database) |      (Logic + Disk I/O)
  +------------------+      (Returns Data & Updates Cache)
```

### How LBs work in a client-server model

In the Client-Server model, a Load Balancer (LB) acts as a "traffic cop" or reverse proxy sitting directly in front of your server fleet.

LB accepts all incoming network traffic from clients and distributes those requests across multiple backend servers using algorithms like *Round Robin (taking turns)* or *Least Connections*. This prevents any single server from becoming a bottleneck, increases reliability (if one server dies, the LB stops sending traffic to it), and allows you to scale horizontally by adding more servers without changing the client's configuration.

```
       [ Client ]       [ Client ]
            \              /
             \            /
           +----------------+
           |  Load Balancer |   <-- Public IP Address
           |    (Nginx)     |
           +--------+-------+
                    |
      /-------------+-------------\
     /              |              \
+----------+   +----------+   +----------+
| Server A |   | Server B |   | Server C |  <-- Private IPs
| (Active) |   | (Active) |   | (Active) |
+----------+   +----------+   +----------+
```

### Scaling a client server model for high traffic scale

Here is the step-by-step evolution of scaling a system, moving from a simple startup to a high-traffic enterprise.

***Phase 1***: Vertical Scaling (Scaling Up)
- The "Bigger Hardware" Approach. You simply upgrade the existing server with a faster CPU, more RAM, and a bigger SSD.
- Pros: Simplest option. No code changes required.
- Cons: Expensive and has a hard physical limit (you can only buy so much RAM). It is a "Single Point of Failure."

```
 [ Client ] ---> [  SERVER (XL)  ] <--- [ Database ]
               (128GB RAM, 64 Cores)
```

***Phase 2***: Horizontal Scaling (Scaling Out)
- The "More Hardware" Approach. Instead of making one server stronger, you add more servers. This is where the Load Balancer (LB) becomes mandatory to distribute the traffic.
- Statelessness: This only works effectively if your servers are Stateless (as discussed earlier).
- Pros: Infinite scaling (just add more boxes).
- Cons: More complex to manage and deploy.

```
                     /--> [ Server A ]
[ Client ] --> [ LB ] --> [ Server B ]
                     \--> [ Server C ]
```

***Phase 3***: Database Scaling (The Bottleneck)
- Adding more application servers is easy, but eventually, your single Database will choke. You scale it in two ways:
- ***Read Replicas (Master-Slave)***: Split the workload.
	- Master DB: Handles all Writes (INSERT, UPDATE).
	- Slave DBs: Handle all Reads (SELECT).
	- Logic: Since most apps read 10x more than they write, this relieves massive pressure.
```
 [ App Server ] ---(Write)---> [ Master DB ]
      |                             |
      |                        (Replicates Data)
      |                             v
      L----------(Read)-----> [ Slave DB ]
```
- ***Sharding (Partitioning)***: Split the data itself.
	- Method: Store users A-M on Database 1 and N-Z on Database 2.
	- Result: No single database holds all the data. It is infinitely scalable but complex to query (joining data across shards is hard).

***Phase 4***: Optimization Layers
- Before adding more servers, reduce the work they have to do.
- ***Caching*** (Redis/Memcached): Sit a cache in front of the Database. If the data is there, don't touch the DB.
- ***CDN*** (Content Delivery Network): Move static files (Images, CSS, JS) to edge servers closer to the user. The main server should strictly handle API logic, not serve JPEGs.
- ***Asynchronous Queues*** (RabbitMQ/Kafka): If a task is slow (e.g., "Send Email" or "Resize Video"), do not do it in the HTTP Request. Push it to a Queue and let a separate "Worker" server handle it later.

**Summary**:
```
 [ Client ]
    |
    v
[ CDN ] (Serves Images)
    |
    v
[ Load Balancer ]
    |
    v
[ App Servers (x100) ] <---> [ Redis Cache ]
    |             \
    |              \---> [ Message Queue ] ---> [ Worker Server ]
    v
[ DB Master ] ---> [ DB Slave ]
```

### Security challenges in client server model

Here are the key security challenges in the Client-Server model:

1. **Man-in-the-Middle (MITM)**: Attackers intercept data traveling across the network; prevent this by strictly enforcing HTTPS/TLS encryption.
2. **SQL Injection**: Attackers send malicious database commands through input fields; prevent this by using parameterized queries instead of raw SQL.
3. **Denial of Service (DDoS)**: Attackers flood the server with junk traffic to crash it; prevent this by using Rate Limiting and CDNs.
4. **Cross-Site Scripting (XSS)**: Attackers inject malicious scripts into web pages to steal cookies; prevent this by escaping/sanitizing all user output.
5. **Broken Authentication**: Attackers steal session IDs to impersonate users; prevent this by using secure, HttpOnly cookies and short-lived tokens.
6. **Insecure Direct Object References (IDOR):** Users access data belonging to others (e.g., changing id=1 to id=2); prevent this with strict access control checks on the server.

## Beyond REST APIs

### Benefits and drawbacks of classic REST APIs

**The "Menu" Approach**

REST is the classic, standard way of building APIs. It treats everything as a "Resource" (like a User, a Product, or an Order) located at a specific URL.

* **How it works**: If you want data, you go to a specific URL (like `api.com/users/1`). If you want the user's posts, you go to a different URL (`api.com/users/1/posts`)
* **The Vibe**: It's like a restaurant menu. You can order a Burger, or Fries, or a Soda. If you want all three, you have to place three orders (or find a specific "Combo" item)

```
Client                                 Server
  |                                      |
  | (1) GET /users/1                     |
  | -----------------------------------> |
  | <--- { "name": "Alice" } ----------- |
  |                                      |
  | (2) GET /users/1/posts               |
  | -----------------------------------> |
  | <--- { "posts": [...] } ------------ |
  |                                      |
```

**Benefits**:
1. **Universal**: Every developer knows it. It works in every browser
2. **Cacheable**: Browsers can easily save (cache) the results of GET `/users/1`
3. **Simple**: Easy to inspect and debug (you can just paste the URL in your browser)

**Drawbacks**:
1. **Over-fetching**: You want just the user's name, but the server sends their age, address, email, and bio too. You waste bandwidth.
2. **Under-fetching**: You want a user and their posts. You have to make two separate requests (as seen in the diagram). This increases the number of Round Trips (time and bandwidth)

**Best Use Case**: 
1. **Public APIs** (like Stripe or Twitter)
	- REST is the standard choice for Public APIs because it prioritizes ***Adoption*** and ***Simplicity*** over raw performance. When you open an API to the world, you want to lower the barrier to entry as much as possible
	- The "Zero Setup" Advantage: No Contracts unlike gRPC, universal Tooling with every major language (Python, JS, Go) having a built-in HTTP client
	- HTTP Caching (Free Performance)
	- Discoverability (HATEOAS): A well-designed REST API helps developers explore it. The response itself can contain links to other actions. Example: You fetch a user, and the JSON includes a links section: `"orders": "/users/1/orders"`. This allows developers to "crawl" your API without constantly reading documentation
2. **Simple applications**, or 
3. When you need **good caching**

**When NOT to use**: When you have **complex data with many connected parts** (e.g., User -> Friends -> Posts -> Comments) AND you need to **load it all at once**

### GraphQL

GraphQL solves the "over-fetching" problem, but it introduces a new set of headaches

**GraphQL: The "Personal Shopper" for Data**

GraphQL is a query language for APIs that allows the client to dictate exactly what data the server should send back.

#### Benefits

* **Single Endpoint**: Instead of hitting multiple REST URLs (/users, /posts), clients send all requests to just one endpoint (e.g., /graphql)
* **Client-Driven Queries**: The client sends a specific "shopping list" of fields it needs
* **Solves Over-fetching**: You get only the data you asked for, not massive JSON blobs with info you don't need
* **Solves Under-fetching** (One Round-Trip): You can retrieve nested, related data (e.g., a User and their latest Posts) in a single network request, saving round-trips

#### How it works

```
Client (e.g., Mobile App)                 Server (Single Endpoint)
         |                                        |
         | (1) The "Shopping List" Query:         |
         | {                                      |
         |   user(id: 5) { name }                 | <--- "I only want the name."
         |   posts(limit: 1) { title }            | <--- "And just one post title."
         | }                                      |
         | -------------------------------------> |
         |                                        |
         |                                        | (Server executes resolvers
         |                                        |  to fetch precisely this data)
         | (2) The Exact Result:                  |
         | {                                      |
         |   "user": { "name": "Bob" },           |
         |   "posts": [ { "title": "Hello" } ]    |
         | }                                      |
         | <------------------------------------- |
         | (One round-trip, exact shape match)    |
```

#### Core Components

1. **Schema (The Contract)**: The blueprint of your API. It defines exactly what types exist (User, Product), their fields, and the relationships between them. It is strictly typed.
2. **Query & Mutation (The Request)**:
	- Query: Used to read data (equivalent to GET)
	- Mutation: Used to write/modify data (equivalent to `POST/PUT/DELETE`)
3. **Subscription**: Used for real-time updates (over WebSockets)
4. **Resolvers (The Logic)**: Functions that connect the Schema to the actual data. Every single field in your schema has a resolver function responsible for fetching just that piece of data (from a DB, another API, or a file)

#### Data Flow

5. **Receive**: The Server receives a query string from the Client
6. **Validate**: The Server checks the query against the Schema to ensure it is valid (e.g., fields exist, types are correct) i.e ***parse and validate***
7. **Execute**: The Server walks through the query field by field
8. **Resolve**: ***For each field, the Server calls the corresponding Resolver***
9. **Response**: The Server collects all the results from the resolvers, assembles them into a JSON object matching the query shape, and sends it back

#### Responsibilities

* **Client**: Responsible for constructing the query and specifying exactly what data is needed
* **Server**: Responsible for exposing the capabilities (Schema) and figuring out how to get that data (Resolvers). It acts as an ***orchestration layer***, often ***aggregating data from multiple microservices or databases***

```
CLIENT QUERY                 SERVER (The Engine)                 DATA SOURCES
   |                                 |                                |
   | { user(id:1) { name } }         |                                |
   | ------------------------------> | (1) Parse & Validate           |
   |                                 |                                |
   |                                 | (2) Call Resolver: User(1)     |
   |                                 | -----------------------------> | SELECT * FROM Users...
   |                                 | <----------------------------- | Returns { id: 1, name: "Bob" }
   |                                 |                                |
   | <------------------------------ | (3) Return JSON                |
   | { "data": { "user": ... } }     |                                |
```

#### Popular tools
1. Apollo
2. Relay
3. GraphiQL / GraphQL Playground
4. Hasura (An "Instant GraphQL" engine. You point it at a Postgres database, and it automatically generates a full GraphQL API for you instantly)

#### Drawbacks of GraphQL

1. **Complexity**: It is harder to set up on the backend
2. **No Caching**: Since every request is unique and uses `POST`, standard HTTP caching doesn't work well
	- Caching is hard because GraphQL typically uses a single URL endpoint (like /graphql) for every single request, meaning standard tools like Browsers and CDNs—which rely on unique URLs to identify data—cannot tell the difference between a request for "User A" and a request for "User B.
```
	REST (Easy Caching)                  GraphQL (Hard Caching)
   (Unique URLs = Unique Keys)           (Same URL = Ambiguous Keys)

1. GET /api/user/1  --> [ Cache ]      1. POST /graphql  --> [ ? ]
                                          (Body: { user(id:1) })

2. GET /api/book/9  --> [ Cache ]      2. POST /graphql  --> [ ? ]
                                          (Body: { book(id:9) })
                                                  ^
                                     The CDN only sees "/graphql" for both.
                                     It thinks they are the same request!
```
3. **The "N+1 Problem" (Performance Killer)**: This is the most famous GraphQL issue. Because GraphQL resolves fields independently, a nested query can accidentally trigger thousands of database calls
	- The Scenario: You request a list of 10 Authors and their Books
	- The Execution: The server runs 1 query to find the 10 Authors. For each of the 10 authors, the server runs a separate query to find their books
	- The Math: 1 initial query + 10 follow-up queries = 11 queries. If you have 1,000 authors, that's 1,001 database calls for a single HTTP request

**REST approach**:
In a typical REST API, endpoints are "opinionated." The backend developer decides exactly what data an endpoint returns.

To get authors and their books, the developer usually creates a specific endpoint (or uses query parameters) that anticipates this need. Because the backend knows exactly what is needed beforehand, it can fetch all the data in a single, optimized SQL database query using a `JOIN`

```
 [ CLIENT ]
    |
    | 1. HTTP GET REQUEST
    |    URL: /authors
    |    Params: ?ids=101,102,103&include=books  <-- Explicit List
    |
    v
+--------------------------------------+
| REST API SERVER (Backend Controller) |
+--------------------------------------+
    |
    | 2. Controller reads "101,102,103".
    |    It executes ONE optimized SQL query:
    |
    |    "SELECT * FROM authors
    |     JOIN books ON authors.id = books.author_id
    |     WHERE authors.id IN (101, 102, 103)"  <-- Handles all IDs at once
    v
+--------------------------------------+
| DATABASE                             |
+--------------------------------------+
    |
    | 3. Returns combined data for 101, 102, and 103 instantly.
    v
[ CLIENT ]
```

**GraphQL approach**
```
 [ CLIENT ]
    |
    | 1. POST PAYLOAD (The Query)
    |    {
    |      "query": "
    |        query {
    |          authors(ids: [101, 102, 103]) {  <-- The Params
    |            name
    |            books {                        <-- The Nested Request
    |              title
    |            }
    |          }
    |        }"
    |    }
    v
+-------------------------------------------+
| GRAPHQL SERVER (Resolver Execution)       |
+-------------------------------------------+
    |
    | 2. RESOLVER: Query.authors(ids: [101, 102, 103])
    |    Action: Fetch the list of authors first.
    |    DB Query: SELECT * FROM authors WHERE id IN (101, 102, 103);
    |    Result: [ Author(101), Author(102), Author(103) ]
    |
    | 3. RESOLVER: Author.books (MUST RUN FOR EACH RESULT)
    |
    |    > Processing Author(101)...
    |      DB Query: SELECT * FROM books WHERE author_id = 101;
    |
    |    > Processing Author(102)...
    |      DB Query: SELECT * FROM books WHERE author_id = 102;
    |
    |    > Processing Author(103)...
    |      DB Query: SELECT * FROM books WHERE author_id = 103;
    v
+----------+
| DATABASE |  <-- Hit 4 times total (1 for list + 3 for books)
+----------+
```

**Solution**: Use a **DataLoader**

It "batches" these requests. It waits a few milliseconds, collects all the Author IDs (1, 2, 3...), and runs one query: `SELECT * FROM books WHERE author_id IN (1, 2, 3...)`.

```
WITHOUT DATALOADER (N+1)          WITH DATALOADER (Batching)
(Inefficient serial calls)        (Smart grouped call)

Resolver A  Resolver B            Resolvers A, B, C...
    |           |                     |  |  |
    v           v                     v  v  v
   DB          DB                  +-------------+
Query(1)    Query(2)               | DataLoader  | (Wait & Collect)
                                   +-------------+
                                          |
                                          v
                                   One DB Query IN (1,2,3)
```

"***Batch and wait***": DataLoader acts like a smart waiter in a restaurant. Instead of running to the kitchen (Database) every time one person orders a drink, the waiter waits a few milliseconds for the whole table to finish ordering. Then, they take all orders at once to the kitchen

In technical terms, DataLoader uses the **Event Loop**. It collects all the individual requests that happen within a single frame of execution (a ***"tick"***) and coalesces them into one bulk query

Notice the *new layer in the middle*. ***The resolvers no longer talk to the Database directly; they talk to the DataLoader.***

```
 [ CLIENT ]
    |
    | 1. POST PAYLOAD (Same as before)
    |    { query: "authors(ids: [101, 102, 103]) { books { ... } }" }
    |
    v
+-------------------------------------------+
| GRAPHQL SERVER                            |
+-------------------------------------------+
    |
    | 2. RESOLVER: Query.authors
    |    DB Query: SELECT * FROM authors ...
    |    Result: [ Author(101), Author(102), Author(103) ]
    |
    | 3. RESOLVER: Author.books (The Loop runs, but...)
    |
    |    > Processing 101: Calls DataLoader.load(101)  --|
    |    > Processing 102: Calls DataLoader.load(102)  --|-> (Held in Queue)
    |    > Processing 103: Calls DataLoader.load(103)  --|
    |
    |             ( End of Execution Tick )
    |             ( DataLoader wakes up )
    v
+-------------------------------------------+
| DATALOADER (The Batcher)                  |
+-------------------------------------------+
    |
    | 4. "I received 3 IDs: [101, 102, 103].
    |     I will fetch them all now."
    |
    |    Batch DB Query:
    |    SELECT * FROM books WHERE author_id IN (101, 102, 103);
    v
+----------+
| DATABASE |  <-- Hit ONLY 1 time for all books
+----------+
    |
    | 5. DataLoader maps the results back to the Resolvers:
    |    [Books for 101] -> Author 101
    |    [Books for 102] -> Author 102
    |    [Books for 103] -> Author 103
    v
[ CLIENT ]
```

**Dataloader working**

Its primary job is Batching: It waits for your application to ask for several pieces of data, groups them into a single list, and fetches them all at once

***How It Works (The 3 Steps)***
1. **Coalesce (Collect):** When your code asks for a database record (e.g., "Get User 1"), DataLoader doesn't run the query immediately. It creates a `Promise` and waits for a very short time (usually one "tick" of the event loop, just a few milliseconds)
2. **Batch**: During that tiny wait, if other parts of your code ask for "User 2" and "User 3", DataLoader adds them to the list
3. **Dispatch**: Once the wait is over, DataLoader sends one single query to the database for all collected IDs

Resolvers without a dataloader:
```js
// Request: Get 3 Posts and their Authors
// Result: 4 DB Calls (1 for posts, 3 for authors)

db.getPosts(); // SELECT * FROM posts LIMIT 3
// ... wait ...
db.getAuthor(1); // SELECT * FROM authors WHERE id = 1
db.getAuthor(2); // SELECT * FROM authors WHERE id = 2
db.getAuthor(3); // SELECT * FROM authors WHERE id = 3
```

Resolvers with a dataloder:
```js
// Request: Get 3 Posts and their Authors
// Result: 2 DB Calls (1 for posts, 1 for ALL authors)

const authorLoader = new DataLoader(ids => {
  // The 'ids' array will be [1, 2, 3]
  return db.query(`SELECT * FROM authors WHERE id IN (${ids})`);
});

db.getPosts(); 
// ... wait ...
// These three calls happen "simultaneously" in code:
authorLoader.load(1);
authorLoader.load(2);
authorLoader.load(3);

// DataLoader notices they happened in the same "tick".
// It stops them, groups the IDs, and runs ONLY ONE query:
// SELECT * FROM authors WHERE id IN (1, 2, 3)
```

#### GraphQL use cases

Best Use Case: 
1. **Mobile apps (where bandwidth is expensive)**, or 
2. **complex front-ends (like Facebook's News Feed)**

**When NOT to use**:
- ***Simple apps*** where REST is enough, or 
- If you need ***simple, standard caching***

#### Common System Design Interview Questions

> **Q1: "We are designing a news feed. Why would you choose GraphQL over REST?"**

Bad Answer: "Because it's newer and Facebook uses it."

Good Answer: "Because a news feed has complex, nested data (User -> Post -> Comments -> Author). GraphQL allows us to fetch this deep hierarchy in a single network round-trip, which significantly reduces latency on mobile networks compared to making 5-6 sequential REST calls."

> **Q2: "How do you prevent a malicious user from taking down your GraphQL server with a recursive query?"**

Key Terms to use: "I would implement **Depth Limiting** (rejecting queries deeper than X levels) and Query Cost Analysis (rejecting queries that are too expensive)."

**Depth limiting** is a security validation rule that rejects any GraphQL query where the fields are nested beyond a specific allowed level (e.g., a maximum depth of 3), effectively preventing malicious users from crashing the server with recursive queries (cyclic relationships like `author { books { author { books... } } })` that would otherwise consume infinite resources

```
      QUERY ROOT
          |
   1.   author {              (Depth 1: OK)
          |
   2.     books {             (Depth 2: OK)
            |
   3.       similarBooks {    (Depth 3: MAX LIMIT REACHED)
              |
   4.         author { ... }  <-- REJECTED!
                                  (Error: Query is too deep.)
```
Code example (Apollo):
```js
import depthLimit from 'graphql-depth-limit';

const server = new ApolloServer({
  schema,
  validationRules: [
    // The library does the AST traversal for you
    depthLimit(3)
  ]
});
```

> **Q3: "We need to cache user profiles. How do we do that in GraphQL?"**

Key Terms to use: "Since we can't use standard HTTP caching easily, I would use Persisted Queries. The client sends a hash ID (e.g., query_id: 123) instead of the full query string. The server maps 123 to the query and uses standard Redis caching keys based on variables."

> **Q4: "What is the N+1 problem, and how would you solve it in this specific DB schema?"**

Key Terms to use: "This happens when we fetch a list and then fetch related data for each item individually. I would use the DataLoader pattern to coalesce the IDs and fetch the related records in a single batch SQL query using an IN clause."

### gRPC

**The "Direct Line" Approach**

gRPC is a modern, high-performance framework created by Google. It doesn't send text (JSON) like REST or GraphQL; it ***sends Binary (0s and 1s) using a format called Protocol Buffers (Protobuf)***

- **How it works**: It feels like calling a function that runs on a different computer. You define a strict "Contract" (file) beforehand
- **The Vibe**: It's like a highly optimized military walkie-talkie. It’s not English text; it’s a compressed, coded burst of sound that is incredibly fast but requires special equipment to understand

```
Client                                 Server
  |                                      |
  | (1) GetUser(1) [Binary Data]         |
  | -----------------------------------> |
  | (Extremely fast, compacted 0s & 1s)  |
  |                                      |
  | <--- [User Binary Data] ------------ |
  |                                      |
```

#### Benefits

1. **Speed**: It is much faster and smaller than JSON.
2. **Streaming**: It supports two-way streaming (Server and Client talking at the same time)
3. **Strict Contracts**: You cannot make mistakes with data types; the code won't compile if you do

#### Drawbacks 

1. **Browser Support**: Browsers don't speak gRPC natively. You need a special proxy (gRPC-Web) to use it on a website
2. **Hard to Debug**: You can't just read the data; it looks like gibberish binary code unless you have the decoder tool

#### Use cases of gRPC

*Best Use Case*: **Internal Microservices**. When Server A talks to Server B in a massive system (like Netflix or Uber), *speed is everything*.

*When **NOT** to use*: **Public APIs** for outside developers, or **simple websites where browser compatibility is key**

| Stick with REST if...	| Switch to gRPC if... |
| -- | -- |
| You are a startup / small team |	You are "Hyperscale" (Netflix, Uber) |
| Your team is mostly web developers |	Your team handles massive concurrency |
| **Time-to-market** is #1 priority |	**Latency** is #1 priority |
| Your bottleneck is the **Database** |	Your bottleneck is **Network/CPU serialization** |

```
THE ARGUMENT FOR gRPC
      +-------------------+
      |  High Performance |  <-- (Visible Benefit)
      +-------------------+
            |
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  (Water Line)
            |
      THE REALITY (Hidden Costs)
      +-----------------------------+
      |  Complex Load Balancing     | (Need Envoy/Istio)
      +-----------------------------+
      |  Harder Debugging           | (Binary blobs)
      +-----------------------------+
      |  Proto File Management      | (Version conflict)
      +-----------------------------+
      |  Steep Learning Curve       | (HTTP/2, Streams)
      +-----------------------------+
```

It is important to carefully analyze our system and make a choice on the Network protocols to use

#### gRPC working in detail

The Core Concept: **"Code Generation"**

In REST, you manually write code to send an HTTP request (using fetch or axios). In gRPC, you don't write the networking code. A tool writes it for you.

Here is the specific 4-step workflow:

**Step 1**: **The Contract (`.proto` file)**

You start by writing a file that defines the data structure and the function names. This file is language-neutral.

*Protocol Buffers*
```bash
// user.proto
message UserRequest {
  int32 id = 1;
}

message UserResponse {
  string name = 1;
  string email = 2;
}

service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}
```

**Step 2**: **The Code Generation (The "Compiler")**

You run a special tool called **`protoc`**. It reads your `.proto` file and auto-generates code in whatever language you are using (Python, Java, Go, C++).

- *Client Side*: It generates a "Stub". This is a dummy object. When you call a function on it, it secretly sends a network request.
- *Server Side*: It generates a "Base Class". You just fill in the blanks with your actual logic.

**Step 3**: **The Transmission (Serialization)**

When the Client calls `GetUser(1)`, the Stub takes that `1`, crunches it down into binary (`Protobuf`), and shoots it over the network. It does not send the text` "id: 1".`. ***It sends a tiny byte sequence***

**Step 4**: **The Transport (HTTP/2)**

The data travels over `HTTP/2`. This is a **major upgrade** over the `HTTP/1.1` used by standard REST

Benefits of using http/2 for gRPC:
- **Multiplexing**: You can have 1,000 different calls happening at the same time over one single TCP connection
- **Streaming**: The server can start sending data back before the client finishes asking

#### Diagrams for easy understanding

**Diagram 1: The Workflow (From File to Code)**: This shows how one file allows a Python Client to talk to a Go Server effortlessly
```
        [ The "Contract" (.proto file) ]
     ( Defines: "GetUser" takes ID, returns Name )
                   |
        +----------+----------+
        |  (Run protoc tool)  |
        v                     v
 [Python Code Gen]       [Go Code Gen]
 (Auto-creates Stub)     (Auto-creates Base)
        |                     |
        v                     v
  +-------------+       +-------------+
  | Python App  |       |  Go Server  |
  | (Client)    |       |  (Backend)  |
  +-------------+       +-------------+
        | call GetUser(5)     ^
        |                     |
        +----(Network)--------+
```

**Diagram 2: The Wire (Why it's faster)**: This illustrates the difference between sending JSON (Text) and Protobuf (Binary). The Scenario: Sending a user object `{ "id": 1024, "active": true }`
```
REST (JSON)                          gRPC (Protobuf)
      "The Heavy Box"                      "The Flatpack"

Data on Wire:                        Data on Wire:
{                                    [08 80 08 10 01]
  "id": 1024,                        
  "active": true                     (That's it. 5 bytes.)
}                                    
                                     
(Sent as characters:                 (Sent as raw binary values.
 "i", "d", ":", space...)            The schema knows that Field 1
                                     is ID and Field 2 is Boolean)
                                     
SIZE: ~30 Bytes                      SIZE: ~5 Bytes
SPEED: Slow (Parse text)             SPEED: Instant (Read bytes)
```

**Diagram 3: HTTP/2 Multiplexing**: Unlike `HTTP/1`, which blocks the line, `HTTP/2` allows multiple calls to "interweave" on the same connection (Note: This illustration is for a ***single TCP connection***)
```
HTTP/1.1 (REST)                   HTTP/2 (gRPC)
   (One at a time)                   (Interleaved / Multiplexed)

   [Req 1] ->                        [Req 1 Part A] ->
      (Wait...)                      [Req 2 Part A] ->
   <- [Resp 1]                       [Req 1 Part B] ->
                                     <- [Resp 2 Part A]
   [Req 2] ->                        <- [Resp 1 Part A]
      (Wait...)
   <- [Resp 2]                       (One Connection, Heavy Traffic)
```

#### Protocol buffers explained

Protobuf is Google's mechanism for serializing structured data. Think of it as JSON's highly efficient, strict younger brother. Instead of sending easy-to-read text (like JSON or XML), Protobuf crushes data into a tiny, dense binary format. It relies on a pre-defined Schema (`.proto` file) that acts as a contract between the client and server, so they both know exactly what the data looks like without needing to send field names (like "firstName" or "email") over the network every single time.

Key Points:
1. **Binary & Compact**: It removes all whitespace, brackets, and field names, reducing payload size by 30-50%
2. **Language Agnostic**: You define the data once in a .proto file, and a compiler auto-generates code for Python, Java, Go, C++, etc
3. **Faster**: Computers can parse binary numbers significantly faster than scanning and parsing text strings

```
[.proto File (The Key)]             [.proto File (The Key)]
    (Field 1 = Name, Field 2 = Age)     (Field 1 = Name, Field 2 = Age)
              |                                   |
              v                                   v
      +---------------+                   +---------------+
      | SENDER (App)  |                   | RECEIVER (App)|
      +---------------+                   +---------------+
              |                                   ^
              | (Encodes "Alice", 25)             | (Decodes)
              v                                   |
      [ Wire Data:  08 05 12 05 41 6c 69 63 65 ] -+
      (Looks like gibberish to humans, but it's tiny!)
```
```
            JSON (Verbose)                       PROTOBUF (Efficient)
      "Hello, I am a descriptive text"     "The Coded Message"

      {                                    (Field 1) (Length 3) (Value)
        "name": "Bob",                        08        03       "Bob"
        "id": 55                           (Field 2) (Value 55)
      }                                       10        37

      Payload: ~25 Bytes                   Payload: ~7 Bytes
      (Wastes space on quotes/brackets)    (Pure Data + Field IDs)
```

**Field numbers explained**

The "Numbered Box" Analogy: Think of your data like a row of numbered mailboxes:
- REST (JSON) puts a sticky note on the box: "This is the Name box."
- gRPC (Protobuf) just paints a number on the box: "Box #1."

Because computers read numbers faster than sticky notes, gRPC is faster.

How Updates Work (Compatibility)
- Adding a Field (Safe):
	- New Server: "I have a new Box #3 for Email."
	- Old Client: "I only look at Box #1 and Box #2."
	- Result: The Old Client just ignores Box #3. Everything works
- Deleting a Field (Safe-ish):
	- New Server: "I removed Box #2 (Age)
	- Old Client: "Here is data for Box #2."
	- Result: The Server receives the box but throws it away because it doesn't use it anymore. Everything works
- **Reusing a Number (FATAL ERROR)**:
	- You: "I deleted Box #2 (Age). Now I will make Box #2 store 'Zip Code'."
	- Old Client: Sends "25" (Age) in Box #2
	- New Server: Reads "25" as a Zip Code
	- ***Result: Data Corruption***. You think the user lives in Zip Code 00025.

**The One Rule to Remember**: Once you assign a number (like age = 2), that number belongs to "Age" forever. Even if you delete the field, you must ***retire*** the number 2 like a sports jersey so nobody else wears it

```
     CLIENT (Old App)                     SERVER (New App)
     Knows: 1=Name                        Knows: 1=Name, 2=Email
     [ Box 1: "Alice" ]  ------------->   [ Reads Box 1: "Alice" ] (OK!)
                                          [ Reads Box 2: Empty ]   (OK!)

     SERVER (New App)                     CLIENT (Old App)
     [ Box 1: "Bob" ]    ------------->   [ Reads Box 1: "Bob" ]   (OK!)
     [ Box 2: "hi@me" ]  ------------->   [ Box 2? I don't know it.] (Ignores it)
```
