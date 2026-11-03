# Swiggy ASDE / SDE-1 (Backend): The Process and Every Question Asked Before

> **Who this is for:** Vaideeka, preparing for Swiggy's Round 1 for ASDE or SDE-1 backend.
> **What it does:** Part 1 explains how Swiggy hires, in plain words. Part 2 lists every question that real candidates reported, most frequent first, with a short answer to each. Part 3 turns their mistakes into a do/don't list. Part 4 is the source list.
> **Reading time:** about 60–75 minutes. Don't memorise it. Read it once to see the whole picture, then go to the deeper files:
> - `02_DSA_From_Zero.md`: full solutions for the DSA problems marked ✅ below
> - `03_Machine_Coding_and_LLD_From_Zero.md`: machine coding and low-level design
> - `04_CS_Fundamentals_From_Zero.md`: OS, DBMS, SQL, networks, security
> - `05_Java_and_Spring_From_Zero.md`: Java, JVM and Spring Boot questions
> - `06_Resume_Projects_Behavioural_for_Swiggy.md`: your resume, "Why Swiggy?", stories

**How to read the citations.** A number in square brackets, like [17], is a real interview report. The full list with links is in Part 4. "OLD" means the report is from before 2023. Older reports still show which topics Swiggy likes, but the format may have changed since.

**Where this comes from.** About 38 separate candidate reports from 2021 to 2026, collected in November 2026 from GeeksforGeeks, LeetCode Discuss (through full-text mirrors), Jointaro, Glassdoor, AmbitionBox, Naukri Code360 and a few blogs. If no report says something, this file says "unknown". Nothing here is made up.

---

## Part 1: How Swiggy hires

### 1.1 First, the words you'll see

| Word | What it means in plain English |
|---|---|
| **ASDE** | *Associate* Software Development Engineer. The fresher role (0–1 years of experience). Its questions are noticeably easier than SDE-1's. |
| **SDE-1** | Software Development Engineer 1. Officially the entry level, but many SDE-1 candidates have 1–2 years of experience, so the bar is a bit higher. Freshers do apply too. |
| **OA** | Online Assessment. A timed test on a website before any interview, scored automatically by hidden test cases. |
| **HackerRank** | The website Swiggy uses for its OAs. You write code in the browser and press "Submit". |
| **DSA** | Data Structures and Algorithms. Problems like "find the first and last position of 5 in this sorted array". |
| **LeetCode (LC)** | A practice website. "LC 20" means problem number 20 there. Interview reports use these numbers. |
| **LLD** | Low-Level Design. Deciding which classes, objects, methods and database tables a feature needs. Think of it as planning the rooms of one house. |
| **HLD** | High-Level Design. Planning the whole system: servers, databases, caches, queues. Planning a whole town rather than one house. |
| **Machine coding** | A long round (60 min to 3 hours) where you build a small *working* program, such as a Snake & Ladder game or a Splitwise clone, on your own laptop. It is LLD plus real, running code. |
| **HM round** | Hiring Manager round. Your future manager talks about your resume and projects, and often asks one DSA or design question. |
| **Third-party interviewer** | Swiggy sometimes outsources early rounds to an interview company (reports name InterviewVector). The interviewer is not a Swiggy employee, but follows Swiggy's rubric. [17][18][20][29] |

### 1.2 The big picture

```
Apply (referral / LinkedIn form / HR mail / campus)
   │
   ▼
Online Assessment on HackerRank  ──►  (ASDE backend only, often) Take-home "Magical Arena" game
   │                                              │
   ▼                                              ▼
Round 1  (DSA, or DSA + LLD, or LLD discussion, or machine coding)
   │
   ▼
Round 2  (LLD / machine coding, or a second DSA round)
   │
   ▼
Hiring Manager round  (resume deep dive + a DSA or design question + CS fundamentals)
   │
   ▼
HR round  → offer
```

The whole process took **about 1 week [14] to 3 weeks [17]**.

A real-life picture: think of joining a restaurant kitchen. The OA is a written test on recipes. The take-home is cooking one dish at home and bringing it in. Round 1 is cooking in front of a chef. The LLD round is designing how the kitchen should be organised. The HM round is the head chef asking about your past jobs.

### 1.3 The Online Assessment (OA)

**Where:** HackerRank, in every report that names a platform [1][2][3][4][5][7][14][20][21][22][27][33][41][45].

**ASDE OA (your most likely one).** It is a mix, and the DSA part is easy to medium:

| Report | What was in it |
|---|---|
| ASDE, Dec 2024, off-campus [20] | "Two very simple DSA questions, two easy-to-medium SQL queries (simple joins), and a few easy CS fundamentals MCQs." |
| ASDE, 2022 passout [22] | 1 SQL question on joins, 1 medium-hard coding question (array + binary search), a few basic MCQs. Done in 30–40 min. |
| ASDE Backend, June 2024 [33] | "HackerRank test with 4 questions testing your backend knowledge." |
| ASDE Backend, Mar 2026 [16] | 3 DSA questions, "easy to medium". |
| ASDE campus, Nov 2025 [12] | 24 MCQs (iOS, Android, HTML, CSS) plus 2 coding: one easy-medium string problem, one medium-hard 2D-grid DP. |
| Campus, Dec 2025 [35] | "DSA questions and CS fundamental questions." |

**SDE-1 OA.** Usually **3 coding questions in 75–90 minutes**, often with **one SQL question** too [2][3][4][5][14][17][18][23]. The mix ranges from easy/medium/hard to 3 mediums or 2 mediums + 1 hard [17].

**Unusual extras:**
- One Dec 2024 OA had "call an API endpoint and process the response" [14]. That means writing code that fetches data from a URL (usually JSON) and calculates something from it.
- One OA had a "hard SQL query" [14].

**The Java/Go rule (important for 2026).** Recent reports say the OA and DSA rounds allowed **only Java or Golang** [16][17]. One candidate got a Go/Python OA for a Java role [18]. **Be ready to write DSA in Java (or Go).** If you've only done DSA in C++, start practising in Java now. That's why the code in `02` is in Java.

**Cut-offs.** No source gives an official cut-off score. On campus in 2021, 295 students sat the test and 30 were shortlisted, mostly those who solved everything [3]. Aim to pass *all* test cases on all questions.

### 1.4 The ASDE take-home: "Magical Arena"

Three different ASDE backend cycles mention the same take-home after the OA, so expect it if you're on the ASDE track [20][22][33].

- **What it is:** "design a playing arena in which two players can fight based on the given logic" [22], a "simple attack/defend game" [20], also called the "Magical Game Arena Assignment" [33].
- **Languages allowed:** Java, NodeJS or Go [20][22][33].
- **Time:** one candidate finished within a day, using Java and Maven [22].
- **The rules:** no first-hand report gives Swiggy's exact text. Public candidate solutions on GitHub [44] describe each player having **health, strength and attack** values, with dice rolls deciding damage on each turn. Treat that as community material and read the real brief carefully when you get it.

**What they want to see** (from how Swiggy grades machine coding; see 1.6): clean classes (`Player`, `Arena`, `Dice`), a program that runs, unit tests, a README explaining how to run it, and code that's easy to extend. `03_Machine_Coding_and_LLD_From_Zero.md` shows how to build one.

### 1.5 Round 1: the four versions people got

Round 1 isn't the same for everyone. These are the versions that appear in recent reports.

**Version A: pure DSA (most common for SDE-1).** Two medium problems in about 60 minutes [17][24][31]. Older reports had 3–4 questions [1][2][3][4][8]. The code usually has to run.
- *2026 variant [17]:* run by a third-party interviewer. About 30 min of coding where the code had to run, then **about 25 min of rapid-fire Java questions (20–25 of them)** on OOP, JVM/JDK, garbage collection, Collections and multithreading. See `05`.

**Version B: easy DSA plus basic LLD (seen for ASDE in 2026).** ASDE Backend, Mar 2026 [16]: "first and last occurrence of an element in a sorted array" and "design a stack using an array", in Java or Go. The interviewer asked for classes and interfaces ("basic LLD concepts"). So even a simple stack had to be written as a proper class with an interface.

**Version C: LLD discussion (seen for ASDE in Dec 2024).** ASDE, Dec 2024 [20], third-party interviewer: design **follow/unfollow for a social media app**. Cover the requirements, the database schema (tables), the API endpoints and which HTTP verbs to use (GET/POST/DELETE). It was a conversation, and code wasn't the priority.

**Version D: one big technical round (seen on campus, Nov 2025).** ASDE campus [12]: a resume project, 2 easy-medium DSA questions (Valid Parentheses and an array problem), **in-depth OS** (deadlock, semaphores, threads), security basics (hashing vs encryption, salting, JWT), OOP, and explaining some Java multithreading code.

