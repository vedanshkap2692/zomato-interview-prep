# CS Fundamentals, SQL & Design: Zomato Round 1

> The sections are ordered by **how often Zomato asked them**. DBMS comes first: it is the single most-probed area. Each answer is written to be *said out loud* in 30–90 seconds.
> "(Asked N×)" = reported in N independent Zomato/Blinkit/District interviews.

---

## PART A: DBMS (highest priority)

### A1. SQL vs NoSQL (asked 12+×)

| | SQL (MySQL, Postgres) | NoSQL |
|---|---|---|
| Data model | Tables, fixed schema, relations via foreign keys | Key-value (Redis, Aerospike, DynamoDB), document (MongoDB), wide-column (Cassandra), graph (Neo4j) |
| Transactions | ACID, multi-row and multi-table | Usually per-key/per-document atomicity; often eventual consistency (tunable) |
| Joins | Native | Mostly none; you denormalise or join in the application. MongoDB has `$lookup`, but it is limited. |
| Scaling | Vertical first; read replicas; sharding is manual or hard | Built for horizontal scaling and partitioning |
| Query | Flexible ad-hoc queries | Access-pattern-driven design: you design for known queries |

**When to use which (Zomato examples):**

| Data | Choice | Why |
|---|---|---|
| Orders, payments, refunds, user accounts | **SQL** | Money needs ACID and relations |
| Session data, menu cache, rider live location, rate-limit counters | **Redis / KV store** | Low latency, TTLs |
| Restaurant search (text, filters, geo) | **Elasticsearch** | Inverted index, geo queries |
| Event streams (order events, rider pings) | **Kafka** | Durable, replayable log |
| Reviews / flexible-schema content | Document store **or** SQL with a JSON column | Depends on the access pattern |

**Follow-ups:**
- *"Can NoSQL do joins?"* Not in general. Data is denormalised or joined in the app. MongoDB `$lookup` exists but is costly.
- *"What happens if you use only Redis and no DB?"* Redis is in-memory. Persistence (RDB snapshots, AOF logs) can lose the last few seconds of writes, memory is expensive, there are no relational queries, and transactions are limited. It's fine as a cache, risky as the system of record for money.
- *"How do you scale SQL?"*
  1. Indexes and query tuning.
  2. A cache in front (Redis).
  3. **Read replicas** (async replication, with replication lag).
  4. Vertical scaling.
  5. Partitioning (a table split by date).
  6. **Sharding** by key (user_id / city_id). Its costs: cross-shard joins and transactions, resharding, hot shards. Consistent hashing helps with resharding.

### A2. MySQL vs PostgreSQL (one candidate was rejected for not justifying this)

| | MySQL (InnoDB) | PostgreSQL |
|---|---|---|
| Strength | Simple, very fast for read-heavy OLTP; huge ecosystem; easy replication | Feature-rich: advanced SQL, CTEs, window functions, **JSONB**, **PostGIS** (geo), partial and expression indexes, extensibility |
| Concurrency | MVCC via undo logs; clustered primary key | MVCC via row versions (needs VACUUM) |
| Index | Table **clustered on the primary key**; secondary indexes store the PK | Heap table; every index points to the heap (TID) |
| Default isolation | REPEATABLE READ | READ COMMITTED |
| Pick when | Simple, high-throughput read/write OLTP (Myntra used MySQL) | Complex queries, geo (nearby restaurants with PostGIS), JSON, strict SQL |

> **Your line:** "At Myntra we used MySQL, which is a good fit for high-QPS key lookups by style ID. If I needed geo queries, like nearby restaurants, I'd lean towards Postgres with PostGIS."

### A3. Indexing internals (asked 10+×), *the most important DB topic*

**What an index is:** a separate data structure (usually a **B+ tree**) that maps column values to row locations. Lookups become O(log n) instead of a full scan.

**Why a B+ tree, and not a BST, hash or sorted array?**
- **High fan-out.** Each node is one disk page (about 16 KB in InnoDB) holding hundreds of keys, so the tree is only 3–4 levels deep even for a billion rows, which means 3–4 disk reads per lookup. A BST would be about 30 levels deep (30 disk reads).
- **All data/pointers live in the leaves, and the leaves are linked.** That makes **range scans** (`BETWEEN`, `ORDER BY`, `>`) fast.
- **A hash index** gives O(1) equality lookups but **can't do range queries** or ordering.
- **A sorted array** gives O(log n) search, but inserts cost O(n) shifts. A B+ tree insert is O(log n), with occasional node splits.
- *"Effect of order k?"* Higher order means more keys per node, so a shallower tree and fewer I/Os. Height is about log_k(n). Each node is sized to a disk page.
- *Insert complexity:* O(log n). Find the leaf and insert. If the leaf overflows, split it and push the middle key up; splits can cascade to the root, which is the only way the height grows.

**Index types:**

| Type | Meaning |
|---|---|
| **Clustered** | The table rows themselves are stored in index order. Only one per table. In InnoDB it is the primary key. |
| **Non-clustered / secondary** | A separate structure holding (key → PK or row pointer). In InnoDB a secondary lookup is index → PK → clustered index, which is a "double lookup". |
| **Composite** | An index on (a, b, c), sorted by a, then b, then c |
| **Covering** | The index holds every column the query needs, so the table is never touched ("Using index" in EXPLAIN) |
| **Unique** | Enforces uniqueness |
| **Dense vs sparse** | Dense: an entry for every key. Sparse: an entry per block. |

**Composite index and the leftmost-prefix rule.** An index on `(city_id, cuisine, rating)` can serve:
- ✅ `WHERE city_id = 1`
- ✅ `WHERE city_id = 1 AND cuisine = 'X'`
- ✅ `WHERE city_id = 1 AND cuisine = 'X' ORDER BY rating`
- ❌ `WHERE cuisine = 'X'`: the leftmost column is missing.
- ⚠️ `WHERE city_id = 1 AND rating > 4`: the index is used for `city_id` only, because the gap at `cuisine` stops it there.
- ⚠️ A range on an earlier column stops the use of later columns for seeking.

**"How does `WHERE A = 10 AND B = 20` run with two separate indexes vs one composite index?"** (asked in a 2024 intern round)
- **Two single-column indexes:**
  - The optimiser usually picks **the more selective one**, then filters the other condition on the fetched rows.
  - Or it may use an **index merge (intersection)**: scan both indexes, intersect the row IDs, then fetch.
  - Both cost more than a single composite seek.
