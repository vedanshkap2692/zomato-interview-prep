# DSA: Must-Code Templates (C++)

> Every ★ problem in `01_Zomato_Interview_Guide.md` is a variation of something in this file.
> **How to practise:**
> 1. Read a solution once.
> 2. Close the file.
> 3. Type it in a blank Google Doc with no IDE.
> 4. Dry-run it on paper.
> 5. Compare with this file.
>
> Repeat until it takes under 10 minutes and has zero bugs.
>
> Assumed headers: `#include <bits/stdc++.h>` and `using namespace std;`. Tree and list node structs follow LeetCode's.

```cpp
struct TreeNode { int val; TreeNode *left, *right; TreeNode(int v): val(v), left(nullptr), right(nullptr) {} };
struct ListNode { int val; ListNode* next; ListNode(int v = 0, ListNode* n = nullptr): val(v), next(n) {} };
```

---

## 1. Design data structures (highest priority)

### 1.1 LRU Cache (LC 146): STL version
**Idea:** a hash map for O(1) lookup plus a doubly linked list for recency order.
- The front of the list is the most recently used entry; the back is the least recently used.
- `splice` moves a node in O(1) without reallocating it.

```cpp
class LRUCache {
    int cap;
    list<pair<int,int>> dll;                                   // {key, value}; front = MRU
    unordered_map<int, list<pair<int,int>>::iterator> pos;     // key -> node
public:
    LRUCache(int capacity) : cap(capacity) {}

    int get(int key) {
        auto it = pos.find(key);
        if (it == pos.end()) return -1;
        dll.splice(dll.begin(), dll, it->second);              // move to front
        return it->second->second;
    }

    void put(int key, int value) {
        auto it = pos.find(key);
        if (it != pos.end()) {
            it->second->second = value;
            dll.splice(dll.begin(), dll, it->second);
            return;
        }
        if ((int)dll.size() == cap) {                          // evict LRU
            pos.erase(dll.back().first);
            dll.pop_back();
        }
        dll.emplace_front(key, value);
        pos[key] = dll.begin();
    }
};
```

### 1.2 LRU Cache: hand-written doubly linked list
Some interviewers say "don't use `std::list`". Use this version then.

```cpp
class LRUCache {
    struct Node { int key, val; Node *prev, *next; Node(int k, int v): key(k), val(v), prev(nullptr), next(nullptr) {} };
    int cap;
    unordered_map<int, Node*> mp;
    Node *head, *tail;                       // sentinels: head <-> ... <-> tail

    void unlink(Node* n) { n->prev->next = n->next; n->next->prev = n->prev; }
    void pushFront(Node* n) { n->next = head->next; n->prev = head; head->next->prev = n; head->next = n; }
public:
    LRUCache(int capacity) : cap(capacity) {
        head = new Node(0, 0); tail = new Node(0, 0);
        head->next = tail; tail->prev = head;
    }
    int get(int key) {
        if (!mp.count(key)) return -1;
        Node* n = mp[key];
        unlink(n); pushFront(n);
        return n->val;
    }
    void put(int key, int value) {
        if (mp.count(key)) {
            Node* n = mp[key]; n->val = value;
            unlink(n); pushFront(n);
            return;
        }
        if ((int)mp.size() == cap) {
            Node* lru = tail->prev;
            unlink(lru); mp.erase(lru->key); delete lru;
        }
        Node* n = new Node(key, value);
        pushFront(n); mp[key] = n;
    }
};
```

**Follow-ups to expect:**
- **Thread safety.** Wrap `get` and `put` in a `std::mutex`. A read-write lock does *not* help, because `get` also mutates the recency order.
- **TTL per key.** Store an expiry time in the node and treat expired entries as misses.
- **"Modified LRU"** (e.g., evict by priority, or a capacity based on size): same skeleton, different eviction rule.

### 1.3 LFU Cache (LC 460)
**Idea:**
- `kv`: key → {value, freq}.
- `freqList`: freq → list of keys, with the most recent at the front so ties break by LRU.
- `where`: key → iterator into its list.
- Track `minFreq`.

```cpp
class LFUCache {
    int cap, minFreq = 0;
    unordered_map<int, pair<int,int>> kv;              // key -> {value, freq}
    unordered_map<int, list<int>> freqList;            // freq -> keys (front = most recent)
    unordered_map<int, list<int>::iterator> where;     // key -> position in its freq list

    void touch(int key) {                              // increase frequency by 1
        int f = kv[key].second;
        freqList[f].erase(where[key]);
        if (freqList[f].empty()) {
            freqList.erase(f);
            if (minFreq == f) minFreq++;
        }
        kv[key].second = f + 1;
        freqList[f + 1].push_front(key);
        where[key] = freqList[f + 1].begin();
    }
public:
    LFUCache(int capacity) : cap(capacity) {}

    int get(int key) {
        if (!kv.count(key)) return -1;
        touch(key);
        return kv[key].first;
    }

    void put(int key, int value) {
        if (cap == 0) return;
        if (kv.count(key)) { kv[key].first = value; touch(key); return; }
        if ((int)kv.size() == cap) {                   // evict least frequent, then least recent
            int victim = freqList[minFreq].back();
            freqList[minFreq].pop_back();
            if (freqList[minFreq].empty()) freqList.erase(minFreq);
            kv.erase(victim); where.erase(victim);
        }
        kv[key] = {value, 1};
        freqList[1].push_front(key);
        where[key] = freqList[1].begin();
        minFreq = 1;                                   // a new key always has freq 1
    }
};
```

