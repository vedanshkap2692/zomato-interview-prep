# Zomato / Eternal backend: sourced research notes

Compiled 23 Oct 2026, for a Zomato SDE-1 system design round.

**How to read this file**
- **[P]** marks a primary source: Zomato or Eternal's own blog, or an AWS/vendor post co-written with Zomato engineers.
- **[V]** marks a vendor case study, such as WarpStream, MongoDB or an AWS partner site. The numbers are self-reported.
- **[S]** marks a secondary source: press, or a third-party write-up. Treat these as unverified.
- **[D]** marks a figure I derived from sourced numbers. It is not stated in any source.

The Zomato tech blog moved from blog.zomato.com to zomato.com/blog, and newer posts are at eternal.com/blog. I read every [P] post below in full through its live URL.

---

## 0. Headline scale numbers (memorise these)

| # | Fact | Number | Source date | Source |
|---|------|--------|-------------|--------|
| 0.1 | Peak orders per minute, NYE 2023 | **8,400 OPM** | 29 Feb 2024 | [P] https://www.zomato.com/blog/a-tale-of-scale-behind-the-scenes-at-zomato-tech-for-nye-2023 |
| 0.2 | Peak request rate, NYE 2023 | **80.92 million requests/min** | 29 Feb 2024 | [P] same |
| 0.3 | Milestone order count, NYE 2023 | **3 million orders** | 29 Feb 2024 | [P] same |
| 0.4 | Microservices, NYE 2023 | **300+** microservices | 29 Feb 2024 | [P] same |
| 0.5 | Compute on NYE 2023 | **11,000 EC2 instances**, about **40,000 ECS tasks**, **5.5 PB** of storage provisioned | 29 Feb 2024 | [P] same |
| 0.6 | Goyal's peak-minute figure for NYE 2023 | **8,422 orders at 8:06 pm**, about **140 orders/sec** | 1 Jan 2024 | [S] https://www.theweek.in/news/biz-tech/2024/01/01/zomato-received-140-orders-every-second-on-new-years-eve-swiggy-says-broke-all-records.html |
| 0.7 | NYE 2020 and NYE 2021 | 2020: **over 4,000 OPM**. 2021: "almost double that" per minute, and **over 2.5 million orders** | 29 Jan 2022 | [P] https://www.zomato.com/blog/our-learnings-from-nye/ |
| 0.8 | NYE 2020 peak, as reported by press | **4,254 OPM** per Goyal's tweet. Business Today reported **4,100 OPM** at 8:22 pm, so the two figures conflict | 1 Jan 2021 | [S] https://www.businesstoday.in/latest/trends/story/zomato-receives-record-4100-orders-per-minute-on-new-years-eve-283215-2021-01-01 |
| 0.9 | NYE 2025, Zomato and Blinkit combined | **Over 75 lakh (7.5M) orders** in one day, delivered by **over 4.5 lakh delivery partners** to **over 63 lakh customers** (Goyal) | 1 Jan 2026 | [S] https://ianslive.in/record-deliveries-on-new-years-eve-despite-strike-calls-deepinder-goyal--20260101183238 |
| 0.10 | Food delivery, FY24 averages | Monthly averages: **18M** transacting customers, **247k** active restaurant partners, **400k** active delivery partners. **63M** annual transacting customers, ordering **11.9** times a year on average | May 2024 | [P] Investor overview, https://b.zmtcdn.com/investor-relations/Zomato_overview_May-24.pdf |
| 0.11 | FY24 annual food orders | About **750M orders/yr**, roughly **2M/day** (63M × 11.9) | — | [D] from 0.10 |
| 0.12 | FY25 annual food orders | **853M** orders | 2025 | [S] blog summaries of the FY25 annual report, not checked against the report itself |
| 0.13 | FY24 GOV | Food delivery **INR 32,224 cr**. B2C consolidated **INR 47,918 cr**. Food delivery footprint **800+ cities** | May 2024 | [P] Investor overview |
| 0.14 | Q2FY27, Eternal (latest quarter) | B2C NOV **INR 35,644 cr**, up 54% YoY. Food delivery NOV **INR 11,445 cr**, up 21.5%. Blinkit NOV **INR 20,940 cr**, up 79.3%, from **2,751 stores**. District NOV **INR 3,259 cr**. Food delivery MTC **28.0M**. Blinkit MTC **37M**. Hyperpure serves about **85,000 restaurants** | 21 Oct 2026 | [P] https://www.eternal.com/blog/q2fy27 (the shareholder letter PDF is linked from that page) |
| 0.15 | Blinkit store plan | Over **3,500 stores by Mar 2027**, and about **5,000 by Mar 2028** | 21 Oct 2026 | [P] Q2FY27 letter |
| 0.16 | Blinkit SLA metrics | Average delivery time **12.5 min**. **75%+** of orders arrive within 2 minutes of the promised time. Fill rate **99%+** (Mar 2024) | May 2024 | [P] Investor overview |
| 0.17 | FY26 Eternal-wide | **109M** Indians transacted over **$10B** across Blinkit, District and Zomato | 28 Apr 2026 | [P] https://www.eternal.com/blog/q4fy26 |

