<!-- TOC --><a name="networking-concepts-and-basics"></a>
# Networking Concepts and Large Scale Networks Basics


<!-- TOC --><a name="networking-layers"></a>
## Networking layers

<!-- TOC --><a name="osi-model"></a>
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

<!-- TOC --><a name="tcpip-model"></a>
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

<!-- TOC --><a name="transport-layer-protocols"></a>
## Transport layer protocols

Transport layer protocols provide end-to-end communication between processes, handling reliability, ordering, flow control, and congestion (e.g., TCP) or fast best-effort delivery (e.g., UDP).

> They move data between processes on two hosts, deciding ***how reliable*** and ***how fast*** the delivery should be.

<!-- TOC --><a name="tcp-transmission-control-protocol"></a>
### TCP — Transmission Control Protocol

Core idea: **Reliable, ordered, congestion-aware communication**

Use TCP when:
1. You cannot lose data
2. Order matters

Example use cases (TCP at the transport layer will support the below application later protocols):
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

**TCP 3-way handshake**: **Opening a connection**
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

Why they must be different
1. TCP is full-duplex (data flows independently in both directions)
2. Each direction needs its own sequence space for ordering, retransmissions, and loss detection
3. Prevents confusion between old and new connections (especially after crashes)

> A 2-way handshake can’t confirm bidirectional readiness or protect against delayed/duplicate packets, leading to half-open or bogus connections. The third ACK is what proves “I heard you hear me."

**TCP 4-way handshake**: **Terminating a connection**

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

Why termination needs 4 steps:
* TCP is full-duplex — each direction closes independently
* One side may finish sending earlier than the other
* Clean shutdown without data loss

> TCP uses a 4-way handshake for teardown because each side must independently close its send channel.

<!-- TOC --><a name="head-of-line-hol-blocking-tcp-level"></a>
#### Head-of-Line (HOL) Blocking (TCP-level)

*Problem*: TCP delivers data in **order**.

If packet #2 is lost:
```
Packet 1 → OK
Packet 2 → LOST ❌
Packet 3 → WAITING ❌ (even if received)
```

So *everything waits.*

This hurts:
1. HTTP/1.1
2. Mobile / lossy networks

The fix for H.O.L blocking: You can’t fix Head-of-Line (HoL) blocking inside TCP — it’s a fundamental property of in-order, reliable delivery. Alternatives:
* Use multiple TCP connections: Parallel connections reduce the blast radius of one lost packet (HTTP/1.1 browsers used this trick)
* Application-level multiplexing: Move stream management above TCP so one stalled stream doesn’t block others (HTTP/2 tried this, but still suffers from TCP-level HoL)
* Use QUIC (UDP-based) ✅ real solution: Runs over UDP, has multiple independent streams, packet loss in one stream doesn’t block others (HTTP/3)
* Reduce packet loss (mitigation, not cure)

**TCP/IP (Quick clarity)**
* IP → Addressing + routing
* TCP → Reliability + order

IP doesn’t care if packets arrive. TCP does.

**Questions on TCP**:
Q: Why is TCP slower than UDP?
A: TCP adds reliability, ordering, congestion control, and handshake overhead.

Q: Why does HTTP/2 still use TCP?
A: To reuse existing infrastructure and reliability, while fixing HOL at application level.

**Enterprise Tools using TCP**
1. Nginx
2. Envoy
3. HAProxy
4. Databases (Postgres, MySQL)
5. REST APIs

<!-- TOC --><a name="user-datagram-protocol"></a>
### User Datagram Protocol

Core idea: **Fast, connectionless, best-effort delivery**

No:
- Handshake
- Retries
- Ordering

Just fire-and-forget.

**UDP characteristics**:
- ❌ No guarantee of delivery
- ❌ No order
- ❌ No congestion control
- ✅ Very low latency

**Where UDP is used**:
| Use Case	| Why UDP |
| -- | -- |
| Video streaming |	Losing a frame is okay |
| Online gaming	| Latency > reliability |
| VoIP	| Real-time |
| DNS	| Small, fast |
| QUIC / HTTP/3	| Custom reliability |

**UDP flow**:
```
Client ---> Packet ---> Server
Client ---> Packet ---> Server
(no ACKs)
```

**Examples of UDP usage**:
1. QUIC (used in HTTP/3)
2. WebRTC (Cloudflare, Google)
3. DNS servers
4. Media servers

> “UDP pushes complexity to the application layer.”

