# System Design Solution Sheet: Zomato

> Every system design question that has been reported in Zomato / Blinkit / District interviews (2024–26), plus the ones most likely to come, each with a complete worked answer. Concepts are explained in `06_System_Design_Master.md`; read that first.
> **How to practise:** read the question, spend 10 minutes sketching your own answer on paper using the 7-step framework, *then* read the solution. Say the solution out loud once. That's all the practice you need.

**Contents**

| # | Question | Seen at Zomato? |
|---|---|---|
| 1 | Food delivery app end to end (Zomato) | Yes — the core question |
| 2 | Live order tracking | Yes |
| 3 | Rider assignment / dispatch | Yes (as part of 1) |
| 4 | Nearby restaurants and search | Yes (as part of 1) |
| 5 | Rate limiter | Yes — 6+ times, sometimes with code |
| 6 | Notification system | Yes — 5+ times |
| 7 | Payment system with refunds | Yes |
| 8 | Seat / ticket booking (BookMyShow, District) | Yes — 3+ times |
| 9 | Quick-commerce inventory (Blinkit) | Likely |
| 10 | Coupons and promo codes | Likely |
| 11 | Ratings and reviews | Likely (and your Myntra work) |
| 12 | Search autocomplete | Likely |
| 13 | Delayed jobs: auto-cancel if restaurant doesn't accept | Likely |
| 14 | Peak traffic: New Year's Eve | Likely as a follow-up |
| 15 | URL shortener | Classic warm-up |
| 16 | Chat (customer support / WhatsApp) | Seen |
| 17 | Music streaming (Spotify) | Seen 3× |
| 18 | Image upload service | Seen |
| 19 | Feature flags | Seen |
| 20 | Leaderboard / trending restaurants | Likely |
| 21 | Distributed cache, load balancer, consistent hashing | Seen |
| 22 | LLD: menu with customisations | Yes |
| 23 | LLD: banner shown X times in Y hours | Yes |
| 24 | LLD: cart with discounts | Likely |
| 25 | LLD: parking lot | Classic |
| 26 | LLD: delivery partner and order schema | Yes |
| 27 | LLD: "which restaurants are open now?" | Seen |
| 28 | Rapid-fire follow-ups (40 short Q&As) | — |

**The 7 steps, every time:** requirements → estimates → APIs → data model → diagram → deep dive + failures → trade-offs. Numbers below are **assumptions for the exercise**, not Zomato's real figures — say "let's assume" in the interview.

---

## 1. Design a food delivery app (Zomato)

### Requirements
**Functional (agree on these):**
1. Customers browse nearby restaurants and menus, and search.
2. Add to cart, place an order, pay.
3. The restaurant accepts or rejects and marks the food ready.
4. A delivery partner (rider) is assigned, picks up and delivers.
5. Customers track the order live and get notifications.
6. Cancel and refund.

**Out of scope (say so):** ratings, recommendations, loyalty, restaurant onboarding, ads.

**Non-functional:**
- **Scale:** assume 20 lakh orders a day, 5 crore browse sessions, sharp lunch/dinner peaks.
- **Latency:** browsing < 200 ms (p99 < 500 ms); placing an order < 1–2 s.
- **Availability:** browsing and ordering must stay up (99.9%+).
- **Consistency:** orders and payments must be correct — never charge twice, never lose an order. Menus and ratings can be slightly stale.

### Estimates
- Orders: 2×10⁶/day ÷ 10⁵ s ≈ **20/s average, ~100/s at peak** (5×). A sharded MySQL handles that easily.
- Reads (browse, menu, status): ~100× orders → **several thousand/s at peak** → cache + read replicas.
- Rider location: assume 3 lakh active riders at peak pinging every 4 s → **~75,000 writes/s** → Redis + Kafka, not MySQL.
- Storage: ~1 KB per order → ~2 GB/day → ~0.7 TB/year. Archive old orders to cheaper storage.

### APIs
```
GET  /v1/restaurants?lat=&lng=&q=&filters=&cursor=   → ranked nearby restaurants
GET  /v1/restaurants/{id}/menu
PUT  /v1/cart                     {restaurant_id, items[{item_id, qty, options}]}
POST /v1/orders                   {cart_id, address_id, payment_method}
     header Idempotency-Key: <uuid>                  → {order_id, payment_url, status}
GET  /v1/orders/{id}              → status, ETA, rider
POST /v1/orders/{id}/cancel
WS   /v1/orders/{id}/track        → live status + rider location

Restaurant app:  POST /v1/orders/{id}/accept | /reject | /ready
Rider app:       POST /v1/riders/me/location {lat, lng}   (every ~4 s)
                 POST /v1/deliveries/{id}/accept | /picked_up | /delivered
Payment gateway: POST /v1/payments/webhook   (provider calls us)
```

### Data model
Orders and payments → **MySQL** (transactions, correctness). Menus → MySQL with a **Redis** cache. Search → **Elasticsearch**. Rider live location → **Redis GEO**. Events → **Kafka**.

```sql
users(id PK, name, phone UNIQUE, created_at)
addresses(id PK, user_id, lat, lng, text)
restaurants(id PK, name, lat, lng, geohash, city_id, is_open, avg_prep_min)
menu_items(id PK, restaurant_id, name, price, is_veg, is_available)

orders(id PK, user_id, restaurant_id, rider_id NULL, address_id,
       status, item_total, delivery_fee, discount, total,
       idempotency_key UNIQUE, created_at, updated_at, version)
order_items(order_id, item_id, qty, unit_price_at_order)   -- snapshot the price
order_status_history(order_id, from_status, to_status, actor, at)
payments(id PK, order_id, provider_ref UNIQUE, amount, status, created_at)
riders(id PK, name, phone, zone_id, status)                -- OFFLINE / AVAILABLE / BUSY
deliveries(id PK, order_id UNIQUE, rider_id, assigned_at, picked_at, delivered_at)
outbox(id PK, aggregate_id, event_type, payload, created_at, sent_at NULL)
```

Why snapshot `unit_price_at_order`? Menu prices change; the order must remember what the customer actually paid.

