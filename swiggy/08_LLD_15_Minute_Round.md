# LLD in 15 Minutes: The Last Part of Round 1

> **Your round:** 45 minutes of coding/PSDS, then **15 minutes of LLD** (low-level design). Questions may be in **Java or Go**.
> **What this file does:** teaches you how to use those 15 minutes, then walks through the 9 most likely questions. Each comes with a class sketch you can write fast. Read it in about 1.5 hours, then practise each problem against a 15-minute timer.
> **Need the basics first?** OOP, SOLID and the full patterns are in `03_Machine_Coding_and_LLD_From_Zero.md`, Parts 1–2. Read those before this file if words like "interface" or "Strategy pattern" feel unclear.
> All code here was compiled (Java 21, Go 1.26).

---

## Part 0: What a 15-minute LLD round is (and isn't)

**LLD** means designing the *inside* of one piece of software: what classes exist, what each one is responsible for, and how they talk to each other. Think of it like planning a restaurant kitchen. You don't build it in 15 minutes. You show *who does what*: the order counter takes orders, the chef cooks, the billing desk charges.

**15 minutes is short.** It is **not** a machine coding round, where you'd build a full working app in 60–90 minutes. In 15 minutes they want to see:

1. That you **ask what's needed** before designing.
2. That you can pick **sensible classes and interfaces**.
3. That your design is **easy to extend**: adding a new feature shouldn't mean rewriting old code.
4. One or two **key methods** written properly.

They do **not** expect every getter, setter and edge case, or a running `main`.

**What the reports say Swiggy asked in short LLD slots** (from `01`):
- 2026 ASDE Round 1: "design a stack using an array", with the interviewer asking for classes and interfaces.
- 2026 SDE-1: driver–order assignment (VIP vs normal orders, driver ratings).
- 2025 SDE-1: food ordering cart with discounts (the candidate used the Strategy pattern and was selected).
- Splitwise (the most reported LLD problem overall).
- Follow/unfollow (schema + APIs), parking lot (with a Singleton follow-up), rate limiter.

---

## Part 1: The 15-minute clock

| Minutes | What you do | What you say |
|---|---|---|
| 0–2 | **Clarify.** Ask 3–4 questions and write the answers down as a short list of requirements. | "Before designing, let me confirm the scope…" |
| 2–4 | **Nouns → classes.** Underline the nouns in the requirements: Order, Driver, Cart… | "The main entities are…" |
| 4–6 | **Find what changes → interface.** Anything with several variants (discount types, payment modes, matching rules) becomes an interface. | "Discount rules will keep changing, so I'll put them behind an interface." |
| 6–12 | **Write the skeleton.** Interfaces, classes with key fields, and 1–2 core methods written fully. | Narrate: "The service holds the logic; the entities just hold data." |
| 12–15 | **Extend and wrap up.** Show how a new requirement fits, and mention thread safety and storage. | "To add a new discount type, I just add one class; nothing else changes." |

**The golden rule: write the interface first.** It shows design thinking in 30 seconds, even if you run out of time for the rest.

---

## Part 2: The tiny toolkit

### 2.1 Four habits that make any design look good

1. **Entities hold data; services hold logic.** `Order` knows its items and status. `OrderService` knows how to place and cancel orders. *Kitchen analogy:* the ingredients don't cook themselves; the chef does.
2. **Program to an interface.** `PaymentMethod pm = new UpiPayment();` The rest of the code only knows "it can pay", so swapping UPI for a card changes nothing else.
3. **Use enums for fixed states.** `OrderStatus { PLACED, PREPARING, PICKED_UP, DELIVERED, CANCELLED }`. Never use raw strings like `"delivered"`.
4. **Name the extension point out loud.** "If tomorrow we add X, only this class changes." That sentence is what interviewers listen for. It's the Open/Closed principle: open for adding, closed for editing.

### 2.2 The 5 patterns, one line each

| Pattern | What it is | Swiggy example | Use when you hear… |
|---|---|---|---|
| **Strategy** | Swap the algorithm behind one interface | Discount rules, driver-matching rules, split types | "different types of…", "rules may change" |
| **Factory** | One place that decides which class to create | `NotifierFactory.get("SMS")` | "create X based on type" |
| **Observer** | When something happens, notify all subscribers | Order status changes → notify customer, restaurant, rider | "notify", "when X happens, update Y" |
| **Singleton** | Exactly one instance, shared everywhere | One `ParkingLot`, one config, one DB pool | "only one", "global" |
| **State** | The object behaves differently depending on its state | An order can't be cancelled after pickup | "status", "lifecycle" |