**UDP pros**:
* Very fast
* No handshake
* Great for real-time

**UDP cons**:
* Unreliable
* Harder to implement correctly
* Firewalls sometimes block UDP

**Questions**:

> Q: Why not use UDP everywhere?
A: Most applications need reliability and ordering, which UDP doesn’t provide.

> Q: Why is QUIC built on UDP?
A: UDP allows "custom reliability" without TCP’s HOL blocking.

**TCP vs UDP comparison**:
| Feature     | TCP      | UDP           |
| ----------- | -------- | ------------- |
| Reliability | Yes      | No            |
| Order       | Yes      | No            |
| Latency     | Higher   | Lower         |
| Handshake   | Yes      | No            |
| Use cases   | APIs, DB | Video, gaming |


**Head-of-Line (HoL) Blocking: HTTP/2 vs HTTP/3**

- HTTP/2:
	- Multiplexes many streams over a single TCP connection
	- Fixes application-layer HoL (one slow request doesn’t block others at HTTP level)
	- ❌ Still suffers from TCP HoL
	- TCP enforces in-order delivery

> One lost packet blocks all streams behind it

Result: Loss in one stream pauses every stream

- HTTP/3
	- Runs on QUIC (over UDP)
	- Each stream has independent ordering and retransmission
	- ❌ No TCP → no transport-layer HoL

Result: Loss in one stream blocks only that stream

**"Streams" explained**:
> A stream is a logical, independent, bidirectional sequence of bytes representing one request–response pair, multiplexed over a single connection.

Key points:
- Multiple streams share one connection
- Each stream has its own ID and ordering
- Streams are interleaved, not sent end-to-end

Think of streams like ***multiple conversations happening over one phone call***.

**H.O.L blocking in TCP connections**:
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

**No H.O.L blocking in UDP connection**:
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

<!-- TOC --><a name="transport-layer-security"></a>
## Transport Layer Security

TLS stands for Transport Layer Security

<!-- TOC --><a name="understanding-tls"></a>
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

TLS handshake steps:
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

Important detail (but simple)
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

**What is a TLS certificate or cert?**:

> A TLS certificate ***proves*** the server’s identity and ***provides the public key*** needed to establish a secure encrypted session.

What a TLS Certificate Is: A TLS certificate is a digital identity card for a server (or sometimes a client) that proves it is who it claims to be.

Key points:
1. Authenticates the server – Ensures you are really talking to the legitimate website/server.
2. Contains a public key – Used by the client to set up encryption for the session.
3. Signed by a trusted Certificate Authority (CA) – The CA vouches that the server’s identity is valid.
4. Optional info – Domain name, organization, expiration date, etc.

Analogy:
- Think of it like a passport for websites:
- Shows your identity (domain)
- Verified by a trusted authority (CA)
- Lets others communicate securely with you

If invalid:🚨 “Your connection is not private”

**Trusted authority**:

A trusted authority (CA) is an entity that *verifies and vouches for a server’s identity*, allowing clients to trust its TLS certificate.

Here are some well-known **Certificate Authorities (CAs)**:
- GlobalSign
- DigiCert (includes Symantec, Thawte, GeoTrust)
- Let's Encrypt (free, automated)
- Comodo / Sectigo
- Entrust

These are entities browsers and operating systems trust to verify website identities

**TLS 1.2 vs TLS 1.3**
| Feature       | TLS 1.2 | TLS 1.3 |
| ------------- | ------- | ------- |
| Handshake RTT | 2       | 1       |
| Security      | Good    | Better  |
| Legacy crypto | Yes     | Removed |
| Performance   | Slower  | Faster  |

> “TLS 1.3 reduces handshake latency by one round trip.”

**Drawbacks of TLS**:
- TLS adds:
	- Handshake latency
	- CPU cost (encryption)
- Bad for:
	- Mobile
	- High-QPS APIs

**TLS optimizations**

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

**Problem with the "O-RTT" optimisation**:

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

**TLS Pros & Cons**

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

**TLS termination**

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

<!-- TOC --><a name="application-layer-protocols"></a>
## Application layer protocols

<!-- TOC --><a name="hypertext-transfer-protocol"></a>
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

Characteristics:
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

Problems:
1. Sequential; slow for many small resources
2. Multiple TCP connections needed → overhead

<!-- TOC --><a name="http-2"></a>
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

<!-- TOC --><a name="hol-differences-between-http-1-and-http-2"></a>
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

<!-- TOC --><a name="http-3"></a>
#### HTTP 3

