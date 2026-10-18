# Projects: TraceX and SupplySenseAI

> **Reading time:** about 25 minutes. Written for someone seeing these projects for the first time. Every technical word is explained the first time it appears.
> **Basis:** both GitHub repos were read in full; facts and numbers come from the code. Where the README says something the code doesn't do, this file tells you, because an interviewer who opens GitHub sees the code.
> **Zomato framing:** they won't grade the ML. They'll ask about architecture, APIs, data modelling, graph algorithms, scaling and *why you chose X*. Lead with that.

---

# Part 1: TraceX

## 1.1 The problem, in plain words

Criminals **launder** money: they move dirty money through many bank accounts until it looks clean. Banks are legally required to catch this (*AML*, anti-money laundering). Their systems raise thousands of **alerts** ("this account looks suspicious"), and human **investigators** have to check each one — tracing where the money came from and went, usually by hand in spreadsheets. Most alerts turn out to be harmless (*false positives*), so investigators waste most of their time.

**TraceX helps investigators by:** finding laundering patterns automatically, ranking alerts so the riskiest are checked first, turning them into **cases** assigned to investigators, and giving each investigator a workspace that shows the money trail as a picture, with an AI assistant that summarises the case.

**Background:** built for the **Union Bank of India iDEA hackathon** (problem: "tracking of funds within a bank for fraud detection"), where the team placed **top 10 of 27,000+**. It was then rebuilt properly as a production-style system with 500+ automated tests.

**Ownership — say this clearly.** It's a two-person project with Vedansh Kapoor. Say "I built" only for your parts:
- Login and permissions (JWT auth and RBAC).
- Moving the detection engine into the new system, including making the ranking model's learning survive restarts.
- Alert generation, the case workflow, automatic case assignment and deadlines.
- Similar-case search.
- **The entire frontend** (Next.js).

For the AI assistant and report generation, say "my teammate built that; here's how it works."

## 1.2 Why a graph?

A **graph** is a set of **nodes** (here, bank accounts) connected by **edges** (here, money transfers, with direction, amount and time). Laundering is a *relationship* pattern — chains, loops, one account scattering money to many — so looking at one transaction at a time can't see it. As a graph, these patterns become standard algorithm problems.

The graph library is **NetworkX** (Python), which comes with ready-made graph algorithms. It's hidden behind a small interface (`GraphStore`) so that a real graph database such as **Neo4j** could replace it later without changing the detection code.

## 1.3 The six laundering patterns it detects

| Pattern | What criminals do | How TraceX finds it |
|---|---|---|
| **Layering** | Money hops quickly through many accounts to hide its origin | From an account, follow the next outgoing transfer forward in time, never revisiting an account (a depth-first walk). Flag chains of 3+ hops within 2 hours where at least half the money is passed on. |
| **Round-tripping** | Money goes out and comes back to where it started | Find **cycles** in the graph (explained below). Flag if 85%+ of the money returns. |
| **Structuring** | Many deposits just under the ₹10 lakh reporting limit | Count transactions of ₹9–10 lakh within a time window. |
| **Dormancy burst** | An account silent for 6+ months suddenly gets busy | Find the longest gap between transactions, then compare activity before and after. |
| **Profile mismatch** | Money far beyond what the person earns | Compare volume with declared income and with people in the same job and income bracket. |
| **Fan-out / fan-in** | One account sprays money to many, or collects from many | Count each account's incoming and outgoing connections (its *degree*). |

**The best DSA talking point: finding round-trips efficiently.** Searching every possible cycle in a 5-million-transaction graph is extremely slow (the number of cycles can grow exponentially). The trick:
1. A **strongly connected component (SCC)** is a group of nodes where every node can reach every other by following edges. **A cycle can only exist inside one SCC.**
2. So first compute SCCs — fast, O(V + E), with **Kosaraju's** algorithm (two depth-first searches, the second on the reversed graph) or **Tarjan's** (one DFS). Throw away everything outside them.
3. Search for cycles only inside SCCs of size 2–500, with a maximum cycle length and a cap of 2,000 cycles.

