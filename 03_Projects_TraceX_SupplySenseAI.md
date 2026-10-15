# Projects Deep Dive: TraceX & SupplySenseAI

> **Basis:** both repos were read in full: TraceX at commit `0e7217b`, plus all 3 commits of SupplySenseAI. Every algorithm, number and file path below is taken from the code or the repo's docs. Where the code **does not** match the README, this file says so. **An interviewer who opens your GitHub will see the code, not the README.**
>
> **Zomato framing:** Zomato won't grade the ML. They'll probe **architecture, data modelling, APIs, graph algorithms, scaling, and why you chose X over Y**. Lead with backend and graph work. Treat ML as one component.

---

# PART 1: TraceX, a graph-based AML investigation platform

## 1.1 Pitches

**30 seconds**
> "TraceX is an anti-money-laundering investigation platform for banks. Banks raise thousands of suspicious-transaction alerts, and investigators reconstruct money trails by hand. TraceX models transactions as a graph and detects laundering patterns like layering chains, round-trips and structuring. It scores accounts with an ensemble of XGBoost, pattern weights and graph features. It ranks alerts and auto-creates and assigns cases. Investigators get a case-scoped workspace with the money-flow graph, timeline, similar cases, and an LLM copilot whose every claim is validated against real data. We built it for the Union Bank of India iDEA 2.0 hackathon, where we placed top 10 of 27,000+ teams, and then rebuilt it as a production-style system with FastAPI, Next.js, 500+ tests and about 97% coverage."

**One-line problem statement:** "Every rupee leaves a trail. We make it visible."

## 1.2 Context you must know
- **Hackathon:** Union Bank of India × iDEA 2.0, problem statement PS3, "Tracking of Funds within Bank for Fraud Detection". This is the **iDEA Hackathon 2026 top 10** line on the resume. It was later pitched at Global Fintech Fest 2026.
- **Two systems live in the repo:**
  1. `archive/fund-flow-tracker/`: the **original hackathon build**. One large `api/server.py` with 45+ endpoints, Server-Sent Events (SSE) real-time demo, and Random-Walk-with-Restart accomplice discovery.
  2. `backend/` + `frontend/`: a **greenfield rebuild done in July 2026 in 22 phases.** This is the current system and what you should describe.
  - The **root README still describes the archived system.** If an interviewer quotes it, say: "The README is from the hackathon version. We rebuilt the system afterwards; the current backend is in `backend/`."
- **Team:** the commits split roughly half-and-half between **Vaideeka** and **Vedansh Kapoor**. The README also credits Adarsh Panjwani and Shourya Kalbande on the graph engine and backend.
- **Vaideeka's own PRs, the parts to claim with "I":**

| PR | What you built |
|---|---|
| #3 | **JWT auth, RBAC, secrets handling** |
| #4 | **Detection port:** ML ensemble, graph store, rule DSL, LinUCB bandit with DB persistence |
| #5 | **Alert generation, case FSM, workload-based auto-assignment + SLA, RL-ranked queue, audit trail** |
| #6 | Demo data studio |
| #9 | **Similar cases** (cosine over a 16-dimension vector), **Relationship Explorer**, path-recommendation plumbing |
| #10–#26 | **The entire frontend:** Next.js scaffold, BFF auth, dashboard, workspace shell (Zustand), L1 triage, L2 deep investigation (graph, replay, evidence, relationships), usability pass (dagre layout), copilot UI, STR UI, accessibility + frontend CI |
| #30 | GFF26 deck and voiceover |

- **Built by Vedansh,** which Vaideeka should explain at a high level as "my teammate built":
  - The data model and ingest.
  - The L1/L2 APIs.
  - The AI substrate: LLM gateway, tool catalog, grounding validator, PII gate.
  - The Recommendation Engine and Copilot.
  - The STR reports.
  - CI and deploy.

> **Interview rule:** say "I built X" only for your PRs. For the AI layer, say "my teammate built the AI substrate; I integrated it into the frontend and know how it works: …". Interviewers respect clear ownership. Over-claiming falls apart on the second follow-up.

## 1.3 Tech stack and why each piece is there

| Layer | Technology | Why (say this) |
|---|---|---|
| API | **FastAPI** (≥0.115), Uvicorn | Python, so the ML and graph code (pandas, NetworkX, XGBoost) runs in-process. Pydantic validation and auto-generated OpenAPI docs. |
| ORM / DB | **SQLAlchemy 2**, **Alembic** (2 migrations), **SQLite** (Postgres URL supported) | SQLite gives zero-ops pilot/dev. SQLAlchemy keeps it database-portable. Alembic versions the schema. |
| Auth | python-jose (JWT HS256, 480-min expiry), passlib + bcrypt (pinned below 4.1) | Stateless auth. bcrypt is slow by design, which resists brute force. |
| Graph | **NetworkX** (MultiDiGraph) behind a **`GraphStore` interface** | Fast to build with and rich in algorithms (SCCs, simple_cycles, ego-graph). The interface allows swapping to **Neo4j** in production. |
| ML | **XGBoost** (supervised), **IsolationForest** (unsupervised), scikit-learn, joblib | XGBoost is the best fit for imbalanced tabular data. IsolationForest finds anomalies without labels. |
| Ranking | **LinUCB contextual bandit** | Learns alert priority from investigator feedback (explore vs exploit). |
| LLM | OpenAI SDK pointed at **OpenRouter** (default Claude Sonnet 4.5) | An OpenAI-compatible gateway means switching to an on-prem model (vLLM/Ollama) is just a `base_url` change, which matters for banks. |
| Reports | reportlab | STR/SAR PDF generation |
| Frontend | **Next.js 16** (App Router), React 19, TypeScript, Tailwind 4, shadcn | Typed UI; a server-side **BFF** layer (below). |
| Graph UI | **Cytoscape** + **dagre** layout; Recharts; **Zustand** | Cytoscape renders large graphs interactively; dagre lays out left-to-right flows; Zustand holds workspace tab state. |
| Quality | ruff, mypy, pytest, **coverage gate at 90%**; GitHub Actions for backend and frontend | 500+ tests, about 97% coverage |
| Deploy | Dockerfile (non-root uid 1000), **k8s manifest** (3 replicas, HPA 3→20, PDB minAvailable 2, liveness/readiness probes) | Production-shaped, although there is no evidence of a live deployment |