**How to use in interview:** When you size the system, say that Zomato handles about 2M+ food orders a day on average and about 8.4k orders/min at the NYE 2023 peak, and that it scaled to about 80M requests/min across 300+ microservices. The design target is therefore peak QPS, which is about 4x the average. Average OPM is about 1.4k (2M orders over 1,440 minutes).

---

## 1. Overall architecture, languages and service mesh

1. **Languages: Go, with PHP legacy.** The old PDF generator was "powered by a PHP-based library". Zomato rewrote it in **Go**, citing "performance and our team's familiarity with it". The new service, Espresso, uses headless Chromium through the Go library Rod, signs PDFs in **under 200 ms**, handles **120K RPM** at peak, and signed **1.6M PDFs in 19 minutes** (about 1,400/s). Each task has 4 vCPU and 4 GB and runs on **AWS ECS**. It is open source at github.com/zomato/espresso. — 17 Mar 2025, [P] https://www.zomato.com/blog/espresso-brewing-pdfs-at-zomato-faster-than-you-can-say-cappuccino
2. **Monolith plus microservices coexist.** The logging post refers to "our internal microservices and monolithic applications". — 20 Jul 2023, [P] https://www.zomato.com/blog/building-a-cost-effective-logging-platform-using-clickhouse-for-petabyte-scale
3. **The ML API Gateway is written in Golang.** It works as a DAG workflow engine and fetches features from the Feature Store. — 18 Aug 2021, [P] https://www.zomato.com/blog/elements-of-scalable-machine-learning/
4. **The log ingestion workers are Go**, and they run on spot instances (see §9). — 20 Jul 2023, [P]
5. **Service mesh: Kuma (Envoy-based).** Zomato's chaos testing injects upstream latency "via our internal service mesh layer built using Kuma". — 29 Feb 2024, [P] NYE 2023 post
6. **Kuma scale: 500+ microservices with 10,000 dataplanes.** The talk also covered a plan for zone-aware routing to cut cross-AZ data transfer costs, and header-based routing to local services. Talk: "Unlocking the Power of Kuma Service Mesh", Arpit Mishra (Zomato), **KubeCon + CloudNativeCon India 2025**. — 2025, [P/S] talk abstract at https://sessionize.com/arpit-mishra/ (I read the abstract only, not the talk itself.)
7. **Local Preview developer tooling.** A Kuma traffic-route policy plus a unique HTTP header sends staging traffic to a developer's laptop. Zomato's in-house CLI, **zctl**, opens an SSH reverse tunnel to a "Local Proxy" (SSH daemon plus Envoy) running in staging. There is also an ephemeral "Preview Environment" that runs in parallel with staging. The post mentions "hundreds of microservices". — 9 Jun 2025, [P] https://www.zomato.com/blog/embracing-shift-left-testing-with-local-preview/
8. **Compute is mostly ECS on EC2 for services.** NYE 2023 ran about 40,000 ECS tasks on 11,000 EC2 instances. **ML serving runs on Kubernetes with spot instances**, and models are tuned for CPU inference. — [P] NYE 2023 post (29 Feb 2024) and ML post (18 Aug 2021)
9. **AWS footprint.** About **90% of business-critical workloads run on AWS** (2022). — 27 Apr 2022, [V/S] https://smeoncloud.in/zomato-handles-up-to-2-5x-more-transactions-after-migrating-non-relational-database-to-aws/
10. **No public PHP-to-Go migration post exists.** I searched and found no Zomato post that describes a monolith decomposition or a language migration. The PHP and Go facts above come only from the Espresso post. Do not claim a specific migration story.

