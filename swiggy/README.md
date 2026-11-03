# Swiggy ASDE / SDE-1, Round 1: start here

> **Who this is for:** Vaideeka. You're preparing for the first live interview round at Swiggy (backend). This assumes you're a beginner in these topics.
> **How these docs are written:** they teach you, they're not revision notes. Every term is explained the first time it appears, with a real-life example (usually Swiggy, a kitchen or a bank queue). Read them in order the first time.
> **Where the questions came from:** about 38 real Swiggy interview write-ups from 2021–2026 (GeeksforGeeks, LeetCode Discuss mirrors, Jointaro, Glassdoor, Code360, AmbitionBox). Every one is linked at the end of `01`.

---

## The files

| # | File | What it teaches | Time |
|---|---|---|---|
| 1 | `01_Process_and_Past_Questions.md` | How Swiggy's process works (OA, take-home, Round 1 formats), and **every question asked before**, ranked by how often, each with a short answer. Rejection reasons and tips. | 1.5 h |
| 2 | `02_DSA_From_Zero.md` | Big-O from zero, then each DSA pattern Swiggy asks (stack, intervals, trees, DP, graphs, sliding window, binary search, heap, linked list, LRU), then 31 Swiggy-asked problems fully solved in **Java**, with dry runs (all compiled and tested). | 6–8 h over several days |
| 3 | `03_Machine_Coding_and_LLD_From_Zero.md` | OOP, SOLID and the design patterns asked (Strategy, Singleton, Factory, Observer), a playbook for the 60–90 min round, and full working Java solutions: Snake & Ladder, Splitwise, food cart with discounts, driver–order assignment, Magical Arena | 5–6 h |
| 4 | `04_CS_Fundamentals_From_Zero.md` | Operating systems, DBMS, SQL queries, computer networks and security, each ending with the questions Swiggy asked plus model answers | 4–5 h |
| 5 | `05_Java_and_Spring_From_Zero.md` | Java for a C++ person, Collections for DSA, OOP in Java, exceptions, JVM/GC, HashMap internals, multithreading, Spring Boot. Ends with a 50-question rapid-fire bank. | 4–5 h |
| 6 | `06_Resume_Projects_Behavioural_for_Swiggy.md` | Your intro and "Why Swiggy?", every concept on your resume explained from zero, model answers for Myntra/Atlassian/projects, and STAR stories for the behavioural questions Swiggy asks | 3 h |

The older Zomato files in the parent folder go deeper on some topics. Use them only after the Swiggy file on the same topic:
- `../05_Myntra_Deep_Dive.md` for Myntra
- `../06_System_Design_Master.md` for system design, if a later round needs it

---

## What Round 1 will probably be (short version; details in `01`)

There isn't a single fixed format. Recent reports show four variants:

1. **DSA only:** 2 medium problems in about 60 minutes. Your code must run. *(Most common.)*
2. **DSA + Java rapid-fire** (SDE-1, 2026, third-party interviewer): about 30 minutes coding, then about 25 minutes of 20–25 quick Java questions (OOP, JVM, GC, Collections, threads).
3. **DSA + basic LLD** (ASDE, 2026): easy problems (first/last occurrence, stack using an array), then "show me classes and interfaces".
4. **LLD discussion** (ASDE, 2024): e.g. follow/unfollow. You talk through requirements → DB schema → API endpoints.

There's also **machine coding**: build a small working app (Splitwise, Snake & Ladder, food cart) in 60–120 minutes. It usually comes after DSA now, but older reports had it as the first round. Prepare for it in any case.

**Language:** 2025–26 reports say DSA rounds allow **only Java or Go**. Practise DSA in Java.

---

## Checklist: everything to know before Round 1

Tick these off. Bold items are the most asked.

**DSA (file 02)**
- [ ] Explain Big-O for any code you write
- [ ] **Stack:** valid parentheses, min stack, next greater element, stack using an array, postfix evaluation
- [ ] **Intervals:** merge intervals, meeting rooms, minimum platforms
- [ ] **Trees:** traversals, **LCA**, build a tree from preorder + inorder, count unique BSTs
- [ ] **DP:** grid paths, **0/1 and unbounded knapsack (coin change)**, edit distance
- [ ] **Graphs:** BFS, multi-source BFS (rotting oranges), Dijkstra (network delay), currency conversion
- [ ] Sliding window (longest substring without repeats), two pointers (3Sum, trapping rain water)
- [ ] Binary search: first/last occurrence, rotated array, Nth root
- [ ] Heap: connect sticks at minimum cost, running median
- [ ] Linked list: reverse in k-groups, start of a cycle
- [ ] **LRU cache** (the single most-quoted "design" question)