## 1.4 Architecture

```
                         ┌──────────────────────── Next.js 16 frontend ───────────────────────┐
 Browser ── HTTPS ──►    │  Pages: /login, /dashboard, /workspace, /my-center                  │
                         │  ~45 BFF route handlers under src/app/api/** (server side)          │
                         │  JWT kept in an httpOnly cookie, so the browser never sees it       │
                         └──────────────────────────────┬──────────────────────────────────────┘
                                                        │ server-to-server calls (no CORS needed)
                         ┌──────────────────────────────▼──────────────────────────────────────┐
                         │ FastAPI app (12 routers)                                            │
                         │  foundation/  auth, RBAC, config & secret validation                │
                         │  investigation/ cases, FSM, assignment+SLA, alerts, similar cases,  │
                         │                 relationships, N-hop graph, timeline, STR reports    │
                         │  orchestration/ LLM gateway, 12 case-scoped tools, grounding,        │
                         │                 PII gate, agent loop, recommendation, copilot        │
                         │  detection/   6 detectors, rule DSL, 28-dim features, ensemble,      │
                         │               LinUCB bandit, GraphStore(NetworkX)                    │
                         └───────┬───────────────────────────┬─────────────────────────────────┘
                                 │ SQLAlchemy                │ OpenAI-compatible API
                         ┌───────▼────────┐          ┌───────▼─────────┐
                         │ SQLite/Postgres│          │ OpenRouter LLM  │
                         │ + audit hash   │          │ (Claude Sonnet) │
                         │   chain        │          └─────────────────┘
                         └────────────────┘
 Offline CLI scripts: ingest.py → train_detection_model.py → run_detection_pipeline.py
```

**Why a BFF (backend-for-frontend)?**
- The browser talks only to Next.js route handlers. Those call FastAPI server-side.
- The JWT lives in an **httpOnly cookie**, so XSS can't steal it.
- FastAPI doesn't need CORS, because all calls are server-to-server.
- Each page can aggregate several backend calls into one.

## 1.5 The main flows (be able to draw these)

### A. Offline batch: ingest → train → detect → cases
1. **Ingest** (`db/ingest.py`, CLI):
   - Loads the accounts and transactions CSVs: the **IBM AML "HI-Small" benchmark**, which is **5,078,345 transactions, 518,573 accounts and 166,207 customers**, plus a 200-customer India demo overlay.
   - Validates the input: size and row caps, and de-duplicating repeated header rows.
   - Every write appends an **audit-log hash-chain** row.