### 2.3 Interfaces in Java vs Go (in case they ask for Go)

```java
// Java: explicit "implements"
interface DiscountStrategy {
    double apply(double amount);
}
class FlatDiscount implements DiscountStrategy {
    private final double off;
    FlatDiscount(double off) { this.off = off; }
    public double apply(double amount) { return Math.max(0, amount - off); }
}
```

```go
// Go: implicit. Having the method is enough.
type DiscountStrategy interface {
	Apply(amount float64) float64
}

type FlatDiscount struct{ Off float64 }

func (f FlatDiscount) Apply(amount float64) float64 { return math.Max(0, amount-f.Off) }
```

Go has no classes or inheritance. Use **structs** for data, **methods** for behaviour, **interfaces** for the swappable parts, and a **constructor function** `NewX(...)` in place of a constructor. Everything else in this file translates directly.

---

## Part 3: The 9 likely questions

Each question has: questions to ask → requirements → the design → code sketch → how to extend → follow-ups.

---

### Q1. Design a stack using an array (asked: 2026 ASDE Round 1)

**Why it's an LLD question:** they want to see you hide *how* the stack works (an array) behind *what* it does (an interface). Then you could swap in a linked-list version without touching the code that uses it.

**Ask:** Fixed size or growable? What happens on pop from empty: an exception or a null? Generic type, or ints only?

**Design:** an interface `Stack<T>` and one implementation `ArrayStack<T>` that doubles its array when full.

```java
interface Stack<T> {
    void push(T item);
    T pop();          // throws if empty
    T peek();
    boolean isEmpty();
    int size();
}

class ArrayStack<T> implements Stack<T> {
    private Object[] data = new Object[4];
    private int top = 0;                       // next free slot = current size

    public void push(T item) {
        if (top == data.length) data = java.util.Arrays.copyOf(data, data.length * 2); // grow
        data[top++] = item;
    }
    @SuppressWarnings("unchecked")
    public T pop() {
        if (isEmpty()) throw new RuntimeException("Stack is empty");
        T item = (T) data[--top];
        data[top] = null;                      // let GC reclaim it (avoid a memory leak)
        return item;
    }
    @SuppressWarnings("unchecked")
    public T peek() {
        if (isEmpty()) throw new RuntimeException("Stack is empty");
        return (T) data[top - 1];
    }
    public boolean isEmpty() { return top == 0; }
    public int size() { return top; }
}
```

**Say:** "push and pop are O(1). Growing is O(n), but it happens rarely because we double each time, so the average stays O(1); that's called *amortised* O(1). I set the popped slot to null so the garbage collector can free the object. Because callers use the `Stack` interface, I could add a `LinkedStack` later without changing them."

**Follow-ups:**
- *"Make it thread-safe?"* Mark the methods `synchronized`, or use a lock. In Go, use a `sync.Mutex`.
- *"Add getMin in O(1)?"* Keep a second stack holding the minimum so far (LC 155; solved in `02` as P3).
- *"Why not `T[]`?"* Java can't create generic arrays directly, so we use `Object[]` and cast.

---

### Q2. Driver–order assignment (asked: 2026 SDE-1)

**Ask:** How do we match a driver today: nearest, or highest-rated? What makes VIP different? Can a driver hold more than one order? What if no driver is free?

**Requirements (example answers):**
- Orders are VIP or normal. **VIP orders get a driver rated ≥ 4.5**; normal orders get any free driver.
- Among eligible drivers, pick the **nearest**.
- One order per driver at a time. If nobody is eligible, the order waits.

**Design idea:** the *matching rule* is the thing that changes, so it's an interface (Strategy).