This turns an impossible search into a fast one. Be ready to code Kosaraju or Tarjan (`01b` has it).

## 1.4 How an alert is scored and ranked

**Step 1 — a risk score per account (0–100)** combines three signals:
- **Machine learning:** **XGBoost**, a model that builds hundreds of small decision trees one after another, each correcting the previous ones' mistakes. It's the usual best choice for table-shaped data. It reads 28 numbers describing each account (*features*: amounts, number of counterparties, timing…) and outputs a fraud probability.
- **Patterns:** each detected pattern adds a weight (round-trip 30, layering 25, …).
- **Graph position:** accounts that receive a lot of money or sit between many others score higher.
- A bonus when ML and patterns agree.

**Step 2 — ranking the alert queue with a contextual bandit (LinUCB).** A *bandit* is a learning method that chooses between options and improves from feedback. Each alert is described by 16 numbers (its *context*). LinUCB gives each alert a score = *predicted usefulness* + *a bonus for uncertainty*. The first part **exploits** what it has learned; the second **explores** alerts it isn't sure about, so it doesn't get stuck. When an investigator closes a case, the model is updated: **+1** if it was real fraud, **−0.3** if it was a false alarm. Over time, the kind of alerts that investigators confirm move to the top.

*Your contribution here:* in the hackathon version, the bandit's learned numbers lived only in memory and were lost on every restart. You saved them in the database, so learning persists.

**Honest numbers.** On the public IBM money-laundering dataset (5 million transactions, but only **0.48%** of accounts are launderers), the model's **F1 is 0.21** and AUC 0.78. Explain the terms and defend it:
- **Precision** = of the accounts flagged, how many were really fraud. **Recall** = of the real fraud, how much was caught. **F1** combines the two. **AUC** = how well the model ranks fraud above non-fraud (0.5 is random, 1.0 perfect).
- *Defence:* "With 0.48% positives, accuracy is meaningless — predicting 'never fraud' gets 99.5%. The ML score is one signal among three, and the result is a ranked queue for humans, not an automatic decision. Next I'd add better graph features and better labels." Don't quote the older hackathon numbers (F1 0.68); they weren't reproduced.

## 1.5 Architecture

```
Browser
   │
   ▼
Next.js frontend  ── its server-side routes act as a "BFF" ──┐
(pages: login, dashboard, case workspace)                    │ server-to-server calls
                                                             ▼
FastAPI backend (Python), four modules:
   foundation     login, roles, config checks
   investigation  cases, workflow, assignment, alerts, graph views, reports
   detection      the 6 detectors, rules, features, ML scoring, bandit
   orchestration  the AI assistant (teammate)
   │                                   │
   ▼                                   ▼
SQL database (SQLite; Postgres-ready)  LLM via OpenRouter
+ tamper-evident audit log

Offline scripts: load data → train model → run detection → create cases
```

The terms in that picture:
- **Next.js** is a framework on top of **React** (the library for building UIs from components). It adds page routing and lets you write server-side code next to the UI.
- **BFF (backend-for-frontend):** the browser never talks to FastAPI directly. It talks to Next.js's own server routes, which call FastAPI. The login token stays on the server side in a secure cookie (below), and one page can combine several backend calls into one.
- **FastAPI** is a Python web framework. Chosen because the ML and graph code is Python, so it runs in the same process; it also validates request data automatically and generates API documentation.
- **SQLAlchemy** is an *ORM* — it lets you work with database tables as Python objects. **SQLite** is a database stored in a single file, needing zero setup; because of SQLAlchemy, switching to **Postgres** is a configuration change. **Alembic** keeps versioned database schema changes (*migrations*).

## 1.6 What you built, explained