2. **Train** (`train_detection_model.py`):
   - Builds the graph and extracts a **28-dimension feature vector** per account.
   - Trains XGBoost with a **temporal 70/15/15 split** (by each account's last transaction).
   - Saves a joblib artifact and a `model_runs` row; exactly one run is marked active.
3. **Detect** (`run_detection_pipeline.py`):
   - Load the model.
   - Run every enabled rule through the **rule engine** (detector primitives).
   - Compute **ensemble scores**.
   - Generate or refresh alerts.
   - **Rank the queue with LinUCB.**
   - For the top N (default 20): create a case, **auto-assign it to the investigator with the fewest open cases**, move it NEW → ASSIGNED through the FSM, set the SLA due date, and compute network risk.

### B. L1 triage (investigator)
1. **Login.** The browser calls the BFF, which calls `POST /auth/login`. The JWT is stored in an httpOnly cookie.
   - **Timing-attack defence:** if the user doesn't exist, bcrypt still runs against a dummy hash, so every failure takes about 0.4s and an attacker can't tell which usernames exist.
2. Dashboard: `GET /dashboard/summary` and `GET /alerts`.
3. The investigator opens a case: `POST /cases/{id}/start` moves it ASSIGNED → IN_PROGRESS.
4. Panels load the customer snapshot, transaction summary, money flow, geo-risk, previous alerts, network risk and similar cases.
   - **Every account route checks that the account belongs to that case** (`require_case_scoped_account`), so data can't leak across cases.
5. **Decision** (`POST /cases/{id}/decision`):
   - The FSM validates the transition.
   - Closing a case writes a feature vector (for similar cases) and detection feedback.
   - It **updates LinUCB: +1 for a true positive, −0.3 for a false positive.**

### C. L2 deep investigation
- `GET /cases/{id}/accounts/{acc}/graph?radius=N` returns an **N-hop ego-graph**: a BFS from the account, **capped at 500 edges per account per hop**, with an optional time window. The call is audited.
- Also available: profile, behaviour, timeline, transaction search, **relationships** (shared PAN, phone, email or employer, compared using **HMAC-hashed** values), pattern and graph explanations, evidence pinning and notes.

### D. Recommendation engine (agentic LLM, built by the teammate)
1. **Rule grounding** lists which rules fired and which actions are eligible.
2. **Phase 1:** the model calls fact tools for up to 12 iterations. Each tool is **bound to the case** when it is constructed, and each result passes through the **PII gate** before the model sees it.
3. **Phase 2:** a **forced `submit` tool call** returns structured claims with citations.
4. **The grounding validator** checks every claim (gates below). The action must come from the eligible list. Ranking is deterministic, never the LLM's opinion.

### E. Reports
- STR/SAR reports can be made only for cases **closed as true positive**.
- The output is canonical JSON with a **SHA-256 hash** (tamper evidence) plus a PDF.
- Lifecycle: edit → finalise → submit.

## 1.6 Detection algorithms (DSA-relevant; know these)

| Pattern | What it means in AML | How the code detects it | DSA concept |
|---|---|---|---|
| **Layering** | Money hops quickly through many accounts to hide its origin | `get_transaction_chains` does a **greedy forward temporal walk**: from an account, take the first outgoing transaction within the time window, never revisit a node, up to 20 hops. Tight pass: ≥3 hops within 120 min and amount decay ratio ≥ 0.5. Extended pass: ≥4 hops within 30 days. Score = `decay×0.4 + min(hops/10, 0.3) + (1−preservation)×0.3`. | DFS-style path walk on a time-ordered graph, with a visited set |
| **Round-trip** | Money returns to its origin through intermediaries | Aggregate to a DiGraph → keep **strongly connected components** of size 2–500 → `nx.simple_cycles(length_bound=3)`, then up to 12, with at most 2000 cycles → flag if return ratio ≥ 0.85 | **SCC (Tarjan / Kosaraju)** + **cycle detection** |
| **Structuring** | Splitting deposits to stay under the ₹10L reporting threshold | Classic: ≥3 transactions of ₹9L–₹10L in a window. Split: daily total ₹9L–₹10L across ≥2 transactions. | Sliding window / group-by |
| **Dormancy** | A long-inactive account suddenly bursts | Largest gap ≥180 days, then ≥5 transactions, with the post-gap average ≥10× the pre-gap average | Sort + max gap |
| **Profile mismatch** | Volume far beyond declared income | volume / income > threshold; **peer z-score > 3** within the occupation × income bracket | Group statistics |
| **Fan-out / fan-in** | One account scatters money to many, or gathers from many | Degree-based checks | In/out degree |

**Why SCCs before cycle search?**
- **A cycle can only exist inside one SCC.**
- Pruning to SCCs removes most of the graph before running the expensive cycle enumeration, which is exponential in the worst case.
- `length_bound` caps the path length, and the 2000 cap bounds the output.
- **This is a great DSA talking point.** Be ready to explain Kosaraju (two DFS passes, the second on the reversed graph) or Tarjan (one DFS with low-link values), both O(V+E).

**Rule DSL** (`rules/engine.py`):
- 11 primitives: cycle, chain, fan_degree, amount_band_count, split_sum_threshold, inactivity_then_burst, peer_zscore_deviation, and others.
- **Tier 1** is a single condition. **Tier 2** combines conditions with AND/OR/NOT over per-account match sets, which are plain set operations.
- Parameter "ceilings" stop risky values, and there is a `dry_run` mode.
- Investigators can propose rules; an admin approves or rejects them (`rule_proposals`).

## 1.7 Scoring and ML

**XGBoost:**
- Settings: 500 trees, depth 6, learning rate 0.03.
- **`scale_pos_weight` is capped at 15.** Auto-tuning gave about 80, which dropped precision to 4.9%.
- Early stopping on **AUC-PR**.
- The **decision threshold is chosen to maximise F1** on the validation PR curve; it came out at 0.4977.
- Uses a **temporal split** and source-only labels.

**IsolationForest:** contamination 0.05, 200 trees.

**Ensemble score** (`EnsembleScorer.compute_all`), as the code actually computes it:
- `ml = fraud_p × 100 × 0.3`, only if the account is predicted fraud.
- `pattern = min(Σ pattern weights, 100) × 0.55`. The weights are layering 25, round-trip 30, structuring 20, dormancy 20, profile 15, fan 22.
- `graph = (max(0, pr_pct − 0.5)×50 + max(0, bc_pct − 0.5)×50) × 0.3`, only if the account has flags.
- Plus a **convergence bonus** of up to 15 when patterns and ML agree.
- The total is capped at 100. Buckets map it to priorities P1–P4.

**Honest notes:**
- **The README's "ML 30% / Pattern 40% / Graph 30%" doesn't match the code.**
- **The IsolationForest score isn't used** in the final formula; a code comment admits it.
- **"PageRank" and "betweenness" are cheap proxies:** normalised weighted in-flow, and in-degree × out-degree.

**Model metrics on the full IBM dataset** (from `docs/METRICS.md`):

| Metric | Value |
|---|---|
| Samples | 496,995 accounts (0.48% positive) |
| Precision / recall / F1 | 0.254 / 0.173 / 0.206 |
| AUC-ROC | 0.778 |
| Training time | 7.4 s on GPU |
| Older hackathon figures | F1 0.683, AUC 0.933. Not reproduced on the rebuilt code; don't quote them. |

**How to defend the modest F1:**
> "The positive rate is 0.48%, so accuracy is meaningless. We optimise PR-AUC and pick the threshold by F1. The ML score is one signal: the final ranking combines rule-based pattern evidence, graph features and investigator feedback through the bandit. The model is a triage aid, not an auto-decision. Next I'd add graph-native features such as real PageRank and GNN embeddings, and improve labels."

**LinUCB contextual bandit:**
- **Context:** a 16-dimension vector per alert (risk, anomaly, fraud probability, pattern flags, role, log amount, counterparties, income ratio, channel diversity, bias).
- **Score:** `UCB = θᵀx + α·√(xᵀA⁻¹x)`, where θ = A⁻¹b. The first term **exploits** what has been learned; the second **explores** uncertain alerts.
- **Update on feedback:** `A += x xᵀ`, `b += r·x`, with r = +1 for a true positive and −0.3 for a false positive.
- A and b are **persisted in the DB** (`rl_arm_state`). This fixed a bug in the hackathon version, where learning was lost on restart.
- **Weak spot:** there is one global arm, and `np.linalg.inv` runs on every score. The fix is to cache A⁻¹ and update it incrementally with **Sherman–Morrison**.

**Similar cases:** cosine similarity between the current case's 16-dimension vector and the vectors of closed cases.

## 1.8 Data model (main tables)

| Group | Tables |
|---|---|
| Reference | `customers` (PAN, Aadhaar, phone, email, occupation, declared income, PEP/sanction, KYC), `accounts` (FK customer), `transactions` (source, dest, amount, channel enum UPI/NEFT/RTGS/IMPS, narration, is_laundering) |
| Detection | `alerts` (type, account_ids, score, severity, priority, status, rule_ids, model_run_id), `model_runs`, `rule_definitions` (DSL JSON), `rule_proposals`, `rl_arm_state` (A, b), `detection_feedback` |
| Investigation | `cases` (status, L1/L2, P1–P4, risk, assigned_to, sla_due_at, resolution), `case_accounts` (role, hop_distance), `case_status_history`, `evidence`, `notes`, `reports` (json_hash, pdf_path), `case_feature_vector` |
| Platform | `users` (role), **`audit_log` (prev_hash, row_hash: a SHA-256 chain)**, `watchlist`, `ingestion_log` |
| AI | `ai_interactions` (tools_called, facts, response, model, latency_ms), `relationships` (entity pair, shared attribute, **HMAC value hash**) |

**Case state machine** (`investigation/fsm.py`):

```
NEW → ASSIGNED → IN_PROGRESS → { AWAITING_REVIEW, ESCALATED, CLOSED_FP }
AWAITING_REVIEW ↔ IN_PROGRESS
AWAITING_REVIEW / ESCALATED → { CLOSED_TP, CLOSED_FP, MONITORING }   (terminal states)
ESCALATED → AWAITING_REVIEW
```

- `transition_case` is the **only** code that writes case status.
- It validates the move against `VALID_TRANSITIONS` and writes the status change and history in one audit row.
- **Why an FSM?** Illegal transitions can't happen (e.g., a closed case can't reopen silently), and the audit trail is clean. This maps directly to a **Zomato order-state machine**: PLACED → ACCEPTED → PREPARING → PICKED_UP → DELIVERED / CANCELLED.