**Sharding:** orders sharded by `user_id` (a customer's history on one shard). Restaurants need "my orders today" too, so either a secondary table keyed by `restaurant_id`, or the restaurant dashboard reads from a separate store fed by Kafka. Say this trade-off.

### Architecture
```
 Customer / Restaurant / Rider apps
        │ HTTPS + WebSocket
   ┌────▼─────────────────────────────┐       CDN ◄── images, static
   │ API Gateway: auth, rate limit,   │
   │ routing                          │
   └─┬──────┬──────┬──────┬──────┬────┘
     ▼      ▼      ▼      ▼      ▼
  Search  Menu   Cart/  Payment Dispatch ◄── Location svc ◄── rider pings
  (ES)    (+Redis Order  (ledger) (assign)       │
           cache) (MySQL)   │       │        Redis GEO
     ▲      │      │        │       │
     └──────┴──────┴── Kafka: order-events, payment-events, delivery-events
                                 │
               Notification · Tracking gateway (WS) · ETA (ML) · Analytics
```

### The main flow: placing an order
1. **Gateway** authenticates the user (JWT) and applies rate limits.
2. **Order service** checks the **idempotency key**. If it has seen it, return the stored result.
3. It **re-prices** the cart on the server from the Menu service — never trusts client prices — checks the restaurant is open and serviceable for the address, and applies any coupon.
4. It creates the order as `PAYMENT_PENDING` and writes an outbox row, in one transaction.
5. It asks the **Payment service** to create a payment with the provider (Razorpay etc.) and returns a payment URL/intent to the app. *This part is synchronous — the user is waiting.*
6. The user pays. The provider calls our **webhook**; Payment service verifies the signature, marks the payment `SUCCESS` (idempotently, keyed on the provider's reference), and publishes `PaymentSucceeded`.
7. Order service consumes it and moves the order to `CONFIRMED`, publishing `OrderConfirmed` via the outbox.
8. **Async fan-out** from Kafka: the restaurant app gets the order (push + WebSocket); **Dispatch** starts looking for a rider (often timed so the rider arrives when the food is ready); **Notification** tells the customer; **Analytics** records it.
9. Restaurant accepts → `ACCEPTED` → `PREPARING`; rider assigned → `RIDER_ASSIGNED`; picked up; delivered. Every change goes through the order's **state machine** and is recorded in history.

**Order state machine:**
```
PAYMENT_PENDING → CONFIRMED → ACCEPTED → PREPARING → READY → PICKED_UP → DELIVERED
PAYMENT_PENDING → PAYMENT_FAILED
CONFIRMED / ACCEPTED / PREPARING → CANCELLED (→ refund)
CONFIRMED → REJECTED by restaurant (→ refund)
```
One function performs transitions, using a conditional update: `UPDATE orders SET status='ACCEPTED', version=version+1 WHERE id=? AND status='CONFIRMED'`. If 0 rows change, the transition was illegal or someone else already did it.

### Deep dive: what goes wrong (say these unprompted)
- **User taps "Place order" twice / network retry** → the idempotency key returns the same order.
- **Payment succeeded but our server crashed before recording it** → the provider's webhook retries until we acknowledge; plus a **reconciliation job** every few minutes asks the provider about payments stuck in `PENDING`.
- **Payment succeeded but order can't be fulfilled** (restaurant rejects, no rider) → a saga: compensate with an automatic refund and mark the order cancelled.
- **Kafka consumer (e.g. Notification) is down** → events wait in Kafka; it catches up on recovery. Every consumer is idempotent (dedupe on event ID) because delivery is at-least-once.
- **Event lost between DB write and publish** → the outbox pattern prevents it.
- **Restaurant doesn't respond** → auto-cancel or escalate after N minutes (see Q13).
- **No rider available** → widen the search radius in steps, raise the delivery fee incentive, alert ops, update the customer's ETA.
- **Menu service slow** → cached menus in Redis; circuit breaker with fallback to the last cached copy.
- **Dinner peak** → autoscale stateless services, queues absorb bursts, rate limits protect the core, degrade non-critical features (see Q14).
- **Price changed between viewing and ordering** → re-price on the server at checkout and show the new total before payment.

### Trade-offs to mention
- Microservices let search and menu scale separately from orders, at the cost of distributed failure handling (sagas, idempotency).
- Strong consistency only where needed (orders, payments); eventual consistency for menus, search, ratings.
- Sharding orders by user makes customer history fast; restaurant views need a second copy.

### Likely follow-ups
- **"Why Kafka and not direct calls after the order?"** Decoupling, buffering peaks, retries, and adding consumers without touching the order service. Ordering per order via `order_id` key.
- **"How does the restaurant app get new orders instantly?"** A persistent WebSocket from the restaurant tablet to a gateway, plus push notification as backup, plus a sound alarm in the app.
- **"How do you compute ETA?"** Prep time (restaurant history) + rider's travel to the restaurant + travel to the customer (road distance, traffic, weather), predicted by an ML model, updated as the order progresses. Start with a simple formula and say "an ML model trained on past deliveries improves it".
- **"How do you handle COD (cash on delivery)?"** Skip the payment step: order goes straight to `CONFIRMED`; limit COD for risky users; reconcile cash with the rider.

---

## 2. Live order tracking

### Requirements
- Customer sees the rider's position on a map, updating every few seconds, plus status changes.
- Must handle **lakhs of concurrent deliveries** and millions of watching customers.
- Low latency (< 2 s from rider to customer). Losing one location ping is fine.

### Estimates
3 lakh riders × 1 ping / 4 s ≈ **75,000 pings/s** in; each delivery watched by ~1 customer, so similar out. Each ping ~100 bytes → ~7.5 MB/s. Small data, very high rate.

### Design
```
Rider app ──HTTP/WebSocket──► Location service (stateless, many copies)
                                   │ 1. validate (rider is on an active delivery)
                                   │ 2. GEOADD riders <lng> <lat> rider:55   (latest position)
                                   │ 3. PUBLISH track:order:123 {lat,lng,ts}   (Redis pub/sub)
                                   │ 4. produce to Kafka "rider-locations"   (history, ETA, analytics)
                                   ▼
Customer app ◄──WebSocket/SSE── Tracking gateway servers
                                   (each subscribes to the channels of the orders
                                    its connected users are watching)
```

- **Latest position** in **Redis** (in memory; overwriting is cheap). Never one MySQL write per ping.
- **Fan-out to the customer:** the customer opens a WebSocket (or SSE) to any tracking gateway; that gateway subscribes to `track:order:123`. When the location service publishes, Redis delivers it to whichever gateway holds that customer's connection. This solves "which server is the user connected to?".
- **History** (for disputes, ETA training, distance-based payouts) goes through **Kafka** into a time-series or columnar store, possibly downsampled (keep one point every 30 s).
- **Status changes** (picked up, delivered) arrive from the order-events topic and are pushed through the same connection.

### Smart details
- **Client-side smoothing:** the app animates the marker between points so movement looks smooth even with 4-second pings.
- **Adaptive ping rate:** ping more often when moving fast or near the destination, less when waiting at the restaurant — saves battery and load.
- **Reconnect:** on reconnect, the app first fetches current status and last position via a normal GET, then resumes the stream.
- **Fallback:** if WebSockets are blocked or the gateway is down, the app falls back to polling `GET /orders/{id}` every 10 s.

### Failures
- **Rider loses network:** pings stop; show "last updated 1 min ago"; the app buffers pings and sends them when back online.
- **Gateway server dies:** clients reconnect (through the load balancer) to another gateway, which subscribes again.
- **Redis node dies:** Redis replica takes over; a few seconds of missing points is acceptable for this feature.
- **GPS jumps (bad readings):** discard points implying impossible speeds.

### Follow-ups
- **"Why not poll?"** At millions of watchers, polling every 4 s is millions of mostly-useless requests a minute; push sends only real updates.
- **"WebSocket or SSE?"** The customer only *receives*, so SSE is enough and simpler; WebSocket if you also want chat with the rider in the same connection.
- **"How many connections can one gateway hold?"** Tens of thousands to a few lakh per server with an async server; scale horizontally behind a load balancer.

---

## 3. Rider assignment (dispatch)

### Requirements
- When an order is confirmed (or shortly before the food is ready), assign the best available rider.
- Never assign one rider to two orders by mistake (unless deliberately batching).
- Assign within seconds; handle riders rejecting or not responding.

### Data
- Rider status in MySQL (`OFFLINE / AVAILABLE / BUSY`) and mirrored in Redis.
- Live locations in **Redis GEO**, ideally one set per city or zone (`riders:blr`).

### Algorithm
1. **Find candidates:** `GEOSEARCH riders:blr FROMLONLAT <restaurant> BYRADIUS 3 km ASC COUNT 20`. Filter to `AVAILABLE`.
2. **Score each candidate:** estimated time to reach the restaurant (road distance, not straight line), versus when the food will be ready (no point arriving 15 minutes early), rider's current load, acceptance rate, rating, fairness (riders who haven't had an order in a while).
3. **Reserve atomically:** `UPDATE riders SET status='BUSY', current_order=? WHERE id=? AND status='AVAILABLE'`. 1 row → reserved; 0 → someone else got them, try the next. (Or a Redis lock `SET rider:55:lock order123 NX EX 30`.)
4. **Offer:** push the offer to the rider's app with a 30-second timeout.
5. **Accept** → create the delivery, publish `RiderAssigned`. **Reject or timeout** → release the rider (back to `AVAILABLE`), move to the next candidate.
6. **Nobody within radius** → expand to 5 km, then 8 km; add an incentive; alert ops; update the customer's ETA.

### Better at scale
- **Batch matching:** instead of assigning each order the instant it arrives, collect orders and free riders in a zone every ~10–30 seconds and solve the assignment for all of them together (minimise total wait time). Greedy one-by-one is simple but globally worse; batching gives better overall matches. This is a known **bipartite matching / assignment problem** — mention it.
- **Order batching:** give one rider two orders from the same restaurant or nearby restaurants going the same way.
- **Partitioning:** run dispatch per city or zone so each instance handles a manageable area.

### Failures
- **Dispatcher crashes after reserving a rider but before offering** → the reservation has a timeout; a sweeper releases riders stuck `BUSY` with no accepted delivery.
- **Rider app offline after accepting** → if no movement towards the restaurant within N minutes, reassign.
- **Duplicate `OrderConfirmed` events** → dispatch is idempotent: if the order already has an active delivery, ignore.

### Follow-ups
- **"Nearest rider by straight line — good enough?"** No; a river or a one-way road can make a close rider far away. Use road-network travel time (a routing service), at least for the top few candidates.
- **"How is this a DSA problem?"** K-nearest search (geohash/k-d tree/heap), and the batch version is weighted bipartite matching (Hungarian algorithm) — in practice approximated with greedy plus heuristics.

---

## 4. Nearby restaurants and search

### Requirements
- "Show restaurants that deliver to me", ranked, with filters (veg, rating ≥ 4, cuisine, offers, open now), and text search for dishes and restaurant names with typo tolerance.
- Fast: < 200 ms. Read-heavy: thousands of searches per second at peak.

### Design
- **Source of truth:** restaurants and menus in MySQL.
- **Search index:** **Elasticsearch**, one document per restaurant containing name, cuisines, dish names, location (geo point), rating, price for two, is_open, delivery zones. Fed by Kafka events (or CDC) whenever restaurant or menu data changes.
- **Serviceability:** "delivers to me" isn't just distance — each restaurant has a delivery polygon or zone. Precompute zone → restaurant lists (by geohash/H3 cell) and refresh when zones change.
- **Query flow:** user location → find their cell → fetch candidate restaurants serviceable in that cell → apply filters and text query in Elasticsearch → rank → return a page with a cursor.
- **Ranking:** a weighted score of relevance (text match), distance/ETA, rating, popularity, conversion rate, open now, sponsored placement, and personal history. Start with a formula; mention "a learning-to-rank model trained on clicks and orders" as the next step.
- **Caching:** the default list for a cell with no filters (the home page) is the same for everyone in that cell → cache it in Redis for a minute or two, then lightly personalise on top.

### Nearby without Elasticsearch
Store each restaurant's geohash (precision 6, ~1.2 km cells) with an index. For a user, compute their geohash, query their cell plus the 8 neighbours (`WHERE geohash IN (...)`), compute exact distance in code, filter by radius, sort. That's the version to code if asked.

### Failures and freshness
- **Restaurant closes suddenly** → `is_open` changes are pushed immediately via events; the search results check a fast Redis open/closed flag at serve time so closed restaurants drop out instantly even if the index is a few seconds behind.
- **Elasticsearch cluster slow** → serve the cached cell list; circuit breaker.

### Follow-ups
- **"Why not `LIKE` in MySQL?"** No index use for `%x%`, no typos, no relevance ranking.
- **"How do typos work?"** Fuzzy matching using edit distance (Levenshtein) — "biriyani" still matches "biryani".

---

## 5. Rate limiter

*Asked 6+ times; at least once candidates had to code token bucket and fixed window.*

### Requirements
- Limit each client (user ID, API key or IP) to N requests per time window, e.g. 100/min per user; stricter for login/OTP (5/min).
- Works across many API servers. Adds < 1–2 ms. Rejected requests get **HTTP 429** with a `Retry-After` header.

### Where it lives
In the **API gateway** (or as middleware in each service), before any business logic. Different rules per endpoint.

### Algorithms (explain two, code one)
| Algorithm | How it works | Good | Bad |
|---|---|---|---|
| **Token bucket** | Bucket holds B tokens, refills at R/s; request takes 1 | Allows bursts, smooth average, 2 numbers per key | Two parameters to tune |
| Leaky bucket | Requests queue, processed at fixed rate | Perfectly smooth | Adds delay; bursts dropped |
| **Fixed window** | Counter per key per minute | Simplest (`INCR`+`EXPIRE`) | 2× burst across the window edge |
| Sliding window log | Store timestamps, count last 60 s | Exact | Memory per request |
| Sliding window counter | `current + previous × (1 − elapsed fraction)` | Accurate and cheap | Approximation |

**Token bucket (C++, single server):**
```cpp
class TokenBucket {
    double capacity, rate, tokens;
    chrono::steady_clock::time_point last;
    mutex m;
public:
    TokenBucket(double cap, double perSec)
        : capacity(cap), rate(perSec), tokens(cap), last(chrono::steady_clock::now()) {}
    bool allow() {
        lock_guard<mutex> g(m);
        auto now = chrono::steady_clock::now();
        double secs = chrono::duration<double>(now - last).count();
        tokens = min(capacity, tokens + secs * rate);   // refill lazily, no background thread
        last = now;
        if (tokens >= 1) { tokens -= 1; return true; }
        return false;
    }
};
// one bucket per key: unordered_map<string, TokenBucket>
```

**Fixed window (C++):**
```cpp
class FixedWindow {
    int limit, windowSec;
    unordered_map<string, pair<long long,int>> m;   // key -> {window id, count}
    mutex mu;
public:
    FixedWindow(int l, int w) : limit(l), windowSec(w) {}
    bool allow(const string& key, long long nowSec) {
        lock_guard<mutex> g(mu);
        long long win = nowSec / windowSec;
        auto& [w, c] = m[key];
        if (w != win) { w = win; c = 0; }
        if (c >= limit) return false;
        ++c; return true;
    }
};
```

### Distributed version
Each API server can't keep its own counters, or a user hitting 10 servers gets 10× the limit. Keep counters in **Redis**:
- Fixed window: `INCR rl:{user}:{minute}`; if the result is 1, `EXPIRE` it to 60 s; reject if > limit.
- Token bucket: store `tokens` and `last_refill` in a Redis hash and update them in a **Lua script**, which Redis runs atomically, so two servers can't both take the last token.

### Failures and details
- **Redis down?** Choose: **fail open** (allow traffic, keep the app usable — normal for most APIs) or **fail closed** (block — for login/OTP to stop brute force). Keep a rough per-server in-memory limiter as a fallback.
- **Latency:** one Redis round trip (< 1 ms). For very hot paths, servers can take tokens from Redis in small batches.
- **Headers:** `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`.
- **Rules** stored in config (per endpoint, per user tier), cached in each gateway.

### Follow-ups
- **"Why token bucket over fixed window?"** No edge burst, allows natural short bursts, still cheap.
- **"How would you rate-limit per restaurant for incoming orders?"** Same machinery, key = restaurant ID; protects a kitchen from more orders than it can cook — then show "busy, try later" or extend prep time.
- **Your Myntra story:** "I used a per-pod token bucket to protect a shared internal service; during a cache warmup I learned to pace the producer to the bucket's refill rate."

---

## 6. Notification system

*Asked 5+ times.*

### Requirements
- Send order updates, OTPs, offers via **push, SMS, email, WhatsApp, in-app**.
- Respect user preferences and quiet hours; never send duplicates; retry failures.
- **Priorities:** an OTP or "your order is out for delivery" must never wait behind a marketing blast to 1 crore users.
- Track delivery status.

### Estimates
Transactional: ~5–10 notifications per order × 20 lakh orders ≈ 1–2 crore/day (~200/s avg). Marketing campaigns: crores in a burst — must be throttled.

### Design
```
Order / Payment / Marketing services
        │ events or POST /notify {user_id, template_id, params, priority, dedupe_key}
        ▼
 Kafka topics by priority:  notif-critical (OTP, order) · notif-normal · notif-bulk (marketing)
        ▼
 Notification service (consumers)
   1. dedupe: Redis SET notif:{dedupe_key} NX EX 86400 → skip if exists
   2. load preferences + device tokens (Redis cache → DB)
   3. apply rules: opted out? quiet hours? per-user frequency cap?
   4. render template (language, personalisation)
   5. route to channel queues: push-q, sms-q, email-q, whatsapp-q
        ▼
 Channel workers ── FCM / APNs, SMS providers (primary + backup), email provider
   retries with exponential backoff → DLQ after N attempts
   write status to notification_log (queued / sent / delivered / failed / opened)
```

### Key points
- **Separate topics and workers per priority**, so a campaign backlog never delays OTPs. Critical workers have reserved capacity.
- **Idempotency:** every notification has a dedupe key (e.g. `order123:OUT_FOR_DELIVERY:user9`); Kafka is at-least-once, so the Redis `SET NX` check prevents double-sends.
- **Rate limiting** per user (no more than X marketing messages a day) and per provider (SMS vendors have throughput limits — the bulk workers pace themselves with a token bucket).
- **Provider failover:** if the primary SMS provider's error rate spikes (circuit breaker opens), route to the backup.
- **Fallback chain:** for critical messages, if push fails or isn't delivered within a minute, send SMS.
- **Templates** stored in DB, cached; versioned; multi-language.
- **Device tokens** change; when FCM says a token is invalid, delete it.
- **Big campaigns:** a scheduler splits the audience into batches and feeds `notif-bulk` at a controlled rate — no thundering herd on providers or on our own app servers when everyone taps the push at once.

### Data
```sql
user_preferences(user_id, channel, category, enabled)     -- e.g. (9, 'sms', 'marketing', false)
device_tokens(user_id, platform, token, updated_at)
templates(id, channel, language, body, version)
notification_log(id, user_id, dedupe_key UNIQUE, channel, status, provider_ref, created_at)
```
`notification_log` is huge and append-heavy → Cassandra or a partitioned table with a short retention.

### Follow-ups
- **"Notify users when a restaurant they like reopens — but only if riders are available."** Subscriptions table keyed by restaurant; the restaurant's `OPENED` event triggers a job; the job checks rider availability in that zone (from dispatch's cache); then fans out in batches (not all at once, or they all order at the same moment and the kitchen and riders are swamped).
- **"How do you know it was delivered/opened?"** Provider delivery callbacks (webhooks) and app open events update `notification_log`.
- **"Exactly once?"** Not achievable end to end with external providers; at-least-once plus dedupe keys gives effectively-once.

