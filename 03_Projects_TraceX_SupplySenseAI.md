# Projects: TraceX and SupplySenseAI

> About 25–30 minutes. Read it top to bottom; each part builds on the one before. Facts come from reading both GitHub repos. Where the README claims something the code doesn't do, this file says so — an interviewer who opens GitHub sees the code.
> Zomato won't grade the ML. They'll ask about architecture, APIs, data, graph algorithms, scaling and *why you chose X*. Lead with those.

---

## TraceX: catching money laundering with graphs

### What problem it solves

Criminals **launder** money: they push dirty money through many accounts until its origin is hidden and it looks clean. Banks must catch this by law — that's **AML** (anti-money laundering). Their systems raise thousands of **alerts** ("this account looks suspicious"), and human **investigators** check each one, tracing where the money came from and went, mostly by hand. Most alerts turn out to be nothing (**false positives**), so most of that time is wasted.

TraceX does four things for them: finds laundering patterns automatically, ranks alerts so the riskiest come first, turns them into **cases** assigned to investigators, and gives each investigator a workspace showing the money trail as a picture, with an AI assistant that summarises the case.

It was built for the **Union Bank of India iDEA hackathon** (top 10 of 27,000+ teams), then rebuilt properly as a production-style system with 500+ automated tests.

**Ownership — get this right.** It's a two-person project with Vedansh Kapoor. Say "I built" only for Vaideeka's parts: **login and permissions, the detection engine's port into the new system (including saving the ranking model's learning), alert generation, the case workflow, automatic assignment and deadlines, similar-case search, and the whole frontend.** For the AI assistant and reports: "my teammate built that, and here's how it works."

### Why a graph

A **graph** is just dots and arrows. Each dot (**node**) is a bank account; each arrow (**edge**) is a transfer, with a direction, amount and time. Laundering is about *relationships* — chains, loops, one account spraying money to many — so you can't see it by looking at transactions one at a time. Draw them as a graph and the patterns become shapes you can search for with standard algorithms.

The graph library is **NetworkX** (Python), which ships with those algorithms built in. It's wrapped behind a small interface (`GraphStore`), so a proper graph database like **Neo4j** could replace it later without touching the detection code.

### The six patterns it detects

- **Layering** — money hops quickly through many accounts to hide where it came from. TraceX starts at an account and follows the next outgoing transfer forward in time, never revisiting an account (a depth-first walk). A chain of 3+ hops within 2 hours, where at least half the money keeps moving, gets flagged.
- **Round-tripping** — money leaves and comes back to where it started, through middlemen. That's a **cycle** in the graph. Flagged if 85%+ of the money returns.
- **Structuring** — many deposits just under the ₹10 lakh reporting limit, to dodge it. Count transactions of ₹9–10 lakh within a time window.
- **Dormancy burst** — an account quiet for 6+ months suddenly gets busy. Find the longest gap, compare activity before and after.
- **Profile mismatch** — far more money than the person earns. Compare volume with declared income, and with people in the same job and income bracket.
- **Fan-out / fan-in** — one account sending to many, or collecting from many. Count each account's connections (its **degree**).

### Finding cycles fast — the best DSA talking point

Searching for every cycle in a 5-million-transaction graph is hopeless — the number of possible cycles explodes. The trick uses **strongly connected components (SCCs)**: groups of nodes where every node can reach every other by following arrows.

The key fact: **a cycle can only exist inside one SCC.** If A can reach B and B can reach A, they're in the same SCC; if not, no cycle can contain both. So:
1. Compute the SCCs first. That's fast — O(V + E) — with **Kosaraju's algorithm** (one DFS to order the nodes, a second DFS on the graph with all arrows reversed) or **Tarjan's** (a single DFS).
2. Throw away everything outside SCCs. Most of the graph disappears.
3. Search for cycles only inside SCCs of size 2–500, with a maximum cycle length and a cap of 2,000 cycles.

Small example: transfers A→B, B→C, C→A, C→D. SCCs are {A, B, C} and {D}. D can't be on any cycle, so you never search it. Be ready to code Kosaraju or Tarjan (`01b` has it).

### How alerts are scored

Each account gets a risk score out of 100, from three signals:

**Machine learning.** A model learns from past examples labelled "launderer" or "not" and then predicts for new accounts. TraceX uses **XGBoost**, which builds hundreds of small decision trees (each a chain of yes/no questions like "more than 20 counterparties?") one after another, each new tree fixing the mistakes of the previous ones. It's the go-to model for table-shaped data. It reads 28 numbers per account — amounts, counterparties, timing and so on, called **features** — and outputs a fraud probability.