**How to use in interview:** Say that Zomato runs hundreds of Go microservices on AWS ECS behind a Kuma/Envoy service mesh. Point out that the mesh lets them inject faults for chaos tests and route traffic by header for dev previews. Mention that ML serving is separate, on Kubernetes with spot capacity.

---

## 2. Databases

11. **Primary databases are DynamoDB, Aurora RDS (MySQL) and MongoDB.** NYE 2023 peaks were:
    - DynamoDB: **12.3M RCUs and 5.2M WCUs per minute**
    - RDS: **610k selects/s and 16,000 write transactions/s**
    - MongoDB: **470k read ops/s** scanning **3.3M docs/s**, and **180k write ops/s** writing **1.75M docs/s**

    — 29 Feb 2024, [P] NYE 2023 post
12. **The billing platform moved from TiDB to DynamoDB.**
    - The TiDB cluster had grown from 5 to **25 nodes**, and was over-provisioned and manually scaled for peak days.
    - Results after the move: throughput rose from **2,000 to 8,000 RPM** (4x), average response time fell **about 90%** to around **75 ms**, and monthly cost fell **about 50%**.
    - The platform handles about **10M events/day** and about **1M payments/week**, and its services talk to each other over Kafka.
    - The migration had zero downtime, using a phased rollout with **dual writes**, replication, sync and failover.
    - Authors: Neha Gupta and Kanica Mandhania (Zomato), with Vishal Gupta (AWS). — 22 Dec 2023, [P] https://aws.amazon.com/blogs/database/unlocking-performance-scalability-and-cost-efficiency-of-zomatos-billing-platform-by-switching-from-tidb-to-dynamodb/ (mirrored at zomato.com/blog/switching-from-tidb-to-dynamodb)
13. **The billing DynamoDB model is a single table using the adjacency-list pattern.** Tables such as `payout_details`, `billing_ledger` and `payout_order_mapping` were merged into one. The partition key is **merchant ID + payout cycle + business vertical**, chosen for high cardinality. A GSI uses an inverted index with keys suffixed `_<division-number>` to avoid hot partitions. — 22 Dec 2023, [P] same
14. **Profile Store (the ML feature store) moved to DynamoDB, Oct–Dec 2020.**
    - The previous database was an unnamed NoSQL store with no batch support. Scaling it for a peak took about 2 weeks of analysis plus up to 2 days of provisioning.
    - After the move it was benchmarked at **17.5M RPM** and served **12M RPM on 31 Dec 2020**. It absorbs **2.5x** traffic spikes and batches **500 requests per call**.
    - Weekly cost fell **25%**, and uptime was near 100%. — 27 Apr 2022, [V/S] smeoncloud.in (an AWS-partner customer story)
15. **Data platform stores.** Aurora MySQL changes are captured by **Debezium** and published to **Kafka** in Avro. **Flink** writes them as ORC into **Iceberg** tables on S3. DynamoDB Streams also land in S3, and Spark converts them to ORC. Analytics run on **Trino** (over 250K queries scanning 2 PB) and **Druid** (over 20B events ingested and 8M queries a week). Aurora MySQL and EMR had already moved to Graviton. — 25 Apr 2023, [P] https://aws.amazon.com/blogs/opensource/how-zomato-boosted-performance-25-and-cut-compute-cost-30-migrating-trino-and-druid-workloads-to-aws-graviton/
16. **MongoDB Atlas since 2017.** Zomato moved all self-hosted Mongo clusters to Atlas in 2017. For Nugget, its AI customer-support platform, Atlas beat DynamoDB and DocumentDB in Zomato's evaluation. Nugget handles **15M+ conversations a month** with an **85% resolution rate** (Jan 2026), and cut support cost from **$20M to $9M a year**. — 24 Mar 2026, [V] https://www.mongodb.com/company/blog/innovation/zomato-ai-solution-nugget-reduces-support-costs
17. **ScyllaDB and Cassandra.** I found no evidence that Zomato uses either. Do not claim it.

**How to use in interview:** For orders and payments, use a relational store (Aurora MySQL). For high-throughput key-value access such as billing ledgers and feature or profile stores, use DynamoDB with a composite partition key. Mention that Zomato moved billing off TiDB to DynamoDB and got 4x throughput at half the cost. Use MongoDB for flexible-schema data such as support conversations.

---

## 3. Caching