**Version E: machine coding as the first live round (older, or for some teams).**
- Snake & Ladder in 120 min [4] (OLD 2021).
- Calendar app in 3 hours [7] (OLD).
- Multilevel parking lot [27] (OLD).
- Splitwise in 120 min, described as "crucial for selection" [39] (OLD 2022).
- Since 2024, machine coding/LLD usually comes **after** the DSA round [17][18][19][24][31].

### 1.6 What Round 1 will most likely look like for you

You're a fresher with backend internships (Myntra, Atlassian). Based on the reports, expect one of these.

| Likely format | Chance | How you'll know | What to prepare |
|---|---|---|---|
| **1. Two DSA problems, then Java rapid-fire** (SDE-1 style) | High for SDE-1 | Invite mentions a "DSA / problem-solving" round, a 60-min slot, maybe a third-party platform link | 2 mediums in 60 min with running code in Java. Then 20 quick Java answers (`05`). |
| **2. Easy DSA plus "write it as classes"** (ASDE 2026 style) | High for ASDE | Short round; interviewer asks for classes/interfaces after a simple problem | Binary search, stack/queue design, writing an `interface` and a class that implements it. |
| **3. LLD discussion: requirements → tables → APIs** (ASDE 2024 style) | Medium | Round called "LLD" or "design"; no code editor needed | A clear step order: requirements, entities, tables, endpoints, HTTP verbs (`03`, `04`). |
| **4. Machine coding: build a working app on your laptop** | Medium (more often Round 2 now) | Invite says "machine coding", 60–120 min, "bring your laptop / share screen / use your IDE" | Splitwise, Snake & Ladder, parking lot, food cart with discounts (`03`). Practise until you can get a working core in 45 min. |

**If the invite doesn't say:** reports say the recruiter may not tell you the format [4]. **Ask the recruiter politely**: *"Could you share the format of Round 1 — DSA, LLD or machine coding — and which languages are allowed?"* That's normal and expected.

Most rounds also start with **5–15 minutes on your resume** [12][17]. Be ready with a 60-second intro and the Myntra story (`06`).

### 1.7 Later rounds (so nothing surprises you)

- **LLD / machine coding round** (usually Round 2 now): Splitwise [19][31][39][40][43], food ordering with discounts [18], driver-to-order assignment [17], Job Scheduler [14], Instagram APIs [24]. About 60 minutes [17][19][24].
- **Hiring Manager round:** resume deep dive, plus often one DSA question (Merge Intervals [20], Coin Change II [18]) or a design question (Google Sheets [1], food-order matching [25], DB + APIs for a resume project [31], medicine inventory [8]), plus CS fundamentals and SQL/indexing [18][20].
- **HR round:** "Why Swiggy?", other offers, joining date [1][14].
- **A warning from [22]:** one "ASDE offer" turned out to be a 3-month internship with possible conversion to full time, which the job description hadn't said. **Ask HR whether the offer is full time.**

### 1.8 Tools you'll code in

| Tool | Reports |
|---|---|
| Your own IDE (VS Code etc.) with screen share, for machine coding | [4][19] |
| Sublime Text | [5] |
| GFG online IDE | [4] |
| Shared Google Doc (no compiler!) | [6] (OLD) |
| InterviewVector / third-party platform | [17][18][29] |
| CoderPad | No report mentions it (unknown) |

**Practical tip:** before the interview, make sure Java runs on your laptop (`java -version`) and that you can create and run a small project in VS Code or IntelliJ in under 2 minutes. In a machine coding round, losing 10 minutes to setup hurts.

---

## Part 2: Every question asked before, with short answers

**How to use this part.** For each question, read the short idea and ask yourself: "Could I explain this to a friend?" If not, go to the deeper file. ✅ means `02_DSA_From_Zero.md` has a full, step-by-step Java solution.

**Complexity in one line:** "time O(n)" means the work grows in step with the input size n. "Space O(1)" means you only use a fixed amount of extra memory, however large the input. `02` explains this from zero.

### 2.1 DSA, Tier 1: asked in 3 or more reports (do these first)

**1. Valid Parentheses (LC 20) ✅ — 3 reports [2][12][24]**
*Question:* given a string like `"({[]})"`, check whether every bracket is closed in the right order.
*Idea:* use a **stack**, a pile where you only touch the top, like a stack of plates. Push every opening bracket. When a closing bracket comes, the top of the stack must be its matching opener; pop it. At the end the stack must be empty.
*Complexity:* time O(n), space O(n).
*Related, also asked:*
- **Valid Parenthesis String (LC 678)** [2]: `*` can be `(`, `)` or empty. Track a *range* of possible open counts (low, high); fail if high < 0; answer is whether low reaches 0. Time O(n), space O(1).
- **Longest Valid Parentheses (LC 32)** [40]: stack of indices. Push -1 first. On `)`, pop; if the stack is empty push the current index, else the length is `i - stack.top()`. Time O(n), space O(n).

**2. Merge Intervals (LC 56) ✅ — 3 reports [17 "variation"][20][32]**
*Question:* given time ranges like `[1,3],[2,6],[8,10]`, merge the overlapping ones into `[1,6],[8,10]`.
*Idea:* **sort by start time**. Walk through. If the current interval starts before the last merged one ends, they overlap: extend the end. Otherwise start a new block. Like merging overlapping meeting slots in a calendar.
*Complexity:* time O(n log n) for the sort, space O(n) for the output.
*Related:* **Meeting Rooms (LC 252)** [24]: can one person attend all meetings? Sort by start; if any meeting starts before the previous one ends, the answer is no. O(n log n).

**3. Lowest Common Ancestor of a binary tree (LC 236) ✅ — 3 reports [4][8][28]**
*Question:* in a family-tree-like structure, find the lowest node that has both p and q below it.
*Idea:* recursion. Ask the left and right subtrees, "Did you find p or q?" If both say yes, the current node is the answer. If only one says yes, pass that answer up. If the current node is p or q itself, return it.
*Complexity:* time O(n), space O(h), where h is the tree height (the recursion stack).
*BST version (LC 235)* [4]: in a Binary Search Tree (left < node < right), if both values are smaller go left, if both are bigger go right, otherwise the current node is the answer. Time O(h).

**4. LRU Cache (LC 146) ✅ — 3 reports [9][10][32 "most asked"]**
*Question:* build a cache (a small fast memory) of fixed size. When it's full, throw out the **Least Recently Used** item. `get` and `put` must be O(1).
*Idea:* a **HashMap** (key → node) for instant lookup, plus a **doubly linked list** ordered by recent use. Every access moves the node to the front; eviction removes from the back. Real life: the "recently opened" list on your phone, where the oldest one drops off.
*Complexity:* O(1) per operation, space O(capacity).
*Note:* [9] asked it as part of an OS question on page replacement, and [10] only discussed it without code. Know both the code and the OS link (`04`).

**5. Knapsack family ✅ — 5 reports**
- **0/1 Knapsack** [37][38]: items with weight and value, a bag with a capacity; each item is taken once or not at all; maximise value. *Idea:* `dp[w]` = best value with capacity w; for each item, go w from high to low: `dp[w] = max(dp[w], dp[w - wt] + val)`. Time O(n·W), space O(W).
- **Coin Change II, number of ways (LC 518)** [18]: count the ways to make an amount from coins, with unlimited use of each coin. *Idea:* `dp[0] = 1`; for each coin, for amount from coin to target: `dp[a] += dp[a - coin]`. The coin loop must be outside so that orders aren't counted twice. Time O(n·amount), space O(amount).
- **"Select weights so the sum ≤ capacity" (subset sum)** [27]: a boolean DP, `can[s] = can[s] || can[s - w]`. O(n·cap).
- **"Coin change (similar)"** [28]: the minimum-coins version (LC 322): `dp[a] = min(dp[a], dp[a - coin] + 1)`.