```java
enum OrderType { VIP, NORMAL }

class Order {
    final String id; final OrderType type; final double x, y;
    Order(String id, OrderType type, double x, double y) { this.id = id; this.type = type; this.x = x; this.y = y; }
}

class Driver {
    final String id; final double rating; double x, y; boolean available = true;
    Driver(String id, double rating, double x, double y) { this.id = id; this.rating = rating; this.x = x; this.y = y; }
    double distanceTo(Order o) { return Math.hypot(x - o.x, y - o.y); }
}

// The part that changes: who is allowed to take which order.
interface EligibilityRule { boolean isEligible(Driver d, Order o); }

class VipRule implements EligibilityRule {
    public boolean isEligible(Driver d, Order o) { return d.rating >= 4.5; }
}
class NormalRule implements EligibilityRule {
    public boolean isEligible(Driver d, Order o) { return true; }
}

class AssignmentService {
    private final java.util.List<Driver> drivers;
    private final java.util.Map<OrderType, EligibilityRule> rules = new java.util.HashMap<>();

    AssignmentService(java.util.List<Driver> drivers) {
        this.drivers = drivers;
        rules.put(OrderType.VIP, new VipRule());
        rules.put(OrderType.NORMAL, new NormalRule());
    }

    // Returns the assigned driver, or null if nobody is eligible (the caller queues the order).
    synchronized Driver assign(Order order) {
        EligibilityRule rule = rules.get(order.type);
        Driver best = null;
        for (Driver d : drivers) {
            if (!d.available || !rule.isEligible(d, order)) continue;
            if (best == null || d.distanceTo(order) < best.distanceTo(order)) best = d;
        }
        if (best != null) best.available = false;
        return best;
    }
}
```

**The same idea in Go** (if they want Go):

```go
type EligibilityRule interface {
	IsEligible(d *Driver, o Order) bool
}

type VipRule struct{}

func (VipRule) IsEligible(d *Driver, o Order) bool { return d.Rating >= 4.5 }

type AssignmentService struct {
	mu      sync.Mutex
	drivers []*Driver
	rules   map[OrderType]EligibilityRule
}

func (s *AssignmentService) Assign(o Order) *Driver {
	s.mu.Lock()
	defer s.mu.Unlock()
	var best *Driver
	for _, d := range s.drivers {
		if !d.Available || !s.rules[o.Type].IsEligible(d, o) {
			continue
		}
		if best == nil || d.DistanceTo(o) < best.DistanceTo(o) {
			best = d
		}
	}
	if best != nil {
		best.Available = false
	}
	return best
}
```

**Say:** "`assign` is locked, so two orders can't grab the same driver at once; that's a race condition otherwise. Adding a new order type, say 'Swiggy One members', means one new rule class and one map entry. The service doesn't change."

**Follow-ups:**
- *"Scanning all drivers is slow for a whole city."* Only look at drivers in the order's area. Group drivers by zone or geohash (a short code for a map grid square), or keep a priority queue per zone. Real Swiggy batches orders over a short window and solves a matching problem per zone (`../sources/india_companies_engineering.md`).
- *"No VIP driver available?"* Queue the order and retry every few seconds. After a timeout, relax the rule, for example accept rating ≥ 4.0. That relaxation is just another rule.
- *"Pick by rating instead of distance?"* Make the *ranking* a second Strategy: `Comparator<Driver>`.

---

### Q3. Food cart with discounts (asked: Dec 2025 SDE-1, Strategy got selected)

**Ask:** Which discount types: flat ₹ off, percentage with a cap, buy-one-get-one? Can discounts stack, or is only one applied? Is there a minimum order value? Is the cart limited to one restaurant?

**Requirements:** add/remove items; one restaurant per cart; compute the bill; apply **one** offer (the best one); offers can have a minimum order value.

```java
class MenuItem {
    final String id, name; final double price;
    MenuItem(String id, String name, double price) { this.id = id; this.name = name; this.price = price; }
}

interface Offer {
    boolean isApplicable(double subtotal);
    double discount(double subtotal);       // how many rupees off
}

class FlatOffer implements Offer {           // "₹100 off above ₹499"
    private final double off, minOrder;
    FlatOffer(double off, double minOrder) { this.off = off; this.minOrder = minOrder; }
    public boolean isApplicable(double s) { return s >= minOrder; }
    public double discount(double s) { return Math.min(off, s); }
}

class PercentOffer implements Offer {        // "20% off up to ₹120"
    private final double pct, cap;
    PercentOffer(double pct, double cap) { this.pct = pct; this.cap = cap; }
    public boolean isApplicable(double s) { return true; }
    public double discount(double s) { return Math.min(s * pct / 100, cap); }
}

class Cart {
    private final String restaurantId;
    private final java.util.Map<MenuItem, Integer> items = new java.util.LinkedHashMap<>();
    Cart(String restaurantId) { this.restaurantId = restaurantId; }

    void add(MenuItem item, int qty) { items.merge(item, qty, Integer::sum); }
    void remove(MenuItem item) { items.remove(item); }
    double subtotal() {
        double s = 0;
        for (var e : items.entrySet()) s += e.getKey().price * e.getValue();
        return s;
    }
}

class BillingService {
    // Pick the offer that saves the customer the most.
    double finalAmount(Cart cart, java.util.List<Offer> offers) {
        double sub = cart.subtotal(), best = 0;
        for (Offer o : offers) if (o.isApplicable(sub)) best = Math.max(best, o.discount(sub));
        return sub - best;
    }
}
```