### 1.4 Insert Delete GetRandom O(1) (LC 380)
**Idea:** a vector holds the values; a hash map gives each value's index. To delete, swap the value with the last element and pop.

```cpp
class RandomizedSet {
    vector<int> a; unordered_map<int,int> idx;
public:
    bool insert(int v) { if (idx.count(v)) return false; idx[v] = a.size(); a.push_back(v); return true; }
    bool remove(int v) {
        if (!idx.count(v)) return false;
        int i = idx[v], last = a.back();
        a[i] = last; idx[last] = i;            // move last into the hole
        a.pop_back(); idx.erase(v);
        return true;
    }
    int getRandom() { return a[rand() % a.size()]; }
};
```

### 1.5 Orders in the last 5 minutes / Hit Counter (≈ LC 362)
**Idea:** one bucket per second, in a circular array of 300.
- A **write** resets its bucket if the bucket holds a stale timestamp. Then it increments the count.
- A **read** sums only the buckets whose timestamp is within the window.

```cpp
class HitCounter {
    static const int W = 300;              // window in seconds
    int times[W] = {0}, counts[W] = {0};
    mutex m;                               // interviewer asked for a lock in one Zomato round
public:
    void hit(int t) {
        lock_guard<mutex> g(m);
        int b = t % W;
        if (times[b] != t) { times[b] = t; counts[b] = 0; }
        counts[b]++;
    }
    int getHits(int t) {
        lock_guard<mutex> g(m);
        int total = 0;
        for (int i = 0; i < W; i++) if (t - times[i] < W) total += counts[i];
        return total;
    }
};
```

### 1.6 Median of a Data Stream (LC 295): two heaps
**Idea:**
- `lo` is a max-heap holding the smaller half; `hi` is a min-heap holding the larger half.
- Keep `lo.size() == hi.size()` or `lo.size() == hi.size() + 1`.

```cpp
class MedianFinder {
    priority_queue<int> lo;                                 // max-heap
    priority_queue<int, vector<int>, greater<int>> hi;      // min-heap
public:
    void addNum(int x) {
        lo.push(x);
        hi.push(lo.top()); lo.pop();                        // largest of lo moves to hi
        if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
    }
    double findMedian() {
        return lo.size() > hi.size() ? lo.top() : (lo.top() + (double)hi.top()) / 2.0;
    }
};
```
**Percentile variant** ("price between the 1/3 and 2/3 percentiles"): same structure, but rebalance so `lo` holds ⌈n/3⌉ elements.

### 1.7 Fisher-Yates shuffle (LC 384): "Spotify shuffle", asked 2×

```cpp
void shuffleArr(vector<int>& a) {
    for (int i = (int)a.size() - 1; i > 0; i--) {
        int j = rand() % (i + 1);          // uniform in [0, i]
        swap(a[i], a[j]);
    }
}
```
**Why it's uniform:** there are n! equally likely sequences of choices, and each maps to exactly one permutation. The naive "swap with any index in [0, n)" is biased, because nⁿ choices can't divide evenly into n! permutations.

### 1.8 Trie: autocomplete / Ctrl+R search

```cpp
struct Trie {
    struct Node { Node* ch[26] = {}; bool end = false; int latest = -1; };  // latest = most recent command index
    Node* root = new Node();

    void insert(const string& w, int id = -1) {
        Node* cur = root;
        for (char c : w) {
            int k = c - 'a';
            if (!cur->ch[k]) cur->ch[k] = new Node();
            cur = cur->ch[k];
            cur->latest = max(cur->latest, id);   // for "most recent command with this prefix"
        }
        cur->end = true;
    }
    Node* walk(const string& p) {
        Node* cur = root;
        for (char c : p) { cur = cur->ch[c - 'a']; if (!cur) return nullptr; }
        return cur;
    }
    bool search(const string& w)     { Node* n = walk(w); return n && n->end; }
    bool startsWith(const string& p) { return walk(p) != nullptr; }
};
```

---

## 2. Trees

### 2.1 Delete Node in a BST (LC 450): asked 4×