**Login with JWT.** A **JWT (JSON Web Token)** is a small signed piece of text the server gives you after login. It has three parts — `header.payload.signature`. The payload says who you are and your role; the signature is made with a secret key only the server knows. On every request the server checks the signature instead of looking you up in a session table, so the server stays **stateless** (any server can handle any request). TraceX's tokens expire after 8 hours.
- **Where the token is stored:** in an **httpOnly cookie** — a cookie the browser sends automatically but JavaScript on the page can't read. So even if an attacker injects a script into the page (**XSS**, cross-site scripting), they can't steal the token.
- **Passwords** are stored as **bcrypt** hashes. A *hash* is a one-way scramble — you can check a password against it but can't reverse it. bcrypt is deliberately slow, so guessing millions of passwords is impractical.
- **Timing-attack defence:** if someone logs in with a username that doesn't exist, the code still runs bcrypt on a dummy hash. Every failed login takes the same ~0.4 s, so an attacker can't tell which usernames are real from response times.
- **JWT weaknesses to know:** a token can't be cancelled before it expires (fix: short expiry plus refresh tokens, or a block list); cookies can be abused by forged requests from other sites (**CSRF**; fix: `SameSite` cookies or CSRF tokens).

**Permissions with RBAC.** **Role-based access control** means permissions attach to roles (investigator, supervisor, admin) rather than to individual users. On top of roles, every case endpoint checks the user may see *that* case, and every account endpoint checks the account belongs to that case. So one investigator can't read another case's data by changing a number in the URL.

**The case workflow as a state machine.** A **finite state machine (FSM)** is a fixed list of states and the allowed moves between them:

```
NEW → ASSIGNED → IN_PROGRESS → AWAITING_REVIEW / ESCALATED / CLOSED_FP
AWAITING_REVIEW or ESCALATED → CLOSED_TP / CLOSED_FP / MONITORING
```
(TP = true positive, real fraud; FP = false positive, false alarm.) Only one function in the codebase may change a case's status, and it rejects any move not on the list — so, for example, a closed case can't silently reopen. Every change is recorded in history. **Zomato link:** an order works the same way — placed → accepted → preparing → picked up → delivered or cancelled.

**Auto-assignment and SLAs.** The detection run creates cases for the top 20 alerts and gives each to the investigator with the **fewest open cases**. An **SLA** (service-level agreement) here is a deadline by which the case must be handled, set from its priority. **Zomato link:** assigning an order to the nearest, least-busy rider — a min-heap keyed by (distance, load).

**Similar cases.** Each closed case is stored as a list of 16 numbers (a *vector*). For a new case, TraceX finds closed cases whose vectors point in the most similar direction (**cosine similarity**, which measures the angle between two vectors: 1 = identical direction). *Honest weakness:* all similarities came out above 0.99, so it barely tells cases apart. The cause is that large-valued features (like amounts) dominate the angle. Fix: rescale each feature to a common scale first (z-score: subtract the mean, divide by the standard deviation).

**The frontend.** The dashboard, the L1 triage view (first-level quick check of an alert) and the L2 investigation view (deep dive), including the interactive money-flow graph drawn with **Cytoscape** (a graph-drawing library) and laid out left-to-right with **dagre**. **Zustand** stores the workspace's open tabs (it's a small state-management library for React).

## 1.7 Other parts worth knowing (built by the teammate)