Analogy: Multi-lane highway with independent lanes, each lane has its own toll booth and traffic light. Packet loss in one lane doesn’t block others.

Characteristics:
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

<!-- TOC --><a name="domain-sharding"></a>
#### Domain sharding

Domain Sharding is a ***concept, not a protocol***: 
- Splitting requests across multiple subdomains to increase parallel downloads.
- Example: `assets1.example.com`, `assets2.example.com`
- Old hack from HTTP/1.1 era because only ~6 connections per domain.

Drawback today:
- With HTTP/2 multiplexing → domain sharding can hurt performance.

**Note**: Do not domain shard in today's day and age (HTTP 2+ world)

<!-- TOC --><a name="https"></a>
#### HTTPS

TLS + HTTP interplay

- ***Browsers force HTTP/2 over TLS (except local or special cases).***
- `TLS handshake + TCP handshake cost → mitigated by connection reuse, HSTS, 0-RTT.`

Clean explanation:
1. HTTP is an application protocol that defines how clients and servers request and send data.
2. TLS is a security protocol that encrypts and authenticates communication.
3. HTTPS is simply HTTP running over TLS, providing secure, encrypted communication.

In one line: **`HTTPS = HTTP + TLS (secure HTTP)`**

✅ Pros of HTTPS
- Security: Encrypts data, preventing eavesdropping and tampering
- Authentication: Verifies server identity via certificates
- Trust & SEO: Required by browsers; improves user trust and search ranking
- Modern features: Enables HTTP/2, HTTP/3, service workers, etc.

❌ Cons of HTTPS
- Slight performance overhead: TLS handshake and encryption (mostly negligible today)
- Operational complexity: Certificate issuance, renewal, and management
- TLS termination concerns: Encryption often ends at CDN/LB, not always end-to-end

**HTTPS: Mandatory vs Optional** 

Mandatory when:
- Login, payments, personal or sensitive data
- Authentication (cookies, tokens)
- Public internet or modern browser features

Optional (rare cases):
- Public, read-only content
- Trusted internal networks

*Rule of thumb: If it’s on the internet or involves users → use HTTPS.*

<!-- TOC --><a name="http-optimisations"></a>
#### HTTP optimisations

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

<!-- TOC --><a name="http-header-basics"></a>
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

<!-- TOC --><a name="server-and-cdn-caching"></a>
## Server and CDN caching

**Server side caching**
| Type               | Example Tools              | Notes                                     |
| ------------------ | -------------------------- | ----------------------------------------- |
| In-memory          | Redis, Memcached           | Very fast, small TTL                      |
| HTTP reverse proxy | Varnish, Nginx             | Can cache full pages or partial responses |
| Application-level  | Django cache, Spring cache | Cache objects, queries                    |

**CDNs**
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

Popular CDNs
- Cloudflare
- Akamai
- Fastly
- AWS CloudFront

Tip: CDNs also handle TLS termination for HTTPS.

**Asset compression**

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

<!-- TOC --><a name="server-push"></a>
## Server push

It is an *HTTP/2 feature*

Concept: Server proactively sends resources to the client before requested

Benefit: Reduces round-trips for critical assets

```
Client → Server : GET /index.html
Server → Client : HTML + CSS + JS (pushed)
```

Pros
- Faster page load
- Fewer round trips

Cons
- Can push unnecessary data → waste bandwidth
- Hard to control caching

Enterprise Tools
- Nginx, Apache HTTP/2
- CDNs (Cloudflare, Fastly)

<!-- TOC --><a name="server-sent-events"></a>
## Server-Sent Events

SSE stands for Server-Sent Events

Concept: Browser opens one-way, long-lived connection

How it works: Server pushes text/event-stream updates

Use case: live notifications, stock tickers

```
Browser → Server : GET /events (HTTP)
Server → Browser : event1
Server → Browser : event2
```

Pros
- Simple (HTTP-based)
- Auto-reconnect built-in
- Works with existing proxies

Cons
- One-way only
- Less control than WebSocket
- Limited browser support on older versions

Enterprise Tools
- Node.js `EventSource` API
- Spring `SSEEmitter`
- Django `Channels`

<!-- TOC --><a name="websockets"></a>
## WebSockets

Concept: Full-duplex, persistent TCP connection

Benefit: Low-latency, real-time communication

```
Client ↔ WebSocket Server
   ↕
Bidirectional messages
```

