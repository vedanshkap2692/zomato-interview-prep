# Global delivery and ride-hailing engineering: sourced facts for "Design Zomato"

Compiled 2026-10-23. Every fact lists its source date and URL.

**Verification tags**
- **[P]**: I read the primary post myself, either directly or through the Wayback Machine copy. DoorDash and Instacart block direct fetches, so those were read from archive.org.
- **[S]**: I only saw the claim in a search-engine summary of the primary post, not the full text. Treat it as lower confidence.
- **[3rd]**: The claim comes from a press or third-party write-up, or is a company marketing claim.

Facts are numbered F1 to F70 for easy citation.

---

## 1. Geo-indexing

- **F1 [P]** Uber's H3 is a hexagonal grid. It has 122 base cells, 12 pentagons (one at each icosahedron vertex), and **16 resolutions (0 to 15)**. Each finer resolution has cells about **1/7 the area** of the coarser one, and each cell index is a **64-bit integer**. (Uber, 2018-06-27, https://www.uber.com/blog/h3/)
- **F2 [P]** Why Uber chose hexagons:
  - They "minimize the quantization error introduced when users move through a city".
  - A hexagon has a single centre-to-neighbour distance. A square has two and a triangle has three. This simplifies smoothing and gradient analysis.
  - Source: same post.
- **F3 [P]** Uber calculates surge pricing "by measuring supply and demand in hexagons in each city". `kRing(k)` returns all cells within grid distance k, and the post shows k = 0, 1 and 2. That is the "nearby cells" query. (same post)
- **F4 [P]** H3 compaction: in the post's example, a set of **10,633** resolution-6 cells compacts to **901** cells of mixed resolution. This saves storage when you store a delivery zone polygon as cells. (same post)
- **F5 [P]** H3 cell sizes from the official resolution table:

  | Resolution | Avg area | Avg edge | Cells worldwide |
  |---|---|---|---|
  | 7 | 5.16 km² | 1.41 km | 98.8M |
  | 8 | 0.737 km² | 0.53 km | 692M |
  | 9 | 0.105 km² | 0.20 km | 4.84B |
  | 10 | 0.015 km² | 0.076 km | — |

  Source: H3 docs, https://h3geo.org/docs/core-library/restable/ (accessed 2026-10). Rule of thumb: res 7 or 8 for city-zone surge, res 9 for rider and restaurant neighbourhoods.