- **A composite index on (A, B):** one B+ tree seek straight to the matching entries, with the fewest rows touched. Best.
- **Column order:** put the equality columns first, and the more selective column first when the access patterns allow it.

**When indexes hurt:**
- Every INSERT/UPDATE/DELETE must update every index, so writes slow down.
- They take storage.
- Low-cardinality columns (gender, boolean) are barely helped.
- Too many indexes confuse the optimiser.
- Applying a function to the column (`WHERE YEAR(created) = 2025`) or a leading wildcard (`LIKE '%abc'`) prevents index use.

**`EXPLAIN`:** check `type` (ALL = full scan is bad; ref/range/const are good), `key` (which index was chosen), `rows` (the estimate) and `Extra` (Using index / Using filesort / Using temporary).

### A4. ACID, transactions and isolation

| Property | Meaning | How (InnoDB) |
|---|---|---|
| **Atomicity** | All or nothing | Undo log for rollback |
| **Consistency** | Constraints hold before and after | Constraints, FKs, application logic |
| **Isolation** | Concurrent transactions don't interfere | Locks + MVCC |
| **Durability** | Committed data survives a crash | Redo log (write-ahead log), fsync on commit |

**Read anomalies, and which isolation level prevents each:**

| Isolation level | Dirty read | Non-repeatable read | Phantom read |
|---|---|---|---|
| Read Uncommitted | possible | possible | possible |
| Read Committed (Postgres default) | ✗ | possible | possible |
| Repeatable Read (MySQL default) | ✗ | ✗ | possible in theory; InnoDB blocks most phantoms with **gap/next-key locks** |
| Serializable | ✗ | ✗ | ✗ |

**What each anomaly is:**
- **Dirty read:** reading another transaction's uncommitted data.
- **Non-repeatable read:** reading the same row twice and getting different values.
- **Phantom read:** re-running the same range query and getting new rows.

**MVCC:** each transaction reads a consistent snapshot through row versions, so readers don't block writers.

**Locks:**
- Shared vs exclusive.
- Row vs table.
- **Optimistic locking** uses a version column: `UPDATE ... SET v = v + 1 WHERE id = ? AND v = ?`. If 0 rows are updated, someone else changed the row; retry. Good when contention is low.
- **Pessimistic locking** uses `SELECT ... FOR UPDATE`. Good when contention is high (seat booking).

**Deadlock:** two transactions each hold a lock the other needs. InnoDB detects the cycle and rolls one back. Prevention: acquire locks in a consistent order and keep transactions short.

### A5. Normalisation

| Form | Rule | Example of a violation |
|---|---|---|
| 1NF | Atomic values, no repeating groups | `phones = "98..,99.."` |
| 2NF | 1NF + no partial dependency on part of a composite key | In `(order_id, item_id) → item_name`, the name depends only on item_id |
| 3NF | 2NF + no transitive dependency | `restaurant_id → city_id → city_name` stored in the restaurants table |
| BCNF | Every determinant is a candidate key | |

**Denormalise** for read-heavy paths (e.g., store `restaurant_name` on the order for history and speed). The trade-off is that updates must touch every copy.

### A6. CAP, replication, sharding, connection pooling

- **CAP:** during a network **P**artition, a distributed system must choose **C**onsistency (refuse requests or return errors) or **A**vailability (serve possibly stale data).
  - Single-node SQL isn't partitioned, so CAP doesn't really apply. Replicated SQL with async replicas is AP-ish for reads.
  - Payments: CP. Restaurant ratings / feeds: AP.
- **PACELC:** even without a partition, you trade **L**atency against **C**onsistency.
- **Replication:**
  - Leader–follower (master–slave).
  - Synchronous (safe, slower) vs **async** (fast, with replication lag, so reads can be stale).
  - "Read-your-writes" fix: read from the leader for a short time after a write.
- **Sharding:**
  - Shard key choice: high cardinality, even distribution, matches the main query (e.g., `user_id` for order history).
  - Hot shard problem: a celebrity restaurant or a big city.
  - **Consistent hashing:** keys and nodes sit on a ring, and a key goes to the next node clockwise. Adding or removing a node moves only about 1/N of the keys. Virtual nodes balance the load.
- **Connection pooling** (Myntra tie-in):
  - Opening a DB connection is expensive (TCP + auth + memory per connection). MySQL has a `max_connections` limit.
  - A pool keeps N open connections that requests borrow and return.
  - **Too large:** you exhaust the DB and context-switching thrashes. **Too small:** requests queue.
  - Go's `database/sql` is a pool: `SetMaxOpenConns`, `SetMaxIdleConns`, `SetConnMaxLifetime`.
  - Other options: PgBouncer for Postgres, HikariCP for Java.
- **"How do DBs make apps slow?"**
  - N+1 queries.
  - Missing indexes, causing full scans.
  - `SELECT *`.
  - Long transactions holding locks.
  - Connection exhaustion.
  - Chatty round trips.
  - Lock contention on hot rows.
  - Unbounded result sets (no LIMIT / pagination).

### A7. Pagination (asked at District)
- **Offset:** `LIMIT 20 OFFSET 10000`. Simple, but the DB still scans and discards 10,000 rows, and results shift when new rows are inserted.
- **Cursor / keyset:** `WHERE (created_at, id) < (:last_ts, :last_id) ORDER BY created_at DESC, id DESC LIMIT 20`. It uses the index, so it's O(page size), and stable. You can't jump to page N. Use it for feeds and history.
- **Merging 3 tables into one history feed:** `UNION ALL` of the three queries, each with its own keyset filter and LIMIT, then an outer ORDER BY with LIMIT. A better long-term answer is a single denormalised `ledger` table.

---

## PART B: SQL queries (write these by hand)

**Sample schema (Zomato-like):**

```sql
users(id, name, city, created_at)
restaurants(id, name, city, rating)
cuisines(id, name)
restaurant_cuisines(restaurant_id, cuisine_id)
orders(id, user_id, restaurant_id, rider_id, amount, status, created_at)
riders(id, name, area)
employees(id, name, salary, dept_id, manager_id)
```

### B1. Second-highest / Nth-highest salary (asked 3×)