**Patterns.** Each detected pattern adds points (round-trip 30, layering 25, …).

**Graph position.** Accounts that receive a lot of money, or sit between many others, score higher. A bonus is added when ML and patterns agree.

### Be honest about the model's numbers

On the public IBM money-laundering dataset (5 million transactions, but only **0.48%** of accounts are launderers) the model gets **F1 = 0.21** and **AUC = 0.78**. Know what these mean:
- **Precision** — of the accounts the model flagged, what fraction were really fraud. **Recall** — of all the real fraud, what fraction it caught. Roughly: flag 100 accounts and 25 are real → precision 0.25.
- **F1** combines precision and recall into one number. **AUC** measures how well the model ranks fraud above non-fraud: 0.5 is a coin toss, 1.0 is perfect.

How to defend it: *"With 0.48% positives, accuracy is meaningless — predicting 'nobody is a launderer' scores 99.5%. The ML is one of three signals, and the output is a ranked queue for humans to check, not an automatic verdict. Next I'd add better graph features and better labels."* Don't quote the old hackathon numbers (F1 0.68) — they weren't reproduced on the rebuilt code.

### Ranking that learns from investigators (LinUCB)

Scores are a starting point, but investigators know best. TraceX ranks the alert queue with a **contextual bandit** called **LinUCB** — a method that keeps choosing and learns from the outcome of each choice.

Each alert is described by 16 numbers (its *context*). LinUCB scores it as *what I predict this alert is worth* **plus** *a bonus for how unsure I am about alerts like this*. The first part **exploits** what it has learned; the bonus makes it **explore** unfamiliar alerts so it doesn't get stuck on its early guesses. When an investigator closes a case, it updates: **+1** if it was real fraud, **−0.3** if it was a false alarm. Over time the kinds of alerts investigators confirm rise to the top.

Vaideeka's fix here: in the hackathon version the bandit's learned values lived only in memory and vanished on every restart. She saved them to the database so learning persists.

If asked "is this reinforcement learning?" — be honest: it's a bandit, which makes one independent decision at a time with instant feedback. Full RL plans over sequences of steps. A bandit is the right tool here because each alert is its own decision.

### The application: how the pieces connect

```
Browser
   │
Next.js frontend (pages + its own server routes = the "BFF")
   │ server-to-server calls
FastAPI backend (Python)
   ├─ foundation:     login, roles, config checks
   ├─ investigation:  cases, workflow, assignment, alerts, graph views, reports
   ├─ detection:      6 detectors, rules, features, ML scoring, bandit
   └─ orchestration:  AI assistant (teammate)
   │                              │
SQL database + audit log          LLM (via OpenRouter)

Offline scripts: load data → train model → run detection → create cases
```

The **frontend** is the part users see. It's built with **React** (UIs made of reusable components) using **Next.js**, which adds page routing and lets you write server-side code alongside the pages. The **backend** holds the logic and data. It's built with **FastAPI**, a Python web framework — chosen because the ML and graph code is already Python, so it runs in the same process; FastAPI also checks incoming data automatically and generates API documentation.

**BFF (backend-for-frontend):** the browser never talks to FastAPI directly. It talks to Next.js's own server routes, which call FastAPI on its behalf. That keeps the login token on the server side and lets one page combine several backend calls.

**Database:** **SQLAlchemy** lets the code work with tables as Python objects; **SQLite** is a zero-setup database in a single file. Because of SQLAlchemy, moving to **Postgres** is a config change. **Alembic** keeps a versioned history of schema changes (*migrations*).

### Login: hashing, JWT and cookies (Vaideeka's work)

**Passwords** are never stored as-is. They're stored as a **hash** — the result of a one-way scrambling function. You can check a typed password by hashing it and comparing, but you can't turn the hash back into the password. TraceX uses **bcrypt**, which is deliberately slow, so an attacker who steals the hashes can't try billions of guesses.

After login, the server gives you a **JWT (JSON Web Token)**: a small piece of text with three parts, `header.payload.signature`. The payload says who you are and your role; the signature is made with a secret only the server knows. On every request, the server just checks the signature — no need to look you up in a session table. So the server is **stateless**: any server copy can handle any request. Tokens expire after 8 hours.

Where the token lives matters. It's kept in an **httpOnly cookie** — a cookie the browser sends automatically but JavaScript on the page can't read. So if an attacker manages to inject a script into the page (**XSS**, cross-site scripting), they still can't steal the token.

One subtle defence: if someone tries a username that doesn't exist, the code still runs bcrypt on a dummy hash. Every failed login takes the same ~0.4 s, so an attacker can't work out which usernames are real from how fast the reply comes back (a **timing attack**).