---

## 7. Payment system (with refunds)

### Requirements
- Charge customers via UPI, cards, wallets, net banking through payment gateways (Razorpay, PayU, etc.).
- Handle success, failure and **unknown** outcomes; refunds (full/partial); payouts to restaurants and riders.
- **Correctness above everything:** never charge twice, never lose money, full audit trail.

### Core ideas (the interviewer is listening for these words)
1. **Idempotency keys** on every payment and refund request, stored with a unique constraint.
2. **A payment state machine:** `CREATED → PENDING → SUCCESS | FAILED`, plus `REFUND_PENDING → REFUNDED`.
3. **Webhooks + reconciliation:** the gateway tells us the result asynchronously; we also poll for anything stuck.
4. **A double-entry ledger:** every money movement is recorded as two entries that balance.

### Flow
1. Order service calls `POST /payments {order_id, amount, idempotency_key}`. Payment service creates a `payments` row (`CREATED`) and calls the gateway to create a payment order; returns the gateway's token to the app. Status → `PENDING`.
2. The app completes payment with the gateway's SDK (UPI app, card OTP…). **Our servers never see card numbers** — the gateway handles that (PCI compliance).
3. The gateway calls our **webhook** with the result. We **verify its signature** (HMAC with a shared secret, so nobody can fake a "success"), then update the payment with a conditional update `WHERE status='PENDING'` and the gateway's reference as a unique key. Duplicate webhooks do nothing.
4. Publish `PaymentSucceeded` / `PaymentFailed` (via outbox) → Order service confirms or fails the order.
5. **Reconciliation job:** every few minutes, query the gateway for payments `PENDING` longer than N minutes and settle them. Daily, compare our records against the gateway's settlement report line by line; flag mismatches.