```cpp
TreeNode* deleteNode(TreeNode* root, int key) {
    if (!root) return nullptr;
    if (key < root->val)      root->left  = deleteNode(root->left, key);
    else if (key > root->val) root->right = deleteNode(root->right, key);
    else {
        if (!root->left)  { TreeNode* r = root->right; delete root; return r; }   // 0 or 1 child
        if (!root->right) { TreeNode* l = root->left;  delete root; return l; }
        TreeNode* succ = root->right;                    // 2 children: inorder successor
        while (succ->left) succ = succ->left;
        root->val = succ->val;
        root->right = deleteNode(root->right, succ->val);
    }
    return root;
}
```
- **Complexity:** O(h). That is O(log n) when balanced and O(n) when skewed. Mention that a skewed tree makes the recursion O(n) deep.
- **Variants:**
  - Delete all nodes in a range [lo, hi], which is the same as LC 669 "Trim a BST".
  - Delete by swapping with the inorder *predecessor* (the maximum of the left subtree) instead.

### 2.2 Lowest Common Ancestor (LC 236 binary tree, LC 235 BST)

```cpp
TreeNode* lca(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* L = lca(root->left, p, q);
    TreeNode* R = lca(root->right, p, q);
    if (L && R) return root;           // p and q are on different sides
    return L ? L : R;
}
TreeNode* lcaBST(TreeNode* root, TreeNode* p, TreeNode* q) {
    while (root) {
        if (p->val < root->val && q->val < root->val) root = root->left;
        else if (p->val > root->val && q->val > root->val) root = root->right;
        else return root;
    }
    return nullptr;
}
```

### 2.3 Iterative traversals (LC 94 / 144 / 145)

```cpp
vector<int> inorder(TreeNode* root) {
    vector<int> res; stack<TreeNode*> st; TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }
        cur = st.top(); st.pop();
        res.push_back(cur->val);
        cur = cur->right;
    }
    return res;
}
vector<int> postorder(TreeNode* root) {                 // one stack + last-visited pointer
    vector<int> res; stack<TreeNode*> st; TreeNode *cur = root, *last = nullptr;
    while (cur || !st.empty()) {
        if (cur) { st.push(cur); cur = cur->left; }
        else {
            TreeNode* peek = st.top();
            if (peek->right && last != peek->right) cur = peek->right;
            else { res.push_back(peek->val); last = peek; st.pop(); }
        }
    }
    return res;
}
```

### 2.4 Binary tree → doubly linked list, in place (inorder order)

```cpp
TreeNode *headDLL = nullptr, *prevDLL = nullptr;
void toDLL(TreeNode* root) {                 // left = prev, right = next
    if (!root) return;
    toDLL(root->left);
    if (!prevDLL) headDLL = root;
    else { prevDLL->right = root; root->left = prevDLL; }
    prevDLL = root;
    toDLL(root->right);
}
```

### 2.5 Validate BST and maximum path sum

```cpp
bool valid(TreeNode* n, long lo = LONG_MIN, long hi = LONG_MAX) {
    if (!n) return true;
    if (n->val <= lo || n->val >= hi) return false;
    return valid(n->left, lo, n->val) && valid(n->right, n->val, hi);
}
int best = INT_MIN;
int gain(TreeNode* n) {                       // LC 124
    if (!n) return 0;
    int l = max(0, gain(n->left)), r = max(0, gain(n->right));
    best = max(best, n->val + l + r);         // path that bends at n
    return n->val + max(l, r);                // path that continues upward
}
```

---

## 3. Graphs

### 3.1 Multi-source BFS: Rotting Oranges (LC 994) / nearest guard / 01 Matrix

```cpp
int orangesRotting(vector<vector<int>>& g) {
    int m = g.size(), n = g[0].size(), fresh = 0, minutes = 0;
    queue<pair<int,int>> q;
    for (int i = 0; i < m; i++) for (int j = 0; j < n; j++) {
        if (g[i][j] == 2) q.push({i, j});         // ALL sources enter the queue at time 0
        else if (g[i][j] == 1) fresh++;
    }
    int dr[] = {1, -1, 0, 0}, dc[] = {0, 0, 1, -1};
    while (!q.empty() && fresh > 0) {
        for (int s = q.size(); s > 0; s--) {      // process one level = one minute
            auto [r, c] = q.front(); q.pop();
            for (int d = 0; d < 4; d++) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr < 0 || nc < 0 || nr >= m || nc >= n || g[nr][nc] != 1) continue;
                g[nr][nc] = 2; fresh--; q.push({nr, nc});
            }
        }
        minutes++;
    }
    return fresh == 0 ? minutes : -1;
}
```
**Zomato framing:** "distance from every cell to the nearest delivery partner / dark store". Put every source in the queue at distance 0.

### 3.2 Disjoint Set Union: provinces, redundant edges, cluster sizes