18. **The cache is ElastiCache (Redis).** On NYE 2023 the fleet held **35 TB in memory** and pushed **20 Gbps**. At peak it served **1 billion GETs per minute** and about **100M SETs per minute**. — 29 Feb 2024, [P] NYE 2023 post
19. **The online feature store runs on Redis Cluster.** Real-time features are computed from Kafka by Flink and stored in Redis Cluster. Batch features are computed with Spark and kept in DynamoDB, with hot keys cached in Redis. In preparation for NYE 2021 it was load-tested to about **18M requests/min**, a 3x improvement over NYE 2020. — 18 Aug 2021, [P] ML post
20. **Cache-miss chaos tests.** Zomato injects cache misses on purpose "to evaluate database reliability under stress". Before NYE, it also caches frequently used queries to take load off the database. — 29 Feb 2024, [P]

**How to use in interview:** The read-to-write ratio is about 10:1 (1B GETs vs 100M SETs per minute). Say that you would front the menu and restaurant reads with Redis, and that you would test what happens when the cache fails, because Zomato injects cache misses before NYE.

---

## 4. Kafka and event streaming

21. **Kafka is "the backbone of our event-driven architecture".** On NYE 2023 it handled **450M+ messages per minute**, with a combined cluster peak of **5.6 GBps**. — 29 Feb 2024, [P]
22. **Kafka runs on three tiers by workload.**
    - Logging and observability moved from self-hosted Kafka, kept in a single AZ with replication factor 1 to avoid inter-AZ fees, to **WarpStream** (diskless Kafka on S3). That cut cost by **60%**. Log throughput ranges from **2.3 GiB/s at the daily low to 25 GiB/s at the peak**.
    - The benchmarking traffic-replay pipeline moved from managed Kafka to WarpStream, saving **80%**. It averages **52M messages/min** during a backfill.
    - **Latency-sensitive topics run on Confluent Cloud.**

    — 13 Aug 2026, [V] https://www.warpstream.com/blog/eternal-case-study
23. **Ads feedback pipeline on Flink.**
    - The legacy job ran Flink 1.8 in Java and held over **150 GB** of state for deduplicating user IDs across clicks and impressions. State loss reset the campaign counters, which caused ads to over-deliver and billing to be wrong.
    - The fix moved to Flink SQL that emits **incremental counts** instead of daily totals, plus an hourly **reconciliation job** that acts as a fail-safe.
    - The dedup state TTL dropped from **24 h to 2 h**. State size fell **99%**, infrastructure cost fell by more than **$3,000/month**, and downtime was eliminated.

    — 22 Apr 2025, [P] https://www.zomato.com/blog/eliminating-bottlenecks-in-real-time-data-streaming-a-zomato-ads-flink-journey/