### The ledger
Never just `UPDATE wallet SET balance = balance - 100`. Record every movement as entries:
```sql
ledger_entries(id, txn_id, account, direction, amount, created_at)
-- order 123 paid ₹500:
--   DEBIT  customer_payable      500
--   CREDIT restaurant_payable    420
--   CREDIT zomato_commission      60
--   CREDIT delivery_fee           20
```
Debits equal credits for every `txn_id`, so the books always balance; balances are sums of entries (cached, but always recomputable). Entries are **append-only** — mistakes are fixed with new reversing entries, never edits. This gives a full audit trail.

### Refunds
`POST /refunds {payment_id, amount, reason, idempotency_key}` → check refundable amount (sum of previous refunds ≤ paid) inside a transaction → call the gateway's refund API → `REFUND_PENDING` → webhook → `REFUNDED` → ledger reversing entries. Partial refunds (one item missing) work the same way.

### Failures
- **Gateway timeout — did the charge happen?** Don't retry blindly and don't assume failure. Mark `PENDING/UNKNOWN`, show "payment processing", and let the webhook or reconciliation decide. Retry only *with the same idempotency key*, which the gateway also honours.
- **Money debited but order failed** → automatic refund saga; customer sees "refund initiated".
- **Gateway down** → circuit breaker; route to a second gateway; offer other payment methods.
- **Double-click** → same idempotency key.

### Follow-ups
- **"SQL or NoSQL for payments?"** SQL, without question — ACID transactions, constraints, strong consistency.
- **"Kafka vs SQS for payment events?"** Kafka if several services consume them and you want replay/audit; either works if it's one consumer. Ordering per payment via the payment ID key.
- **"How do you pay restaurants?"** Daily/weekly settlement jobs sum their ledger balance and trigger bank payouts, each payout idempotent and itself recorded in the ledger.

---

## 8. Seat / ticket booking (BookMyShow, Zomato District)

*Asked 3+ times; District sells movie and event tickets.*

### Requirements
- Users see a show's seat map, select seats, hold them while paying (~10 minutes), then confirm.
- **Two users must never get the same seat.**
- Huge spikes when a big movie or concert opens bookings.

### Data
```sql
shows(id, venue_id, movie_or_event_id, starts_at)
show_seats(show_id, seat_id, status, held_by NULL, hold_expires_at NULL,
           booking_id NULL, version,  PRIMARY KEY(show_id, seat_id))
-- status: AVAILABLE / HELD / BOOKED
bookings(id, user_id, show_id, status, amount, idempotency_key UNIQUE, created_at)
```

### The core: holding seats without double-booking
**Option A — atomic conditional update (recommended):**
```sql
BEGIN;
UPDATE show_seats
SET status='HELD', held_by=:user, hold_expires_at = NOW() + INTERVAL 10 MINUTE
WHERE show_id=:show AND seat_id IN (:s1, :s2, :s3) AND status='AVAILABLE';
-- if affected rows < 3: ROLLBACK (someone took one) and tell the user
COMMIT;
```
All or nothing; only one of two racing users can match the `AVAILABLE` rows.

**Option B — pessimistic lock:** `SELECT … FOR UPDATE` on those seat rows, check, update, commit. Correct but others queue behind the lock.

**Option C — Redis hold:** `SET seat:{show}:{seat} {user} NX EX 600` per seat (or a Lua script for all seats at once). Very fast for hot shows; the TTL releases holds automatically. The final `BOOKED` state still goes to the database with a conditional update, which is the real guarantee.

### Flow
1. **Seat map:** read from cache (refresh every few seconds — slight staleness is fine; the hold step is the real check).
2. **Hold:** the atomic update above → the user has 10 minutes.
3. **Pay** (with an idempotency key).
4. **Confirm:** on payment success, `UPDATE … SET status='BOOKED', booking_id=? WHERE … AND status='HELD' AND held_by=:user`.
5. **Expiry:** a sweeper job runs every minute: `UPDATE show_seats SET status='AVAILABLE', held_by=NULL WHERE status='HELD' AND hold_expires_at < NOW()`. (Or Redis TTL expiry.)
6. **Payment arrives after hold expired** and someone else took the seat → automatic refund, apology.

