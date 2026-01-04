# Design a Notification System

A notification system is a centralized platform that delivers alerts and messages (notifications) to users on behalf of multiple separate applications (tenants). It acts as a shared service that any application – be it a social network, an e-commerce site, or a productivity tool – can use to send notifications to its users

**Problem**

Design a centralized notification system that can send notifications across multiple channels (Push, SMS, Email) to millions of users. It acts as a middleware between internal microservices (like Billing, Order Service) and external third-party providers (like Twilio, SendGrid, APNS (***Apple Push Notifications***) / FCM (***Firebase Cloud Messaging*** for Android))

**Note**

These interfaces or tools are the way in which notification systems work. We do not have to build it from scratch. However, we have to build the system around it based on our requirements

```
 [ YOUR INTERNAL SYSTEM ]   |        [ EXTERNAL VENDOR GATEWAYS ]            [ END USER DEVICES ]
                            |
+------------------+        |    /-> [ Apple APNs ] -----------------------> [ iOS Device ]
|                  |        |   /    (Push Network)
| Notification     | ---API-+--/
| Service          |        |
| (Producer)       | ---API-+------> [ Google FCM ] -----------------------> [ Android Device ]
|                  |        | \      (Push Network)
+------------------+        |  \
                            |   \--> [ Email Vendor (e.g., SendGrid) ] ----> [ Email Inbox ]
                            |    \   (SMTP Servers)
                            |     \
                            |      > [ SMS Vendor (e.g., Twilio) ] --------> [ Mobile Phone ]
                            |        (Telecom Network)
```

Payload examples:
1. iOS
```json
{
  "aps": {
    "alert": {
      "title": "Friend Request",
      "body": "Bob wants to connect."
    },
    "badge": 1
  }
}
```
2. Android (FCM)
```json
{
  "to": "device_token_xyz123",
  "notification": {
    "title": "Friend Request",
    "body": "Bob wants to connect."
  },
  "data": {
    "userId": "88"
  }
}
```
3. Email
```json
{
  "personalizations": [
    { "to": [{ "email": "bob@example.com" }] }
  ],
  "from": { "email": "noreply@myapp.com" },
  "subject": "Friend Request",
  "content": [
    { "type": "text/plain", "value": "Bob wants to connect." }
  ]
}
```

## Functional requirements

Questions to ask:
1. What?: *What types of notifications must the system support?*
2. What?: *What triggers notifications?*
3. Result?: *Can the users opt-out of receiving notifications?*
4. Result?: *Should we track the notifications sent?*
5. Where/Scale?: *Is it a separate system?*. *Ans: Yes*. **No formal FR required**

### Functional Requirements as a result of these questions

1. System must support push notifications for iOS and Android, SMS messages, and email
2. Clients should trigger notifications or services should be able to schedule them
3. Users should be able to opt-out of specific channels (e.g., "Don't send me SMS") or categories (e.g., "Marketing vs. Transactional")
4. The system must track the status of notifications (Queued, Sent, Delivered, Read, Failed)

## Non-functional requirements

Questions to ask:
1. Scale (Throughput) and Volume (Overall storage or Bandwidth): 
	- *How many notifications must we handle per day?*
2. Speed (Latency/CPU/Memory):
	- *What is the type of system? Is the speed Real-time or not?*
3. CAP trade-off (AP vs CP):
	- *Should we prioritise availability or consistency? i.e what happens if there are delays in notifying a device?*
4. Safety net (Fault Tolerance): 
	- *What should happen if a notification fails? Ex: A third party API is down*
5. Geography:
	- *Does notification delivery matter based on region?*. *Ans: No*. **No formal FR required**

### Non-Functional Requirements as a result of these questions

1. Must handle millions of notifications per day (e.g., 10M-100M/day)
2. Soft-real time system. If a system is under heavy workload, a slight delay is acceptable
3. System should be available i.e a slight delay in notification is okay if the system is overloaded (No error/dropping of notifications)
4. Reliability / Fault tolerance: No notification should be lost (ex: Even if a third party is down)

**Other unique NF requirements**:
- *Extensibility*: Easy to plug in new third-party providers (e.g., swapping SendGrid for Mailgun)
 
## Core entities

Objects and users persisted in the database or exchanged via APIs:

1. **Notification**: The core object (`id`, `user_id`, `type`, `content`, `status`, `created_at`)
2. **UserPreference**: Stores settings (`user_id`, `channel_sms: bool`, `channel_email: bool`)
3. **Device**: Stores tokens for push (`user_id`, `device_token`, `os_type`)
4. **Template**: Pre-defined message structures (`id`, `name`, `body_text` with placeholders)

## System interface or API endpoints

System interface or API? We can go with an API as this is not a pipeline for data engineering. Clients can ask for this

1. **`POST /v1/send --> 202 Accepted (Returns a notification_id immediately)`**
```
{
  "user_id": "12345",
  "type": "ORDER_CONFIRMATION",
  "channels": ["EMAIL", "SMS"], // Optional, defaults to user pref
  "payload": {
    "order_id": "ABC-99",
    "amount": "$50.00"
  }
}
```
2. **`GET /v1/status/{notification_id} --> 200 OK`**
```
{"status": "DELIVERED", "updated_at": "..."}
```

## High Level Design

1. Starting with MVP: Generally start with the  `client-server(database)` model
2. Go through *Functional Requirements* one by one and see if it is fulfilled. If not, modify design  (***Note: Need not go in order***)

Before we consider FRs, we must consider some *unique challenges* for this problem:
- **User details**: We need phone numbers for sending SMSes, Email address for sending emails
- **Device details**: We need "device tokens" to be stored to identifying phones to send iOS/Android notifications to

Both these details are needed to make a notification system work. It is okay to store them once and use later. Hence, we will briefly show the *Sign up flow*:

```
 [client] -- (sign up) --> [LB] --> [Server 1 | Server 2 | ...] --> [DB 1, DB2, ...]
```
Data model (tables):
```
1. User: { user_id (PK), email, country_code, phone_number, created_at }
2. Device { id (PK), device_token, user_id (FK), last_logged_in_at }
```

Coming to FRs:

*FR#1: System must support push notifications for iOS and Android, SMS messages, and email*
- This will be supported when our notification system connects to 4 different notification tools/services (3rd party)

*FR#2: Clients should trigger notifications or services should be able to schedule them*
- The notification system will be connected to application servers (APIs) and these servers can generate notifications but also take requests from clients to send them

*FR#3: Users should be able to opt-out of specific channels (e.g., "Don't send me SMS") or categories (e.g., "Marketing vs. Transactional")*
- The notification system can have a persistent storage where details of user opt-out and other configuration can be stored
- Ex: `user_exception_list{ key: [user_id] }`, `device_exception_list { key: [device_token] }`

*FR#4: The system must **track** the status of notifications (Queued, Sent, Delivered, Read, Failed)*
- This is typically done via **webhooks** invoked from a vendor service
- This works via Webhooks (Callbacks). When you send the notification, you attach a **unique `Tracking ID`**. The vendor (Twilio/SendGrid) then acts like a courier, actively calling your server back ("pinging" a specific URL you provided) every time the status changes
```
 [ YOUR SERVER ]                          [ VENDOR (Twilio/FCM) ]
      |                                           |
      | 1. POST /send (ID: 101) ----------------> | (Queued)
      |                                           |
      | 2. <----- webhook (ID: 101, "Sent") ----- | (Sent)
      |                                           |
      |              (Time passes...)             |
      |                                           |
      | 3. <----- webhook (ID: 101, "Failed") --- | (Failed)
      |           (Reason: "Invalid Number")      |
      v                                           v
(Update DB: "ID 101 = Failed")
```

```
(1) Trigger Request (Can eminate from CLIENT directly)
        |
+---------------------+      +---------------------+
| One of the Services | ---> |    Load Balancer    |
| (Billing / Order)   |      +----------+----------+
+---------------------+                 |
                                        v
                            +-----------------------+
                            | NOTIFICATION SERVICE  |
                            +---+-------+-------+---+
                                |       |       |
            (2) Send Request    |       |       |
                to Vendors      |       |       |
                +---------------+       |       +---------------+
                |                       |                       |
                v                       v                       v
         +------------+          +------------+          +-------------+
         |   Twilio   |          |  SendGrid  |          | FCM / APNS  |
         |   (SMS)    |          |  (Email)   |          |   (Push)    |
         +------+-----+          +------+-----+          +------+------+
                |                       |                       |
                v                       v                       v
           [SMS/Phone]             [Email Inbox]          [Mobile App]
```