```sql
-- 2nd highest, handling duplicates and the no-result case
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Nth highest (N = 3), with a window function
SELECT DISTINCT salary
FROM (SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk FROM employees) t
WHERE rnk = 3;

-- Nth highest with LIMIT/OFFSET (MySQL)
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 2;
```
**ROW_NUMBER vs RANK vs DENSE_RANK** for the values 100, 100, 90:
- ROW_NUMBER: 1, 2, 3.
- RANK: 1, 1, 3.
- DENSE_RANK: 1, 1, 2.

### B2. Highest salary per department (top-1 per group)

```sql
SELECT dept_id, name, salary
FROM (SELECT e.*, DENSE_RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) r FROM employees e) t
WHERE r = 1;
```

### B3. Restaurants with no cuisine, using JOINs only (anti-join; asked at District)

```sql
SELECT r.id, r.name
FROM restaurants r
LEFT JOIN restaurant_cuisines rc ON rc.restaurant_id = r.id
WHERE rc.restaurant_id IS NULL;
-- Alternatives: NOT EXISTS (preferred; NULL-safe) or NOT IN (dangerous if the subquery returns NULL)
```

### B4. Top 5 riders in an area by count of delivered orders (asked 2026)

```sql
SELECT rd.id, rd.name, COUNT(*) AS delivered
FROM orders o
JOIN riders rd ON rd.id = o.rider_id
WHERE rd.area = 'Koramangala' AND o.status = 'DELIVERED'
GROUP BY rd.id, rd.name
ORDER BY delivered DESC
LIMIT 5;
-- Index: orders(rider_id, status), or orders(status, rider_id), depending on selectivity
```

### B5. Users who ordered from at least 3 different restaurants in the last 30 days

```sql
SELECT user_id
FROM orders
WHERE created_at >= NOW() - INTERVAL 30 DAY
GROUP BY user_id
HAVING COUNT(DISTINCT restaurant_id) >= 3;
```
**WHERE vs HAVING:** WHERE filters rows *before* grouping; HAVING filters groups *after* aggregation.

### B6. Users who never ordered

```sql
SELECT u.* FROM users u
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

### B7. Duplicate emails / delete duplicates, keeping the lowest id

```sql
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;

DELETE u1 FROM users u1
JOIN users u2 ON u1.email = u2.email AND u1.id > u2.id;    -- MySQL syntax
```

### B8. Employee earning more than their manager (self-join)

```sql
SELECT e.name
FROM employees e JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

### B9. Running total and day-over-day change

```sql
SELECT DATE(created_at) d,
       SUM(amount) AS revenue,
       SUM(SUM(amount)) OVER (ORDER BY DATE(created_at)) AS running_total,
       SUM(amount) - LAG(SUM(amount)) OVER (ORDER BY DATE(created_at)) AS change_vs_prev
FROM orders
GROUP BY DATE(created_at);
```

### B10. Each user's first order

```sql
SELECT * FROM (
  SELECT o.*, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at) rn FROM orders o
) t WHERE rn = 1;
```

### B11. Persons with both parents alive (asked; GROUP BY/HAVING)

```sql
-- people(id, name, father_id, mother_id, is_alive)
SELECT c.id, c.name
FROM people c
JOIN people f ON f.id = c.father_id
JOIN people m ON m.id = c.mother_id
WHERE f.is_alive = 1 AND m.is_alive = 1;
-- "At least one parent alive": use LEFT JOINs, then WHERE f.is_alive = 1 OR m.is_alive = 1
```

### B12. Exchange Seats (LC 626, on the Zomato tag)

```sql
SELECT CASE
         WHEN id % 2 = 1 AND id = (SELECT MAX(id) FROM seat) THEN id
         WHEN id % 2 = 1 THEN id + 1
         ELSE id - 1
       END AS id, student
FROM seat ORDER BY id;
```

### B13. Insert a user with a unique phone number in one query (asked)
- Add `UNIQUE(phone)`, then run a plain `INSERT`. A duplicate fails atomically, with no race condition.
- `INSERT IGNORE` silently skips the duplicate.
- `INSERT ... ON DUPLICATE KEY UPDATE` upserts in MySQL. Postgres: `ON CONFLICT (phone) DO NOTHING / DO UPDATE`.
- **Why not "SELECT, then INSERT"?** Two concurrent requests can both see no row and both insert: a **check-then-act race**. The unique constraint is what guarantees correctness.

### B14. Join types (quick recap)
- **INNER:** only matching rows.
- **LEFT:** every left row, with NULLs where the right has no match.
- **RIGHT:** the mirror of LEFT.
- **FULL OUTER:** every row from both sides. MySQL lacks it; emulate with LEFT UNION RIGHT.
- **CROSS:** the Cartesian product.
- **SELF:** a table joined to itself.

**Joins vs subqueries, which is faster?** Modern optimisers often rewrite them into the same plan. Correlated subqueries can run once per row and be slow. Prefer JOIN or EXISTS, and check with EXPLAIN.

---

## PART C: Operating systems

### C1. Process vs thread (asked 6+×)

| | Process | Thread |
|---|---|---|
| Memory | Own address space | Shares the process's heap, code and globals; own stack and registers |
| Creation / context switch | Expensive (page tables, TLB flush) | Cheap |
| Communication | IPC: pipes, sockets, shared memory, message queues | Shared memory directly (needs synchronisation) |
| Fault isolation | A crash doesn't affect other processes | A crash takes down the whole process |

- **Goroutines** are user-space threads managed by the Go runtime, using **M:N scheduling** (M goroutines over N OS threads).
  - They start with about a 2 KB growable stack, versus about 1 MB for OS threads.
  - That's why Go handles 100k concurrent goroutines easily.
- **Kotlin coroutines** are similar: suspendable computations on a thread pool.

### C2. Concurrency vs parallelism
- **Concurrency:** dealing with many tasks at once by interleaving them (possible on one core).
- **Parallelism:** literally executing them at the same time on multiple cores.
- Go's motto: "Concurrency is about structure, parallelism is about execution."

### C3. Synchronisation
- **Mutex:** a lock with ownership. Only the thread that locks it unlocks it. Gives mutual exclusion for a critical section.
- **Semaphore:** a counter allowing N concurrent holders, with no ownership. A binary semaphore is similar to a mutex. Use it for a connection pool of N, or to limit concurrency.
- **RWMutex:** many concurrent readers or one writer. Good for read-heavy data such as a config cache.
- **Condition variable:** wait until a condition is true (producer–consumer).
- **Race condition:** the outcome depends on timing, for example two threads doing `count++` (read-modify-write). Fix it with a mutex or atomics.
- **Go channels:** "Don't communicate by sharing memory; share memory by communicating."