### Handling the opening-day rush
- A **virtual waiting room / queue:** let users in at a controlled rate (token bucket on entry), show their position.
- Shard by `show_id` — each show's seats are independent, so they spread naturally.
- Cache the seat map aggressively; only the hold step hits the database.
- Rate limit per user to stop bots.

### Follow-ups
- **"Optimistic or pessimistic?"** For most shows, the atomic conditional update (an optimistic style) — no long locks, fast failure. For extremely hot seats, Redis holds in front to keep load off the DB.
- **"General admission (no seat numbers) — e.g. 5,000 tickets for a concert?"** It's an inventory counter: `UPDATE events SET sold = sold + :n WHERE id=? AND sold + :n <= capacity`, or a Redis `DECRBY` with a check, then the same hold/expire/confirm logic.

---

## 9. Quick-commerce inventory (Blinkit, 10-minute delivery)

### Requirements
- Each **dark store** (small warehouse) has its own stock. Users see only items available at the store serving them.
- **Never sell what's not on the shelf** (overselling = cancelled order = angry user).
- Fast: 10-minute promise means picking must start within seconds.

### Design
- **Serviceability:** user location → which dark store serves this cell (H3/geohash → store mapping). If none, "not serviceable".
- **Inventory table:** `store_inventory(store_id, sku_id, on_hand, reserved, version, PRIMARY KEY(store_id, sku_id))`. Available = on_hand − reserved.
- **Catalogue/browse:** read availability from a Redis cache per store (refreshed by inventory events); show "only 2 left".
- **Checkout — reserve atomically:**
  ```sql
  UPDATE store_inventory SET reserved = reserved + :qty
  WHERE store_id=? AND sku_id=? AND on_hand - reserved >= :qty;
  -- do this for every cart line in one transaction; any 0-row result → rollback, show "out of stock"
  ```
- **Payment success** → order confirmed → picking. When packed, `on_hand -= qty, reserved -= qty`.
- **Payment failure / timeout / cancel** → release the reservation (`reserved -= qty`). A sweeper releases reservations older than ~10 minutes with no payment.
- **Restocking and audits** update `on_hand` through events from the warehouse system; stock counts by store staff correct drift.

### Hot items
A flash sale on one item → thousands of decrements on one row. Fixes: Redis `DECRBY` with a Lua check as the front gate, then persist; or split the stock into several "buckets" (rows) and decrement any bucket that has stock.