```cpp
struct DSU {
    vector<int> p, sz;
    DSU(int n) : p(n), sz(n, 1) { iota(p.begin(), p.end(), 0); }
    int find(int x) { return p[x] == x ? x : p[x] = find(p[x]); }      // path compression
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;                                       // already connected → redundant edge
        if (sz[a] < sz[b]) swap(a, b);                                  // union by size
        p[b] = a; sz[a] += sz[b];
        return true;
    }
};
// Number of provinces = number of i with find(i) == i.
// Redundant edges = number of unite() calls that return false.
// Largest cluster = max sz[find(i)].
```
**Complexity:** O(α(n)) per operation (inverse Ackermann), which is effectively constant.

### 3.3 Topological sort: Kahn's algorithm (Course Schedule, LC 207 / 210)

```cpp
vector<int> topoSort(int n, vector<vector<int>>& edges) {     // edge {a, b}: a must come before b
    vector<vector<int>> adj(n); vector<int> indeg(n, 0);
    for (auto& e : edges) { adj[e[0]].push_back(e[1]); indeg[e[1]]++; }
    queue<int> q;
    for (int i = 0; i < n; i++) if (indeg[i] == 0) q.push(i);
    vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop(); order.push_back(u);
        for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
    }
    if ((int)order.size() < n) return {};                      // cycle exists
    return order;
}
```
- **Cycle detection with DFS** uses 3 colours: 0 = unvisited, 1 = on the current stack, 2 = done. Reaching a node with colour 1 means there's a back edge, so a cycle.
- **Note:** LC 207 gives prerequisites as `[course, prereq]`, so the edge goes prereq → course.

### 3.4 Dijkstra (LC 743), plus the "K stops" constraint variant

```cpp
vector<long long> dijkstra(int n, vector<vector<pair<int,int>>>& adj, int src) {  // adj[u] = {v, w}
    vector<long long> dist(n, LLONG_MAX);
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    dist[src] = 0; pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;                     // stale entry
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) { dist[v] = dist[u] + w; pq.push({dist[v], v}); }
        }
    }
    return dist;
}
// Cheapest Flights Within K Stops (LC 787): Bellman-Ford limited to K+1 rounds.
int cheapestK(int n, vector<vector<int>>& flights, int src, int dst, int k) {
    vector<int> dist(n, INT_MAX); dist[src] = 0;
    for (int i = 0; i <= k; i++) {
        vector<int> nd = dist;                         // copy, so each round adds at most one edge
        for (auto& f : flights)
            if (dist[f[0]] != INT_MAX && dist[f[0]] + f[2] < nd[f[1]]) nd[f[1]] = dist[f[0]] + f[2];
        dist = nd;
    }
    return dist[dst] == INT_MAX ? -1 : dist[dst];
}
```
- **Dijkstra:** O((V + E) log V). It **does not work with negative edge weights**; use Bellman-Ford there.
- **Bellman-Ford:** O(V·E). It also detects negative cycles: if any edge can still relax after V−1 rounds, there is one.

### 3.5 Jump Game IV (LC 1345): reported 2026

```cpp
int minJumps(vector<int>& arr) {
    int n = arr.size(); if (n == 1) return 0;
    unordered_map<int, vector<int>> same;
    for (int i = 0; i < n; i++) same[arr[i]].push_back(i);
    vector<bool> vis(n, false); queue<int> q; q.push(0); vis[0] = true;
    for (int steps = 0; !q.empty(); steps++) {
        for (int s = q.size(); s > 0; s--) {
            int i = q.front(); q.pop();
            if (i == n - 1) return steps;
            auto& nb = same[arr[i]];
            nb.push_back(i - 1); nb.push_back(i + 1);
            for (int j : nb) if (j >= 0 && j < n && !vis[j]) { vis[j] = true; q.push(j); }
            nb.clear();                       // KEY: never scan this value's list again → O(n)
        }
    }
    return -1;
}
```
**Follow-up:** "each jump has a different cost". BFS no longer finds the cheapest path, so switch to Dijkstra over the same neighbours.

### 3.6 Strongly connected components: Kosaraju (TraceX round-trip detection)

```cpp
void dfs1(int u, vector<vector<int>>& g, vector<bool>& vis, vector<int>& order) {
    vis[u] = true;
    for (int v : g[u]) if (!vis[v]) dfs1(v, g, vis, order);
    order.push_back(u);                                  // record by finish time
}
void dfs2(int u, vector<vector<int>>& rg, vector<int>& comp, int c) {
    comp[u] = c;
    for (int v : rg[u]) if (comp[v] == -1) dfs2(v, rg, comp, c);
}
int kosaraju(int n, vector<vector<int>>& g) {
    vector<vector<int>> rg(n);
    for (int u = 0; u < n; u++) for (int v : g[u]) rg[v].push_back(u);
    vector<bool> vis(n, false); vector<int> order;
    for (int i = 0; i < n; i++) if (!vis[i]) dfs1(i, g, vis, order);
    vector<int> comp(n, -1); int c = 0;
    for (int i = n - 1; i >= 0; i--) if (comp[order[i]] == -1) dfs2(order[i], rg, comp, c++);
    return c;                                            // number of SCCs
}
```
- **Complexity:** O(V + E).
- **Why it works:** the node that finishes last lies in a "source" SCC. In the reversed graph, a DFS from it cannot leave that SCC.