24. **Talk title only:** "Scaling Kafka at Blinkit & Zomato", Confluent DSWT 2026, on YouTube (https://www.youtube.com/watch?v=kjGP_kBQdVc). [S] I did not watch it.

**How to use in interview:** For counters that drive billing, publish increments rather than running totals, and add a periodic reconciliation job. That way a lost stream state cannot corrupt the totals. This is a strong talking point for any idempotency or exactly-once question.

---

## 5. Real-time tracking, rider location and MQTT

25. **The move from HTTP polling to MQTT.** Zomato started with periodic HTTP polling. It stopped scaling: TLS and header overhead often outweighed the tiny payloads, and rider markers jumped instead of moving smoothly. Zomato moved to **MQTT** (publish/subscribe over persistent connections) for live location and delivery lifecycle events. These events feed live tracking, ETA, notifications and the merchant view.
    - The Android library **Pulse-MQTT** is built on Eclipse Paho. It runs operations through a command queue and handles auto-reconnect and re-subscribe, exponential retry and health checks through WorkManager or AlarmManager.
    - It runs in the delivery partner, consumer and Hyperpure apps, and is open source (zomato.github.io/pulse-droid).

    — 23 Jun 2026, [P] https://www.eternal.com/blog/pulse/
26. **MQTT was part of NYE 2021 load testing.** A team asked, "Can MQTT handle the load?" Zomato tested at **10x the expected NYE load** by replaying amplified, randomized production traffic against a production clone. — 29 Jan 2022, [P] NYE 2021 post
27. **Zomato Trace.** Zomato acquired Sparse Labs in Sep 2016. Its Android app streamed rider location to the restaurant and the user in real time, and it allocated orders by proximity, using ML to judge a rider's familiarity with a neighbourhood. It was renamed **Zomato Trace**. — 25 Sep 2016, [P] https://www.zomato.com/blog/welcoming-sparse-labs-to-zomato/
28. **iOS Live Activities.** The iOS app uses **PushToStartToken** (iOS 17.2+) so the server can start a lock-screen Live Activity over APNs even when the app is terminated. — 13 Mar 2026, [P] https://www.eternal.com/blog/live-activities-push-to-start-dynamic-images
29. **Not found:** I found no primary source on how rider pings are stored server-side (Redis GEO, geohash and so on). Third-party write-ups that claim Redis plus WebSockets are [S] and unverified.

**How to use in interview:** For tracking, propose MQTT or another persistent connection rather than polling, and cite Zomato's own reasons: per-request overhead and jumpy markers. Mention client-side reconnect and re-subscribe as a reliability concern.

---

## 6. Rider assignment and dispatch

30. **Active Dispatch.** A Deep Q-Network multi-agent reinforcement learning model recommends where idle delivery partners should move, based on predicted demand and available supply. It beat both "no recommendation" and rule-based baselines: partners who followed it got a higher share of orders and shorter gaps until their next order. — 18 Aug 2021, [P] ML post
31. **ETA feeds assignment.** A fast, equation-based ETA built from partner speed and "medal" features **raised the compliance of the assignment system's predictions by 4%**. — 25 Mar 2022, [P] https://www.zomato.com/blog/the-accurate-eta-to-customer-satisfaction-part-two/
32. **Demand throttling at peak.** On NYE 2021, Zomato throttled demand where delivery partners or kitchen capacity were running out. It used an automated demand-management system, plus manual control at times. — 29 Jan 2022, [P]
33. **Order clubbing.** Zomato was experimenting with POI-based order clubbing and multi-drop deliveries. — 6 Sep 2021, [P] https://www.zomato.com/blog/to-help-us-locate-you-better/
34. **Not found:** I found no primary source for the batching or assignment algorithm itself (cost function, matching method). Secondary "batching needs spatial and temporal closeness" claims are [S].

**How to use in interview:** Describe assignment as a matching problem with ETA as an input. Mention supply repositioning (Active Dispatch) and demand throttling as the levers Zomato actually uses at peak.

---

## 7. ETA and food preparation time (FPT/KPT)

35. **A Time Microservice owns ETA.** It computes the ETA and distributes it to other services. ETA is used in menu browsing, restaurant selection, assignment, tracking and delay communication.
    - Total ETA is made of delivery partner travel time (DP-ETA), kitchen prep time (KPT), real-time dynamic buffers (demand, traffic, weather, supply, system stress) and the partner's arrival time at the restaurant.
    - Zomato moved from a map-graph-based model to a **tree-based model**, because Indian map data is sparse.
    - Requirements were millisecond latency on CPU, at "thousands of QPS".

    — 16 Mar 2022, [P] https://www.zomato.com/blog/the-accurate-eta-to-customer-satisfaction-part-one/
36. **The DP-ETA model.**
    - **LightGBM** with a **Tweedie** loss (log-link, L1 metric), with **per-city models**, tuned with Optuna's LightGBMTuner.
    - The prediction is split into two parts: pickup to the drop-zone geofence, and the geofence to handover. Features include polar coordinates centred on the restaurant.
    - Serving moved from **MLflow to FastAPI**: 3x throughput per pod and 2x better latency (MLflow took about **7–8 ms** per request vs about **2–3 ms** for FastAPI).
    - Serving millions of calls cost **under $10/day**. Models are stored in an S3 artifact store.
    - Zomato does not show the ETA to delivery partners, to avoid rash driving.

    — 25 Mar 2022, [P] ETA Part 2
37. **The FPT model, v1.**
    - The ground-truth label was originally "restaurant accepted" to "partner picked up". Zomato added a **Food Order Ready (FOR) button** to the restaurant app to get true FPT, which gave a **9% improvement in 5-minute accuracy**.
    - Dishes are represented with **Word2Vec embeddings**, because there are **about 3.5M distinct dishes**. Restaurants are represented with entity embeddings, covering **150k+ restaurants** that receive orders each month.
    - A **stacked LSTM** reads the last 5 running and last 5 completed orders. It cut MAE from **4.64 to 4.13 min**.

    — 4 Feb 2020, [P] https://www.zomato.com/blog/food-preparation-time/
38. **The FPT model, v2.** Embeddings were about 95% of the model's parameters. Training them separately made the model **one-tenth the size** and faster in production. Zomato also built per-city models and an asymmetric loss to avoid underestimation. Results: **+4% 3-min accuracy and −9% 3-min breaches**. — 22 Jun 2022, [P] https://www.zomato.com/blog/predicting-fpt-optimally/
39. **Road detection.** Zomato reconstructs missing roads from delivery partner GPS pings and trip data, using sensor fusion and GPS noise filtering, to densify open-source maps. — 18 Aug 2021, [P] ML post

**How to use in interview:** Break ETA into prep time + pickup + travel + handover, and give each part its own model. Say that Zomato fixed its label problem with a "food ready" button, and that it serves the model through a separate Time service with a latency budget of a few milliseconds.

---

## 8. Search, discovery, recommendations and menus

40. **Search engine: Solr/Lucene, not Elasticsearch.**
    - Search handles **100M+ queries/day**. "Solr-Lucene is the base of all search functions."
    - Some clusters run master–slave: the master indexes, and slaves sync and serve reads behind a load balancer.
    - An unbounded FieldCache caused OOMs. Switching to **DocValues** (column-oriented, memory-mapped) gave **10x throughput per slave** and cut cost **about 80% (about ₹30L/month)**. Zomato upgraded from Solr 6.x to 7.6.0 and then 8.7.0, moved some clusters to SolrCloud, and forked Lucene for fixes.

    — 31 Mar 2023, [P] https://www.zomato.com/blog/explained-how-zomato-handles-100-million-daily-search-queries-p1/
41. **Ranking and autocorrect.**
    - **Learn-to-Rank** is a pairwise, gradient-boosted model whose features include regional dish popularity, historical orders and keyword match.
    - **Autocorrect** uses a phonetic-weighted edit distance plus a Siamese network.
    - Auto-suggest has delivery and dining tabs, and removes outlets outside the delivery radius.

    — 31 May 2022, [P] https://www.zomato.com/blog/rethinking-zomato-search-pt-1
42. **Natural-language and voice search.** Query understanding uses a **BiLSTM-CRF** tagger over Word2Vec and BPE embeddings to tag dish, restaurant and intent in Hinglish queries such as "Burger 150rs wala". It runs fast on CPU and was A/B tested with a higher CTR. — 25 Aug 2022, [P] https://www.zomato.com/blog/how-we-make-our-search-more-conversational-and-inclusive/
43. **Unified search scoring.** Relevance is text score + popularity score + intent score, where the intent score comes from location, time and day. If one result crosses a threshold, it becomes the "leader"; this happened in about 65% of queries. Search uses server-driven UI (the **Sushi** design system) and tracks every tap with `search_id` and `previous_search_id`. — 2 Dec 2019, [P] https://www.zomato.com/blog/all-new-search-and-discovery-on-zomato/
44. **Homepage recommendations.**
    - Ranking uses LightGBM **LambdaMART (LGBMRanker)** to optimize the order-through rate.
    - A **contextual multi-armed bandit** with Bayesian regression re-ranks results for GMV. Per-customer weights are updated daily. It raised **GMV per app open by ₹3 and AOV by ₹6**.

    — 18 Aug 2021, [P] ML post
45. **Menu digitization.** Menu images go through text detection, OCR, section detection and a dish classifier, so no human input is needed. This powers dish search and speeds up menu creation. — 18 Aug 2021, [P] ML post
46. **Ads ranking.** Restaurant ads are ranked as a Learn-to-Rank problem (pointwise, pairwise and listwise approaches were compared), with MRR and mean ordered rank as metrics. — 16 Aug 2022, [P] https://www.zomato.com/blog/powering-restaurant-ads-on-zomato/

**How to use in interview:** Retrieval runs on an inverted index (Zomato uses Solr) filtered by delivery radius. Ranking uses LTR features (popularity, distance, ETA, personalization). Mention DocValues and column storage for the sort and facet fields, which gave Zomato 10x throughput.

---

## 9. Observability, logging and cost optimisation

47. **LogStore (ClickHouse) replaced self-hosted Elasticsearch/ELK.**
    - Volume: up to **150M logs/min** and **over 50 TB/day** uncompressed.
    - Cluster: 10 × M6g.16xlarge nodes serving about 12,000 queries/day.
    - Ingestion: custom Go workers on spot instances batch **20k messages per table**, with **at most 5 s lag**. The native protocol is about 1.8x faster than HTTP.
    - Storage: there is no replication or ZooKeeper; durability comes from S3 and EBS backups. Data moves from hot to cold after 24 h and is deleted after 3 months.
    - Indexing: a `tokenbf_v1` bloom-filter index serves LIKE queries.
    - Results: P99 query time is 10 s. Zomato expects to save **more than $1M a year**.

    — 20 Jul 2023, [P] https://www.zomato.com/blog/building-a-cost-effective-logging-platform-using-clickhouse-for-petabyte-scale
48. **Observability volume on NYE 2023.**
    - Metrics: **2.2B active time series**, with a peak of **17.8M/s**.
    - Logs: LogStore took **60 TB** in a day, with an **80M RPM** ingestion peak.
    - Chat support: **19,500 concurrent sessions**, and **51,400 messages/min**.

    — 29 Feb 2024, [P]
49. **Graviton migration.** Moving Trino and Druid to Graviton (R6g/R6gd) cut **EC2 cost by 30%** and improved performance by **up to 25%**. Druid p99 improved 25%, Trino p99 about 33%, and peak capacity fell 25% for Druid and 20% for Trino. — 25 Apr 2023, [P] AWS Open Source blog
50. **Other cost levers.** Spot instances for ML on Kubernetes and for the log workers. The planned Kuma zone-aware routing to cut cross-AZ traffic. WarpStream, which removes inter-AZ replication cost. — [P]/[V] sources above

**How to use in interview:** Logs are append-only and need only time-range plus filter queries. A columnar store with batched inserts (ClickHouse) beats Elasticsearch on cost at Zomato's 50 TB/day.

---

## 10. Peak handling (NYE): load testing, chaos and kill switches

51. **NYE 2023 preparation.**
    - Zomato ran service design reviews, optimized slow queries, cached hot queries, reviewed past incidents, filled monitoring gaps, and tuned alerts.
    - It reviewed **kill switches and degraded modes**, so that lower-tier dependencies cannot take down critical services.

    — 29 Feb 2024, [P]
52. **Chaos tests.** Zomato injected upstream latency through Kuma, flipped kill switches to test degraded mode, and injected cache misses. — 29 Feb 2024, [P]
53. **In-house Benchmarking Platform (built by SRE).**
    - Safety rules: read-only or safe-write traffic only, isolated from production load, load ramped gradually so autoscaling can react, and automatic abort on critical alerts or rising upstream errors.
    - Results: **14,000 minutes** of benchmarking across 100+ services, a peak of **50M RPM**, **30B requests** fired, **400 TB** of stored requests, and no service outage.

    — 29 Feb 2024, [P]
54. **Pre-scaling and war rooms.** The SRE team pre-scaled the infrastructure ahead of NYE, and war rooms were staffed across the businesses. — 29 Feb 2024, [P]
55. **NYE 2021 learnings.**
    - SRE spent **over 15 days** load-testing every component above the expected load, with infrastructure pre-scaled.
    - Teams tuned circuit breakers, added kill switches for each feature, and found the bottleneck of each database.
    - When the third-party calling provider failed, Zomato switched to an alternate provider.
    - Payments routed traffic around downtime at individual payment gateways.

    — 29 Jan 2022, [P] https://www.zomato.com/blog/our-learnings-from-nye/
56. **Traffic replay through Kafka.** Backend services record live requests into Kafka and replay them later for load tests. This is the benchmarking pipeline that moved to WarpStream. — 13 Aug 2026, [V]

**How to use in interview:** This is your best Zomato-specific story. Pre-scale ahead of the peak, because autoscaling lags a spike. Benchmark each service with recorded production traffic. Put a kill switch and a degraded mode on every non-critical dependency. Use multiple providers for third-party dependencies (SMS and calling, payment gateways).

---

## 11. Payments

57. **Billing platform:** see fact 12 (10M events/day, 1M payments/week, Kafka plus DynamoDB).
58. **Payment gateway failover at peak.** On NYE, Zomato routes traffic away from gateways, providers or networks that are down, across multiple payment methods. — 29 Jan 2022, [P]
59. **Licences and UPI.**
    - Zomato Payments received an RBI online payment aggregator licence effective **24 Jan 2024**.
    - It voluntarily surrendered the licence about four months later (reported 14 May 2024), saying a payments business was not commercially viable at that stage.
    - Zomato UPI was launched with ICICI Bank in 2023.

    — [S] https://www.businesstoday.in/latest/corporate/story/zomato-payments-gets-rbis-nod-to-operate-as-online-payment-aggregator-414863-2024-01-25 and https://www.outlookbusiness.com/corporate/zomato-payment-rbi-payment-aggregator-license
60. **Not found:** I found no engineering post on Zomato's wallet or payment internals.

**How to use in interview:** For payments, describe multi-gateway routing with health-based failover, plus an async, idempotent ledger fed by Kafka events (the billing platform pattern).

---

## 12. Location and addresses

61. **POI mapping.** Zomato tokenized customer addresses, ranked the tokens TF-IDF-style, density-clustered them per city, and produced GeoJSON building footprints that pass through a moderation queue and an A/B test. The result was fewer partner-to-customer calls and shorter handshake times. Device GPS accuracy is **60–150 m** on mid to high-end phones, and can be off by **2–3 km**. — 6 Sep 2021, [P] https://www.zomato.com/blog/to-help-us-locate-you-better/
62. **Weather Union.** A network of **650+** on-ground weather stations in **45 cities**, launched **8 May 2024** with a free API. Zomato uses it for weather-aware delivery operations. — May 2024, [S] https://entrackr.com/2024/05/zomato-launches-real-time-hyperlocal-weather-info-network

---

## 13. Notifications and rate limiting

63. **Not found:** I found no primary Zomato post on the notification platform or on internal rate limiting. The only first-party numbers are the public developer API limit of **1,000 calls/day** (https://www.zomato.com/policies/api-policy/) and LogStore's per-user and system-level query throttling (fact 47). Third-party notification "teardowns" are [S].

**How to use in interview:** Present rate limiting as generic design: a token bucket at the gateway or mesh. Do not attribute a specific design to Zomato.

---

## 14. Blinkit, District, Hyperpure

64. **Blinkit system shape** (from the Q2FY27 letter): quick commerce means "real-time inventory checks against a specific dark store, picker assignment, dispatch, all within a 10-minute delivery window", which is "sequences of tightly coupled, real-time handshakes". — 21 Oct 2026, [P]
65. **Blinkit engineering blog** (blinkit.com/blog and lambda.blinkit.com; both blocked my fetches, so the content below comes from search snippets, [S]):
    - **ARS Lite** auto-raises stock transfer orders between warehouses.
    - An Inventory Management System decides whether an item is enabled or disabled in the CMS.
    - A Debezium-based CDC "Source Replication System" feeds the data platform.
    - Replenishment runs several times a day, with on-demand triggers when availability runs low.
66. **District.** Launched in Nov 2024. Its ticketing base came from Paytm's movies, events and ticketing business, bought for ₹2,048 cr. — [S] The press coverage I found dates this deal to Aug 2023. From memory, the deal was announced in Aug 2024, so check the date before you quote it. **I found no engineering post on District's seat-locking or queueing.** One secondary source mentions a "queue-based booking for high-demand events" with no technical detail.
67. **Hyperpure** serves about 85,000 restaurants and delivers within 2 hours in all major cities. 25% of its orders already arrive within that window. — 21 Oct 2026, [P] Q2FY27 letter
68. **AI evaluation (Gavel).** An Eternal-wide eval platform has assessed **over 1M items** (images, chatbot transcripts, voicebot calls) in six months. — 9 Jul 2026, [P] https://www.zomato.com/blog/building-gavel-ai-evaluation-infrastructure-across-eternal

---

## 15. Outages and postmortems

69. **Not found:** Zomato has published no public outage postmortem that I could find. Status-tracker sites are unreliable. The closest material is the NYE learnings above: the third-party call provider outage and failover (2021), and payment gateway downtime routing.

---

## Gaps (searched and not found; do not fabricate)
- A PHP monolith to Go microservices migration story.
- Rider location storage (Redis GEO or geohash) and the WebSocket vs MQTT server-side broker.
- The dispatch or batching algorithm itself.
- Notification fan-out architecture, and internal rate limiting.
- Any Zomato use of ScyllaDB, Cassandra, Couchbase or Elasticsearch for search. Zomato used Elasticsearch only for logs, before ClickHouse.
- A Zomato talk at AWS re:Invent.
- District ticketing internals.
- Outage postmortems.