Use Cases
- Chat apps
- Multiplayer games
- Collaborative editing

Pros
- Full-duplex
- Low-latency
- Works over TCP

Cons
- More complex to scale
- Needs connection management
- Firewalls/proxies sometimes block

Enterprise Tools
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

Key points:
- Starts as a normal HTTP request (Upgrade: websocket)
- Server responds with `101` Switching Protocols
- After handshake, bi-directional messages flow ***without*** HTTP overhead


**Server push vs SSE vs Web sockets**
| Concept                       | Use Case                                                  | Example                                     | Pros                                                        | Cons                                                                                     |
| ----------------------------- | --------------------------------------------------------- | ------------------------------------------- | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Server Push (HTTP/2 Push)** | Server proactively sends resources before client requests | HTTP/2 pushes CSS/JS when HTML is requested | Reduces page load time; leverages HTTP/2                    | Hard to control; may send unnecessary resources; not suitable for dynamic real-time data |
| **SSE (Server-Sent Events)**  | One-way real-time updates from server to client           | Live news feed, stock tickers               | Simple to implement; built-in reconnection; works over HTTP | Only server → client; no binary support; limited browser support in some cases           |
| **WebSocket (WS)**            | Full-duplex, low-latency communication                    | Chat apps, multiplayer games                | Bi-directional; low overhead; supports binary data          | More complex; connection management; firewall/port issues                                |

**More comparison**:
| Feature     | SSE             | WebSocket             | Server Push      |
| ----------- | --------------- | --------------------- | ---------------- |
| Direction   | Server → Client | Bidirectional         | Server → Client  |
| Protocol    | HTTP            | TCP                   | HTTP/2           |
| Use Case    | Notifications   | Chat, games           | Preload assets   |
| Complexity  | Low             | Medium-High           | Medium           |
| Scalability | Moderate        | Needs connection mgmt | Depends on cache |


<!-- TOC --><a name="webrtc"></a>
## WebRTC

Concept: Peer-to-peer,  real-time communication

**Peer-to-Peer (P2P) in WebRTC**
- Meaning: Two clients (browsers or apps) communicate directly with each other without sending all data through a central server.
- Why it matters: Reduces latency and server load.
- Example: Video call between Alice and Bob — their browsers send audio/video directly to each other.

**Real-Time in WebRTC**
- Meaning: Data (audio, video, or messages) is sent immediately as it’s generated, with minimal delay.
- Why it matters: Enables live communication, like calls, screen sharing, or gaming.
- Example: You hear someone speak as they speak, not seconds later.

**Advantage**: Supports audio, video, data channels

Use Cases
- Video conferencing (Zoom, Google Meet)
- Live streaming
- Peer-to-peer file transfer

**How it works**:
- Uses ICE, STUN, TURN to traverse NATs
- Most clients are behind NATs (Network Address Translators) or firewalls.
	- NAT changes private IPs to public IPs.
	- Without help, two browsers can’t easily find each other to do P2P.
- ICE (Interactive Connectivity Establishment)
	- Framework that finds the best way for two peers to connect.
	- Tries multiple options (direct, relayed) to establish connection.
- STUN (Session Traversal Utilities for NAT)
	- Lets a client discover its public IP and port as seen from the internet.
	- Example: “I’m behind NAT, my public IP is 203.0.113.5, port 54321”
- TURN (Traversal Using Relays around NAT)
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

<!-- TOC --><a name="mac-address"></a>
## MAC address

MAC Address stands for Media Access Control

- Purpose: A unique hardware identifier for a network device’s interface (like Wi-Fi or Ethernet).
- How it works: Every network card has a fixed 48-bit address, usually written as `AA:BB:CC:DD:EE:FF.`
- Used for local network communication (LAN), ***not for routing across the internet***.

Analogy
- Think of a MAC address as your device’s permanent name tag in your house or office.
- Even if you change rooms (IP address), the name tag stays the same, so people know exactly which device to talk to.

Key points
- Unique per device interface
- Works within the local network
- Assigned by the manufacturer (can sometimes be spoofed)

<!-- TOC --><a name="ip-address"></a>
## IP address

Concept:
- An IP address is like a home address for your computer/device on a network.
- It tells other computers where to send data.
- Without it, the internet would be like a city with no street numbers.

<!-- TOC --><a name="ipv4-vs-ipv6"></a>
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

<!-- TOC --><a name="public-vs-private-ips"></a>
### Public vs private IPs

