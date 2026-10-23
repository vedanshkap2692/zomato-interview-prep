# Zomato (Eternal) SDE-1 Fresher: Interview Guide

> **What this is built from.** About 40 real Zomato, Blinkit, District and Nugget interview write-ups from 2021 to Oct 2026 (LeetCode Discuss, GeeksforGeeks, Medium, Jointaro, the IIT Bhubaneswar placement notes, Glassdoor snippets), plus the LeetCode "Zomato" company-tag list.
> **"Reported N×"** means it appeared in N independent write-ups.
> **Companion files:**
> - `01b_DSA_Must_Code.md`: C++ code for the problems you must be able to write from memory.
> - `01c_CS_Fundamentals_SQL_Design.md`: DBMS, SQL, OS, CN, OOP, HLD and LLD answers.
> - `02_Resume_and_Skills_Brief.md`, `03_Projects_TraceX_SupplySenseAI.md` and `04_Experience_Myntra_Atlassian.md`.

---

## 0. Round 2: read this first (you cleared Round 1)

> Built from a second research pass: 81 Zomato-group interview write-ups read in full (2022–26), 25 of them naming the Round 2+ DSA or confirming there was none. Counts below are "seen in N reports", not probabilities. Report IDs (Z2, B10, N11…) refer to `sources/interview_reports.md` and `sources/round2_reports.md`.

### 0.1 What Round 2 looks like

| | Zomato | District | Blinkit |
|---|---|---|---|
| **Length / interviewer** | 75–90 min; EM, Team Lead or senior engineer (sometimes SDE-2) | ~90 min | Team Lead / SDE-3 |
| **Content** | **Mix:** resume grill ("why X over Y"), design (HLD/LLD/schema), and **1–2 DSA problems, usually "design-flavoured"** | Resume grill + custom DSA, or data-structure design + SQL + HLD, or API code | **System design on draw.io, usually no DSA** |
| **Code** | Working code expected (some got away with pseudocode). "Logic must be right first time" (EM, Z9) | Backend code + SQL in some reports | A curl-able API in one report |

**How Round 2 differs from Round 1:**
1. **More senior interviewer**, more "why", more trade-offs. Expect to defend every tech choice on your resume.
2. **DSA becomes a small component for a product feature**, written as a clean class: a rate limiter, "orders in the last X minutes", a percentile price tracker, Ctrl+R search, a "restaurants open now" index, an LFU cache after a Redis eviction discussion.
3. **Hard problems still appear** with coder interviewers: a hard DP with working code (Z1), Jump Game IV plus a weighted follow-up plus rate-limiter code in 60 minutes (Z2), two medium-hard graph problems (Z10).
4. **Concurrency shows up:** a lock around time buckets (Z7), goroutines and channels with timeouts (N12), concurrent seat requests (N11), deadlock analysis (Z11).

**Round 3:** usually a VP culture fit (why Zomato, pace, Gurgaon, 5 days in office), 20–40 minutes, offer often on the spot. **Newer 2026 loops insert an eliminatory technical EM round first** (notification-system HLD, DB choice). No DSA was reported in any Round 3+.

### 0.2 Round 2 DSA, ranked by what was actually asked

| Rank | Pattern | Seen | What was asked → where to practise |
|---|---|---|---|
| 1 | **Rate limiter / time-window counters, coded as classes** | 5 | Token bucket + fixed window coded, sliding window asked (Z2); simple limiter + sliding window (Z5); orders-in-last-X-minutes with circular buckets and a lock (Z7) → `01b` 12.1, 1.5 |
| 2 | **Graphs: BFS → Dijkstra, constrained shortest path, topological sort** | 3–4 | Jump Game IV + "each step has a cost" (Z2); constrained shortest path ≈ LC 787 and topo sort with cycles (Z10); topo-order uniqueness (Blinkit B3) → `01b` 3.3–3.5, 12.3, 12.4 |
| 3 | **Design-a-data-structure for a feature** | 5 | Open-hours index (Z11), Ctrl+R (Z29), browser history (B10), LFU + LRU after Redis talk (Z1) → `01b` 1.x, 12.5–12.7 |
| 4 | **Trie / string search** | 2 | Ctrl+R; trie over text + longest common substring (N11) → `01b` 1.8, 12.5, 12.8 |
| 5 | **DP (often hard)** | 2 | Unnamed hard DP with working code (Z1); "implement this library's algorithm, recursion then DP" (Z13) → `01b` 5, 12.9 |
| 6 | **Heaps / order statistics** | 1 | Dish price between the 1/3 and 2/3 percentiles: sort → two heaps (Z6) → `01b` 1.6, 12.2 |
| 7 | **Trees / BST** | 2 | BST traversal tested on skewed trees and negatives (N1); BST insert/update/delete + BST vs red-black tree (Z11) → `01b` 2.1–2.3, 12.6 |
| 8 | **Linked list** | 1–2 | Delete Fibonacci-valued nodes from a DLL, "give 4–5 approaches, then code one" (EM, Z9) → `01b` 12.8 |
| 9 | **Stack / parentheses** | 1 | Longest valid parentheses *subsequence* (Z5) → `01b` 12.8 |
| 10 | **Greedy observation** | 1 | Flipping the Matrix (Z2) → `01b` 12.8 |
| 11 | **Concurrency coding** | 2 | Locks (Z7), goroutines/channels + timeouts (N12) → `01b` 12.10 |

Round 1 (for contrast) is classic LeetCode: DP, graphs/DSU, binary search, sliding window, BST, LRU/LFU. You've passed that; keep it warm, but **shift practice time to the table above**.

### 0.3 The Round 2 problem list (52, tiered)

**Tier 1 — must do (18).** Do all of these, as clean classes, with no IDE.