JWT weaknesses to mention if asked: a token can't be cancelled before it expires (use short expiry with refresh tokens, or a block list), and cookies can be abused by forged requests from other websites (**CSRF** — prevented with `SameSite` cookies or CSRF tokens).

### Permissions and the case workflow (Vaideeka's work)

**RBAC (role-based access control)** means permissions belong to roles — investigator, supervisor, admin — not to individual people. On top of that, every case endpoint checks the user is allowed to see *that* case, and every account endpoint checks the account belongs to that case. So nobody can read another case by changing a number in the URL.

Each case moves through a **finite state machine (FSM)** — a fixed list of states and the moves allowed between them:

```
NEW → ASSIGNED → IN_PROGRESS → AWAITING_REVIEW / ESCALATED / CLOSED_FP
AWAITING_REVIEW or ESCALATED → CLOSED_TP / CLOSED_FP / MONITORING
```
(TP = true positive, real fraud. FP = false positive, a false alarm.)

Only one function in the whole codebase can change a case's status, and it rejects any move not on the list — a closed case can't quietly reopen — and records every change. This is the same idea as a Zomato order: placed → accepted → preparing → picked up → delivered or cancelled.

**Auto-assignment:** each detection run creates cases for the top 20 alerts and gives each to the investigator with the **fewest open cases**, with a deadline (an **SLA**, service-level agreement) based on priority. The Zomato version is assigning an order to the nearest, least-busy rider — a min-heap keyed on (distance, load).

### Similar cases (Vaideeka's work)

Every closed case is saved as a list of 16 numbers — a **vector**. For a new case, TraceX finds closed cases whose vectors point in the most similar direction, using **cosine similarity** (the cosine of the angle between two vectors: 1 means same direction). Investigators see how similar cases were resolved.

Honest weakness: all similarities came out above 0.99, so it barely tells cases apart. The reason is that big-number features like amounts dominate the angle. The fix is to put every feature on the same scale first — subtract its mean and divide by its spread (a **z-score**).

### The frontend (Vaideeka's work)

She built all of it: the dashboard; the first-level triage screen (L1 — quick check of an alert) and the deep investigation screen (L2); and the interactive money-flow graph, drawn with **Cytoscape** (a graph-drawing library) and laid out left-to-right with **dagre**. **Zustand**, a small state library for React, remembers the investigator's open tabs.

### The teammate's parts (know how they work)

- **Tamper-evident audit log.** Every write adds a log row whose fingerprint (a **SHA-256** hash) is computed from the row *and* the previous row's fingerprint. Edit any old row and every fingerprint after it stops matching — the same idea as a blockchain. Weakness: it's one global chain, so two servers writing at once could fork it; send audit writes through a single writer.
- **AI assistant with guardrails.** An **LLM** (large language model) writes case summaries and suggests next steps. It can only fetch data through tools locked to the current case, and every number it writes must match a value a tool really returned, or the answer is rejected. Personal data (PAN, Aadhaar, phone) is blocked before anything reaches the model. That's your answer to "what if the AI makes things up?".
- **Reports.** Suspicious-transaction reports as PDFs, with a SHA-256 hash so any later change is detectable.

### Numbers worth quoting

- **N+1 query fix: graph endpoint 325 ms → 96 ms.** An N+1 problem is when code runs one query to get a list, then one more query per item in it — N extra round trips. Fetching it all in one query fixes it. Same family as the Myntra 2→1 fix — a strong Zomato story.
- **Tests grew from 106 to 502+**, about **97% coverage** (share of code lines exercised by tests). **CI** (continuous integration) runs them on every change and fails the build below 90%.
- One detection run produced **44,790 alerts**.