**6. Grid DP / path problems ✅ — 5 reports**
- **Minimum Path Sum (LC 64)** [6]: move only right or down; minimise the sum. `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`. O(m·n) time; O(n) space with one row.
- **Unique Paths II with obstacles (LC 63)** [7]: count paths; an obstacle cell gets 0 paths. `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. O(m·n).
- **Maximum points top-left → bottom-right with negative cells** [8]: same as LC 64 but with `max`. Negatives just mean you can't skip a cell greedily, so DP handles it.
- **"Medium-hard 2D grid DP"** in the ASDE OA [12]: the exact problem is unknown. Master LC 62/63/64 and LC 120 (Triangle).
- **Reachability A→B moving only right, up-right or down-right in a 0/1 grid** [1]: go column by column. A cell is reachable if it's open and any of its three left-side neighbours (left, up-left, down-left) is reachable. O(m·n).

### 2.2 DSA, Tier 2: asked in 2 reports

| # | Problem | Reports | The idea in simple words | Complexity |
|---|---|---|---|---|
| 7 | **Unique BSTs / count binary trees from 1..n (LC 96)** ✅ | [4][18] | Pick each number as the root. Left side has i-1 nodes, right side n-i. `count[n] = Σ count[i-1] × count[n-i]` (the Catalan numbers). | O(n²) time, O(n) space |
| 8 | **Edit Distance (LC 72)** ✅ | [14][29] | Min insert/delete/replace operations to turn word A into word B. `dp[i][j]`: if the letters match, take the diagonal; else 1 + min(insert, delete, replace). Like autocorrect measuring how far a typo is. | O(m·n) |
| 9 | **Rotting Oranges (LC 994) variant** ✅ | [27][28] | **Multi-source BFS** (BFS spreads level by level, like ripples in water). Put every rotten orange in a queue at once; each minute, rot the neighbours. Answer = minutes taken, or -1 if a fresh orange remains. | O(m·n) |
| 10 | **Alien Dictionary (LC 269)** | [8][28] | Compare adjacent words; the first differing letters give an edge `a → b` ("a comes before b"). Then a **topological sort** (ordering tasks so that every prerequisite comes first) gives the alphabet. Edge cases: a cycle, or a longer word before its own prefix ("abc" before "ab") → invalid. | O(total chars) |
| 11 | **Building Bridges (DP / LIS)** | [8][28] | Bridges connect north city a_i to south city b_i; no two may cross. Sort by north coordinate, then find the **Longest Increasing Subsequence** of the south coordinates. | O(n log n) |
| 12 | **Maximum Length of Pair Chain (LC 646)** ✅ | [28][40] | Pairs (a,b); chain if b < c of the next. **Greedy:** sort by end, and always take the pair that ends earliest, just like picking the most meetings in one room. | O(n log n) |
| 13 | **Subarrays with sum divisible by k (LC 974)** ✅ | [10][28] | **Prefix sums:** if two prefix sums have the same remainder mod k, the part between them is divisible by k. Count remainders in a map; add the count each time you see a remainder again. Fix negative remainders with `((x % k) + k) % k`. [10] asked to *print* them: store the indices for each remainder. | O(n) count; printing is O(n + output) |
| 14 | **Longest Substring Without Repeating Characters (LC 3)** ✅ | [18 OA][28] | **Sliding window:** two pointers form a "window". Grow the right end; when a character repeats, move the left end just past its last position. Remember the best length. | O(n) time, O(alphabet) space |
| 14b | *Related:* Longest substring with at most k distinct characters (LC 340) | [30] | Same window plus a count map; shrink while the map has more than k keys. | O(n) |
| 15 | **"Pick two numbers, replace them with their sum, add the sum to a total; minimise the total" (LC 1167 Min Cost to Connect Sticks)** ✅ | [1][11] | Always join the **two smallest**, using a **min-heap** (a structure that always hands you the smallest item quickly). Same idea as Huffman coding. | O(n log n) |
| 16 | **Min Stack, O(1) getMin (LC 155)** ✅ | [2] | Keep a second stack of the minimum so far. [2] asked for **O(1) extra space**: store `2*x - min` when x is a new minimum and decode it on pop. `02` explains this trick. | O(1) per op |
| 16b | **Design a stack using an array** | [16] | An array plus a `top` index. `push`: check full, `arr[++top] = x`. `pop`: check empty, return `arr[top--]`. They wanted it as a class implementing an interface, with errors on overflow/underflow. Bonus: grow the array by doubling when full. | O(1) per op |
| 17 | **Construct a binary tree from preorder + inorder (LC 105)** ✅ | [3] | The first preorder value is the root. Find it in inorder: everything to its left is the left subtree, everything to its right is the right subtree. Recurse. Use a HashMap of value → inorder index for speed. | O(n) |
| 18 | **Graph shortest path / BFS: Network Delay Time (LC 743)** ✅ | [14][18][26] | **Dijkstra:** from the source, always expand the closest unvisited node, using a min-heap. The answer is the largest shortest distance, or -1 if some node can't be reached. Like a signal spreading through cities along roads of different lengths. | O(E log V) |

### 2.3 DSA, Tier 3: asked once (grouped by report)

These were each asked only once, but they show the *style*. Don't try to code all of them. Read each idea and make sure it makes sense. ✅ means a full solution is in `02`.

**Recent reports (2024–2026): most relevant**

| Problem | Report | Short idea | Complexity |
|---|---|---|---|
| **First and last position in a sorted array (LC 34)** ✅ | [16] ASDE Mar 2026, Round 1 | Two binary searches. One keeps going left after finding the target (first occurrence); the other keeps going right (last occurrence). Like finding the first and last page of a word's entries in a dictionary. | O(log n) |
| **Merge Intervals variation** ✅ | [17] | See #2. | |
| **A bitwise-manipulation medium** | [17] | Exact problem unknown. Know: `x & (x-1)` removes the lowest set bit; `x & 1` tests odd; XOR cancels pairs (single number); counting bits. | — |
| **Integer Nth root of a number** | [18] OA | **Binary search on the answer** from 1 to x: if `mid^n ≤ x` go right, else go left. Watch for overflow; use `long` and stop multiplying early once it's too big. | O(log x · n) |
| **A sliding-window problem beyond LC 3** | [18] OA | Unspecified. Practise LC 3, 76, 424 and 904. | — |
| **Codeforces 1133C "Balanced Team"** | [19] | Largest team where any two skills differ by ≤ 5. Sort, then a sliding window: move left while `a[r] - a[l] > 5`. | O(n log n) |
| **Partition array into two arrays to minimise the sum difference (LC 2035)** | [19] | Hard. **Meet in the middle:** split into halves, list all subset sums of each half grouped by size, then binary search for the best match. | O(2^(n/2) · n) |
| **Currency conversion** — given rates like `['USD','GBP',0.77]`, return the rate for a from/to pair ✅ (as LC 399) | [23] | Build a graph: each currency is a node, and each rate is an edge with weight r one way and 1/r the other way. DFS/BFS from `from` to `to`, multiplying the weights along the path. | O(V+E) per query |
| **"Counting nice substrings"** (LeetCode medium) | [14] OA | Exact problem unknown. Likely a sliding-window or prefix-count problem. | — |
| **Sum Root to Leaf Numbers (LC 129)** | [14] | DFS carrying `num = num*10 + node.val`; at a leaf, add num to the total. | O(n) |
| **Next Permutation (LC 31)** ✅ | [29] | From the right, find the first i with `a[i] < a[i+1]`. From the right again, find the first j with `a[j] > a[i]`. Swap them, then reverse everything after i. If no such i exists, reverse the whole array. | O(n) |
| **Edit Distance** ✅ | [29] | See #8. That candidate got a "weak hire" for not finishing it in time. | |
| **Minimum platforms** ✅ (with Meeting Rooms) | [31] | Sort arrival times and departure times separately. Walk both with two pointers: an arrival adds a platform, a departure frees one. Track the maximum in use. | O(n log n) |
| **Trapping Rain Water (LC 42)** ✅ | [31] | Water above a bar = `min(tallest on left, tallest on right) - height`. Two pointers from both ends, always moving the side with the smaller max. | O(n), O(1) space |
| **Reverse nodes in k-groups (LC 25)** ✅ | [30] | Check that k nodes remain, reverse those k, connect them to the previous group, repeat. | O(n) |
| **Linked list cycle + start node (LC 142)** ✅ | [30] | **Floyd's algorithm:** a slow pointer (1 step) and a fast pointer (2 steps) meet inside the cycle. Then move one pointer back to the head; step both by 1, and they meet at the cycle start. | O(n), O(1) |
| **"Delivery zone coverage" graph reachability** | [30] OA | Running BFS from every node timed out. Fix: **reverse all edges once** and run one BFS (or multi-source BFS) from the targets. One BFS instead of n. | O(V+E) |
| **Postfix expression evaluation** | [32] ASDE | Stack: push numbers; on an operator, pop b then a, push `a op b`. Example: `"2 3 + 4 *"` → 20. | O(n) |
| **Reverse the odd-indexed nodes of a linked list** | [32] ASDE | Split into two lists (even and odd positions), reverse the odd list, then weave them back together. | O(n) |
| **Graphs, LRU, merge intervals** | [32] ASDE | See #4, #2, and BFS/DFS in `02`. | |
| **Greedy; Trie; BFS + heap** (names not given) | [26] intern 2025 | Know the basic **Trie** (prefix tree, like phone-contact search: typing "va" shows all names starting with "va"): insert, search, startsWith. | — |
| **Sliding window, graph traversal, hard DP, tree** (names not given) | [25] | Pattern list only. | — |

**Older reports (pre-2023): style reference**

| Problem | Report | Short idea |
|---|---|---|
| Row with max 1s in a row-sorted binary matrix | [1] | Start at the top-right. On a 1, move left (and remember the row); on a 0, move down. O(n+m). |
| Add two numbers stored as linked lists, without reversing (LC 445) | [1] | Push both lists onto two stacks, pop and add with a carry, and build the result from the front. O(n+m). |
| 3Sum, unique triplets (LC 15) | [1] | Sort; fix i; two pointers on the rest; skip duplicates. O(n²). |
| Stepping Numbers (LC 1215) | [1] | Numbers whose adjacent digits differ by 1. BFS from 1..9: from x, append last digit ±1. Collect those in [low, high]. |
| Task scheduling with cooldown k, minimum time and the order (LC 621 variant) ✅ | [1] | Most-frequent task first. Formula `(maxFreq - 1) * (k + 1) + countOfMax`, answer = max(that, n). For the order, use a max-heap plus a cooldown queue. |
| Search a row- and column-sorted matrix (LC 240) | [2] | Start at the top-right; if bigger than the target go left, if smaller go down. O(m+n). |
| Fountains: min fountains to cover the garden (LC 1326-like) | [3] | Turn each fountain into an interval, then "jump game II" greedy. O(n). |
| Weighted tree: +1 operations so adjacent nodes differ by ≤ 1 | [3] | BFS-based relaxation from the largest weights (as reported). |
| Array of size 2n, last n are zero: duplicate each element in place | [3] | Fill from the back: write `a[i]` twice at positions `2i+1` and `2i`, going from i = n-1 down to 0. O(n). |
| Next greater element | [3] | **Monotonic stack.** That candidate wrongly said a stack can't solve it and was waitlisted. Learn it! O(n). |
| First non-repeating character with one 26-size array | [3] | Count pass, then a second pass over the string to find the first count of 1. O(n). |
| Count binary strings without consecutive 1s | [5] | `end0[n] = end0[n-1] + end1[n-1]`, `end1[n] = end0[n-1]` (Fibonacci). Follow-up: matrix exponentiation for O(log n). |
| N segments (x1,x2): the vertical line where the most segments meet | [5] | Sweep line: +1 at start, -1 after end, sort events, track the max. O(n log n). |
| Count pairs with a given sum | [5] | HashMap of counts seen so far. O(n). |
| Detect a cycle in a graph (Spring bean dependency DAG) | [5] | DFS with three colours (unvisited, in-progress, done); meeting an in-progress node means a cycle. Or Kahn's topological sort. |
| Parking Dilemma (IBM OA 2019) | [6] | Cover k cars with the minimum roof length. Sort positions; best `a[i+k-1] - a[i] + 1`. |
| Count distinct substrings via suffix array | [6] | Total substrings minus the sum of LCPs. Advanced; low priority. |
| Reverse letters of each word (LC 557) | [7] | Split on spaces, reverse each word. O(n). |
| Rotate a matrix by 90° (LC 48) | [7] | Transpose, then reverse each row. O(n²), in place. |
| Travelling salesman (greedy) | [7] | Nearest-neighbour heuristic. True TSP is NP-hard (no known fast exact method). |
| Top view of a binary tree | [7] | BFS with a horizontal distance; keep the first node seen at each distance (TreeMap). |
| Kadane's maximum subarray (LC 53) ✅ | [8] | `cur = max(x, cur + x)`, `best = max(best, cur)`. O(n). |
| f6() → f12() uniform random | [8] | `(f6() - 1) * 2 + coin()`, where `coin()` comes from f6 (1–3 vs 4–6), plus 1. Each of 1..12 is equally likely. |
| Packets of 6/9/20: can N be made, and the min number of packets | [8] | Coin-change DP with coins {6, 9, 20}. |
| Space-optimised pin code → location store | [8] | Discussion: a Trie on digits, or a sorted array plus binary search. |
| Stock max (HackerRank) | [9] | Scan from the right keeping the max; profit += max - price. O(n). |
| All subsets of a string | [9] | Recursion: take or skip each char. O(2^n). |
| Robot move sequence is circular ("GLGLGLG") | [9] | Simulate once (or 4 times); circular if back at the start or not facing north. |
| Single non-repeated element in a sorted array, O(log n) (LC 540) | [9] | Binary search on pairs: at an even mid, if `a[mid] == a[mid+1]` the answer is on the right. |
| Running median of a stream (LC 295) ✅ | [9] | Two heaps: a max-heap for the lower half and a min-heap for the upper half; keep them balanced. O(log n) per add. |
| Search in a rotated sorted array (LC 33) ✅ | [9] | Binary search; one half is always sorted, so check whether the target lies in it. O(log n). |
| Expression evaluation | [9] | Two stacks (numbers, operators) with precedence. |
| Print all contiguous subarrays with sum divisible by x ✅ (as LC 974) | [10] | See #13. |
| Sort a 900 MB file with 100 MB of memory | [10] | **External sort:** sort 100 MB chunks, write them to disk, then do a k-way merge with a min-heap. |
| Count distinct subarrays with at most k odd elements | [27] | Sliding window for "at most k odd" plus a set/hash to dedupe (as reported). |
| Number line 0..n, move string: count subsequences of moves that go from x to y | [27] | DP over positions as you scan the moves. |

### 2.4 SQL questions from the OA and interviews

SQL showed up in the ASDE OA ("two easy-to-medium SQL queries, simple joins" [20]; "1 SQL on joins" [22]), in SDE-1 OAs ("1 SQL + 3 coding" [2]; "hard SQL query" [14]), and in HM rounds ("SQL, indexing, DB performance" [18]). `04` teaches SQL from zero.

**The one exact SQL question reported [3]:**
> Tables `Employee(emp_id, name, dept_id)` and `Department(dept_id, dept_name)`. List every department with its number of employees, **including departments with 0 employees**, sorted by count (high to low), then by department name, then by department ID.

```sql
SELECT d.dept_id,
       d.dept_name,
       COUNT(e.emp_id) AS emp_count
