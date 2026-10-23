# Zomato group (Zomato, District, Nugget, Blinkit, Hyperpure): Round 2 and later, with DSA focus (2023 to Oct 2026)

## 0. Method and how far to trust this

- **LeetCode Discuss:** searched through the public GraphQL endpoint (`ugcArticleDiscussionArticles`, keywords zomato, eternal, blinkit, district, hyperpure, nugget, grofers, bistro). That gave 337 matching titles. I then read the full text of every 2022–2026 post through `ugcArticleDiscussionArticle`, which came to 81 posts with round content.
  - The search seems to match **titles only**, so a post that mentions Zomato only in its body is missed.
- **Other sources:**
  - Medium, read through `freedium-mirror.cfd`: Shagun Yadav (District intern, Dec 2025) and Shivam Bhadani (Zomato SDE-1, Dec 2024).
  - GeeksforGeeks: Zomato x Blinkit SDE-I on-campus 2024.
  - The LeetCode "Zomato" company tag, via github.com/liquidslr/leetcode-company-wise-problems. It has only 30 rows and no Blinkit list.
  - Glassdoor and Naukri Code360 return 403. I used their search snippets only, marked [SNIPPET].
- **Excluded:** jobrise.io "Zomato interview questions 2026". It cites no individual report, so the problems look generated.
- **Report IDs:**
  - Z/B/N-numbers (Z1, B3, ...) match `sources/interview_reports.md`.
  - **N** and **NB** numbers are reports that were **not** in that file (new in this pass).
  - URLs are `https://leetcode.com/discuss/post/<id>/` unless given in full.
- **LeetCode mappings:**
  - "=" means the same problem.
  - "≈" means the same technique on a reworded or story version. This is my mapping, not the candidate's.
  - Premium problems are marked (P).
- **Sample size is small.** About 45 Zomato-group reports give round-level detail from 2023–2026, and about 25 of them name an actual R2+ DSA problem or say there was none. Read the frequencies as "seen N times", not as probabilities. Candidates repeatedly say Zomato interviews are "very unpredictable" and depend on the interviewer (Shivam Bhadani; 4832993; 5007288).

---

## (a) What Round 2 looks like

### a1. The typical loop

| Company | R1 | R2 | R3 (+R4) |
|---|---|---|---|
| **Zomato** | DSA + resume + CS/DB fundamentals, often a mini schema or HLD. 60–105 min. SDE-2/SDE-3/Senior SE. | **Mixed: design/HLD/LLD + resume grill + 1–2 DSA, usually "design-flavoured" DSA.** 75–90 min. EM, Team Lead, SDE-2 or Senior Engineer. | Usually VP culture fit (20–40 min, onsite, offer often on the spot). **Sometimes a technical EM round before the VP**, which is eliminatory (Z1, Z2). |
| **District** | DSA + backend fundamentals, or DSA + HLD | Resume grill + DSA (Z13), or DS design + SQL + HLD (Z11), or API + code (Z15), or Trie + seat-allocation algorithm (N11) | Managerial/EM, 30 min (Z13, N11); experience + demos (Z11) |
| **Blinkit** | **The DSA round** (SDE-3 interviewer): 1–2 problems, often LRU/LFU or BS + DP | **The system design round** (Team Lead / SDE-3), on draw.io. **Usually no DSA** (B1, B2, B4, B6, B12, B13, B14, NB1, NB6). Exceptions: a DSA warm-up (B3) and a working API in code (B5). | Manager/HM culture fit, 20–40 min, offer in the round. Sometimes a Kafka deep-dive (B6) or live business problems (B3). |
| **Nugget** | DSU problem + light design | Pure HLD (ad pacing) | Senior EM culture stress test, then VP |
| **Hyperpure** | No usable report found | | |

### a2. Evidence for "Zomato R2 = design + DSA mix", one line per report