**Producer–consumer (bounded buffer) in Go, a natural follow-up from your worker-pool work:**

```go
jobs := make(chan Job, 100) // buffered channel = bounded queue; producers block when it is full
```

### C4. Deadlock
**The four Coffman conditions** (all must hold):
1. Mutual exclusion.
2. Hold and wait.
3. No preemption.
4. Circular wait.

**Ways to handle it:**
- **Prevention:** break one condition. The common choice is a **global lock ordering**, which breaks circular wait. Or acquire all locks at once.
- **Avoidance:** the Banker's algorithm (a safe-state check).
- **Detection and recovery:** a wait-for graph with cycle detection, then kill or roll back a participant (this is what DBs do).
- **Livelock:** threads keep reacting to each other without making progress. **Starvation:** a thread never gets the resource.

### C5. Scheduling algorithms

| Algorithm | Behaviour |
|---|---|
| FCFS | Simple; suffers the convoy effect |
| SJF / SRTF | Optimal average wait time; needs burst prediction; starves long jobs |
| Priority | Can starve low priority; fixed by aging |
| Round Robin | Time quantum; fair; good response time |
| Multilevel feedback queue | Used by real OSes |

### C6. Memory
- **Paging:** fixed-size pages map to frames through page tables. The **TLB** caches those translations.
- **Virtual memory:** each process sees its own address space. **Page fault:** the page isn't in RAM, so it's loaded from disk.
- **Page replacement:**
  - FIFO (suffers Belady's anomaly).
  - **LRU**, the classic MCQ.
  - Optimal (theoretical).
  - Clock (an approximation of LRU).
- **Thrashing:** too many page faults, so the system spends its time swapping instead of working.
- **Stack vs heap:** the stack holds function frames and locals (automatic, fast, limited). The heap holds dynamic allocations (manual or GC-managed).

### C7. `fork()` counting (OA/MCQ favourite)
```c
fork(); fork(); fork(); printf("hi\n");   // prints 2^3 = 8 times
```
- n sequential forks produce 2ⁿ processes.
- For `if (fork() && fork())`-style expressions, trace the tree. `fork()` returns 0 in the child and the child's PID (> 0) in the parent.

### C8. Merge sort on 8 cores vs 1 (asked 2×)
- **Split the work:** divide the array into 8 chunks and sort each on its own core (O((n/8) log(n/8)) each, in parallel). Then merge.
- **The merge is the bottleneck:**
  - A tree of merges takes log₂8 = 3 levels, and the final merge is O(n) on one core.
  - Or use a **k-way merge with a min-heap**: O(n log 8).
  - Or use parallel merge (split by the median using binary search).
- **Speedup is below 8×,** for three reasons:
  - Amdahl's law: the serial merge.
  - Memory bandwidth.
  - Thread overhead.
- **Don't spawn a thread for tiny subarrays.** Below a threshold, sort sequentially. Go's version is goroutines plus a `WaitGroup`.

### C9. Why is Redis single-threaded (for command execution)?
- Operations are in-memory and very fast, so the CPU isn't the bottleneck; **the network and memory are**.
- A single thread gives **no locks**, **atomic commands for free**, and no context switching.
- It uses **I/O multiplexing** (epoll) to serve thousands of connections on one thread.
- Since Redis 6 there is optional multithreaded network I/O, but commands still execute on one thread.
- To scale: run several instances, Redis Cluster sharding, or read replicas.

---

## PART D: Computer networks

### D1. "What happens when you type zomato.com and press Enter?" (asked 7+×)

1. **URL parsing:** the scheme (https), host and path.
2. **DNS resolution:**
   - Lookup order: the browser cache, the OS cache (`/etc/hosts`), then the router, then the **recursive resolver** (the ISP's or 8.8.8.8).
   - The resolver queries the **root** server, then the **.com TLD** server, then the **authoritative** server for zomato.com, which returns the IP (often a CDN or load-balancer IP).
   - Results are cached according to their TTL.
3. **TCP 3-way handshake** with the IP on port 443: SYN → SYN-ACK → ACK.
4. **TLS handshake:**
   - Exchange ClientHello and ServerHello and agree a cipher.
   - The server sends its certificate; the browser verifies the chain of trust.
   - A key exchange (ECDHE) produces a symmetric session key.
   - TLS 1.3 needs 1 round trip.
5. **HTTP request:** `GET / HTTP/2` with headers (Host, cookies, User-Agent).
6. **Server side:**
   - The request passes through the **CDN** (static assets served from the edge).
   - Then a **load balancer**, then the API gateway or web server, then the app services. These read from a cache (Redis) or the DB, and each service may call others.
7. **Response:** status line, headers (Cache-Control, Set-Cookie), and the HTML body.
8. **Browser rendering:**
   - Parse the HTML into the DOM and the CSS into the CSSOM, then build the render tree.
   - Run layout, then paint.
   - JS executes, and more requests fetch the JS/CSS/images and the API calls.
9. **Connection reuse:** keep-alive or HTTP/2 multiplexing reuses the connection.

### D2. TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Connection | Connection-oriented (handshake) | Connectionless |
| Reliability | Ordered, reliable, retransmission | Best effort, no ordering |
| Control | Flow control (window) + congestion control | None |
| Overhead | Higher (20-byte+ header) | Low (8-byte header) |
| Use cases | HTTP/1.1, HTTP/2, DB connections, email | DNS, video calls, gaming, live streaming, **HTTP/3 (QUIC runs over UDP)** |

**Connection setup and teardown:**
- **3-way handshake:** SYN(seq = x) → SYN-ACK(seq = y, ack = x+1) → ACK(ack = y+1).
- **4-way teardown:** FIN → ACK → FIN → ACK. Then TIME_WAIT.

### D3. OSI vs TCP/IP

| OSI layer | Examples |
|---|---|
| 7 Application | HTTP, DNS, SMTP |
| 6 Presentation | TLS / encoding |
| 5 Session | Session management |
| 4 Transport | TCP, UDP (ports) |
| 3 Network | IP, routing |
| 2 Data link | Ethernet, MAC addresses, switches |
| 1 Physical | Cables, signals |

TCP/IP has 4 layers: Application, Transport, Internet, Link.

### D4. HTTP essentials
- **Methods:**
  - GET (safe, idempotent).
  - POST (not idempotent).
  - PUT (idempotent replace).
  - PATCH (partial update).
  - DELETE (idempotent).
- **Status codes:**
  - 200 OK, 201 Created, 202 Accepted (async, as at Atlassian), 204 No Content.
  - 301/302 redirects, 304 Not Modified.
  - 400, 401 Unauthenticated, 403 Forbidden, 404, 409 Conflict, 429 Too Many Requests (rate limit).
  - 500, 502 Bad Gateway, 503 Unavailable, 504 Gateway Timeout.
- **HTTP versions:**
  - HTTP/1.1: keep-alive, but head-of-line blocking.
  - HTTP/2: multiplexed streams over one TCP connection, header compression.
  - HTTP/3: QUIC over UDP, no TCP head-of-line blocking.
- **REST:** resources identified by URLs, stateless requests, standard methods and status codes.
  - Good: `GET /restaurants/{id}/menu`, `POST /orders`.
  - Bad: `POST /getMenu`.
- **Idempotency key for POST** (e.g., placing an order): the client sends `Idempotency-Key: uuid`. The server stores the key with its result, and retries return the same result. This avoids double orders and double charges.

### D5. WebSockets / long polling / SSE (real-time order tracking)

| Technique | How it works | Use |
|---|---|---|
| Short polling | The client asks every N seconds | Simple, wasteful |
| Long polling | The server holds the request until there's data | Older chat systems |
| **SSE** | A one-way server→client stream over HTTP | Live order status, notifications |
| **WebSocket** | Full-duplex and persistent, after an HTTP Upgrade | Chat, live rider location |
| WebRTC | Peer-to-peer media | Video calls |

### D6. Security basics
- **Authentication vs authorisation:** *who you are* (login, JWT) vs *what you may do* (RBAC, ACLs).
- **JWT:** `header.payload.signature`, stateless.
  - Cons: hard to revoke, and the payload is visible (base64, not encrypted).
  - Mitigations: short expiry plus refresh tokens; store it in an httpOnly cookie.
- **Session vs JWT:**
  - Sessions are server-side, keyed by a session ID in a cookie: easy to revoke, but need shared session storage (Redis).
  - JWTs need no server-side lookup.
- **Hashing vs encryption:**
  - Hashing is one-way: passwords with **bcrypt or argon2, plus a salt**.
  - Encryption is reversible with a key: AES (symmetric) or RSA (asymmetric).
  - Why bcrypt and not SHA-256 for passwords? bcrypt is slow and has a tunable cost, which resists brute force.
- **Preventing session-ID theft when the attacker shares your IP:**
  - HTTPS only, with `Secure`, `HttpOnly` and `SameSite` cookie flags.
  - Short TTL, and rotate the ID on login.
  - Bind the session to a device fingerprint or user-agent.
  - Re-authenticate for sensitive actions.
  - An IP check alone isn't enough, because the attacker is behind the same NAT.
- **CORS:** the browser blocks cross-origin XHR unless the server sends `Access-Control-Allow-Origin`. A preflight `OPTIONS` request is sent for non-simple requests. *(TraceX avoids CORS entirely with its BFF.)*
- **Common attacks:**
  - **SQL injection:** fix with parameterised queries.
  - **XSS:** fix with escaping and a CSP.
  - **CSRF:** fix with SameSite cookies and CSRF tokens.
  - **DDoS:** fix with rate limiting, a CDN/WAF, and autoscaling.
- **Image upload via S3 pre-signed URLs:**
  1. The client asks the backend for an upload URL.
  2. The backend signs a time-limited PUT URL.
  3. The client uploads directly to S3, which offloads bandwidth from the servers.
  4. The client sends the object key to the backend.

### D7. CDN, load balancer, API gateway
- **CDN:** caches static content at edge locations near users, lowering latency and origin load.
- **Load balancer:**
  - **L4** (TCP-level) vs **L7** (HTTP-aware: routes by path, terminates TLS).
  - Algorithms: round robin, least connections, IP hash, consistent hashing.
  - Health checks remove dead instances.
- **API gateway:** auth, rate limiting, routing, request aggregation.

---

## PART E: OOP and language internals

### E1. The 4 pillars (with a Zomato example)
- **Encapsulation:** `Order` hides its state; status changes only through `order.markDelivered()`, which validates the transition.
- **Abstraction:** a `PaymentGateway` interface with `pay(amount)`; callers don't know whether it's Razorpay or Paytm.
- **Inheritance:** `DeliveryPartner` and `Customer` extend `User`.
- **Polymorphism:**
  - **Compile-time:** overloading and templates.
  - **Run-time:** virtual functions and overriding. `notifier->send()` calls `SMSNotifier` or `PushNotifier`.

### E2. Virtual functions and dynamic dispatch (C++)
- Each class with virtual functions has a **vtable** (an array of function pointers). Each object holds a **vptr** pointing to its class's vtable. A call through a base pointer looks up the vtable at run time.
- **Virtual destructors:** a base class with virtual functions needs a `virtual ~Base()`. Otherwise `delete basePtr` leaks the derived part (undefined behaviour).
- **Pure virtual:** `virtual void f() = 0;` makes the class abstract.

**Constructor/destructor order:**
- Construction: base, then members, then derived.
- Destruction: the exact reverse (derived, then members, then base).

```cpp
struct A { A(){cout<<"A ";} virtual ~A(){cout<<"~A ";} };
struct B : A { B(){cout<<"B ";} ~B(){cout<<"~B ";} };
A* p = new B(); delete p;     // prints: A B ~B ~A   (without virtual ~A: A B ~A)
```

### E3. HashMap internals (asked several times; a candidate was rejected over it)
- **The structure:** an array of buckets. `index = hash(key) & (capacity - 1)`.
- **Java HashMap:**
  - Collisions are handled by **chaining** with a linked list. Since Java 8, a bucket with more than 8 entries becomes a **red-black tree** (O(log n) worst case).
  - The **load factor is 0.75**. Above it, capacity doubles and every entry is rehashed: O(n), but amortised O(1).
  - The `equals` and `hashCode` contract: equal objects must have equal hashes. If you override equals without overriding hashCode, lookups break.
  - Not thread-safe. **ConcurrentHashMap** uses fine-grained locking or CAS per bucket.
- **C++:**

  | | `std::unordered_map` | `std::map` |
  |---|---|---|
  | Implementation | Hash table with chaining | **Red-black tree** |
  | Operations | O(1) average, O(n) worst | O(log n) |
  | Order | Unordered | **Keys sorted**; supports `lower_bound` and ordered iteration |
  | Use when | Speed matters and order doesn't | You need order or range queries |

- **Open addressing vs chaining:**
  - **Open addressing:** probe within the array itself (linear, quadratic, double hashing). Cache-friendly, but clusters, and deleting entries is tricky (needs tombstones).
  - **Chaining:** a list per bucket.

**"Implement a HashMap from scratch with thread safety"** (asked at Zomato):

```cpp
class MyHashMap {
    vector<list<pair<string,int>>> buckets; size_t sz = 0; mutable shared_mutex mtx;
    size_t idx(const string& k) const { size_t h = 0; for (char c : k) h = h * 31 + c; return h % buckets.size(); }
    void rehash() {
        vector<list<pair<string,int>>> old(buckets.size() * 2); swap(old, buckets);
        for (auto& b : old) for (auto& kv : b) buckets[idx(kv.first)].push_back(kv);
    }
public:
    MyHashMap() : buckets(16) {}
    void put(const string& k, int v) {
        unique_lock lk(mtx);                                  // writer lock
        for (auto& kv : buckets[idx(k)]) if (kv.first == k) { kv.second = v; return; }
        buckets[idx(k)].push_back({k, v});
        if (++sz > buckets.size() * 3 / 4) rehash();          // load factor 0.75
    }
    optional<int> get(const string& k) const {
        shared_lock lk(mtx);                                  // many readers at once
        for (auto& kv : buckets[idx(k)]) if (kv.first == k) return kv.second;
        return nullopt;
    }
};
```
**Next step, lock striping:** one lock per segment of buckets, so writers to different segments don't block each other. This is how the old ConcurrentHashMap worked.

### E4. Quick language facts
- **`push_back` vs `emplace_back`:** `emplace_back` constructs the object in place from constructor arguments, avoiding a temporary plus a move or copy.
- **`vector` growth:** capacity doubles, so `push_back` is amortised O(1). Reallocation invalidates iterators, pointers and references.
- **Java vs C++:** Java runs on the JVM with GC, has no pointers, and only single class inheritance. C++ is compiled to native code with manual or RAII memory management, multiple inheritance and templates.
- **ArrayList vs LinkedList:** ArrayList gives O(1) random access and cache-friendly iteration. LinkedList gives O(1) insert and delete given the node, but O(n) access. In practice ArrayList usually wins.
- **SOLID** (a common follow-up after LLD):
  - **S**ingle responsibility.
  - **O**pen/closed: extend without modifying.
  - **L**iskov substitution.
  - **I**nterface segregation.
  - **D**ependency inversion: depend on abstractions.

---

## PART F: High-level design (yes, asked of freshers too)

**A framework for any design question** (aim for 15–20 minutes):
1. **Requirements** (2 min): functional requirements, plus non-functional ones (scale, latency, availability, consistency).
2. **Estimates** (1 min): QPS, storage. Example: 2M orders/day ≈ 25 orders/s on average, with peaks about 10× at dinner.
3. **APIs** (2 min).
4. **Data model** (3 min).
5. **High-level diagram** (5 min): client, then LB/gateway, then services, then DB, cache and queue.
6. **Deep dive on 1–2 parts, and the failure cases** (5 min). *Interviewers specifically mark down happy-path-only answers.*
7. **Trade-offs and bottlenecks.**

### F1. Rate limiter (asked 6×; once you had to implement token bucket and fixed window)

**Algorithms:**

| Algorithm | How | Pros | Cons |
|---|---|---|---|
| **Token bucket** | A bucket with capacity B refills at r tokens/s; each request takes 1 token | Allows bursts up to B; smooth average; cheap (2 numbers per key) | Two parameters to tune |
| Leaky bucket | Requests enter a queue that drains at a constant rate | Perfectly smooth output | Bursts are queued or dropped; adds latency |
| **Fixed window** | A counter per (key, minute); reject above the limit | Very simple (`INCR` + `EXPIRE`) | **Boundary burst**: 2× the limit across a window edge |
| Sliding window log | Store the timestamps; count those in the last 60 s | Exact | Memory O(requests) |
| **Sliding window counter** | Weighted: `curr + prev × (1 − elapsed/window)` | Accurate and cheap | An approximation |

**Token bucket code:**

```cpp
class TokenBucket {
    double capacity, refillPerSec, tokens; chrono::steady_clock::time_point last; mutex m;
public:
    TokenBucket(double cap, double rate) : capacity(cap), refillPerSec(rate), tokens(cap), last(chrono::steady_clock::now()) {}
    bool allow() {
        lock_guard<mutex> g(m);
        auto now = chrono::steady_clock::now();
        double elapsed = chrono::duration<double>(now - last).count();
        tokens = min(capacity, tokens + elapsed * refillPerSec);   // lazy refill
        last = now;
        if (tokens >= 1) { tokens -= 1; return true; }
        return false;
    }
};
```

**Fixed window code** (key → {windowStart, count}):

```cpp
class FixedWindow {
    int limit, windowSec; unordered_map<string, pair<long long,int>> m; mutex mu;
public:
    FixedWindow(int l, int w) : limit(l), windowSec(w) {}
    bool allow(const string& key, long long nowSec) {
        lock_guard<mutex> g(mu);
        long long win = nowSec / windowSec;
        auto& [w, c] = m[key];
        if (w != win) { w = win; c = 0; }
        if (c >= limit) return false;
        c++; return true;
    }
};
```

**Distributed rate limiter:**
- **Where it lives:** in the API gateway or as middleware.
- **State:** in **Redis**, keyed per user, IP or API key.
  - Fixed window: `INCR key` then `EXPIRE key 60`.
  - Token bucket: a **Lua script**, so the read-and-update is atomic.
- **Response:** **429** with `Retry-After` and `X-RateLimit-Remaining` headers.
- **Failure mode:** if Redis is down, choose between *fail-open* (allow traffic, keeping availability) and *fail-closed* (protect the backend). Add a local in-memory fallback limiter.
- **Hot keys:** shard by key. For multi-region setups, use approximate local limits.
- **Your tie-in:** "At Myntra I used rate limiting inside the ratings batch pipeline to protect MySQL and downstream services."

### F2. Notification system (asked 5×)

**Requirements:**
- Send order updates through push, SMS, email and in-app channels.
- Respect user preferences.
- Deduplicate.
- Retry failures.
- Support priorities: an OTP or order update beats marketing.

**Flow:**

```
Order Service ──event──► Kafka topic "order-events"
                              │
                     Notification Service (consumers)
                     1. look up user preferences + device tokens (cache → DB)
                     2. render template
                     3. dedup check: Redis SETNX notif:{userId}:{eventId}, TTL 24h
                     4. enqueue per-channel: push-queue / sms-queue / email-queue
                              │
                Channel workers ──► FCM/APNs, SMS provider, email provider
                     retries with exponential backoff → DLQ after N attempts
                     write status to notification_log (sent / delivered / failed)
```

**Key points:**
- **Idempotency:** dedup on `(userId, eventId)`, because Kafka is at-least-once.
- **Priority:** separate topics or queues per priority, so marketing can never delay OTPs.
- **Rate limiting** per user, to avoid spamming.
- **Provider failover:** a second SMS vendor.
- **Crash resilience:**
  - Commit the Kafka offset **after** processing.
  - Make sends idempotent.
  - Use the outbox pattern in the order service, so an event is never lost between the DB commit and the publish.
- **Restaurant-reopen variant** ("notify users when a restaurant reopens, only if riders are available"):
  1. A subscribers table keyed by restaurant.
  2. A trigger event when the restaurant goes online.
  3. A rider-availability check per zone (from the dispatch service's cache) before fan-out.
  4. Fan out in batches; don't notify everyone at once (thundering herd).

### F3. Food ordering and order tracking (the core Zomato design)

**Services:**
- User.
- Restaurant/Menu.
- **Search** (Elasticsearch plus geo).
- Cart.
- **Order** (state machine).
- Payment.
- **Dispatch/Delivery** (rider assignment).
- **Location** (rider pings).
- Notification.
- ETA (ML).

**APIs:**

```
GET  /restaurants?lat=&lng=&cuisine=&cursor=      → nearby, ranked
GET  /restaurants/{id}/menu
POST /orders            {items, address_id, payment_method}  + Idempotency-Key header
GET  /orders/{id}       → status, ETA, rider
POST /orders/{id}/cancel
WS   /orders/{id}/track → live rider location
```

**Schema (core):**

```sql
orders(id PK, user_id, restaurant_id, rider_id NULL, status, amount, address_id,
       idempotency_key UNIQUE, created_at, updated_at, version)
order_items(order_id, item_id, qty, price_at_order)       -- snapshot the price!
order_status_history(order_id, from_status, to_status, at)
riders(id, name, zone_id, status)                          -- AVAILABLE / BUSY / OFFLINE
restaurant_managers(restaurant_id, user_id, role)          -- many-to-many
```

**Order state machine:**

```
PLACED → PAYMENT_PENDING → CONFIRMED → ACCEPTED_BY_RESTAURANT → PREPARING
       → RIDER_ASSIGNED → PICKED_UP → DELIVERED
(any pre-pickup state) → CANCELLED ; payment failure → FAILED
```

Enforce the transitions in code, and store every transition in a history table (exactly like TraceX's case state machine).

**Placing an order:**
1. The API gateway runs auth and rate limiting.
2. The order service checks the idempotency key, then validates the cart and prices (prices are re-read from the menu service, never trusted from the client).
3. It creates the order as PAYMENT_PENDING and calls payment.
4. The payment callback or webhook moves the order to CONFIRMED.
5. The service publishes `OrderConfirmed` to **Kafka**.
6. Consumers react: the restaurant app (via WebSocket or push), **Dispatch** (rider assignment), Notification, and Analytics.

**Sync vs Kafka?**
- Use a **synchronous** call where the user needs the answer immediately: payment initiation and validation.
- Use **async Kafka** for fan-out side effects (notify, dispatch, analytics). It decouples services and keeps them resilient when a consumer is down.

**Rider assignment:**
- Riders ping their location every few seconds. Pings go to the Location service, which keeps the latest position in **Redis GEO** (`GEOADD`, `GEOSEARCH` by radius) or a geohash or H3 cell index.
- On an order: find the AVAILABLE riders within radius R of the restaurant. Score each by distance/ETA to the restaurant, current load and rating.
- Offer the order to the best rider with a timeout. On reject or timeout, offer it to the next one.
- To stop two orders grabbing the same rider, assign **atomically**: `UPDATE riders SET status='BUSY' WHERE id=? AND status='AVAILABLE'` and check the affected row count; or take a Redis lock.
- Batching: combine nearby orders for one rider.

**Nearby restaurants:**
- Use a **geohash** prefix match (with neighbouring cells), **PostGIS** `ST_DWithin` with a GiST index, or an Elasticsearch `geo_distance` query.
- Cache popular cells. Filter by open hours and serviceability.

**Live tracking:**
- The rider app sends a location every 3–5 s to the Location service, which writes to Redis.
- Clients subscribe over **WebSocket/SSE**, with pub/sub per order ID.
- Never write every ping to the primary DB. Use a time-series store or sampled persistence.

**Failure cases** (say them unprompted):
- Payment succeeded but the order service crashed → reconcile via the payment webhook and a periodic reconciliation job.
- Duplicate submit → the idempotency key.
- Kafka consumer down → events wait in the topic and are processed on recovery (consumers must be idempotent).
- No rider found → widen the radius, alert ops, and notify the user with an updated ETA.
- Peak traffic (NYE/IPL) → autoscaling, queue buffering, rate limits, graceful degradation (turn off non-critical features like recommendations).

### F4. BookMyShow / seat locking (asked 3×; District does events and movies)

**The problem:** two users must never book the same seat.

**Approach:**
1. **Temporary hold.** When a user selects seats, take a lock with a TTL (e.g., 10 min):
   - **Pessimistic:** `SELECT ... FOR UPDATE` on the seat rows inside a transaction.
   - **Optimistic:** `UPDATE show_seats SET status='HELD', held_by=?, hold_expiry=NOW()+10min WHERE show_id=? AND seat_id IN (...) AND status='AVAILABLE'`. If the affected row count is lower than the number requested, roll back. Atomic.
   - Or **Redis** `SET seat:{show}:{seat} user NX EX 600`.
2. **Payment:** on success, move HELD → BOOKED. On failure or expiry, release back to AVAILABLE (a scheduled sweeper, or the Redis TTL).
3. **Schema:** `show_seats(show_id, seat_id, status, held_by, hold_expiry, version)`, with primary key `(show_id, seat_id)`.

**Optimistic vs pessimistic:**
- Optimistic scales better when contention is low, and fails fast.
- Pessimistic is simpler for hot shows but holds DB locks.

### F5. Others that appeared (one-line approach each)
- **Spotify / music player (3×):** song metadata in SQL; audio files in S3 + CDN; streaming in chunks; playlist CRUD; shuffle with Fisher-Yates; play counts via Kafka, aggregated.
- **WhatsApp / chat:**
  - Transport: WebSocket gateway, with a user → connection map in Redis.
  - Storage: messages in Cassandra (partitioned by conversation).
  - Offline delivery: a queue plus push notification.
  - Receipts: delivered and read.
- **Image upload with CRUD:** S3 pre-signed URLs; metadata in the DB; async thumbnailing via a queue; CDN delivery.
- **Load balancer + consistent hashing:** described in A6 and D7.
- **Feature-flag service:** flags stored in the DB, cached in each service with a TTL or pushed via pub/sub; percentage rollout by `hash(userId) % 100`; a kill switch.
- **Payments, refunds, chargebacks:**
  - Double-entry ledger.
  - Idempotency keys.
  - State machine.
  - Webhook reconciliation.
  - Never update a balance without a ledger entry.
- **Inventory (Blinkit):**
  - Stock per dark store.
  - Reserve on checkout with atomic decrement: `UPDATE ... SET qty = qty - 1 WHERE qty > 0`.
  - Release on cancel or timeout.
  - Sync with the warehouse via events.

---

## PART G: LLD and schema questions

### G1. Restaurant menu with customisations (Domino's: size, crust, toppings)

```sql
items(id, restaurant_id, name, base_price, is_veg, available)
option_groups(id, item_id, name, min_select, max_select)       -- "Size" (1,1), "Toppings" (0,5)
options(id, group_id, name, price_delta, available)            -- "Large" +150, "Cheese Burst" +99
order_item_options(order_item_id, option_id, price_at_order)
```

In code: `Item` has `List<OptionGroup>`, and `OptionGroup` validates the selection count. Price = base + Σ option deltas. This is the composite or decorator pattern.

### G2. Banner shown X times in Y hours for Z duration (asked 2025)

```sql
banners(id, content, start_at, end_at, max_impressions_per_user, window_hours, priority)
```

- **Impression tracking:** Redis `INCR imp:{banner}:{user}` with `EXPIRE window_hours`, the same fixed-window idea as rate limiting.
- **Serving rule:** show the banner if `now ∈ [start, end]` and the count is below the max. Pick by priority.

### G3. Delivery partner + order schema
See F3. Add `rider_shifts(rider_id, start, end)`, `rider_locations` (latest in Redis, history in a time-series store) and `payouts(rider_id, order_id, amount)`.

### G4. Parking lot / elevator / deck of cards / attendance system (classic LLD)
Approach:
1. Identify the entities (nouns).
2. Identify the behaviours (verbs).
3. Choose the relationships (has-a / is-a).
4. Apply patterns:
   - **Strategy:** pricing, assignment.
   - **Factory:** creating vehicles or notifications.
   - **Observer:** notifications.
   - **State:** order or elevator state.
   - **Singleton:** config.
5. Write the class skeletons with their key methods.

### G5. Your own projects' schemas (asked: "the schema of your project")
- **TraceX:** `customers → accounts → transactions`; `alerts`; `cases` (FSM) with `case_accounts`; `audit_log` hash chain (file 03, section 1.8).
- **Myntra ratings:** `style_rating(style_id PK, count_1..count_5, total, avg, updated_at)` and `style_group(style_id, group_id)` with an index on group_id. Aggregate = Σ histograms over the group. This is a good answer to "why store a histogram?": the aggregate is exact and incremental updates are easy.

---

## PART H: Kafka and Redis (recurring infrastructure topics, directly on your resume)

**Kafka**
- **Topic → partitions:** each partition is an ordered, append-only log. Offsets identify positions.
- **Producers:** choose the partition by **key hash**, so the same key always lands in the same partition and **ordering holds per key**.
  - Use order_id as the key and every event of one order stays in order.
- **Consumer groups:**
  - Each partition goes to **one** consumer in the group, so parallelism is capped at the number of partitions.
  - Different groups each get every message (pub/sub).
- **Offsets and delivery semantics:**
  - **At-least-once:** commit the offset after processing. Duplicates are possible, so consumers must be **idempotent**.
  - **At-most-once:** commit before processing; messages can be lost.
  - **Exactly-once:** idempotent producer plus transactions (Kafka Streams).
- **Durability:** a replication factor (e.g., 3), a leader per partition, followers in the ISR, and `acks=all` for durability.
- **Retention:** time- or size-based. Consumers can **replay** by resetting their offsets, unlike SQS.
- **Rebalancing:** when a consumer joins or leaves, partitions are reassigned and processing pauses briefly.
- **Kafka vs SQS:** covered in file 02, section 3.4, and file 04.

**Redis**
- **Data structures:** strings, hashes, lists, sets, sorted sets (leaderboards, top-K), GEO, streams, HyperLogLog, bitmaps.
- **Persistence:** RDB snapshots and/or an AOF log.
- **Eviction policies:** `allkeys-lru`, `volatile-lru`, `allkeys-lfu`, `volatile-ttl`, `noeviction`.
- **Caching patterns:**
  - **Cache-aside:** read the cache; on a miss, read the DB and set the cache. On a write, update the DB, then **delete** the cache key.
  - **Write-through:** write to the cache and the DB synchronously.
  - **Write-back:** write to the cache and flush to the DB asynchronously. Fast, but data can be lost.
- **Cache problems and fixes:**
  - **Stampede / thundering herd** (a hot key expires): a mutex or single-flight so only one request rebuilds it; jittered TTLs; **cache warm-up** (Myntra!); stale-while-revalidate.
  - **Penetration** (repeated queries for missing keys): cache null results; a Bloom filter.
  - **Hot key:** replicate it across nodes; a local in-process cache; key splitting.
- **Pub/Sub:** fire-and-forget, so offline subscribers miss messages. Use Streams or Kafka when you need durability.
- **Distributed lock:**
  - Take it with `SET lock val NX PX 30000`.
  - Release it with a Lua script that checks the value before deleting.
  - Caveats: clock and GC pauses can outlast the lock. Redlock exists but is debated; use a fencing token for safety.