### 3.7 Prim's MST

```cpp
long long prim(int n, vector<vector<pair<int,int>>>& adj) {
    vector<bool> in(n, false); long long total = 0;
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;   // {weight, node}
    pq.push({0, 0});
    while (!pq.empty()) {
        auto [w, u] = pq.top(); pq.pop();
        if (in[u]) continue;
        in[u] = true; total += w;
        for (auto [v, wt] : adj[u]) if (!in[v]) pq.push({wt, v});
    }
    return total;
}
```
**Kruskal's alternative:** sort the edges by weight and call `DSU.unite` on each; keep the edge if `unite` returns true. O(E log E).

---

## 4. Sliding window and two pointers

### 4.1 Minimum Size Subarray Sum (LC 209): asked 2×

```cpp
int minSubArrayLen(int target, vector<int>& a) {
    int best = INT_MAX, l = 0; long long sum = 0;
    for (int r = 0; r < (int)a.size(); r++) {
        sum += a[r];
        while (sum >= target) { best = min(best, r - l + 1); sum -= a[l++]; }
    }
    return best == INT_MAX ? 0 : best;
}
```

### 4.2 Follow-up: the array has negatives (LC 862, shortest subarray with sum ≥ K)
**Why the window breaks:** with negatives, shrinking the window can *increase* the sum. Use prefix sums with a **monotonic deque** instead.

```cpp
int shortestSubarray(vector<int>& a, int k) {
    int n = a.size(); vector<long long> P(n + 1, 0);
    for (int i = 0; i < n; i++) P[i + 1] = P[i] + a[i];
    deque<int> dq; int best = INT_MAX;
    for (int j = 0; j <= n; j++) {
        while (!dq.empty() && P[j] - P[dq.front()] >= k) { best = min(best, j - dq.front()); dq.pop_front(); }
        while (!dq.empty() && P[dq.back()] >= P[j]) dq.pop_back();     // keep prefix values increasing
        dq.push_back(j);
    }
    return best == INT_MAX ? -1 : best;
}
```

### 4.3 Minimum Window Substring (LC 76)

```cpp
string minWindow(string s, string t) {
    vector<int> need(128, 0); for (char c : t) need[c]++;
    int missing = t.size(), l = 0, bestL = 0, bestLen = INT_MAX;
    for (int r = 0; r < (int)s.size(); r++) {
        if (need[s[r]]-- > 0) missing--;
        while (missing == 0) {
            if (r - l + 1 < bestLen) { bestLen = r - l + 1; bestL = l; }
            if (++need[s[l++]] > 0) missing++;
        }
    }
    return bestLen == INT_MAX ? "" : s.substr(bestL, bestLen);
}
```

### 4.4 Generic "longest window with at most K bad elements" (LC 1004, 3, 2958)

```cpp
int longestOnes(vector<int>& a, int k) {            // LC 1004: at most k zeros
    int l = 0, zeros = 0, best = 0;
    for (int r = 0; r < (int)a.size(); r++) {
        zeros += (a[r] == 0);
        while (zeros > k) zeros -= (a[l++] == 0);
        best = max(best, r - l + 1);
    }
    return best;
}
```

### 4.5 Count distinct elements in every window of size k

```cpp
vector<int> distinctInWindows(vector<int>& a, int k) {
    unordered_map<int,int> cnt; vector<int> res;
    for (int i = 0; i < (int)a.size(); i++) {
        cnt[a[i]]++;
        if (i >= k && --cnt[a[i - k]] == 0) cnt.erase(a[i - k]);
        if (i >= k - 1) res.push_back(cnt.size());
    }
    return res;
}
```

### 4.6 Subarray Sum Equals K (LC 560): prefix sum plus hash map

```cpp
int subarraySum(vector<int>& a, int k) {
    unordered_map<long long,int> seen{{0, 1}}; long long sum = 0; int cnt = 0;
    for (int x : a) { sum += x; if (seen.count(sum - k)) cnt += seen[sum - k]; seen[sum]++; }
    return cnt;
}
```

---

## 5. Dynamic programming