### Follow-ups
- **"How is this different from food delivery?"** Inventory is the hard constraint (not a kitchen's capacity); the store is fixed and close, so dispatch is simpler; the time budget is tiny, so picking/packing workflow and in-store routing matter.
- **"Cache says in stock, DB says no?"** The DB reservation is the truth; the cache is only for display. Show "just sold out" and suggest substitutes.

---

## 10. Coupons and promo codes

### Requirements
- Codes like `WELCOME50` (₹50 off first order), `BIRYANI20` (20% off up to ₹100 on biryani restaurants), bank offers, one-per-user codes, limited-quantity codes (first 10,000 users).
- Validate fast at checkout; never over-redeem.

### Data
```sql
coupons(id, code UNIQUE, type, value, max_discount, min_order_value,
        starts_at, ends_at, total_limit, per_user_limit, rules_json, is_active)
coupon_redemptions(id, coupon_id, user_id, order_id UNIQUE, status, created_at)
coupon_counters(coupon_id, redeemed_count)   -- or a Redis counter
```

### Logic
1. **Validate** (synchronous, at checkout): active and in date; order value ≥ minimum; rules match (restaurant, cuisine, city, payment method, new user); user's redemption count < per-user limit.
2. **Compute discount** with a Strategy per type: flat, percentage with cap, free delivery, BOGO.
3. **Reserve** a redemption when the order is placed: insert into `coupon_redemptions` (status `RESERVED`) and increment the counter conditionally: `UPDATE coupon_counters SET redeemed_count = redeemed_count + 1 WHERE coupon_id=? AND redeemed_count < total_limit`.
4. **Confirm** on payment success; **release** on failure/cancel (decrement, mark `RELEASED`).
5. **Per-user limit** enforced with a unique index, e.g. `UNIQUE(coupon_id, user_id)` for one-per-user codes.

### Follow-ups
- **"Limited to first 10,000 — how do you avoid overselling under a burst?"** The conditional counter update (or Redis `INCR` and reject if > limit, with a compensating `DECR` on failure).
- **"Show the best coupon automatically?"** Evaluate all applicable coupons for the cart and pick the maximum discount — a small rules engine; cache eligible coupons per user segment.
- **"Abuse — many accounts for WELCOME50?"** Tie first-order offers to device ID and phone/payment instrument, not just account; fraud rules.

---

## 11. Ratings and reviews

*Tie this to your Myntra work — it's literally what you built.*

### Requirements
- Users rate (1–5) and review delivered orders; show average rating and count per restaurant (and per dish); show reviews sorted by recent/helpful; moderation.
- Reads hugely outnumber writes.

### Design
- **Write:** `POST /reviews {order_id, rating, text, photos}` → check the order was delivered to this user (only real customers can review) → insert into `reviews` (unique on order_id) → publish `ReviewCreated`.
- **Aggregates:** don't compute `AVG()` over millions of rows on every page view. Keep a summary row per restaurant: `restaurant_rating(restaurant_id, count_1..count_5, total, avg, updated_at)`. A consumer updates it from `ReviewCreated` events (idempotently — dedupe on review ID). Average = Σ(k × count_k) / total. Storing the histogram (count per star) lets you update incrementally and show the star breakdown.
- **Cache** the summary in Redis; it's eventually consistent and that's fine.
- **Reviews list:** paginated with a cursor, sorted by time or helpfulness; photos in S3 + CDN.
- **Moderation:** async — a classifier flags abusive text or fake reviews; flagged reviews hidden pending review.
- **Anti-gaming:** only verified orders can review; weight recent ratings more; detect bursts from related accounts.

### Follow-ups
- **"Chain with 50 outlets — show one brand rating?"** Exactly the Myntra problem: aggregate the histograms across all outlets in the group (never average the averages), cache the result per outlet, and recompute on any outlet's new rating.
- **"Bayesian average?"** A restaurant with one 5★ review shouldn't outrank one with 4.6★ from 5,000. Use `(C×m + Σratings) / (C + n)`, where m is the global average and C a confidence constant, for ranking.

---

## 12. Search autocomplete (typeahead)

### Requirements
- As the user types "bir", suggest "biryani", "Biryani Blues", "biryani near me" within ~50–100 ms.
- Suggestions ranked by popularity (and personalised/local if possible).

### Design
- **Data:** aggregate past search queries and restaurant/dish names with counts (from Kafka search logs, batch job daily or hourly).
- **Structure:** a **trie** (prefix tree): each node is a character; at each node store the **top-K suggestions** for that prefix, precomputed. A lookup walks the prefix (O(length)) and returns the stored list — no searching at query time.
- **Serving:** the trie (or a prefix → top-K map) is in memory on suggestion servers, rebuilt periodically and swapped in; or Redis sorted sets keyed by prefix; or Elasticsearch's completion suggester.
- **Location:** keep separate tries/top-K lists per city ("Truffles" matters in Bangalore).
- **Client:** debounce (call after ~150–200 ms of no typing), cache recent prefixes on the device, cancel stale requests.
- **Typos:** fuzzy match if the exact prefix returns nothing.

### Follow-ups
- **"Update in real time when something trends?"** Count queries in a streaming job over short windows and merge trending terms into the top-K lists more often (e.g. every few minutes) for hot prefixes.
- **DSA link:** implement a trie with insert and prefix search (`01b`).

---

## 13. Delayed jobs: auto-cancel if the restaurant doesn't accept within 5 minutes

### Requirements
- When an order is confirmed, if the restaurant hasn't accepted in 5 minutes, escalate (call them) and at 8 minutes auto-cancel and refund. Similar needs: release seat holds after 10 minutes, send "rate your order" 1 hour after delivery.

### Options
1. **Database + poller:** a `scheduled_jobs(id, run_at, type, payload, status)` table with an index on `run_at`. Workers poll every few seconds: `SELECT … WHERE run_at <= NOW() AND status='PENDING' LIMIT 100 FOR UPDATE SKIP LOCKED` (SKIP LOCKED lets several workers take different rows without blocking each other), run each job, mark done. Simple and reliable.
2. **Redis sorted set as a delay queue:** `ZADD delayed <run_at_timestamp> <job_id>`; workers `ZRANGEBYSCORE delayed 0 <now>` and claim jobs atomically (Lua or `ZREM` returning 1). Fast for many jobs.
3. **Queue delay features:** SQS delay messages (up to 15 min), or Kafka with tiered retry/delay topics (e.g. `delay-1m`, `delay-5m`) consumed only after their delay.

### The important detail
When the job runs, **check the current state** before acting: auto-cancel only if the order is *still* `CONFIRMED` — `UPDATE orders SET status='CANCELLED' WHERE id=? AND status='CONFIRMED'`. If the restaurant accepted in the meantime, the update does nothing. This makes the job safe even if it runs twice or late. Cancelling a job is optional; making it a no-op is more robust.

---

## 14. Peak traffic: New Year's Eve (or an IPL final)

*Usually a follow-up: "how would your design survive 10× traffic?"*

**Before the day:**
- **Load test** at 2–3× the expected peak; find the first thing that breaks.
- **Pre-scale:** add servers, database replicas, Kafka partitions, cache capacity *ahead of time* — autoscaling reacts too slowly to a 10× jump in minutes.
- **Warm caches** (menus, restaurant lists for every city cell) so the peak doesn't start with misses.
- **Freeze deploys** for the risky window.

**During:**
- **Rate limiting and admission control** at the gateway: protect the core; queue or shed excess non-critical traffic.
- **Queues absorb spikes** (orders keep being accepted; downstream work catches up).
- **Graceful degradation:** switch off non-critical features via feature flags — recommendations, review writing, heavy personalisation, analytics writes, image-heavy banners.
- **Restaurant-side throttling:** if a kitchen is overloaded, increase its displayed prep time or temporarily hide it, rather than accept orders it can't cook.
- **Rider supply:** surge incentives, batching multiple orders per rider, longer ETAs shown honestly.
- **Watch the dashboards:** error rate, p99 latency, consumer lag, DB CPU and connections, cache hit ratio, payment success rate. On-call ready with runbooks.

**After:** reconcile payments, clear DLQs, write down what broke.

---

## 15. URL shortener (classic warm-up)

**Requirements:** `POST /shorten {long_url}` → `zma.to/aZ3k9Q`; `GET /aZ3k9Q` → redirect. Billions of links, very read-heavy (100:1), redirects in < 50 ms, links never collide.

**Short code:** 7 characters of base62 (a–z, A–Z, 0–9) gives 62⁷ ≈ 3.5 trillion codes.
- **Option A — counter + base62:** a unique ID generator gives an increasing number; encode it in base62. No collisions. Use ranges per server (server 1 gets IDs 1–1M, server 2 1M–2M…) so they don't coordinate on every request. Downside: codes are guessable — fine for most uses, or shuffle the bits.
- **Option B — hash:** first 7 chars of a hash of the URL; on collision, add a salt and retry. Same URL → same code (deduplication), but collisions need handling.

**Storage:** `links(code PK, long_url, user_id, created_at, expires_at)` in a key-value store (DynamoDB/Cassandra) — simple lookups by key, huge scale. **Cache** hot codes in Redis; a CDN can even cache redirects.

**Redirect:** **301** (permanent; browsers cache it — less load, but you lose click tracking) vs **302** (temporary; every click hits you — use it if you need analytics). Click events go to Kafka for analytics, asynchronously.

---

## 16. Chat (customer support / WhatsApp-style)

**Requirements:** one-to-one messages (customer ↔ support agent or rider), delivered in real time, stored history, delivered/read receipts, works when the recipient is offline.

**Design:**
- Clients keep a **WebSocket** to a **chat gateway** server. A **connection registry** in Redis maps `user_id → gateway server`.
- **Send:** A sends to their gateway → chat service assigns a message ID and timestamp, **stores it** (Cassandra, partitioned by `conversation_id`, sorted by time — built for heavy writes and "last N messages" reads) → looks up B's gateway in the registry → forwards; B's gateway pushes it down B's socket.
- **B offline:** message stays stored as undelivered; send a push notification; when B reconnects, the client fetches messages after its last received ID.
- **Receipts:** B's client sends "delivered" and "read" acks, which update status and are forwarded to A.
- **Ordering:** per conversation, by server-assigned sequence numbers.
- **Group chat:** fan out to each member's connection (fine for small groups); large groups fan out via a queue.
- **Support twist:** a routing service assigns the chat to an available agent (skills, language, load) — like rider dispatch.

---

## 17. Music streaming (Spotify)

*Reported 3 times.*

- **Metadata** (songs, artists, albums, playlists) in SQL, cached; **search** in Elasticsearch.
- **Audio files** in object storage (S3), encoded at several bitrates, split into small **chunks** (a few seconds each), served via **CDN**. The player fetches chunks ahead (buffering) and switches bitrate with network quality (**adaptive bitrate streaming**).
- **Playback API** returns a signed, time-limited CDN URL so only paying users can fetch.
- **Playlists:** `playlists(id, owner_id, name)`, `playlist_songs(playlist_id, position, song_id)`.
- **Shuffle:** Fisher–Yates shuffle — for i from n−1 down to 1, swap item i with a random item 0..i — gives every order equal probability.
- **Play counts and history:** events to Kafka → aggregated in batches (don't update a counter row per play) → royalties and recommendations.
- **Offline downloads:** encrypted files on the device with a licence that expires.

---

## 18. Image upload service (e.g. review photos, restaurant menus)

1. App asks `POST /uploads {content_type, size}` → server checks auth and limits, returns a **pre-signed S3 URL** valid for a few minutes and a file ID.
2. App uploads **directly to S3** — the bytes never pass through our servers.
3. S3 fires an "object created" event → queue → workers: virus scan, strip location metadata, moderation (no inappropriate images), resize into thumbnail/medium/large, convert to efficient formats (WebP).
4. Metadata row `images(id, owner_id, s3_key, status, sizes, created_at)` → status `READY`.
5. Served via **CDN**. Deletion: mark deleted, remove from S3 asynchronously, purge CDN.

Failures: upload abandoned → lifecycle rule deletes unconfirmed files after a day. Worker crash → queue redelivers; processing is idempotent (same input → same outputs, overwrite).

---

## 19. Feature flags

**Use:** turn features on/off without deploying; roll out to 1% → 10% → 100%; target by city/app version; instant kill switch during incidents (perfect for NYE degradation).

- **Store:** `flags(name, enabled, rollout_percent, rules_json, updated_at)` in a database, edited via an admin UI with an audit log.
- **Evaluation in each service:** an SDK keeps flags **in local memory**, refreshed every ~30 s or pushed via pub/sub on change — so checking a flag is a memory lookup, not a network call, and services keep working if the flag service is down (use last known values).
- **Percentage rollout:** `hash(flag_name + user_id) % 100 < rollout_percent` — deterministic, so a user consistently sees the same variant; adding the flag name means different flags pick different users.
- **Hygiene:** remove flags once fully rolled out.

---

## 20. Leaderboard / trending restaurants

**Example:** "top 10 most-ordered restaurants in your city this week", or a rider leaderboard by deliveries.

- **Redis sorted set** per city/period: on each `OrderDelivered` event, `ZINCRBY trending:blr:2026-w43 1 restaurant:42`. Top 10: `ZREVRANGE trending:blr:2026-w43 0 9 WITHSCORES`. A user's rank: `ZREVRANK`. Each operation is O(log n).
- **Sliding "last 24 hours":** keep one sorted set per hour and combine the last 24 with `ZUNIONSTORE` (cached for a minute), and expire old hourly sets.
- **Huge scale:** shard by city; or for approximate top-K over enormous streams, a **count-min sketch** plus a heap.
- **Persistence:** periodically snapshot results to the database; Redis isn't the source of truth — the order events are, so you can rebuild.
- **DSA link:** top-K with a min-heap of size K (`01b`).

---

## 21. Distributed cache, load balancer and consistent hashing

**Design a distributed cache (like a mini Redis cluster):**
- **API:** `get(key)`, `set(key, value, ttl)`, `delete(key)`.
- **Single node:** a hash map plus a doubly linked list for **LRU** eviction (exactly the LRU DSA question), plus TTL checking (lazily on read and with a periodic sweep).
- **Many nodes:** distribute keys with **consistent hashing** (with virtual nodes) so adding a node moves only ~1/N of keys. Clients (or a proxy) hash the key to find its node.
- **Replication:** each key also stored on the next 1–2 nodes on the ring; if a node dies, replicas serve its keys.
- **Hot keys:** replicate hot keys more widely or cache them in the client.
- **Consistency:** caches usually choose availability — a stale or missing value just falls back to the database.

**Design a load balancer:** health checks; algorithms (round robin, weighted, least connections, consistent-hash for stickiness); L4 vs L7; run in an active–passive pair sharing a virtual IP (if the active one dies, the passive takes over the IP); TLS termination; connection draining during deploys (stop sending new requests to a server but let existing ones finish).

---

# Low-level design and schema questions

## 22. LLD: restaurant menu with customisations (Domino's: size, crust, toppings)

**Requirements:** items have option groups (Size: choose exactly 1; Crust: exactly 1; Toppings: 0 to 5); each option changes the price; options can be unavailable; the order must remember exactly what was chosen and its price.

**Schema:**
```sql
items(id, restaurant_id, name, base_price, is_veg, is_available)
option_groups(id, item_id, name, min_select, max_select)       -- Size (1,1), Toppings (0,5)
options(id, group_id, name, price_delta, is_available)        -- Large +150, Cheese Burst +99
order_items(id, order_id, item_id, qty, unit_price_at_order)
order_item_options(order_item_id, option_id, price_delta_at_order)
```

**Classes (Java-style):**
```java
class Item {
    long id, restaurantId; String name; int basePrice; List<OptionGroup> groups;
    int priceFor(Map<Long, List<Option>> chosen) {          // groupId -> chosen options
        int price = basePrice;
        for (OptionGroup g : groups) {
            List<Option> picks = chosen.getOrDefault(g.id, List.of());
            g.validate(picks);                                // throws if invalid
            for (Option o : picks) price += o.priceDelta;
        }
        return price;
    }
}
class OptionGroup {
    long id; String name; int minSelect, maxSelect; List<Option> options;
    void validate(List<Option> picks) {
        if (picks.size() < minSelect || picks.size() > maxSelect)
            throw new IllegalArgumentException(name + ": choose " + minSelect + "-" + maxSelect);
        for (Option o : picks)
            if (!options.contains(o) || !o.available) throw new IllegalArgumentException("invalid option");
    }
}
class Option { long id; String name; int priceDelta; boolean available; }
```
**Talking points:** composition (Item has OptionGroups has Options); price computed on the server; snapshot prices into the order; nested options (a topping with "extra") would make groups recursive — the **Composite** pattern; a **Decorator** view (each topping wraps the pizza and adds cost) is the textbook alternative.

## 23. LLD: show a banner X times in Y hours, for Z days

*Asked 2025.*

**Requirements:** each banner runs between a start and end date (Z days); each user should see it at most X times in any Y-hour window; several banners compete by priority.

**Schema:**
```sql
banners(id, content_url, start_at, end_at, max_views, window_hours, priority, target_rules_json)
```
**Counting views — Redis:**
- Simple fixed window: key `bv:{banner}:{user}:{window_index}` where `window_index = floor(now / (Y hours))`; `INCR` on each view, `EXPIRE` after Y hours. Show if count < X.
- Exact sliding window: a sorted set `bv:{banner}:{user}` of view timestamps; on each request remove entries older than now − Y (`ZREMRANGEBYSCORE`), count (`ZCARD`), and add the new view if under X. Expire the whole key after Y hours of inactivity.

**Serving:** `GET /banners?user=&placement=` → fetch active banners (`start_at <= now <= end_at`, cached list), filter by targeting, drop those the user has hit X for, return the highest priority. Record the view when it's actually displayed (the app sends an impression event), not when it's fetched.

**Point to make:** this is the same machinery as a rate limiter, applied to impressions.

## 24. LLD: cart with discounts

```java
interface Discount { int apply(Cart cart); }                 // returns amount off, in paise
class FlatDiscount implements Discount {
    int amount, minOrder;
    public int apply(Cart c) { return c.subtotal() >= minOrder ? Math.min(amount, c.subtotal()) : 0; }
}
class PercentDiscount implements Discount {
    int percent, cap;
    public int apply(Cart c) { return Math.min(c.subtotal() * percent / 100, cap); }
}
class FreeDelivery implements Discount {
    public int apply(Cart c) { return c.deliveryFee(); }
}

class Cart {
    long userId, restaurantId; List<CartLine> lines = new ArrayList<>(); Discount discount;
    void add(Item item, Map<Long, List<Option>> opts, int qty) {
        if (lines.isEmpty()) restaurantId = item.restaurantId;
        else if (item.restaurantId != restaurantId)
            throw new IllegalStateException("cart has items from another restaurant");
        lines.add(new CartLine(item, opts, qty, item.priceFor(opts)));
    }
    int subtotal() { return lines.stream().mapToInt(l -> l.unitPrice * l.qty).sum(); }
    int deliveryFee() { return DeliveryFeeCalculator.forCart(this); }     // Strategy too
    int total() {
        int off = discount == null ? 0 : discount.apply(this);
        return Math.max(0, subtotal() + deliveryFee() + taxes() - off);
    }
}
```
**Talking points:** Strategy pattern for discounts (new coupon type = new class — open/closed); money in integer paise, never floating point (0.1 + 0.2 ≠ 0.3); one restaurant per cart (a real Zomato rule); recompute everything on the server at checkout.

## 25. LLD: parking lot

**Requirements:** multiple floors; spot sizes (bike, car, truck); assign the nearest free spot that fits; issue a ticket; compute the fee on exit; show free counts per floor.

```java
enum SpotSize { SMALL, MEDIUM, LARGE }
abstract class Vehicle { String plate; abstract SpotSize size(); }
class Bike extends Vehicle { SpotSize size() { return SpotSize.SMALL; } }
class Car  extends Vehicle { SpotSize size() { return SpotSize.MEDIUM; } }

class Spot { int id, floor; SpotSize size; Vehicle parked; boolean free() { return parked == null; } }

class Ticket { String id; Vehicle v; Spot spot; Instant entry; }

interface FeeStrategy { int fee(Duration d, SpotSize s); }             // hourly, flat, weekend…

class ParkingLot {
    Map<SpotSize, TreeSet<Spot>> freeSpots;    // TreeSet with a comparator on (floor, id) → nearest first
    Map<String, Ticket> active = new HashMap<>();
    FeeStrategy feeStrategy;

    synchronized Ticket park(Vehicle v) {
        for (SpotSize s : sizesThatFit(v.size())) {   // helper: this size and larger, smallest first
            Spot spot = freeSpots.get(s).pollFirst();
            if (spot != null) {
                spot.parked = v;
                Ticket t = new Ticket(UUID.randomUUID().toString(), v, spot, Instant.now());
                active.put(t.id, t);
                return t;
            }
        }
        throw new IllegalStateException("lot full");
    }
    synchronized int unpark(String ticketId) {
        Ticket t = active.remove(ticketId);
        t.spot.parked = null;
        freeSpots.get(t.spot.size).add(t.spot);
        return feeStrategy.fee(Duration.between(t.entry, Instant.now()), t.spot.size);
    }
}
```
**Talking points:** `TreeSet` (or a min-heap) gives the nearest free spot in O(log n); `synchronized` (or per-size locks) prevents two cars getting one spot; Strategy for fees; Factory for vehicles; Observer to update the display boards.

## 26. Schema: delivery partner and orders

*Asked.*

```sql
riders(id PK, name, phone UNIQUE, vehicle_type, city_id, status, rating, joined_at)
rider_shifts(id PK, rider_id, starts_at, ends_at)
rider_documents(rider_id, doc_type, url, verified)
orders(id PK, user_id, restaurant_id, address_id, status, total, created_at, ...)
deliveries(id PK, order_id UNIQUE, rider_id, status, assigned_at, picked_up_at,
           delivered_at, distance_km)
delivery_events(delivery_id, event, at, lat, lng)          -- audit trail of the trip
rider_payouts(id PK, rider_id, delivery_id UNIQUE, base, distance_pay, incentive, tip, paid_at)
rider_locations                                             -- NOT a table on MySQL:
    latest in Redis GEO; history streamed via Kafka to a time-series store
```
**Indexes:** `deliveries(rider_id, assigned_at)` for a rider's history; `orders(restaurant_id, created_at)` for the restaurant dashboard; `orders(user_id, created_at)` for the customer's history.

**Typical queries they ask you to write:**
```sql
-- Top 5 riders in a city by deliveries last week
SELECT r.id, r.name, COUNT(*) AS deliveries
FROM deliveries d JOIN riders r ON r.id = d.rider_id
WHERE r.city_id = 3 AND d.status = 'DELIVERED'
  AND d.delivered_at >= NOW() - INTERVAL 7 DAY
GROUP BY r.id, r.name
ORDER BY deliveries DESC
LIMIT 5;

-- Average delivery time per restaurant today
SELECT o.restaurant_id,
       AVG(TIMESTAMPDIFF(MINUTE, o.created_at, d.delivered_at)) AS avg_minutes
FROM orders o JOIN deliveries d ON d.order_id = o.id
WHERE d.delivered_at >= CURDATE()
GROUP BY o.restaurant_id;
```
**Why separate `deliveries` from `orders`?** Different lifecycles and owners (the delivery service), and one order could need reassignment (a new delivery row) without touching the order's history.

## 27. "Which restaurants are open right now?"

*Seen in interviews; mixes schema and data structures.*

**Schema:** `opening_hours(restaurant_id, day_of_week, opens_at, closes_at)` — several rows per day allowed (lunch and dinner shifts), plus `holiday_overrides(restaurant_id, date, is_closed)` and a live `is_online` flag (restaurant can pause itself).

**Past midnight:** 10 pm–2 am on Friday becomes two rows: Friday 22:00–24:00 and Saturday 00:00–02:00. Then every interval is within a day.

**Query:**
```sql
SELECT DISTINCT restaurant_id FROM opening_hours
WHERE day_of_week = :today AND opens_at <= :now_time AND closes_at > :now_time;
-- then AND is_online, and not in today's holiday overrides
```
**At scale:** this changes only at shift boundaries, so precompute: every minute (or at each boundary), a job computes the open set per city and stores it in Redis (`SET open:blr`), and search filters against it. Alternatively an **interval tree** per city answers "intervals containing time t" in O(log n + k) if asked for the data structure.

---

## 28. Rapid-fire follow-ups (40 short answers)

1. **How do you scale reads?** Cache (Redis, CDN), read replicas, denormalised read models.
2. **How do you scale writes?** Shard; batch writes; queue them; use write-optimised stores (Cassandra) for logs.
3. **How do you pick a shard key?** Even spread + queries that stay on one shard; avoid keys with hot values.
4. **What if one shard is hot?** Split it, add a random suffix to spread a hot key, cache, or move it to its own shard.
5. **SQL vs NoSQL for chat messages?** NoSQL (Cassandra): huge writes, simple access by conversation and time.
6. **Why not store sessions in app memory?** Breaks with multiple servers and crashes; use Redis or tokens.
7. **What's a CDN good for?** Static content near users: images, JS bundles, video chunks.
8. **What is a reverse proxy?** A server in front of app servers that forwards requests (load balancing, TLS, caching).
9. **API gateway responsibilities?** Auth, rate limiting, routing, request logging, sometimes response aggregation.
10. **Monolith or microservices for a startup?** Monolith first; split when team size or scaling needs demand it.
11. **How do services find each other?** Service discovery (a registry or Kubernetes DNS) plus load balancing.
12. **Sync vs async between services?** Sync when the caller needs the answer now; async for side effects and fan-out.
13. **What happens if Kafka is down?** Producers buffer briefly or fail; use the outbox so events are sent when Kafka returns; Kafka itself is replicated to make this rare.
14. **What's consumer lag?** How far consumers are behind the latest message; alert when it grows.
15. **How many partitions?** Enough for peak parallelism (≥ max consumers you'll run), with headroom; hard to reduce later.
16. **How do you handle a poison message?** Retry a few times, then DLQ, alert, fix, replay.
17. **Exactly-once delivery?** Practically: at-least-once + idempotent consumers.
18. **What's a saga?** A multi-service transaction as local steps with compensating undo actions.
19. **What's 2PC and why avoid it?** Coordinator-driven atomic commit; slow, blocks if the coordinator fails.
20. **Write-through vs write-back?** Write-through: cache and DB together, safe. Write-back: cache first, flush later, fast but can lose data.
21. **How do you avoid stale cache after an update?** Delete on write, short TTLs, change events.
22. **What's a Bloom filter?** A compact structure that says "definitely not present" or "maybe present"; stops lookups for missing keys.
23. **How does Redis stay fast?** In memory, single-threaded command execution (no lock overhead), efficient data structures.
24. **Redis cluster vs replica?** Cluster shards keys across nodes (scale); replicas copy a node (availability).
25. **How do you secure service-to-service calls?** mTLS or signed tokens; network policies; least-privilege credentials.
26. **How do you prevent double-charging?** Idempotency keys end to end, unique constraints, webhook dedupe, reconciliation.
27. **What's a webhook?** An HTTP call another system makes to you when something happens (payment succeeded).
28. **How do you verify a webhook is real?** Check its HMAC signature with the shared secret, and its timestamp.
29. **Long polling vs WebSocket?** Long polling is HTTP request-held-open; WebSocket is a persistent two-way connection.
30. **How do you scale WebSocket servers?** Horizontally; route messages via pub/sub (Redis/Kafka) to whichever server holds the connection.
31. **Geohash edge problem?** Search neighbouring cells too.
32. **Why Redis GEO for riders but not restaurants?** Riders move every few seconds (cheap in-memory updates); restaurants are static and need rich filters (Elasticsearch/DB).
33. **How do you generate unique IDs across servers?** UUIDs, or Snowflake-style IDs (timestamp + machine ID + sequence) that are unique and roughly time-ordered.
34. **What is CDC?** Change data capture: streaming the database's change log (e.g. MySQL binlog via Debezium) into Kafka to update caches and search indexes.
35. **How do you do zero-downtime deploys?** Rolling or blue-green deploys behind the load balancer, with health checks, connection draining and backward-compatible DB changes.
36. **How do you change a database schema safely?** Expand–contract: add new column, write both, backfill, switch reads, then remove the old column.
37. **What would you monitor for the order service?** Order rate, success/failure rate, p99 latency, payment success rate, consumer lag, DB CPU/connections.
38. **What's an SLO?** A target like "99.9% of order placements succeed in < 1 s over 30 days"; the error budget is the allowed 0.1%.
39. **How do you protect PII?** Encrypt at rest, mask in logs, restrict access by role, proxy phone numbers.
40. **Your design is too complex — simplify?** Start with a monolith + MySQL + Redis + one queue; add services, shards and Kafka only where scale demands. Interviewers like candidates who can say this.

---

**Final advice for the room:** draw boxes before details; say the numbers; name the failure cases before they ask; use the words *idempotency, outbox, at-least-once, cache-aside, rate limit, circuit breaker, eventual consistency* where they genuinely fit; and connect to your own work — "at Myntra I did exactly this with a token bucket and an error queue" is worth more than any textbook answer.