**Problems with this design**
1. *Single point of failure*: Only one notification system - if it fails, notification system goes down
2. *Hard to scale*: One notification service handles everything related to notifications - Need to scale out?
3. *Performance bottleneck*: One notification system can get overloaded with notification requests - leads to system degradation, especially at huge scale

## Deep dive

1. Starting with constructed High Level Design:
2. Go through *Non-Functional Requirements* one by one and see if it is fulfilled. If not, modify design (***Note: Need not go in order***)
3. Also check some unique constraints of the problem you might have missed (scan or read the problem again) and apply solutions for them

*NFR#1: Must handle millions of notifications per day (e.g., 10M-100M/day)*

B.O.T.E calculations:
- `100M / 24 * 3600` notifications per second = `10^8 / (~800 x 10^2)` = `10^8 / (~1000 x 10^2)` = `10^8 / 10^5` = **`1000`** notifs / sec
- Storage:
	- Payload: assume 1kb average
	- Assume 5 days' retention since notifications are not meant for long retention periods in general
	- `1kb x 100M x 5` = `1 x 10^3 x 10^2 x 10^6 x 5` B = `5 x 10^11` bytes = **`500 GB`**

- Most SQL/NoSQL databases can handle 1000req/sec on average but the storage size is way too high!
- We can use a Redis store: More writes per second and can storage 10s of GBs per partition key
	- We need to have sharding (Replicas for the database)
	- Sharding/partitioning can be leaderless. Each replica can be used to service a notification
- However, we ***cannot use either!***
	- The notification service is still tightly coupled with the APNs, FCM, Email sendgrid, etc
	- We need to decouple it to scale!
		1. **Add message brokers** (Ex: Kafka)
		2. **Add workers (distributed) that pick up events from the message brokers**

- We can also have a distributed notification service (the servers) with load balancer to handle the load for ***parallel processing*** (Again, can be ***leaderless***)
- On top of this, we can have a cache layer in front of the database system to read the user / device configurations fast (and updated periodically) like `user_exceptions_list` and `device_exceptions_list`

```
                           (1) Trigger Request
                                      |
+---------------------+      +--------v---------+
|  Internal Service   | ---> |   Load Balancer  |
| (Billing / Order)   |      +--------+---------+
+---------------------+               |
                                      v
                             +--------+------------+
                             | Notification API    |  <-- (Validates, Rate Limits, Checks Preferences)
                             | Service (scaled-out)|
                             |                     | --> [ Horizontally scaled-out cache ] --> [ Horizontally scaled DBs ]
                             +---------+-----------+
                                      |
                                      | (2) Enqueue Message
                                      v
               +---------------------------------------------+
               |             Message Queue (Kafka)           |
               |  [Topic: SMS]  [Topic: Email]  [Topic: Push]|
               +------+---------------+--------------+-------+
                      |               |              |
           (3) Consume|    (3) Consume|   (3) Consume|
                      v               v              v
               +------------+   +------------+   +------------+
               | SMS Worker |   |Email Worker|   |Push Worker |
               |(scale-out) |   |(scale-out) |   |(scale-out) |
               +------+-----+   +-----+------+   +------+-----+
                      |               |                 |
                      | (4) Call 3rd Party              |
                      v               v                 v
               +------------+   +------------+   +-------------+
               |   Twilio   |   |  SendGrid  |   | FCM / APNS  |
               +------------+   +------------+   +-------------+
                      |               |                 |
                      v               v                 v
               [User Phone]      [User Inbox]     [User Mobile]
```
- Each type of worker in the diagram can be scaled-out (multiple works for each type of notifcations) and load balanced 
- Scale: "RabbitMQ" vs. "Kafka" for message broker
	1. RabbitMQ: Good for individual task tracking and complex routing (e.g., retry queues per channel). Better for "Task" based processing
	2. Kafka: Good for massive throughput and log retention. Better if we need to replay events or analyze notification history later
	- Alex Xu Recommendation: ***Kafka is generally preferred for the high-volume ingestion layer, while RabbitMQ is often used for the actual task dispatching if complex retry logic is needed***
	- The Short Answer: **Choose RabbitMQ**. If you are forced to pick just one for the entire system, pick RabbitMQ
		- Why? A notification system is functionally a Task Processing System, not just a Data Streaming system. The hardest part of a notification system is handling failures (e.g., "Twilio is down, retry this SMS in 5 minutes")
			- RabbitMQ has built-in features for this that Kafka does not:
				1. *Per-Message Acknowledgement*: You can confirm individual messages were processed
				2. *Native Delay/Retry Support*: You can easily push a failed message to a "Retry Queue" with a Time-To-Live (TTL) delay (e.g., 5 mins). Kafka cannot delay individual messages without blocking the entire partition
				3. *Complex Routing*: You can easily route messages to different queues based on priority (e.g., "OTP" vs "Marketing") using routing keys