```cpp
// House Robber I (LC 198) and II (LC 213, circular)
int robLine(vector<int>& a, int lo, int hi) {
    int prev2 = 0, prev1 = 0;
    for (int i = lo; i <= hi; i++) { int cur = max(prev1, prev2 + a[i]); prev2 = prev1; prev1 = cur; }
    return prev1;
}
int robCircle(vector<int>& a) {
    int n = a.size(); if (n == 1) return a[0];
    return max(robLine(a, 0, n - 2), robLine(a, 1, n - 1));   // first and last are adjacent
}

// Maximum Product Subarray (LC 152)
int maxProduct(vector<int>& a) {
    int hi = a[0], lo = a[0], best = a[0];
    for (int i = 1; i < (int)a.size(); i++) {
        if (a[i] < 0) swap(hi, lo);               // a negative flips max and min
        hi = max(a[i], hi * a[i]);
        lo = min(a[i], lo * a[i]);
        best = max(best, hi);
    }
    return best;
}

// Kadane, returning the subarray (LC 53 variant)
tuple<int,int,int> kadane(vector<int>& a) {       // {sum, l, r}
    int best = a[0], cur = a[0], s = 0, bl = 0, br = 0;
    for (int i = 1; i < (int)a.size(); i++) {
        if (cur < 0) { cur = a[i]; s = i; } else cur += a[i];
        if (cur > best) { best = cur; bl = s; br = i; }
    }
    return {best, bl, br};
}

// LIS in O(n log n) (LC 300)
int lis(vector<int>& a) {
    vector<int> tails;                            // tails[k] = smallest tail of an increasing subsequence of length k+1
    for (int x : a) {
        auto it = lower_bound(tails.begin(), tails.end(), x);   // use upper_bound for non-decreasing
        if (it == tails.end()) tails.push_back(x); else *it = x;
    }
    return tails.size();
}
// Russian Doll Envelopes (LC 354): sort by width ascending, height DESCENDING, then lis(heights).

// LCS (LC 1143)
int lcs(string& a, string& b) {
    int n = a.size(), m = b.size(); vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
    for (int i = 1; i <= n; i++) for (int j = 1; j <= m; j++)
        dp[i][j] = a[i-1] == b[j-1] ? dp[i-1][j-1] + 1 : max(dp[i-1][j], dp[i][j-1]);
    return dp[n][m];
}
// Follow-up asked at Zomato: if string a has all-unique characters, map each char of b to its index
// in a (dropping chars not in a), then LCS = LIS of that index sequence: O(n log n).

// Maximal Square (LC 221)
int maximalSquare(vector<vector<char>>& g) {
    int m = g.size(), n = g[0].size(), best = 0;
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
    for (int i = 1; i <= m; i++) for (int j = 1; j <= n; j++)
        if (g[i-1][j-1] == '1') {
            dp[i][j] = 1 + min({dp[i-1][j], dp[i][j-1], dp[i-1][j-1]});
            best = max(best, dp[i][j]);
        }
    return best * best;
}

// 4 Keys Keyboard (LC 651): asked in a 2026 Round 1
int maxA(int n) {
    vector<long long> dp(n + 1);
    for (int i = 1; i <= n; i++) {
        dp[i] = dp[i - 1] + 1;                                  // press 'A'
        for (int j = 1; j <= i - 3; j++)                        // at step j: Ctrl-A, Ctrl-C, then (i-j-2) pastes
            dp[i] = max(dp[i], dp[j] * (i - j - 1));
    }
    return dp[n];
}

// Sell from either end; the k-th sale earns k × price ("max profit wines")
int wines(vector<int>& p) {
    int n = p.size(); vector<vector<int>> dp(n, vector<int>(n, 0));
    for (int len = 1; len <= n; len++)
        for (int l = 0; l + len - 1 < n; l++) {
            int r = l + len - 1, year = n - len + 1;
            if (l == r) { dp[l][r] = year * p[l]; continue; }
            dp[l][r] = max(year * p[l] + dp[l + 1][r], year * p[r] + dp[l][r - 1]);
        }
    return dp[0][n - 1];
}
```

**The DP method to say out loud:**
1. Define the state.
2. Write the recurrence.
3. Give the base cases.
4. State the order of computation.
5. State the answer's location.
6. Optimise the space.

---

## 6. Intervals and heaps

