# System Design, From Zero

> **Who this is for:** someone who has never studied system design and has one shot to learn it properly before a Zomato interview.
> **How to read it:** top to bottom. Every part builds on the one before. Each part ends with a few questions — try to answer them in your head before reading the answer. That's what makes it stick.
> **Reading time:** about 2.5–3 hours. Do it in two or three sittings.
> **Then:** `07_System_Design_QA.md` has full worked answers to every design question Zomato has asked or is likely to ask.

---

## Part 0: What a system design interview actually is

In a DSA round you write code that runs on one machine. In system design, you're asked to sketch how a whole product works when millions of people use it at once — "design Zomato's order tracking", "design a rate limiter". There's no single right answer. The interviewer wants to see that you can:

1. Ask what's actually needed before building anything.
2. Break the system into sensible parts.
3. Pick the right tool for each part and **say why**.
4. Think about what goes wrong — servers crash, networks fail, the same request arrives twice, dinner-time traffic is 10× normal.
5. Talk about trade-offs honestly ("this is faster but can show slightly old data").

**What Zomato expects from a fresher.** Interview reports from 2024–26 show off-campus freshers often get a 10–20 minute design or schema question inside Round 1, and a fuller design in Round 2. They don't expect you to know Zomato's real architecture. They do expect clean basics, and reports specifically say **happy-path-only answers were marked down** — always talk about failures.

Most-asked at Zomato, from those reports: **rate limiter** (6+ times), **notification system** (5+), **food ordering and order tracking**, **seat booking (BookMyShow / District)**, **payments**, **menu with customisations**, **"show a banner X times in Y hours"**, plus Kafka and Redis questions. All of them are solved in `07`.

### The answer framework (memorise this)

Use the same seven steps for every question. It keeps you calm and makes sure you don't forget anything.

1. **Clarify requirements (2–3 min).** *Functional:* what must it do? ("place an order, track it, cancel it"). *Non-functional:* how big, how fast, how reliable? ("10 lakh orders a day, tracking updates every 5 seconds, payments must never double-charge"). Write them down. Say out loud what you're leaving out.
2. **Estimate the scale (1–2 min).** Requests per second, data size. Rough numbers are fine; it tells you whether one database is enough.
3. **Define the APIs (2 min).** The handful of endpoints the app calls.
4. **Design the data model (3 min).** Main tables, keys, and SQL vs NoSQL.
5. **Draw the high-level design (5 min).** Boxes and arrows: client → load balancer → services → database, cache, queue.
6. **Deep-dive one or two hard parts, including failures (5–10 min).** This is where you score points.
7. **Wrap up with trade-offs and bottlenecks (1–2 min).** "If I had more time, I'd…"

Talk while you draw. Ask "does this direction make sense, or should I go deeper on X?" — interviewers like to steer, and it shows you're collaborative.

---

## Part 1: From one server to many

### 1.1 What happens when you open the Zomato app

Your phone is the **client**. It needs data — say, restaurants near you — so it sends a **request** to a **server**, a computer somewhere in a data centre running Zomato's code. The server works out the answer, often by asking a **database**, and sends back a **response**. That's the client–server model, and everything else is built on it.

The request travels over the internet using **HTTP** (or **HTTPS** — the same thing, encrypted). Before it can be sent, your phone has to find the server's address. Names like `api.zomato.com` are for humans; computers use **IP addresses** like `13.235.10.4`. **DNS** (Domain Name System) is the internet's phone book that turns one into the other. Your phone asks DNS once, remembers the answer for a while, then connects.

### 1.2 One server hits a wall

Picture Zomato on day one: one server running the code and the database. It works. Then users grow and the server starts to struggle — CPU at 100%, pages slow, eventually it crashes. Two ways out:

- **Vertical scaling (scale up):** buy a bigger machine — more CPU, more RAM. Simple, no code changes. But there's a ceiling (machines only get so big), it gets expensive fast, and it's still **one machine — if it dies, everything dies**. That's a **single point of failure (SPOF)**.
- **Horizontal scaling (scale out):** run the same code on many ordinary machines and spread the traffic across them. No real ceiling, and if one machine dies the others carry on. This is how every large system works. It brings new problems — how do you spread the traffic, and how do machines share data? — which is what the rest of this document is about.

### 1.3 The load balancer

If you have ten servers, something has to decide which one gets each request. That's a **load balancer (LB)**. Clients talk to the LB's address; the LB forwards each request to a healthy server.

How it chooses:
- **Round robin:** 1, 2, 3, …, 10, then back to 1. Simple and fair when requests are similar.
- **Least connections:** send to whichever server is currently handling the fewest requests. Better when some requests are slow.
- **Hash-based:** hash something about the request (user ID, IP) to pick the server, so the same user always lands on the same server. Useful when servers keep per-user data in memory.

**Health checks:** the LB pings each server every few seconds ("are you alive?"). If a server stops answering, the LB stops sending it traffic. That's how a crash becomes invisible to users.

**L4 vs L7:** an L4 load balancer looks only at network-level info (IP and port) and is very fast. An L7 load balancer understands HTTP, so it can route by URL — `/search` to the search servers, `/orders` to the order servers. Most web systems use L7.

And the LB itself? Run two of them, so one can take over if the other fails.

### 1.4 Stateless servers — the rule that makes scaling easy

If server 3 remembers "user A is logged in" in its own memory, and the LB sends user A's next request to server 7, server 7 doesn't know them. So the rule is: **app servers keep no user data in their own memory between requests.** They're **stateless**. Anything that must persist — sessions, carts, orders — goes into a shared store (database or cache) that every server can reach.

Then any server can handle any request, you can add or remove servers freely, and a crashed server loses nothing. Logins usually work with a **token** (like a JWT) that the client sends with every request, so the server doesn't even need to look up a session.

### 1.5 Other front-door pieces

- **Reverse proxy:** a server that sits in front of your app servers and forwards requests to them. Load balancers are a kind of reverse proxy. Nginx is the famous one. It can also handle HTTPS encryption, compression and caching.
- **API gateway:** a smarter front door for microservices. One entry point that does the common jobs for every request — checks login (authentication), applies rate limits, routes to the right service, logs — so each service doesn't repeat them.
- **CDN (Content Delivery Network):** a network of servers spread around the world (or across Indian cities) that keep copies of **static files** — images, app bundles, videos — close to users. A restaurant photo loads from a CDN server in your city instead of from Mumbai. Faster for users, far less load on your servers. Zomato's food photos are a perfect CDN use case.

### 1.6 Monolith vs microservices

A **monolith** is one big application containing everything — users, menus, orders, payments — deployed as one unit. It's simple to build, test and run, which is why most products start this way.

**Microservices** split it into small services, each owning one job and its own data: a User service, Menu service, Order service, Payment service, Delivery service. They talk over the network (HTTP calls or messages).

Why split? Each team can deploy its service independently; you can scale only the busy parts (search gets 100× the traffic of payments); one service crashing doesn't take down the rest; each can use the best tool for its job.

