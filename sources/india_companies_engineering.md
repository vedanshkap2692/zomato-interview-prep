# How Indian food and quick-commerce companies build backends: sourced facts

Compiled 2026-10-23 for Zomato SDE-1 system design prep.

## How to read this file

- **Verification levels:**
  - **[V-full]**: I fetched and read the full page (AWS and MongoDB posts).
  - **[V-snip]**: The fact comes from search-engine extracts of the original page. Medium-hosted blogs (bytes.swiggy.com, lambda.blinkit.com, medium.com/dunzo) returned HTTP 403 when fetched, so I could not open them directly. The fact and number came from the original page's text, but I did not read the full article.
  - **[Vendor]**: A vendor case study. The numbers are real claims, but they are marketing material.
  - **[Press]**: A news report.
  - **[UNVERIFIED]**: A third-party blog or a claim I could not trace to a primary source. Do not quote these as fact in an interview.
- **Dates:** Medium posts do not show dates in the extracts I could see. Where a date is marked "~", I estimated it from the search index's "N days old" figure against today (2026-10-23). Treat those dates as approximate.
- **Swiggy posts are mostly 2019 to 2022.** Say "Swiggy wrote in their engineering blog a few years ago..." rather than claiming this is how they work today.

---

## 1. Delivery partner assignment, dispatch and batching

1. **Batching definition and value.** Swiggy describes batching as a restaurant getting two orders at about the same time from customers who live near each other. Batching "could double the capacity of the entire delivery fleet". Optimal batching is NP-hard. [V-snip]
   - Source: "The Swiggy Delivery Challenge (Part One)", ~2019. https://bytes.swiggy.com/the-swiggy-delivery-challenge-part-one-6a2abb4f82f6
2. **Batching window trade-off.** Orders can be batched when they arrive close together and the customers are close together. A wider wait window gives more chances to batch, but the first order waits longer to be picked up. [V-snip]
   - Source: same URL as #1.
3. **Assignment engine.** Swiggy's assignment is an integer linear program. It minimises total delivery cost and maximises customer experience. It runs over all active batches and delivery partners in a city, at short, regular intervals (micro-batch, not per-order). [V-snip]
   - Source: "The Swiggy Delivery Challenge (Part Two)". https://bytes.swiggy.com/the-swiggy-delivery-challenge-part-two-f095930816e3