FROM Department d
LEFT JOIN Employee e
       ON e.dept_id = d.dept_id
GROUP BY d.dept_id, d.dept_name
ORDER BY emp_count DESC, d.dept_name ASC, d.dept_id ASC;
```

**Why this works:**
- **LEFT JOIN** keeps *every* department, even one with no matching employee. Its employee columns just become NULL. A plain (INNER) JOIN would silently drop the empty departments. That is the trap in this question.
- **COUNT(e.emp_id)** counts only non-NULL values, so an empty department gets 0. `COUNT(*)` would wrongly give 1, because the joined row still exists.
- **GROUP BY** squashes the rows into one row per department.

Real-life example: listing every Swiggy city with its number of active restaurants. A new city with no restaurants yet must still show up with 0.

**Since the other SQL questions were only described as "joins" or "hard", practise these shapes:**

```sql
-- 1. Join + filter: names of customers who ordered from restaurant 7
SELECT DISTINCT c.name
FROM Customer c
JOIN Orders o ON o.customer_id = c.id
WHERE o.restaurant_id = 7;

-- 2. Rows with NO match (anti-join): customers who never ordered
SELECT c.name
FROM Customer c
LEFT JOIN Orders o ON o.customer_id = c.id
WHERE o.id IS NULL;

-- 3. Group + HAVING: customers with more than 3 orders
SELECT customer_id, COUNT(*) AS n
FROM Orders
GROUP BY customer_id
HAVING COUNT(*) > 3;

-- 4. Second-highest salary
SELECT MAX(salary) FROM Employee
WHERE salary < (SELECT MAX(salary) FROM Employee);

-- 5. Highest-paid employee in each department (window function)
SELECT dept_id, name, salary
FROM (
  SELECT dept_id, name, salary,
         RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rnk
  FROM Employee
) t
WHERE rnk = 1;

-- 6. Employee who earns more than their manager (self-join)
SELECT e.name
FROM Employee e
JOIN Employee m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

`WHERE` filters *rows before grouping*; `HAVING` filters *groups after grouping*. That difference is a favourite question.

### 2.5 CS fundamentals actually asked

Each answer below is what you'd say in 30–60 seconds. `04` (OS/DBMS/networks/security) and `05` (Java/Spring) explain each one properly with examples.

#### Operating systems