### TraceX questions
- **"Walk me through it."** Problem → graph → six detectors → scoring and bandit ranking → cases and workflow → investigator workspace. Then: "I built auth, the case workflow and assignment, similar cases and the whole frontend."
- **"How do you find round-trips on 5M transactions?"** SCCs first, cycles only inside them, with length and count caps.
- **"Would NetworkX scale?"** No — it's in memory, in one process, and each server would hold a full copy. That's why it's behind an interface: swap in Neo4j, and only look at a rolling window such as the last 90 days.
- **"How would you make it real-time?"** Stream transactions through **Kafka** (a durable event log services read from), run cheap checks like structuring and fan-out on each event, and run the expensive cycle search on a schedule or only on the affected part of the graph.
- **"Hardest bug?"** The bandit losing its learning on restart (her fix), or the N+1 query.
- **"Did you use AI tools?"** Be upfront: "We used Claude Code for a lot of the implementation. I designed and reviewed it and can explain every part." Make sure that's true.
- **"What would you change?"** Actually use the Isolation Forest score (it's computed but not used in the final score — Isolation Forest is a no-labels method that flags points that are easy to separate from the rest), learn the score weights instead of hand-setting them, fix the similar-case scaling, make the audit log safe for concurrent writers.

---

## SupplySenseAI: predicting shipment delays

### What problem it solves

Indian logistics companies move goods between cities by road, sea and air. Floods, cyclones, port strikes, festival rush and monsoons cause delays — and one delay ripples outward: a closed port delays everything that passes through it. Big companies pay for tools like SAP to see this coming; smaller ones can't.

SupplySenseAI is a dashboard that gives each shipment a **delay-risk score (0–100) with its top three reasons**, pulls **live weather and disaster data**, shows how one disruption **spreads** through the network of cities, suggests **alternative routes**, and has a chat assistant for questions. It covers 20 hub cities and 8 carriers. It's a solo project — the repo credits only Vaideeka.

### How it's built

```
Next.js + React + TypeScript frontend        FastAPI backend (Python)          Outside data
(dashboard, map, cascade view, reroute, ─►   shipments, cascade, routes,  ◄──  OpenWeatherMap
 what-if simulator, chat)                    chat, resilience, alerts          GDACS (disasters)
 refreshes every 30–60 s                     ALL DATA HELD IN MEMORY           Gemini (chat)

Packaged with Docker, runs on Google Cloud Run
```

The frontend asks the backend for data over a **REST API** — plain HTTP requests to URLs like `/api/v1/shipments` (GET to read, POST to create). The map uses **Leaflet**, which needs the browser, so it's loaded only on the client side. Weather comes from **OpenWeatherMap**, disaster alerts from **GDACS**; each call has a timeout and a fallback, and results are cached (30 and 15 minutes) so the app doesn't ask again on every request. Chat uses **Gemini**, Google's LLM, with a simple keyword-based fallback if it's unavailable.

**Docker** packages the app and everything it needs into an *image*; a running copy is a *container*, and it behaves the same everywhere. **Google Cloud Run** runs containers for you — no servers to manage — adding copies as traffic grows and scaling to zero when idle, so it costs almost nothing. Here it runs 0–3 copies.

### Risk scores and their explanations

Each shipment is scored by one of two scorers. The code tries the ML one first and falls back to the rule-based one if the model isn't available.

The **ML scorer** is XGBoost again, reading 11 features: the carrier's late rate, transport mode, season, distance, how much buffer there is before the deadline, regional risk, month, day of week, high-value, express. The **rule-based scorer** adds up similar factors with fixed weights, plus extra risk if a live disruption is nearby.

The reasons come from **SHAP**. SHAP splits one prediction into how much each feature pushed it up or down — e.g. "risk 78: carrier's late rate +20, monsoon season +15, tight deadline +10". The top three are shown. It's based on *Shapley values* from game theory, a fair way to share credit among players, and it's exact and fast for tree models.

### The cascade: how a disruption spreads (BFS)

Cities are nodes and shipping lanes are arrows. To see how a disruption spreads, the code runs **BFS (breadth-first search)** from the affected city: first all its direct neighbours, then their neighbours, level by level.

Impact starts at the disruption's severity. At each hop it's multiplied by the lane's dependency weight (0.7–0.8) and by 0.85, so it fades with distance. For example, a port closure at full strength might reach the next city at about 0.6 and the one after at about 0.4 *(illustrative)*. It stops after 4 hops, or once impact drops below 0.05. For each affected city it estimates delay hours (port closure = 36 h × impact), plus revenue and customers affected.

Two improvements to raise yourself:
- The code removes items from the front of a Python list (`pop(0)`), which is O(n) each time. A `deque` (double-ended queue) does it in O(1).
- BFS keeps the *first* path that reaches a city, not the *strongest* one. To get the maximum-impact path, run **Dijkstra** on −log(weight): taking logs turns multiplying into adding, so the biggest product becomes the shortest path.

### Route suggestions

For a risky shipment the code builds three candidate routes — direct road, via another port, and air freight — and measures each on cost (₹45/km by road), time, carbon and risk. Distances use the **Haversine formula** (the distance between two latitude/longitude points over the Earth's curved surface), times about 1.3 for real roads.

Since cost is in rupees and time in hours, the metrics can't be added directly. So each is rescaled to 0–1 across the three candidates (**min-max normalisation**: cheapest = 0, dearest = 1). Then they're combined with the user's weights (defaults: time 35%, cost 25%, risk 25%, carbon 15%), and the lowest total wins.

Be precise if asked: the README says "Pareto-optimal", but the code ranks three fixed candidates by a weighted sum. Call it "a weighted multi-criteria ranking of three candidate routes".

### Other features

A **resilience score** (0–100) rates the whole network on alternative paths between cities, carrier diversity, geographic spread, time buffers and recovery speed. A **what-if simulator** lets you pick a city, disruption type and severity, runs the cascade and lists affected shipments.

### Weak spots — raise them before the interviewer does

1. **The model was trained on synthetic labels.** There was no real delivery data, so the training data was generated from a formula. The **92% accuracy** shows XGBoost can copy that formula — it says nothing about real-world accuracy. Say: "It proves the pipeline end to end — features, training, explanations, serving with a fallback. With real delivery outcomes I'd retrain and test on later dates." **Never present 92% as a real result.**
2. **Train/serve skew:** some features are computed differently at training time and at prediction time. Fix: one shared feature module used by both.
3. **All data lives in memory**, with no database. Each Cloud Run copy has different data, and a restart regenerates it. Production fix: Postgres (with PostGIS for location queries), Redis as a cache, stateless servers.
4. **No login, and a Gemini API key is hard-coded in `deploy.sh` in a public repo.** Say: "That was a mistake. The key should be rotated and stored in a secret manager, and I'd add secret scanning to CI." **Rotate it before the interview.**
5. **"Real-time" is the browser re-asking every 30–60 seconds** (polling). Next step: push updates with server-sent events or WebSockets, and a scheduled job to refresh the feeds.
6. **Slow spots:** some heavy calculations re-run on every request, and 20 weather calls run one after another at startup. Cache results and make the calls concurrently.
7. **The README lists libraries the code doesn't use** (SWR, Zustand, Recharts) and overstates a few things. Tidy it if you can.

### SupplySenseAI questions
- **"Walk me through it."** Problem → risk score with reasons → live data → cascade → reroute → chat; FastAPI + Next.js on Cloud Run.
- **"Explain the cascade and its complexity."** BFS with fading impact and two stopping rules: O(V + E) over the part it reaches. Then the deque and Dijkstra improvements.
- **"How would you find the best route on a real road map?"** Roads as a weighted graph, then **Dijkstra**, or **A\*** — Dijkstra steered by a straight-line distance estimate that never overestimates, so it still finds the best route. In production, use a routing engine such as OSRM or Google's Routes API.
- **"What if the weather API goes down?"** 5-second timeouts, cached values, rule-based fallback, mock chat. Better still: retries with growing waits, and a **circuit breaker** — after repeated failures, stop calling the API for a while and go straight to the fallback.
- **"How would you handle a million shipments a day?"** A database plus cache; shipment events through Kafka; recompute risk when something changes, not on every page load; stateless server copies; push updates to the UI; add login and keep each company's data separate.
- **"Why Cloud Run?"** No servers to manage, automatic scaling, near-zero cost when idle. Downsides: **cold starts** (the first request after idle is slow), and no shared memory between copies — exactly why the in-memory design breaks at scale.

---

## Both projects

**"Which are you proudest of?"** TraceX — a real multi-module system where she owned auth, the case workflow, assignment, the bandit fix and the entire frontend, with 500+ tests and a top-10 finish out of 27,000+ teams.

**"Why FastAPI in both?"** The ML and graph code is Python, so the model runs in the same process with no network hop; FastAPI adds async handling, automatic validation and API docs. Trade-off: Python's **GIL** lets only one thread run Python code at a time, so CPU-heavy work scales with more processes or servers, not threads.

**"SQL or NoSQL?"** TraceX's data is relational (cases, accounts, links between them) and needs transactions, so SQL. SupplySense has no database yet; it should use Postgres with PostGIS.

**Zomato links to drop into answers:** case state machine ↔ order lifecycle · least-busy assignment ↔ rider assignment · bandit ranking ↔ restaurant ranking · SLA deadlines ↔ delivery-time promises · N+1 fix ↔ fast menu reads · cascade BFS ↔ effect of a store or rider shortage on an area · Haversine ↔ ETAs · weather raising risk ↔ "rain mode" · API fallbacks ↔ graceful degradation when maps or payments are down.

**Before the interview**
- [ ] Rotate the Gemini key in SupplySenseAI's `deploy.sh` and remove it from the repo history.
- [ ] Check the demo links on the resume work.
- [ ] Draw both architectures from memory in under two minutes each.
- [ ] Be able to code Kosaraju or Tarjan SCC, BFS with a deque, and Dijkstra (`01b`).