4. **Two timing tactics.**
   - Just-In-Time assignment: assign the rider so they do not wait at the restaurant for food.
   - Next Order Assignment: consider a rider who is about to finish a delivery for the next order.
   - [V-snip] Source: Part One (#1). These are also described in the ICMR case study on Swiggy's 2016 scaling (business case, not engineering): https://icmrindia.org/casestudies/catalogue/Leadership%20and%20Entrepreneurship/LDEN148.htm
5. **Logistic zones.** City-wide optimisation runs were slow. Swiggy proposed restaurant clusters (zones) so the solver works on smaller sub-problems, and so one rider is unlikely to be considered for batches in two different clusters. [V-snip]
   - Source: "Logistic Zones for Assignment". https://bytes.swiggy.com/logistic-zones-for-assignment-48d9ce06c4a8
6. **ML scoring before assignment.** City-level batch jobs trigger at a configured interval. They score every (order x delivery executive) edge with an ML model, and an optimiser then picks the best assignment.
   - After moving to a batch-prediction API, P99 latency fell from 144 ms to 71 ms for a batch of 30.
   - The platform served more than 5K peak predictions per second for a single model.
   - The platform runs on Akka, with Redis as the feature store.
   - [V-snip, syndicated from Swiggy engineer Shrinath Agarwal's post] Sources:
     - https://bytes.swiggy.com/2x-improvement-in-latency-in-swiggy-data-science-platform-6101b7607530
     - https://akka.io/blog/2x-latency-improvement-in-swiggy-ml-and-ai-platform
   - [Vendor] The Akka customer story adds a "one-minute decision window" and about 50% less compute per prediction: https://akka.io/customer-stories/swiggy
7. **Batchability prediction (talk).** A Swiggy senior data scientist gave the talk "Predicting order batchability" at The Fifth Elephant Logistics Conf 2018. The talk listing has little technical detail. [V-snip]
   - Source: https://hasgeek.com/fifthelephant/logistics-conf-2018/sub/predicting-order-batchability-SLNkVpc9LpLWwKztqLUjBq
8. **Academic batching paper on Swiggy data.** "Batching and Matching for Food Delivery in Dynamic Road Networks" (arXiv 2008.12905, 2020) runs all its experiments on Swiggy datasets. [V-snip]
   - Source: https://arxiv.org/pdf/2008.12905
9. **Logistics monolith re-architecture.** The old "Delivery monolith" powered all critical delivery use cases, about a third of Swiggy's critical business logic. Its problems:
   - low test coverage, which meant long QA cycles and regressions;
   - MySQL and Redis clusters already scaled vertically to their limit, and the data was hard to partition;
   - it was built for food delivery, so it could not support new delivery types or multi-leg orders.
   - The new logistics platform powers deliveries for all Swiggy businesses. [V-snip]
   - Source: "Re-Architecting Swiggy's logistics systems", ~Jan 2022. https://bytes.swiggy.com/re-architecting-swiggys-logistics-systems-ddf301a29fa0

> **Interview use:** "Don't assign per order greedily. Collect orders for a short window (seconds to a minute), score order x rider edges with an ETA model, then solve a matching problem (Hungarian or ILP) per zone. Swiggy does city-level micro-batches split into logistic zones, plus just-in-time assignment so riders reach the restaurant as the food is ready."

---

## 2. ETA and prep-time prediction

10. **Prep time and ETA at cart.** A food-prep-time model runs at checkout. It uses:
    - restaurant busyness from recent orders;
    - order size;
    - the restaurant's past performance on the items ordered.
    - The assignment engine uses this estimate so the rider arrives about when the food is ready. [V-snip]
    - Source: "Predicting Food Delivery Time at Cart". https://bytes.swiggy.com/predicting-food-delivery-time-at-cart-cda23a84ba63
11. **Live ETA split into legs.** The ETA on the tracking screen starts from the checkout prediction and is refreshed at fixed intervals using live signals (restaurant stress, rider pings). It is modelled as four legs:
    - order-to-assignment;
    - first mile (rider to restaurant);
    - wait at the restaurant;
    - last mile.
    - [V-snip] Source: "How ML powers - When is my order coming? Part I". https://bytes.swiggy.com/how-ml-powers-when-is-my-order-coming-part-i-4ef24eae70da
12. **Last-mile speed from pings.** Last-mile features use rider location pings. Distance between pings is divided by the time between the first and last ping timestamps. Pings are grouped into time windows and outliers are removed first. [V-snip]
    - Source: Part II. https://bytes.swiggy.com/how-ml-powers-when-is-my-order-coming-part-ii-eae83575e3a9
13. **Guardrail on inputs.** Swiggy caps prep-time inputs at 60 minutes because the model behaves erratically on long outliers. [V-snip]
    - Source: "Building Rock-Solid ML Systems". https://bytes.swiggy.com/building-rock-solid-ml-systems-bb775f8a7126
14. **Incident: a bad model broke serviceability.** A new serviceability (PSLA, the predicted delivery time) model produced negative predictions after rollout. Swiggy rolled back to the old model and restored service within about 3 hours. The post-mortem noted there was no fallback for negative predictions. [V-snip, from the series listing]
    - Source: "#BehindTheBug - Cart Carnage: Unpacking the Effects of Negative PSLA Predictions on Serviceability". https://bytes.swiggy.com/behindthebug-unpacking-the-effects-of-negative-psla-predictions-on-serviceability-6e1712eceb56
15. **Kafka powers the predicted SLA.** Swiggy uses Confluent Kafka to power the predicted delivery SLA shown when a customer opens the app. [Vendor, case-study date not stated]
    - Source: https://www.confluent.io/customers/swiggy/

> **Interview use:** "ETA = sum of legs (assign + first mile + prep/wait + last mile), each with its own model. Predict once at cart and re-predict on events and pings. Always add sanity bounds and a fallback (Swiggy had an outage when a model returned negative ETAs)."

---

## 3. Geo-indexing, serviceability and delivery zones

16. **What "serviceable" means.** Swiggy calls an order serviceable when:
    - the customer and restaurant are "nearby" by road distance and travel time, not radial distance; and
    - there is enough delivery capacity around the restaurant.
    - [V-snip] Source: "A brief introduction to Engineering challenges at Swiggy". https://bytes.swiggy.com/engineering-challenges-at-swiggy-430dea6c86a3
17. **Serviceability scale.** Swiggy estimates about 100K app visits per minute at peak, times about 2,000 nearby pickup locations per customer, which gives about 200 million serviceability evaluations per minute. The target is P99 < 200 ms. [V-snip]
    - Source: "Designing the Serviceability Platform at Swiggy for High Scale - Part 1", ~Sep 2021. https://bytes.swiggy.com/designing-the-serviceability-platform-at-swiggy-for-high-scale-part-1-751a631f0379
18. **Geohash-keyed distance cache.**
    - Distances are cached with a geohash at some resolution as the key. Level-8 geohash gave the best accuracy trade-off.
    - Caching every level-8 origin-destination pair would need hundreds of billions of keys. (The idea of caching only dense pairs is the search summary's inference, not a stated fact.)
    - The cache has a high-water mark for eviction and a stricter stop-write threshold.
    - On a miss, the system falls back to a real-time A* search on OpenStreetMap, then to Haversine (straight-line) distance.
    - [V-snip] Source: same URL as #17.
19. **Distance Service.** Swiggy built a single Distance Service in front of the Google Distance Matrix API. Before it, each team called the API itself with a shared key. When usage limits were hit, Swiggy fell back to OpenStreetMap, which was not accurate enough. Central caching was the reason for building it. [V-snip via newsletter summary]
    - Source: https://biweekly-engineering.beehiiv.com/p/biweekly-engineering-blog-reading-list-issue-6
    - Swiggy also notes that about 2,000 to 2,500 restaurants or stores can be near a typical customer. That makes per-request routing expensive.
20. **Blinkit (Grofers) merchant discovery uses S2.** "Iris" (June 2016) uses Google's S2:
    - the earth is projected onto a cube, a quad-tree is built per face, and cells are enumerated along a Hilbert curve to get cell IDs.
    - Merchants are mapped to their cell plus its neighbours.
    - A cache keyed by cell ID holds the set of visible merchants, precomputed from delivery-app seed data. "No computation happens in real time."
    - [V-snip] Source: "Iris - Merchant Discovery at Grofers", Lambda by Blinkit. https://lambda.blinkit.com/iris-359b5dfd589b (mirror: https://blinkit.com/blog/iris-merchant-discovery-grofers/)
21. **Instamart dark-store assignment.** Instamart assigns the customer to the nearest dark store ("pod") based on location. Inventory is tracked per SKU per pod. [V-snip]
    - Source: "Adaptive Metric Alignment for Demand Forecasting in Swiggy Instamart". https://bytes.swiggy.com/adaptive-metric-alignment-for-demand-forecasting-in-instamart-9aa5d2fca308
22. **Serviceability under high demand (talk).** A Fifth Elephant 2018 talk by Swiggy data scientist Venkateshan K listed the demand shocks as:
    - IPL weekends;
    - rain;
    - New Year's Eve;
    - a competitor going down.
    - Its listed levers include demand forecasting, measuring delivery-system stress, batching and "supply side parameter control". [V-snip]
    - Source: https://hasgeek.com/fifthelephant/2018/sub/serviceability-under-high-demand-Pat71efxNFnLmGAqJNv2Fi
23. **Swiggy surge pricing (2016).** Swiggy introduced surge pricing on select holidays, festivals and heavy-rain days to attract delivery executives. [Press, May 2016]
    - Source: https://yourstory.com/2016/05/swiggy-surge-pricing/

> **Interview use:** "Index restaurants and riders by geohash, S2 or H3 cells. Query the cell plus its neighbours, then refine with road distance. Precompute serviceability per cell and cache it (Blinkit's Iris does S2 cell to merchants with no real-time compute). Fall back in steps for distance: cache, then routing engine, then Haversine. Serviceability = distance AND capacity, so shrink the radius or show 'high demand' when riders are scarce."

---

## 4. Real-time location tracking and the delivery partner app

24. **Delivery partner app architecture.**
    - The UI is React Native. Long-running, data-heavy work such as location tracking runs in native Android and iOS modules.
    - Sessions average 4 to 5 hours with the app in the background.
    - The app is event-driven, with a finite state machine per delivery flow.
    - For bad networks: lighter critical flows, retry flows, lazy sync and "business continuity modes".
    - For battery: dark mode and fewer animations.
    - [V-snip] Source: "Architecture and Design Principles Behind the Swiggy's Delivery Partners app". https://bytes.swiggy.com/architecture-and-design-principles-behind-the-swiggys-delivery-partners-app-4db1d87a048a
25. **Offline workflows.** Offline workflows were built for on-duty partners, and gRPC, MQTT and server push were being evaluated for reliable transport. [V-snip]
    - Source: "App-solutely No More Deli-worries!". https://bytes.swiggy.com/app-solutely-no-more-deli-worries-df3fa56bcaff
26. **[UNVERIFIED]** Zepto rider location "syncs every 3 seconds" (third-party marketing blog, no primary source): https://miracuves.com/blog/what-is-zepto-app-and-how-does-it-work/

> **Interview use:** "Rider app sends pings every few seconds. Batch and compress them, and buffer them offline. Ingest into Kafka, keep the latest location in Redis (GEOADD or a cell index) for matching, and push to the customer over WebSocket or SSE. Model each order as a state machine so a retried event cannot jump states."

---

## 5. Dark-store inventory, stock reservation and quick commerce

27. **Instamart inventory flow.** Units from the warehouse are inwarded into pods after a quality check. The inventory management system (IMS) is updated with available units and adjusted with each unit sold. Availability is defined as "% of store-open hours the SKU was in stock". Higher availability means more stock, which means more wastage. [V-snip]
    - Source: same URL as #21.
28. **Instamart picking time.** An average Instamart order is picked, packed and staged in under 4 minutes. The picker app directs staff to specific rack locations. Delivery averages under 10 minutes on a normal day and under 20 minutes on a busy one. [V-snip, Swiggy corporate blog, 25 Mar 2023]
    - Source: https://blog.swiggy.com/2023/03/25/pick-pack-and-deliver-how-women-are-playing-an-essential-role-in-ensuring-your-instamart-orders-reach-you-quickly/
29. **Zepto: hub-local inventory and search.** [V-full, AWS Big Data Blog, 17 Aug 2026, co-written by Zepto engineers]
    - Each of Zepto's hundreds of hubs has its own inventory, pricing and assortment.
    - Search is resolved against the customer's hub(s), not a global catalog.
    - Catalog, Inventory, Pricing and Offers services emit events to MSK (Kafka) topics of those names.
    - Source: https://aws.amazon.com/blogs/big-data/how-zepto-powers-sub-second-search-using-opensearch-service-or2-instances/
30. **Zepto: search index trusts the inventory service.** Zepto raised the OpenSearch refresh interval from 1 s to 10 s. That was acceptable because stock status is only "a tiebreaker" in ranking, and final availability is re-checked against the real-time inventory service after retrieval. [V-full] Source: same as #29.
31. **Zepto: orders moved to DynamoDB.** [V-full, AWS Database Blog, 4 Nov 2025, authors include Zepto principal engineers]
    - The old Aurora PostgreSQL `Customer_Orders` table had billions of rows. Row locks on status updates caused write contention, and autovacuum fell behind at peak.
    - Zepto built a Draft-Order service on DynamoDB. It accepts orders before payment and consumes payment events from MSK. On success it calls an **idempotent** OMS Create Order API.
    - A sparse GSI holds only orders still awaiting payment. A job runs every 5 minutes to confirm or terminate stuck orders. TTL deletes old drafts.
    - Results: Create Order about 60% faster on average and 40% faster at p99.
    - Rollout: 10%, then 30%, then 100%.
    - Lesson: `TransactWriteItems` averaged about 18 ms. Two sequential `PutItem` calls were each under 10 ms and halved write capacity use.
    - Source: https://aws.amazon.com/blogs/database/how-zepto-scales-to-millions-of-orders-per-day-using-amazon-dynamodb/
32. **Zepto: monolith to MongoDB.** [Vendor, MongoDB blog, 17 Dec 2024, from a MongoDB.local Bengaluru talk in Sep 2024]
    - Zepto moved from a monolith on PostgreSQL plus several Redis clusters to MongoDB Atlas.
    - Reasons: join-heavy queries burned CPU and served stale data, and the Redis cluster "could no longer scale".
    - Claimed results: 40% lower latency on critical APIs, 6x traffic capacity, 14% faster page loads. Dark-store packing takes under 75 seconds.
    - Source: https://www.mongodb.com/blog/post/commerce-scale-zepto-reduces-latency-by-40-percent-mongodb
33. **Blinkit replenishment.** Blinkit's availability team builds systems that decide what, when and how much to order. Orders go directly into vendors' ERPs via EDI, with multiple replenishments and on-demand triggers when availability runs low. [V-snip, ~2021]
    - Source: https://blinkit.com/blog/first-12-months-blinkit-manik-chawla/
34. **Swiggy Instamart checkout flow.** Swiggy's MCP developer docs describe the Instamart checkout. For UPI, place-order returns `PENDING_PAYMENT`, and a separate confirm step runs after payment is verified. The docs do not say whether stock is held in that window. [V-snip]
    - Source: https://mcp.swiggy.com/builders/docs/reference/instamart/
35. **Oversell prevention: no primary source found.** No Indian company has published how it prevents overselling (for example, reserve-on-cart vs decrement-on-order). The common pattern is "available and reserved counts per (sku, store) row, with an atomic conditional decrement and a TTL hold". That pattern appears in interview-prep blogs only (for example https://www.designgurus.io/answers/detail/what-to-expect-in-the-zepto-system-design-interview). **[UNVERIFIED as company practice]**

> **Interview use:** "Inventory is per (sku, dark_store), not global. Search can be eventually consistent (Zepto accepts 10 s index lag). Re-check stock against the inventory service at add-to-cart or checkout. Reserve with an atomic conditional decrement (`UPDATE ... SET avail = avail - n WHERE avail >= n`, or a Redis Lua script) plus a TTL hold released on payment failure. Treat 'draft order then pay then idempotent confirm' as a state machine with a sweeper job, as Zepto does."

---

## 6. Catalog and search

36. **Zepto search numbers.** [V-full, same post as #29]
    - The OpenSearch index is organised by city-product pair, with hub data as nested documents. A flat `active_hubs` array produced most of the latency gain.
    - Moving to OR2 instances (r7g to or2.12xlarge): about one-third fewer nodes, more than 2x indexing throughput, about 30% lower cost.
    - p90 search latency went from 187 ms to 89 ms; p99 went from 244 ms to 175 ms. Peak indexing lag went from about 12M to about 6M documents.
    - There are two Flink indexing jobs. P0 handles near-real-time updates (inventory, price). P1 handles bulk updates (tags, embeddings, offers).
    - Festive peaks previously needed 1.4x the data nodes.
    - Segment merges caused p99 spikes. The fixes were bigger EBS volumes (for more IOPS) and smaller `max_merged_segment` and `segments_per_tier`.
    - The migration took about 2 months with bucket-based traffic routing and zero downtime.
37. **Swiggy search evolution.**
    - Food search ran on Elasticsearch with fuzzy matching and geo filters. A query-embedding model ran at about 2 ms p99. [V-snip, ~2021] Source: "Find My Food: Semantic Embeddings...". https://bytes.swiggy.com/find-my-food-semantic-embeddings-for-food-search-using-siamese-networks-abb55be0b639
    - Later, small language models were used for relevance with a 100 ms embedding budget on CPU. [V-snip, ~2024] Source: https://bytes.swiggy.com/improving-search-relevance-in-hyperlocal-food-delivery-using-small-language-models-ecda2acc24e6
38. **Swiggy autocomplete.** Autocomplete runs learning-to-rank inside OpenSearch with a sub-10 ms ranking budget, because it runs on every keystroke. The model runs in OpenSearch to avoid extra network hops. [V-snip, ~Apr 2026]
    - Source: https://bytes.swiggy.com/real-time-ml-ranking-in-autocomplete-part-1-3cdbbd44f85a
39. **Swiggy latency budgets.** Search needs about 100 ms responses, while chatbots can take about 3 s. [V-snip, ~2024]
    - Source: https://bytes.swiggy.com/reflecting-on-a-year-of-generative-ai-at-swiggy-a-brief-review-of-achievements-learnings-and-13a9671dc624
    - A related figure: Swiggy's segmentation service on the page-load path has a p99 SLA under 50 ms. https://bytes.swiggy.com/segmentation-at-swiggy-part-1-d9566ab1a442

> **Interview use:** "Catalog writes go through Kafka into a search index (CDC or Flink). Split the indexing pipeline by priority: stock and price fast, enrichment slow. Filter by the user's serviceable store IDs in the query, and validate stock live after retrieval. Give each hop a latency budget (autocomplete < 10 ms ranking, search about 100 ms)."

---

## 7. Peak handling, load testing and capacity

40. **Zomato NYE 2023.**
    - Peak was 8,400 orders per minute and 80.92 million requests per minute.
    - Before the event, Zomato load-benchmarked to 50 million requests per minute across more than 100 services.
    - The EC2 fleet was pre-scaled to about 11,000 instances, and cross-team war rooms were set up.
    - [V-snip; fetch timed out] Source: "A Tale of Scale: Behind the Scenes at Zomato Tech for NYE 2023". https://www.zomato.com/blog/a-tale-of-scale-behind-the-scenes-at-zomato-tech-for-nye-2023
41. **Zomato NYE 2020.** Peak was 4,254 orders per minute, per Deepinder Goyal's tweet. [Press, Jan 2021]
    - Sources: https://www.freepressjournal.in/business/zomato-received-4100-orders-per-minute-on-new-years-eve ; Zomato blog "Our learnings from the biggest night of the year": https://www.zomato.com/blog/our-learnings-from-nye
42. **Zomato and Blinkit NYE 2025.** More than 75 lakh (7.5M) orders on NYE by more than 4.5 lakh delivery partners, per Deepinder Goyal; an all-time high. [Press, Jan 2026]
    - Source: https://entrepreneurloop.com/zomato-blinkit-new-year-orders-record-2026/
    - **[UNVERIFIED]**: "5,200 orders per minute" for that night.
43. **Swiggy dinner peak and IPL.** Far more orders come in 8 to 10 PM than 4 to 6 PM. Weekend dinners and IPL matches are named as peak periods. [V-snip]
    - Sources:
      - https://bytes.swiggy.com/designing-the-serviceability-platform-at-swiggy-for-high-scale-part-1-751a631f0379
      - https://bytes.swiggy.com/curbing-cod-loss-with-technology-ecf554c759a3
44. **Swiggy feed ranking.** Swiggy served feed-ranking models at peak loads of 300,000 QPS. [V-snip, as summarised by search]
    - Source: https://bytes.swiggy.com/enabling-data-science-at-scale-at-swiggy-the-dsp-story-208c2d85faf9
45. **Swiggy DynamoDB cost guard.** On-demand DynamoDB autoscaled without limit. Swiggy built a guard that flips a table to provisioned capacity once consumption crosses a threshold, open-sourced as `ddb-ondemand-protector`. [V-snip]
    - Source: https://bytes.swiggy.com/how-to-limit-autoscaling-in-on-demand-dynamodb-tables-c57e20cbbbcf
46. **Flipkart Big Billion Days.**
    - Aerospike reported 95M transactions per second during BBD (Nov 2024). A Nov 2025 account says about 90M QPS across 3 data centres and more than 50 use cases. [Vendor] Source: https://aerospike.com/blog/flipkart-journey-90-million-qps/
    - Traffic is 6 to 8x business-as-usual. [Vendor, Google Cloud] Source: https://cloud.google.com/blog/products/data-analytics/flipkarts-dx-journey-to-futureproof-its-platform
47. **Zepto order DB under peak.** Zepto says the DynamoDB order store scales "from quiet overnight periods to festival and sports-event peaks". [V-full] Source: same as #31.

> **Interview use:** "Know your peak (Zomato NYE: about 8.4K orders per minute, about 81M requests per minute). Load-test above it (Zomato benchmarked 50M requests per minute across more than 100 services), pre-scale the day before instead of trusting autoscaling, run a war room, and put caps on autoscaling and cost (Swiggy's DynamoDB protector)."

---

## 8. Kafka and event pipelines

48. **Swiggy messaging platform.** Swiggy's Kafka-based messaging platform does event streaming, microservice orchestration and choreography. [V-snip]
    - Source: https://bytes.swiggy.com/behindthebug-kafka-under-the-water-288c3d05b202
49. **Swiggy on Confluent.** Swiggy moved from self-managed Kafka to Confluent Cloud. Uses:
    - real-time predictions, recommendations and analytics;
    - async communication between services;
    - the predicted SLA.
    - It plans an asynchronous DR cluster to fail over to. Context: more than 3 billion orders across 680 cities as of 2024. [Vendor] Source: https://www.confluent.io/customers/swiggy/
50. **Rill.** Rill is Swiggy's Flink-based streaming platform. It reads Kafka and Delta tables and writes Kafka, Delta and Redis. Most Kafka topics keep data for at most 3 days, which limits backfills. [V-snip]
    - Source: https://bytes.swiggy.com/rill-state-implementation-937bf51e077f
51. **DSP feature pipeline.** Feature jobs write to Kafka, which streams to S3 (offline) and Redis (online feature store). [V-snip]
    - Source: https://bytes.swiggy.com/enabling-data-science-at-scale-at-swiggy-the-dsp-story-208c2d85faf9
52. **COD pipeline.** Kafka consumers process checkout and rider events.
    - The order ID is the idempotency key, giving exactly-once effect.
    - After 3 retries a message goes to a retry queue.
    - [V-snip] Source: "Curbing COD Loss With Technology", ~Jan 2022. https://bytes.swiggy.com/curbing-cod-loss-with-technology-ecf554c759a3
53. **Klaxon alerting.**
    - Klaxon is Swiggy's real-time business monitoring system. It publishes business events to Kafka and handles more than 1.5M alerts a day, delivered to Slack, Freshdesk and SMS. [Press summary of Swiggy blog]
    - It consolidated alert jobs from 30 to 8. [V-snip]
    - It reports a 70% cut in Kafka jobs and more than 50% infra savings after a Databricks-based redesign. [Press summary of Swiggy blog]
    - Sources:
      - https://bytes.swiggy.com/enabling-real-time-business-monitoring-with-klaxon-668fa14c5e38
      - https://www.cxodigitalpulse.com/swiggys-klaxon-fires-1-5-million-daily-alerts-to-power-real-time-operations/
54. **CDC at Blinkit and Dunzo.**
    - Grofers/Blinkit built a data lake with Debezium CDC and Apache Hudi. [V-snip] Source: https://lambda.blinkit.com/origins-of-data-lake-at-grofers-6c011f94b86c
    - Most Dunzo microservices use PostgreSQL. Postgres WAL is streamed with Debezium Server to consumers and the warehouse. [V-snip] Source: https://medium.com/dunzo/a-deep-dive-into-dunzo-data-platform-part-1-4cdb9778d655
    - Zepto syncs its S3 data lake via CDC on MSK and DynamoDB Streams into Flink. [V-full] Source: same as #31.

> **Interview use:** "Order and rider state changes are events on Kafka, partitioned by order_id for ordering. Consumers are idempotent on order_id. Retry a few times, then send to a retry or DLQ topic. Use CDC (Debezium) instead of dual writes to feed search, analytics and caches."

---

## 9. Notification platform

55. **Kabootar.** Kabootar is Swiggy's internal, business-agnostic communication platform.
    - It reaches customers, delivery executives, restaurants and support staff over push, SMS and email.
    - Each message carries a self-contained context object so any node can process it.
    - Messages pass through event, campaign, templating and router services.
    - Each message has a persisted state machine, so a message stuck in a non-final state can be reprocessed.
    - Scheduling uses Quartz, with retries, fallback-template sequences, and open and click tracking.
    - [V-snip, ~2024] Source: "Kabootar - Swiggy's Communication Platform". https://bytes.swiggy.com/kabootar-swiggys-communication-platform-e5a43cc25629
56. **Smart push timing.** Swiggy uses multi-armed bandits to choose push notification timing and content. [V-snip, title-level]
    - Source: https://bytes.swiggy.com/smart-push-notifications-multi-armed-bandits-at-swiggy-part-4-f5698f2af0a6

> **Interview use:** "One notification service: producers emit events, then templating, then user preferences and channel routing (push, SMS, email, WhatsApp), then a per-provider queue with retries and fallback to another channel or vendor. Keep a state per message so stuck ones get retried, and keep transactional and promotional traffic in separate priority lanes."

---

## 10. Payments

57. **ML payment-gateway routing at Swiggy.**
    - Swiggy routes each online transaction to a payment gateway (PG) to maximise success rate and minimise cost.
    - Gateways perform differently per payment method, so there are separate models for netbanking (rule-based on recent success rate, with a small random-exploration share), UPI and cards (scores each PG's likely success).
    - The post discusses 5% exploration to estimate PG success for the top cards, which make up 80% of transactions.
    - [V-snip, ~Oct 2022] Source: "An ML approach for routing payment transactions". https://bytes.swiggy.com/an-ml-approach-for-routing-payment-transactions-5a14efb643a8
58. **Swiggy finance engineering scope.** Swiggy's Finance Engineering team owns money movement, payouts, commission, PG reconciliation, restaurant credit scores and a multi-tenant ledger. [V-snip] Source: same as #52.
59. **Zepto payment-then-order flow.** Orders are accepted before payment. Payment events arrive asynchronously via Kafka. The OMS create call is idempotent. A 5-minute sweeper confirms or terminates stuck unpaid orders. [V-full] Source: same as #31.

> **Interview use:** "Payments are async: create the order as PENDING_PAYMENT with an idempotency key, take the PG webhook or event, move the state, run a sweeper or reconciler for stuck ones, and keep a double-entry ledger. Route across several gateways by recent success rate, with a little exploration (Swiggy)."

---

## 11. Rate limiting and resilience

60. **Swiggy resilience post.** A circuit breaker can detect both infrastructure and dependency failures. Misconfigured retries can turn a brownout into a blackout. The author proposes gRPC deadlines propagated through Go contexts. [V-snip]
    - Source: "Designing Resilient Microservices - Part 1". https://bytes.swiggy.com/designing-resilient-microservices-part-1-6a72fe964759
61. **External API limits.** Swiggy's Distance Service hit Google's API usage limits, which drove both the central cache and the OSM fallback (#19). Google's own limits are 25 origins or destinations, 100 elements per request and 60,000 elements per minute. [V-snip]
    - Source: https://developers.google.com/maps/documentation/distance-matrix/usage-and-billing
62. **No API rate-limiter post found.** I found no Indian food or quick-commerce engineering post specifically about API rate limiting. BookMyShow's 4-tickets-per-user cap (#66) is a business-level limit.

> **Interview use:** "Rate-limit at the gateway with a token bucket per user or IP in Redis. Protect paid or limited dependencies with a shared client plus cache (Swiggy's Distance Service). Use timeouts and deadlines everywhere, retries with jitter and a retry budget, circuit breakers, and graceful degradation (Haversine instead of road distance)."

---

## 12. Ticketing: BookMyShow, District and Paytm Insider

63. **District's origin.** On 21 Aug 2024 Zomato agreed to buy Paytm's entertainment ticketing business (TicketNew and Insider) for Rs 2,048 crore, all cash.
    - Orbgen Technologies (movies): Rs 1,264.6 cr.
    - Wasteland Entertainment (events): Rs 783.8 cr.
    - FY24 revenue Rs 297 cr, adjusted EBITDA Rs 29 cr. About 280 employees moved.
    - The business would launch as a new app, "District". [Press]
    - Sources:
      - https://www.theweek.in/wire-updates/business/2024/08/21/del88-biz-ld-paytm-zomato.html
      - https://www.outlookbusiness.com/news/zomato-set-to-acquire-paytms-entertainment-and-ticketing-business-for-rs-2048-crore
64. **Coldplay Mumbai sale crash, 22 Sep 2024.**
    - BookMyShow's site and app went down just before the sale opened, for roughly 15 to 20 minutes.
    - When it came back, queues were reported at 7 to 10 lakh users. Fans saw messages like "six lakh people ahead of you".
    - BookMyShow said the queue system was introduced to manage the demand, and capped purchases at 4 tickets per user.
    - A third Mumbai show (21 Jan 2025) was added. [Press]
    - Sources:
      - https://www.businesstoday.in/india/story/coldplay-india-tour-2025-band-adds-3rd-mumbai-show-amid-ticket-chaos-on-bookmyshow-details-here-446947-2024-09-22
      - https://www.outlookmoney.com/news/coldplay-bookmyshow-ticket-fiasco-why-are-fans-calling-out-a-foul-play
65. **Aftermath: bots, resale and police.**
    - A complaint by lawyer Amit Vyas alleged that bots and queue manipulation let touts take control. Tickets were listed on Viagogo at 30 to 50x face value.
    - BookMyShow denied any association with resale platforms and filed a police complaint.
    - The Mumbai EOW summoned the CEO. [Press, Sep 2024]
    - Sources:
      - https://thenewsminute.com/news/coldplay-india-concert-complaint-against-bookmyshow-alleging-unethical-practices
      - https://www.theticketingbusiness.com/2024/09/bookmyshow-dismisses-scalping-claims-following-criminal-complaints-over-coldplay-sale-in-india/
      - https://rollingstoneindia.com/coldplay-india-tour-bookmyshow-controversy-timeline
66. **Ahmedabad sale redesign, 16 Nov 2024.**
    - Users entered a virtual waiting room between 11:00 and 11:59, and arriving early did not affect position.
    - At 12:00 each user got a random queue position.
    - When a user's turn came, they had 4 minutes to pick up to 4 tickets.
    - Reported queues were about 3.5 lakh, and the first show sold out in minutes. Users still reported being dropped from the queue or logged out. [Press]
    - Sources:
      - https://embed.wionews.com/entertainment/coldplay-fans-upset-once-again-as-ahmedabad-concert-queues-go-in-lakhs-776663
      - https://www.theweek.in/news/entertainment/2024/11/16/coldplay-ahmedabad-tickets-sold-out-third-party-seller-viagogo-starts-re-selling.amp.html
67. **BookMyShow's own description of its queue (X, Aug 2023).** A queue system manages traffic so users wait their turn to enter the booking page. "A queue position along with the approximate wait time will be assigned to you at random once the booking goes live." [V-snip, primary]
    - Source: https://x.com/bookmyshow/status/1696528962837729549
    - **[UNVERIFIED]**: which vendor BookMyShow uses. Blogs variously claim Queue-it, or Cloudflare Waiting Room capped at 10,000 concurrent users.
68. **BookMyShow scale and data platform.** BookMyShow sells up to 200M tickets a year (pre-COVID run rate). It moved its 15-year-old analytics platform to AWS in 4 months, with 90% storage-cost and 80% analytics-cost savings. [Vendor, AWS]
    - Source: https://aws.amazon.com/blogs/business-intelligence/how-bookmyshow-saved-80-in-costs-by-migrating-to-an-aws-modern-data-architecture/
69. **Seat locking: no primary source found.** BookMyShow, District and Insider have published nothing on how they lock seats. The usual design ("Redis SETNX with a 5 to 10 minute TTL per seat, DB as source of truth") appears only in third-party blogs. **[UNVERIFIED as company practice]**

> **Interview use:** "For a flash sale, put a waiting room in front of the system. Give random positions to everyone present at T0 so there is no advantage to arriving early (BookMyShow's Ahmedabad model). Admit users at a rate the booking service can take, give each admitted user a fixed checkout window (4 minutes), and cap tickets per user. Hold seats with a TTL lock (Redis SET NX PX or a DB row with expires_at), confirm on payment, and release on expiry. Rate-limit and use bot detection at the edge, because fairness is a design goal: the Coldplay sale became a police case."

---

## 13. Outage stories and lessons

70. **Instamart "Indexing Gone Wrong" (Nov 2022).**
    - Symptoms: Kafka consumer lag on the order topic, then RDS CPU above 90% (normal is 20 to 30%) on the picker master DB.
    - Cause: a composite index in the wrong column order, so it was used only intermittently. It was triggered by a picker-payout feature release.
    - A feature-flag rollback did not reach the backend, so Swiggy shut Instamart down pan-India and killed long-running queries. One summary puts the outage at about 20 minutes.
    - [V-snip] Source: https://bytes.swiggy.com/behindthebug-indexing-gone-wrong-6b4d682fd805
71. **"Kafka Under The Water".** A few partitions in one Kafka cluster went offline for 4 minutes and took down 18 internal services. A Java Kafka client bug left producers with stale metadata. Force restarts restored normal operation within about 2 hours. [V-snip, from the series listing]
    - Source: https://bytes.swiggy.com/behindthebug-kafka-under-the-water-288c3d05b202
72. **"Breaking the Event Pipeline" (Nov 2022).** A 95-minute outage. During a Kafka-to-SQS migration, a misconfigured feature flag in the Order Data Service stopped event processing for all topics, not just the intended one. [V-snip, from the series listing]
    - Source: https://bytes.swiggy.com/behindthebug-breaking-the-event-pipeline-a-migration-gone-wrong-23ed215d6754
73. **Ad events lost.** A malformed backend payload in app v1075 caused ad events to go missing, losing ad revenue for about a week. Tests covered the new feature but not the ad events. [V-snip, from the series listing]
    - Source: https://bytes.swiggy.com/behindthebug-the-invisible-mistake-an-overlooked-backend-payload-and-its-impact-on-ad-revenue-d57a7bcdf260
74. **Swiggy RCA process.** RCAs are blameless and reviewed in "Architect Office Hours", with action items prioritised P0 to P2. [V-snip]
    - Source: https://bytes.swiggy.com/root-cause-analysis-swiggy-2fac5fe4510b
75. **Other outages (press).**
    - Swiggy outage of more than 30 minutes at peak lunch, 16 Mar 2022: https://www.theweek.in/news/biz-tech/2022/03/16/swiggy-faces-outage-delivery-service-disrupted.amp.html
    - Instamart outage of more than 1 hour on 29 Oct 2024, a week before IPO; cause not disclosed: https://inc42.com/buzz/swiggy-instamart-glitches-a-week-before-going-public/
76. **Lambda by Blinkit outage posts.** Grofers' 2019 engineering review lists posts on Kubernetes deployments and "sustaining DNS outages". [V-snip, title-level]
    - Source: https://blinkit.com/blog/grofers-lambda-2019-year-review/

> **Interview use:** "Failure modes to name:
> - a bad index or query pinning the DB CPU;
> - a feature-flag change with too wide a blast radius;
> - Kafka client metadata bugs;
> - a model emitting impossible values;
> - a ticket on-sale crashing at T0.
>
> Mitigations: kill switches that really reach the backend, query review with EXPLAIN in CI, canary and percentage rollouts (Zepto used 10%, 30%, 100%), output validation with a fallback model, and alerts on consumer lag as an early signal."

---

## Gaps (searched, nothing primary found)

- How any Indian quick-commerce company reserves inventory or prevents overselling.
- BookMyShow, District or Insider seat-locking internals, and which waiting-room vendor BookMyShow uses.
- Blinkit's current engineering architecture. Lambda posts are mostly from the Grofers era, and Blinkit pages returned 403 when fetched.
- Flipkart Minutes architecture.
- Swiggy's ping frequency for rider location, and an official Swiggy statement of geohash vs H3 usage beyond the distance-cache keying.
- Exact publication dates for most bytes.swiggy.com posts. Medium blocked fetching; dates marked "~" are estimates.