| Question | Report | Short answer |
|---|---|---|
| **Process vs thread, and why do we need both?** | [20] | A **process** is a running program with its own memory, like a separate restaurant. **Threads** are workers inside one process that share its memory, like cooks in the same kitchen. Threads are cheap to create and can share data quickly, but one bad thread can corrupt shared data. Processes are isolated and safer, but heavier and slower to talk to each other. We use processes for isolation (a Chrome tab crash doesn't kill the others) and threads for doing several things inside one program at once (a web server handling many requests). |
| **Deadlock, semaphores, threads, concurrency "in depth"** | [12] | **Deadlock:** two or more threads each hold something and wait forever for what the other holds. Two cars on a narrow bridge, each waiting for the other to back up. It needs all four conditions: mutual exclusion, hold-and-wait, no preemption, circular wait. Break any one to prevent it (e.g. always lock resources in the same order). A **semaphore** is a counter that limits how many threads may enter at once, like a parking lot with N spaces. |
| **Mutex vs semaphore** | [9] | A **mutex** is a lock with one key: only the thread that locked it can unlock it, and only one thread is inside. Like a single washroom key. A **semaphore** is a counter with N permits, and any thread can release a permit. Like a lot with N parking tokens. A semaphore with N=1 behaves *almost* like a mutex, but has no ownership. |
| **4 resources, 4 processes: when can deadlock happen?** | [9] | Deadlock is possible only if all four conditions hold and the requests form a **cycle**, e.g. P1 holds R1 and wants R2, P2 holds R2 and wants R1. If each process needs only 1 resource, there's no deadlock. With 4 identical resources, if each process needs at most 2, at least one can always finish (by the pigeonhole principle), so there's no deadlock. |
| **A process has a Word file open in write mode and is preempted. Can another process open it in write mode?** | [9] | Preemption only pauses the CPU work. The process **still holds the file and its lock**. If the OS or app uses an exclusive write lock (as Word does), the second process is blocked or gets "file in use". Preemption doesn't release resources. |
| **Cache memory and the TLB** | [9] | **Cache:** a small, very fast memory near the CPU that keeps recently used data, like keeping your most-used spices on the counter instead of in the store room. **TLB** (Translation Lookaside Buffer): a tiny cache of recent virtual-address → physical-address translations, so the CPU doesn't look up the page table in RAM every time. |
| **Which page-replacement policy would you use? Then implement LRU.** | [9] | When RAM is full and a new page is needed, evict one. **FIFO** evicts the oldest loaded; **Optimal** evicts the one used furthest in the future (impossible in practice, so it's a benchmark); **LRU** evicts the least recently used. Say "LRU, because programs usually reuse recent data (locality)". Then code the LRU cache (#4). |
| **Define multithreading; async programming plus a problem** | [33] | Multithreading: one program running several threads at the same time, e.g. a server handling 100 users at once. **Async:** start a slow task (a network call) and carry on without waiting; get notified when it's done. Like ordering food on Swiggy and continuing your work until the "delivered" notification. |
| **Explain this Java multithreading code** | [12] | Be ready to read code with `Thread`, `Runnable`, `synchronized`, `ExecutorService`, and spot a race condition (two threads doing `count++` at once and losing updates). See `05`. |
| **Concurrency** (HM round) | [23] | Same topics; tie them to Atlassian's worker pool (`06`). |

#### DBMS

| Question | Report | Short answer |
|---|---|---|
| **ACID** | [5][26] | The four promises of a database **transaction** (a group of steps that must all happen or none). **Atomicity:** all or nothing; if the payment fails, the order isn't created. **Consistency:** rules (like "balance ≥ 0") always hold. **Isolation:** transactions running at the same time don't see each other's half-done work. **Durability:** once committed, data survives a crash. Bank transfer example: debit A and credit B together, or neither. |
| **What is indexing, where is it used, and how is it implemented?** | [20] | An **index** is like the index at the back of a book: instead of reading every page (a full table scan), you jump to the right page. It's used on columns you search, join or sort by (`user_id`, `order_date`). It's usually a **B+ tree**, a sorted, balanced tree where each lookup takes about 3–4 disk reads even for millions of rows. Cost: extra storage, and slower inserts and updates, because the index must be updated too. *(That ASDE candidate couldn't answer this and still got the offer, but don't rely on that.)* |
| **All types of joins** | [6] | **INNER:** only matching rows from both. **LEFT:** all rows from the left, plus matches (NULL if none). **RIGHT:** the mirror of LEFT. **FULL OUTER:** all rows from both. **CROSS:** every pair. **SELF:** a table joined with itself (employee and manager). |
| **Sharding** | [6] | Splitting one huge table across several database servers by a key, e.g. orders of users 1–1M on server A and 1M–2M on server B. Like opening several counters at a bank so one queue doesn't get too long. Costs: queries across shards are hard, and rebalancing is hard. |
| **Normalisation** | [6] | Arranging tables so each fact is stored **once**, to avoid duplicate and inconsistent data. 1NF: no lists in one cell. 2NF: no partial dependency on part of a key. 3NF: no non-key column depending on another non-key column. Example: don't store the restaurant's address inside every order row; store the `restaurant_id` and keep the address in a Restaurant table. |
| **SQL vs NoSQL** | [12] | **SQL** (MySQL, Postgres): tables with a fixed schema, joins and strong transactions; good for orders and payments. **NoSQL** (MongoDB, Redis, Cassandra, Aerospike): flexible or simple key-value data, built to spread over many machines; good for caches, sessions, live locations, huge write volumes. |
| **MongoDB vs SQL; why MongoDB?** | [2][26] | MongoDB stores JSON-like **documents** with a flexible schema, which is handy when each record can have different fields (a product catalogue). Choose SQL when you need joins and multi-row transactions (money). Always justify your choice from the data's shape and needs, not "it's popular". |
| **Design a DB for a food shop, reduce its complexity, then joins** | [9] | Tables: `Customer`, `Item(id, name, price)`, `Order(id, customer_id, time, total)`, `OrderItem(order_id, item_id, qty)`. Normalise (don't repeat item names in orders), add indexes on the foreign keys, then write join queries ("total spent per customer"). |
| **SQL queries, indexing and DB performance** (HM) | [18] | Use `EXPLAIN` to see whether a query uses an index; add the right index; avoid `SELECT *`; paginate; cache hot reads. Tie it to your Myntra story: 2 queries became 1, halving database connections (`06`). |

#### Networks and web

| Question | Report | Short answer |
|---|---|---|
| **TCP vs UDP, and their use cases** | [20] | **TCP:** connection first (a "handshake"), guaranteed, in-order delivery, retransmits lost data. Like a registered parcel with tracking. Used for web pages, payments, APIs. **UDP:** just sends packets with no guarantee, but it's fast. Like shouting across a room. Used for video calls, live games, DNS. |
| **TCP vs HTTP** | [20] | Different layers. **TCP** is the *transport*: it reliably carries bytes between two machines. **HTTP** is an *application* protocol on top of TCP that defines the message format: "GET /menu", status codes like 200 and 404, headers. TCP is the delivery truck; HTTP is the language written on the order form inside. |
| **HTTP vs REST; what is REST?** | [20] | **HTTP** is a protocol. **REST** is a *style* of designing APIs on top of HTTP: everything is a **resource** with a URL (`/users/42/orders`), and you use HTTP verbs for actions (GET read, POST create, PUT/PATCH update, DELETE remove). REST requests are **stateless**: each request carries everything needed and the server keeps no memory of earlier ones. |
| **How does a URL work? What is a load balancer?** | [5] | URL `https://swiggy.com/restaurants?city=blr` = protocol + domain + path + query. The browser asks DNS for the IP, connects, and sends the HTTP request. A **load balancer** sits in front of many servers and spreads requests across them, like a host at a restaurant seating guests at different tables so no waiter is overloaded. It also stops sending traffic to dead servers. |
| **What happens when you type www.facebook.com? What data structure does DNS use? DNS and IP** | [9] | (1) The browser checks its cache for the IP. (2) Otherwise it asks **DNS**, the internet's phone book: domain name → IP address. The query goes from a resolver to the root, then the `.com` servers, then facebook's own name server. (3) TCP connection, then TLS (HTTPS encryption). (4) HTTP GET. (5) The server responds and the browser renders HTML, CSS and JS. DNS data is a **hierarchical tree** (root → com → facebook), with caching at each level. |
| **"Computer networks questions"** | [6] | Unspecified. Cover OSI/TCP-IP layers, HTTP status codes, HTTPS/TLS and WebSockets (`04`). |

#### Security

| Question | Report | Short answer |
|---|---|---|
| **Hashing vs encryption** | [12] | **Encryption** is two-way: lock with a key and unlock with a key. Used for data you need back (messages, card data in transit). **Hashing** is one-way: turns data into a fixed "fingerprint", and you can't get the original back. Used for passwords: store the hash, and at login hash the typed password and compare. |
| **Salting** | [12] | Add a random string (the **salt**) to each password before hashing, and store it with the hash. Two users with the password "hello123" then get different hashes, and pre-computed "rainbow tables" of common-password hashes become useless. |
| **JWT** | [12] | **JSON Web Token:** after login, the server gives you a signed token containing who you are (user id, expiry). You send it with every request. The server checks the **signature** with its secret key, so it doesn't need to look you up in a session database. Like a wristband at an event: staff just check it's genuine. It's signed, not encrypted, so never put secrets inside. |
| **Encryption in your project; block cipher and AES; Polybius cipher strength** | [9] | A **block cipher** encrypts fixed-size blocks (AES uses 128-bit blocks, with keys of 128/192/256 bits). **AES** is the standard symmetric cipher (the same key encrypts and decrypts). The Polybius square is a classical substitution cipher and is very weak: frequency analysis breaks it. |
| **Secure API design (e.g. no unencrypted data in headers)** | [14] | Always HTTPS; auth via tokens; never put passwords or personal data in URLs or plain headers; validate every input; rate-limit; return generic error messages. |

#### OOP, Java and Spring (heavier for SDE-1 in 2025–26)

The 2026 SDE-1 Round 1 had **20–25 rapid-fire Java questions** after the DSA [17]. These are the ones reported. `05` explains each with examples.

| Question | Report | Short answer |
|---|---|---|
| **OOP, JVM/JDK, garbage collection, Collections, multithreading** | [17] | JDK = tools to *write and compile* Java; JRE = what *runs* it; JVM = the engine that executes bytecode. GC automatically frees objects nothing refers to anymore. Collections: `ArrayList`, `HashMap`, `HashSet`, `LinkedList`, `PriorityQueue`, `TreeMap`. |
| **Interface vs abstract class** | [18] | **Interface:** a contract of *what* to do (methods to implement); a class can implement many. **Abstract class:** a partly built base with shared code and state; a class can extend only one. Example: `PaymentMethod` as an interface (UPI, Card, COD); `Vehicle` as an abstract class holding shared `registrationNumber` logic. |
| **Exception handling; checked vs unchecked** | [18] | **Checked** exceptions (`IOException`) must be handled or declared, because the compiler forces you. They're expected problems like a missing file. **Unchecked** (`NullPointerException`, `IllegalArgumentException`) are usually programming bugs, and the compiler doesn't force you. Use `try / catch / finally` or try-with-resources. |
| **The `final` keyword** | [18] | `final` variable: can't be reassigned. `final` method: can't be overridden. `final` class: can't be extended (`String` is final). |
| **`@RestController`** | [18] | A Spring annotation marking a class whose methods handle HTTP requests and return data (JSON) directly, not a web page. It's `@Controller` + `@ResponseBody`. |
| **How Spring Boot manages configuration** | [18] | Through `application.properties` / `application.yml`, environment variables, and profiles (`dev`, `prod`). Values are injected with `@Value` or `@ConfigurationProperties`. Auto-configuration sets up sensible defaults based on what's on the classpath. |
| **Spring Bean lifecycle** | [18] | A **bean** is an object Spring creates and manages for you. Lifecycle: create (constructor) → inject dependencies → `@PostConstruct` init → in use → `@PreDestroy` on shutdown. |
| **Inheritance vs composition** | [5] | Inheritance: "is-a" (a `Car` is a `Vehicle`). Composition: "has-a" (a `Car` has an `Engine`). Prefer composition: it's more flexible, because you can swap the engine. |
| **`static`; `volatile`** | [5] | `static` belongs to the class, not each object (a shared counter). `volatile` makes every thread read a variable's latest value from main memory instead of a stale cached copy. It gives visibility, not atomicity. |
| **Spring beans and dependency validation** | [5] | Spring builds a dependency graph; a missing bean or a circular dependency fails at startup. |
| **Polymorphism, static vs dynamic, with examples** | [6] | "One name, many forms." **Static** (compile time): method overloading, e.g. `add(int,int)` and `add(double,double)`. **Dynamic** (run time): overriding, e.g. `PaymentMethod p = new UpiPayment(); p.pay()` runs UPI's version. |
| **Singleton pattern** | [27] | A class with exactly one instance (one shared config or one DB connection pool). Private constructor + a static `getInstance()`; make it thread-safe. |
| **"Implement the OOP pillars in an editor, fully functional, and explain each"** | [6] | Encapsulation (private fields + getters), Abstraction (interface), Inheritance (extends), Polymorphism (override). Write one tiny example that shows all four. `05` has one. |

#### Theory and maths

| Question | Report | Short answer |
|---|---|---|
| **Which is bigger, O(2^n) or O(n!)? Prove it.** | [9] | n! grows faster. n! = 1·2·3·…·n, while 2^n = 2·2·…·2. From the 4th term on, every factor of n! is bigger than 2, so for n ≥ 4, n! > 2^n, and the ratio n!/2^n keeps growing. |
| **Why is hashing O(1)?** | [9] | A hash function turns the key into an array index directly, so you jump to the slot without searching. It's O(1) *on average*; with many collisions it can degrade (Java 8 turns long chains into trees, giving O(log n) at worst). |

### 2.6 LLD and machine coding problems asked (requirements only)

`03_Machine_Coding_and_LLD_From_Zero.md` teaches the method and solves the main ones in Java. Here's just *what was asked*, most frequent first.

| # | Problem | Reports | Requirements as reported |
|---|---|---|---|
| 1 | **Splitwise** (expense sharing) | [19][31][39][40][43] — 5 reports, the most frequent | Users add expenses paid by one person and split among several (equally, by exact amounts, or by percentage); show who owes whom; show balances. One version had "5 key features" plus trade-offs and scaling [40]. [19]: 60 min in total, about 10 min of requirements and objects, then code in VS Code. The interviewer "wanted a fully working application" and rejected partial code. [31]: fully working code was not expected, and that candidate was selected. [39]: 120 min, working code. |
| 2 | **Snake & Ladder** | [4][40] (OLD) | **Mandatory:** a working game with OOP classes (Player, Game, Board). **Optional:** (1) board size N, (2) M players, (3) a ladder at a snake's tail, (4) a snake at a ladder's end, (5) keep playing until all players finish. Tested with the interviewer's test cases. 120 min. |
| 3 | **Magical Arena** (ASDE take-home) | [20][22][33][44] | A two-player attack/defend fight game in Java, Node or Go. Exact rules unknown; community solutions use health/strength/attack plus dice. |
| 4 | **Food ordering / cart with discounts** | [18] (Dec 2025) | Browse restaurants, add dishes to an order or cart, calculate the bill, apply offers/discounts, payment. The candidate used the **Strategy pattern** for discount rules and was selected. |
| 5 | **Driver–order assignment** | [17] (Mar 2026) | "Assign drivers to orders, where orders can be VIP or Non-VIP and drivers have ratings, with different matching conditions." Expected: design, choice of data structures, code, dry run. 1 hour. |
| 6 | **Calendar / meeting scheduler** | [7] (OLD) | Schedule meetings, show users' calendars, accept or reject meetings. 7 requirements, 3 hours; the code had to run. |
| 7 | **Multilevel parking lot** | [27] (OLD) | Park and unpark vehicles across floors and slots. Discussion of unfinished features, then a Singleton follow-up. |
| 8 | **Follow/unfollow for social media** | [20] (ASDE Dec 2024) | Requirements, DB schema, API endpoints, HTTP verbs. Discussion only. |
| 9 | **Instagram Post / GetFeed / Follow APIs + SQL queries** | [24] | API design plus the SQL behind it. |
| 10 | **Job Scheduler** | [14] (Dec 2024) | Write the interface and scaffold code: add, cancel, suspend and queue jobs. |
| 11 | **Library management app** | [29] (Android, 45 min) | Build it, then discuss the approach and architecture. |
| 12 | **Bike rental** | [24] (extra in the HM round) | LLD discussion. |
| 13 | **Binary tree from tuples: validate and serialise** | [10] (OLD 2015) | Check errors in order: more than 2 children, duplicate tuples, a cycle, multiple roots. If valid, print e.g. `(A(B(C(E)))(D))`. |
| 14 | **Rate limiter LLD** | [34] | Glassdoor snippet only; no details. |
| 15 | **Implement the OOP pillars, fully functional** | [6] (OLD campus) | See 2.5. |

**How machine coding is graded** (as candidates describe it):
- clean OOP classes and extensibility [4]
- code that **runs without errors** [7][19]
- data-structure choices plus a dry run [17]
- design patterns where they fit (Strategy [18], Singleton [27])
- how you'd add the features you didn't finish [27]

### 2.7 HLD questions seen in early or HM rounds

For ASDE, HLD came up mainly as *concept* questions (SQL vs NoSQL, JWT) [12], not whole designs. For SDE-1 it sometimes came up in the HM round. Know the outline of these; the existing `../06_System_Design_Master.md` covers them in depth.

| Problem | Report |
|---|---|
| Google Sheets: DB choice, schema, row/column shifting, copy-paste, collaborative editing | [1] (OLD) |
| Instagram APIs | [24] |
| Food-order matching system | [25] |
| API endpoints for a simple web app; failover and resiliency | [14] |
| DB + endpoints for an internal tool from the candidate's resume | [31] |
| Medicine inventory with a low-stock alarm | [8] |

### 2.8 Resume, project and behavioural questions asked

Almost every HM round had a **project deep dive** [1][2][4][7][18][20][31], and Round 1 itself often starts with one [12][17]. Prepare these with your real stories in `06_Resume_Projects_Behavioural_for_Swiggy.md`.

**Project deep-dive questions actually asked:**
- Scope and requirements of the project; implementation choices; **production issues you faced, how you fixed them, what would have prevented them, and what you'd do differently** [18]. *Use the Myntra SQL bug (Go map order) and the 5.7M cache warmup.*
- "Why React over vanilla JS?", "Why MongoDB?", "Can you do full stack?", "Backend or Android?" [2]. *Swap in your own tech: "Why Go?", "Why Kafka?", "Why SQS?", "Why FastAPI?"*
- "Why did you use [tool]? What were the challenges and the algorithms?" [3]. *For TraceX: why NetworkX, why XGBoost + Isolation Forest.*
- "Self-rate yourself on your tech stack." [18] *Be honest. A 7/10 you can defend beats a 9/10 you can't.*

**Behavioural questions (exact words where reported):**
- "Why do you want to join Swiggy?" [18][29]
- "Tell me one thing that you don't like about Swiggy." [43]
- "What two strengths and two weaknesses would a professor mention, and what have you done about the weaknesses?" [2]
- "Why a startup like Swiggy when you have other offers?" [9]
- "How did you find the online round?" [9]
- "Most challenging task in the last two years." [9]
- SWOT analysis, career goals, other offers. [1]
- Handling ambiguous requirements from a PM. [30]
- A production incident under pressure; leading a bug-fix sprint (STAR stories). [25]

**Short guide.** Use **STAR**: **S**ituation (one line of context), **T**ask (what you had to do), **A**ction (what *you* did — most of the answer), **R**esult (a number if possible). Keep each answer under 2 minutes. For "one thing you don't like about Swiggy", give a small, fair product observation with a suggestion, never an insult. Full prepared answers are in `06`.

---

## Part 3: Why people were rejected, and the rules that follow

### 3.1 What went wrong for real candidates

| What happened | Report | The lesson |
|---|---|---|
| Splitwise LLD: wrote partial code; the interviewer wanted a **fully working app**; rejected | [19] | Get a small working core first, then add features. |
| Wrongly told the interviewer a stack *can't* solve next-greater-element; waitlisted | [3] | Don't make confident claims you're unsure of. Say "Let me think…" and reason it out. |
| Ran out of time coding "binary tree from preorder + inorder" | [3] | Practise common problems until you can code them in 15–20 min. |
| Pretended not to know a puzzle they actually knew; the interviewer noticed | [3] | Be honest: "I've seen this before." Interviewers respect that and usually give another problem. |
| Didn't finish Edit Distance in time → "weak hire" | [29] | Know the classic DPs cold. |
| ASDE Android machine round: did the mandatory parts but **missed edge cases**; rejected | interviewdigest | Handle empty input, invalid input and boundary cases, and say them out loud. |
| SDE-III: "not being fast enough… the machine coding round has probably cost me" | snippet | Speed comes from practice: do 4–5 timed machine coding problems. |
| Offer rescinded after clearing everything: no degree | [29] | Not relevant to you, but have your documents ready. |
| Not the candidate's fault: hiring paused [16], interviewer an hour late [13], recruiter went silent for an agency candidate [11] | | Some rejections are luck. Keep applying elsewhere in parallel. |

### 3.2 Do and don't

**Do:**
- **Ask the recruiter for the Round 1 format and the allowed languages.** They may not tell you otherwise [4].
- **Code DSA in Java (or Go).** 2026 reports restrict rounds to these [16][17].
- **Talk through brute force → optimised**, with time and space complexity at each step [1][18][26].
- **Think aloud.** Interviewers give hints when they can follow your thinking [3][30].
- **Expect more questions than announced**, and keep moving [4].
- **In machine coding:** clarify requirements in 5–10 minutes, list the classes, then code. **Get a working core running early**, then add optional features one by one [4][7][19].
- **Write proper classes and interfaces even for small problems** (ASDE R1 "stack using array" wanted that) [16].
- **Test your code** with the interviewer's cases and your own edge cases (empty, one element, duplicates, negatives) [4].
- **Know your resume deeply.** One candidate called it "my identity in an interview" [26]. Expect "what broke in production and how did you fix it?" [18]
- **Prepare indexing, TCP/HTTP/REST and process vs thread.** All were asked of ASDE [20].
- **Practise SQL joins and GROUP BY** on LeetCode [2].
- **Read recent questions** on LeetCode Discuss and GFG. The time from OA to interview can be short [26].
- **Ask HR whether the offer is full time or an internship** [22].

**Don't:**
- Don't spend 20 minutes drawing UML in a machine coding round. A quick list of classes is enough; then code [19].
- Don't say "this can't be done" without proof [3].
- Don't fake not knowing a problem [3].
- Don't stay silent for long stretches.
- Don't claim a skill you can't talk about for two minutes. "I've used it at a basic level, for X" is fine.
- Don't ignore the edge cases [interviewdigest].

### 3.3 Your priority order for Round 1 (from the frequencies above)

1. **DSA in Java:** stack (parentheses, min stack, stack via array, next greater), intervals, binary search (LC 34), trees (LCA, build tree), BFS/Dijkstra, sliding window, knapsack and grid DP, LRU. → `02`
2. **Your resume:** Myntra and Atlassian end to end; the 60-second intro; "Why Swiggy". → `06`
3. **CS fundamentals:** process vs thread, deadlock, mutex vs semaphore, indexing, ACID, joins, SQL vs NoSQL, TCP vs UDP, REST, hashing vs encryption, JWT. → `04`
4. **Java rapid-fire:** interface vs abstract class, `final`, `static`, exceptions, Collections, HashMap internals, GC, threads, Spring basics. → `05`
5. **LLD / machine coding:** Splitwise, Snake & Ladder, food cart with discounts (Strategy), driver–order assignment, follow/unfollow schema + APIs. → `03`
6. **SQL:** LEFT JOIN + GROUP BY + COUNT, anti-join, HAVING, second-highest, window functions. → `04`

---

## Part 4: Sources

F = read in full. S = search-result text only (LeetCode, Glassdoor, Medium, Code360 and AmbitionBox blocked automated reading; most LeetCode posts were read through full-text mirrors on interviewexperiences.in). Reddit r/developersIndia and YouTube gave nothing usable.

**GeeksforGeeks** (all pages show "last updated 23 Jul 2025", which is not the interview date)
- [1] F https://www.geeksforgeeks.org/?p=598921 — SDE-1 off-campus 2021: 3 DSA rounds, Google Sheets HLD, offer.
- [2] F https://www.geeksforgeeks.org/?p=703061 — SDE-1 on-campus 2021: HackerRank 90 min, 1 SQL + 3 coding; parentheses / min stack; selected.
- [3] F https://www.geeksforgeeks.org/?p=732529 — SDE-1 on-campus 2021: fountains / weighted tree / SQL OA; waitlisted.
- [4] F https://www.geeksforgeeks.org/?p=728763 — SDE-1 off-campus 2021: Snake & Ladder machine coding; offer.
- [5] F https://www.geeksforgeeks.org/?p=596168 — SDE-1, undated: binary strings DP, OOP/ACID, URL/load balancer; selected.
- [6] F https://www.geeksforgeeks.org/?p=862699 — SDE on-campus 2021: 4-question coding round, Google Doc, OOP pillars, DBMS; selected.
- [7] F https://www.geeksforgeeks.org/interview-experiences/swiggy-interview-experience-for-sde-1-2/ — SDE-1 Jan 2021, 2+ YOE: calendar machine coding, 3 hr.
- [8] F https://www.geeksforgeeks.org/?p=528154 — SDE-1, 1.5 YOE: alien dictionary, f6→f12, 6/9/20 packets, medicine inventory.
- [9] F https://www.geeksforgeeks.org/swiggy-interview-experience-set-2-on-campus/ — old campus report: stockmax / robot OA, heavy OS/DBMS/networks.
- [10] F https://www.geeksforgeeks.org/?p=135209 — SDE-1 2015: binary-tree-tuples machine coding, LRU discussion.
- [11] F https://www.geeksforgeeks.org/?p=679932 — SDE-2: min-sum merge, OYO OOD (context only).

**Jointaro**
- [12] F https://www.jointaro.com/interviews/companies/swiggy/experiences/associate-software-development-engineer-bengaluru-karnataka-november-1-2025-accepted-offer-neutral-ff5fab1a — ASDE campus, Nov 2025: MCQ + 2-coding OA; OS/JWT/OOP round; HM; HR; offer.
- [13] F https://www.jointaro.com/interviews/companies/swiggy/experiences/associate-software-development-engineer-kanpur-uttar-pradesh-july-1-2024-no-offer-negative-d1fd56f0 — ASDE July 2024: no offer.
- [14] F https://www.jointaro.com/interviews/companies/swiggy/experiences/sde-1-bengaluru-december-1-2024-accepted-offer-neutral-fbb9ae78/ — SDE-1 Dec 2024: nice substrings / API / hard SQL OA; Job Scheduler; offer.
- [15] F https://www.jointaro.com/interviews/companies/swiggy/experiences/software-development-engineer-sde-india-october-13-2025-accepted-offer-neutral-429a4417 — SDE frontend Oct 2025.

**interviewexperiences.in** (full-text mirrors of LeetCode Discuss posts)
- [16] F https://interviewexperiences.in/experience/swiggy/swiggy-asde-backend-interview-experience — ASDE Backend Mar 2026: 3-DSA OA; R1 first/last occurrence + stack via array; hiring paused.
- [17] F https://interviewexperiences.in/experience/swiggy/swiggy-sde-1-interview-experience — SDE-1 Mar 2026: Java/Go-only OA; third-party DSA + Java round; driver–order LLD; joined.
- [18] F https://interviewexperiences.in/experience/swiggy/swiggy-sde-1-interview-experience-dec25-backend-engineer-sde-1-interview-leetcode-bangalore and https://takeuforward.org/community/interview-experiences/swiggy-sde-1-interview-experience-backend-engineer-bangalore — SDE-1 backend Dec 2025: LC 3 / LC 743 / Nth root OA; LC 96; Java/Spring; food-ordering LLD; Coin Change II in HM; selected.
- [19] F https://interviewexperiences.in/experience/swiggy/swiggy-sde-1-interview-experience-a-very-high-expectation-in-lld-round — SDE-1 Oct 2025: CF 1133C + LC 2035; Splitwise LLD; rejected.
- [20] F https://interviewexperiences.in/experience/swiggy/swiggy-asde-online-interviews-december-2024-offer — ASDE Dec 2024: HackerRank DSA + SQL + MCQ; arena take-home; follow/unfollow LLD; HM; offer.
- [21] F https://interviewexperiences.in/experience/swiggy/swiggy-asde-hackerrank-test-experience-cleared-round — ASDE frontend OA July 2024.
- [22] F https://interviewexperiences.in/experience/swiggy/swiggy-asde-associate-software-development-engineerasde-2022-passout — ASDE 2022 passout: SQL + coding + MCQ OA; arena take-home; internship-conversion offer.
- [23] F https://interviewexperiences.in/experience/swiggy/swiggy-sde-1-recro-software-engineer-interview-process — SDE-1 (RECRO) Jan 2024: currency conversion; offer.
- [24] F https://interviewexperiences.in/experience/swiggy/swiggy-sde-1-bangalore-remote-aug-2024-offer-accepted — SDE-1 Aug 2024: Valid Parentheses + Meeting Rooms; Instagram APIs; bike rental; offer.
- [25] F https://interviewexperiences.in/experience/swiggy/swiggy-sde-1-offer-bengaluru — SDE-1 May 2025: phone screen + onsite; food-order matching; offer.
- [26] F https://interviewexperiences.in/experience/swiggy/sde-intern-interview-experience-swiggy — SDE intern Jan 2025: greedy/DP OA, trie, BFS + heap; converted.
- [27] F https://interviewexperiences.in/experience/swiggy/swiggy-sde-1-backend-bangalore-september-2021-offer — SDE-1 backend Sep 2021: OA, parking lot machine coding, Rotting Oranges; offer.
- [28] F https://interviewexperiences.in/experience/swiggy/swiggy-all-interview-questions — 2022 compiled list of 24 Swiggy SDE-1/2 onsite questions.
- [29] F https://interviewexperiences.in/experience/swiggy/swiggy-sde-1-interview-android — SDE-1 Android May 2025: library machine coding, Edit Distance, Next Permutation; offer rescinded.

**Other first-hand reports**
- [30] F https://devsunite.com/blog/my-swiggy-sde-1-android-interview-experience — SDE-1 Android: LL k-group / Floyd; delivery-zone OA; offer.
- [31] S https://leetcode.com/discuss/post/5790100/Swiggy-or-SDE-1-or-Accepted-or-1-yoe/ — SDE-1 July 2024: min platforms, trapping rain water, Splitwise LLD; accepted.

**Glassdoor, X, Code360** (search text only)
- [32] S https://www.glassdoor.co.in/Interview/Swiggy-ASDE-Interview-Questions-EI_IE952680.0,6_KO7,11.htm — ASDE Nov–Dec 2025: graphs, LRU, merge intervals; postfix; reverse odd LL nodes.
- [33] S https://www.glassdoor.co.in/Interview/Swiggy-aSDE-Backend-Interview-Questions-EI_IE952680.0,6_KO7,19.htm — aSDE Backend June 2024: HackerRank 4 Qs, Magical Game Arena, LLD, HM.
- [34] S https://www.glassdoor.ca/Interview/Swiggy-Associate-Software-Engineer-Interview-Questions-EI_IE952680.0,6_KO7,34.htm — Associate SE 2025: HR → OA → DSA → LLD → HLD → HM; rate limiter LLD.
- [35] S https://www.glassdoor.com/Interview/Swiggy-Interview-E952680-RVW98242993.htm — on-campus Dec 2025: OA (DSA + CS fundamentals), DSA, HLD/LLD.
- [36] S https://www.glassdoor.com/Interview/Swiggy-SDE-1-Interview-Questions-EI_IE952680.0,6_KO7,12.htm — SDE-1 reviews Dec 2023 / Feb 2026.
- [37] S https://twitter.com/TautikA/status/1705892530322088107 — SDE intern backend 2023: OA with 0-1 knapsack.
- [38] S https://www.naukri.com/code360/interview-experiences/swiggy/swiggy-interview-experience-by-akshat-aggarwal-off-campus-sep-2023 — backend intern Sep 2023: 0/1 knapsack.
- [39] S https://www.naukri.com/code360/interview-experiences/swiggy/swiggy-interview-experience-jan-2022-exp-0-2-years — 0–2 YOE Jan 2022: 120-min Splitwise machine coding as Round 1.

**Compiled and snippet sources**
- [40] F https://www.placementpreparation.io/blog/swiggy-interview-questions-and-experience/ — two condensed SDE-1 reports (Snake & Ladder; Splitwise; LC 646 / LC 32).
- [41] S https://leetcode.com/discuss/interview-experience/1172971/swiggy-sde-1-bangalore-january-2021-offer — SDE-1 Jan 2021: HackerRank 1.5 hr, 3-hr machine coding.
- [41b] S https://leetcode.com/discuss/interview-experience/1064610/swiggy-sde-1-india-jan-2021-reject — SDE-1 Jan 2021 reject.
- [42] S https://medium.com/@adarsingh2002/my-interview-experience-with-swiggy-for-the-asde-ios-role-4ded6cedc15e and https://medium.com/@js.saini267/my-swiggy-experience-asde-frontend-role-5745210d006a — ASDE iOS / frontend 2025.
- [43] S https://ambitionbox.com/interviews/swiggy-interview-questions/sde1 — Splitwise first round; "one thing you don't like about Swiggy".
- [44] S https://github.com/TanishDusane/Swiggy-aSDE---Round-1-coding-test-Magical-Arena and https://github.com/rahulvj-21/Swiggy-aSDE---Round-1-coding-test — candidate solutions to the arena assignment (not official).
- [45] S https://leetcode.com/discuss/post/6334442/swiggy-sde-1-frontend-interview-experien-u08h/ — SDE-1 frontend Dec 2024.
- [46] S https://leetcode.com/discuss/interview-experience/6259176/Swiggy-Instamart-or-Round-2-or-SDE-1-or-Interview-Experience — Instamart SDE-1: 2 DSA in Java.
- interviewdigest.me/swiggy — aggregator; one undated ASDE Android machine-round log (rejected).

**Unknowns (no source covers them):** official ASDE OA duration or cut-off; the exact Magical Arena rules; whether CoderPad is used. The food-ordering problem was reported as an LLD discussion, not a timed machine coding task, and the rate limiter only as a one-line Glassdoor mention.