**Audit hash chain:**
- `row_hash = SHA-256(canonical_json(row) + prev_hash)`.
- Changing any past row breaks every hash after it, and `verify_chain()` detects that. It's the same idea as a blockchain, without the distribution.
- **Weak spot:** the chain is global and serial, so concurrent writers on several replicas could fork it. Fixes: serialise audit writes (a single writer or queue), use a DB sequence with a lock, or keep per-entity chains.

## 1.9 AI guardrails (built by the teammate, but the most interesting part, so know it)
- **Grounding validator** (`orchestration/grounding.py`). Every claim must cite a fact, and three gates apply:
  1. The cited fact key must exist.
  2. The cited value must equal the tool's actual value (with numeric tolerance).
  3. **Every number in the prose must appear among the cited values.** Dates and times are stripped first.
  - Fact keys include the call arguments, e.g. `get_account_facts(account_id=A1).total_in`. That fixed a bug where account B's figures could be attributed to account A.
  - An adversarial fabrication test ("9,400,000 rupees… 37 offshore counterparties") was **blocked**.
- **PII egress gate:**
  - It **fails closed**: it raises rather than strips.
  - It detects known PII values from the case, plus regexes for PAN, Aadhaar and Indian mobile formats.
  - It runs **before** data reaches the LLM.
- **Prompt-injection defence:** transaction narration text is attacker-controlled, so it's never put in prompts. Investigator free text is length-capped.
- **The forced tool call** for structured output exists because `response_format` was silently ignored for that model on OpenRouter.
- **LLM choice** (measured cost and latency per explanation):

| Model | Cost | Latency | Outcome |
|---|---|---|---|
| Claude Sonnet 4.5 | $0.0041 | 6.1 s | **Chosen** |
| GPT-5 | $0.0154 | 21.6 s | |
| Gemini 2.5 Flash | $0.00054 | 2.0 s | |

## 1.10 Performance numbers to quote
- **Graph route latency: 325 ms → 96 ms after fixing an N+1 query.** This is a great Zomato-relevant story, since it's the same class of fix as Myntra.
- Other L2 routes: 7–13 ms.
- Detection run: **44,790 alerts** (profile_mismatch 35,988, structuring 3,366, round_trip 2,000, layering 453).
- Tests: **106 → 502+**, about 97% coverage, with a CI coverage gate of 90%.

## 1.11 TraceX: interview questions and model answers

**Q1. Walk me through the architecture.**
Use the diagram in 1.4. Cover: Next.js BFF → FastAPI with 4 modules → SQL DB + in-memory NetworkX graph → LLM gateway, plus the offline CLI pipeline. About 90 seconds.

**Q2. Why model transactions as a graph?**
Laundering is a **relationship** pattern (chains, cycles, fan-in/fan-out) that a per-transaction model can't see. Graph algorithms find these directly: SCCs and cycles for round-trips, path walks for layering, degree for fan-out. The graph also gives investigators a visual money trail.