```cpp
// Merge Intervals (LC 56)
vector<vector<int>> merge(vector<vector<int>>& v) {
    sort(v.begin(), v.end());
    vector<vector<int>> res;
    for (auto& it : v) {
        if (res.empty() || res.back()[1] < it[0]) res.push_back(it);
        else res.back()[1] = max(res.back()[1], it[1]);
    }
    return res;
}

// Max overlap and its first point (OA 2×) / Meeting Rooms II: line sweep
pair<int,int> maxOverlap(vector<vector<int>>& v) {       // closed intervals [s, e]
    vector<pair<int,int>> ev;
    for (auto& it : v) { ev.push_back({it[0], +1}); ev.push_back({it[1] + 1, -1}); }
    sort(ev.begin(), ev.end());                          // at equal x, -1 sorts before +1
    int cur = 0, best = 0, at = 0;
    for (auto& [x, d] : ev) { cur += d; if (cur > best) { best = cur; at = x; } }
    return {best, at};
}

// Kth Largest (LC 215): min-heap of size k
int kthLargest(vector<int>& a, int k) {
    priority_queue<int, vector<int>, greater<int>> pq;
    for (int x : a) { pq.push(x); if ((int)pq.size() > k) pq.pop(); }
    return pq.top();
}

// Reorganise String (LC 767): "no author repeats consecutively"
string reorganize(string s) {
    vector<int> cnt(26, 0); for (char c : s) cnt[c - 'a']++;
    priority_queue<pair<int,char>> pq;
    for (int i = 0; i < 26; i++) if (cnt[i]) pq.push({cnt[i], 'a' + i});
    string res; pair<int,char> held = {0, '#'};
    while (!pq.empty()) {
        auto [c, ch] = pq.top(); pq.pop();
        res += ch;
        if (held.first > 0) pq.push(held);               // the previous char becomes available again
        held = {c - 1, ch};
    }
    return (int)res.size() == (int)s.size() ? res : "";
}

// Most transactions keeping the running balance >= 0 (OA): greedy + min-heap
int maxTransactions(vector<int>& a) {
    priority_queue<int, vector<int>, greater<int>> taken; long long sum = 0; int dropped = 0;
    for (int x : a) {
        sum += x; taken.push(x);
        if (sum < 0) { sum -= taken.top(); taken.pop(); dropped++; }   // drop the most negative so far
    }
    return a.size() - dropped;
}
```

---

## 7. Binary search