- **Tamper-evident audit log.** Every write adds a row whose hash = SHA-256(row contents + the previous row's hash). **SHA-256** is a standard hash function. Because each row includes the previous hash, editing any old row breaks every hash after it — the same idea as a blockchain. *Weakness:* it's one global chain, so two servers writing at the same moment could fork it; fix by sending audit writes through a single writer.
- **AI assistant with guardrails.** An **LLM** (large language model) writes case summaries and suggests next steps. It's only allowed to get data through *tools* fixed to the current case, and every number it writes must match a value a tool actually returned, or the answer is rejected. Personal data (PAN, Aadhaar, phone numbers) is blocked before anything is sent to the model. This is the answer to "what if the AI makes things up?".
- **Reports.** Suspicious-transaction reports (STR) as PDFs, with a SHA-256 hash of the content so any later change is detectable.

## 1.8 Numbers you can quote

- **N+1 query fix: graph endpoint 325 ms → 96 ms.** An **N+1 query** problem is when code runs one query to get a list, then one extra query *per item* (N more). Fetching everything in one query fixes it. Great Zomato story — same class of fix as Myntra.
- **Tests: 106 → 502+**, about **97% coverage** (share of code lines run by tests); CI fails the build below 90%. **CI** (continuous integration) runs the tests automatically on every change.
- One detection run produced **44,790 alerts**.

## 1.9 Questions you'll likely get

1. **"Walk me through TraceX."** Problem → graph model → six detectors → scoring and bandit ranking → cases with workflow → investigator workspace. Then: "I built auth, the case workflow and assignment, similar cases and the whole frontend."
2. **"How do you find round-trips on 5M transactions?"** SCCs first, cycles only inside them, with length and count limits (section 1.3).
3. **"Would NetworkX scale?"** No — it's in memory, in one process, and each server would hold its own copy. That's why it's behind an interface: swap in Neo4j, and limit detection to a rolling window (e.g. the last 90 days).
4. **"How would you make detection real-time?"** Stream transactions through **Kafka** (an event log that services read from), run cheap checks (structuring counts, fan-out) on each event, and run the expensive cycle search on a schedule or only on the affected part of the graph.
5. **"How does auth work, and how do you stop someone seeing another case?"** JWT in an httpOnly cookie via the BFF, bcrypt passwords, roles, plus case-level and account-in-case checks.
6. **"Why a state machine?"** Illegal moves become impossible, there's one place that changes status, and history is clean.
7. **"The F1 is only 0.21 — is the model bad?"** Section 1.4 defence.
8. **"Is LinUCB really reinforcement learning?"** Be honest: it's a *contextual bandit* — one decision at a time with immediate feedback — not full RL, which plans over sequences of states. That's the right tool here because each alert is an independent decision.
9. **"Hardest bug?"** Bandit learning lost on restart (you fixed it), or the N+1 query.
10. **"Did you use AI tools?"** Be upfront: "We used Claude Code for a lot of implementation. I designed and reviewed the work and can explain every part." Then make sure that's true.
11. **"What would you do differently?"** Use the Isolation Forest score (it's computed but not used in the final score), learn the score weights instead of hand-setting them, fix similar-case scaling, make the audit log safe for concurrent writers.

---

# Part 2: SupplySenseAI

## 2.1 The problem, in plain words

Indian logistics teams move goods between cities by road, sea and air. Floods, cyclones, port strikes, festival rush and monsoons cause **delays**, and one delay ripples downstream — a closed port delays every shipment passing through it. Big companies have expensive tools (SAP) to see this coming; smaller ones don't.

**SupplySenseAI** is a dashboard that:
1. Gives each shipment a **delay-risk score from 0 to 100** and the top three reasons.
2. Pulls **live weather and disaster data** to know what's happening now.
3. Shows how one disruption **spreads** through the network of cities (the *cascade*).
4. Suggests **alternative routes**, ranked by cost, time, carbon emissions and risk.
5. Has a chat assistant that answers questions about the current situation.

It covers 20 hub cities and 8 carriers. It's a solo project: the repo credits only you.

## 2.2 Architecture

```
Next.js + React + TypeScript frontend          FastAPI backend (Python)             Outside data
(dashboard, Leaflet map, cascade view,  REST   routes: shipments, cascade,          OpenWeatherMap
 reroute, what-if simulator, chat)    ──────►  optimise, chat, resilience, alerts ◄─ GDACS (disasters)
 refreshes every 30–60 s                       services: risk scorer, cascade        Gemini (chat)
                                               engine, route optimiser, weather
                                               ALL DATA HELD IN MEMORY (no DB)
Packaged with Docker, deployed on Google Cloud Run
```

The terms:
- **REST API:** the frontend asks for data over HTTP at URLs like `/api/v1/shipments`, using GET to read and POST to create.
- **Leaflet** is a map library. It needs the browser's `window` object, so it's loaded only on the client side, not during server rendering.
- **OpenWeatherMap** gives weather per city; **GDACS** (Global Disaster Alert and Coordination System) gives live disaster alerts. External calls have timeouts and fallbacks, and results are cached for 30 and 15 minutes. A **cache** keeps a recent answer so you don't ask again every time.
- **Docker** packages the app with everything it needs into an *image*; a running copy is a *container*. **Google Cloud Run** runs containers without you managing servers, adds copies when traffic grows and scales to zero when idle, so it costs almost nothing. Here: 0–3 instances.
- **Gemini** is Google's LLM, used for the chat; if it's unavailable, a simple keyword-based reply is used instead.

## 2.3 How it works

**Risk scoring.** Each shipment gets a score from one of two scorers, tried in order (a *fallback*):
- **ML scorer:** XGBoost reads 11 features (carrier's late rate, transport mode, season, distance, how much buffer before the deadline, regional risk, month, day of week, high-value or express) and gives a delay probability × 100.
- **Rule-based scorer** (if the model isn't available): a weighted sum of similar factors, plus extra risk if a live disruption is nearby.
- **Explanations with SHAP.** SHAP splits one prediction into each feature's contribution — e.g. "risk 78: carrier late rate +20, monsoon season +15, tight deadline +10". The top three become the reasons shown. It's based on *Shapley values* from game theory: a fair way to share credit among players.

**The cascade (BFS).** Cities are nodes and shipping lanes are directed edges. **BFS (breadth-first search)** explores a graph level by level — first all direct neighbours, then their neighbours. Starting from the disrupted city:
- Impact starts at the disruption's severity.
- At each hop it is multiplied by the edge's dependency weight and by 0.85, so it fades with distance.
- It stops after 4 hops or when impact falls below 0.05.
- For each affected city, it estimates delay hours (e.g. port closure = 36 h × impact) and revenue and customers affected.

*Two improvements to raise yourself:*
- The code removes items from the front of a Python list (`pop(0)`), which is O(n) each time. A `deque` (double-ended queue) does it in O(1).
- BFS keeps the *first* path that reaches a city, not the *strongest*. To get the maximum-impact path, use Dijkstra's algorithm on −log(weight): multiplying weights becomes adding costs, and the biggest product becomes the shortest path.

**Route suggestions.** For a risky shipment, three candidates are built: direct road, a route via an alternative port, and air freight. Each is measured on cost (₹45/km by road), time, carbon and risk. Each metric is rescaled to 0–1 across the three (*min-max normalisation*), then combined with the user's weights (defaults: time 35%, cost 25%, risk 25%, carbon 15%). Lowest total wins.
- Distances use the **Haversine formula** — the straight-line distance over the Earth's curved surface between two latitude/longitude points — multiplied by about 1.3 for real roads.
- *Be precise:* the README says "Pareto-optimal", but the code ranks three fixed candidates by weighted sum. Say "weighted multi-criteria ranking of three candidate routes".

**Resilience score.** One 0–100 number for how robust the whole network is, from five parts: alternative paths between cities, carrier diversity (are you relying on one carrier?), geographic spread, time buffers, and recovery speed.

**What-if simulator.** Pick a city, a disruption type and severity; it runs the cascade and lists affected shipments and options.

## 2.4 Weak spots — raise them before they do

1. **The ML is trained on synthetic labels.** There was no real delivery data, so the training data was generated with a formula. The reported **92% accuracy measures how well XGBoost copies that formula, not real-world accuracy.** Say: "The model proves the pipeline end to end — features, training, explanations, serving with a fallback. With real delivery outcomes I'd retrain and validate on later dates." **Never quote 92% as a real result.**
2. **Train/serve skew.** Some feature values are computed differently at training time and at prediction time. Fix: one shared feature module used by both.
3. **Everything is in memory**, with no database. Each Cloud Run instance has its own different data, and a restart regenerates it. Production fix: Postgres (with PostGIS for location queries), Redis for caching, stateless servers.
4. **No login, and a Gemini API key is hard-coded in `deploy.sh` in a public repo.** Say: "That was a mistake. The key should be rotated and kept in a secret manager, and I'd add secret scanning in CI." **Rotate the key before the interview.**
5. **"Real-time" is the browser polling** every 30–60 seconds. Next step: push updates with server-sent events or WebSockets, and a scheduled job to refresh the feeds.
6. **Slow spots:** some heavy computations re-run on every request, and 20 weather calls are made one after another at startup. Fix: cache results and make the calls concurrently.
7. **README drift:** it lists libraries the code doesn't use (SWR, Zustand, Recharts) and overstates a few things. Tidy it if you can.

## 2.5 Questions you'll likely get

1. **"Walk me through it."** Problem → risk score with reasons → live data → cascade → reroute → chat; FastAPI + Next.js on Cloud Run.
2. **"Explain the cascade. What's its complexity?"** BFS with fading impact and two stopping rules, so O(V + E) over the reachable part. Then the deque and Dijkstra improvements.
3. **"How would you find the best route on a real road network?"** Treat roads as a weighted graph and run **Dijkstra**, or **A\*** (Dijkstra guided by a straight-line-distance estimate, which never overestimates and so still finds the best route). In production, use a routing engine such as OSRM or Google's Routes API.
4. **"Why XGBoost and SHAP?"** Small table-shaped data, fast, handles imbalance; SHAP gives per-shipment explanations users trust, and is exact and fast for tree models.
5. **"What if the weather API is down?"** 5-second timeouts, cached values, rule-based fallback, mock chat. Improvements: retries with increasing waits (*backoff*), and a **circuit breaker** — after repeated failures, stop calling the API for a while and use the fallback immediately.
6. **"How would you handle 1 million shipments a day?"** Database plus cache; shipment events through Kafka; risk recomputed on events rather than on every page load; stateless API copies; push updates to the UI; add login and per-company data separation.
7. **"Why Cloud Run?"** No servers to manage, automatic scaling, near-zero cost when idle. Downsides: *cold starts* (the first request after idle is slow) and no shared memory between instances — which is exactly why the in-memory design breaks at scale.

---

# Part 3: Both projects

**"Which project are you proudest of?"** TraceX — a real multi-module system where you owned auth, the case workflow, assignment, bandit persistence and the whole frontend, with 500+ tests and a top-10 finish out of 27,000+ teams.

**"Why FastAPI in both?"** The ML and graph code is Python, so the model runs in the same process with no network hop. FastAPI adds async request handling, automatic validation and API docs. Trade-off: Python's **GIL** lets only one thread run Python code at a time, so CPU-heavy work scales with more processes or servers, not threads.

**"SQL or NoSQL?"** TraceX: relational data (cases, accounts, links between them) that needs transactions — SQL. SupplySense: none today; it should use Postgres with PostGIS.

**Zomato links to drop into answers**

| In your project | At Zomato |
|---|---|
| Case state machine | Order lifecycle: placed → accepted → preparing → picked up → delivered |
| Assign to the least-busy investigator | Rider assignment: nearest, least-loaded rider |
| Bandit ranking from feedback | Ranking restaurants and recommendations |
| SLA deadlines | Delivery-time promises and breach alerts |
| N+1 fix, 325 → 96 ms | Speeding up hot reads such as menus |
| Cascade BFS | Effect of a store or rider shortage on an area's orders |
| Haversine and routes | ETAs and rider routing |
| Weather raising risk | "Rain mode" for delivery times |
| Fallbacks when APIs fail | Graceful degradation when maps or payments are down |

**Before the interview**
- [ ] Rotate the Gemini key in SupplySenseAI's `deploy.sh` and remove it from the repo history.
- [ ] Check the demo links on the resume work.
- [ ] Draw both architectures from memory in under two minutes each.
- [ ] Be able to code Kosaraju or Tarjan SCC, BFS with a deque, and Dijkstra (`01b`).