**Machine coding / LLD (file 03)**
- [ ] The 4 OOP pillars with examples; interface vs abstract class
- [ ] Strategy, Singleton, Factory, Observer: what each is and when to use it
- [ ] Type Snake & Ladder from scratch in under 60 min
- [ ] Type Splitwise from scratch in under 75 min
- [ ] Explain the food cart + discounts design (Strategy) and the driver–order matching design
- [ ] Talk through follow/unfollow: tables, indexes, endpoints, HTTP verbs

**CS fundamentals (file 04)**
- [ ] Process vs thread, mutex vs semaphore, **deadlock** (4 conditions), paging, TLB, LRU page replacement
- [ ] **Indexing** (what it is, how it works, when not to use it), **ACID**, joins, normalisation, SQL vs NoSQL
- [ ] Write: department-wise count including zero (LEFT JOIN), Nth highest salary, GROUP BY / HAVING
- [ ] **What happens when you type a URL**, TCP vs UDP, HTTP vs REST, status codes, load balancer
- [ ] Hashing vs encryption, salting, **JWT**

**Java (file 05)**
- [ ] Use ArrayList, HashMap, ArrayDeque, PriorityQueue (with a comparator) and TreeMap without looking anything up
- [ ] interface vs abstract class, checked vs unchecked exceptions, `final`/`static`/`volatile`
- [ ] How HashMap works inside
- [ ] Thread pools (`ExecutorService`), `synchronized`, a deadlock example
- [ ] Spring: dependency injection, beans and their lifecycle, `@RestController`, configuration

**You (file 06)**
- [ ] 60-second intro said out loud, under a minute
- [ ] "Why Swiggy?" and "one thing you don't like about Swiggy"
- [ ] Myntra and Atlassian: what you built, a production issue, what you'd do differently
- [ ] 5 STAR stories ready

---

## 10-day plan (squeeze or stretch it to fit your date)

| Day | Morning (new learning) | Evening (practice) |
|---|---|---|
| 1 | `01` fully: know the battlefield | `05` parts 1–2 (Java for DSA); solve Valid Parentheses and Merge Intervals in Java |
| 2 | `02`: Big-O, stack, intervals | Solve 4 problems from those patterns without looking |
| 3 | `02`: binary search, sliding window, two pointers | `06`: write and say your intro and "Why Swiggy?" |
| 4 | `02`: trees + linked list | `04` part A (OS) |
| 5 | `02`: DP from zero (grid, knapsack, edit distance) | `04` part B–C (DBMS + SQL); write the queries yourself |
| 6 | `02`: graphs, heap, LRU | `04` parts D–F (networks, security) |
| 7 | `03` parts 0–3 (OOP, SOLID, patterns, playbook) | `05` parts 3–7 (OOP, exceptions, JVM, HashMap, threads) |
| 8 | `03`: type Snake & Ladder yourself, timed (60 min) | `05` Spring + the rapid-fire bank out loud |
| 9 | `03`: type Splitwise yourself, timed (75 min); read the food cart and driver solutions | `06`: Myntra/Atlassian questions and STAR stories out loud |
| 10 | Mock: 2 random problems from `01`'s list in 60 min, in Java, speaking aloud | Checklist above; reread "rejection reasons" in `01` |

**Rules for practising:**
1. Never just read a solution. Close the doc and type it.
2. Speak while you code, even when you're alone. The interviewer grades your thinking, not only the final code.
3. Every problem: brute force first → better idea → code → dry run on a tiny example → edge cases (empty, one element, duplicates, negatives).

---

## On the day
- Ask which language is allowed and whether the code needs to run.
- For DSA, restate the problem and give an example back before coding.
- For machine coding, get a **working core first**, then add extras. A candidate was rejected in 2025 for a half-finished Splitwise.
- If you don't know something, say what you do know and how you'd find out. Don't bluff, and don't pretend you haven't seen a problem you have (a candidate was waitlisted for that).