*NFR#2: Soft-real time system. If a system is under heavy workload, a slight delay is acceptable*
- Since we are using a message broker, producers and consumers can scale independently and more importantly push and consume events when they are ready! Delays will exist when load increases but system will continue to work and hence, be available

*NFR#3: System should be available i.e a slight delay in notification is okay if the system is overloaded (No error/dropping of notifications)*
- Covered by NFR#2 solution
- System does not crash and show an error to the user when the notifications throughput increases
	- It causes delays (Acceptable) due to message brokers but system will still be available

*NFR#4: Reliability / Fault tolerance: No notification should be lost (ex: Even if a third party is down)*
- **Deduplication**: Prevents sending the same alert twice if the client retries. Use `notification_id` as an ***idempotency*** key.
- **Rate Limiting**: Prevents a buggy service from triggering 1,000 "Payment Failed" emails in 1 minute.
	- Use a *Leaky Bucket* algorithm in Redis
	- Key: `ratelimit:user_id:12345`. Limit: 5 per minute. If exceeded, discard
- **Reliability**:
	1. ***Prevent data loss***: 
		- *Loss in message broker*: Add a **retry mechanism** in the message broker (RabbitMQ is very good for this)
		- *Loss in notification service itself*: The database must persist the notifications and be able to retry them or update the status (once the webhook indicates success)
		- ***Exactly once message?***
			- Very difficult to implement practically
			- Solutiom: Use an ***"at-least once"*** event system with retry but have an *IDEMPOTENT* operation on the worker side (Ex: based on a notification ID)

## Other points

Notification service is also responsible for:
1. **Verification** (Verify phone numbers, email IDs, user IDs, etc)
2. Saving **Notification Templates**: Why? Each type of notification has a template and variables like username are filled in
	- These templates are saved within the notification service **OR** the workers to save on bandwidth when the requests come from the (***performance***)
3. **Monitor queued notifications and analytics**: Check thresholds for loads on the system and analyse what type of events are triggered, their frequency or volume, timelines, etc for a holistic report generation


**Complete whiteboard diagram**

```
(1) Trigger
                                         |
+---------------------+       +----------v----------+
|  Internal Services  | ----> |    Load Balancer    |
| (Billing, Shipping) |       +----------+----------+
+---------------------+                  |
                                         v
                      +--------------------------------------+
                      |         Notification Service         |
                      |      (API / Validation Logic)        |
                      +---+----------------------------+-----+
                          |                            |
          (2) Read/Write  | [Cache layer?]             | (3) Enqueue Valid Msg
           Preferences &  |                            |
           Blocklist      v                            v
               +----------+-----------+      +---------+----------+
               |      DB Cluster      |      |   Message Queue    |
               | (User Settings &     |      |      (Kafka)       |
               |  Exception Lists)    |      +----+----+-----+----+
               +----------------------+           |    |     |
                                                  |    |     |
            +-------------------------------------+    |     +------------------+
            | (4) Consume Push                         | (4) Consume SMS        | (4) Consume Email
            v                                          v                        v
   +------------------+                       +------------------+     +------------------+
   |   Push Workers   |                       |   SMS Workers    |     |   Email Workers  |
   |  [W1] [W2] [W3]  |                       |  [W1] [W2] [W3]  |     |  [W1] [W2] [W3]  |
   +--------+---------+                       +--------+---------+     +--------+---------+
            |                                          |                        |
            +-------------------+   +------------------+                        |
            |                   |   |                  |                        |
            | (5) Log           v   v (5) Log          | (5) Log                |
            |            +------+---+-------+          |                        |
            |            | Analytics Service|          |                        |
            |            +----------+-------+          |                        |
            |                       |                  |                        |
            v (6) Send              v                  v (6) Send               v (6) Send
     +-------------+         +------+-------+    +-----+------+          +------+-----+
     | FCM / APNs  |         | Data Warehse |    |   Twilio   |          |  SendGrid  |
     +-------------+         +--------------+    +------------+          +------------+
```