- **F6 [P]** Before H3 (around 2015), Uber dispatch used **Google S2 level-12 cells (3.31 to 6.38 km² each)**, and the **cell ID was the sharding key**. (High Scalability talk write-up of Matt Ranney's Uber talk, 2015-09-14, http://highscalability.com/blog/2015/9/14/how-uber-scales-their-real-time-market-platform.html)
- **F7 [P]** Grab uses **geohash** to locate drivers and passengers, and to compute distance, ETA and fare. Each cell subdivides into **32** child cells. A longer string means a more precise location. Grab notes one weakness: buildings across a street can fall into different geohashes and get different fares. (Grab, 2023-03-08, https://www.grab.com/inside-grab/stories/grab-geohashing-location-grid/)
- **F8 [P]** Grab's **Pharos** service (in-memory K-nearest-driver search by *road* distance or ETA):
  - It replaced "Sextant", which used haversine (straight-line) distance. That could pick drivers who are close as the crow flies but far by road.
  - Driver locations sit in **Adaptive Radix Trees** with two indexes: driver ID → driver object, and road segment → drivers on that segment.
  - Search uses incremental network expansion along the road graph, up to a range such as **3 km**.
  - Production P99 latency is **10 ms per driver update and 50 ms per nearby query**.
  - Location updates arrive "e.g. per second", and drivers can move at more than 20 m/s.
  - Data is partitioned **by city and vehicle type**. Each partition has 2 replicas in different AZs.
  - It "favors high throughput over strong consistency", and failed replica updates are not retried.
  - Source: Grab Engineering, 2020-12-22, https://engineering.grab.com/pharos-searching-nearby-drivers-on-road-network-at-scale
- **F9 [S]** Lyft's marketplace uses **geohash-6 cells** ("a few city blocks") as the spatial unit for pricing and supply signals. It later restricted these to *drivable* geohashes after its map showed demand over water. Lyft's on-device map data is served from DynamoDB, keyed by an **S2** index. (Lyft Engineering, 2026 reachability post, https://eng.lyft.com/refreshing-the-travel-time-map-behind-lyfts-marketplace-rebuilding-neighborhood-reachability-5be3efbc82ea, and https://eng.lyft.com/using-client-side-map-data-to-improve-real-time-positioning-a382585ac6e)

**How to use this in an interview answer:**
- Index rider, restaurant and driver positions by H3 cell. Use res 9 (about 0.1 km², 200 m edge) for "nearby" and res 7 or 8 for surge and zone aggregates.
- Query `kRing(cell, k)` rather than a radius. This avoids the geohash edge problem, where a neighbour across a boundary has a different prefix.
- Shard the live-location store by cell ID (Uber did this with S2).
- Mention that straight-line nearest is not road-nearest (Grab Pharos). Use the grid for candidate generation, then rank candidates by ETA.

---

## 2. Dispatch and matching

- **F10 [P]** DoorDash's first dispatcher assigned one delivery at a time. It was a **bipartite matching problem solved with the Hungarian algorithm**. This had two limits: the runtime was too high for large real-time instances, and it could not handle routes with 2 or more deliveries. (DoorDash, 2020-02-28, https://doordash.engineering/2020/02/28/next-generation-optimization-for-dasher-dispatch-at-doordash/ [read through the archive.org copy])
- **F11 [P]** DoorDash reformulated dispatch as a **mixed-integer program (MIP)**:
  - Binary variables represent dasher-to-order decisions, and the objective trades off delivery speed against dasher efficiency.
  - It is solved with **Gurobi**, which is "up to 10 times faster" than the Hungarian algorithm and **34x faster on average than open-source CBC**.
  - The optimizer "is run on several instances distributed based on regional boundaries **multiple times a minute**".
  - Source: same post.
- **F12 [P]** DoorDash's safe cutover had three steps:
  1. Run the new MIP optimizer alongside the old one in a combined "shadow" optimizer.
  2. Return the old optimizer's results and log the differences.
  3. Compare the outputs. They matched **more than 99% of the time**, and every mismatch was a tie between equally good solutions.

  The switch had zero downtime on a tier-0 service. (same post)
- **F13 [P]** DoorDash pointed out an advantage of a faster solver: it can solve **larger regions** in the same time, which reduces edge effects at region boundaries. (same post)
- **F14 [P]** DoorDash's **DeepRed** works in three layers:
  1. Offer candidate generation: nearby or soon-free Dashers, plus other orders at the same store or block.
  2. An ML layer that predicts order-ready (prep) time, travel time (including separate models for parking and walking), and **Dasher acceptance probability**.
  3. A MIP optimization layer that scores and ranks offers, batches orders, and decides whether to **delay dispatch** so the Dasher doesn't wait at the store.

  Source: DoorDash, 2021-08-17, https://doordash.engineering/2021/08/17/using-ml-and-optimization-to-solve-doordashs-dispatch-problem/
- **F15 [P]** DeepRed handles uncertainty in three ways:
  - Its scoring adds a **variance penalty that grows with route complexity**, because errors cascade across batched legs.
  - It tunes optimizer parameters with **Bayesian optimization** to avoid overfitting.
  - It retrains ML models continuously to fight drift.
  - Source: same post.
- **F16 [P]** DoorDash tests dispatch changes with **switchback experiments**, which randomize by region and time window to handle network effects. Each week, regions and time periods are split into an exploratory group and a confirmation group, and the confirmation group runs **no more than 3** experiments. (same post)
- **F17 [P]** DeepRed was DoorDash's **first microservice, extracted from the Django monolith in 2014**. It was written in **Scala** because "Python was not a good fit for CPU-intensive tasks". (DoorDash, 2020-12-02, https://doordash.engineering/2020/12/02/how-doordash-transitioned-from-a-monolith-to-microservices/)
- **F18 [S]** DoorDash rebuilt dispatch in 2026 as a next-generation real-time platform:
  - The old engine tightly coupled business logic, state and heavy maths. At peak, demand was **nearly 4x dispatch processing capacity**, and the algorithms scaled quadratically.
  - The new design has a decoupled **coordinator plus distributed compute**.
  - It delivered **10x throughput**, **99.9% of runs finishing in under 20 s**, and **more than 99.99% availability on Super Bowl Sunday**, despite a 10% week-over-week rise in peak volume.
  - Source: DoorDash, Oct 2026, https://careersatdoordash.com/blog/how-doordash-built-a-next-generation-real-time-dispatch-platform/ (403 to fetch, and not yet archived. Figures come from the search summary.)
- **F19 [P]** Uber Eats dispatch "launched with a **greedy** matching algorithm". It later moved to **global matching**. In the article's example, greedy gave 6 minutes of total travel and global gave 4 minutes. Global matching "couldn't live without accurate time predictions". (Zi Wang, Uber, InfoQ, 2019-11-20, https://www.infoq.com/articles/uber-eats-time-predictions)
- **F20 [P]** Uber dispatch (DISCO, "dispatch optimization") was generalised into a **Supply service** (vehicle capabilities and state) and a **Demand service** (rider or delivery requirements). It can plan ahead, for example matching a driver who is about to finish a trip. (High Scalability, 2015-09-14, URL in F6)
- **F21 [P]** Grab models matching as **minimum-weight bipartite matching (Kuhn-Munkres, O(m³))**. It batches bookings over a window of W seconds and recomputes "every X seconds".
  - The **cost matrix (Dijkstra on a road graph of about 400k nodes in Singapore) dominates runtime for m < 1,500**. Assignment time only overtakes it above m ≈ 2,500.
  - Their **Incremental KM** exploits the "spatial locality of matching" (the best passenger is usually one of the nearest). It computes only the costs it needs, cuts runtime by more than an order of magnitude, and returns the same optimum.
  - Source: Grab, 2021-11-22, https://engineering.grab.com/using-real-world-patterns-to-improve-matching
- **F22 [P]** Grab "doesn't always assign the nearest driver".
  - The process: candidates within an allocation radius (by geohash), then narrowed by ETA, then fraud checks (for example, the driver and passenger phone are the same person).
  - It weighs **more than 40 factors**, including driver preferences, a home-destination preference, and Jakarta's odd-even licence-plate rule.
  - For food, it assigns larger vehicles to larger or longer orders.
  - Source: Grab, 2023-08-14, https://www.grab.com/inside-grab/stories/matching-driver-partners-and-riders-allocation/
- **F23 [P]** Gojek moved away from a **broadcast** model in which every nearby driver saw every order:
  - With 100 orders and 200 drivers in an area, each driver saw each order. The result was unfair "fastest phone wins" races, no accountability, and high concurrency load.
  - Gojek replaced it with 1:1 allocation in a 2016 rewrite in **Clojure**, after an earlier rewrite in Go.
  - At the time Gojek handled **300,000+ orders per day** (late 2015), later "35+ orders per second", with drivers on average within **300 m**.
  - Source: Gojek, 2018-07-02, https://blog.gojek.io/how-gojek-manages-1-million-drivers-with-12-engineers-part-2-2/ (archived)
- **F24 [P]** Gojek's Allocations team supported **1,000,000+ registered drivers with 12 engineers**. (Gojek, 2018-03-24, https://blog.gojek.io/how-gojek-manages-1-million-drivers-with-12-engineers-part-1/)
- **F25 [3rd]** Deliveroo's "Frank" dispatch engine "constantly calculates and recalculates" the best rider-to-order combination. It uses ML predictions of travel and prep time, and can "change its mind" when delays happen. Deliveroo claimed about **20% faster deliveries in the UK (2017)**. The rider help page says Frank aims for the rider to arrive "as close as possible to the order collection time". (Silicon Republic, 2017-07, https://siliconrepublic.com/machines/deliveroo-algorithm-frank-machine-learning ; https://rider.deliveroo.co.uk/support/orders/how-and-why-am-i-offered-specific-orders)

**How to use this in an interview answer:**
- Avoid greedy nearest-driver matching. Collect orders and riders over a short window (seconds), build a cost matrix from predicted ETA, prep time and acceptance probability, and solve the assignment per region. Use Hungarian or KM for 1:1, or MIP if batching is involved.
- Mention that building the cost matrix (the road-network ETA calls) dominates the runtime, so prune candidates with an H3 kRing first.
- Shard dispatch by region or city so each instance solves a bounded problem. Bigger regions give better optima but cost more compute.
- Avoid broadcasting orders to every rider (Gojek's lesson). Offer each order to one rider at a time, with a timeout and re-offer on decline.
- To roll out a new matcher, shadow-compare it against the old one (DoorDash matched more than 99%) and use switchback tests.

---

## 3. Batching (multiple orders per rider)

- **F26 [P]** Uber Eats "triple batching":
  - One courier can carry **up to 3 orders** from **1 or 2 restaurants**.
  - The trade-off is the risk of never finding a courier against customer wait time. Inputs are distances, prep times, predicted delivery times and courier availability.
  - Batching is more likely at lunch and dinner peaks, and is limited when delivery times would be extensive.
  - Source: Uber, 2023-02-27, https://www.uber.com/en-AU/blog/triple-batching-for-restaurants
- **F27 [P]** DoorDash batching:
  - It is most effective for the same merchant (fewer pickups) or nearby merchants (fewer parking stops).
  - Prep time "can afford us extra time to pick up another order along the way".
  - The solver compares batched and single offers in one scoring function, which lets it trade efficiency against speed.
  - Source: DoorDash 2021-08-17 (URL in F14)
- **F28 [P]** The MIP formulation is what made **multi-delivery routes** possible. The Hungarian algorithm could only do 1:1 matching. (DoorDash 2020-02-28, URL in F10)
- **F29 [P]** Grab batches when supply is short:
  - It weighs item type, weight and prep time, then sequences pickups and drop-offs. The "Saver" delivery tier depends on batching.
  - Separately, Grab's simulator frames each decision as "dispatch a nearby driver now vs. hold briefly for a possible second order from the same merchant", within a short decision window.
  - Sources: Grab 2023-08-14 (URL in F22); https://engineering.grab.com/powering-ai-led-research-through-simulation [S]

**How to use this in an interview answer:**
- Present batching as a variant of the same solver: candidate generation adds pairs of orders at the same or nearby restaurants.
- Score each batch with a penalty for extra delay to the first customer and for variance (DoorDash's cascading-variability penalty). Cap a batch at 2 or 3 orders.
- Enable batching more aggressively at peak, when riders are scarce, and gate it on a "no promise broken" constraint.

---

## 4. ETA and prep-time prediction

- **F30 [P]** Uber **DeepETA**:
  - The routing engine sums segment travel times. An ML model then predicts the **residual** ("ETA post-processing").
  - The team tried 7 architectures and chose an encoder-decoder with **linear-transformer** self-attention, because a full transformer was too slow.
  - Latency must be "a few milliseconds at most". It is Uber's highest-QPS model.
  - Each prediction touches only about 0.25% of the model's parameters.
  - Loss is an **asymmetric Huber loss**, which penalises under- and over-prediction differently.
  - Features include origin and destination on **multi-resolution geo grids with feature hashing**, time, real-time traffic, and request type (for example, delivery drop-off).
  - Source: Uber, 2022-02-10, https://www.uber.com/blog/deepeta-how-uber-predicts-arrival-times/
- **F31 [P]** Uber Eats makes several time predictions: **food prep time, dispatch time, ETD and travel time**.
  - In the old method, a fixed estimate (25 minutes prep plus 5 minutes travel) produced bad dispatch times.
  - In the new method, prep time comes from **XGBoost GBDT** with hyperparameters tuned by Bayesian optimization on Michelangelo.
  - Features:
    - historical: average prep time over the past week
    - near-real-time: average prep time over the last 10 minutes
    - real-time: time of day, order size
    - menu-word embeddings, so a beef stew and a salad get different prep times
  - Source: InfoQ / Uber, 2019-11-20 (URL in F19)
- **F32 [P]** Uber Eats has **no ground truth for "food ready"**, because restaurants don't report it. Uber infers labels from each completed order and retrains on them. A **conditional random field** on courier phone sensor signals predicts the courier's state (parking, walking to pickup and so on). Context in the article: Uber Eats was in more than 600 cities with more than 220,000 restaurant partners. (same)
- **F33 [P]** DoorDash prep time is a **censored-data** problem. When a Dasher arrives after the food is ready, the true ready time is unobserved.
  - The team tried Cox proportional hazards, XGBoost AFT and WTTE-RNN.
  - The winner was a simple **two-stage data adjustment** followed by standard gradient boosting. It was validated with a switchback test.
  - Source: DoorDash, 2020-10-14, https://doordash.engineering/2020/10/14/solving-for-unobserved-data-in-a-regression-model/
- **F34 [P]** DoorDash models each leg separately: store travel, **parking at the merchant and at the consumer**, pickup logistics, and return to the car. Some legs use tree models on the Sibyl platform. Others use simple averages, such as average parking time at a store over the past month. (DoorDash 2021-08-17, URL in F14)
- **F35 [P]** Deliveroo's Frank uses ML predictions of rider travel time and food prep time from historical data. (2017 coverage, URL in F25) [3rd]

**How to use this in an interview answer:**
- Split ETA into prep time + rider-to-restaurant + wait + restaurant-to-customer, and model each part separately.
- Use a routing engine for the base travel time, then add an ML residual correction (DeepETA).
- For prep time, combine merchant history (7-day average), real-time load (10-minute average), order size and cuisine. Acknowledge that the labels are censored or missing.
- Use an asymmetric loss, because a late ETA hurts more than an early one.
- Dispatch riders so they arrive at the predicted ready time, not at order time.

---

## 5. Real-time tracking and push

- **F36 [P]** Uber drivers sent a **location update every 4 seconds**. The geospatial index was designed for "**a million writes per second**", with reads far exceeding writes. (High Scalability, 2015-09-14, URL in F6)
- **F37 [P]** Uber **Ringpop**:
  - It is a Node.js library combining **SWIM gossip membership**, a consistent-hash ring (FarmHash, red-black tree) and **"handle or forward"** request routing over TChannel.
  - Its first use was **Geospatial**, the live driver-location layer of matching. That replaced a design that held every vehicle in memory and scanned all cars per request.
  - When a node fails, its keys move to the next node on the ring, with no full rebalance.
  - It is **AP** (favours availability) and scaled to thousands of nodes.
  - Sources: Uber, 2016-02-04, https://www.uber.com/blog/ringpop-open-source-nodejs-library/ ; High Scalability 2015 (URL in F6)
- **F38 [P]** Uber's failover trick for in-flight trip state: Dispatch periodically sends an **encrypted state digest to the driver's phone**. After a datacenter failover, the phone's next location update restores the trip state, so the backup datacenter doesn't need replicated trip data. (High Scalability, 2015-09-14)
- **F39 [P]** Uber **RAMEN** push platform:
  - Why it exists: at one point **80% of API-gateway requests were polling**.
  - Protocol: first SSE, later gRPC streams. Messages carry **sequence numbers**, and on reconnect the client sends the highest sequence number it saw, so the server replays the rest.
  - Timing: a **4 s heartbeat**, with reconnect after 7 s of silence and an ack every 30 s.
  - Delivery: 3 priority levels, a TTL from seconds up to 30 minutes, and at-least-once delivery.
  - Scale: **more than 1.5M concurrent connections, more than 250k messages/s, 99.99% server-side reliability**.
  - Source: Uber, 2020-12-18, https://www.uber.com/blog/real-time-push-platform/
- **F40 [P]** Grab Pharos expects location updates around **once per second** and handles them in memory. Reads use **snapshots of the ART root**, so concurrent writes never block a nearby query. (Grab 2020-12-22, URL in F8)
- **F41 [S]** Lyft map-matches noisy phone GPS onto the road network with a **marginalized particle filter**. This was more accurate than its earlier HMM approach, especially downtown. (https://eng.lyft.com/a-new-real-time-map-matching-algorithm-at-lyft-da593ab7b006)

**How to use this in an interview answer:**
- The rider app sends GPS every 3 to 5 seconds (Uber used 4 s) to a location service. Keep only the latest position, in memory or Redis, sharded by H3 cell or rider ID through consistent hashing. Stream the full history to Kafka for analytics.
- Push updates to customers over SSE, WebSocket or gRPC streams rather than polling (Uber found 80% of its traffic was polling).
- Make the stream resumable with sequence numbers and heartbeats.
- Do the back-of-envelope sum out loud: 1M active riders at one update every 4 s is about 250k writes/s.

---

## 6. Storage

- **F42 [P]** Uber **Schemaless** (2014 to 2016) was built because Uber was running out of Postgres capacity for trip data and expected failure by the end of 2014.
  - It is an append-only sparse 3-D hash map on **sharded MySQL**.
  - Cells are **immutable JSON** addressed by (row UUID, column name, ref key). The ref key is used for versioning.
  - It adds triggers (CDC-style) and eventually consistent secondary indexes, with lag typically **under 20 ms**.
  - Trips are split into columns (BASE, STATUS, NOTES, FARE ADJUSTMENTS). Writing a BASE cell triggers billing.
  - Source: Uber, 2016-01-12, https://www.uber.com/blog/schemaless-part-one-mysql-datastore/
- **F43 [P]** Uber **Docstore** is the evolution of Schemaless:
  - Partitions of **3 to 5 nodes run Raft** across zones, on a MySQL engine.
  - It offers **strict serializability per partition** and is explicitly "**CP**: favors consistency over availability".
  - It adds schemas, materialized views and ACID transactions.
  - Why Uber moved on: Schemaless's append-only API was restrictive, and Cassandra's eventual consistency complicated application code.
  - Source: Uber, 2021-02-23, https://www.uber.com/blog/schemaless-sql-database/
- **F44 [P]** DoorDash's original stack was a **single Django app with one PostgreSQL primary**. It hit the limits of vertical scaling and read replicas, needed **PgBouncer** because monolith replicas exhausted DB connections, and moved some domains to **Cassandra**. (DoorDash 2020-12-02, URL in F17)
- **F45 [P]** DoorDash's inventory platform (grocery, tens of thousands of items per store) runs on **CockroachDB**:
  - Batching upserts at **1,000 rows per SQL statement** and rewriting a JDBI batch into a single multi-row UPSERT cut **per-item processing time by 75%, storage QPS by 99%, and DB CPU by 50%**.
  - Item details flow through Kafka, then Flink, then Snowflake for analytics.
  - Source: DoorDash, 2023-02-22, https://doordash.engineering/2023/02/22/how-doordash-designed-a-successful-write-heavy-scalable-and-reliable-inventory-platform/
- **F46 [P]** DoorDash moved part of its ML feature store from Redis to **CockroachDB**:
  - It reached about **2M rows/s inserted on 63 m6i.8xlarge nodes**, at about 30% CPU.
  - Cost per stored value fell **75%**, but more than 50% of features still stayed on Redis.
  - Source: DoorDash, 2023-03-21, https://doordash.engineering/2023/03/21/using-cockroachdb-to-reduce-feature-store-costs-by-75/
- **F47 [P]** Deliveroo moved its **Dispatcher** from RDS Postgres to **DynamoDB**:
  - Why: some tables had more than 1B rows, schema and index changes were painful, and autovacuum needed constant tuning.
  - Single-table design with type-prefixed keys: `o#` order, `r#` rider, `p#` pickup, `d#` delivery.
  - A GSI keyed by **sub-area** has the ready-for-pickup time as its sort key.
  - Sub-area partition keys were unbalanced, so they added **random-suffix write sharding** with a shard count configurable per area.
  - GSIs are kept sparse, and DynamoDB Streams feed a Lambda that publishes to MSK (Kafka), with a version attribute that preserves ordering.
  - Source: AWS Database Blog, 2023-05-18, https://aws.amazon.com/blogs/database/how-deliveroo-migrated-their-dispatcher-service-to-amazon-dynamodb/
- **F48 [P]** Instacart serves real-time availability scores from a DB, joined in SQL at retrieval time, because RPC was too slow for search-time filtering.
  - Scores arrive on two paths: a full sync from Snowflake several times a day, and a **lazy refresh** when a stale item appears in search, aggregated through Kinesis.
  - Lazy refresh cut ingestion load by **two-thirds**.
  - Scale is "hundreds of millions" of items across more than 80,000 stores.
  - Source: Instacart, 2023-09-06, https://company.instacart.com/tech-innovation/instacarts-item-availability-architecture-solving-for-scale-and-consistency
- **F49 [P]** Instacart treats availability as a **probability (0 to 1)** learned from shopper "found / not found" scans. **Dynamic thresholds** per retailer, region or category decide what is shown as available, and can change without retraining. Customers with a perfect found rate are **2x as likely to reorder**. (Instacart, 2023-07-10, https://company.instacart.com/tech-innovation/how-instacarts-item-availability-evolved-over-the-pandemic)
- **F50 [P]** Uber **M3** metrics platform: **6.6B time series, about 500M metrics/s aggregated, 20M metrics/s persisted**. It was built after Graphite (manual resharding, no replication) and Cassandra (compaction overhead) failed at scale. (Uber, 2018-08-07, https://www.uber.com/blog/m3/)

**How to use this in an interview answer:**
- Use polyglot storage:
  - orders and payments: a sharded relational or strongly consistent store (Docstore, CockroachDB), because order state needs transactions and must be CP
  - menus, catalog and search: Elasticsearch plus a cache
  - live rider location: in-memory or Redis with TTL (AP is fine)
  - availability or ML scores: precomputed and joined at query time
- Mention hot partitions (Deliveroo's sub-area keys) and fix them with sharded keys.
- Batch writes: 1,000 rows per statement cut DoorDash's DB QPS by 99%.

---

## 7. Workflow orchestration and async processing

- **F51 [P]** Uber **Cadence** (open-sourced 2017) is a workflow orchestration engine.
  - It provides its own persistence, queues and **durable timers**.
  - Architecture: a front end, a **history service** that owns shards, a **matching service** that dispatches tasks, and a **Cassandra** backend.
  - Workers can be killed and restarted without losing workflow state.
  - Uber Eats used it to coordinate the **8 standard steps of an Eats order workflow**.
  - Source: Uber, 2019-11-12, https://www.uber.com/blog/open-source-orchestration-tool-cadence-overview/
- **F52 [P]** DoorDash Drive used **Cadence as a fallback** behind its primary delivery-creation flow (order created, then a Kafka topic, then a Drive consumer).
  - The main challenge was that **not all downstream calls were idempotent**, and duplicate deliveries per order would be very bad.
  - Options considered: add idempotency keys, use child activities around the non-idempotent parts, or **save a context of completed steps and skip them on retry**.
  - Source: DoorDash, 2020-08-14, https://doordash.engineering/2020/08/14/workflows-cadence-event-driven-processing/
- **F53 [P]** DoorDash's inventory pipeline runs asynchronously in **Cadence** workflows, with automatic retries (maximum attempts set) and worker **heartbeats**. The client sends a whole store's inventory in one call, the server saves it to **S3**, and a Cadence job processes it asynchronously. (DoorDash 2023-02-22, URL in F45)
- **F54 [P]** DoorDash replaced **Celery + RabbitMQ** with Kafka. The old setup powered **more than 900 async task types** (checkout, order transmission to merchants, Dasher location processing).
  - RabbitMQ went down at peak load. HA failover took **more than 20 minutes** and lost messages.
  - Celery countdowns (delayed tasks) overloaded the broker.
  - Kafka brought **head-of-line blocking** on slow messages. The fix was a non-blocking worker: one Kafka-consumer process feeds a bounded **local queue** read by many task-executor processes.
  - The outages stopped almost immediately after rollout.
  - Source: DoorDash, 2020-09-03, https://doordash.engineering/2020/09/03/eliminating-task-processing-outages-with-kafka/
- **F55 [P]** Uber Kafka scale and the Consumer Proxy:
  - Kafka processes "**trillions of messages and multiple petabytes per day**". Pub-sub queuing grew from **1M to 12M messages/s** over 5 years.
  - Problems with the standard consumer: head-of-line blocking and poison pills. A message taking about 10 minutes blocks its whole partition, and at 100 ms per message a partition does only 10 messages/s.
  - The proxy pushes messages to gRPC consumers and uses **out-of-order acks**: it commits an offset only once every earlier message is acked or nacked.
  - A **DLQ** handles poison pills, and parallelism is no longer capped by partition count.
  - Source: Uber, 2021-08-31, https://www.uber.com/blog/kafka-async-queuing-with-consumer-proxy/
- **F56 [P]** Uber's "Reliable Reprocessing" pattern:
  - Failed messages go to a chain of **retry topics with increasing delays**, then to a **DLQ**. The original offset is committed so the partition is not blocked.
  - Transient errors are retried. Code bugs (for example, a null pointer exception) go straight to the DLQ, which can be merged back into the first retry topic.
  - Source: Uber, 2018-02-16, https://www.uber.com/blog/reliable-reprocessing/

**How to use this in an interview answer:**
- Model the order lifecycle (placed → paid → restaurant accepted → rider assigned → picked up → delivered) as a durable workflow (Cadence or Temporal) or as an event-driven state machine on Kafka.
- Every step must be idempotent, using an idempotency key per order and per step.
- Use retry topics with backoff plus a DLQ instead of blocking a partition.
- Mention head-of-line blocking and how to fix it with per-key ordering and a parallel worker pool.
- Avoid in-memory brokers with delayed tasks at peak (DoorDash's RabbitMQ outages).

---

## 8. Peak handling and resilience

- **F57 [P]** DoorDash failure modes:
  - **Retry storms**: payment-service latency led Dasher-app retries to amplify load.
  - **Death spirals**: pods fail readiness, and the survivors overload.
  - **Metastable failures**.
  - A misconfigured **circuit breaker** cut traffic between unrelated services, which caused a wide outage.
  - Source: DoorDash, 2023-03-14, https://doordash.engineering/2023/03/14/failure-mitigation-for-microservices-an-intro-to-aperture/
- **F58 [P]** DoorDash's **load shedding** uses Netflix's **adaptive concurrency limit**, which acts as a gRPC interceptor and lowers the in-flight limit as latency rises. It can **prioritise requests by header**, keeping only high-priority calls during overload.
  - Limitation: shedding is local only and hard to tune. One service started with too high a limit and overloaded at startup.
  - The proposed fix is a centralised controller (Aperture) that watches SLOs and applies load shedding or distributed rate limits across services.
  - Source: same.
- **F59 [P]** DoorDash on circuit breakers: "50% error rate" is a common rule of thumb but does not fit every service. Autoscaling helps only if it is **predictive, not reactive**, and adding capacity to one service often just moves the bottleneck. (same)
- **F60 [P]** A **more than 2-hour site-wide outage in mid-2021** pushed DoorDash to adopt an **Envoy service mesh**.
  - The first features were adaptive concurrency, outlier detection and metrics. The payment and notification services were onboarded first, rolled out by canary over 2 weeks.
  - The mesh now covers **more than 1,000 microservices** and **more than 80M requests/s at peak**.
  - Source: DoorDash, 2025-06-18, https://careersatdoordash.com/blog/inside-doordashs-service-mesh-journey-part-1-migration-at-scale/ (archived)
- **F61 [P]** Grab **"Spampede"** is a circuit breaker for localised demand spikes (rain at rush hour, concert endings):
  - It counts **unallocated bookings per (service, geohash bucket at reduced precision, time bucket)** with Redis `INCR` plus `EXPIRE`.
  - When the count passes a configured threshold, new bookings from that area are rejected **before any downstream calls**.
  - This stops one hotspot from slowing the app everywhere. Grab accepted lower accuracy in exchange for CPU efficiency.
  - Source: Grab, 2020-01-08, https://engineering.grab.com/preventing-app-performance-degradation-due-to-sudden-ride-demand-spikes
- **F62 [P]** Grab large campaigns:
  - It processes users in **batches of about 100**, and batched DB queries cut API latency by **up to 85%**.
  - The **Kafka partition count must be at least the number of consumer servers**.
  - Separate topics per action type and per country stop small, urgent campaigns from waiting behind large ones.
  - Source: Grab, 2022-04-01, https://engineering.grab.com/supporting-large-campaigns-at-scale
- **F63 [S]** DoorDash's new dispatch platform held **more than 99.99% availability on Super Bowl Sunday** with a 10% week-over-week rise in peak volume (see F18).

**How to use this in an interview answer:**
- Lunch and dinner peaks, rain and cricket finals are Zomato's equivalent of the Super Bowl. Plan for them:
  - pre-scale predictively, not reactively
  - shed load by priority: protect checkout and payment, degrade recommendations and ratings first
  - use circuit breakers with retry budgets and jittered backoff
  - add a Spampede-style per-H3-cell breaker when riders run out in one area
- Mention retry storms and death spirals by name, to show you understand failure modes.

---

## 9. Monolith to microservices migration

- **F64 [P]** DoorDash's monolith history:
  - The monolith was a **Django app** from 2013. It showed strain by 2017, with brittle code, slow test suites, costly rollbacks and Kubernetes capacity limits.
  - 2018 brought major reliability issues and a code freeze.
  - In **late 2019, DoorDash halted all new feature work** to re-architect.
  - Source: DoorDash 2020-12-02 (URL in F17)
- **F65 [P]** DoorDash's migration phases:
  - Prehistory (2014 to 2019): "satellite" services still depended on the monolith's DB, so a monolith failure took them down too.
  - **Project Reach (2019)**: standardised on **Kotlin + gRPC**, and began moving some data from Postgres to **Cassandra**.
  - **Project Metronome (Q4 2019)**: one representative per team on extraction.
  - **Project Phoenix**: mapped every workflow and dependency, with tiered milestones.
  - Source: same.
- **F66 [P]** DoorDash's final layering: **Frontend, then BFF (backend-for-frontend), then Backend** (cart, feed, delivery services), **then Platform** (identity, communication), **then Infrastructure**. The stated reasons for the move: site stability, isolation between business lines, development agility, scaling the organisation, and allowing different tech stacks. (same)
- **F67 [P]** A cost DoorDash names: cross-domain calls that were in-process function calls became network calls, bringing latency, retries, circuit breaking, load shedding and API compatibility concerns. (same) The migration took **2019 to 2023** and was what motivated the service mesh. (DoorDash 2025-06-18, URL in F60)
- **F68 [P]** Gojek kept its legacy code ("Stan Marsh") running and replaced it piece by piece, a "Ship of Theseus" approach. About 10% of it survived in 2018. The Clojure rewrite went live, and "after 3 days … no one noticed". (Gojek 2018-07-02, URL in F23)

**How to use this in an interview answer:**
- Start with a modular monolith. Split along domain boundaries (order, menu or catalog, dispatch, payments, notifications) when team size or scaling forces it.
- Extract the CPU-heavy, independently scaling piece first. For DoorDash that was DeepRed, the dispatch service.
- Give each service its own database. Shared-DB "satellites" do not isolate failures.
- Put a BFF per client (customer, restaurant, rider apps).
- Budget for the network-call tax with timeouts, retries and breakers, ideally in a mesh.

---

## 10. Rate limiting

- **F69 [P]** Uber's **Global Rate Limiter (GRL)**:
  - Why: the old Redis-backed per-service limiters were inconsistent and added latency, and a central counter "can't scale to hundreds of millions of RPS" (it would need hundreds of Redis clusters).
  - Three tiers: mesh-proxy clients enforce locally, zone **aggregators** sum usage, and regional or global **controllers** push **drop ratios** every second.
  - It replaced token buckets with probabilistic **drop-by-ratio**: drop = (actual − limit) / actual, so 1.5x the limit drops about 33%.
  - It **fails open** if the control plane is down, and enforcement lags real traffic by 2 to 3 seconds.
  - Scale: **about 80M RPS across more than 1,100 services**. Removing the Redis hop cut P99.5 latency by up to 90%.
  - Source: Uber, 2026-02-12, https://www.uber.com/blog/ubers-rate-limiting-system/
- **F70 [P]** Lyft **Ratelimit** (open-sourced 2017) is a Go/gRPC service that **Envoy** calls:
  - Requests carry a **domain + hierarchical descriptors** (for example, per IP, or per DB connection), which form a **Redis** key.
  - Limits are set per second, minute, hour or day, which is a fixed window.
  - It handled "tens of thousands of rate limit requests per second" at Lyft.
  - Source: Lyft Engineering, 2017-02-08, https://eng.lyft.com/announcing-ratelimit-c2e8f3182555 (archived)

**How to use this in an interview answer:**
- Use two layers:
  - an edge or API-gateway limiter (Envoy plus a Redis fixed or sliding window, keyed by user, IP or API key) to stop abuse, for example OTP spam or coupon scraping
  - service-to-service overload protection (adaptive concurrency or load shedding)
- At very large scale, avoid a central Redis counter on every request. Aggregate locally and push down drop ratios (Uber GRL).
- Choose fail-open or fail-closed per endpoint: fail open for browsing, fail closed for OTP and login.

---

### Gaps and caveats
- **Not verified:**
  - Deliveroo's specific Frank algorithm. There is no engineering post; only press coverage and the 20% claim.
  - Uber Eats' own original "food delivery time prediction" post. I used the InfoQ article written by Uber's Zi Wang instead.
  - The DoorDash 2026 dispatch numbers, which are search-summary only.
  - A DoorDash notification-platform post and an explicit DoorDash idempotency-key post. I found none; the idempotency facts come from the Cadence post, F52.
  - Gojek's surge and geo internals.
- **Not used:** Instacart's "200M items every 30 min" figure appears only in third-party summaries, because the Medium post was blocked.
- **Dated material:** the High Scalability figures (4 s updates, S2 level 12, 1M writes/s) are from a 2015 talk. Present them as historical.