**Say:** "Each offer type is a Strategy. Marketing adds new offer types every week, so a new one is a new class, and `BillingService` never changes. `Cart` only holds items. Billing logic is separate, so I can add taxes or delivery fees in one place."

**Follow-ups:**
- *"Allow stacking two offers?"* Add a `CompositeOffer` that holds a list of offers and applies them in order. That's the Composite pattern.
- *"Use Go?"* `Offer` becomes a Go interface; `Cart` is a struct with `map[string]int` (item ID → qty).
- *"Money as double?"* "In production I'd store paise as a `long`, or use `BigDecimal`, because doubles have rounding errors. I used double here only for speed."

---

### Q4. Splitwise (the most reported LLD at Swiggy)

**Ask:** Which split types: equal, exact amounts, percentages? Groups, or just users? Do we need "simplify debts"?

**Core idea:** keep a **balance sheet**: `balances[A][B] = 200` means A owes B ₹200. Split *types* are a Strategy.

```java
interface SplitStrategy {
    // returns how much each participant owes for this expense
    java.util.Map<String, Double> split(double amount, java.util.List<String> users, java.util.List<Double> values);
}

class EqualSplit implements SplitStrategy {
    public java.util.Map<String, Double> split(double amt, java.util.List<String> users, java.util.List<Double> v) {
        java.util.Map<String, Double> res = new java.util.HashMap<>();
        for (String u : users) res.put(u, amt / users.size());
        return res;
    }
}

class PercentSplit implements SplitStrategy {
    public java.util.Map<String, Double> split(double amt, java.util.List<String> users, java.util.List<Double> pct) {
        double total = 0;
        for (double p : pct) total += p;
        if (Math.abs(total - 100) > 0.01) throw new IllegalArgumentException("Percents must sum to 100");
        java.util.Map<String, Double> res = new java.util.HashMap<>();
        for (int i = 0; i < users.size(); i++) res.put(users.get(i), amt * pct.get(i) / 100);
        return res;
    }
}

class ExpenseService {
    // balances.get(a).get(b) > 0 means a owes b that much
    private final java.util.Map<String, java.util.Map<String, Double>> balances = new java.util.HashMap<>();

    void addExpense(String paidBy, double amount, java.util.List<String> users,
                    java.util.List<Double> values, SplitStrategy strategy) {
        java.util.Map<String, Double> shares = strategy.split(amount, users, values);
        for (var e : shares.entrySet()) {
            String user = e.getKey();
            if (user.equals(paidBy)) continue;
            update(user, paidBy, e.getValue());   // user owes payer
            update(paidBy, user, -e.getValue());  // mirror entry
        }
    }
    private void update(String from, String to, double amt) {
        balances.computeIfAbsent(from, k -> new java.util.HashMap<>()).merge(to, amt, Double::sum);
    }
    double owes(String a, String b) {
        return balances.getOrDefault(a, java.util.Map.of()).getOrDefault(b, 0.0);
    }
}
```

**Say:** "Each split type validates its own input; for example, percents must add up to 100. The service only updates balances. A new split type, like 'by shares', is one new class."

**Follow-ups:**
- *"Simplify debts?"* Compute each person's **net** amount (total owed to them minus total they owe). Then repeatedly match the person who is owed the most with the person who owes the most. That gives at most n−1 transactions.
- *"Rounding: ₹100 split 3 ways?"* Give ₹33.33 to two people and ₹33.34 to the first, so the shares add up exactly.

---

### Q5. Parking lot (asked, with a Singleton follow-up)

**Ask:** How many floors? Which vehicle types and spot sizes? Do we charge by the hour? Is there one entrance or several?

**Entities:** `ParkingLot` (Singleton) → `Floor` → `Spot` (with a size), `Vehicle` (with a type), `Ticket`, and a `FeeStrategy`.