What it costs: network calls are slower and can fail; data is spread out, so a single "transaction" across services becomes hard; there's much more to monitor and debug. **Say this trade-off in interviews** — "microservices aren't free; I'd split along clear business boundaries."

### Check yourself
**Q: What's a single point of failure, and how do you remove one?**
A: Any component whose failure brings the whole system down — one server, one database, one load balancer. You remove it with redundancy: run two or more and have a way to switch over automatically (health checks plus failover).

**Q: Why must app servers be stateless to scale horizontally?**
A: Because the load balancer can send a user's requests to any server. If state lived in one server's memory, other servers wouldn't have it, and a crash would lose it. Keep state in a shared database or cache.

**Q: Vertical or horizontal — which would you pick for Zomato's order service, and why?**
A: Horizontal: run many stateless copies behind a load balancer. It has no ceiling, survives machine failures, and lets you add capacity for dinner peaks. Vertical is fine early on, or for a database you can't easily split yet.

**Q: Where would you put restaurant images?**
A: In object storage (like Amazon S3), served through a CDN so users download them from a nearby server. Never in the database.

---

## Part 2: Storing data

### 2.1 SQL vs NoSQL

**SQL (relational) databases** — MySQL, PostgreSQL — store data in tables with fixed columns. Tables link to each other with keys (an order row has a `user_id` pointing to the users table) and you combine them with **joins**. Their superpower is **transactions with ACID guarantees**:
- **Atomicity:** a group of changes all happen or none do. Deduct money *and* create the order — never just one.
- **Consistency:** rules (like "amount can't be negative", "every order has a real user") always hold.
- **Isolation:** two transactions running at the same time don't see each other's half-finished work.
- **Durability:** once committed, it survives a crash (it's written to a log on disk first).

**NoSQL databases** give up some of that for flexibility and easy horizontal scaling. Four families:
- **Key-value** (Redis, DynamoDB, Aerospike): look up a value by key. Extremely fast. Caches, sessions, counters.
- **Document** (MongoDB): store JSON-like documents; each can have different fields. Product catalogues, user profiles.
- **Wide-column** (Cassandra): built for huge write volumes spread across many machines. Chat messages, event logs, location history.
- **Graph** (Neo4j): nodes and relationships. Social networks, fraud rings.

**How to choose — say this in interviews:**
- Money, orders, inventory, anything needing transactions and correctness → **SQL**.
- Massive write volume, simple lookups by key, flexible shape, easy scale-out → **NoSQL**.
- Real systems use both: Zomato-style orders and payments in MySQL; caching in Redis; search in Elasticsearch; location pings in a fast store.

### 2.2 Indexes — why lookups are fast

Without help, finding `WHERE phone = '98…'` in a 10-crore-row table means reading every row (a **full table scan**). An **index** is a separate, sorted structure — almost always a **B+ tree** — that maps column values to row locations, like the index at the back of a book. A B+ tree is short and wide (3–4 levels even for billions of rows), and its bottom level is a linked, sorted list, so both exact lookups and ranges ("orders between 7 and 9 pm") are fast — O(log n).

Things to know:
- **Composite index** on `(restaurant_id, created_at)` serves queries filtering by restaurant, or restaurant plus time — but not time alone. That's the **leftmost-prefix rule**, like a phone book sorted by surname then first name: you can't look up by first name alone.
- **Covering index:** if the index contains every column the query needs, the database never touches the table.
- **Cost:** every insert or update must also update every index, so too many indexes slow down writes. Don't index columns with few distinct values (like a true/false flag).

### 2.3 Replication — copies of the database

One database server is a single point of failure and has a limit on how many queries it can serve. **Replication** keeps copies of the same data on several servers.

The common setup is **leader–follower** (also called primary–replica or master–slave). All **writes** go to the leader. The leader streams its changes to the followers, which apply them. **Reads** can go to any follower. Since most apps read far more than they write (people browse menus far more than they order), adding followers multiplies read capacity.

Two things to understand:
- **Replication lag.** Followers apply changes a little after the leader — usually milliseconds, sometimes seconds under load. So a user who just updated their address and immediately reads it from a follower might see the old one. The fix is **read-your-own-writes**: for a short time after a user writes, send their reads to the leader.
- **Failover.** If the leader dies, a follower is promoted to be the new leader. Changes the old leader hadn't yet sent may be lost — which is why some systems wait for at least one follower to confirm a write (**synchronous** replication) before telling the user it's done. Safer, a bit slower.

### 2.4 Sharding — splitting the data

Replication copies all the data everywhere, so it helps reads but not when the data itself is too big for one machine, or there are too many writes for one leader. Then you **shard** (or **partition**): split the data so each server holds only part of it.

You pick a **shard key** and a rule:
- **Range-based:** users A–F on shard 1, G–M on shard 2… Easy range queries, but uneven — some ranges get far more traffic.
- **Hash-based:** shard = hash(user_id) % N. Spreads data evenly. Range queries across shards become hard.
- **Directory/geo-based:** a lookup table says which shard holds what — e.g. one shard per city. Zomato-style data often splits naturally by city.

Choosing the shard key is the big decision. Pick something that (a) spreads load evenly and (b) keeps data that's queried together on the same shard. For orders, `user_id` keeps a user's history on one shard; `restaurant_id` keeps a restaurant's orders together. You can't have both, which is why systems sometimes store data twice, keyed both ways.

Problems sharding brings:
- **Hot shards:** one key gets huge traffic (a celebrity, a mega-popular restaurant during a sale). Fix: split that key further or cache it.
- **Cross-shard queries and joins** are slow or impossible; design so common queries hit one shard.
- **Resharding:** with `hash % N`, changing N moves almost every key. That's what consistent hashing fixes.

### 2.5 Consistent hashing

With `hash(key) % N`, adding one server changes N, and nearly every key now maps somewhere new — a cache would suddenly miss on everything.

**Consistent hashing** fixes this. Imagine a circle (a ring) of hash values. Place each server on the ring at the position of hash(server name). To find a key's server, hash the key, land on the ring, and walk clockwise to the first server. Now when you add a server, it only takes over the keys between it and the server before it — roughly 1/N of the keys move, not all of them. Remove a server and only its keys move to the next one.

One refinement: with few servers, the gaps on the ring are uneven, so one server gets more keys. **Virtual nodes** fix that — each real server is placed on the ring at many points (say 100), which evens things out.

Used in: distributed caches, Cassandra, DynamoDB, load balancers that need stickiness.

### 2.6 CAP and consistency

When data lives on several machines, the network between them will sometimes fail — a **partition**. The **CAP theorem** says that during a partition you must choose:
- **Consistency (C):** every read returns the latest write — so if a node can't confirm it's up to date, it refuses or waits.
- **Availability (A):** every request gets an answer — even if that answer might be slightly stale.

(P, partition tolerance, isn't optional — networks do fail — so the real choice is C or A *during* a failure.)

Pick per feature, not per company:
- **Payments, wallet balance, seat booking, inventory count** → **consistency.** Better to show an error than double-charge or double-book.
- **Restaurant ratings, review counts, the home feed, "x people ordered this"** → **availability.** A slightly old number is fine; an error page is not.

**Eventual consistency** means: copies may disagree briefly, but if no new writes come in, they all converge to the same value. Most large systems are eventually consistent for most data and strongly consistent only where it matters. (Your Myntra aggregated ratings are exactly this.)

### 2.7 Transactions across services: the saga

Inside one database, a transaction makes "deduct money + create order" atomic. With microservices, the money is in the Payment service's database and the order in the Order service's database. No single transaction covers both.

- **Two-phase commit (2PC):** a coordinator asks every participant "can you commit?", and only if all say yes tells them "commit". Correct, but slow, and if the coordinator dies mid-way participants are stuck waiting. Rarely used across microservices.
- **Saga:** do it as a sequence of local steps, each with an **undo** (a *compensating action*). Order created → payment charged → restaurant confirms. If the restaurant rejects, run the undos backwards: refund the payment, cancel the order. Each step is a normal local transaction; the system is eventually consistent. Sagas are usually driven by events on a queue (*choreography*) or by a central coordinator service (*orchestration*).

This is how real food-ordering systems work.

### Check yourself
**Q: Orders table — SQL or NoSQL?**
A: SQL. Orders involve money and must be correct: you need transactions (create order + items + payment record together), constraints, and queries like "this user's orders" and "this restaurant's orders today". MySQL or Postgres, with replicas for reads and sharding when it gets big.

**Q: What's the difference between replication and sharding?**
A: Replication copies the *same* data to several machines — for read scaling and failover. Sharding *splits* the data so each machine holds a part — for when data or write volume is too big for one machine. Big systems do both: each shard has its own replicas.

**Q: A user updates their address and immediately sees the old one. Why, and how do you fix it?**
A: Replication lag — the read went to a follower that hadn't applied the update yet. Fix with read-your-own-writes: route that user's reads to the leader for a short window after a write (or read that specific data from the leader).

**Q: Why consistent hashing instead of hash % N?**
A: With hash % N, changing the number of servers remaps nearly every key. With consistent hashing, only about 1/N of keys move when a server is added or removed.

**Q: Give one feature that needs consistency and one that's fine with eventual consistency.**
A: Seat booking needs consistency (never sell one seat twice). Restaurant rating can be eventually consistent (a number a few seconds old is harmless).

**Q: Order placed, payment charged, restaurant rejects. How does the system undo it across services?**
A: A saga: each step has a compensating action. The rejection triggers a refund in the Payment service and marks the order cancelled, each as its own local transaction, usually coordinated through events.

---

## Part 3: Caching

### 3.1 The idea

Reading from a database takes milliseconds and costs the database effort. Reading from memory takes microseconds. A **cache** is a fast store (usually in memory) that keeps copies of data you read often, so most requests never reach the database.

Rough speeds worth remembering: reading from RAM ≈ 100 nanoseconds; a network round trip inside a data centre ≈ 0.5 ms; a database query ≈ 1–10 ms; a cross-country request ≈ 50–150 ms. A cache that answers in under a millisecond can take 90%+ of the load off a database.

A **cache hit** is when the data is there; a **miss** is when it isn't and you fetch from the source. The **hit ratio** (hits ÷ total) is the number you watch.

Caches live at several layers: in the browser/app, on the CDN, in each app server's own memory (fastest, but each server has its own copy), and in a shared cache like **Redis** (all servers see the same data).

### 3.2 Caching patterns

- **Cache-aside (lazy loading)** — the most common. The app checks the cache; on a miss it reads the database, stores the result in the cache, and returns it. On a write, it updates the database and **deletes** the cache entry so the next read reloads fresh data. Only data that's actually requested gets cached. Why delete rather than update the cache? Two concurrent writers updating the cache can leave it with the older value; deleting is safer.
- **Write-through:** every write goes to the cache and the database together. The cache is always fresh, but writes are slower and you cache things nobody may read.
- **Write-back (write-behind):** write to the cache only and flush to the database later in batches. Very fast writes, but if the cache dies before flushing, data is lost. Used for things like view counters, never for money.

### 3.3 Expiry and eviction

**TTL (time to live):** each entry expires after a set time. It bounds how stale data can get and cleans up unused entries. A menu might have a 10-minute TTL; a restaurant's "open/closed" status maybe 30 seconds.

**Eviction:** when the cache is full, something must go.
- **LRU (least recently used):** throw out what hasn't been touched for the longest. The default choice. (Implementing an LRU cache — hash map + doubly linked list, O(1) get and put — is a top Zomato DSA question; see `01b`.)
- **LFU (least frequently used):** throw out what's used least often. Better when some items are steadily popular.
- **FIFO:** oldest inserted goes first. Simple, usually worse.

### 3.4 Invalidation — the hard part

"There are only two hard things in computer science: cache invalidation and naming things." The problem: when the database changes, the cached copy is wrong until it's updated or expires. Options, from simple to strong:
- Rely on a short TTL — accept a little staleness.
- Delete the key on every write (cache-aside).
- Publish a "this changed" event that every service holding a copy listens to.

And remember the Myntra lesson: if the same value is cached in two places, both must be invalidated on the same events, or they drift apart.

### 3.5 Three classic cache failures

- **Cache stampede (thundering herd):** a very popular key expires, and in that instant thousands of requests all miss and hit the database together. Fixes: let only one request rebuild the value while the others wait or get the old value (*request coalescing* / a lock); add random **jitter** to TTLs so keys don't all expire at once; refresh hot keys in the background before they expire; **warm up** the cache before launches.
- **Cache penetration:** requests for keys that don't exist (often an attack, e.g. random restaurant IDs). Every one misses the cache and hits the database. Fixes: cache the "not found" result for a short time; or use a **Bloom filter** — a tiny structure that can say "definitely not in the database" — to reject them early.
- **Hot key:** one key gets a huge share of traffic (the restaurant with a viral offer). One cache node melts. Fixes: keep a copy in each app server's local memory for a few seconds; or replicate that key across several cache nodes.

### 3.6 Redis — the cache you'll name in every answer

**Redis** is an in-memory data store. It's fast (100k+ operations per second on one node, sub-millisecond), and it's more than a key-value store — it has data structures that solve common problems directly:
- **Strings** with `INCR` (atomic counters — rate limiting, view counts) and `SET key value NX EX 30` (set only if absent, with expiry — locks and deduplication).
- **Hashes** (an object with fields — a user's cart).
- **Lists** (queues), **sets** (unique items — who liked this).
- **Sorted sets** (items with a score, kept sorted — leaderboards, "top 10 restaurants", ranked feeds).
- **GEO** commands (`GEOADD`, `GEOSEARCH` — "riders within 3 km").
- **Pub/Sub** and **Streams** for messaging.

It runs commands on a single thread, so each command is **atomic** — two clients' `INCR`s can't interleave badly. It can persist to disk (snapshots or an append-only log), and can be run as a cluster with replicas. But treat it as a cache: if losing the data would be a disaster, the source of truth belongs in a database.

### Check yourself
**Q: Walk me through cache-aside on a read and on a write.**
A: Read: check cache → hit, return → miss, read DB, put in cache with a TTL, return. Write: update the DB, then delete the cache key, so the next read loads fresh data.

**Q: Why delete the cache key on write instead of updating it?**
A: If two writes happen close together, their cache updates can land in the wrong order and leave the older value cached. Deleting means the next read always reloads the current value from the database.

**Q: The "top restaurants in Bangalore" key expires at 8 pm and the database falls over. What happened, and how do you prevent it?**
A: A cache stampede: thousands of requests missed at once and all hit the DB. Prevent it by letting one request rebuild the value while others get the old one, refreshing hot keys before they expire, and adding jitter to TTLs.

**Q: What would you store in Redis for Zomato?**
A: Restaurant and menu data (cache), sessions, rate-limit counters, live rider locations (GEO), leaderboards or trending lists (sorted sets), short-lived locks (seat holds, rider assignment), deduplication keys for notifications.

**Q: Someone floods your API with random restaurant IDs that don't exist. What's the problem and fix?**
A: Cache penetration — every request misses the cache and hits the DB. Cache the "not found" result briefly, or check a Bloom filter of valid IDs first, and rate-limit the caller.

---

## Part 4: Doing work later — queues and events

### 4.1 Why you need a queue

When a user places an order, a lot must happen: confirm payment, tell the restaurant, find a rider, send a notification, update analytics, give loyalty points. If the Order service called each of those services directly and waited, the user would wait for all of them, and if *any* one was down, the order would fail.

Instead, the Order service saves the order, puts one message — "order 123 confirmed" — onto a **message queue**, and replies to the user immediately. Each interested service reads that message and does its job on its own time. This is **asynchronous** (async) processing.

What you get:
- **Decoupling:** the Order service doesn't know or care who's listening. Add a new consumer (say, fraud checks) without touching it.
- **Absorbing spikes:** at 8 pm, orders pour in faster than the restaurant-notification service can handle. The queue holds the backlog; workers catch up. Nothing crashes. This is **load levelling**.
- **Resilience:** if the notification service is down for 5 minutes, messages wait in the queue and get processed when it's back.
- **Retries:** failed work can be retried automatically.

When **not** to go async: when the user needs the answer right now — "is my payment successful?", "is this coupon valid?". Those are synchronous calls.

### 4.2 Queue vs pub/sub

- **Point-to-point queue (work queue):** each message is processed by **one** worker, then deleted. Good for jobs: "send this email", "resize this image". Example: **AWS SQS**, RabbitMQ.
- **Publish/subscribe (pub/sub):** each message goes to **every** subscriber group. "Order confirmed" goes to restaurant, dispatch, notification and analytics — each gets its own copy. Example: **Kafka**, Google Pub/Sub, SNS.

### 4.3 Kafka

**Kafka** is the event backbone you'll name in most Zomato answers. Picture it as a set of **append-only logs** on disk:
- A **topic** is a named stream of events ("order-events").
- A topic is split into **partitions** so it can be spread across machines and processed in parallel. Each partition is an ordered log; every message in it gets an increasing number, its **offset**.
- **Producers** append messages. Each message has a **key**; Kafka hashes the key to choose the partition. **Same key → same partition → strict order for that key.** Use `order_id` as the key and all events for one order (placed → accepted → picked up → delivered) are processed in the right order. There's no ordering across partitions.
- **Consumers** read messages and track their own offset (how far they've read). They belong to a **consumer group**. Within a group, each partition is read by exactly one consumer — so 12 partitions allow up to 12 consumers sharing the work. A *different* group (say, analytics) reads the same messages independently. That's how one topic serves many services.
- **Retention:** messages stay for a configured time (e.g. 7 days) whether read or not. So a consumer that crashes can resume where it left off, and you can **replay** old events — e.g. rebuild a broken database from history. That's the big difference from SQS, where messages are deleted once processed.
- **Durability:** each partition is copied to several machines (*replication factor*, often 3). Writes with `acks=all` aren't confirmed until replicas have them.

**Kafka vs SQS in one line:** Kafka is a durable, replayable log for many consumers with per-key ordering and huge throughput; SQS is a simple managed job queue where each message is handled once and deleted.

### 4.4 Delivery guarantees and idempotency

Networks and machines fail at the worst moments. Consider a consumer that processes a message and then crashes before recording that it did. When it restarts, it processes the message again. So there are three possible guarantees:
- **At-most-once:** mark the message done *before* processing. Never duplicated, but can be lost. Fine for unimportant metrics.
- **At-least-once:** mark it done *after* processing. Never lost, but sometimes duplicated. **This is the normal choice.**
- **Exactly-once:** very hard in general; Kafka offers it within Kafka using transactions. In practice, systems use at-least-once plus idempotent processing, which gives the same effect.

**Idempotent** means doing something twice has the same effect as doing it once. "Set status = DELIVERED" is naturally idempotent; "add ₹50 to wallet" is not. To make processing idempotent:
- Give every event or request a unique ID. Keep a record of processed IDs (a database table with a unique constraint, or a Redis key with `SET … NX`). If you've seen it, skip.
- Use database constraints so a duplicate insert fails harmlessly.
- Write updates that are safe to repeat: `UPDATE orders SET status='PICKED_UP' WHERE id=? AND status='ASSIGNED'` does nothing the second time.

**Say "idempotency" in every design that involves payments, orders or queues.** It's the single most valued word in these interviews.

### 4.5 Retries, dead letters and backpressure

- **Retries with exponential backoff and jitter:** when a call fails, wait and try again — 1 s, then 2 s, 4 s, 8 s — plus a random bit (*jitter*) so thousands of clients don't all retry at the same instant and knock the recovering service over again.
- **Dead-letter queue (DLQ):** after N failed attempts, move the message to a separate queue so one broken message ("poison message") doesn't block everything. Alert on it, inspect it, fix and replay.
- **Backpressure:** when consumers can't keep up, don't let producers overwhelm them. With a queue, the backlog just grows in the queue (which is fine for a while); you watch the **consumer lag** (how far behind consumers are) and add consumers or slow producers.

### 4.6 The outbox pattern

Here's a subtle bug. The Order service does two things: save the order in its database, and publish "order confirmed" to Kafka. These are two different systems. If the service crashes after saving but before publishing, the order exists but nobody is ever told — the restaurant never sees it. If it publishes first and then the database save fails, everyone hears about an order that doesn't exist.

**The outbox pattern:** in the **same database transaction** that saves the order, also insert a row into an `outbox` table ("event: order 123 confirmed"). Both happen or neither does. A separate small process (a *relay*) reads new outbox rows and publishes them to Kafka, marking them sent. If it crashes, it re-sends on restart — a possible duplicate, which idempotent consumers handle.

### Check yourself
**Q: Which steps of placing an order are synchronous and which async?**
A: Synchronous: validate the cart and prices, check the coupon, start the payment — the user needs these answers right now. Async (via Kafka): notify the restaurant, start rider assignment, send push notifications, update analytics and loyalty points.

**Q: How do you keep all events of one order in order in Kafka?**
A: Use the order ID as the message key. Kafka sends all messages with the same key to the same partition, and a partition is strictly ordered and read by one consumer in a group.

**Q: Your consumer processed a "credit ₹100 refund" message, then crashed before committing its offset. What happens and how do you stay correct?**
A: On restart it reads the message again (at-least-once delivery). To avoid a double refund, the handler is idempotent: it records the refund ID with a unique constraint, and skips it if it's already there.

**Q: Explain the outbox pattern and the problem it solves.**
A: Writing to the database and publishing to a queue are two systems, so a crash between them loses or invents events. Save the business change and an outbox row in one DB transaction; a relay publishes outbox rows to the queue. Events are never lost; duplicates are possible and handled by idempotent consumers.

**Q: Kafka or SQS for sending order events to five different services?**
A: Kafka. Each service is its own consumer group and gets every event; ordering per order is kept by keying on order ID; events can be replayed. SQS would need one queue per service (or SNS in front to fan out), and has no replay.

---

## Part 5: APIs and real-time updates

### 5.1 REST basics

An **API** is the contract between client and server. Most are **REST**: resources have URLs, and HTTP methods are the verbs.

```
GET    /restaurants?lat=12.9&lng=77.6&cursor=abc   list nearby restaurants
GET    /restaurants/42/menu                        one restaurant's menu
POST   /orders                                     place an order
GET    /orders/123                                 order status
POST   /orders/123/cancel                          cancel
```

- **GET** reads, **POST** creates, **PUT** replaces, **PATCH** partially updates, **DELETE** removes.
- **Idempotent methods:** GET, PUT and DELETE are safe to repeat. POST is not — repeat it and you create two orders.
- **Status codes:** 200 OK, 201 Created, 202 Accepted (queued for later), 400 Bad Request, 401 not logged in, 403 not allowed, 404 Not Found, 409 Conflict, 429 Too Many Requests, 500 server error, 503 unavailable.

### 5.2 Idempotency keys

The user taps "Place order", the network drops before the response arrives, the app retries. Did the first one go through? Without protection, they might be charged twice.

The client generates a unique **idempotency key** (a random ID) for each checkout attempt and sends it in a header with every retry: `Idempotency-Key: 7f3a…`. The server stores the key with the result. If the same key comes again, it returns the stored result instead of creating a new order. Payment providers like Stripe and Razorpay work exactly this way. In the database: a `UNIQUE` constraint on `idempotency_key` in the orders table.

### 5.3 Pagination

Never return 10,000 restaurants in one response.
- **Offset pagination:** `?page=5&size=20` → `LIMIT 20 OFFSET 80`. Simple, but the database still walks past the first 80 rows (slow for deep pages), and if new items are inserted while you scroll, you see duplicates or skip items.
- **Cursor pagination:** the response includes a cursor (e.g. the last item's ID or score); the next request says `?after=<cursor>` → `WHERE id > last_id LIMIT 20`. Fast at any depth and stable while data changes. **Use cursor for feeds and lists.**

### 5.4 Real-time: how the app sees the rider move

The order-tracking screen needs updates every few seconds. Four ways to do it:
- **Polling:** the app asks "any update?" every 5 seconds. Simple, works everywhere, but wasteful — most answers are "no change" — and updates arrive up to 5 s late.
- **Long polling:** the app asks, and the server **holds the request open** until there's news (or a timeout), then responds; the app immediately asks again. Fewer wasted requests, near-instant updates.
- **Server-Sent Events (SSE):** one long-lived HTTP connection over which the **server pushes** updates to the client. One direction only (server → client), simple, reconnects automatically. Good for live status and scores.
- **WebSockets:** a persistent **two-way** connection. Both sides can send any time. Best for chat, or when the client also streams data (the rider app sending its location).

For order tracking, SSE or WebSockets for the customer; the rider app sends location via plain HTTP calls or a WebSocket. Mention that these connections are **stateful** — a user is connected to one specific server — so you need a way to route a message to the right server (e.g. a Redis pub/sub channel per order that all gateway servers subscribe to).

### 5.5 Push notifications

When the app is closed, none of the above works. Then you use the platform's push services — **FCM** (Firebase Cloud Messaging, Android) and **APNs** (Apple). Your server sends the message to FCM/APNs with the device's token; they deliver it to the phone.

### Check yourself
**Q: The user double-taps "Pay". How do you make sure they're charged once?**
A: The client sends the same idempotency key with both requests. The server stores the key with a unique constraint; the second request finds it and returns the first result instead of charging again.

**Q: Offset or cursor pagination for the restaurant list?**
A: Cursor. It stays fast at any depth and doesn't duplicate or skip items when the list changes while the user scrolls.

**Q: How would you push live rider location to the customer?**
A: The rider app sends its location every few seconds to a location service, which stores the latest position in Redis and publishes it on a channel for that order. The customer's app holds a WebSocket or SSE connection to a gateway server, which is subscribed to that channel and forwards each update.

**Q: Polling vs WebSocket — trade-off?**
A: Polling is simple and stateless but wasteful and laggy. WebSockets are instant and efficient but keep a stateful connection per user, which needs connection-aware routing and more careful scaling.

---

## Part 6: Keeping it running — reliability

### 6.1 Timeouts

Every network call can hang. If your service waits forever on a slow dependency, its threads pile up waiting, it runs out of them, and it stops responding too — and then the services calling *it* stall. One slow service takes down a chain: a **cascading failure**. So **every call gets a timeout** — based on how long that dependency normally takes at worst (its p99), plus a bit.

### 6.2 Retries — carefully

Retry transient failures (timeouts, 503s) with exponential backoff and jitter, a maximum number of attempts, and **only for idempotent operations** (or with an idempotency key). Never blindly retry "charge the card".

### 6.3 Circuit breaker

If a dependency is clearly down, retrying just wastes time and piles more load on it. A **circuit breaker** wraps calls to it and counts failures:
- **Closed** (normal): calls go through.
- After too many failures it **opens**: calls fail immediately (or go to a fallback) without even trying. This protects your service and gives the dependency room to recover.
- After a cool-down it goes **half-open**: lets a few test calls through. If they succeed, it closes; if not, it opens again.

### 6.4 Graceful degradation and fallbacks

When something non-essential fails, keep the core working. If the recommendations service is down, show a generic popular list instead of an error. If the ETA model is down, show a rough distance-based ETA. On New Year's Eve, switch off heavy non-critical features to keep ordering fast. **The core path — browse, order, pay, track — must survive.**

### 6.5 Bulkheads

On a ship, bulkheads are walls that stop one flooded compartment sinking the whole ship. In software: give each dependency its own limited pool of threads or connections, so if one dependency gets slow, it only exhausts its own pool and the rest of the service keeps working.

### 6.6 Rate limiting

Cap how many requests a client can make per time window. It protects you from abuse, bugs and traffic spikes, and keeps one heavy user from hurting everyone. The common algorithms:
- **Token bucket:** a bucket holds up to B tokens, refilled at R per second; each request takes one; empty bucket → reject (HTTP 429). Allows short bursts, caps the average. The most popular choice.
- **Leaky bucket:** requests queue up and are processed at a fixed rate. Perfectly smooth output.
- **Fixed window:** a counter per minute; reset each minute. Very simple, but allows a double burst at the edge (100 at 11:59:59 and 100 at 12:00:00).
- **Sliding window:** fixes the edge problem by looking at the last 60 seconds rather than the current calendar minute.

Across many servers, keep the counters in Redis so all servers share them. Full design in `07`.

### 6.7 Redundancy and failover

Remove single points of failure: multiple app servers, two load balancers, database replicas with automatic failover, data replicated across **availability zones** (separate data centres in the same region), and for the most critical systems, across regions.

**Availability numbers:** "three nines" (99.9%) allows about 8.8 hours of downtime a year; "four nines" (99.99%) about 53 minutes. Each extra nine costs a lot more.

### 6.8 Observability

You can't fix what you can't see.
- **Logs:** records of individual events ("order 123 failed: payment timeout").
- **Metrics:** numbers over time — requests per second, error rate, latency, queue length, cache hit ratio.
- **Traces:** one request's path across all services, with timing at each hop — shows you which service made it slow.

Latency is measured in **percentiles**: **p50** is the median; **p99** means 99% of requests are faster than this. Averages hide pain — if p99 is 3 seconds, 1 in 100 users waits 3 seconds, and at Zomato's scale that's a lot of angry people. Set **alerts** on what users feel: error rate, p99 latency, consumer lag.

### Check yourself
**Q: The payment provider is slow and your order service starts timing out everywhere. What went wrong and what protects you?**
A: A cascading failure — threads blocked waiting on the slow provider until the order service ran out. Protections: tight timeouts, a circuit breaker that fails fast once the provider is clearly unhealthy, a bulkhead (separate connection pool) for provider calls, and a clear "payment pending" state rather than blocking the whole request.

**Q: When is it unsafe to retry?**
A: When the operation isn't idempotent — like charging a card or adding money — unless you send an idempotency key so the server can detect the duplicate.

**Q: Why does p99 matter more than average latency?**
A: Averages hide the slow tail. p99 tells you what the unluckiest 1% experience, and at millions of requests that's many users, often your most active ones.

**Q: On New Year's Eve, what would you turn off first?**
A: Non-critical features: recommendations, review-writing, heavy analytics, fancy banners. Keep browse, cart, payment, order and tracking fast.

---

## Part 7: Two people, one thing — concurrency

### 7.1 Race conditions

Two requests read the same data, both make a decision, both write — and one overwrites the other or both "win". Examples: two users booking the last seat; two orders assigned to the same rider; two coupon uses when only one is allowed; stock going negative. This is a **race condition**, and design interviews love them.

### 7.2 Atomic conditional updates — the simplest fix

Let the database do the check and the change in one step:

```sql
UPDATE inventory SET qty = qty - 1 WHERE item_id = 7 AND qty > 0;
-- check affected rows: 1 = success, 0 = out of stock
```

```sql
UPDATE riders SET status = 'BUSY', order_id = 123
WHERE id = 55 AND status = 'AVAILABLE';
-- 1 row = you got the rider; 0 = someone else did, try the next one
```

The database guarantees only one of two concurrent updates can match. No separate read, no gap for a race.

### 7.3 Pessimistic vs optimistic locking

- **Pessimistic locking:** lock the row before working with it, so nobody else can touch it until you're done. `SELECT … FOR UPDATE` inside a transaction. Safe and simple; but others wait, and under heavy contention it slows down (and badly-ordered locks can deadlock).
- **Optimistic locking:** don't lock; instead keep a `version` number on the row. Read it (say version 5), do your work, then `UPDATE … SET …, version = 6 WHERE id = ? AND version = 5`. If someone else changed it first, the version no longer matches, 0 rows update, and you retry or report a conflict. Great when conflicts are rare.

Rule of thumb: optimistic for low-contention data (editing your profile, a restaurant updating its menu); pessimistic or atomic updates for hot, contended items (the last seat, a flash-sale item).

### 7.4 Distributed locks

Sometimes the thing being protected isn't a single DB row — e.g. "only one worker should run the nightly payout job", or "hold these seats for 10 minutes". A **distributed lock** in Redis: `SET lock:seat:A12 user42 NX EX 600` — set only if it doesn't exist, expire after 600 s. Whoever succeeds holds the lock; the expiry means a crashed holder can't block it forever. To release, delete it **only if you still own it** (check the value — usually with a small Lua script so check-and-delete is atomic).

Caveat worth mentioning: if the holder pauses longer than the expiry (a long garbage-collection pause), two holders can briefly think they own it. For critical correctness, also use a **fencing token** — an increasing number issued with each lock, which the database checks — or rely on the database's own constraints.

### Check yourself
**Q: Two customers tap "Book" on the last seat at the same time. How do you guarantee only one gets it?**
A: An atomic conditional update: `UPDATE show_seats SET status='HELD', held_by=? WHERE show_id=? AND seat_id=? AND status='AVAILABLE'`. Only one update can match; the other gets 0 rows and is told the seat is gone.

**Q: Optimistic or pessimistic locking for a restaurant editing its menu?**
A: Optimistic — two people rarely edit the same item at the same time, so add a version column and reject the stale update if it happens.

**Q: Why does a Redis lock need an expiry, and why must release check ownership?**
A: The expiry stops a crashed holder blocking everyone forever. The ownership check stops you deleting a lock someone else acquired after yours expired.

---

## Part 8: Location — "what's near me?"

Most of Zomato is location: restaurants near you, riders near a restaurant, which zone an address is in. The naive query — compute the distance from the user to every restaurant and sort — is far too slow across lakhs of restaurants.

### 8.1 Geohash

A **geohash** turns a latitude/longitude into a short string by repeatedly cutting the world into halves and quarters. Each extra character zooms in: about 5 characters ≈ a 5 km square, 6 ≈ 1.2 km × 0.6 km, 7 ≈ 150 m. The key property: **nearby places share a prefix.** "tdr1y" and "tdr1v" are neighbours.

So "restaurants near me" becomes: compute the user's geohash at the right precision, then look up restaurants whose geohash starts with that prefix — an ordinary indexed string lookup. Catch: you could be standing at the edge of a cell, with the nearest restaurant just across the border in a different cell. So you always search **your cell plus its 8 neighbours**, then compute exact distances for that small set and sort.

### 8.2 Other options
- **Quadtree:** a tree where each node is a square that splits into four smaller squares when it holds too many points. Dense city centres get split finely; empty areas stay as big squares. Good for in-memory "find nearby".
- **H3 (Uber's hexagon grid):** the world divided into hexagons at many resolutions. Hexagons have the nice property that all neighbours are the same distance away, which makes things like delivery zones and surge pricing areas cleaner. Many delivery companies use hex grids.
- **Database support:** PostGIS (Postgres) with spatial indexes, Elasticsearch `geo_distance` queries, MongoDB geo indexes.
- **Redis GEO:** `GEOADD riders 77.59 12.97 rider:55` and `GEOSEARCH riders FROMLONLAT 77.6 12.9 BYRADIUS 3 km ASC`. Perfect for **moving things** like riders, because updates are cheap and everything is in memory.

**Rule of thumb for answers:** static things (restaurants) → geohash/PostGIS/Elasticsearch with a spatial index; fast-moving things (riders, updating every few seconds) → Redis GEO or an in-memory grid, never written to the main database on every ping.

### Check yourself
**Q: How do you find restaurants within 3 km of a user quickly?**
A: Store each restaurant's geohash (indexed). Compute the user's geohash at a precision whose cell is about the search radius, fetch restaurants in that cell and its 8 neighbours, compute exact distances, filter to 3 km, then rank.

**Q: Why search neighbouring cells too?**
A: The user may be near a cell border; the closest restaurant can be in the next cell even though it's only 100 m away.

**Q: Riders update location every 4 seconds. Where do you store it?**
A: Latest position in Redis GEO (in memory, cheap updates, radius search built in). Optionally stream pings to Kafka for history and analytics, stored in a time-series or columnar store — never one database write per ping on the main orders database.

---

## Part 9: Search

"Biryani", "pizza near me", "Truffles" — search over restaurants and dishes. A SQL `LIKE '%biryani%'` can't use an index and won't handle typos or ranking.

**Inverted index:** the structure behind every search engine. Instead of "document → words", store "word → list of documents containing it". `biryani → [r12, r45, r88]`, `chicken → [r12, r19]`. A search for "chicken biryani" intersects the two lists in milliseconds. Before indexing, text is processed: lower-cased, split into words (*tokens*), reduced to root forms ("biryanis" → "biryani"), and sometimes expanded with synonyms.

**Elasticsearch** is the standard tool: a distributed search engine built on inverted indexes, with full-text search, typo tolerance (*fuzzy matching*), filters (veg only, rating ≥ 4, open now), geo-distance filters and relevance scoring. Typical setup: the source of truth stays in MySQL; changes flow to Elasticsearch through events (Kafka or *CDC* — change data capture, which streams the database's change log), so the search index is eventually consistent.

**Ranking** combines text relevance with business signals: distance, rating, delivery time, popularity, whether it's open, personal history.

**Autocomplete (typeahead):** as you type "bir…", show suggestions. Use a **trie** (prefix tree) of popular queries with the top few results pre-stored at each node, or Elasticsearch's completion suggester; keep it in memory and cache hot prefixes. Debounce on the client (wait ~200 ms after typing stops before calling).

### Check yourself
**Q: Why not use SQL `LIKE` for restaurant search?**
A: `LIKE '%x%'` can't use a normal index, so it scans the whole table, and it can't do typos, word roots or relevance ranking. An inverted index (Elasticsearch) does all of that fast.

**Q: How does a menu change in MySQL reach the search index?**
A: The menu service publishes a change event (or CDC streams the DB change log) to Kafka; an indexer consumes it and updates Elasticsearch. Search is eventually consistent — a few seconds behind.

---

## Part 10: Files and media

Images, menus as PDFs, review photos, invoices: big binary files. Don't put them in the database (it bloats backups and slows everything). Put them in **object storage** — Amazon **S3** or Google Cloud Storage — which stores any file by a key, cheaply, durably (copies across data centres) and at unlimited scale. Store only the file's key/URL in the database.

**Uploads with pre-signed URLs:** instead of the app sending a 5 MB photo through your servers, your server gives the app a **pre-signed URL** — a temporary link that allows uploading one specific file directly to S3 for a few minutes. The app uploads straight to S3; your servers never carry the bytes. After upload, an event triggers async work: virus scan, resize into thumbnails, moderation.

**Downloads** go through a **CDN** so images load from a server near the user.

---

## Part 11: Back-of-the-envelope estimation

Interviewers want rough numbers to justify choices. The handy facts:
- **1 day ≈ 86,400 s ≈ 10⁵ s** (round it).
- **Requests per second (RPS/QPS)** = requests per day ÷ 10⁵. Then multiply by a **peak factor** (food delivery peaks hard at lunch and dinner — assume 3–5× the average, more on big days like NYE).
- Sizes: a character ≈ 1 byte; a typical order row ≈ 1 KB; a photo ≈ 200 KB–2 MB.
- What one machine can roughly do: a well-tuned MySQL server handles a few thousand writes and tens of thousands of simple reads per second; one Redis node 100k+ operations per second; one app server hundreds to a few thousand requests per second depending on work.

**Worked example (assumed numbers):** 20 lakh (2 million) orders a day.
- Average = 2×10⁶ ÷ 10⁵ = **20 orders/s**. Peak at 5× = **100 orders/s**. One well-indexed MySQL primary can handle that write load; the challenge is everything else around it.
- Each order causes maybe 20 status reads and many menu/search reads — browsing is perhaps 50–100× orders, so **thousands of reads per second** → cache and replicas.
- Live tracking: say 3 lakh active deliveries at peak, each rider pinging every 4 s → 3×10⁵ ÷ 4 ≈ **75,000 location writes/s**. That's why pings go to Redis/Kafka, not MySQL.
- Storage: 2×10⁶ orders × 1 KB = 2 GB a day ≈ 730 GB a year — fine for sharded MySQL, with old data archived.

Say the numbers, then say what they imply. That's the whole point.

---

## Part 12: Security basics for design answers

- **Authentication (who are you?)** vs **authorisation (what may you do?).** Login proves identity; roles and ownership checks decide access (a restaurant owner can edit only their own menu).
- **Tokens:** after login, the server issues a signed token (a **JWT**: header.payload.signature). The client sends it on every request; any server can verify the signature without a session lookup. Keep tokens short-lived with a refresh token for renewal.
- **HTTPS everywhere:** encrypts traffic so no one on the network can read or alter it.
- **Never trust the client:** recompute prices and totals on the server; never accept "amount = ₹1" from the app.
- **Secrets** (DB passwords, API keys) live in a secrets manager, not in code.
- **Rate limit** login and OTP endpoints to block brute force.
- **PII** (phone numbers, addresses) — limit who can see it. Delivery apps commonly mask phone numbers between customer and rider by routing calls through a proxy number, so neither sees the other's real number.

---

## Part 13: Low-level design (LLD) — designing the classes

Some rounds ask for **LLD** instead of (or as well as) high-level design: "design a parking lot", "design the menu with customisations", "design a cart with coupons". Now you're designing **classes and their relationships**, in code-like detail.

### 13.1 How to answer an LLD question
1. **Clarify requirements and scope** (what must it support? which features are out?).
2. **Find the entities** — the nouns: ParkingLot, Floor, Spot, Vehicle, Ticket.
3. **Find the behaviours** — the verbs: park, unpark, calculate fee.
4. **Define relationships:** *has-a* (a Floor has Spots — composition) vs *is-a* (a Car is a Vehicle — inheritance). Prefer composition; use inheritance only for real "is-a".
5. **Apply a design pattern where it genuinely fits** (below).
6. **Write class skeletons** with key fields and method signatures, and code one or two core methods fully.
7. **Mention concurrency** (two cars, one spot → lock or atomic assignment) and extensibility.

### 13.2 OOP and SOLID in one breath
- **Encapsulation** (hide fields behind methods), **abstraction** (expose what, hide how), **inheritance** (reuse via is-a), **polymorphism** (one interface, many implementations).
- **SOLID:**
  - **S**ingle responsibility: a class has one reason to change (the Order class shouldn't also send emails).
  - **O**pen/closed: add new behaviour by adding classes, not editing old ones (new payment method = new class).
  - **L**iskov substitution: a subclass must work anywhere its parent is expected.
  - **I**nterface segregation: small focused interfaces rather than one fat one.
  - **D**ependency inversion: depend on interfaces, not concrete classes (OrderService depends on `PaymentGateway`, not `RazorpayClient`), so implementations can be swapped and tested.

### 13.3 The patterns that come up
- **Strategy:** pluggable algorithms behind one interface. `PricingStrategy` (normal, surge, festival), `DiscountStrategy` (flat ₹100, 20% up to ₹150), `AssignmentStrategy` (nearest rider, least busy). Switch behaviour without `if-else` chains.
- **Factory:** one place that creates the right object type. `NotificationFactory.create("SMS")` returns an `SmsNotification`.
- **Observer:** objects subscribe to events and get notified. When an order's status changes, the customer app, restaurant and notification service (observers) are told. (Pub/sub is the distributed version.)
- **State:** an object's behaviour depends on its state, with each state as a class that knows its allowed transitions. Orders, elevators, vending machines.
- **Singleton:** exactly one instance (a config manager, a connection pool). Use sparingly; make it thread-safe.
- **Decorator:** wrap an object to add behaviour. A pizza decorated with toppings, each adding to the price.
- **Builder:** construct complex objects step by step (an order with many optional parts).

### Check yourself
**Q: You need several discount types, and marketing adds a new one every month. How do you design it?**
A: The Strategy pattern: a `Discount` interface with `apply(cart) → amount`, and one class per type (`FlatDiscount`, `PercentDiscount`, `BuyOneGetOne`). New types are new classes; the cart code never changes (open/closed principle).

**Q: Which pattern models an order moving through placed → preparing → delivered?**
A: The State pattern (or a state machine table): each state defines which transitions are allowed, and one method performs a transition, rejecting illegal moves and recording history.

**Q: Has-a or is-a: a Restaurant and its Menu?**
A: Has-a (composition): a Restaurant has a Menu. A Menu isn't a kind of Restaurant.

---

## Part 14: Putting it together — the shape of Zomato

You now have every building block. Here's how they fit for the system you'll most likely be asked about. (`07` walks through it fully.)

```
Customer app / Restaurant app / Rider app
        │  HTTPS, WebSocket
        ▼
   CDN (images)          API Gateway: auth, rate limiting, routing
                                │
   ┌──────────┬──────────┬──────┴─────┬────────────┬─────────────┬────────────┐
   ▼          ▼          ▼            ▼            ▼             ▼            ▼
 User      Restaurant  Search      Cart &       Payment      Delivery /    Location
 service   & Menu      (Elastic-   Order        service      Dispatch      service
           service     search)     service                   service       (rider pings)
   │          │          ▲            │            │             │            │
 MySQL     MySQL +      │          MySQL        MySQL         MySQL         Redis GEO
           Redis cache  │          (sharded)    (ledger)                    + Kafka
                        │             │            │             │
                        └─────────────┴──── Kafka (order-events, payment-events, …) ──┐
                                                                                      ▼
                                                   Notification service · Analytics · ETA (ML)
```

The story to tell: the customer searches (Search service on Elasticsearch, geo-filtered), opens a restaurant (Menu service, heavily cached in Redis), builds a cart, places the order with an idempotency key (Order service, MySQL, state machine), pays (Payment service, synchronous call to a gateway, then webhook confirmation), and the order service publishes `OrderConfirmed` to Kafka via an outbox. The restaurant app is notified, Dispatch finds a rider near the restaurant using Redis GEO and assigns them atomically, the rider's pings flow through the Location service to Redis and out to the customer over WebSocket, and Notification sends pushes at each step. Every arrow has a failure story: timeouts, retries, idempotent consumers, DLQs, circuit breakers, and a degraded-but-working mode at peak.

---

## Part 15: Final quiz — 25 rapid-fire questions

Cover the answers, say yours out loud, then check.

1. **What does a load balancer do besides spreading traffic?** Health checks and removing dead servers; often HTTPS termination and routing by path.
2. **What makes a server stateless?** It keeps no per-user data in its own memory between requests; state lives in a shared DB/cache or a token.
3. **When would you choose NoSQL over SQL?** Huge write volume, simple key-based access, flexible schema, easy scale-out — and no need for multi-row transactions.
4. **What's the leftmost-prefix rule?** A composite index on (a, b) helps queries on a, or a and b, but not b alone.
5. **What's replication lag and one fix?** Followers trail the leader; route a user's reads to the leader right after they write.
6. **What's a hot shard and a fix?** One shard getting outsized traffic due to a popular key; split the key further, cache it, or rebalance.
7. **CAP in one line?** During a network partition you choose between consistency and availability.
8. **What is eventual consistency?** Copies may disagree briefly but converge when writes stop.
9. **Cache-aside on write?** Update DB, then delete the cache key.
10. **Name three cache failure modes.** Stampede, penetration, hot key.
11. **LRU vs LFU?** Evict least recently used vs least frequently used.
12. **Queue vs pub/sub?** Queue: one consumer per message. Pub/sub: every subscriber group gets every message.
13. **How does Kafka keep order?** Per partition; same key goes to the same partition.
14. **What limits a Kafka consumer group's parallelism?** The number of partitions.
15. **At-least-once means you must…?** Make consumers idempotent.
16. **What's a DLQ?** A queue where messages go after repeated failures, so they don't block others.
17. **Why the outbox pattern?** To make "save to DB" and "publish event" atomic via one DB transaction plus a relay.
18. **Which HTTP methods are idempotent?** GET, PUT, DELETE (not POST).
19. **How do you make POST /orders safe to retry?** An idempotency key with a unique constraint.
20. **SSE vs WebSocket?** SSE is server-to-client only over HTTP; WebSocket is two-way.
21. **What does a circuit breaker do?** Stops calling a failing dependency for a while, fails fast, then tests recovery.
22. **Token bucket in one line?** Tokens refill at a steady rate up to a cap; each request takes one; allows bursts, caps the average.
23. **Optimistic locking how?** Version column; update only if the version hasn't changed.
24. **Nearby search structure?** Geohash (cell + 8 neighbours), quadtree, H3, or Redis GEO for moving riders.
25. **Daily requests to per-second?** Divide by ~10⁵, then multiply by a peak factor.

If you can answer all 25 without looking, you have the foundations. Now go to `07` and work through the designs.