| # | Problem | Code in `01b` |
|---|---|---|
| 1 | **Rate limiter: token bucket, fixed window, sliding log, sliding counter** (thread-safe classes) | 12.1 |
| 2 | LC 362 Design Hit Counter / orders in the last X minutes (+ lock) | 1.5, 12.1 |
| 3 | LC 359 Logger Rate Limiter | 12.1 |
| 4 | LC 146 LRU Cache | 1.1, 1.2 |
| 5 | LC 460 LFU Cache | 1.3 |
| 6 | LC 295 Median of a Data Stream + the 1/3–2/3 percentile variant | 1.6, 12.2 |
| 7 | LC 1345 Jump Game IV + weighted follow-up | 3.5, 12.3 |
| 8 | LC 743 Network Delay Time (Dijkstra) | 3.4 |
| 9 | LC 787 Cheapest Flights Within K Stops | 3.4 |
| 10 | LC 207 / 210 Course Schedule I/II | 3.3 |
| 11 | LC 444 Sequence Reconstruction (free substitute: LC 2392) | 12.4 |
| 12 | LC 208 Implement Trie | 1.8 |
| 13 | LC 1268 Search Suggestions + Ctrl+R | 12.5 |
| 14 | LC 1472 Design Browser History | 12.6 |
| 15 | LC 380 Insert Delete GetRandom O(1) | 1.4 |
| 16 | LC 155 Min Stack → min **and** max in O(1) | 12.6 |
| 17 | LC 450 Delete Node in BST + LC 701 Insert | 2.1, 12.6 |
| 18 | "Restaurants open now" index (crosses midnight) | 12.7 |

**Tier 2 — high (20):** LC 547 + 684 (DSU from scratch) · LC 721 Accounts Merge · LC 994 Rotting Oranges · LC 718 longest common substring · LC 1143 LCS + LC 72 Edit Distance · LC 1235 Job Scheduling (hard DP) · LC 300 LIS + 354 · LC 198/213 House Robber + LC 39 · LC 651 4 Keys Keyboard · longest valid parentheses subsequence + LC 32 + LC 1249 · Flipping the Matrix · Fibonacci-node deletion from a DLL · LC 138 + 206 + 25 · LC 94/145 iterative traversals · LC 432 All O`one · LC 981 Time-Based KV Store · LC 1386 Cinema Seats · LC 2958 + 209 → 862 · LC 215 + 703 · flatten nested JSON / paths → YAML.

**Tier 3 — stretch (14):** LC 218 Skyline · bipartite matching (Kuhn) · LC 353 Snake Game · Dijkstra with one optional edge (src + reversed from dst; LC 2203) · LC 1153 · LC 334 (3 approaches) · LC 540 / 162 / 33 · LC 410 · LC 240 · min swaps to sort a permutation · LC 2608 · LC 2812 · LC 2931 · bounded blocking queue (LC 1188; `01b` 12.10).

### 0.4 What Round 2 interviewers push on (follow-ups)

1. **Add weights → change the algorithm.** BFS becomes Dijkstra (Z2). Know *why*: BFS assumes every edge costs the same.
2. **Optimise the structure.** Sort → two heaps (Z6); scan → trie (Z29); recursion → DP (Z13); "can you optimise the variation?" (N3).
3. **Several approaches, then pick one.** An EM asked for 4–5 approaches and **praised choosing the readable one over the most optimal** (Z9). Always list 2–3 options with complexity before coding.
4. **Thread safety.** "What if two threads call this?" Have the lock answer ready for every class (mutex per structure, lock striping, or atomic counters).
5. **Fixed window → sliding window.** One candidate couldn't finish the sliding window (Z2); know both sliding variants cold.
6. **Edge-case batteries.** Skewed trees, negative values (N1); "list all the test cases" before coding (B10).
7. **DSA tied back to systems.** LFU right after Redis eviction policies (Z1); rate-limiter code in the middle of a rate-limiting design (Z2, Z5, Z7).

### 0.5 Why people failed at Round 2+ (and what to do instead)

Nobody in these reports failed for not knowing an algorithm. They failed for:
- **Shaky trade-off justification** — e.g. MySQL vs PostgreSQL, over-explained instead of admitting limited exposure (Z2, rejected at the next round). → *Say "I don't have production exposure to that trade-off; here's how I'd reason about it."*
- **Slow pickup of follow-ups** — "needed 2–3 iterations to grasp a follow-up" was called a negative signal (Z2). → *Restate the follow-up in your own words before answering.*
- **Messy code** (District, Z15) and **unfinished designs** from poor time management (Blinkit, B12). → *Time-box; get a working version first, then improve.*
- Candidates who **couldn't implement a trie** (Z29) or **missed a 2-minute puzzle** (Z9) **still passed** because their reasoning and communication were strong.

### 0.6 Seven-day Round 2 plan

| Day | DSA (2–3 h) | Design and resume (2 h) |
|---|---|---|
| 1 | Rate limiters 12.1 (all four, from memory, with locks) + hit counter + logger | `06` master Parts 0–2; your Myntra rate-limiter/warmup story |
| 2 | LRU, LFU, LC 380, min/max stack, browser history | `06` Parts 3–5 (DB, caching, Kafka); Redis eviction and hot keys |
| 3 | Graphs: Jump Game IV + weighted version, Dijkstra, LC 787, topo sort + LC 444 | `07a` Q1–Q3 (food delivery, tracking, dispatch) |
| 4 | Trie, LC 1268, Ctrl+R; open-hours index; percentile tracker | `07a` Q6–Q7 (notifications, rate limiter); `07b` Q8 payments |
| 5 | DP: LC 1143, 72, 718, 1235; recursion → memo → table talk track | `07b` Q9–Q10 (seat booking, quick commerce); `07c` Q22 (your projects as HLD) |
| 6 | Tier 2 mix: DSU, BST insert/delete, LL one-offs, parentheses subsequence, flipping matrix | `07c` LLD (menu, banner, cart, schema, open-now); SQL from `01c` |
| 7 | Timed mock: 2 problems in 60 min, typed in a plain doc, talking aloud | Mock design (45 min); `08_SD_Revision_Sheet.md`; resume "why X over Y" for every line |

---

## 1. The hiring process

### 1.1 The loop (stable from 2023 to 2026)

```
[Online Assessment]  ──►  [Round 1: Technical]  ──►  [Round 2: Technical]  ──►  [Round 3: Culture fit]
 on-campus only;           60–105 min, Google Meet    75–120 min                 20–40 min, VP / EM / co-CTO
 off-campus usually        resume + DSA + CS fund.    design + DB + resume       often onsite in Gurgaon
 skips it                  (+ small HLD/schema)       + 1–2 DSA                  offer often given on the spot