```java
enum Size { SMALL, MEDIUM, LARGE }   // bike, car, truck

class Spot {
    final String id; final Size size; boolean free = true;
    Spot(String id, Size size) { this.id = id; this.size = size; }
}

class Ticket {
    final String vehicleNo; final Spot spot; final long entryMillis = System.currentTimeMillis();
    Ticket(String vehicleNo, Spot spot) { this.vehicleNo = vehicleNo; this.spot = spot; }
}

class ParkingLot {
    private static volatile ParkingLot instance;          // volatile: see Q14 in file 07
    private final java.util.List<Spot> spots = new java.util.ArrayList<>();
    private ParkingLot() {}

    static ParkingLot getInstance() {                     // double-checked locking
        if (instance == null) {
            synchronized (ParkingLot.class) {
                if (instance == null) instance = new ParkingLot();
            }
        }
        return instance;
    }
    void addSpot(Spot s) { spots.add(s); }

    synchronized Ticket park(String vehicleNo, Size size) {
        for (Spot s : spots) {
            if (s.free && s.size == size) { s.free = false; return new Ticket(vehicleNo, s); }
        }
        return null;                                      // lot full for this size
    }
    synchronized void leave(Ticket t) { t.spot.free = true; }
}
```

**Singleton follow-up, which they ask:** *"Why the double null-check and `volatile`?"*
> "The first check skips the lock in the common case, when the instance already exists, so it's fast. The second check inside the lock stops two threads that both passed the first check from creating two objects. `volatile` makes sure no thread sees a half-built object. A simpler safe option is an `enum` Singleton, or `sync.Once` in Go."

**Faster spot lookup:** keep one queue of free spots per size: `Map<Size, Queue<Spot>>`. Parking becomes O(1) instead of scanning every spot.

---

### Q6. Rate limiter (mentioned in Swiggy reports; you built one at Myntra)

**Ask:** Limit per user or per API? How many requests per second? Do we reject extra requests, or queue them?

**Token bucket in one breath:** each user has a bucket holding up to `capacity` tokens. It refills at `rate` tokens per second. Each request takes one token; if the bucket is empty, reject. *Real-life:* a bakery with 10 free samples on a tray. Staff add one every second, and when the tray is empty you wait.

```java
interface RateLimiter { boolean allow(String userId); }

class TokenBucketLimiter implements RateLimiter {
    private final int capacity; private final double refillPerSec;
    private final java.util.Map<String, double[]> buckets = new java.util.concurrent.ConcurrentHashMap<>();
    // each bucket = {tokens, lastRefillMillis}

    TokenBucketLimiter(int capacity, double refillPerSec) { this.capacity = capacity; this.refillPerSec = refillPerSec; }

    public boolean allow(String userId) {
        double[] b = buckets.computeIfAbsent(userId, k -> new double[]{capacity, System.currentTimeMillis()});
        synchronized (b) {                                    // lock only this user's bucket
            long now = System.currentTimeMillis();
            b[0] = Math.min(capacity, b[0] + (now - b[1]) / 1000.0 * refillPerSec); // refill
            b[1] = now;
            if (b[0] >= 1) { b[0] -= 1; return true; }
            return false;
        }
    }
}
```

**Say:** "`RateLimiter` is an interface, so a fixed-window or sliding-window limiter can be swapped in. I lock per user, not globally, so users don't block each other. Across many servers I'd keep the counters in Redis and update them atomically with a Lua script."

---

### Q7. Notification system (Observer)

**Ask:** Which channels: SMS, email, push? Which events: order placed, out for delivery, delivered? Do users choose their channels?

```java
enum OrderStatus { PLACED, PREPARING, OUT_FOR_DELIVERY, DELIVERED }

interface OrderObserver { void onStatusChange(String orderId, OrderStatus status); }

class PushNotifier implements OrderObserver {
    public void onStatusChange(String id, OrderStatus s) { System.out.println("Push: order " + id + " is " + s); }
}
class SmsNotifier implements OrderObserver {
    public void onStatusChange(String id, OrderStatus s) { System.out.println("SMS: order " + id + " is " + s); }
}

class OrderTracker {                                     // the "subject"
    private final java.util.List<OrderObserver> observers = new java.util.ArrayList<>();
    void subscribe(OrderObserver o) { observers.add(o); }
    void updateStatus(String orderId, OrderStatus s) {
        for (OrderObserver o : observers) o.onStatusChange(orderId, s);
    }
}
```

**Say:** "`OrderTracker` doesn't know who's listening. Adding WhatsApp notifications is one new class plus a `subscribe` call. In a real system the observer list becomes a **Kafka topic**: the order service publishes 'status changed', and the notification service consumes it. Then a slow SMS provider can't slow down orders. It's the same idea as the event-driven pipeline I built at Atlassian."

---