**Q3. How do you detect round-tripping efficiently on 5M transactions?**
- Aggregate to account→account edges, compute **SCCs**, and only search inside SCCs of size 2–500.
- Use `simple_cycles` with a **length bound** (3 first, then up to 12) and cap the result at 2000.
- Then compute the return ratio.
- Complexity: SCC is O(V+E). Cycle enumeration is output-sensitive (Johnson's algorithm is O((V+E)(C+1))), so bounding the length and count keeps it tractable.

**Q4. Why NetworkX? Would it scale?**
- It's in-memory and single-process. That's fine for a pilot and for batch detection.
- But the whole graph is built per process, and the k8s manifest's 3–20 replicas would each hold a copy.
- **It sits behind a `GraphStore` interface**, so production can swap in **Neo4j** (or TigerGraph) with no detector changes.
- Also use **windowed subgraphs** (e.g., rolling 90 days) to bound the size.

**Q5. How would you make detection real-time?**
Today detection is a batch CLI (the hackathon version had an SSE demo). The real-time design:
1. Transactions stream into **Kafka**.
2. A stream processor updates the graph incrementally.
3. Cheap rules (structuring windows, fan-out counters) run per event.
4. Expensive cycle searches run on a schedule, or only for affected subgraphs.
5. Alerts go to a topic.

**Q6. The F1 is only 0.206. Isn't the model bad?**
See the defence in 1.7: 0.48% positives, PR-AUC focus, the ensemble design, and human-in-the-loop triage. Then name the next steps: better graph features, label quality, threshold per segment.

**Q7. How does the investigator feedback loop work?**
On a decision, the LinUCB update applies (+1 / −0.3). The case vector is stored for similar-case search, and detection feedback is stored for retraining and rule tuning. Explain explore vs exploit.

**Q8. How is auth done? How do you stop an investigator seeing another's case?**
- JWT (HS256) in an httpOnly cookie through the BFF.
- RBAC with `require_role`, plus **case-level access checks** (`require_case_access`) and **account-in-case checks**.
- Startup fails outside dev if secrets are missing.
- Login timing parity uses a dummy bcrypt hash.
- An open-redirect bug in the `next` parameter was fixed.

**Q9. What is JWT, and what are its risks?**
- Structure: header.payload.signature (base64url), signed with HMAC-SHA256 here. The server is stateless and just verifies the signature.
- Risks: it can't be revoked before expiry (mitigate with short expiry plus refresh tokens, or a denylist), XSS theft if stored in localStorage (mitigated here by the httpOnly cookie), and CSRF with cookies (mitigate with SameSite and CSRF tokens).

**Q10. How do you guarantee the audit log isn't tampered with?**
The hash chain (1.8). Mention the concurrency weakness and its fix yourself, before they ask.

**Q11. How do you stop the LLM hallucinating numbers?**
The three grounding gates, structured forced-tool output, deterministic action ranking (the LLM never decides confidence), and refusing rather than guessing. Credit your teammate for building it.

**Q12. What was the hardest bug?**
Pick one and own it. Options:
- **The LinUCB state being lost on restart:** you fixed it by persisting A and b in `rl_arm_state` (PR #4).
- **The N+1 query on the graph route**, 325 → 96 ms.
- **A FSM transition found missing through real use** (ESCALATED → AWAITING_REVIEW).

**Q13. Design question: how would you scale TraceX to a real bank with 1B transactions?**
- **Storage:** OLTP Postgres (partitioned by date) for cases and accounts, and a graph DB for relationships.
- **Ingest:** a streaming ingest via Kafka with **batched** audit writes. The current ingest took about **2h11m single-threaded** because it audits per row.
- **Detection:** incremental detection, with windowed subgraphs.
- **Serving:** stateless API replicas, a Redis cache for case-panel data and LLM explanation caching, and async LLM calls with a job queue (recommendations take about 126 s today).

**Q14. What would you do differently?**
1. Put the IsolationForest score into the ensemble, or remove it.
2. Learn the ensemble weights instead of hand-setting them.
3. Use real PageRank and betweenness, computed offline.
4. Make the audit chain safe for concurrency.
5. Make the LLM calls async and cached.

**Q15. Is LinUCB with a single arm really reinforcement learning?**
Be honest. With one global arm, it is effectively **online ridge regression with an exploration bonus** (α = 1): a contextual bandit, not full RL, because there are no states or transitions. That's still the right tool for the job. Each alert is an independent decision with immediate feedback, so you don't need multi-step RL. The upgrade path: one arm per typology or investigator team, and incremental A⁻¹ updates with Sherman–Morrison.

**Q16. Does similar-case search actually work?**
Cosine similarity over 16-dimensional vectors. The measured similarities are all above 0.99, so it **barely discriminates**. The reason is that unnormalised features with large magnitudes, such as log amount, dominate the angle. Fixes: standardise each feature (z-score) before computing cosine, weight the features, or use learned embeddings.

**Q17. How do you stop the LLM tools from reading another case's data?**
- The case ID is bound when the tool catalog is constructed; **no tool schema exposes a `case_id` parameter**.
- Account IDs go through the same scope check as the HTTP routes.
- In a live test the model attempted 2 out-of-scope calls, and both were blocked.

**Q18. Why does ingest take over 2 hours, and how would you fix it?**
Every row insert also appends to the audit hash chain, one at a time, and the process is single-threaded. To fix it:
- Bulk-insert in batches (`executemany` / COPY).
- Write one audit entry per batch, holding a hash of the batch, instead of one per row.
- Load in parallel by partition.
- Build indexes after the load.

---

# PART 2: SupplySenseAI, supply-chain risk management

## 2.1 Pitches

**30 seconds**
> "SupplySense AI is a supply-chain risk dashboard for Indian logistics teams that can't afford SAP-level tools. It tracks shipments across 20 hub cities and 8 carriers. It gives each one a 0–100 delay-risk score with its top 3 reasons, using XGBoost with SHAP or a rule-based fallback. It pulls live disaster and weather data from GDACS and OpenWeatherMap, and runs a BFS cascade on a NetworkX graph to show how a disruption in one city ripples downstream. It suggests three alternative routes, ranked by the user's weighting of cost, time, carbon and risk. It's a FastAPI + Next.js app, containerised and deployed on Google Cloud Run."

Tagline: "We don't just detect disruptions — we show you their future, and rewrite it."

## 2.2 Context you must know
- A **hackathon-style project**: the repo has a slide-by-slide `PRESENTATION.md`.
- It was created on **2026-04-28** and has only **3 commits**, including one squashed "clean release" import, so you can't show history.
- The frontend folder is named `chainguard` (an earlier name).
- **The repo credits only Vaideeka.** If asked about teammates, answer honestly about how it was built.

## 2.3 Tech stack

| Layer | Technology | Why |
|---|---|---|
| API | **FastAPI**, Uvicorn, Pydantic (about 30 models) | Fast async Python API, with ML in-process |
| Graph | **NetworkX DiGraph** | BFS cascade, `all_simple_paths`, `betweenness_centrality` |
| ML | **XGBoost** `XGBClassifier`, **SHAP** `TreeExplainer`, scikit-learn, joblib | Tabular classifier plus per-prediction explanations |
| LLM | Google **Gemini 2.5 Flash** (`google-generativeai`) | Chat assistant over dashboard context, with a mock fallback |
| External data | **OpenWeatherMap**, **GDACS** (global disaster alerts), **ReliefWeb** | Live weather and disaster signals |
| HTTP | httpx (sync and async) | External API calls with timeouts |
| Frontend | **Next.js 14**, React 18, TypeScript, Tailwind, **Leaflet** maps | Leaflet loaded with `next/dynamic({ssr:false})` because it needs `window` |
| Deploy | **Docker** (python:3.11-slim, non-root user, trains the model at build time), **Google Cloud Run** (`deploy.sh`: 1 GiB, 1 CPU, 0–3 instances) | Serverless containers that scale to zero, so the cost is about $0 |
| Tests | pytest + pytest-asyncio, **80 tests** | |

**Honest note:**
- `package.json` lists SWR, Zustand, Recharts, Framer Motion and axios, and the README credits them, but **the code doesn't import them**. It uses plain `fetch`, `useState` and `setInterval`.
- Every page is a client component.
- If asked, say: "We scaffolded with those libraries but ended up using plain fetch and React state. I'd remove the unused dependencies."

## 2.4 Architecture

```
 Next.js 14 (chainguard/)          FastAPI monolith (backend/)                External
 ┌──────────────────────┐   REST   ┌──────────────────────────────────────┐  ┌──────────────┐
 │ dashboard, create,   │ ───────► │ routers/: shipments, cascade,        │  │OpenWeatherMap│
 │ nodes, simulate,     │ /api/v1  │   optimization, chat, resilience,    │◄─┤GDACS         │
 │ cascade, reroute,    │          │   alerts, nodes                      │  │ReliefWeb     │
 │ chat (Leaflet map)   │          │ services/: data_store (singleton),   │  │Gemini 2.5    │
 │ polls every 30–60 s  │          │   risk_scorer, cascade_engine,       │  └──────────────┘
 └──────────────────────┘          │   route_optimizer, resilience_engine,│
                                   │   weather, disruption_feed, festival │
                                   │ ml/: dataset generators, train_model │
                                   │ ALL STATE IN MEMORY (no database)    │
                                   └──────────────────────────────────────┘
```

**Startup:**
1. `DataStore` (a lazy singleton) fetches weather for the 20 cities, then the GDACS disaster feed.
2. It builds about 50 shipments on fixed origin-destination lanes, the disruptions near those shipments (within 300 km), and alerts.
3. It builds the city graph and re-scores every shipment.
4. The lifespan hook precomputes resilience.

## 2.5 Main flows
1. **Dashboard:** on load and every 60 s, it calls rescore, then fetches KPIs, shipments, disruptions and the top-5 risky shipments (each with route alternatives).
2. **Cascade** (`GET /cascade/{disruption_id}`): BFS from the disrupted city (below) → a nested tree → rendered on the map and as a tree.
3. **What-if simulate** (`POST /simulate/node`, `/cascade/simulate`): pick a city, type and severity → run the cascade → list affected shipments → count the alternatives with risk < 50 → return fixed recommendations by disruption type.
4. **Reroute** (`POST /optimize/routes`): the user sets weights for cost, time, carbon and risk → 3 candidate routes are scored → a before/after comparison is returned.
5. **Create shipment:** validate the city, carrier and mode → compute risk → store it → rebuild the graph → suggest routes.
6. **Chat** (`POST /chat`): context (KPIs, top-10 risky shipments, disruptions) goes to Gemini as the system instruction, with per-session history (last 20 turns). It falls back to a keyword-based mock.

## 2.6 Algorithms (these are your DSA hooks)

### Cascade BFS (`services/cascade_engine.py`)

```python
propagated = impact * dep_weight * (0.85 ** depth)
if propagated >= 0.05:
    queue.append((neighbor, propagated, depth + 1, node))
```

- BFS over successors in the DiGraph.
- **Impact decays** at each hop: it is multiplied by the edge's dependency weight (0.7 + risk/1000) and by 0.85 per level of depth.
- It stops at **depth > 4** or **impact < 0.05**.
- Per node: delay = base hours by disruption type (congestion 18 h, port closure 36 h, …) × impact; revenue and customers affected = the node's values × impact.

**Interview-ready critique (raise it yourself; it shows depth):**
- It uses `list.pop(0)`, which is **O(n)** per pop. Use `collections.deque` for O(1).
- The visited check happens at dequeue, so **the first arrival wins, not the highest impact**. To get the **maximum-impact path**, use a **max-heap, Dijkstra-style**: impact multiplies along a path and is ≤ 1, so take −log(weight) as the edge cost and run Dijkstra on it.

### Route optimisation (`services/route_optimizer.py`)
- **Three candidate routes:**
  1. Direct road: Haversine distance × a 1.28–1.38 road factor.
  2. Via an alternate port hub (Nhava Sheva, Ennore or Haldia, only for Mumbai, Chennai and Kolkata), or a generic route at 1.15× the distance.
  3. Air freight: 6× the cost, high CO₂, lowest risk.
- **Constants:** ₹45/km; 55 km/h by road; 650 km/h by air.
- **Scoring:** **min-max normalise** each metric across the candidates, then take a **weighted sum** (defaults: cost 0.25, time 0.35, carbon 0.15, risk 0.25). The lowest score wins.
- **Honest note:**
  - The README says "Pareto-optimal", but the code does weighted-sum scalarisation over 3 candidates; there's no Pareto filtering.
  - Accepting a route only marks the shipment as rerouted; it doesn't change its path.
  - Be precise: "weighted multi-criteria ranking over three candidate routes; a Pareto filter would be the next step".

**Haversine formula** (they may ask):
- `a = sin²(Δφ/2) + cos φ1 · cos φ2 · sin²(Δλ/2)`
- `d = 2R · atan2(√a, √(1−a))`, with R = 6371 km.

### Risk scoring (`services/risk_scorer.py`)
- **`score_shipment()` is a strategy/facade:** it tries `MLScorer` and falls back to `RuleBasedScorer`.
- **Rule-based scorer:**
  - **Inputs:** carrier late rate, mode risk, seasonal risk by month, distance, ETA buffer, origin/destination state risk, active-disruption boost, overdue penalty.
  - **Combination:** a weighted sum, scaled and capped at 100. The **top 3 factors** are shown as percentages.
- **ML scorer:**
  - XGBoost `predict_proba × 100` on **11 features** (carrier_late_rate, mode_risk, season_risk, distance_norm, eta_buffer, origin/destination region risk, month, day of week, is_high_value, is_express).
  - The **top 3 features by |SHAP|** become the explanation.
- **Training** (`ml/train_model.py`):
  - Stratified 80/20 split.
  - `n_estimators=200, max_depth=5, lr=0.05, subsample=0.8, colsample_bytree=0.8, scale_pos_weight=neg/pos`.
  - 5-fold CV AUC and a SHAP sanity check.

### Resilience score (`services/resilience_engine.py`): 5 components, weighted 25/20/20/15/20
1. **Route redundancy:** the share of city pairs with ≥2 simple paths of length ≤3 (`nx.all_simple_paths(cutoff=3)`). The paths are **not** checked for being vertex-disjoint, although the README says so.
2. **Carrier diversity:** `(1 − HHI)/(1 − 1/n)`, where HHI (the Herfindahl index) is the sum of squared market shares.
3. **Geographic spread:** Shannon entropy over 4 lat/lng quadrants.
4. **Buffer capacity:** average (deadline − ETA)/48 h.
5. **Recovery speed:** based on the average resolution hours of past disruptions.

### Others
- **Bottleneck detection:** `nx.betweenness_centrality` (Brandes' algorithm, O(VE)). Recomputed on every request; it should be cached.
- **Festival and monsoon calendar:** hard-coded 2025–27 dates and congestion factors per region (Diwali, sale windows, and so on).
- **Caches:** module-level dicts with TTLs (weather 30 min, disasters 15 min).

## 2.7 Weak spots: be ready (an interviewer who reads the code will find these)

| # | Weak spot | Your honest answer |
|---|---|---|
| 1 | **ML trained on synthetic labels.** The data generators label rows with a hand-written formula plus small noise, so the **92.25% accuracy / 0.98 AUC** mostly measures how well XGBoost reconstructs that formula. The test suite even checks that ML and rule scores correlate. | "We had no real delivery-outcome data, so the model is a **surrogate** that proves the pipeline end to end: features, training, SHAP explanations, serving with fallback. With real labelled outcomes I'd retrain with a time-based split and calibrate the probabilities. I wouldn't claim the 92% as real-world accuracy." **Don't quote 92% as a real result.** |
| 2 | **Train/serve skew.** Season risk and carrier tables differ between dataset generation and inference. One generator uses `datetime.now()`, so month and day-of-week are constant within a dataset. | "That's a classic train/serve skew bug. The fix is a single shared feature module used by both training and serving, plus feature-distribution monitoring." |
| 3 | **Four different risk formulas** (startup heuristic, rule scorer, ML scorer, create-shipment scorer). The ML path ignores live weather and disruptions. | "We should consolidate into one scorer and feed live signals in as features." |
| 4 | **All state in memory, a singleton with no locks.** Each Cloud Run instance (up to 3) has different state; a cold start regenerates random data. | "Fine for a demo. For production: Postgres/PostGIS, Redis for caches, stateless API instances." |
| 5 | **No auth. CORS `*` with credentials.** **`deploy.sh` contains a hard-coded Gemini API key in a public repo.** | "That's a mistake. The key should be rotated and moved to Secret Manager, and auth should be added. I'd now use secret scanning in CI." **Rotate this key now, before the interview.** |
| 6 | **Performance:** `/kpis` recomputes resilience (about 380 path searches) on every call; betweenness is recomputed per request; startup makes 20 sequential blocking weather calls; BFS uses `pop(0)`. | "Precompute on data change, cache the results, fetch weather concurrently with async httpx (`asyncio.gather`), use a deque." |
| 7 | **"Real-time" means client polling** every 30–60 s, plus a mutating POST on every dashboard load. | "Next step: server-sent events or WebSockets, with a scheduled worker refreshing feeds (Cloud Scheduler + Pub/Sub)." |
| 8 | **Docs drift:** unused libraries; "Pareto"; "vertex-disjoint"; "15,000+ lines" (actually about 10.8k); Gemini 2.0 vs 2.5. | Fix the README before the interview if possible. |

## 2.8 SupplySenseAI: interview questions and model answers

**Q1. Explain the cascade algorithm. What's its complexity?**
BFS with multiplicative decay and a depth cutoff of 4 and impact cutoff of 0.05, so it's bounded by the reachable subgraph: O(V+E). Then give your own improvements: a deque, and max-impact Dijkstra on −log weights.

**Q2. How would you find the best route on a real road network?**
Model it as a weighted graph and run **Dijkstra**, or **A\*** with a Haversine heuristic, which is admissible because it never overestimates the remaining distance. For several objectives: a weighted-sum cost per edge, or a Pareto (multi-label) search. For production, use OSRM or the Google Routes API with contraction hierarchies.

**Q3. Why XGBoost? Why SHAP?**
- XGBoost: tabular data, small dataset, fast, handles mixed scales, `scale_pos_weight` for imbalance.
- SHAP: per-prediction, additive explanations with theoretical guarantees. `TreeExplainer` is exact and fast for trees.

**Q4. How do you handle an external API (weather or disasters) being down?**
- Timeouts (5 s), then a heuristic fallback.
- TTL caches (30 min / 15 min).
- ML falls back to rules, and chat falls back to the mock.
- Improvements: circuit breaker, retries with backoff, serving stale data from cache with a "stale" flag.

**Q5. How would you make it production-ready for 1M shipments a day?**
1. **Storage:** a Postgres/PostGIS DB (shipments, routes, geo queries), with a Redis cache.
2. **Ingestion:** a Kafka (or Pub/Sub) stream of shipment events; a scheduled worker for the feeds.
3. **Compute:** precompute risk on events; stateless API replicas.
4. **Push:** server-sent events or WebSockets to the UI.
5. **Platform:** auth plus multi-tenancy; a model registry with monitoring.

**Q6. Why Cloud Run?**
Serverless containers: scale to zero (cost near $0), autoscaling, no cluster to manage. Trade-offs: cold starts, and no shared in-memory state across instances, which is why the in-memory store becomes a problem.

**Q7. What's the Herfindahl index / Shannon entropy doing in resilience?**
- HHI = Σ sᵢ², which measures carrier concentration; normalised diversity = (1−HHI)/(1−1/n).
- Entropy = −Σ pᵢ log pᵢ, which measures how evenly shipments spread across regions.

---

# PART 3: Cross-project questions and Zomato tie-ins

## 3.1 Common questions

| Question | Answer outline |
|---|---|
| Which project are you proudest of, and why? | **TraceX.** It's a real multi-module system, and you owned auth/RBAC, the case FSM, auto-assignment, the bandit persistence and the entire frontend. It has 500+ tests and placed top 10 of 27,000+ teams. |
| Why FastAPI in both projects? | The ML and graph stack is in Python, so in-process inference avoids a network hop. FastAPI gives async support, Pydantic validation and OpenAPI docs. Trade-off: the GIL limits CPU-bound concurrency; scale with processes or replicas, or move hot paths to Go. |
| Why Next.js? | File-based routing, server-side route handlers (the BFF pattern in TraceX), TypeScript. |
| SQL vs NoSQL for these projects? | TraceX: relational data (cases, accounts, FKs, audit), with ACID needed, so SQL. A graph DB for relationships. SupplySense: currently none; I'd pick Postgres + PostGIS for geo queries. |
| How did you test? | TraceX: pytest, 500+ tests, coverage gate at 90% in CI, plus ruff and mypy. SupplySense: 80 pytest tests; no CI yet (an honest gap). |
| Used AI tools while building? | TraceX was built with AI-assisted sessions (the repo has `.claude/` and session logs). **Be upfront:** "We used Claude Code heavily for implementation. I designed the phases, reviewed every PR, and can explain every component." Zomato interviewers have rejected candidates whose code they couldn't explain "without AI assistance", so make sure you can explain all of it. |
| What would you do differently? | TraceX: concurrency-safe audit log, learned ensemble weights, async LLM. SupplySense: real labels, one shared feature pipeline, a database, auth, no secrets in the repo. |

## 3.2 Zomato tie-ins: turn project answers toward Zomato's problems

| Your project concept | Zomato analogue you can mention |
|---|---|
| Case FSM (TraceX) | **Order lifecycle state machine:** placed → accepted → preparing → picked up → delivered / cancelled |
| Workload-based auto-assignment (fewest open cases) | **Rider assignment / dispatch:** assign the nearest available rider with the lowest load, using a min-heap keyed by (distance, load) |
| LinUCB bandit for alert ranking | **Ranking restaurants and recommendations** with explore/exploit; Zomato does ranking and personalisation |
| SLA due dates | **Delivery-time promises** and breach alerts |
| Hash-chained audit log | **Payments and refunds audit trail** |
| N+1 fix, 325 → 96 ms | Same class of fix as **Myntra's 50% DB-connection reduction**; hot read paths like menus |
| Cascade BFS | **Impact of a restaurant / dark store / rider shortage** on orders in an area |
| Route optimisation, Haversine | **ETA prediction and rider routing** |
| Weather/disaster signals affecting risk | **Rain surge and peak handling** (a "rain mode" for delivery ETAs) |
| Fallbacks when external APIs fail | **Graceful degradation** when a payment gateway or maps service is down |

## 3.3 Before the interview: checklist
- [ ] **Rotate the Gemini API key** committed in `SupplySenseAI/deploy.sh`, and remove it from the repo history.
- [ ] Optionally fix the stale README claims in both repos (Pareto, vertex-disjoint, unused libraries, ensemble weights, accuracy numbers), or be ready to explain them.
- [ ] Make sure the "Demo" links on the resume still work.
- [ ] Draw both architectures from memory on paper in under 2 minutes each.
- [ ] Be able to code **Tarjan's or Kosaraju's SCC**, **BFS with a deque** and **Dijkstra**. They tie the projects to DSA (code is in `01b_DSA_Must_Code.md`).