Public IP
- Visible on the internet
- Assigned by your ISP
- Example: `103.21.45.67`

Analogy: Your house number that anyone in the world can send mail to.

Private IP
- Only visible inside your network (LAN)
- Not routable on the public internet
- Common ranges (IPv4):
```
10.0.0.0 – 10.255.255.255
172.16.0.0 – 172.31.255.255
192.168.0.0 – 192.168.255.255
```

Example: 192.168.1.10

Analogy: Apartment number inside a building; the mailman needs the building address (public IP) to reach it.

How They Work Together
- Device has private IP: `192.168.1.10`
- Router has public IP: `103.21.45.67`
- When you access a website:
	- Router translates private → public IP using NAT
	- Website sees only 103.21.45.67
	- Response comes back → router sends to correct private IP

Benefits of Private IPs:
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

Additional benefits of IP addresses in general (All types of IPs):
1. Load balancing for scalability: It enables load balancers to distribute traffic by using the IP addreess
2. Security based on IP: Firewall rules, VPNs, etc
3. Microservices and container communication: They can talk to each other using internal private IPs

<!-- TOC --><a name="nat"></a>
## NAT

NAT (Network Address Translator)

Purpose: Lets many devices in a private network share a single public IP to access the internet.

How it works:
- Devices have private IPs (like `192.168.x.x`) that aren’t visible on the internet.
- NAT translates private IP + port → public IP + port when sending packets out.

Important: ***Incoming replies are mapped back to the correct private device.***

Analogy
- Your house has many rooms (devices) but only one mailbox (public IP).
- NAT is the mailman who ensures letters from the internet get to the correct room.

**NAT vs other tools**:
NAT translates IPs, Load Balancer spreads traffic, API Gateway manages and secures API requests.

| Component     | Purpose                            | Scope            | Key Point                                             |
| ------------- | ---------------------------------- | ---------------- | ----------------------------------------------------- |
| NAT           | Maps private IPs to public IPs     | Network (L3/4)   | Enables multiple devices to share one IP              |
| Load Balancer | Distributes traffic across servers | L4/L7            | Improves performance, availability, may terminate TLS |
| API Gateway   | Manages and routes API requests    | Application (L7) | Handles auth, rate limiting, logging, transformations |

<!-- TOC --><a name="subnetting"></a>
## Subnetting

Subnetting **splits** a large network into smaller networks (subnets).

Use:
Helps organize IPs, improve security, and reduce broadcast traffic.

Analogy
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

When to Subnet
- You have a big network and want to divide it into smaller networks for organization, security, or traffic management.
- Example: Your `192.168.1.0/24` network for an office can be split into `/26` subnets for different departments.

<!-- TOC --><a name="cidr"></a>
## CIDR

CIDR stands Classless Inter-Domain Routing. CIDR is a **notation** to specify IP ranges using a suffix `/n` instead of old classful addresses.

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

When to Use CIDR
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

<!-- TOC --><a name="dns"></a>
## DNS

Definition: 
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

Why DNS matters
- Converts human-friendly names → machine-friendly IPs
- Enables scalable and distributed internet addressing
- Supports email, web, and app services

<!-- TOC --><a name="types-of-dns-and-dns-components"></a>
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

<!-- TOC --><a name="dns-request-flow-and-hierarchy"></a>
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


<!-- TOC --><a name="dns-caching"></a>
### DNS caching

DNS caching is the process of temporarily storing the IP address of a recently visited domain (like saving a contact in your phone) so that your computer doesn't have to ask the global network where to find it every single time you load a page.

DNS caching helps to:
1. Reduce latency i.e less time to know the IP address and hit server
2. Reduce load on DNS servers themselves i.e fewer requests going through them

Where does caching occur? It occurs in 3 places:
1. Browser (Browser can cache the previous IP lookups)
2. OS cache (`etc/hosts` file on MacOS, Windows DNS cache) i.e OS too can cache lookups at the system level
3. Recursive resolver (@ ISP): Your ISP too will usually maintain a lookup table as a cache

When does the cache get invalidated?
- **Time-To-Live (TTL)**: This attribute to the cached lookup determines when the cache should be invalidated so that fresh request do actually hit the DNS. It helps avoid stale lookups. Ex: If server IP has changed but we still use the old one indefinitely, it is a problem!

### DNS in large scale systems

1. It ensures "high availability":
  1. DNS can act as a load balancer (a primitive one though)
  2. Anycast DNS: Instead of a single DNS, we use multiple geographically distributed DNSes. Why? Ensures users get the response from the closest DNS -- also the fastest!