### Q8. Follow / unfollow (asked: Dec 2024 ASDE, discussion only)

This one is **schema + APIs**, not classes. Talk it through.

**Tables:**
```sql
users(user_id PK, name, created_at)
follows(follower_id, followee_id, created_at,
        PRIMARY KEY (follower_id, followee_id))   -- the same pair can't be followed twice
CREATE INDEX idx_followee ON follows(followee_id); -- fast "who follows me?"
```

**APIs:**

| Action | Method + path | Why this HTTP verb |
|---|---|---|
| Follow | `POST /users/{id}/follow` | Creates a relationship |
| Unfollow | `DELETE /users/{id}/follow` | Removes it |
| Followers list | `GET /users/{id}/followers?cursor=…&limit=20` | Read-only; paginated |
| Following list | `GET /users/{id}/following?cursor=…&limit=20` | Read-only |

**Say:**
- "The composite primary key stops duplicate follows, and makes 'whom do I follow?' fast. The extra index on `followee_id` makes 'who follows me?' fast."
- "Follow is **idempotent** (repeating it has the same effect as doing it once): following twice just returns success. I use `INSERT … ON DUPLICATE KEY` or ignore the duplicate error."
- "Follower counts: store a counter column and update it on follow/unfollow, rather than running `COUNT(*)` on every profile view."
- "For pagination I use a cursor, which says 'give me 20 after this ID', rather than a page number with OFFSET. OFFSET gets slower the deeper you go."

---

### Q9. LRU cache (the single most-quoted design question at Swiggy)

**What it is:** a fixed-size cache that throws out the **least recently used** item when full. *Real-life:* a small fridge at a restaurant counter. When a new item comes in and it's full, out goes whatever nobody has touched for the longest time.

**Design:** a **HashMap** (key → node) for O(1) lookup, plus a **doubly linked list** to keep the usage order: most recent at the front, least recent at the back. Full code is in `02` (P31). In a 15-minute LLD, show the interface and say the trick:

```java
interface Cache<K, V> {
    V get(K key);          // O(1): look up the node in the map, move it to the front
    void put(K key, V val); // O(1): insert at the front; if over capacity, drop the back node
}
```

**Quick Java version (good to mention):** `LinkedHashMap` with `accessOrder = true`, overriding `removeEldestEntry`:

```java
class LRUCache<K, V> extends java.util.LinkedHashMap<K, V> {
    private final int capacity;
    LRUCache(int capacity) { super(16, 0.75f, true); this.capacity = capacity; } // true = access order
    @Override protected boolean removeEldestEntry(java.util.Map.Entry<K, V> eldest) { return size() > capacity; }
}
```

**Say:** "If they want it from scratch I'll write the map + doubly linked list. For thread safety I'd wrap get/put in a lock. Note that `get` also *changes* the order, so even reads need the lock."

---

## Part 4: Common mistakes in short LLD rounds

| Mistake | Do this instead |
|---|---|
| Starting to code straight away | Spend 2 minutes on questions, and write the requirements down |
| One giant `Manager` class doing everything | Entities hold data; small services hold logic |
| `if (type == "VIP") … else if …` chains | Put the varying rule behind an interface (Strategy) |
| Spending 10 minutes on getters/setters | Write the interface and 1–2 core methods; say "the rest is boilerplate" |
| Ignoring concurrency | One sentence: "two requests could grab the same driver/spot, so this method is synchronized / uses a mutex" |
| Silence | Narrate every decision. In 15 minutes, *what you say* is half the score |
| Not finishing | Interfaces first, so even a half-done design shows the structure |

---

## Part 5: Practice plan (4 days, 30 minutes a day)

| Day | Practice (set a 15-minute timer for each) |
|---|---|
| 1 | Q1 stack (Java, then Go) and Q3 food cart |
| 2 | Q2 driver assignment and Q4 Splitwise |
| 3 | Q5 parking lot (say the Singleton answer out loud) and Q6 rate limiter |
| 4 | Q7 notifications, Q8 follow/unfollow (talk only) and Q9 LRU. Then redo the one that felt worst. |

**Final checklist**
- [ ] I can write an interface + 2 implementations in both Java and Go in under 3 minutes.
- [ ] I know when to say Strategy, Factory, Observer, Singleton or State.
- [ ] I can explain double-checked locking in 30 seconds.
- [ ] For every design, I can say "to add X, only this class changes".
- [ ] I mention thread safety and storage (DB/Redis) in the last 2 minutes.