| ID | Year | Interviewer | R2 contents | DSA in R2? |
|---|---|---|---|---|
| Z1 8434690 | Aug 2026 | SDE-2 Core team, 1h20 | **Hard DP, working code in an IDE** (20–25 min) → Redis internals, hot key, thundering herd → **LFU pseudocode + how LRU works** | Yes, hard |
| Z2 8377019 | Jul 2026 | Team Lead, 1 h | Rate-limiting discussion → **code token bucket + fixed window** (sliding window not finished) → **Jump Game IV (LC 1345)** + "each step has a cost" follow-up → **2N×2N matrix row/col flips, max top-left N×N sum** | Yes, 3 problems in 60 min |
| Z5 7407389 | Dec 2025 | Onsite, level not stated | Internship phase-2 redesign, monolith vs microservices, RabbitMQ/Redis Streams/Kafka → **implement a simple rate limiter, then sliding window in pseudocode** → support-chat design → Mongo compound index → **longest valid parentheses *subsequence*** | Yes, medium |
| Z6 7206483 | Aug 2025 | Engineer (named) | Project grill → LLD menu schema (Domino's configs) → **suggest a dish price between the 1/3 and 2/3 percentiles: sort, then two heaps** (pseudocode) | Yes, medium |
| Z7 6493240 | Mar 2025 | not stated | Multiprocessing vs multithreading → **rate limiter (fixed/sliding window with Redis; token/leaky trade-offs)** → **order-count API for the last X ≤ 30 min: 30 circular buckets + timestamps, ReentrantLock** | Design-DS |
| Z9 6023375 | Nov 2024 | **EM, 9 yrs at Zomato**, 1h20 | Intro without the resume, "why not your own startup", hardest problem → **puzzle (10 boxes, one weighing)** → **delete Fibonacci-valued nodes from a DLL. Asked for 4–5 approaches, then "pick one and code it; logic must be right first time"** | Yes, easy-medium + puzzle |
| Z10 6164662 | Dec 2024 | not stated | **2 medium-hard graph problems: constrained shortest path (Dijkstra/Bellman-Ford) + topological sort with cycles** → ACID/isolation, NoSQL → resume deep-dive | Yes, 2 graph |
| Z29 (Shivam Bhadani, Medium) | Dec 2024 | Senior Engineer, 6 yrs, 75–80 min | ~70 min HLD ("notify users when a restaurant reopens after New Year, only if delivery partners are available") → last 10 min: **code Ctrl+R reverse command search with OOP; Trie expected** | Yes (Trie), short |
| N1 6997937 | Jul 2025 | not stated | OS projects grill → **"very standard BST traversal", tested on skewed trees and negative values** | Yes, easy |
| N3 4832993 | Feb 2024 | not stated, 1h15 | REST, thread safety of a project, **HashMap internals** → **observation-based coding question run on an online compiler, then a variation; "more optimised?"** | Yes, medium |
| Z8 4068294 | Sep 2023 | not stated | Zomato-situation questions → image-upload system design → **write the CRUD API code**, SQL joins, indexing | Code, not DSA |
| Z19 3384363 | Apr 2023 | not stated | Load balancer, token/leaky bucket, consistent hashing → request lifecycle → Spotify schema | No |
| N12 (GFG Zomato x Blinkit on-campus 2024) | 2023–24 | not stated | **Go coding: backend request logic with goroutines/channels and timeouts** + rate-limiter design + GitHub/projects review | Concurrency code |
| Z11 6647834 (District) | Apr 2025 | not stated | **Restaurant timing: one in-memory structure for "slots of restaurant R" and "restaurants open now"** → index internals, BST vs RB tree, BST insert/update/delete → anti-join SQL → deadlock analysis → contract-management HLD/LLD | Design-DS |
| Z13 7617100 (District) | Mar 2026 | not stated, 1h30 | 1 h resume grill (why X over Y, library trade-offs) → **implement the algorithm behind a library you used** (recursion; DP optimisation asked) | Yes, custom |
| Z15 8559623 (District) | Oct 2026 | not stated | Paginated transaction-history API; cursor (createdAt, id); 3 source tables; **write the backend code + SQL** | Code, not DSA |
| N11 (Shagun Yadav, District intern) | Dec 2025 | not stated | **Trie over a large text: fast traversal, longest common substring of two sentences** ("implementation-heavy"), push_back vs emplace_back → **movie seat-allocation algorithm**: edge cases, concurrent requests, seat suggestion | Yes |
| Z14 8540075 (Nugget) | Aug 2025 | SDE-III | Ad-pacing HLD only | No |

**Blinkit R2 (for contrast):**
- B1 and B2: design on draw.io, "more about design thinking than code".
- B3: a DSA warm-up first: two arrays of unique numbers; does a unique-element array exist with both as subsequences? Then BookMyShow probes.
- B4: rider onboarding + DynamoDB.
- B5: **a curl-able working API with Postgres**.
- B6, B13: design.
- B12: BookMyShow LLD.
- B14: ZoomCar LLD.
- B7: WhatsApp + nth-salary SQL + mutex/semaphore.
- NB1: inventory system.
- NB6 (SDE-2): event scheduler HLD.
- B10 (intern): R2 was titled "Problem Solving & DSA" but was a **browser back/forward/new-tab class** with test cases.

### a3. How R2 differs from R1, in Zomato proper

1. **Interviewer seniority goes up.** R1 is SDE-2/SDE-3/Senior SE. R2 is EM, Team Lead or a senior engineer (Z2 TL, Z9 EM, Z29 6-yr SE; Z1 was SDE-2).
2. **DSA becomes design-flavoured.** R1 problems are classic LeetCode: LC 209, 651, 268, 218, 215, 207, DSU, min-swaps. R2 problems are often a **data structure built for a product feature**:
   - rate limiter classes, the last-X-minutes order counter, the percentile price with two heaps, Ctrl+R with a Trie, the restaurant open-hours structure, Fibonacci DLL deletion, implementing a library's algorithm.
3. **Hard problems still appear in R2** when the interviewer is a coder:
   - Z1: hard DP with working code.
   - Z2: LC 1345 (hard) + matrix flip + rate-limiter code in 60 min.
   - Z10: two medium-hard graph problems.
4. **More of R2 is resume grill and "why X over Y".** Z6 and Z13 spent most of the round on it, and B15 warned that you need to know your systems' failure modes.
5. **Code is still expected.**
   - Z1 wanted working code in an IDE.
   - Z2 wanted the rate limiter implemented.
   - Z15 was rejected for messy hand-written code.
   - Z8 and B5 had to write API code.
   - Z9's EM did not care about syntax, but "logic should be correct in the first go".
   - Z6 and Z5 got away with pseudocode.
6. **Concurrency appears more in R2:** ReentrantLock in Z7, goroutines and channels in N12, the seat-allocation race in N11 and B3, deadlock analysis in Z11.

### a4. R3 / EM / culture rounds

- **VP culture fit, the most common case:**
  - Why Zomato, why not your own startup, goals, pace, 5 days in office in Gurgaon, the non-9-to-5 warning.
  - 20–40 min, often onsite, offer verbal on the spot.
  - Reports: Z5, Z6, Z7, Z9, Z10, Z29, N3, Z8, N1, Z25.
  - Z29 said the VP round "was not an elimination round".
- **Technical EM round before the VP (newer, 2026):**
  - Z1 R3: an EM with 10+ years: internship design decisions + one HLD + DB choice for AI agents.
  - Z2 R3: a Senior EM, in person: a resilient notification system (dedupe, crash resilience, retries). **The candidate was rejected here.**
  - Z27 (Medium, 2026) had 5 rounds: R3 was a 1h40 Kafka/LB/HLD round; R4 was multi-tenant DBs.
- **District R3:** managerial, 30 min, offer details (Z13); live project demos + Docker (Z11); EM discussion (N11).
- **Nugget:** Senior EM culture stress test ("10–14 hr days, weekends"), then VP (Z14).
- **Blinkit R3:**
  - Manager/HM: resume + behavioural (challenging problem, disagreement, ownership, prioritisation); 20–40 min; offer in the round (B1, B2, B13, B14, B4, NB1).
  - It can turn technical: a Kafka async-order deep-dive (B6); two live business problems in 30 min (B3); HLD + project for SDE-2 (NB6).
- **No DSA was reported in any R3/R4** in the Zomato group from 2023–2026.

---

## (b) DSA problems by round

### b1. R2 and later, Zomato group 2023–2026 (frequency-ranked by pattern)

| Rank | Pattern | Count (reports) | Problems as asked → LeetCode mapping | Report IDs |
|---|---|---|---|---|
| 1 | **Rate limiter / time-window counters, coded** | **5** | Token bucket + fixed window coded; sliding window asked (Z2). Simple limiter + sliding window pseudocode (Z5). Fixed/sliding window with Redis + log (Z7). Orders in the last X minutes using circular buckets + lock (Z7) **≈ LC 362 Design Hit Counter (P)**. Rate-limiter design with memory optimisation (N12). Warm-up: **LC 359 Logger Rate Limiter (P)** | Z2, Z5, Z7, N12 (+Z19 discussed only) |
| 2 | **Graphs: BFS / Dijkstra / topological sort** | **3** (+1 uncertain) | **Jump Game IV = LC 1345** + weighted follow-up → Dijkstra (Z2). Constrained shortest path ≈ **LC 787** (Z10). Topo sort + cycles ≈ **LC 207/210** (Z10). Unique supersequence of two sequences ≈ topo order uniqueness / cycle check, **≈ LC 444 Sequence Reconstruction (P)** (B3, Blinkit R2). Observation-based problem that "looked like a graph" (N3) [UNCERTAIN] | Z2, Z10, B3, (N3) |
| 3 | **Design-a-data-structure for a product feature** | **5** | Restaurant open-hours structure (Z11) ≈ intervals + per-minute buckets / sweep. Ctrl+R reverse search (Z29) ≈ **LC 208 Trie** / **LC 1268**. Browser history (B10) = **LC 1472**. LFU pseudocode + LRU explanation (Z1) = **LC 460 / LC 146**. Order counter (Z7, counted in rank 1) | Z1, Z11, Z29, B10, Z7 |
| 4 | **Trie / string search** | **2** | Ctrl+R (Z29). Trie over text + longest common substring of two sentences (N11) ≈ **LC 208 + LC 718** (substring DP) | Z29, N11 |
| 5 | **DP (often hard)** | **2** | "Hard DP, working code expected" (Z1, unnamed). Implement a library's algorithm: recursion → DP asked (Z13, custom) | Z1, Z13 |
| 6 | **Heaps / order statistics** | **1** | Percentile price via two heaps (Z6) ≈ **LC 295** generalised to the 1/3 and 2/3 positions | Z6 |
| 7 | **Trees / BST** | **2** | "Standard BST traversal" with skewed/negative tests (N1) ≈ LC 94/145 iterative. BST insert/update/delete theory + BST vs RB tree (Z11) ≈ **LC 450/701** | N1, Z11 |
| 8 | **Linked list** | **1** (+1 Blinkit) | Delete Fibonacci-valued nodes from a DLL, 4–5 approaches (Z9). Blinkit "R2 DSA + tech", effectively the first technical round: circular LL + **LC 138 Copy List with Random Pointer** + a non-standard array problem (B8) | Z9, (B8) |
| 9 | **Stack / parentheses** | **1** | Longest valid parentheses *subsequence* (Z5) → greedy count of matched pairs ×2 (≈ the LC 1249 / 921 counting idea; **not** LC 32) | Z5 |
| 10 | **Greedy / matrix observation** | **1** | 2N×2N matrix, reverse any rows/cols, maximise the top-left N×N sum (Z2) = HackerRank "Flipping the Matrix" (max of the 4 mirror cells). Not on LeetCode | Z2 |
| 11 | **Concurrency coding** | **2** | ReentrantLock around buckets (Z7). Goroutines/channels + timeout handling (N12) | Z7, N12 |
| 12 | **Seat allocation algorithm (semi-design)** | **1** (+B3 design) | Movie seat allocation: edge cases, simultaneous requests, suggestions (N11) ≈ LC 1386 + locking | N11 |
| — | **Working backend code instead of DSA** | 3 | CRUD API code (Z8). Paginated multi-table API + SQL (Z15). curl-able transactions API on Postgres (B5) | Z8, Z15, B5 |
| — | **No DSA in R2** | 9+ | Z14, Z19, B1, B2, B4, B6, B12, B13, B14, NB1, NB6, B7 (SQL only) | |

**Older R2 evidence (2022, weaker weight):**
- 2666881 (EM R2): alerting when more than 3 requests in 1 s or more than 25 in 10 s, counted over a sorted timestamp array. This is a two-pointer sliding window, again a rate-limit flavour.
- 1798957 (techno-managerial R2): reverse a linked list, recursive and iterative (LC 206), + e-commerce schema.
- 2251731 (Android R2): card-deck LLD (pop, shuffle, distribute). R3: implement LiveData + LRU internals.

### b2. R1 for contrast, Zomato group 2023–2026

| Pattern | Count | Problems → LeetCode | Report IDs |
|---|---|---|---|
| **DP** | 9 | LC 651 4-Keys Keyboard (P) (Z2). House Robber I/II LC 198/213 (B3). House Robber II + Combination Sum blend LC 213 + LC 39 (B6). LIS variant LC 300 (B5). DP on strings O(n²) (Z18). Unnamed DP (Z10). 1D DP (NB2). Best Team With No Conflicts LC 1626 (B7). Maximal Square LC 221 (N12). Graph + DP, hard (Z24) | Z2, B3, B6, B5, Z18, Z10, NB2, B7, N12, Z24 |
| **Graphs / DSU / MST** | 8 | Friend circles LC 547 + redundant edges LC 684, DSU from scratch (Z14). PQ greedy → Prim's (Z7). Components + "same component?" queries (Z29). Course Schedule LC 207 (N4). Rotting Oranges LC 994 (B5). Dijkstra with "add at most one edge from a set" in O(E log V): Dijkstra from src + reversed from dst (B12, SDE-2). Group plagiarising students ≈ DSU, ≈ LC 721 / LC 952 (B13, SDE-2). Max bipartite matching ≈ LC 1820 (P) / Kuhn's algorithm (Z17, SDE-2) | Z14, Z7, Z29, N4, B5, B12, B13, Z17 |
| **Binary search** | 6 | Find Peak LC 162 (B1). Rotated array LC 33 (B2). BS on answer (N3). Search 2D Matrix II LC 240 (NB4). Single Element in Sorted Array LC 540 (B14). Search in a spirally sorted 2D array (NB6, SDE-2) | B1, B2, N3, NB4, B14, NB6 |
| **Sliding window / prefix** | 5 | LC 209 + negatives follow-up → LC 862 (N1). LC 209 (N2). LC 2958 (Z15). Count distinct in every window of size k (NB1). Longest substring with equal 0s and 1s ≈ LC 525 (Z19) | N1, N2, Z15, NB1, Z19 |
| **Design-a-DS** | 7 | LFU LC 460 (B1). LRU LC 146 (B2). LC 380 (B15 SDE-2; NB5 SDE-2, failed). O(1) push/pop/getMin/getMax ≈ LC 155 + LC 716 (P) (Z16 SDE-2, rejected). Snake game ≈ LC 353 (P) (Z12 SDE-2). Oncall system, multi-part, full code (B11). [SNIPPET] Glassdoor Mar 2026 Zomato: "LRU Cache, Median in a stream" | B1, B2, B15, NB5, Z16, Z12, B11 |
| **Trees / BST** | 5 | BST delete LC 450 (B4 needed a hint; Z24 two-children variant; Z21). Iterative postorder LC 145 (B7). LCA LC 236 (NB2) | B4, Z24, Z21, B7, NB2 |
| **Arrays / greedy / math** | 11 | Min swaps to sort a permutation (Z6). O(n), no extra space (N1). Missing Number LC 268 (N2). Airplane seats greedy + Increasing Triplet LC 334 (N11). Cinema Seat Allocation LC 1386 (Z11). Frogs on stones, O(1) greedy (B4). Merge-interval variant O(n log n) → O(n) with a map (Z13). Majority Element II LC 229 (Z22). Find and Replace Pattern LC 890 (N16). Single Number LC 136 + String Transforms LC 1153 (P) (B14). Flatten JSON (Z19) | many |
| **Heaps** | 3 | Kth largest LC 215 (Z8). Kth Largest in a Stream LC 703 (NB3, SDE-3, rejected anyway). PQ greedy (Z7) | Z8, NB3, Z7 |
| **Hard "classic"** | 2 | Skyline LC 218 (N2). LC 1153 (B14) | N2, B14 |
| **Linked list / stack** | 3 | Reverse LL LC 206 (NB7). Reverse k-group LC 25 (Z21). Min/max stack (Z16) | NB7, Z21, Z16 |

**Takeaway:**
- R1 is where the classic LeetCode set lives: DP, graphs/DSU, binary search, sliding window, BST, LRU/LFU.
- R2 in Zomato proper reuses graphs, DP and heaps, but more often asks you to **code a small system component**: a rate limiter, a time-bucket counter, a Trie-backed search, an open-hours index, a percentile tracker, an LFU.
- Blinkit R2 is mostly not DSA.

### b3. LeetCode "Zomato" company tag (top 30, frequency %, liquidslr repo)

- **100:** Maximum Spending After Buying Items (2931), Split the Array to Make Coprime Products (2584), Bitwise OR of All Subsequence Sums (2505, P), Shortest Cycle in a Graph (2608), Number of Ways to Divide a Long Corridor (2147).
- **96:** Count the Hidden Sequences (2145).
- **87:** Exchange Seats (626, SQL).
- **81:** Skyline (218).
- **73:** Maximum Subarray (53), LC 3, LC 540, Minimize Malware Spread (924).
- **62:** First Missing Positive (41), LC 200, LC 909, LC 260, LC 221, LC 205, LC 207, Rearrange String k Distance Apart (358, P), LC 46, LC 283, LC 39, LC 240, LC 56, LC 354, LC 2812, Maximum Xor Product (2939), Get the Maximum Score (1537), **LRU Cache (146)**.
- The tag mixes OA and interview problems and is not split by round. Several hards (2584, 2505, 2147) look like OA problems [UNCERTAIN]. Overlap with interview reports: LC 146, 207, 221, 218, 540, 240, 39, 56.

---

## (c) "Design a data structure" questions and their follow-ups

| Question | Where / round / level | Follow-ups reported |
|---|---|---|
| **LRU Cache (LC 146)** | B2 R1 (2026); Z1 R2 "how LRU works" (2026); 2251731 R3 Android (2022); Glassdoor Mar 2026 [SNIPPET]; Zomato tag | Why both a hash map and a DLL; walk through a sample sequence of operations; edge cases; complexity (B2) |
| **LFU Cache (LC 460)** | B1 R1 (pseudocode + dry run); **Z1 R2 pseudocode after a Redis eviction discussion** | Tied to Redis: eviction policies, hot keys, thundering herd (Z1). Get/put only (B1) |
| **Insert Delete GetRandom O(1) (LC 380)** | B15 R1 SDE-2 (solved, still rejected); NB5 R1 SDE-2 (failed) | Swap-with-last deletion; map index update |
| **O(1) insert/delete-at-end + getMin + getMax** (≈ LC 155 + LC 716) | Z16 R1 SDE-2, dropped in the last 15 min | Candidate failed to see the "stack of (val, min, max)" idea and was rejected. "Interviewer will never drop hints." |
| **AllOne (incr/decr/getMax/getMin, LC 432)** | 1743607 R1 (2022), after "most frequent string" | Most efficient structures |
| **Rate limiter classes** (token bucket, fixed window, sliding log/counter, leaky bucket) | Z2 R2 (code), Z5 R2 (code + pseudocode), Z7 R2, N4 R1 (leaky bucket coded; "optimise for scaling" not achieved), Z31 R1 District, N12 R2 | Boundary burst in fixed window; sliding window; memory optimisation; Redis-backed; scaling across nodes (N4) |
| **Orders in the last X minutes** (≈ LC 362) | Z7 R2 | Circular buckets with timestamps to reset stale ones; **thread safety (ReentrantLock)** |
| **Percentile price tracker** (≈ LC 295 generalised) | Z6 R2 | Sorting O(n log n) → **two heaps** that keep the 1/3 and 2/3 boundaries |
| **Restaurant open-hours index** | Z11 R2 District | One in-memory structure must answer both queries; data refreshed daily; (my note: handle intervals that cross midnight) |
| **Ctrl+R reverse command search** | Z29 R2 | OOP class with public/private methods; naive → **Trie** expected (candidate couldn't implement the Trie and still passed) |
| **Browser history (LC 1472)** | B10 R2 Blinkit intern | List all test cases; choice of data structures; class design; optimise after the first version |
| **Snake game, O(1) move (≈ LC 353)** | Z12 R1 SDE-2 | Growth on eating food; data-structure trade-offs (deque + hash set); pseudocode |
| **Oncall system (multi-part)** | B11 R1 Blinkit | Each part builds on the last; explain the model, then write complete code |
| **Alarm clock (add/delete/update/snooze)** | N6 6173435, SDE-2, round not stated | Infinite loop → **min-heap of next alarm + sleep until it**, reset on update/snooze; interviewer wanted an OS-scheduler/timer approach "like cron" |
| **Thread-safe queue** | Z22 single round, on-campus 2023 | Multithreading emphasis |
| **Priority queue from scratch** | N15 3926420, on-campus 2023 | (not detailed) |
| **Paths → YAML** (`/web/leetcode/contest: 435` → nested YAML) | N5 5989692 SDE-2, round not stated | Build a tree/trie of path segments, DFS print with indentation |
| **Flatten nested JSON to dotted keys** | Z19 R1 | |
| **1M objects with 3–4 properties: fast group queries** | Z19 R1 | Which structure and why; how a query runs on it |
| **Card deck (pop, shuffle, distribute)** | 2251731 R2, 2022 Android | Card class with 4 suits and 2 colours |

---

## (d) R2+ follow-up patterns and rejection reasons

**Follow-up patterns seen in R2+:**
1. **Add weights → switch algorithm.** LC 1345 BFS becomes Dijkstra when each step has a cost (Z2).
2. **Optimise the data structure.** Sort → two heaps (Z6). Naive search → Trie (Z29). Recursion → DP (Z13). "Can you optimise the variation further?" (N3).
3. **Many approaches, then pick one.** The EM asked for 4–5 approaches to the Fibonacci DLL deletion and **praised picking the readable one over the optimal one** (Z9). Increasing Triplet with multiple approaches (N11, R1).
4. **Thread safety / concurrency.** Locks around buckets (Z7). Concurrent seat requests (N11, B3). Goroutines with timeouts (N12). Deadlock analysis (Z11).
5. **Fixed window → sliding window.** Z2 could not finish the sliding window; Z5 was asked to pseudocode it.
6. **Edge-case batteries.** Skewed trees, negative values (N1). "Figure out all test cases" (B10).
7. **Prove it, not code it.** Graph problem, proof of correctness only (Z3, Search team R1, 2026).
8. **Design DSA → systems tie-in.** LFU pseudocode right after Redis eviction (Z1). Rate-limiter code inside a rate-limiting design talk (Z2, Z5, Z7).

**Rejection reasons and feedback at R2+:**

| ID | Round | Stated reason |
|---|---|---|
| Z2 | R3 (Senior EM) | Could not confidently justify **MySQL vs PostgreSQL**; over-explained instead of admitting limited exposure. **Needed 2–3 iterations to grasp a follow-up, which was "a negative signal".** The design itself was judged good. The candidate had solved 2 LC hards + 2 mediums in R1/R2. |
| Z15 | R2 (District) | Self-reported: messy hand-written backend code, "too comfortable with AI-assisted coding". |
| B7 | R2 (Blinkit) | Not prepared for system design (WhatsApp). |
| B12 | R2 (Blinkit SDE-2) | LLD (BookMyShow) left unfinished; too much time on the project discussion. |
| N3 | R2 (passed, with feedback) | Weak explanation of own prior work and REST; **confused hash map with ordered map**. Passed because the reasoning was good. |
| Z9 | R2 (passed) | Failed the puzzle in its 2-minute limit and still passed; readable-code choice praised. |
| Z29 | R2 (passed) | Could not implement a Trie and still passed on the strength of everything else. |
| Z4 | R1 Senior EM (for contrast) | Happy path only; needed non-happy paths (retries, cross-service consistency, DB modelling). "~80% close". |
| Z16, NB5, B15 | R1 SDE-2 (for contrast) | Couldn't recall the stack trick (Z16); couldn't solve LC 380 (NB5); solved LC 380 but was rejected, probably on resume depth (B15, candidate's guess). |
| Z24, N2, Z23 | R1 on-campus (for contrast) | Needed hints on the 3rd DSA problem, "they want 100% accuracy" (Z24, guess). No reason given (N2). An off-by-one in a rushed DSA problem plus SQL (Z23). |

**Pattern:** R2+ rejections are rarely about not knowing an algorithm. They come from:
- shaky justification of trade-offs;
- slow pickup of follow-ups;
- messy or unfinished code;
- an unfinished design because of poor time management.

---

## (e) Recommended Round 2 problem list (52 items)

Tiers weight R2 evidence most. R1 classics that R2 interviewers have reused stay in, because Zomato R2 interviewers are "unpredictable". (P) = LeetCode premium; free substitutes are given where they exist.

### Tier 1: must do (18)

| # | Problem | Pattern | Why (evidence) |
|---|---|---|---|
| 1 | **Rate limiter: token bucket, fixed window, sliding-window log, sliding-window counter** (code all four as classes, thread-safe) | Design-DS / time windows | Coded in R2: Z2, Z5, Z7; R1: N4, Z31; N12. **The most repeated R2 coding item.** |
| 2 | **LC 362 Design Hit Counter (P)**; free version: circular 300/30-bucket array | Time buckets | Z7 R2 "orders in the last X minutes" + lock |
| 3 | **LC 359 Logger Rate Limiter (P)** | Hash map + time | Warm-up for #1 |
| 4 | **LC 146 LRU Cache** | Design-DS | B2 R1, Z1 R2, Glassdoor 2026 [SNIPPET], tag list |
| 5 | **LC 460 LFU Cache** | Design-DS | B1 R1, **Z1 R2** |
| 6 | **LC 295 Find Median from Data Stream** + variant "keep the 1/3 and 2/3 percentiles" | Two heaps | Z6 R2 |
| 7 | **LC 1345 Jump Game IV** | BFS with value buckets | Z2 R2 |
| 8 | **LC 743 Network Delay Time** (Dijkstra template) | Dijkstra | Z2 R2 weighted follow-up; B12 R1 |
| 9 | **LC 787 Cheapest Flights Within K Stops** | Constrained shortest path | Z10 R2 |
| 10 | **LC 207 / LC 210 Course Schedule I/II** | Topological sort + cycle | Z10 R2, N4 R1, tag |
| 11 | **LC 444 Sequence Reconstruction (P)**; free substitute: **LC 2392 Build a Matrix With Conditions** | Topo-order uniqueness | B3 R2 (supersequence of two arrays) [mapping is mine] |
| 12 | **LC 208 Implement Trie** | Trie | Z29 R2, N11 R2 |
| 13 | **LC 1268 Search Suggestions System** (+ code Ctrl+R: latest matching command) | Trie / prefix search | Z29 R2 |
| 14 | **LC 1472 Design Browser History** | Design-DS | B10 R2 |
| 15 | **LC 380 Insert Delete GetRandom O(1)** | Design-DS | B15, NB5 (SDE-2 R1) |
| 16 | **LC 155 Min Stack** + extend to getMax (LC 716 Max Stack (P) idea) | Stack design | Z16 rejection |
| 17 | **LC 450 Delete Node in a BST** + LC 701 insert | BST | Z11 R2, B4/Z24/Z21 R1 |
| 18 | **"Restaurants open now" index** (LC 56 Merge Intervals + LC 253 Meeting Rooms II + minute buckets; handle intervals that cross midnight) | Intervals / sweep | Z11 R2, Z13 R1 (O(n log n) → O(n) with a map) |

### Tier 2: high (20)

| # | Problem | Pattern | Why |
|---|---|---|---|
| 19 | LC 547 Number of Provinces + **LC 684 Redundant Connection** (DSU with rank + path compression, from scratch) | DSU | Z14 R1, Z29 R1 (components + queries) |
| 20 | LC 721 Accounts Merge | DSU on shared keys | B13 R1 SDE-2 (plagiarism groups ≈ LC 952) |
| 21 | LC 994 Rotting Oranges | Multi-source BFS | B5 R1 |
| 22 | LC 718 Maximum Length of Repeated Subarray (= longest common substring DP) | String DP | N11 R2 (longest common substring of sentences) |
| 23 | LC 1143 LCS + LC 72 Edit Distance | 2D DP | Z18 R1 "DP on strings"; practice for "hard DP" (Z1 R2) |
| 24 | LC 1235 Maximum Profit in Job Scheduling | DP + binary search (hard) | Proxy for Z1's unnamed **hard DP in R2** [UNCERTAIN proxy] |
| 25 | LC 300 LIS (n log n) + LC 354 Russian Doll | LIS | B5 R1 variant, tag |
| 26 | LC 198 / LC 213 House Robber I/II + LC 39 Combination Sum | 1D DP / backtracking | B3 R1, B6 R1 blend |
| 27 | LC 651 4 Keys Keyboard (P); free: GfG "special keyboard" | DP | Z2 R1 |
| 28 | Longest valid parentheses *subsequence* (greedy pair count) + LC 32 (substring) + LC 1249 | Stack / greedy | Z5 R2 |
| 29 | HackerRank "Flipping the Matrix" (2N×2N) | Greedy symmetry | Z2 R2 |
| 30 | Delete Fibonacci-valued nodes from a DLL: prepare 3+ approaches | Linked list + hashing | Z9 R2 (EM) |
| 31 | LC 138 Copy List with Random Pointer + LC 206 Reverse LL (iterative and recursive) + LC 25 | Linked list | B8, NB7, Z21, 1798957 R2 (2022) |
| 32 | LC 145 / LC 94 iterative traversals (skewed trees, negative values) | Trees | N1 R2, B7 R1 |
| 33 | LC 432 All O`one Data Structure | Design-DS | 1743607 R1 (2022) |
| 34 | LC 981 Time Based Key-Value Store | Hash map + BS | Redis/KV design follow-ups (Z30, Z1) [my extrapolation] |
| 35 | LC 1386 Cinema Seat Allocation + an airplane-seat greedy with blocked seats | Bitmask / greedy | Z11 R1, N11 R1 + R2 seat allocation |
| 36 | LC 2958 + LC 209 → LC 862 (negatives) | Sliding window / mono-deque | Z15 R1, N1/N2 R1 |
| 37 | LC 215 Kth Largest + LC 703 | Heap / quickselect | Z8 R1, NB3 R1 |
| 38 | Flatten nested JSON + "paths → nested YAML" | Tree build + DFS | Z19 R1, N5 (SDE-2) |

### Tier 3: stretch (14)

| # | Problem | Pattern | Why |
|---|---|---|---|
| 39 | LC 218 The Skyline Problem | Sweep + multiset | N2 R1, tag 81% |
| 40 | LC 1820 Maximum Number of Accepted Invitations (P) / Kuhn's bipartite matching | Matching | Z17 SDE-2 R1 |
| 41 | LC 353 Design Snake Game (P) | Deque + set | Z12 SDE-2 R1 |
| 42 | Dijkstra with "add at most one optional edge": run from src, run on the reversed graph from dst, try each edge (practice with LC 2203) | Dijkstra ×2 | B12 SDE-2 R1 |
| 43 | LC 1153 String Transforms Into Another String (P) | Mapping / cycles | B14 R1 SDE-2 |
| 44 | LC 334 Increasing Triplet Subsequence (3 approaches) | Greedy | N11 R1 |
| 45 | LC 540 Single Element in a Sorted Array + LC 162 + LC 33 | Binary search | B14, B1, B2 R1 |
| 46 | LC 410 Split Array Largest Sum | BS on answer | N3 R1 |
| 47 | LC 240 Search a 2D Matrix II (+ "spirally sorted matrix" variant) | Matrix search | NB4, NB6 R1, tag |
| 48 | Min swaps to sort a permutation (cycle decomposition) | Cycles | Z6 R1 |
| 49 | LC 2608 Shortest Cycle in a Graph | BFS | Tag 100% |
| 50 | LC 2812 Find the Safest Path in a Grid | Multi-source BFS + BS/Dijkstra | Tag |
| 51 | LC 2931 Maximum Spending After Buying Items | Heap greedy | Tag 100% |
| 52 | Bounded blocking queue / producer-consumer with timeouts (LC 1188 (P)) | Concurrency | Z22, N12 R2, Z27 R3 |

**How to practise for R2 specifically:**
- Write each Tier-1 item as a **clean class** with no IDE: names, public/private methods, a lock where it applies.
- Then say out loud: the complexity; the follow-up ("weighted?", "concurrent?", "what if it's 10× bigger?"); and **2–3 alternative approaches with their trade-offs**.
- Every R2 also includes a resume grill. Be ready to defend each tech choice, or to say plainly "I don't have production exposure to that trade-off" (Z2).

---

## Appendix: new reports found in this pass (not in `sources/interview_reports.md`)

| ID | Source | Company / role / date | Content |
|---|---|---|---|
| N1 | 6997937 | Zomato SDE-1, 2025 grad, off-campus, Jul 2025 | R1: O(n) array with no extra space; LC 209 + negatives. R2: OS project grill + BST traversal edge cases. R3: culture. Outcome implied offer [UNCERTAIN]. |
| N2 | 6994052 | Eternal on-campus SDE-1, Jul 2025 | Single interview: LC 268, LC 209, LC 218 + Postgres/CAP. Rejected. |
| N3 | 4832993 | Zomato SDE-1, 1.5 YOE, Feb 2024, offer | R1: SQL + BS on answer, working code. R2: HashMap internals, thread safety, observation-based coding + variant on an online compiler. R3: VP. Google Form to choose the focus. |
| N4 | 5763557 | Zomato SE, 14 months, Sep 2024 | R1: 8-core merge sort, leaky-bucket rate limiter coded, LC 207, Go mutex vs channels, CAP. Withdrew before R2. |
| N5 | 5989692 | Zomato SDE-2, Oct 2024, rejected | Multicore merge sort; paths → YAML. Round not stated. |
| N6 | 6173435 | Zomato SDE (SDE-2, 4 YOE), Dec 2024 | Machine coding: alarm clock; heap/scheduler follow-up. Round not stated. |
| N7–N9 | 7371586, 7478818, 7573378 | Zomato SDE-2 R1s | Already Z16, Z17, Z12 in the old file; their DSA is now mapped. |
| N10 | 8275298 | Zomato SDE-2, May 2026 | "DSA round completed (1 Medium + 1 Hard LeetCode standard)"; later rounds "focus on resume". A question post, not a full report. |
| N11 | medium.com/@shagunydv/zomato-eternal-interview-experience-c9effde999af | District SDE intern, Dec 2025, offer | R1: airplane seat greedy + LC 334. R2: Trie text search + longest common substring + seat-allocation algorithm. R3: EM. |
| N12 | geeksforgeeks.org/zomato-x-blinkit-interview-experience-for-sde-i-on-campus-2024 | Zomato x Blinkit SDE-I on-campus 2024 | R1: LC 221 + ML/Redis/consistent hashing. R2: Go concurrency coding + rate limiter. R3: HR. 3 of 8 selected. |
| N13 | Glassdoor RVW98355437 [SNIPPET] | Zomato, Mar 2026, New Delhi | "DSA rounds medium and medium-hard: LRU Cache, Median in a stream". Not readable (403). |
| N14 | 4559267 | Zomato SE2, 2023, via 3rd party | Take-home course-management MC; R2 DSA + SQL (graph shortest path). Failed. |
| N15 | 3926420 | Zomato on-campus, Aug 2023 | Interview: implement a priority queue from scratch, SQL, concurrency. Not selected. |
| N16 | 3760268 | Zomato off-campus, Jul 2023 | Find and Replace Pattern (LC 890), mutex, indexing. |
| NB1 | 6367405 | Blinkit SDE-1, Jan 2025, offer | R1: distinct elements in each window of size k. R2: inventory system. R3: culture. |
| NB2 | 5750850 | Blinkit SDE-1 frontend, 2024, accepted | R1: LCA + 1D DP. R2: frontend coding. |
| NB3 | 6762060 | Blinkit SDE-3, May 2025 | R1: LC 703. Rejected without feedback. |
| NB4 | 5049266 | Blinkit SDE-1, Apr 2024 | R1: LC 240 (the interviewer insisted on an incorrect "better" solution). Rejected. |
| NB5 | 5111745 | Blinkit SDE-2, 2024 | R1: LC 380, not solved. |
| NB6 | 8356567 | Blinkit SDE-2, Jun 2026, offer | R1: Kafka/DB/OS + search in a spirally sorted 2D array. R2: event-scheduler HLD + project. R3: culture + project + HLD. |
| NB7 | 7512333 | Blinkit (level not stated), Jul 2025, rejected | DSA round: reverse a linked list + Kafka + GC. "LLD" round became HLD: warehouse management + rule engine. |
| NB8 | 5300699 | Blinkit SDE-3, Jun 2024 | R1 = 30 min tech (Kafka, SQS, NoSQL) + 30 min DSA; eliminated on the tech part. |
| NB9 | Code360 snippets [SNIPPET] | Blinkit, Oct 2025 (Rajdeep C.) | Tree path-sum to target via prefix-sum DFS (≈ LC 437) in a medium 60-min round; the next round was a project deep-dive on caching and distributed locks. A Jan 2025 Code360 report mentions a debounce implementation + system design (likely frontend). Pages return 403, so this is unverified. |

**No Hyperpure or Bistro interview content was found** beyond timeline questions (7369370, 7367617, 6743710).