2. DNS failover strategies: Use a primary and a secondary DNS (Fault tolerance?)
3. DNS routing to CDNs: DNS can route requests to the nearest CDN for faster loading -- mainly for static assets that need to load quickly and they don't change frequently
4. **DNS security risks**:
  1. DNS poisoning: This is hacking the "address book" of the internet so that when a user types a legitimate website name (like https://www.google.com/search?q=google.com), they are secretly redirected to a fake malicious site (**Analogy**: It’s like a prankster breaking into the phone company’s database and swapping your bank's phone number with their own; when you dial the bank, you unknowingly talk to the scammer). **Solution**: The most effective solution is to implement **DNSSEC** (Domain Name System Security Extensions), which adds cryptographic signatures to DNS records so that computers can verify the data actually came from the authoritative source and was not faked. 
  3. Cache poisoning: This involves tricking a system (like a browser or server) into storing a fake or malicious file, which it then automatically serves to other users thinking it is the real version (Analogy: It’s like someone slipping a fake answer key into a teacher's desk; the teacher (the cache) confidently hands out the wrong answers to every student who asks, spreading the error automatically). **Solution**: configure your cache to treat requests with different inputs (like headers) as completely different pages so that the previous page's cache is not affected
  5. DDOS attacks (Distributed Denial of Service): This is an attack where thousands of hijacked computers flood a target server with junk traffic to crash it or make it unusable for real people (Analogy: It’s like a group of 500 people crowding into a small coffee shop and refusing to buy anything, making it impossible for actual customers to enter or get served). **Note**: **CDNs** help avoid DDOS attacks on DNS and your company servers

<!-- TOC --><a name="load-balancers"></a>
## Load balancers

A Load Balancer is a device or service that **distributes incoming network or application traffic across multiple servers** to *improve availability, reliability, and performance*.

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

Examples
- Hardware: F5 BIG-IP, Citrix ADC
- Cloud: AWS ELB, Google Cloud Load Balancing, Azure Load Balancer
- Software: NGINX, HAProxy

Pros
- High availability → if a server fails, LB redirects traffic
- Scalability → add/remove servers without affecting clients
- Health checks → ensures traffic only goes to healthy servers
- SSL/TLS termination → reduces load on backend servers

Cons
- Single LB failure → if not redundant, it becomes a bottleneck
- Added latency → traffic goes through an extra layer
- Complexity → configuration and monitoring required

When to Use
- Web apps with many concurrent users
- Microservices architectures needing traffic distribution
- High availability or fault tolerance requirements

When Not to Use
- Small apps with very few users
- Simple static content sites where direct server access is fine

<!-- TOC --><a name="api-gateways"></a>
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

Analogy
- API Gateway = Reception desk in a large office building
- Visitors (clients) arrive
- Reception checks ID (auth), enforces rules (rate limits), and directs them to the correct department (service).
- Reduces chaos and ensures security and logging

Examples
- Cloud: AWS API Gateway, Azure API Management, GCP Apigee
- Open-source: Kong, Tyk, Traefik, NGINX as API gateway

Pros
- Centralized security and authentication
- Simplifies client interaction with many microservices
- Rate limiting, caching, logging for observability and performance
- Can transform requests/responses to match service requirements (Request translation)

Cons
- Single point of failure if not redundant
- Added latency since all traffic passes through it
- Extra complexity in configuration and maintenance

When to Use
- Microservices architecture with multiple APIs
- Need centralized auth, rate limiting, logging, or request transformation
- Public APIs that need security and monitoring

When Not to Use
- Small monolithic apps
- Direct, simple client-server setups with minimal security or routing requirements

**API gateways vs Load balancers**

Load balancers focus on distributing traffic across servers, while API gateways focus on managing, securing, and routing API requests.

| Feature                   | API Gateway                                                         | Load Balancer                                                              | Example Use Case                                             |
| ------------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------ |
| **Purpose**               | Central entry point for APIs; manages, secures, and routes requests | Distributes traffic across servers to improve availability and performance | API Gateway: AWS API Gateway; LB: AWS ELB                    |
| **Layer**                 | Application Layer (L7)                                              | Layer 4 (TCP/UDP) or Layer 7 (HTTP/HTTPS)                                  | NGINX, HAProxy                                               |
| **Routing**               | Routes based on URL paths, headers, methods                         | Routes based on IP, TCP/UDP, or HTTP host/path                             | `/api/user` → UserService vs any request → least busy server |
| **Security**              | Handles auth, tokens, rate limiting, logging                        | Usually does TLS termination, may do basic access control                  | OAuth2 token check vs SSL termination                        |
| **Transforms / Features** | Can modify requests/responses, caching, protocol translation        | Mostly pass-through or simple load distribution                            | Convert JSON → XML or route HTTP → gRPC                      |
| **When to use**           | Microservices, public APIs, need security & monitoring              | Distribute traffic across multiple servers, ensure high availability       | Microservice architecture vs high-traffic web app            |

<!-- TOC --><a name="proxy-and-reverse-proxy"></a>
## Proxy and reverse proxy

| Type                      | Definition                                                                                                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Proxy (Forward Proxy)** | A server that acts on behalf of **clients**, forwarding requests to the internet and returning responses. Clients know and configure the proxy.                           |
| **Reverse Proxy**         | A server that acts on behalf of **servers**, receiving requests from clients and forwarding them to backend servers. Clients usually don’t know the reverse proxy exists. |

**Forward proxy (proxy)**

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

Flow explained
1. Client sends request to the proxy instead of directly to the internet.
2. Forward Proxy forwards the request to the target server.
3. Server responds to the proxy.
4. Proxy sends the response back to the client.

**Reverse proxy**

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

Flow explained
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


**Reverse proxy vs Load balancer**

A reverse proxy primarily secures, manages, and routes requests to servers, while a load balancer primarily distributes traffic across servers for availability and performance.

| Feature                 | Reverse Proxy                                                                                   | Load Balancer                                                                           | Example Use Case                                                                                        |
| ----------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Purpose**             | Acts on behalf of servers, forwards client requests, can add security, SSL termination, caching | Distributes traffic across multiple servers to ensure high availability and performance | Reverse Proxy: NGINX in front of web servers; LB: AWS ELB distributing requests to multiple app servers |
| **Layer**               | Application Layer (L7)                                                                          | Layer 4 (TCP/UDP) or Layer 7 (HTTP/HTTPS)                                               | NGINX, HAProxy                                                                                          |
| **Routing**             | Can route based on URL path, host headers, or other application info                            | Can route based on IP, TCP/UDP, or HTTP host/path                                       | `/api/user` → UserService (Reverse Proxy) vs Any request → least busy server (LB)                       |
| **Security**            | Can handle SSL/TLS termination, hide backend servers, enforce auth                              | Usually handles TLS termination; not primarily for security                             | Reverse Proxy hides real server IPs                                                                     |
| **Additional Features** | Caching, compression, request/response transformation                                           | Health checks, sticky sessions                                                          | Reverse Proxy can modify requests; LB mainly balances load                                              |
| **When to use**         | Securing backend servers, centralizing routing, adding caching                                  | Distributing traffic for scalability and fault tolerance                                | Microservices with exposed APIs (Reverse Proxy) vs Web app needing high availability (LB)               |

**Reverse proxy vs API gateway**

Reverse Proxy is mainly about protecting and routing to servers, while an API Gateway is about managing, securing, and transforming API requests for clients.

| Type              | Acts on behalf of         | Main job                                                                           |
| ----------------- | ------------------------- | ---------------------------------------------------------------------------------- |
| **Reverse Proxy** | The **server**            | Protects backend servers, routes requests, can do caching or SSL termination       |
| **API Gateway**   | The **API/microservices** | Manages all API traffic, handles security, routing, rate limiting, transformations |

- Reverse Proxy: NGINX sits in front of Server1, Server2, Server3 → forwards HTTP requests.
- API Gateway: AWS API Gateway sits in front of User Service, Payment Service, Order Service → handles authentication, rate limiting, routing, and request/response changes.

Tip to remember:
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

Flow Explained
- Clients send requests to the Reverse Proxy first.
- RP protects backend web servers, hides their IPs, may handle SSL termination.
- Requests needing microservice APIs are routed through the API Gateway.
- API Gateway handles authentication, rate limiting, request/response transformation, logging, etc.
- API Gateway forwards requests to the correct microservice.
- Responses flow back the same path to the client.

Memory Tip:
- Reverse Proxy = server-focused, first line of defense
- API Gateway = API-focused, smart traffic manager for microservices

<!-- TOC --><a name="cryptography"></a>
## Cryptography

Cryptography = the science of keeping information secret, safe, and trustworthy.

Main goals:
- Confidentiality → only intended people can read it
- Integrity → data is not tampered
- Authentication → prove identity of sender/receiver
- Non-repudiation → sender cannot deny sending

Analogy: Sending a locked box with a secret message inside. Only someone with the correct key can open it.

**Types of cryptography**

| Type           | How it Works                                  | Example Use  |
| -------------- | --------------------------------------------- | ------------ |
| **Symmetric**  | Same key to encrypt and decrypt               | AES, DES     |
| **Asymmetric** | Public key encrypts, private key decrypts     | RSA, ECC     |
| **Hashing**    | Converts data into fixed-size digest; one-way | SHA-256, MD5 |


<!-- TOC --><a name="rsa"></a>
### RSA

Definition
- RSA is an asymmetric encryption algorithm.
- Uses a public key (everyone can see) to encrypt, and a private key (kept secret) to decrypt.

```
Sender:   Message → Encrypt with Public Key → Ciphertext → Send
Receiver: Ciphertext → Decrypt with Private Key → Original Message
```

Analogy:
- Imagine a mailbox with a public slot (public key).
- Anyone can drop a letter in (encrypt), but only the owner has the key to open the mailbox (private key) and read the letters.

Key Points:
- RSA is asymmetric: public key ≠ private key
- Usually used to secure small data or encryption keys, not large files. Why?
	- 1. Computational Cost
		- RSA uses very large numbers (hundreds or thousands of bits) and complex math (modular exponentiation).
		- Encrypting large files directly would be very slow and resource-intensive.
	- 2. Practical Approach
		- Instead of encrypting the full file, RSA is used to encrypt a small symmetric key (like an AES key).
		- Then the large file is encrypted with AES (fast symmetric crypto).
		- This combines speed of AES with security of RSA.

Analogy
- RSA = a tiny, very strong padlock
- AES = a big, fast padlock

<!-- TOC --><a name="service-mesh"></a>
## Service mesh

**Definition**

> A Service Mesh is a dedicated infrastructure layer for handling service-to-service communication. It uses a sidecar proxy to abstract network complexity (like retries, security, and observability) away from the application code. It's great for complex microservices, but overkill for simple apps.

**The Problem: The "Microservices Headache"**
Imagine you are building a simple e-commerce app. You have a Cart Service and a Payment Service.

- In a Monolith (Old way): The Cart just calls a function payment.process(). It’s instant and reliable because it happens in the same memory space.
- In Microservices (New way): The Cart has to make an HTTP request over a network to the Payment Service.

Problems:
1. Here is where the headache starts. Networks are unreliable.
2. What if the Payment Service is down? You need code to retry.
3. What if the Payment Service is slow? You need code for a timeout.
4. How do you ensure the data is safe? You need code for encryption (SSL/TLS).
5. How do you know who called whom? You need code for logging.

The Nightmare: If you have 50 microservices written in different languages (Java, Python, Go), you have to write this "retry/timeout/security" logic 50 times. If you want to change how retries work, you have to update and redeploy all 50 services.

**The Solution: The "Personal Assistant" (Service Mesh)**

A Service Mesh solves this by saying: "Hey developer, stop writing networking code. Focus on business logic. I will handle the network for you."

It does this using a pattern called the **Sidecar**.

- Imagine every Microservice is a CEO.
- The CEO (Service) shouldn't be dialing numbers, waiting on hold, or checking security badges.
- Instead, every CEO gets a Personal Assistant (Sidecar Proxy) sitting at the desk right next to them.

The Workflow:
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

**The Three Superpowers (Why use it?)**

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

**Service mesh vs Load Balancer vs API Gateway**

The Analogy
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

Does a Service Mesh "live inside" these LB or API gateway tools?
Short Answer: **No**. They are ***neighbors***, not roommates.

Detailed Answer: They are distinct layers, but they are often built using the same underlying technology.

The Shared Engine: A popular tool called **Envoy** is a high-performance proxy.
- It is often used inside an API Gateway to route external traffic.
- It is also used as the sidecar in a Service Mesh to route internal traffic.
- So, while a Service Mesh doesn't "live inside" an API Gateway, they might both be "wearing the same uniform" (using Envoy under the hood).

Exceptions: Some modern tools try to blur the lines. For example, **Istio** (a service mesh) has an "Ingress Gateway" feature that acts like a basic API Gateway, allowing you to use one tool for both. However, in a strict system design sense, treat them as separate boxes.

Use RSA to secure the tiny key, AES to secure the big file