```cpp
// Search in Rotated Sorted Array (LC 33)
int searchRotated(vector<int>& a, int t) {
    int l = 0, r = a.size() - 1;
    while (l <= r) {
        int m = l + (r - l) / 2;
        if (a[m] == t) return m;
        if (a[l] <= a[m]) {                               // left half sorted
            if (a[l] <= t && t < a[m]) r = m - 1; else l = m + 1;
        } else {                                          // right half sorted
            if (a[m] < t && t <= a[r]) l = m + 1; else r = m - 1;
        }
    }
    return -1;
}

// Find Peak Element (LC 162)
int findPeak(vector<int>& a) {
    int l = 0, r = a.size() - 1;
    while (l < r) { int m = l + (r - l) / 2; if (a[m] < a[m + 1]) l = m + 1; else r = m; }
    return l;
}

// Binary search on the answer: Painter's Partition / Split Array Largest Sum (LC 410)
bool canSplit(vector<int>& a, int k, long long cap) {
    int parts = 1; long long cur = 0;
    for (int x : a) { if (cur + x > cap) { parts++; cur = 0; } cur += x; }
    return parts <= k;
}
int splitArray(vector<int>& a, int k) {
    long long l = *max_element(a.begin(), a.end()), r = accumulate(a.begin(), a.end(), 0LL);
    while (l < r) { long long m = l + (r - l) / 2; if (canSplit(a, k, m)) r = m; else l = m + 1; }
    return l;
}
```
**Binary-search-on-the-answer checklist:**
1. Is the answer monotonic? (If capacity X works, does every X' > X also work?)
2. Write a `feasible(x)` check.
3. Find the smallest x that is feasible.

---

## 8. Linked list

```cpp
ListNode* reverseList(ListNode* head) {                     // iterative
    ListNode* prev = nullptr;
    while (head) { ListNode* nx = head->next; head->next = prev; prev = head; head = nx; }
    return prev;
}
ListNode* reverseRec(ListNode* head) {                      // recursive
    if (!head || !head->next) return head;
    ListNode* newHead = reverseRec(head->next);
    head->next->next = head; head->next = nullptr;
    return newHead;
}

// Reverse Nodes in k-Group (LC 25)
ListNode* reverseKGroup(ListNode* head, int k) {
    ListNode dummy(0, head); ListNode* groupPrev = &dummy;
    while (true) {
        ListNode* kth = groupPrev;
        for (int i = 0; i < k && kth; i++) kth = kth->next;
        if (!kth) break;                                    // fewer than k nodes left
        ListNode *groupNext = kth->next, *prev = groupNext, *cur = groupPrev->next;
        while (cur != groupNext) { ListNode* nx = cur->next; cur->next = prev; prev = cur; cur = nx; }
        ListNode* first = groupPrev->next;                  // becomes the tail of this group
        groupPrev->next = kth;
        groupPrev = first;
    }
    return dummy.next;
}

// Palindrome Linked List (LC 234): O(1) space
bool isPalindrome(ListNode* head) {
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
    ListNode* second = reverseList(slow), *p = head, *q = second;
    bool ok = true;
    while (q) { if (p->val != q->val) { ok = false; break; } p = p->next; q = q->next; }
    reverseList(second);                                    // restore the list (good manners)
    return ok;
}
```

---

## 9. Stack: The Skyline Problem (LC 218)

```cpp
vector<vector<int>> getSkyline(vector<vector<int>>& b) {
    vector<pair<int,int>> ev;
    for (auto& x : b) { ev.push_back({x[0], -x[2]}); ev.push_back({x[1], x[2]}); }  // start = negative height
    sort(ev.begin(), ev.end());     // same x: starts before ends; taller starts first; shorter ends first
    multiset<int> hs{0}; int prev = 0; vector<vector<int>> res;
    for (auto& [x, h] : ev) {
        if (h < 0) hs.insert(-h); else hs.erase(hs.find(h));
        int cur = *hs.rbegin();
        if (cur != prev) { res.push_back({x, cur}); prev = cur; }
    }
    return res;
}
```
**Complexity:** O(n log n). Use `hs.erase(hs.find(h))`, **not** `hs.erase(h)`: the latter removes *every* copy of that height.

```cpp
// Next Greater Element: the monotonic stack template (used in Histogram, Daily Temperatures)
vector<int> nextGreater(vector<int>& a) {
    int n = a.size(); vector<int> res(n, -1); stack<int> st;    // stack holds indices with decreasing values
    for (int i = 0; i < n; i++) {
        while (!st.empty() && a[st.top()] < a[i]) { res[st.top()] = a[i]; st.pop(); }
        st.push(i);
    }
    return res;
}
```

---

## 10. Quick warm-ups (must take under 5 minutes each)

```cpp
int missingNumber(vector<int>& a) {                 // LC 268
    int x = a.size();
    for (int i = 0; i < (int)a.size(); i++) x ^= i ^ a[i];
    return x;
}

vector<int> majorityII(vector<int>& a) {            // LC 229: elements appearing > n/3 times
    int c1 = 0, c2 = 1, n1 = 0, n2 = 0;             // two distinct placeholder candidates
    for (int x : a) {
        if (x == c1) n1++;
        else if (x == c2) n2++;
        else if (n1 == 0) { c1 = x; n1 = 1; }
        else if (n2 == 0) { c2 = x; n2 = 1; }
        else { n1--; n2--; }
    }
    vector<int> res; n1 = n2 = 0;
    for (int x : a) { if (x == c1) n1++; else if (x == c2) n2++; }
    if (n1 > (int)a.size() / 3) res.push_back(c1);
    if (n2 > (int)a.size() / 3) res.push_back(c2);
    return res;
}

int minSwapsToSort(vector<int>& a) {                // permutation of 1..n: cycle decomposition
    int n = a.size(), swaps = 0; vector<bool> vis(n, false);
    for (int i = 0; i < n; i++) {
        if (vis[i] || a[i] == i + 1) continue;
        int len = 0, j = i;
        while (!vis[j]) { vis[j] = true; j = a[j] - 1; len++; }
        swaps += len - 1;
    }
    return swaps;
}
// For arbitrary values: sort a copy, map value → target index, then run the same cycle count.

bool increasingTriplet(vector<int>& a) {            // LC 334
    int first = INT_MAX, second = INT_MAX;
    for (int x : a) {
        if (x <= first) first = x;
        else if (x <= second) second = x;
        else return true;
    }
    return false;
}

int findMaxLength(vector<int>& a) {                 // LC 525: longest subarray with equal 0s and 1s
    unordered_map<int,int> first{{0, -1}}; int sum = 0, best = 0;
    for (int i = 0; i < (int)a.size(); i++) {
        sum += a[i] ? 1 : -1;
        if (first.count(sum)) best = max(best, i - first[sum]); else first[sum] = i;
    }
    return best;
}

int trailingZeroes(int n) { int c = 0; while (n) { n /= 5; c += n; } return c; }   // LC 172
```

---

## 11. Complexity cheat sheet (say these confidently)

| Operation | Complexity |
|---|---|
| `unordered_map` find/insert | O(1) average, O(n) worst case (collisions) |
| `map` / `set` (red-black tree) | O(log n); keys kept sorted |
| `priority_queue` push/pop | O(log n); top is O(1) |
| `sort` | O(n log n) (introsort) |
| `vector::push_back` | O(1) amortised (capacity doubles) |
| BFS / DFS | O(V + E) |
| Dijkstra with a binary heap | O((V + E) log V) |
| DSU with path compression + union by size | O(α(n)) per operation |
| Building a heap from an array | O(n) |
| Binary search | O(log n) |

**Input-size hints:**

| n up to | Target complexity |
|---|---|
| ≈ 10⁵–10⁶ | O(n) or O(n log n) |
| ≈ 10³–10⁴ | O(n²) |
| ≈ 20 | O(2ⁿ) bitmask |
| ≈ 10 | O(n!) |