```

- **Every round is eliminatory.**
- **Results** usually come in 1–5 days. Ghosting also happens.
- **Interviewers** usually go SDE-2 or SDE-3 (Round 1), then Engineering Manager or Team Lead (Round 2), then VP of Engineering or co-CTO (Round 3).
- **Same template at the sub-brands.** Blinkit, District and Hyperpure use the same 3 rounds: PS/DSA → system design/LLD → culture fit.
- **6-month intern loops** are now identical to the SDE-1 loop.
- **Some campus drives** ran a **single** interview round and then announced results (NSUT 2023, NIT 2025, Eternal 2025).
- **The form before the interview.** Candidates sometimes fill a **Google Form** first, saying what they want to focus on (DSA or projects). If you get one, ask for **DSA + backend internships**.

### 1.2 What Round 1 actually looks like (your round)

| Phase | Time | What happens | Reported examples |
|---|---|---|---|
| Intro | 2–5 min | "Tell me about yourself" | Use the 60-second version in file 02 |
| Resume / internship deep-dive | 10–40 min | "Why X over Y?", "How would it scale?", "What would you redesign?" | Phase-2 redesign of an internship system; MySQL schema and data types; thread-safety of a project; Docker internals |
| DSA | 25–45 min | 1–3 problems, often typed in a **Google Doc with no IDE**. You write full working code and dry-run it. | Missing Number → Min Size Subarray Sum → Skyline; BST delete; LRU Cache |
| CS fundamentals | 5–20 min | DBMS (most common), OS, CN, OOP | SQL vs NoSQL, indexing, joins, process vs thread |
| Mini design (since 2024) | 0–20 min | A small HLD or schema question | Rate limiter, notification system, order/delivery schema |
| Your questions | 2–5 min | Ask about their team or work | Candidates said this was "cited as a plus" |

**Off-campus fresher Round 1 (2024–2026), the most likely shape for you:**
1. A 20–40 min resume or internship deep-dive.
2. DB questions: SQL vs NoSQL, indexing, connection pooling.
3. **One medium-to-hard DSA problem with fully working code.**
4. Often a small HLD or schema problem.

Your Myntra and Atlassian internships will likely be the **main** subject of that deep-dive. That is good news: they're backend, Go and Java, queue-based, with caching and MySQL. That is exactly the Zomato stack.

### 1.3 Online assessment (in case you get one)

**Format:** HackerRank, 3 coding questions (medium-hard) plus 13–20 MCQs, in 60–90 minutes.

**The coding questions** are Codeforces-flavoured: trees, graphs, DP, counting. Recent examples:
- Minimise tree height with K re-rootings.
- Count arrays where adjacent values differ by at most 1 (CSES "Array Description").
- Interval max-overlap point.
- Functional-graph cycle.
- Reorganise songs so no author repeats.

**The MCQs:**
- DBMS/SQL is the largest share.
- OS (LRU page replacement).
- Sorting internals: heapify state, bubble-sort swap count, which sort produces a given pass.
- BFS order and bridges.
- Output prediction.
- In 2025, a little ML.

**Partial solutions often clear the cutoff.** Attempt all three problems and take the brute-force test cases.

### 1.4 Compensation (so you're not surprised in Round 3)
- **2025–26 fresher SDE-1 (Zomato and Blinkit):** about ₹24L base + ₹5L one-time bonus (₹29L first year).
- **Location:** Gurgaon, **5 days in office**.
- Round 3 asks whether you're fine relocating and working at a fast pace ("not 9-to-5"; some teams mention 10–14-hour days).

---

## 2. What interviewers evaluate: behaviour rules for the round

These come straight from candidate feedback and rejection reasons.

1. **Write runnable code, not pseudo-code.** Name variables properly. Handle edge cases. Several rejections came from off-by-one errors and "messy" code. Assume **no IDE**: practise typing C++ in a plain doc.
2. **Go brute force → better → optimal, and say the complexity at each step.** One EM asked for about 5 approaches and then *praised the readable one over the most optimal*. Show the range of options, then justify your choice.
3. **Dry-run your code** on the sample and on one edge case **before** saying "done". Edge cases interviewers called out: empty input, single element, all negatives, zeros (in max-product), skewed trees, duplicates.
4. **Think aloud.** Break the problem into chunks and pick up hints quickly. "Took several iterations to understand the follow-up" was a stated negative.
5. **Know every resume keyword deeply.** "They will see the keywords written and ask questions about them." Candidates were rejected for not explaining their own project, REST, or HashMap internals.
6. **Design answers must cover failure paths:** retries, duplicates/idempotency, consumer down, cache down. A happy-path-only answer was marked down.
7. **Be honest about gaps.** "I don't have production experience with that trade-off, but here's how I'd reason about it" was explicitly what one interviewer wanted.
8. **Finish DSA fast.** The time you save turns into CS questions, which are easier marks.

**Your language:** say upfront **"I'll code in C++"** (or Go if you're faster in it; Zomato uses Go). Pick one and stick with it.

---

## 3. DSA patterns ranked for Zomato

Ranked by how often each pattern shows up in interview rounds (not the OA). **Tier 1 = must master.**

| Rank | Pattern | Evidence | Your target |
|---|---|---|---|
| 1 | **Design-a-data-structure** (LRU, LFU, AllOne, O(1) insert/delete/random, HashMap from scratch, hit counter, browser history) | LRU 5×, LFU 2×, about 8 other DS-design questions | Code LRU in under 10 min with no bugs. Code LFU in under 25 min. |
| 2 | **Trees / BST** (delete in BST, LCA, traversals including iterative, BT→DLL, validate BST) | BST delete 4×, plus many traversal questions | Delete-in-BST from memory, all 3 cases |
| 3 | **Graphs** (BFS/DFS, multi-source BFS, DSU, topological sort, Dijkstra, Prim's) | 10+ reports | Templates for all six |
| 4 | **Sliding window / two pointers** | 6+ reports (LC 209, 76, 1004, 438, 2958, distinct in window) | Variable-window template |
| 5 | **DP** (1D, house robber, LCS / strings, grid like maximal square, LIS, max-product) | 8+ reports, "often hard" | Standard 1D/2D DP; LCS; LIS in n log n |
| 6 | **Intervals** (merge, max overlap, meeting rooms, "restaurants open now") | 5+ reports | Sort + sweep; difference array / line sweep |
| 7 | **Heaps** (Kth largest, median of stream, two heaps, greedy with PQ, reorganise string) | 5+ reports | Median of stream from memory |
| 8 | **Binary search** (rotated array, peak element, binary search on the answer, painter's partition) | 4+ reports | Lower-bound template plus BS-on-answer template |
| 9 | **Arrays / hashing / math** (Missing Number, Majority Element II, prefix sums, Kadane with indices, cycle sort, Fisher-Yates shuffle) | Many one-offs | Common warm-ups; must be quick |
| 10 | **Linked list** (reverse, reverse in k-group, palindrome list) | 4 reports | Reverse k-group from memory |
| 11 | **Trie** (Ctrl+R reverse search, text search, autocomplete) | 2–3 reports | Insert/search/startsWith |
| 12 | **Stack / monotonic stack** (Skyline, largest rectangle, valid parentheses variants) | 3 reports | Next greater element template |

> Candidates generally say DSA is **mostly LeetCode medium with an observation twist**, and the **Striver SDE Sheet level is "enough"**. Hards do appear (Skyline, Jump Game IV, hard DP), so be comfortable with a few.

---

## 4. Expected DSA problems: the actual list

**★ = reported at Zomato/Blinkit/District interviews. Do every ★ problem.** Non-starred items are on LeetCode's Zomato tag or are close variants.

Each entry gives the approach, then the complexity.

### 4.1 Design-a-data-structure (highest priority)

| # | Problem | Approach | Time / Space |
|---|---|---|---|
| ★ | **LRU Cache (LC 146)**, 5×, sometimes "modified" | Hash map from key to list iterator, plus a doubly linked list with the most recently used at the front. A `get` moves the node to the front. A `put` on a full cache evicts the back. | O(1) per operation / O(cap) |
| ★ | **LFU Cache (LC 460)**, 2× | Map key → {value, freq, iterator}. Map freq → list of keys (LRU within each frequency). Track `minFreq`. On access, move the key from list f to list f+1; if list f is now empty and f == minFreq, increment minFreq. On insert, minFreq = 1. | O(1) / O(cap) |
| ★ | **AllOne data structure (LC 432)** (after a "most frequent string" warm-up) | Doubly linked list of buckets ordered by count, each bucket holding a set of keys, plus a map key → bucket. | O(1) |
| ★ | **Insert Delete GetRandom O(1) (LC 380)** | Vector plus map value → index. To delete, swap with the last element and pop. | O(1) |
| ★ | **Design HashMap (LC 706)**, from scratch with a string hash, collisions and a mutex | Array of buckets with chaining; polynomial rolling hash; resize at load factor 0.75; `std::mutex` / `sync.RWMutex` for thread safety | O(1) average |
| ★ | **Design Hit Counter / orders in the last X minutes (≈ LC 362)** | Circular array of X buckets, each holding (timestamp, count). On a hit, reset the bucket if its timestamp is stale. Sum the valid buckets. Add a lock for concurrency. | O(1) hit, O(X) query |
| ★ | **Browser History (LC 1472)** | Vector plus a current index; `visit` truncates everything forward | O(1) |
| ★ | **Median of a data stream (LC 295)** | Max-heap for the lower half, min-heap for the upper half, kept balanced | O(log n) add, O(1) median |
| ★ | **Flatten nested JSON to dotted keys** | DFS over the map, carrying the prefix `a.b.c` | O(total keys) |
| ★ | **Shuffle with equal probability (Fisher-Yates, LC 384)**, 2× (Spotify shuffle) | For i from n−1 down to 1, swap a[i] with a[rand(0..i)] | O(n) |
| | Min Stack (LC 155) | Store pairs (value, current min) | O(1) |
| | Time-Based Key-Value Store (LC 981) | map key → vector of (time, value); binary search on time | O(log n) get |

### 4.2 Trees / BST

| # | Problem | Approach | Complexity |
|---|---|---|---|
| ★ | **Delete Node in a BST (LC 450)**, 4×, plus variants | Find the node. Case 1, leaf: return null. Case 2, one child: return that child. Case 3, two children: copy the inorder successor (minimum of the right subtree) into the node, then delete the successor from the right subtree. | O(h) |
| ★ | **Lowest Common Ancestor (LC 236 / 235)** | Binary tree: recurse, and if both sides return non-null, the current node is the LCA. BST: walk down using values. | O(n) / O(h) |
| ★ | **Iterative postorder (LC 145)** | Two stacks, or one stack plus a `lastVisited` pointer | O(n) |
| ★ | **Binary tree → doubly linked list** | Inorder traversal with a `prev` pointer | O(n) |
| ★ | BST traversal with **skewed trees and negative values** | Recursion depth is O(n) when skewed, so mention the iterative version | O(n) |
| | Validate BST (LC 98) | Pass min/max bounds down, using `long` | O(n) |
| | Level order / zigzag (LC 102/103) | BFS with a queue | O(n) |
| | Diameter of a binary tree (LC 543) | Height recursion; the answer is the maximum of left + right | O(n) |
| | Kth smallest in a BST (LC 230) | Inorder traversal, stopping at k | O(h + k) |
| | Serialize / deserialize (LC 297) | Preorder with null markers | O(n) |
| | Maximum path sum (LC 124) | Return the max single-branch gain; update the global answer with left + right + node | O(n) |
| ★ (concept) | **B+ tree:** insert complexity, effect of order k, why not a sorted array | See file 01c (indexing) | |

### 4.3 Graphs

| # | Problem | Approach | Complexity |
|---|---|---|---|
| ★ | **Rotting Oranges (LC 994)** | Multi-source BFS from all rotten cells; count levels | O(mn) |
| ★ | **Distance from nearest guard / 01 Matrix (LC 542, 286)** | Multi-source BFS from every source at once | O(mn) |
| ★ | **Number of Provinces / friend circles (LC 547)** + **count redundant edges with DSU** | DSU with path compression and union by rank. A union where `find(a) == find(b)` is a redundant edge (LC 684). | O(E α(V)) |
| ★ | **Connected components with "same component?" queries; largest infection cluster (online union-find)** | DSU, tracking component sizes | ≈ O(1) per query |
| ★ | **Course Schedule (LC 207 / 210)** | Kahn's algorithm (BFS on in-degrees). If fewer than V nodes are processed, there's a cycle. | O(V + E) |
| ★ | **Jump Game IV (LC 1345)**, with a follow-up adding a cost per step → Dijkstra | BFS over neighbours i−1, i+1 and every index with the same value; **clear that value's list after using it** | O(n) |
| ★ | **Constrained shortest path (Dijkstra or Bellman-Ford)** | Dijkstra with a min-heap. With a "at most K stops" constraint, use Bellman-Ford limited to K rounds (LC 787). | O(E log V) |
| ★ | **MST / Prim's variant** | Prim's with a priority queue, or Kruskal's with DSU | O(E log V) |
| | Number of Islands (LC 200), on the Zomato tag | DFS/BFS flood fill | O(mn) |
| | Snakes and Ladders (LC 909), on the tag | BFS over board positions | O(n²) |
| | Shortest Cycle in a Graph (LC 2608), on the tag (OA) | BFS from every node; on a non-tree edge (u, v), the cycle length is dist[u] + dist[v] + 1 | O(V·E) |
| | Find the Safest Path in a Grid (LC 2812), on the tag | Multi-source BFS for distances, then binary search on the answer or a max-bottleneck Dijkstra | O(n² log n) |
| | Network Delay Time (LC 743) | Dijkstra | O(E log V) |

**Zomato flavour:** expect stories like "delivery partners on a grid", "riders and restaurants", "nearest guard/store" (multi-source BFS), "orders depend on each other" (topological sort), or "cheapest route" (Dijkstra).

### 4.4 Sliding window / two pointers

| # | Problem | Approach | Complexity |
|---|---|---|---|
| ★ | **Minimum Size Subarray Sum (LC 209)**, 2×. **Follow-up: with negatives → LC 862** | Positive numbers: a variable window, shrinking while sum ≥ target. With negatives: prefix sums plus a **monotonic deque** of increasing prefix values. | O(n) |
| ★ | **Minimum Window Substring (LC 76)** | Count of needed characters plus a `missing` counter; expand right, shrink left | O(n) |
| ★ | **Max Consecutive Ones III (LC 1004)** | Window holding at most k zeros | O(n) |
| ★ | **Find All Anagrams (LC 438)** | Fixed window with a 26-entry count array | O(n) |
| ★ | **Count distinct elements in every window of size k** | Hash map of counts; add the right element, remove the left | O(n) |
| ★ | **Longest subarray with each element ≤ k times (LC 2958)** | Variable window with frequency counts | O(n) |
| | Longest Substring Without Repeating Characters (LC 3), on the tag | Last-seen index of each character | O(n) |
| | Container With Most Water (LC 11), on the tag | Two pointers; move the shorter side | O(n) |
| | 3Sum (LC 15), on the tag | Sort, then two pointers; skip duplicates | O(n²) |
| | Subarray Sum Equals K (LC 560) | Prefix sums plus a hash map of counts (handles negatives) | O(n) |

### 4.5 Dynamic programming

| # | Problem | Approach | Complexity |
|---|---|---|---|
| ★ | **House Robber I & II (LC 198 / 213)** | `dp[i] = max(dp[i-1], dp[i-2] + a[i])`. II (circular): run I on `[0..n-2]` and on `[1..n-1]`, take the max. | O(n), O(1) space |
| ★ | **4-Keys Keyboard (LC 651)** | `dp[i] = max(dp[i-1] + 1, max over j of dp[j] * (i-j-1))`: select-all + copy at step j, then paste the rest | O(n²) |
| ★ | **Maximal Square (LC 221)**, on the tag | `dp[i][j] = 1 + min(top, left, diag)` if the cell is '1' | O(mn) |
| ★ | **LCS (LC 1143)** + the "unique characters → n log n" optimisation | Classic 2D DP. If one string has all-unique characters, map each to its index in that string; LCS becomes LIS of the mapped sequence, solved in O(n log n). | O(nm), or O(n log n) |
| ★ | **Maximum Product Subarray (LC 152)** | Track both the max and the min ending at i; **swap them on a negative**; reset on zero | O(n) |
| ★ | **LIS (LC 300)** + variants (Russian Doll Envelopes, LC 354, on the tag) | `tails` array plus `lower_bound`. For envelopes: sort by width ascending, height **descending**, then run LIS on heights. | O(n log n) |
| ★ | **Sell from either end, the k-th sale earns k × price ("max profit wines")** | Interval DP: `dp[l][r]` where year = n − (r − l), take the best of the two ends | O(n²) |
| ★ | **Kadane, returning the subarray itself (LC 53)** | Track the start index whenever you reset | O(n) |
| ★ | **DP on strings** (unspecified): practise Edit Distance (LC 72), Longest Palindromic Subsequence (LC 516), Word Break (LC 139) | | O(nm) |
| ★ | **Combination Sum (LC 39) blended with House Robber** | Backtracking or unbounded-knapsack DP | |
| | Unique Paths (LC 62) (OA) | Grid DP, or combinations C(m+n−2, m−1) | O(mn) |
| | Coin Change (LC 322), Partition Equal Subset Sum (LC 416) | Standard knapsack | O(n·amount) |
| | Number of Ways to Divide a Long Corridor (LC 2147), on the tag | Count seats; multiply the gaps between consecutive seat pairs | O(n) |

### 4.6 Intervals

| # | Problem | Approach | Complexity |
|---|---|---|---|
| ★ | **Merge Intervals (LC 56)** | Sort by start; extend the last interval or push a new one. One District follow-up asked to improve this from O(n log n) to O(n) using a map or bucketing when the range is small. | O(n log n) |
| ★ | **Maximum overlap point / first point of maximum overlap** (OA 2×) | Line sweep: events (start, +1) and (end, −1), sorted with end before start at ties if intervals are half-open. Or a difference array when the coordinates are small. | O(n log n) |
| ★ | **Meeting scheduler: common free slot (LC 1229)** | Sort both calendars; two pointers over the intersection | O(n log n) |
| ★ | **"Which restaurants are open now?"** (open/close times, LLD + DS) | For a few queries: an interval list per restaurant. For many time queries: bucket by minute-of-day, or an **interval tree / segment tree**. Handle **past-midnight** closings by splitting [22:00, 02:00] into [22:00, 24:00] + [00:00, 02:00]. | |
| | Meeting Rooms II (LC 253) | Min-heap of end times, or sort starts and ends separately | O(n log n) |
| | Insert Interval (LC 57), Non-overlapping Intervals (LC 435) | Sweep; greedy by end time | O(n) / O(n log n) |

### 4.7 Heaps / greedy

| # | Problem | Approach | Complexity |
|---|---|---|---|
| ★ | **Kth Largest (LC 215)** | Min-heap of size k (O(n log k)), or quickselect (O(n) average) | |
| ★ | **"Suggest a dish price between the 1/3 and 2/3 percentiles"** | Sort and index; for a stream, use two heaps (as in median of stream) | |
| ★ | **Reorganise songs so no author repeats (LC 767 / 621 / 358)** | Max-heap by count; take the top two each time, or hold back the previous one | O(n log k) |
| ★ | **Maximum transactions keeping the running sum ≥ 0** (OA) | Take every transaction greedily, pushing negatives into a min-heap. When the sum goes below 0, drop the most negative one taken so far. | O(n log n) |
| ★ | **Maximum Spending After Buying Items (LC 2931)**, top of the tag | Always buy the cheapest available item on the earliest day (min-heap over the shop rows) | O(mn log m) |
| | Task Scheduler (LC 621), Top K Frequent (LC 347) | Heap or bucket sort | |
| ★ | **Cinema Seat Allocation (LC 1386)**, **airplane seat greedy** | Bitmask per row over the reserved seats; check the 3 possible 4-seat blocks | O(reserved) |

### 4.8 Binary search

| # | Problem | Approach | Complexity |
|---|---|---|---|
| ★ | **Search in Rotated Sorted Array (LC 33)** | One half is always sorted; check whether the target lies in it | O(log n) |
| ★ | **Find Peak Element (LC 162)** | If `a[mid] < a[mid+1]`, go right; otherwise go left | O(log n) |
| ★ | **Binary search on the answer: Painter's Partition / Split Array Largest Sum (LC 410)** / book allocation | Search the answer between max(a) and sum(a); greedily check whether k painters suffice | O(n log sum) |
| | Single Element in a Sorted Array (LC 540), on the tag | Pair-index parity trick | O(log n) |
| | Koko Eating Bananas (LC 875), Search a 2D Matrix II (LC 240, on the tag) | BS on the answer; staircase search from the top-right | |

### 4.9 Arrays / hashing / math / bits (warm-ups: must be fast)

| # | Problem | Approach |
|---|---|---|
| ★ | **Missing Number (LC 268)** | XOR of 0..n with all elements, or n(n+1)/2 − sum (mention overflow) |
| ★ | **Majority Element II (LC 229)** | Extended Boyer-Moore with 2 candidates, then a verification pass |
| ★ | **Minimum swaps to sort a permutation 1..n** | Cycle decomposition: the answer is the sum over cycles of (cycle length − 1) |
| ★ | **Increasing Triplet Subsequence (LC 334)**, "multiple approaches" | `first` and `second` minimums in O(n) / O(1); also prefix-min plus suffix-max arrays |
| ★ | **Contiguous Array (LC 525)** | Treat 0 as −1; use a prefix sum plus a map of the first index each sum appeared |
| ★ | **Global and Local Inversions (LC 775)** | Check that `|a[i] − i| ≤ 1` for all i |
| ★ | **Single Number (LC 136)** / Single Number III (LC 260, on the tag) | XOR; for III, split the numbers by the lowest set bit of the total XOR |
| ★ | **Factorial Trailing Zeroes (LC 172)** | Sum of n/5 + n/25 + … |
| ★ | **Find and Replace Pattern (LC 890)**, Isomorphic Strings (LC 205, on the tag) | Two-way character maps |
| ★ | **"Remove one element to make the array sorted"** | Find the first descent and try removing either side of it (as in LC 665) |
| | First Missing Positive (LC 41), on the tag | Cyclic placement: put each value x at index x−1 |
| | Move Zeroes (LC 283), Rotate Array, Next Permutation (LC 31) | |

### 4.10 Linked list

| # | Problem | Approach |
|---|---|---|
| ★ | **Reverse Nodes in k-Group (LC 25)** | Check that k nodes remain; reverse them; connect `prevGroupTail` to the new head |
| ★ | **Reverse Linked List, iterative AND recursive (LC 206)** | Both versions from memory |
| ★ | **Palindrome Linked List (LC 234)** | Slow/fast pointers to find the middle, reverse the second half, compare, then restore |
| ★ | **Delete nodes with Fibonacci values from a DLL** | Precompute Fibonacci numbers up to the max value in a hash set, then unlink matching nodes. The interviewer wanted about 5 approaches. |
| | Merge K Sorted Lists (LC 23), Detect Cycle II (LC 142), LRU uses a DLL | |

### 4.11 Stack / Trie / strings

| # | Problem | Approach |
|---|---|---|
| ★ | **The Skyline Problem (LC 218)**, high on the tag; asked in R1 2025 | Events (x, −h) for starts and (x, h) for ends; a multiset of heights; record a point whenever the max changes |
| ★ | **Longest valid parentheses (LC 32) / the subsequence version** | Substring: a stack of indices. **Subsequence:** greedily count matched pairs, giving 2 × pairs |
| | Largest Rectangle in Histogram (LC 84) (OA variant) | Monotonic stack |
| ★ | **Trie: Ctrl+R reverse search / text search / autocomplete** | Trie with `insert`, `startsWith`; store the most-recent command index at each node for Ctrl+R |
| | Valid Parentheses (LC 20), Decode String (LC 394), Daily Temperatures (LC 739) | |

---

## 5. The Zomato-specific "story problem" skill

Zomato interviewers like to wrap standard problems in food-delivery stories. Translate the story into the underlying pattern before coding:

| Story phrasing | Underlying pattern |
|---|---|
| "Delivery boy has time T, maximise houses/orders visited" | DP / knapsack-like, or greedy plus a heap |
| "Nearest delivery partner / nearest store / nearest guard" | Multi-source BFS (grid), Dijkstra (weighted), or a k-d tree / geohash (design) |
| "Orders placed in the last X minutes / per-restaurant rate" | Sliding window, circular buckets (hit counter) |
| "Top K restaurants / dishes / riders" | Heap, or bucket sort |
| "Restaurants open at time t" | Intervals, line sweep, segment tree |
| "Recently viewed restaurants / cache of menus" | LRU |
| "Suggest a price between percentiles / running median rating" | Two heaps |
| "Dependencies between tasks / kitchen steps" | Topological sort |
| "Group users or riders into clusters" | DSU |
| "Shuffle the playlist / random restaurant suggestion" | Fisher-Yates, reservoir sampling |
| "Search-as-you-type for dishes" | Trie |

---

## 6. CS fundamentals, design and SQL (summary; full answers in 01c)

What has been asked, in order of frequency:

1. **DBMS:**
   - SQL vs NoSQL (12+ reports).
   - **Indexing internals** (10+): B+ tree, clustered vs non-clustered, composite index and the left-prefix rule, how `WHERE a=10 AND b=20` runs with two single-column indexes vs one composite index.
   - ACID and isolation levels, CAP, normalisation, sharding, replication, connection pooling.
   - MySQL vs Postgres (one rejection came from failing to justify this).
2. **SQL queries:**
   - Nth or second-highest salary (3×).
   - Anti-join (restaurants with no cuisine).
   - GROUP BY/HAVING.
   - Top-K per group.
   - 3-table joins.
   - Joins vs subqueries.
3. **OS:**
   - Process vs thread (6+).
   - Mutex vs semaphore; deadlock (the 4 conditions and their prevention).
   - Context switch; concurrency vs parallelism; scheduling.
   - `fork()` output counting.
   - **Merge sort on 8 cores vs 1** (2×).
   - Why Redis is single-threaded.
4. **CN:**
   - **"What happens when you type zomato.com"** (7+).
   - TCP vs UDP, the TCP handshake, DNS, CDN, OSI.
   - WebSockets.
   - JWT; authentication vs authorisation; hashing vs encryption.
   - CORS.
5. **OOP:**
   - The 4 pillars, virtual functions and dynamic dispatch.
   - Constructor/destructor order.
   - **HashMap internals**; `map` vs `unordered_map` (one candidate lost points mixing them up).
   - `push_back` vs `emplace_back`.
6. **HLD (yes, even for freshers):**
   - **Rate limiter (6×)**.
   - **Notification system (5×)**.
   - **Food ordering / order tracking** (APIs, schema, Kafka, idempotency).
   - **BookMyShow seat locking (3×)**.
   - Spotify (3×).
   - Image upload (S3 pre-signed URLs).
   - Load balancer / consistent hashing.
7. **LLD / schema:**
   - Delivery partner + order schema.
   - Menu with customisations (sizes, crusts, toppings).
   - Banner shown X times in Y hours.
   - Attendance system.
   - The schema of **your own project**.

---

## 7. Resume, project and behavioural questions (summary)

**Resume:**
- Everything in files 02, 03 and 04.
- Expect **"why X over Y?"** for every tech choice: Go vs Java, SQS vs Kafka, MySQL vs NoSQL, FastAPI vs Flask, XGBoost vs a neural network.
- Expect **"how would it scale 10×/100×?"**.
- Expect **"what would you redesign in phase 2?"**.

**Behavioural (mostly Round 3, sometimes the end of Round 1):**
- Why Zomato? Why not start your own startup? (asked 3× in one loop)
- Short- and long-term goals.
- A challenging problem you solved.
- A disagreement; ownership; prioritisation.
- Are you OK with Gurgaon, 5 days in office, and a fast pace?
- What do you know about Zomato, Blinkit, District, Hyperpure and Nugget? What challenges do they face?

**Zomato knowledge to drop naturally:**
- **The company:** Zomato Ltd was renamed **Eternal Ltd** in 2025.
- **Its four businesses:**
  - **Zomato:** food delivery.
  - **Blinkit:** quick commerce, 10-minute delivery from dark stores.
  - **District:** going out (dining, movies, events).
  - **Hyperpure:** B2B supplies for restaurants.
- **Nugget** is its AI customer-support platform.
- **Engineering problems you can mention:**
  - Real-time rider assignment and dispatch, and ETA prediction.
  - Search and ranking.
  - Surge and peak load (IPL nights, New Year's Eve).
  - Dark-store inventory sync at Blinkit.
  - Cache-heavy menus and listings.
  - Kafka-based event pipelines.
- **The stack (from public engineering posts):** Go and Java/Kotlin services, MySQL/Aurora, Redis, Kafka, Elasticsearch, Kubernetes on AWS. Zomato has published engineering blogs on moving to Go and on large-scale caching. Check the **Zomato engineering blog** before the interview and be able to cite one post.

**Good questions to ask the interviewer:**
- "What does the first 3 months look like for an SDE-1 on your team?"
- "What's the hardest scaling problem your team has handled recently, e.g. NYE or IPL peaks?"
- "How do teams split between Go and Java services?"

---

## 8. Preparation plan

Adjust the length to your interview date. This assumes about **10 focused days**. If you have fewer, do days 1–5 and 9–10.

| Day | DSA (3–4 hrs) | Fundamentals / resume (2 hrs) |
|---|---|---|
| 1 | Design: **LRU, LFU**, Insert-Delete-GetRandom, Min Stack. Code all from memory. | Resume intro out loud ×3; Myntra deep-dive |
| 2 | Trees: **BST delete**, LCA, iterative traversals, BT→DLL, validate BST, max path sum | DBMS: indexing (B+ tree, composite, clustered), ACID, isolation |
| 3 | Graphs I: BFS/DFS, **multi-source BFS** (Rotting Oranges, 01 Matrix), islands, **DSU** (provinces, redundant connection) | SQL: 15 queries from 01c, written by hand |
| 4 | Graphs II: **topological sort** (Course Schedule I/II), **Dijkstra**, Prim's/Kruskal, Jump Game IV, Cheapest Flights K stops | Atlassian deep-dive; SQS vs Kafka |
| 5 | Sliding window: LC 209 (+862), 76, 1004, 438, 3, 2958, distinct in window, 560 | OS: process/thread, mutex/semaphore, deadlock, scheduling, paging, `fork` |
| 6 | DP I: House Robber I/II, Kadane with indices, max product, LIS (n log n), Russian dolls, coin change | CN: the "type zomato.com" walkthrough, TCP/UDP, DNS, HTTP/HTTPS, WebSockets, JWT |
| 7 | DP II: LCS (+ n log n trick), edit distance, maximal square, 4-keys keyboard, wines, word break | Projects: TraceX + SupplySenseAI deep-dives (file 03) |
| 8 | Intervals + heaps: merge, meeting rooms II, max overlap, median of stream, Kth largest, reorganise string, task scheduler | HLD: **rate limiter**, **notification system** (01c) |
| 9 | Binary search + linked list + stack: rotated array, peak, LC 410, reverse k-group, palindrome list, Skyline, histogram, Trie | HLD: **food ordering / order tracking**, BookMyShow seat locking; LLD schemas |
| 10 | **Mock round:** 2 random ★ problems, 45 min, typed in a Google Doc, talking aloud. Then redo the ones you got wrong. | Behavioural stories; why Zomato; questions to ask |

**Daily habit:** one ★ problem typed in a plain Google Doc, with no autocomplete and no running, and a dry-run on paper.

### Interview-day checklist
- [ ] Stable internet, charged laptop, quiet room, a Google Doc and a plain editor open, pen and paper.
- [ ] Before coding: restate the problem, ask about constraints (n, value range, negatives, duplicates, empty input), and confirm the input/output format.
- [ ] Say the brute force with its complexity → the optimised approach → **get a nod** → then code.
- [ ] Code cleanly: helper functions, meaningful names, no global mess.
- [ ] Dry-run on the sample plus 1–2 edge cases. State the final time and space complexity.
- [ ] For resume questions: answer in 30–60 seconds, then stop and let them dig.
- [ ] If stuck: say what you're thinking, try small examples, and ask for a hint rather than going silent.
- [ ] End with 1–2 real questions for the interviewer.

---

## 9. Sources

**LeetCode Discuss (Zomato / Eternal, fresher / intern / SDE-1)**
- 8434690 (SDE-1, Aug 2026): hard DP, LFU, Redis, HLD, VP round.
- 8377019 (SDE-1, 2026): schema, 4-keys keyboard, rate limiter, Jump Game IV, notification system.
- 7609011 (SWE-1, Feb 2026): payments HLD, Kafka vs SQS/SNS.
- 7407389 (intern + FTE, Dec 2025): ordering system design, rate limiter, parentheses.
- 7206483 (SDE-1, Aug 2025): min swaps, banner schema, menu LLD, two heaps.
- 7046636 (NIT on-campus, 2025): B+ tree, BST delete variant, graph + DP.
- 6997937 (off-campus SDE-1, 2025): LC 209, BST traversal.
- 6994052 (Eternal on-campus, 2025): Missing Number, LC 209, Skyline.
- 6993844, 6986941, 7053334, 7377966, 8419489, 8424651, 8401062: OA formats and problems, 2025–26.
- 6493240 (off-campus, 2025): PQ greedy + Prim's, rate limiter, order buckets.
- 6023375 (backend intern, Nov 2024): indexing, Fibonacci DLL.
- 6164662 (SDE intern, Dec 2024): DP, Dijkstra, topological sort.
- 5947992, 5763557, 4832993, 5007288, 4473000, 4068294, 3768540, 3760268, 3384363, 2251731, 1806291, 1743607, 1798957, 2054815, 1622710, 1554703.

**Blinkit / District / Nugget**
- 8331559, 8514070, 8515131, 5156809, 6207331, 4630366, 7617100, 8550229, 8559623, 6647834, 8540075.

**Other sources**
- GeeksforGeeks: Zomato × Blinkit SDE-I on-campus 2024; Zomato SDE-1 on-campus 2021 and 2023; Zomato interview experience (SDE).
- Medium: @shivambhadani_, @utkarshagarwal101, @shagunydv, @Ayush-Tibrewal.
- IIT Bhubaneswar placement notes (Zomato, 2023).
- Jointaro (2021, 2025).
- Glassdoor (search snippets only).
- LeetCode company-tag scrapes: github.com/liquidslr/leetcode-company-wise-problems and github.com/snehasishroy/leetcode-companywise-interview-questions.

> Full URLs: LeetCode posts are at `https://leetcode.com/discuss/post/<id>/`. A caveat repeated by candidates: Zomato interviews are "very unpredictable" and depend on the interviewer's own expertise. Cover patterns, not just this list.
