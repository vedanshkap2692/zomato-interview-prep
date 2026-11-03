# Machine Coding and LLD from Zero (Swiggy)

> **Who this is for:** you, if you have never sat a machine coding or low-level design round, and words like "Strategy pattern" or "SOLID" feel vague.
> **Reading time:** about 3 hours, including the code. Read Parts 0–3 first (about 75 minutes). Then do one worked problem per day: read the requirements, try it yourself for 45 minutes, then compare with the solution.
> **Language:** all code is **Java 17+**. In 2026, Swiggy reports say the DSA and OA rounds were restricted to **Java or Go**, and LLD rounds are almost always done in Java. Every program here was compiled and run before it went into this document.
> **Related files:** process and all past questions are in `01`, DSA in `02`, Java/Spring theory in `05`.

---

## Contents

- **Part 0:** What these rounds are, and how Swiggy runs them
- **Part 1:** OOP from zero
- **Part 2:** SOLID and the five design patterns that actually come up
- **Part 3:** The playbook for a 60–90 minute round
- **Part 4:** Five complete worked solutions
  1. Snake & Ladder
  2. Splitwise
  3. Food ordering cart with discounts
  4. Driver–order assignment (VIP vs normal)
  5. Magical Arena (the ASDE take-home)
- **Part 5:** Discussion-style LLD
  - follow/unfollow
  - job scheduler
  - parking lot
  - library
  - LRU cache
  - rate limiter
  - calendar
- **Part 6:** Final checklist and practice plan

---

# Part 0: What these rounds are

## 0.1 Three words you need first

**Design** in software means deciding *how to split a program into pieces* and *how those pieces talk to each other*, before or while you write code.

There are two levels of design. Think of building a restaurant.

| Level | Restaurant analogy | Software meaning | Example question |
|---|---|---|---|
| **HLD**: High-Level Design | Which building, how many kitchens, where the delivery counter goes, how many cooks per shift | Which **servers, databases, caches and queues** you need, and how data flows between them | "Design Swiggy's order tracking for 1 million users" |
| **LLD**: Low-Level Design | Inside one kitchen: which station makes what, who passes the plate to whom, what the order slip looks like | Which **classes, methods and data structures** you write inside **one program** | "Design the classes for a Splitwise app" |

**Machine coding** is LLD where you must actually **write the whole program**, and it must **run**. You get a problem statement ("build a Snake & Ladder game"), 60–120 minutes, and your own laptop and editor. At the end the interviewer runs your code, tries some inputs, and asks how you would add new features.

**An "LLD round" without heavy coding** is a discussion. You talk through classes, a database schema and API endpoints, and maybe write the interfaces. Swiggy uses both styles.

## 0.2 How Swiggy runs these rounds (from ~38 candidate reports, 2021–2026)

| What | What reports say |
|---|---|
| **When it happens** | 2021–2022: often the **first** live round, "plays a crucial role in selection". 2024–2026: usually **after** a DSA round. |
| **Duration** | Old reports: 2–3 hours. 2024–2026: about **60 minutes**. One 2022 breakdown of 2 hours: 15 min explanation, 90 min coding, 15 min discussion. |
| **Tools** | Your own IDE (VS Code, IntelliJ) with screen sharing. Sometimes an online IDE. |
| **Most-asked problem** | **Splitwise** (5 reports). |
| **Other problems** | Snake & Ladder; parking lot (with a Singleton follow-up); calendar/meeting scheduler; **food ordering cart with discounts** (Dec 2025); **driver–order assignment with VIP orders** (Mar 2026); job scheduler; library app. |
| **ASDE (fresher) version** | A **take-home game, "Magical Arena"**: a two-player attack/defend game in Java, Node or Go. Then a discussion LLD round, e.g. **follow/unfollow** (schema + API endpoints + HTTP verbs). In Mar 2026: "design a stack using an array" with classes and interfaces. |

**What the interviewers looked at**, in candidates' own words:
- "OOP classes (Player, Game, Board) and extensibility"
- "Code must run without errors"
- "Wanted a fully working application… keen to see actual code"
- "Design, choose data structures, code, then dry run"
- "How would you add the features you didn't finish?"
- "Used Strategy pattern for discounts" (that candidate was selected)

## 0.3 Why people failed

1. **The code didn't run in time.** The Oct 2025 Splitwise candidate spent too long on requirements and UML diagrams, ended with partial code, and was rejected. This is the number one reason.
2. **Missed edge cases.** For example, they didn't handle invalid input or an empty cart.
3. **Being slow.** One senior candidate wrote "the machine coding round has probably cost me".

**The lesson:** a simple design that **runs** beats a beautiful design that doesn't. Get a working core first, then improve it.

---

# Part 1: OOP from zero

**OOP (Object-Oriented Programming)** means organising code around **things** (objects) that hold their own **data** and their own **behaviour**. Machine coding is judged almost entirely on how well you do this.

## 1.1 Class and object

- A **class** is a **blueprint**. It says what data a thing has and what it can do.
- An **object** is **one real thing made from that blueprint**.

**Real life:** the Swiggy menu card design, "every dish has a name, a price and a veg/non-veg tag", is the class. "Chicken Biryani, Rs 320, non-veg" is one object, and "Paneer Biryani, Rs 280, veg" is another.

```java
class MenuItem {
    String name;          // data (also called "fields" or "attributes")
    long pricePaise;
    boolean veg;

    MenuItem(String name, long pricePaise, boolean veg) {   // constructor: runs when an object is made
        this.name = name;
        this.pricePaise = pricePaise;
        this.veg = veg;
    }
}

MenuItem biryani = new MenuItem("Chicken Biryani", 32000, false);   // an object
```

> **Why paise and not rupees?** Money is stored as a whole number of paise (`long`). Never store it in a `double`: in Java, `0.1 + 0.2` gives `0.30000000000000004`. Interviewers notice this detail and like it.

## 1.2 The four pillars

### Encapsulation: keep your data private and protect it with rules

**Idea:** an object's data is `private`. Other code can change it only through methods that **check the rules**.

**Real life:** a bank locker. You can't walk into the vault and take money. You ask the cashier, and the cashier checks your balance first.

```java
class Wallet {
    private long balancePaise;                 // nobody outside can touch this directly

    void add(long amount) {
        if (amount <= 0) throw new IllegalArgumentException("Amount must be positive");
        balancePaise += amount;
    }
    void spend(long amount) {
        if (amount > balancePaise) throw new IllegalStateException("Not enough balance");
        balancePaise -= amount;
    }
    long getBalance() { return balancePaise; }
}
```

If `balancePaise` were public, any code could write `wallet.balancePaise = -500`, and nothing would stop it.

### Abstraction: show *what* it does, hide *how*

**Idea:** the user of a class sees a simple method like `pay(amount)`. The messy details stay inside.

**Real life:** you press "Pay" on Swiggy. You don't see the bank calls, retries and OTP checks. You only see "Payment successful".

### Inheritance: a child class reuses a parent class

**Idea:** `class UpiPayment extends PaymentMethod` means a UPI payment **is a** payment method. It automatically gets the parent's fields and methods, and adds or changes its own.

```java
abstract class PaymentMethod {
    protected final String ownerName;
    PaymentMethod(String ownerName) { this.ownerName = ownerName; }
    abstract boolean pay(long amountPaise);            // every child MUST say how it pays
    void printReceipt(long amount) {                   // shared by all children
        System.out.println(ownerName + " paid " + amount + " paise");
    }
}

class UpiPayment extends PaymentMethod {
    private final String upiId;
    UpiPayment(String owner, String upiId) { super(owner); this.upiId = upiId; }
    boolean pay(long amount) { System.out.println("UPI collect request to " + upiId); return true; }
}

class CardPayment extends PaymentMethod {
    CardPayment(String owner) { super(owner); }
    boolean pay(long amount) { System.out.println("Charging card, OTP sent"); return true; }
}
```

### Polymorphism: one call, many behaviours

**Idea:** you hold a variable of the **parent type** and call a method. Java runs the **child's** version, based on what the object really is.

**Real life:** the Swiggy checkout says "pay". Whether it becomes a UPI request or a card OTP depends on which method the user picked. The checkout code doesn't care.

```java
PaymentMethod[] methods = { new UpiPayment("Asha", "asha@okbank"), new CardPayment("Ravi") };
for (PaymentMethod m : methods) m.pay(50000);   // UPI version runs first, then the card version
```

**Two kinds of polymorphism** (asked in a Swiggy campus interview):

| Kind | Also called | Decided when | Example |
|---|---|---|---|
| **Compile-time** | static, **method overloading** | When the code compiles | `add(int a, int b)` and `add(double a, double b)`: same name, different parameters |
| **Run-time** | dynamic, **method overriding** | While the program runs | `UpiPayment.pay()` replaces `PaymentMethod.pay()`. Java picks the version from the actual object. |

## 1.3 Interface vs abstract class (asked in Dec 2025)

An **interface** is a **contract**: a list of methods a class promises to have, with no data. An **abstract class** is a **half-built parent**: it can have fields, constructors and some finished methods, plus some unfinished (`abstract`) ones.

**Real life:**
- **Interface:** "anything that can deliver food must have `pickUp()` and `drop()`." A bike rider, a cycle rider and a drone can all sign that contract, even though they share nothing else.
- **Abstract class:** "every Swiggy payment method has an owner name and prints receipts the same way, but each one pays differently."

| | Interface | Abstract class |
|---|---|---|
| Fields (data) | Only constants | Yes, normal fields |
| Constructor | No | Yes |
| Method bodies | Only `default`/`static` methods (Java 8+) | Yes, any |
| How many can a class use? | **Many** (`implements A, B, C`) | **Only one** (`extends X`) |
| Use when | Unrelated classes share a **capability** | Closely related classes share **code and state** |

**Rule of thumb for machine coding:** start with **interfaces** for anything that may have several versions (`DiscountStrategy`, `Dice`, `Notifier`). Use an abstract class only if the versions genuinely share fields.

## 1.4 Composition over inheritance (asked: "inheritance vs composition")

- **Inheritance** is an **is-a** link: a `UpiPayment` *is a* `PaymentMethod`.
- **Composition** is a **has-a** link: an `OrderService` *has a* `Notifier`.

**Real life:** a Swiggy delivery partner **has** a vehicle. They are not a vehicle. If the partner switches from bike to cycle, you swap the vehicle object. You don't rebuild the person.

```java
interface Notifier { void send(String userId, String message); }
class SmsNotifier implements Notifier { public void send(String u, String m) { System.out.println("SMS to " + u + ": " + m); } }
class PushNotifier implements Notifier { public void send(String u, String m) { System.out.println("Push to " + u + ": " + m); } }

class OrderService {
    private final Notifier notifier;                     // HAS-A
    OrderService(Notifier notifier) { this.notifier = notifier; }
    void markDelivered(String userId) { notifier.send(userId, "Your order is delivered"); }
}

new OrderService(new SmsNotifier());   // swap in PushNotifier with no change to OrderService
```

**Why prefer composition:**
- Inheritance chains get rigid. For example, `class SmsOrderService extends OrderService` would need a new class for every channel.
- With composition you plug parts in and out. That is what interviewers mean by "extensible".

Passing the part in through the constructor, as above, is called **dependency injection**. Spring Boot does this for you automatically.

## 1.5 Java features used in this document

| Feature | One-line meaning | Example |
|---|---|---|
| `private` / `public` / `protected` | Who can see a field or method | `private long balance;` |
| `final` | Cannot be changed after it is set | `private final String id;` |
| `static` | Belongs to the class, not to each object | `static int count;` is shared by all objects |
| `enum` | A fixed list of named values | `enum OrderStatus { PLACED, DELIVERED }` |
| `record` (Java 16+) | A short way to write a class that just holds data. It gets a constructor, getters, `equals`, `hashCode` and `toString` for free. | `record User(String id, String name) {}` then `user.name()` |
| `Optional<T>` | "Maybe a value, maybe nothing." Safer than returning `null`. | `Optional<Driver> pick(...)` |
| Exceptions | A way to stop and say "this input is wrong" | `throw new IllegalArgumentException("Unknown user")` |
| `Map`, `List`, `Set` | Key→value lookup, ordered list, unique items | `Map<String, User> users = new HashMap<>();` |
| `Deque` / `PriorityQueue` | A queue you can use from both ends, and a queue that always gives the smallest (or highest-priority) item first | Players' turns, waiting orders |
| `static class` inside a class | Lets you keep everything in one file during the round | `static class Player { ... }` |

---

# Part 2: SOLID and the patterns that actually come up

## 2.1 SOLID in simple words

SOLID is five rules for keeping classes easy to change. You don't need to recite them, but you should *write code that follows them* and be able to explain one or two with an example.

| Letter | Name | Plain meaning | Swiggy example |
|---|---|---|---|
| **S** | Single Responsibility | One class has **one job**. | `Cart` holds items. `BillCalculator` computes the bill. `OrderService` places orders. Don't put all three in one `Cart` class. |
| **O** | Open/Closed | **Add** new behaviour by adding new classes, not by **editing** old, working code. | A new "Rs 50 off on first order" offer is a new `DiscountStrategy` class. You don't touch the bill code. |
| **L** | Liskov Substitution | A child must work anywhere its parent works, without surprises. | If `CashOnDelivery` is a `PaymentMethod`, calling `pay()` on it must not throw "not supported". |
| **I** | Interface Segregation | Many small interfaces beat one giant one. | Don't force a `CycleRider` to implement `refuelVehicle()` because it sits in one big `DeliveryPartner` interface. |
| **D** | Dependency Inversion | Depend on **interfaces**, not on specific classes. | `OrderService` depends on `Notifier`, not on `SmsNotifier`. |

**The one sentence to say in the interview:** "I keep each class to one responsibility, and I put anything that has several variants behind an interface, so a new variant is a new class and no existing code changes."

## 2.2 What a design pattern is

A **design pattern** is a **well-known, named solution to a common design problem**. It works like a recipe name. When you say "Strategy", the interviewer instantly knows the shape of your code. Five patterns cover almost every Swiggy question.

## 2.3 Strategy pattern ★ (the most useful one)

**Problem it solves:** you have **several ways to do the same task**, and you want to pick one at runtime **without a giant if-else**.

**Real life:** Swiggy coupons. "FLAT100" (Rs 100 off), "SAVE20" (20% off up to Rs 120) and "BOGO" (buy one get one) all answer the same question: *how much discount does this cart get?* Each coupon is one **strategy**.

**Without Strategy (bad):**
```java
long discount(Cart cart, String code) {
    if (code.equals("FLAT100")) { ... }
    else if (code.equals("SAVE20")) { ... }
    else if (code.equals("BOGO")) { ... }     // every new offer = edit this method (breaks Open/Closed)
}
```

**With Strategy (good):**
```java
interface DiscountStrategy { long discountFor(Cart cart); }

record FlatDiscount(long offPaise, long minOrderPaise) implements DiscountStrategy {
    public long discountFor(Cart cart) { return cart.itemTotal() >= minOrderPaise ? offPaise : 0; }
}
record PercentDiscount(int percent, long capPaise) implements DiscountStrategy {
    public long discountFor(Cart cart) { return Math.min(cart.itemTotal() * percent / 100, capPaise); }
}
// The bill code just calls offer.discountFor(cart). It never knows which offer it is.
```

**Where it shows up at Swiggy:**
- discounts (Dec 2025, selected candidate)
- driver matching rules for VIP vs normal orders (Mar 2026)
- split types in Splitwise
- payment methods
- slot-finding in a parking lot

## 2.4 Singleton pattern (asked as a parking-lot follow-up)

**Problem it solves:** there must be **exactly one object** of a class in the whole program, and everyone must share it.

**Real life:** a restaurant has **one** order-number counter at the billing desk. If two counters existed, two customers could both get order #41.

```java
class IdGenerator {
    private static volatile IdGenerator instance;   // the one and only object
    private long counter = 0;

    private IdGenerator() {}                          // private: nobody else can call `new`

    static IdGenerator getInstance() {
        if (instance == null) {                       // 1st check: skip locking once created (fast)
            synchronized (IdGenerator.class) {        // only one thread at a time past this line
                if (instance == null) instance = new IdGenerator();   // 2nd check
            }
        }
        return instance;
    }
    synchronized long next() { return ++counter; }
}
```

**Follow-up questions and answers:**
- *Why the private constructor?* So nobody can write `new IdGenerator()` and make a second one.
- *Why check `null` twice?* This is called **double-checked locking**. Two threads could both see `null` at the same moment. Only one gets into the `synchronized` block, and the second check stops the other thread from creating a second object after it gets in.
- *Why `volatile`?* Without it, another thread might see a half-built object because of CPU and compiler reordering. `volatile` guarantees every thread sees the fully built object.
- *Simplest thread-safe Singleton?* An enum: `enum IdGenerator { INSTANCE; ... }`. Java guarantees exactly one instance.
- *Downside of Singleton?* It is global state. It is hard to replace in tests, and it hides dependencies. In real projects (Spring), you let the framework create one shared object (a "singleton bean") and inject it instead.

## 2.5 Factory pattern

**Problem it solves:** creating objects needs a decision ("which class do I make?"). Put that decision in **one place** instead of scattering `new` everywhere.

**Real life:** at a Swiggy kitchen, the waiter writes "1 pizza, thin crust". The kitchen decides which station makes it. The waiter doesn't need to know.

```java
enum Channel { SMS, PUSH }
class NotifierFactory {
    static Notifier create(Channel c) {
        return switch (c) {
            case SMS -> new SmsNotifier();
            case PUSH -> new PushNotifier();
        };
    }
}
Notifier n = NotifierFactory.create(Channel.PUSH);
```

**Use it when:** the type to create depends on input, such as vehicle type in a parking lot, or notification channel or split type chosen by the user. A `Map<Type, Strategy>` lookup, as used in the solutions below, is a lightweight factory.

## 2.6 Observer pattern

**Problem it solves:** when one thing changes, **many others need to know**, and the changing thing shouldn't have to know who they are.

**Real life:** when your order becomes "Out for delivery", your app updates, the restaurant dashboard updates, and you get an SMS. The order doesn't call each of them by name. They **subscribed**, and the order just announces the change.

```java
interface OrderListener { void onStatusChange(String orderId, String status); }

class TrackedOrder {
    private final String id;
    private final List<OrderListener> listeners = new ArrayList<>();
    TrackedOrder(String id) { this.id = id; }
    void subscribe(OrderListener l) { listeners.add(l); }
    void setStatus(String status) { for (OrderListener l : listeners) l.onStatusChange(id, status); }
}

TrackedOrder o = new TrackedOrder("ORD1");
o.subscribe((id, s) -> System.out.println("Customer app: " + id + " is " + s));
o.subscribe((id, s) -> System.out.println("Restaurant dashboard: " + id + " is " + s));
o.setStatus("OUT_FOR_DELIVERY");
```

(`(id, s) -> ...` is a **lambda**: a short way to write a one-method object.)

**Link to your resume:** Kafka is the Observer pattern across machines. A service publishes an event, and any number of other services subscribe to it.

## 2.7 State pattern (briefly)

**Problem it solves:** an object behaves differently depending on its **current stage**, and only some moves between stages are allowed.

**Real life:** you can cancel a Swiggy order before the restaurant starts cooking, but not after. "Cancel" means different things in different stages.

**Simple version (enough for most rounds):** an `enum` for the stages plus one method that checks which moves are allowed. This is used in the food-ordering solution:

```java
void moveTo(OrderStatus next) {
    boolean ok = switch (status) {
        case PLACED -> next == OrderStatus.ACCEPTED || next == OrderStatus.CANCELLED;
        case ACCEPTED -> next == OrderStatus.PREPARING || next == OrderStatus.CANCELLED;
        case PREPARING -> next == OrderStatus.OUT_FOR_DELIVERY;
        case OUT_FOR_DELIVERY -> next == OrderStatus.DELIVERED;
        case DELIVERED, CANCELLED -> false;
    };
    if (!ok) throw new IllegalStateException("Cannot go from " + status + " to " + next);
    status = next;
}
```

**Full State pattern:** each stage is its own class (`Placed`, `Preparing`, …) with its own `next()` and `cancel()`. `Preparing.cancel()` throws "food is already being cooked". Mention that you would switch to it when each stage has a lot of its own logic.

## 2.8 Pattern cheat table: "if you hear X, think Y"

| You hear… | Think… |
|---|---|
| "different types of discount / split / pricing / matching rule" | **Strategy** |
| "only one parking lot / one config / one ID counter" | **Singleton** |
| "create a vehicle/notification based on input type" | **Factory** |
| "notify users / update dashboards when something changes" | **Observer** |
| "order goes through stages; some actions not allowed in some stages" | **State** (or enum + transition check) |
| "add features later without changing old code" | Interfaces + Strategy (Open/Closed) |

---

# Part 3: The playbook for a 60–90 minute machine coding round

## 3.1 The time budget

| Phase | 90-min round | 60-min round | What you produce |
|---|---|---|---|
| Read and clarify | 10 min | 5–7 min | A written list: **must-have** vs **optional** features, plus your assumptions |
| Design | 10–15 min | 8–10 min | Entity list, relationships, interfaces, a quick sketch. **No detailed UML.** |
| Code the core | 45 min | 30 min | Models → services → `Main` demo that **runs** |
| Edge cases and optional features | 10–15 min | 5–10 min | Exceptions for bad input, one or two optional features |
| Demo and discussion | 5–10 min | 5 min | Run it, walk through it, answer "how would you add X" |

**Golden rule:** by the **halfway mark** you must have *something that runs*, even if it's ugly. Then improve it.

## 3.2 Step 1: requirements and clarifying questions

1. **Read the whole statement once.** Then write two lists in a comment at the top of your file: `MUST` and `OPTIONAL`.
2. **Ask 4–6 short questions, then stop.** Good questions change your design. Bad questions just burn time.

Generic questions that work for most problems:
- "Is this in-memory only, or do you want a database?" (Almost always in-memory.)
- "Is input from a `main` method / hardcoded demo, or do you want a command-line parser?"
- "Single-threaded is fine, or should I worry about concurrent users?"
- "How should invalid input be handled: throw an error, or print a message?"
- "Which optional features matter most to you?"

3. **Say your assumptions out loud** and write them down: "I'll assume one cart can only hold items from one restaurant, like the Swiggy app."

## 3.3 Step 2: find the entities (classes)

**Trick:** underline the **nouns** and **verbs** in the problem.
- Nouns become **classes**: Player, Board, Snake, Ladder, Dice, Game.
- Verbs become **methods**: roll, move, play, addExpense, applyCoupon.
- "Kinds of" something (types of splits, types of discount) become an **interface + strategies**.

Then draw the relationships. A quick text sketch is enough:

```
Game ──has──> Board ──has many──> Jump (snake or ladder)
  │
  ├──has──> Dice (interface) <── RandomDice, FixedDice
  └──has many──> Player
```

## 3.4 Step 3: package layout

In a real project each part is a folder (**package**). In a 60-minute round you can keep one file with `static` nested classes, but **mark the boundaries with comments** and say "in a real project these would be separate packages":

```
com.swiggy.splitwise
├── model/        User, Expense, Share, SplitType        (plain data holders)
├── strategy/     SplitStrategy, EqualSplit, ExactSplit, PercentSplit
├── repository/   UserRepository, ExpenseRepository       (in-memory storage)
├── service/      ExpenseService, BalanceService          (business rules)
├── exception/    UserNotFoundException, InvalidSplitException
└── Main.java     demo / driver
```

## 3.5 Step 4: in-memory repositories and services

- A **repository** is the class that **stores and finds** objects. In the interview it's a `HashMap`. Later it could be MySQL, without changing anything else.
  - Real life: the restaurant's storeroom register. You ask for "item #42" and get it, without knowing which shelf it's on.
- A **service** holds the **business rules**: validate, compute, update.
  - Real life: the restaurant manager, who decides whether an order can be accepted.

```java
class UserRepository {
    private final Map<String, User> users = new HashMap<>();
    void save(User u) { users.put(u.id(), u); }
    User get(String id) {
        User u = users.get(id);
        if (u == null) throw new IllegalArgumentException("Unknown user " + id);
        return u;
    }
}
```

**Why separate them:** each class has one job (Single Responsibility). The interviewer can also ask "what if we move to a database?" and you answer "only the repository changes".

## 3.6 Step 5: the driver / `Main` demo

Write a `main` method that runs a **realistic scenario** end to end: create users, do actions, print results, then **try one invalid action** and show the error. This proves the code works. It's also your test script during the discussion.

## 3.7 Handling invalid input

- Check inputs at the **start** of each public method, and fail fast with a clear message.
- Use `IllegalArgumentException` for bad input ("percent adds up to 90").
- Use `IllegalStateException` for "not allowed right now" ("cart is empty", "order already delivered").
- If you have time, create your own exceptions, such as `class InvalidSplitException extends RuntimeException`. They make the code read better.
- In `main`, wrap the bad call in `try/catch` and print the message, so the demo shows the check works.

## 3.8 Making optional features easy to add

- Put every "type of X" behind an **interface**. A new type is then a new class.
- Pass settings in through the **constructor** instead of hard-coding them, e.g. board size, number of dice, "play until all finish".
- Use a `Map<Type, Strategy>` to pick the behaviour, not if-else chains.
- Keep `main` thin. All logic lives in services.

## 3.9 What to say while coding

The interviewer is grading your thinking too. Speak in short sentences like these:
- "I'll start with the model classes, because everything else depends on them."
- "I'm storing money in paise as a long, to avoid floating-point rounding errors."
- "Split type has three variants, so I'll make a `SplitStrategy` interface. A new split type would be one new class."
- "I'm using a `Deque` for player turns: take from the front, put back at the end."
- "I'll skip the CLI parser for now and hardcode a demo in `main`, so we have something running. I can add input parsing at the end if there's time."
- "Here I'm validating that percentages add to 100 before changing any balances, so a bad request never leaves half-updated data."

## 3.10 Before you say "done"

- [ ] It compiles and `main` runs end to end.
- [ ] At least one invalid input is rejected with a clear message.
- [ ] No `public` fields, except in records.
- [ ] Each class has one job. There is no 200-line `Game` class doing everything.
- [ ] Names are readable: `itemTotal()`, not `calc2()`.
- [ ] You can say how to add each optional feature in one sentence.

## 3.11 Common mistakes

- 20 minutes of UML before any code. This is what got the Splitwise candidate rejected.
- Storing money in `double`.
- One giant class with everything in it.
- `if (type.equals("EQUAL")) … else if …` all over the code, where a strategy belongs.
- Forgetting the "exact landing" rule, empty carts, a user who doesn't exist, or splitting with zero people.
- Silence. Explain as you go.

---

# Part 4: Five complete worked solutions

**How to use this part:**
1. Read only the **requirements** and **clarifying questions**.
2. Close the doc and code it yourself for 45–60 minutes.
3. Then compare with the solution.

Every solution compiles with `javac` and runs from its `main` method.

## 4.1 Snake & Ladder (asked as a 120-minute machine coding round)

### Requirements (as the Swiggy report described them)

**Mandatory:** a working game on a 100-cell board, with snakes, ladders, a dice and players taking turns, written with proper OOP classes (Player, Game, Board).

**Optional (the interviewer listed these):**
1. The board can be any size N.
2. Any number of players M.
3. A ladder can start where a snake ends (you slide down and then climb).
4. A snake can start where a ladder ends.
5. Keep playing until all players finish, not just the first.

The candidate was tested with the interviewer's own test cases, and passed.

### Clarifying questions to ask
- "Must the player land exactly on the last cell? If the roll overshoots, do they stay where they are?" (Usual answer: yes, they stay.)
- "Does rolling a 6 give an extra turn?" (Usually no. Mention it as an extension.)
- "One dice or several?" (Make it configurable.)
- "Should I reject a board where snakes and ladders form an endless loop?" (Yes. Good catch to mention.)

### Entities and why

| Class | Why it exists |
|---|---|
| `Player` | Holds a name and a position. Nothing else. |
| `Jump` | A snake and a ladder are the **same thing**: "if you land on `start`, go to `end`". One class avoids duplicate code. `isSnake()` is simply `end < start`. |
| `Board` | Owns the size and all jumps. Validates them, and answers "where do I really end up if I land on cell X?" |
| `Dice` (interface) | `RandomDice` for real games, `FixedDice` for **tests**. Swapping them is the Strategy pattern, and it makes the game testable. |
| `Game` | Runs turns using a **queue** of players, and decides when the game is over. |

**Data structure choices:**
- `Map<Integer, Jump>` gives an O(1) "is there a snake or ladder on this cell?" check.
- `Deque<Player>` handles turns. Poll the front, play, then add to the back. A finished player simply isn't added back.

### Code

```java
import java.util.*;

public class SnakeLadder {

    // ---------- package: model ----------
    static class Player {
        private final String name;
        private int position = 0;          // 0 = not yet on the board
        Player(String name) { this.name = name; }
        String getName() { return name; }
        int getPosition() { return position; }
        void moveTo(int newPosition) { this.position = newPosition; }
    }

    // A snake or a ladder is just a "jump" from one cell to another.
    static class Jump {
        final int start, end;
        Jump(int start, int end) { this.start = start; this.end = end; }
        boolean isSnake() { return end < start; }
    }

    static class Board {
        private final int size;
        private final Map<Integer, Jump> jumps = new HashMap<>();

        Board(int size) {
            if (size < 2) throw new IllegalArgumentException("Board needs at least 2 cells");
            this.size = size;
        }
        int getSize() { return size; }

        void addSnake(int head, int tail) {
            if (head <= tail) throw new IllegalArgumentException("Snake head must be above its tail");
            addJump(new Jump(head, tail));
        }
        void addLadder(int bottom, int top) {
            if (bottom >= top) throw new IllegalArgumentException("Ladder bottom must be below its top");
            addJump(new Jump(bottom, top));
        }
        private void addJump(Jump j) {
            if (j.start <= 1 || j.start >= size || j.end < 1 || j.end > size)
                throw new IllegalArgumentException("Jump " + j.start + "->" + j.end + " is outside the board");
            if (jumps.containsKey(j.start))
                throw new IllegalArgumentException("Cell " + j.start + " already has a snake or ladder");
            jumps.put(j.start, j);
            if (formsLoop(j.start)) {          // e.g. ladder 5->20 and snake 20->5
                jumps.remove(j.start);
                throw new IllegalArgumentException("Jump " + j.start + "->" + j.end + " creates an endless loop");
            }
        }
        private boolean formsLoop(int from) {
            Set<Integer> seen = new HashSet<>();
            int cell = from;
            while (jumps.containsKey(cell)) {
                if (!seen.add(cell)) return true;
                cell = jumps.get(cell).end;
            }
            return false;
        }

        // Optional req 3 & 4: a ladder can start where a snake ends (and vice versa),
        // so keep following jumps until we land on a plain cell.
        int finalCell(int cell, String who) {
            while (jumps.containsKey(cell)) {
                Jump j = jumps.get(cell);
                System.out.println("   " + who + (j.isSnake() ? " bitten by snake " : " climbs ladder ")
                        + j.start + " -> " + j.end);
                cell = j.end;
            }
            return cell;
        }
    }

    // ---------- package: dice ----------
    interface Dice { int roll(); }

    static class RandomDice implements Dice {
        private final Random random;
        private final int count;                     // number of dice rolled together
        RandomDice(int count, long seed) { this.count = count; this.random = new Random(seed); }
        public int roll() {
            int sum = 0;
            for (int i = 0; i < count; i++) sum += random.nextInt(6) + 1;
            return sum;
        }
    }

    // Used in tests: returns the values we give it, in order.
    static class FixedDice implements Dice {
        private final Deque<Integer> values;
        FixedDice(Integer... v) { values = new ArrayDeque<>(Arrays.asList(v)); }
        public int roll() { return values.poll(); }
    }

    // ---------- package: game ----------
    static class Game {
        private final Board board;
        private final Dice dice;
        private final Deque<Player> turnQueue = new ArrayDeque<>();
        private final List<Player> finishOrder = new ArrayList<>();
        private final boolean playUntilAllFinish;

        Game(Board board, Dice dice, List<Player> players, boolean playUntilAllFinish) {
            if (players.size() < 2) throw new IllegalArgumentException("Need at least 2 players");
            this.board = board;
            this.dice = dice;
            this.turnQueue.addAll(players);
            this.playUntilAllFinish = playUntilAllFinish;
        }

        List<Player> play() {
            while (!isOver()) {
                Player p = turnQueue.poll();
                playTurn(p);
                if (p.getPosition() == board.getSize()) {
                    finishOrder.add(p);
                    System.out.println("** " + p.getName() + " finished at rank " + finishOrder.size());
                } else {
                    turnQueue.offer(p);       // back of the line
                }
            }
            return finishOrder;
        }

        private boolean isOver() {
            if (finishOrder.isEmpty()) return false;
            if (!playUntilAllFinish) return true;
            return turnQueue.size() <= 1;    // last player left has nobody to play against
        }

        private void playTurn(Player p) {
            int roll = dice.roll();
            int from = p.getPosition();
            int target = from + roll;
            if (target > board.getSize()) {   // must land exactly on the last cell
                System.out.println(p.getName() + " rolled " + roll + ", needs exact roll, stays at " + from);
                return;
            }
            int to = board.finalCell(target, p.getName());
            p.moveTo(to);
            System.out.println(p.getName() + " rolled " + roll + " and moved from " + from + " to " + to);
        }
    }

    // ---------- driver ----------
    public static void main(String[] args) {
        // Demo 1: 10x10 board, 3 players, play until everyone finishes
        Board board = new Board(100);
        board.addSnake(99, 54); board.addSnake(70, 55); board.addSnake(52, 42); board.addSnake(25, 2);
        board.addLadder(6, 25);   // ladder top == snake head at 25 -> chain
        board.addLadder(11, 40); board.addLadder(60, 85); board.addLadder(46, 90);
        board.addLadder(42, 77);  // ladder at a snake's tail (52 -> 42 -> 77)

        List<Player> players = List.of(new Player("Asha"), new Player("Ravi"), new Player("Meera"));
        Game game = new Game(board, new RandomDice(1, 7), players, true);
        List<Player> ranks = game.play();
        System.out.print("Final ranking: ");
        for (Player p : ranks) System.out.print(p.getName() + " ");
        System.out.println();

        // Demo 2: a deterministic test with FixedDice
        Board small = new Board(10);
        small.addLadder(2, 8);
        small.addSnake(9, 3);
        Player a = new Player("A"), b = new Player("B");
        Game test = new Game(small, new FixedDice(2, 1, 2, 5), List.of(a, b), false);
        List<Player> winner = test.play();
        check(winner.get(0) == a, "A should win: 0->2 ladder 8, then 8->10");

        // Demo 3: invalid input is rejected
        try {
            small.addLadder(3, 9);   // 3 -> 9 (snake) -> 3 ... endless loop
            check(false, "loop must be rejected");
        } catch (IllegalArgumentException e) {
            System.out.println("Rejected as expected: " + e.getMessage());
        }
        System.out.println("All checks passed");
    }

    static void check(boolean ok, String msg) { if (!ok) throw new AssertionError(msg); }
}
```

### How the optional requirements are handled
1. **Any board size:** `new Board(n)`; nothing assumes 100.
2. **Any number of players:** it is just a list passed to `Game`.
3. **Ladder at a snake's tail** and **4. snake at a ladder's top:** `finalCell` keeps following jumps in a `while` loop. In the demo, 52 → 42 (snake) → 77 (ladder) shows this. `formsLoop` rejects a board where jumps loop forever, such as ladder 3→9 with snake 9→3.
5. **Play until all finish:** the `playUntilAllFinish` flag. Finished players leave the queue, and the game stops when one player is left.

### How to extend it
- **Several dice:** already supported, via `new RandomDice(2, seed)`.
- **Extra turn on a 6:** in `playTurn`, if `roll == 6`, call `playTurn` again. Better: give `Game` a `TurnRule` interface.
- **Crocodile / mine cells** (a common add-on): add a `Cell` interface with `int landOn(int cell)`. A snake, ladder, crocodile ("go back 5") and mine ("skip 2 turns") each implement it.
- **Save and resume a game:** store the player positions and turn order (the repository idea).

### Likely follow-ups
- *"Why is the dice an interface?"* So I can inject a fixed dice in tests and get a predictable result. That's how I tested "A wins in two moves".
- *"Why one `Jump` class instead of `Snake` and `Ladder` classes?"* Both behave the same: land on start, move to end. Two classes would duplicate code. If they ever behave differently, I'd split them behind a common interface.
- *"Time per turn?"* O(k), where k is the length of a jump chain (usually 0 or 1). O(1) in practice.
- *"How would you make it multiplayer online?"* Game state lives on a server. Each player's roll is a request, and the server checks it's that player's turn. That's an HLD question, and I'd keep the same `Game` class inside.

---

## 4.2 Splitwise ★ (the most-asked Swiggy machine coding problem)

### Requirements
**Mandatory:**
- Users can add an expense: who paid, how much, and who shares it.
- Three split types:
  - **EQUAL:** everyone pays the same.
  - **EXACT:** you give each person's amount, and the amounts must add up to the total.
  - **PERCENT:** you give each person's percentage, and the percentages must add up to 100.
- Show all balances ("Ravi owes Asha Rs 120").
- Show one user's balances.

**Optional:**
- **Simplify debts:** pay the same total with fewer transfers.
- Expense history.
- Groups.

### Clarifying questions
- "Amounts in rupees with paise? Should I round to two decimals?" (Store paise. Give the leftover paisa to someone.)
- "In an EQUAL split, is the payer always one of the people sharing?" (Not necessarily. Handle both.)
- "Do you want groups, or just users?"
- "Should simplify-debts change the stored balances, or just show suggested payments?" (Usually just show them.)

### Entities and why

| Class | Why |
|---|---|
| `User` (record) | id + name. Pure data. |
| `Share` (record) | "This user's part of this expense = X paise." |
| `Expense` (record) | Who paid, the total and the list of shares. Stored for history. |
| `SplitType` (enum) + `SplitStrategy` (interface) | The three split types differ only in **how shares are computed and validated**. That is a textbook Strategy. A new "by shares" split (2:1:1) is one new class. |
| `ExpenseService` | Validates input, picks the strategy, records the expense and updates balances. |

**The key data-structure decision: how to store balances.**
- **Option A:** `Map<String, Map<String, Long>> balances`. `balances[A][B] = 500` means "B owes A Rs 5". It is always kept **mirrored**: `balances[B][A] = -500`. Showing "who owes whom" is a direct lookup. This is what the code uses.
- **Option B:** store one **net** number per person (+ means others owe you). This is simpler, but you lose the pairwise view. It's exactly what simplify-debts needs, though, so the code computes it on demand.

**The rounding problem (say this, interviewers love it):** Rs 100 split 3 ways is 33.33 + 33.33 + 33.33 = 99.99. One paisa vanishes. The fix is to work in paise, give everyone `total / n`, and hand the leftover `total % n` paise one each to the first people.

### Code

```java
import java.util.*;

public class Splitwise {

    // ---------- package: model ----------
    // Money is stored in paise (long), never in double. 0.1 + 0.2 != 0.3 in double.
    record User(String id, String name) {}

    enum SplitType { EQUAL, EXACT, PERCENT }

    // One person's share of an expense.
    record Share(String userId, long amountPaise) {}

    record Expense(String id, String paidBy, long totalPaise, String description, List<Share> shares) {}

    // ---------- package: split strategies ----------
    // Each split type knows how to turn the input into shares, and how to validate it.
    interface SplitStrategy {
        List<Share> split(long totalPaise, List<String> userIds, List<Double> values);
    }

    static class EqualSplit implements SplitStrategy {
        public List<Share> split(long total, List<String> users, List<Double> ignored) {
            long each = total / users.size();
            long leftover = total - each * users.size();   // 100 / 3 -> 33, 33, 34
            List<Share> shares = new ArrayList<>();
            for (int i = 0; i < users.size(); i++) {
                long amount = each + (i < leftover ? 1 : 0);
                shares.add(new Share(users.get(i), amount));
            }
            return shares;
        }
    }

    static class ExactSplit implements SplitStrategy {
        public List<Share> split(long total, List<String> users, List<Double> amountsRupees) {
            if (amountsRupees.size() != users.size())
                throw new IllegalArgumentException("Need one amount per user");
            List<Share> shares = new ArrayList<>();
            long sum = 0;
            for (int i = 0; i < users.size(); i++) {
                long p = Math.round(amountsRupees.get(i) * 100);
                sum += p;
                shares.add(new Share(users.get(i), p));
            }
            if (sum != total) throw new IllegalArgumentException("Exact amounts add up to " + sum + ", not " + total);
            return shares;
        }
    }

    static class PercentSplit implements SplitStrategy {
        public List<Share> split(long total, List<String> users, List<Double> percents) {
            if (percents.size() != users.size())
                throw new IllegalArgumentException("Need one percent per user");
            double pctSum = percents.stream().mapToDouble(Double::doubleValue).sum();
            if (Math.abs(pctSum - 100.0) > 1e-9) throw new IllegalArgumentException("Percents add up to " + pctSum);
            List<Share> shares = new ArrayList<>();
            long given = 0;
            for (int i = 0; i < users.size(); i++) {
                long p = (i == users.size() - 1) ? total - given        // last person absorbs rounding
                                                 : Math.round(total * percents.get(i) / 100.0);
                given += p;
                shares.add(new Share(users.get(i), p));
            }
            return shares;
        }
    }

    // ---------- package: service ----------
    static class ExpenseService {
        private final Map<String, User> users = new LinkedHashMap<>();
        private final List<Expense> expenses = new ArrayList<>();
        // balances.get(A).get(B) = how much B owes A (positive) in paise
        private final Map<String, Map<String, Long>> balances = new HashMap<>();
        private final Map<SplitType, SplitStrategy> strategies = Map.of(
                SplitType.EQUAL, new EqualSplit(),
                SplitType.EXACT, new ExactSplit(),
                SplitType.PERCENT, new PercentSplit());
        private int nextId = 1;

        void addUser(User u) {
            users.put(u.id(), u);
            balances.put(u.id(), new HashMap<>());
        }

        Expense addExpense(String paidBy, double amountRupees, SplitType type,
                           List<String> participants, List<Double> values, String description) {
            requireUser(paidBy);
            participants.forEach(this::requireUser);
            if (amountRupees <= 0) throw new IllegalArgumentException("Amount must be positive");
            long total = Math.round(amountRupees * 100);

            List<Share> shares = strategies.get(type).split(total, participants, values);
            Expense e = new Expense("E" + nextId++, paidBy, total, description, shares);
            expenses.add(e);

            for (Share s : shares) {
                if (s.userId().equals(paidBy)) continue;   // you don't owe yourself
                adjust(paidBy, s.userId(), s.amountPaise());
            }
            return e;
        }

        // borrower owes lender `amount` more. Keep both directions in sync.
        private void adjust(String lender, String borrower, long amount) {
            balances.get(lender).merge(borrower, amount, Long::sum);
            balances.get(borrower).merge(lender, -amount, Long::sum);
        }

        private void requireUser(String id) {
            if (!users.containsKey(id)) throw new IllegalArgumentException("Unknown user " + id);
        }

        void showAll() {
            boolean any = false;
            for (String lender : balances.keySet()) {
                for (Map.Entry<String, Long> e : balances.get(lender).entrySet()) {
                    if (e.getValue() > 0) {
                        System.out.println("  " + name(e.getKey()) + " owes " + name(lender) + ": " + rupees(e.getValue()));
                        any = true;
                    }
                }
            }
            if (!any) System.out.println("  No balances");
        }

        // Minimum-ish transfers: compute each person's NET amount, then repeatedly
        // let the biggest debtor pay the biggest creditor.
        List<String> simplify() {
            Map<String, Long> net = new HashMap<>();
            for (String a : balances.keySet())
                for (Map.Entry<String, Long> e : balances.get(a).entrySet())
                    net.merge(a, e.getValue(), Long::sum);          // positive = others owe a

            PriorityQueue<long[]> creditors = new PriorityQueue<>((x, y) -> Long.compare(y[1], x[1]));
            PriorityQueue<long[]> debtors = new PriorityQueue<>((x, y) -> Long.compare(y[1], x[1]));
            List<String> ids = new ArrayList<>(users.keySet());
            for (int i = 0; i < ids.size(); i++) {
                long v = net.getOrDefault(ids.get(i), 0L);
                if (v > 0) creditors.add(new long[]{i, v});
                if (v < 0) debtors.add(new long[]{i, -v});
            }
            List<String> transfers = new ArrayList<>();
            while (!creditors.isEmpty()) {
                long[] c = creditors.poll(), d = debtors.poll();
                long amt = Math.min(c[1], d[1]);
                transfers.add(name(ids.get((int) d[0])) + " pays " + name(ids.get((int) c[0])) + " " + rupees(amt));
                if (c[1] > amt) creditors.add(new long[]{c[0], c[1] - amt});
                if (d[1] > amt) debtors.add(new long[]{d[0], d[1] - amt});
            }
            return transfers;
        }

        private String name(String id) { return users.get(id).name(); }
        private static String rupees(long p) { return String.format("Rs %d.%02d", p / 100, p % 100); }
    }

    // ---------- driver ----------
    public static void main(String[] args) {
        ExpenseService s = new ExpenseService();
        s.addUser(new User("u1", "Asha"));
        s.addUser(new User("u2", "Ravi"));
        s.addUser(new User("u3", "Meera"));
        s.addUser(new User("u4", "Kabir"));

        s.addExpense("u1", 1000, SplitType.EQUAL, List.of("u1", "u2", "u3", "u4"), null, "Dinner");
        s.addExpense("u2", 1250, SplitType.EXACT, List.of("u1", "u3"), List.of(370.0, 880.0), "Movie");
        s.addExpense("u4", 1200, SplitType.PERCENT, List.of("u1", "u2", "u3", "u4"),
                List.of(40.0, 20.0, 20.0, 20.0), "Cab");
        s.addExpense("u3", 100, SplitType.EQUAL, List.of("u1", "u2", "u3"), null, "Chai");

        System.out.println("Balances:");
        s.showAll();
        System.out.println("Simplified:");
        s.simplify().forEach(t -> System.out.println("  " + t));

        try {
            s.addExpense("u1", 500, SplitType.PERCENT, List.of("u1", "u2"), List.of(50.0, 40.0), "Bad");
        } catch (IllegalArgumentException e) {
            System.out.println("Rejected as expected: " + e.getMessage());
        }
    }
}
```

### How simplify-debts works (in plain words)
1. For each person, compute the **net** amount: what others owe them minus what they owe others. In the demo, Meera's net is −Rs 1303.33 (she owes overall), and Ravi's is +Rs 726.67.
2. Put the people who are owed in one max-heap, and the people who owe in another.
3. Repeatedly let the **biggest debtor pay the biggest creditor** as much as possible. One of them hits zero, and the other goes back in the heap with what's left.
4. Six pairwise debts in the demo became three transfers.

**Real life:** after a trip, instead of everyone paying everyone, the friend who overspent the least pays the friend who overspent the most, and so on.

(Finding the **true minimum** number of transfers is an NP-hard problem, meaning no known fast method exists for large inputs. The greedy method uses at most n−1 transfers and is what everyone uses in interviews. Say this if asked.)

### How to extend it
- **Groups:** add `Group(id, members)` and key the balances by group.
- **A new split type**, e.g. by shares (2:1:1): write `ShareSplit implements SplitStrategy` and add it to the map. Nothing else changes.
- **Settle up:** `settle(from, to, amount)` is just `adjust(to, from, -amount)`.
- **Persistence:** swap the maps for repositories backed by MySQL.
  - Tables: `users`, `expenses(id, paid_by, total)`, `expense_shares(expense_id, user_id, amount)`, `balances(user_a, user_b, amount)`.

### Likely follow-ups
- *"Why not `double` for money?"* Floating-point can't represent 0.1 exactly, so errors pile up. Paise in a `long` is exact.
- *"Two people add expenses at the same time: what breaks?"* Both read and update the same balance maps. That's a race condition, and one update can be lost. In memory, make `addExpense` `synchronized` or use `ConcurrentHashMap.merge`. In a DB, update both balance rows in **one transaction**.
- *"Complexity of simplify?"* O(n log n) for n users, because of the heaps.
- *"Where's the validation?"* Inside each strategy, which knows its own rules: exact amounts must add to the total, percents to 100. It runs **before** any balance changes.

---

## 4.3 Food ordering cart with discounts (asked Dec 2025, solved with Strategy, selected)

### Requirements
**Mandatory:**
- Browse restaurants and menus.
- Add and remove dishes in a cart.
- Calculate the bill.
- Apply offers/discounts.
- Pay.

**Optional:**
- "Best offer" auto-pick.
- Order status tracking.
- Multiple payment methods.

### Clarifying questions
- "Can one cart hold items from two restaurants?" (No, like the real Swiggy app. Adding from another restaurant asks you to clear the cart.)
- "Can two coupons be combined?" (Usually only one.)
- "What are the delivery fee and tax rules?" (Assume a flat Rs 30 fee, free above Rs 399, and 5% GST on the discounted item total.)
- "Is payment real, or should I mock it?" (Mock it.)

### Entities and why

| Class | Why |
|---|---|
| `MenuItem` (record) | id, name, price, veg. Pure data. |
| `Restaurant` | Owns its menu and its open/closed flag. `getItem` throws if the dish isn't on its menu. |
| `Cart` | Holds `MenuItem → quantity` for **one** restaurant. It only knows its items and their total. It does **not** know about tax or offers (Single Responsibility). |
| `DiscountStrategy` + 3 offers | Each offer computes its own discount. A new offer is a new class (Open/Closed). |
| `OfferService` | Looks up offers by coupon code and finds the best one. |
| `BillCalculator` + `Bill` | All money rules live in one place: discount → delivery → tax → total. |
| `PaymentMethod` | UPI / COD. Strategy again. |
| `Order` + `OrderStatus` | A snapshot of items and the bill at checkout, plus allowed status moves (the State idea). |
| `OrderService` | Checkout: validate → bill → pay → create order → clear cart. |

### Code

```java
import java.util.*;

public class FoodOrdering {

    // ---------- package: model ----------
    record MenuItem(String id, String name, long pricePaise, boolean veg) {}

    static class Restaurant {
        final String id, name;
        private final Map<String, MenuItem> menu = new LinkedHashMap<>();
        private boolean open = true;
        Restaurant(String id, String name) { this.id = id; this.name = name; }
        void addItem(MenuItem item) { menu.put(item.id(), item); }
        MenuItem getItem(String itemId) {
            MenuItem m = menu.get(itemId);
            if (m == null) throw new IllegalArgumentException("Item " + itemId + " not on " + name + "'s menu");
            return m;
        }
        Collection<MenuItem> getMenu() { return menu.values(); }
        boolean isOpen() { return open; }
        void setOpen(boolean open) { this.open = open; }
    }

    // A cart belongs to one user and holds items from ONE restaurant (like the Swiggy app).
    static class Cart {
        final String userId;
        private Restaurant restaurant;
        private final Map<MenuItem, Integer> items = new LinkedHashMap<>();
        Cart(String userId) { this.userId = userId; }

        void add(Restaurant r, String itemId, int qty) {
            if (qty <= 0) throw new IllegalArgumentException("Quantity must be positive");
            if (!r.isOpen()) throw new IllegalStateException(r.name + " is closed");
            if (restaurant != null && restaurant != r)
                throw new IllegalStateException("Cart has items from " + restaurant.name + ". Clear it first.");
            restaurant = r;
            items.merge(r.getItem(itemId), qty, Integer::sum);
        }
        void remove(String itemId) {
            items.keySet().removeIf(m -> m.id().equals(itemId));
            if (items.isEmpty()) restaurant = null;
        }
        long itemTotal() {
            long sum = 0;
            for (Map.Entry<MenuItem, Integer> e : items.entrySet()) sum += e.getKey().pricePaise() * e.getValue();
            return sum;
        }
        Map<MenuItem, Integer> getItems() { return Collections.unmodifiableMap(items); }
        Restaurant getRestaurant() { return restaurant; }
        boolean isEmpty() { return items.isEmpty(); }
        void clear() { items.clear(); restaurant = null; }
    }

    // ---------- package: offers (Strategy pattern) ----------
    interface DiscountStrategy {
        String code();
        long discountFor(Cart cart);   // 0 if not applicable
    }

    // "FLAT100": Rs 100 off on orders above Rs 499
    record FlatDiscount(String code, long offPaise, long minOrderPaise) implements DiscountStrategy {
        public long discountFor(Cart cart) {
            return cart.itemTotal() >= minOrderPaise ? Math.min(offPaise, cart.itemTotal()) : 0;
        }
    }

    // "SAVE20": 20% off up to Rs 120
    record PercentDiscount(String code, int percent, long capPaise) implements DiscountStrategy {
        public long discountFor(Cart cart) {
            return Math.min(cart.itemTotal() * percent / 100, capPaise);
        }
    }

    // "BOGO-<item>": buy one get one free on a specific item
    record BuyOneGetOne(String code, String itemId) implements DiscountStrategy {
        public long discountFor(Cart cart) {
            for (Map.Entry<MenuItem, Integer> e : cart.getItems().entrySet())
                if (e.getKey().id().equals(itemId)) return (e.getValue() / 2) * e.getKey().pricePaise();
            return 0;
        }
    }

    static class OfferService {
        private final Map<String, DiscountStrategy> offers = new HashMap<>();
        void register(DiscountStrategy s) { offers.put(s.code(), s); }
        DiscountStrategy byCode(String code) {
            DiscountStrategy s = offers.get(code);
            if (s == null) throw new IllegalArgumentException("Invalid coupon " + code);
            return s;
        }
        // "Apply best offer" button: try all, pick the biggest saving.
        Optional<DiscountStrategy> best(Cart cart) {
            return offers.values().stream()
                    .filter(s -> s.discountFor(cart) > 0)
                    .max(Comparator.comparingLong(s -> s.discountFor(cart)));
        }
    }

    // ---------- package: billing ----------
    record Bill(long itemTotal, long discount, long deliveryFee, long tax, long payable) {
        void print() {
            System.out.println("  Item total : " + rs(itemTotal));
            System.out.println("  Discount   : -" + rs(discount));
            System.out.println("  Delivery   : " + rs(deliveryFee));
            System.out.println("  GST (5%)   : " + rs(tax));
            System.out.println("  To pay     : " + rs(payable));
        }
    }

    static class BillCalculator {
        static final long FREE_DELIVERY_ABOVE = 39900, DELIVERY_FEE = 3000;
        Bill calculate(Cart cart, DiscountStrategy offer) {
            long items = cart.itemTotal();
            long discount = offer == null ? 0 : offer.discountFor(cart);
            long afterDiscount = items - discount;
            long delivery = afterDiscount >= FREE_DELIVERY_ABOVE ? 0 : DELIVERY_FEE;
            long tax = afterDiscount * 5 / 100;
            return new Bill(items, discount, delivery, tax, afterDiscount + delivery + tax);
        }
    }

    // ---------- package: payment (Strategy again) ----------
    interface PaymentMethod { boolean pay(long amountPaise); }
    static class UpiPayment implements PaymentMethod {
        final String upiId;
        UpiPayment(String upiId) { this.upiId = upiId; }
        public boolean pay(long amount) { System.out.println("  Paid " + rs(amount) + " via UPI " + upiId); return true; }
    }
    static class CashOnDelivery implements PaymentMethod {
        public boolean pay(long amount) { System.out.println("  " + rs(amount) + " to be collected in cash"); return true; }
    }

    // ---------- package: order ----------
    enum OrderStatus { PLACED, ACCEPTED, PREPARING, OUT_FOR_DELIVERY, DELIVERED, CANCELLED }

    static class Order {
        final String id;
        final Map<MenuItem, Integer> items;
        final Bill bill;
        private OrderStatus status = OrderStatus.PLACED;
        Order(String id, Map<MenuItem, Integer> items, Bill bill) {
            this.id = id; this.items = new LinkedHashMap<>(items); this.bill = bill;
        }
        OrderStatus getStatus() { return status; }
        void moveTo(OrderStatus next) {
            boolean ok = switch (status) {
                case PLACED -> next == OrderStatus.ACCEPTED || next == OrderStatus.CANCELLED;
                case ACCEPTED -> next == OrderStatus.PREPARING || next == OrderStatus.CANCELLED;
                case PREPARING -> next == OrderStatus.OUT_FOR_DELIVERY;
                case OUT_FOR_DELIVERY -> next == OrderStatus.DELIVERED;
                case DELIVERED, CANCELLED -> false;
            };
            if (!ok) throw new IllegalStateException("Cannot go from " + status + " to " + next);
            status = next;
        }
    }

    static class OrderService {
        private final BillCalculator calculator = new BillCalculator();
        private final Map<String, Order> orders = new HashMap<>();
        private int next = 1;

        Order checkout(Cart cart, DiscountStrategy offer, PaymentMethod payment) {
            if (cart.isEmpty()) throw new IllegalStateException("Cart is empty");
            Bill bill = calculator.calculate(cart, offer);
            if (!payment.pay(bill.payable())) throw new IllegalStateException("Payment failed");
            Order o = new Order("ORD" + next++, cart.getItems(), bill);
            orders.put(o.id, o);
            cart.clear();
            return o;
        }
    }

    static String rs(long p) { return String.format("Rs %d.%02d", p / 100, p % 100); }

    // ---------- driver ----------
    public static void main(String[] args) {
        Restaurant meghana = new Restaurant("R1", "Meghana Foods");
        meghana.addItem(new MenuItem("M1", "Chicken Biryani", 32000, false));
        meghana.addItem(new MenuItem("M2", "Paneer Biryani", 28000, true));
        meghana.addItem(new MenuItem("M3", "Gulab Jamun", 9000, true));
        Restaurant truffles = new Restaurant("R2", "Truffles");
        truffles.addItem(new MenuItem("T1", "Burger", 22000, false));

        OfferService offers = new OfferService();
        offers.register(new FlatDiscount("FLAT100", 10000, 49900));
        offers.register(new PercentDiscount("SAVE20", 20, 12000));
        offers.register(new BuyOneGetOne("BOGO-M3", "M3"));

        Cart cart = new Cart("user-1");
        cart.add(meghana, "M1", 2);
        cart.add(meghana, "M3", 2);

        try { cart.add(truffles, "T1", 1); }
        catch (IllegalStateException e) { System.out.println("Rejected: " + e.getMessage()); }

        BillCalculator calc = new BillCalculator();
        System.out.println("With SAVE20:");
        calc.calculate(cart, offers.byCode("SAVE20")).print();

        DiscountStrategy best = offers.best(cart).orElse(null);
        System.out.println("Best offer is " + best.code() + ":");
        calc.calculate(cart, best).print();

        Order order = new OrderService().checkout(cart, best, new UpiPayment("asha@okbank"));
        order.moveTo(OrderStatus.ACCEPTED);
        order.moveTo(OrderStatus.PREPARING);
        System.out.println(order.id + " is " + order.getStatus() + ", cart empty now? " + cart.isEmpty());
        try { order.moveTo(OrderStatus.CANCELLED); }
        catch (IllegalStateException e) { System.out.println("Rejected: " + e.getMessage()); }
    }
}
```

**Walking through the demo:**
- Cart = 2 × biryani (Rs 640) + 2 × gulab jamun (Rs 180) = **Rs 820**.
- SAVE20 is 20% = Rs 164, capped at **Rs 120**. FLAT100 = Rs 100. BOGO on jamun = Rs 90. So `best()` picks SAVE20.
- 820 − 120 = 700. That's above Rs 399, so delivery is free. GST is 5% of 700 = 35. **To pay: Rs 735.**

### How to extend it
- **New offer type** (e.g. "Rs 50 off on your first order"): write a new `DiscountStrategy` that also takes the user's order count.
- **Stacking offers:** make a `CompositeDiscount` that holds a list of strategies and adds them up. (This is the **Composite** pattern.)
- **Offer validity** (expiry date, specific restaurants, max uses per user): add an `isApplicable(cart, user, now)` method to the interface.
- **Notifications on status change:** make `Order` notify subscribers (Observer).
- **Inventory / item out of stock:** add `available` to `MenuItem`, and check it in `Cart.add`.

### Likely follow-ups
- *"Why does the bill take a strategy instead of the cart holding the coupon?"* So the cart only knows about items. Offers can change without touching the cart, and I can compute "what if I used coupon X" without changing any state.
- *"Why copy items into the `Order`?"* The order must remember **what you paid for**, even if the menu price changes tomorrow.
- *"Two users order the last biryani at once?"* That needs stock with an atomic decrement: `synchronized`, or a DB `UPDATE … SET stock = stock - 1 WHERE stock > 0`.
- *"Payment fails?"* Don't create the order, and don't clear the cart. In real systems you create the order as `PAYMENT_PENDING` and use an **idempotency key**, so a retry doesn't charge twice.

---

## 4.4 Driver–order assignment, VIP vs normal (asked Mar 2026, 1 hour)

### Requirements (as reported)
"Assign drivers to orders. Orders can be VIP or non-VIP. Drivers have ratings. There are different matching conditions." The interviewer expected you to **design, choose data structures, code, and do a dry run**.

The exact rules weren't published, so state your own clearly. Clear assumptions are part of the test:
- **VIP order:** only drivers rated **4.5+** within **5 km**. Nearest wins, and ties go to the higher rating.
- **Normal order:** drivers rated **3.0+** within **8 km**. Nearest wins.
- If no driver matches, the order **waits**. When a driver becomes free, waiting orders are retried with **VIP first, then oldest first**.

### Clarifying questions
- "How do we measure distance: straight line, or road distance from a maps API?" (Straight line for now, behind a method that's easy to change.)
- "Can a driver carry two orders?" (Assume one at a time.)
- "If a VIP order can't find a top-rated driver, should it fall back to any driver after some time?" (Good extension.)
- "Are orders placed concurrently?" (Say you'll make assignment thread-safe.)

### Entities and why

| Class | Why |
|---|---|
| `Location` (record) | x, y and `distanceTo`. Swap it for a real distance service later. |
| `Driver` | id, rating, location, available flag. |
| `Order` | id, VIP or NORMAL, pickup point, creation time, and the assigned driver. |
| `MatchingStrategy` + `VipMatching` / `NormalMatching` | The matching *rule* differs by order type. Strategy again. A "Gold member" rule is a new class. |
| `AssignmentService` | Holds drivers, picks the strategy by order type, and keeps the **waiting queue**. |

**Data structure choices (they asked about these):**
- `Map<String, Driver>` gives O(1) lookup by driver id.
- `PriorityQueue<Order>` holds the waiting orders. The comparator puts VIP before normal, then sorts by creation time. Polling always gives the most urgent order.
- **Finding the nearest driver:** a linear scan over free drivers, O(D). Say: "For a city with thousands of drivers, I'd index drivers by area using a **geohash** (the map split into small square cells) and only check drivers in nearby cells."

### Code

```java
import java.util.*;

public class DriverAssignment {

    // ---------- package: model ----------
    record Location(double x, double y) {
        double distanceTo(Location o) { return Math.hypot(x - o.x, y - o.y); }   // km, straight line
    }

    static class Driver {
        final String id, name;
        final double rating;               // 1.0 to 5.0
        Location location;
        boolean available = true;
        Driver(String id, String name, double rating, Location loc) {
            this.id = id; this.name = name; this.rating = rating; this.location = loc;
        }
    }

    enum OrderType { VIP, NORMAL }

    static class Order {
        final String id;
        final OrderType type;
        final Location pickup;
        final long createdAt;              // used to keep waiting orders first-come-first-served
        Driver driver;
        Order(String id, OrderType type, Location pickup, long createdAt) {
            this.id = id; this.type = type; this.pickup = pickup; this.createdAt = createdAt;
        }
    }

    // ---------- package: matching (Strategy pattern) ----------
    interface MatchingStrategy {
        Optional<Driver> pick(Order order, Collection<Driver> availableDrivers);
    }

    // VIP: only drivers rated 4.5+, within 5 km; nearest wins, ties broken by higher rating.
    static class VipMatching implements MatchingStrategy {
        public Optional<Driver> pick(Order o, Collection<Driver> drivers) {
            return drivers.stream()
                    .filter(d -> d.rating >= 4.5)
                    .filter(d -> d.location.distanceTo(o.pickup) <= 5.0)
                    .min(Comparator.<Driver>comparingDouble(d -> d.location.distanceTo(o.pickup))
                            .thenComparing(d -> -d.rating));
        }
    }

    // Normal: any driver rated 3.0+ within 8 km; nearest wins.
    static class NormalMatching implements MatchingStrategy {
        public Optional<Driver> pick(Order o, Collection<Driver> drivers) {
            return drivers.stream()
                    .filter(d -> d.rating >= 3.0)
                    .filter(d -> d.location.distanceTo(o.pickup) <= 8.0)
                    .min(Comparator.comparingDouble(d -> d.location.distanceTo(o.pickup)));
        }
    }

    // ---------- package: service ----------
    static class AssignmentService {
        private final Map<String, Driver> drivers = new HashMap<>();
        private final Map<OrderType, MatchingStrategy> strategies = new EnumMap<>(OrderType.class);
        // Orders nobody could take yet. VIP first, then oldest first.
        private final PriorityQueue<Order> waiting = new PriorityQueue<>(
                Comparator.<Order>comparingInt(o -> o.type == OrderType.VIP ? 0 : 1)
                        .thenComparingLong(o -> o.createdAt));

        AssignmentService() {
            strategies.put(OrderType.VIP, new VipMatching());
            strategies.put(OrderType.NORMAL, new NormalMatching());
        }

        void addDriver(Driver d) { drivers.put(d.id, d); }

        // synchronized: two orders must never grab the same driver at the same moment.
        synchronized void placeOrder(Order o) {
            if (!tryAssign(o)) {
                waiting.add(o);
                System.out.println(o.id + " (" + o.type + ") waiting: no matching driver");
            }
        }

        synchronized void completeDelivery(String driverId, Location dropPoint) {
            Driver d = drivers.get(driverId);
            if (d == null) throw new IllegalArgumentException("Unknown driver " + driverId);
            if (d.available) throw new IllegalStateException(d.name + " has no active order");
            d.available = true;
            d.location = dropPoint;
            System.out.println(d.name + " is free again");
            retryWaiting();
        }

        private boolean tryAssign(Order o) {
            List<Driver> free = drivers.values().stream().filter(d -> d.available).toList();
            Optional<Driver> chosen = strategies.get(o.type).pick(o, free);
            if (chosen.isEmpty()) return false;
            Driver d = chosen.get();
            d.available = false;
            o.driver = d;
            System.out.printf("%s (%s) -> %s (rating %.1f, %.1f km away)%n",
                    o.id, o.type, d.name, d.rating, d.location.distanceTo(o.pickup));
            return true;
        }

        // Walk the waiting queue in priority order; keep the ones still unmatched.
        private void retryWaiting() {
            List<Order> stillWaiting = new ArrayList<>();
            while (!waiting.isEmpty()) {
                Order o = waiting.poll();
                if (!tryAssign(o)) stillWaiting.add(o);
            }
            waiting.addAll(stillWaiting);
        }
    }

    // ---------- driver ----------
    public static void main(String[] args) {
        AssignmentService svc = new AssignmentService();
        svc.addDriver(new Driver("D1", "Ramesh", 4.8, new Location(0, 0)));
        svc.addDriver(new Driver("D2", "Suresh", 4.2, new Location(1, 1)));
        svc.addDriver(new Driver("D3", "Imran", 3.5, new Location(2, 2)));

        long t = 0;
        svc.placeOrder(new Order("O1", OrderType.VIP, new Location(1, 0), t++));     // needs 4.5+: Ramesh
        svc.placeOrder(new Order("O2", OrderType.NORMAL, new Location(1, 1), t++));  // Suresh is closest
        svc.placeOrder(new Order("O3", OrderType.VIP, new Location(2, 2), t++));     // Imran too low-rated -> waits
        svc.placeOrder(new Order("O4", OrderType.NORMAL, new Location(3, 3), t++));  // Imran
        svc.placeOrder(new Order("O5", OrderType.NORMAL, new Location(0, 1), t++));  // nobody free -> waits

        // Ramesh finishes. Waiting queue has O3 (VIP) and O5 (normal). VIP goes first.
        svc.completeDelivery("D1", new Location(1, 1));
        // Suresh finishes. O5 is normal, Suresh qualifies.
        svc.completeDelivery("D2", new Location(0, 0));
    }
}
```

### Dry run (the interviewer asked for one)

| Step | Event | Result | Why |
|---|---|---|---|
| 1 | O1 VIP at (1,0) | Ramesh | Only Ramesh is rated ≥ 4.5 |
| 2 | O2 normal at (1,1) | Suresh | Suresh is 0 km away (nearest) |
| 3 | O3 VIP at (2,2) | **waits** | Ramesh is busy; Imran is rated 3.5 |
| 4 | O4 normal at (3,3) | Imran | The only free driver |
| 5 | O5 normal | **waits** | Nobody is free |
| 6 | Ramesh is free | Ramesh takes **O3**, not O5 | VIP is first in the queue |
| 7 | Suresh is free | Suresh takes O5 | |

### How to extend it
- **VIP fallback after 2 minutes:** in `retryWaiting`, if a VIP order has waited longer than a limit, try `NormalMatching` for it.
- **Batching:** one driver takes two orders from the same restaurant. Change `available` to a `capacity` count.
- **Better scoring:** replace "nearest" with a score like `distance × w1 − rating × w2`. It's still just a new strategy.
- **Driver accepts or rejects:** add a `PENDING_ACCEPT` state, and reassign on reject or timeout.

### Likely follow-ups
- *"Why `synchronized`?"* Two orders placed at the same moment could both see Ramesh as free and both take him. `synchronized` lets only one thread assign at a time. At scale, you'd lock per driver, or use an atomic "compare-and-set" on the driver's status. In a database, that's `UPDATE drivers SET status='BUSY' WHERE id=? AND status='FREE'` and checking that one row changed.
- *"Complexity?"* Assigning is O(D) per order. A waiting-queue insert is O(log W).
- *"Greedy nearest isn't globally optimal, is it?"* Right. Swiggy's real system collects orders over a short window and solves a matching problem over all orders and drivers in a zone at once. For an LLD round, greedy behind a strategy interface is the right first step.

---

## 4.5 Magical Arena (the ASDE take-home assignment)

> ⚠️ **The exact official problem text isn't public.** Three ASDE candidates (2022, June 2024, Dec 2024) describe it as "a playing arena in which two players fight based on given logic", "a simple attack/defend game" in Java, Node or Go. Public candidate solutions on GitHub describe the rules below. **Read your actual assignment carefully. If the rules differ, follow them, not this.**

### Rules (common interpretation)
- Each player has **health**, **strength** and **attack**, all positive integers.
- Players take turns. The player with **lower health attacks first**.
- Each turn the attacker rolls a 6-sided die, and damage = `attack × roll`. The defender rolls too, and defended = `strength × roll`.
- The defender's health drops by `damage − defended`, if that's positive. Otherwise nothing happens.
- Roles swap. The game ends when someone's health reaches 0.
- **Example:** A attacks with attack 5 and rolls 5, so damage is 25. B defends with strength 10 and rolls 2, so defended is 20. B loses 5 health.

### Entities and why
- `Player`: its health is private and only changed through `takeDamage`, which never goes below 0 (encapsulation).
- `Dice` interface: `SixSidedDice` for real play, `FixedDice` for tests. Without this, you **can't test** a random game.
- `Arena`: runs the fight. The dice are passed in through the constructor (dependency injection).

### Code

```java
import java.util.*;

public class MagicalArena {

    // ---------- package: model ----------
    static class Player {
        private final String name;
        private int health;
        private final int strength;   // used when defending
        private final int attack;     // used when attacking

        Player(String name, int health, int strength, int attack) {
            if (health <= 0 || strength <= 0 || attack <= 0)
                throw new IllegalArgumentException("health, strength and attack must be positive");
            this.name = name; this.health = health; this.strength = strength; this.attack = attack;
        }
        String getName() { return name; }
        int getHealth() { return health; }
        int getStrength() { return strength; }
        int getAttack() { return attack; }
        boolean isAlive() { return health > 0; }
        void takeDamage(int damage) { health = Math.max(0, health - damage); }
    }

    // ---------- package: dice ----------
    interface Dice { int roll(); }

    static class SixSidedDice implements Dice {
        private final Random random;
        SixSidedDice(Random random) { this.random = random; }
        public int roll() { return random.nextInt(6) + 1; }
    }

    static class FixedDice implements Dice {      // for tests
        private final Deque<Integer> values;
        FixedDice(Integer... v) { values = new ArrayDeque<>(Arrays.asList(v)); }
        public int roll() { return values.poll(); }
    }

    // ---------- package: game ----------
    static class Arena {
        static final int MAX_ROUNDS = 10_000;   // guard: if nobody can ever hurt anybody, stop
        private final Dice attackDice, defendDice;
        Arena(Dice attackDice, Dice defendDice) { this.attackDice = attackDice; this.defendDice = defendDice; }

        // Rule (common interpretation): the player with LOWER health attacks first.
        // attack damage = attack * roll; defence = strength * roll; health drops by (damage - defence) if positive.
        Player fight(Player a, Player b) {
            Player attacker = a.getHealth() <= b.getHealth() ? a : b;
            Player defender = attacker == a ? b : a;
            int round = 1;
            while (a.isAlive() && b.isAlive()) {
                if (round > MAX_ROUNDS) throw new IllegalStateException("No winner after " + MAX_ROUNDS + " rounds");
                int attackRoll = attackDice.roll(), defendRoll = defendDice.roll();
                int damage = attacker.getAttack() * attackRoll;
                int defended = defender.getStrength() * defendRoll;
                int actual = Math.max(0, damage - defended);
                defender.takeDamage(actual);
                System.out.printf("Round %d: %s attacks (%d x %d = %d), %s defends (%d x %d = %d), %s health -> %d%n",
                        round++, attacker.getName(), attacker.getAttack(), attackRoll, damage,
                        defender.getName(), defender.getStrength(), defendRoll, defended,
                        defender.getName(), defender.getHealth());
                Player tmp = attacker; attacker = defender; defender = tmp;   // swap roles
            }
            return a.isAlive() ? a : b;
        }
    }

    // ---------- driver ----------
    public static void main(String[] args) {
        Arena arena = new Arena(new SixSidedDice(new Random(42)), new SixSidedDice(new Random(7)));
        Player winner = arena.fight(new Player("A", 50, 5, 10), new Player("B", 100, 10, 5));
        System.out.println("Winner: " + winner.getName());
        ArenaTests.runAll();
    }

    // ---------- tests (in a real submission: JUnit in src/test/java) ----------
    static class ArenaTests {
        static void runAll() {
            lowerHealthAttacksFirst();
            damageNeverNegative();
            invalidPlayerRejected();
            System.out.println("All tests passed");
        }
        static void lowerHealthAttacksFirst() {
            // A (health 10) attacks first: 10*6=60 vs B's 1*1=1 -> B dies in one hit.
            Player a = new Player("A", 10, 1, 10), b = new Player("B", 50, 1, 1);
            Player w = new Arena(new FixedDice(6), new FixedDice(1)).fight(a, b);
            check(w == a && b.getHealth() == 0, "A should win in one hit");
        }
        static void damageNeverNegative() {
            // attack 1*1 = 1, defence 10*6 = 60 -> no damage, health unchanged
            Player a = new Player("A", 5, 1, 1), b = new Player("B", 100, 10, 100);
            new Arena(new FixedDice(1, 6), new FixedDice(6, 1)).fight(a, b);
            check(b.getHealth() == 100, "defence bigger than attack must not heal or hurt");
        }
        static void invalidPlayerRejected() {
            try { new Player("X", 0, 1, 1); check(false, "should throw"); }
            catch (IllegalArgumentException expected) { }
        }
        static void check(boolean ok, String msg) { if (!ok) throw new AssertionError(msg); }
    }
}
```

### What reviewers of a take-home look for
A take-home is judged like real work. A working program is the minimum; the polish matters.

1. **A README** at the top of the repo, with:
   - what the program does
   - the rules you assumed
   - how to build and run it (`mvn package`, `java -jar …`)
   - how to run the tests
   - design decisions in 3–5 bullets ("Dice is an interface so tests can use fixed rolls")
2. **Unit tests** with JUnit 5 in `src/test/java`. Use the same idea as `ArenaTests` above: fixed dice, check the exact outcome. Cover:
   - normal play
   - zero damage when defence is bigger
   - health never going negative
   - invalid players rejected
   - who attacks first
3. **Clean structure:** `model/`, `game/`, `dice/` packages; small classes; no `static` global state; no magic numbers (use a `DICE_SIDES = 6` constant).
4. **Input validation:** reject zero or negative attributes.
5. **Readable output** that shows each round, like the demo.
6. **Git history** with several meaningful commits ("Add Player model", "Add fight loop", "Add tests"), not one "final" commit.
7. **No over-engineering.** Two players, one clear game loop. Don't add Spring Boot or a database.

### Likely follow-ups (in the discussion after the take-home)
- *"How would you support more than two players?"* Use a turn queue (as in Snake & Ladder) and a rule for picking whom to attack (a `TargetSelectionStrategy`).
- *"How would you add special powers?"* A `Power` interface that modifies damage before it's applied.
- *"How did you test randomness?"* By injecting the dice. A fixed sequence makes the outcome predictable.
- *"Can the game loop forever?"* Yes, with unlucky numbers. If each player's best attack (attack × 6) is smaller than the other's weakest defence (strength × 1), nobody ever takes damage. That's why the code has a `MAX_ROUNDS` guard that stops the fight with a clear error. You could also declare a draw. Pointing out this edge case yourself earns credit.

---

# Part 5: Discussion-style LLD (short answers)

These rounds are conversations. You explain classes, tables and APIs. You may write interfaces, but rarely full code.

## 5.1 Follow/unfollow for a social media app (ASDE, Dec 2024)

**What they wanted:** requirements → DB schema → API endpoints with HTTP verbs.

**Requirements:**
- follow a user
- unfollow a user
- list my followers and the people I follow
- show follower/following counts
- (optional) check whether A follows B

**Clarify:** "Do private accounts need follow *requests*?" If yes, add a status column (`PENDING` / `ACCEPTED`).

**Schema:**
```sql
CREATE TABLE users (
  id          BIGINT PRIMARY KEY,
  username    VARCHAR(50) UNIQUE NOT NULL,
  follower_count  INT DEFAULT 0,      -- stored copy so we don't COUNT(*) every time
  following_count INT DEFAULT 0
);

CREATE TABLE follows (
  follower_id BIGINT NOT NULL,        -- the person who clicks "Follow"
  followee_id BIGINT NOT NULL,        -- the person being followed
  created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (follower_id, followee_id),          -- can't follow the same person twice
  FOREIGN KEY (follower_id) REFERENCES users(id),
  FOREIGN KEY (followee_id) REFERENCES users(id),
  CHECK (follower_id <> followee_id)               -- can't follow yourself
);
CREATE INDEX idx_followee ON follows(followee_id, created_at);  -- fast "who follows X"
```

**Why these indexes:** an **index** is like the index at the back of a book. It lets the database jump to the right rows instead of reading every row.
- The primary key `(follower_id, followee_id)` already makes "who does X follow?" and "does A follow B?" fast.
- "Who follows X?" searches by `followee_id`, so it needs its own index.

**API endpoints:**

| Action | HTTP verb + path | Notes |
|---|---|---|
| Follow | `POST /users/{id}/follow` | Creates a row. The current user comes from the auth token, never from the body. |
| Unfollow | `DELETE /users/{id}/follow` | Deletes the row. |
| My followers | `GET /users/{id}/followers?cursor=…&limit=20` | **Paginated**. Never return 1 million rows at once. |
| Who I follow | `GET /users/{id}/following?cursor=…&limit=20` | |
| Counts | `GET /users/{id}` | Includes `follower_count` and `following_count`. |

**HTTP verbs in plain words:**
- **GET:** read; changes nothing.
- **POST:** create something.
- **PUT:** replace something completely.
- **PATCH:** change part of something.
- **DELETE:** remove something.

**Idempotent** means doing it twice has the same effect as doing it once. GET, PUT and DELETE should be idempotent. Make follow idempotent too: if the row already exists, return success instead of an error. That way, a double-tap on "Follow" doesn't break anything.

**Follow-ups:**
- *Keeping counts correct:* update the `follows` row and the two counts in **one transaction**. Or update the counts later from an event, accepting a small delay.
- *A celebrity with 50 million followers:* the counts row becomes a hotspot. Use a cached counter (Redis `INCR`) and write to the DB in batches.
- *Pagination:* use a **cursor** ("give me 20 after created_at = X"), not `OFFSET 100000`, which makes the DB skip 100k rows each time.

## 5.2 Job scheduler (asked Dec 2024: "write the interface and scaffold; add, cancel, suspend, queue jobs")

```java
enum JobStatus { QUEUED, RUNNING, SUSPENDED, CANCELLED, DONE, FAILED }
record Job(String id, Runnable task, long runAtMs, int priority) {}

interface JobScheduler {
    String submit(Runnable task, long runAtMs, int priority);   // returns job id
    boolean cancel(String jobId);
    boolean suspend(String jobId);
    boolean resume(String jobId);
    JobStatus status(String jobId);
}
```

**How you'd build it:**
- A `PriorityQueue<Job>` ordered by `runAtMs`, then by priority. The next job due is always at the front.
- A `Map<String, Job>` and a `Map<String, JobStatus>` for O(1) lookups by id.
- One **dispatcher** thread looks at the front job. If it's due, it hands the job to a **worker pool**: a fixed set of threads that run jobs, like a kitchen with 4 cooks taking tickets. In Java, that's `Executors.newFixedThreadPool(4)`.
- **Cancel / suspend:** change the status in the map. When the dispatcher pops a job, it skips anything not `QUEUED`. This is cheaper than removing it from the middle of a heap.
- **Resume:** set the status back to `QUEUED` and re-insert the job.
- **Retries:** on failure, re-submit with `runAt = now + backoff` and keep a retry count.

**Link to your resume:** this is exactly your Atlassian SQS + worker pool design, inside one program instead of across machines.

## 5.3 Parking lot (asked, with a Singleton follow-up)

**Entities:**
- `ParkingLot` (Singleton)
- `Floor`
- `Slot(id, type, occupied)`
- `Vehicle(number, type)`, with `VehicleType` = BIKE / CAR / TRUCK
- `Ticket(id, vehicle, slot, entryTime)`
- `SlotAllocationStrategy` (nearest-first, or a floor-wise fill)
- `PricingStrategy` (hourly, flat)
- `ParkingService` with `park(vehicle)` → Ticket and `unpark(ticketId)` → fee

**Data structures:** per floor and per vehicle type, a `TreeSet<Slot>` or `PriorityQueue` of free slots, ordered by distance from the entrance. Taking the nearest free slot is then O(log n).

**Singleton follow-up:**
- *"Why is ParkingLot a Singleton?"* There is one physical lot, and every entry gate must see the same slot state.
- *"How do you make it thread-safe?"* Use double-checked locking with `volatile`, or an `enum` (see 2.4).
- *"What about two cars at two gates grabbing the same slot?"* That's a separate problem from Singleton. Make `park` `synchronized` per floor, or use atomic slot claims.

## 5.4 Library management (asked in 45 minutes, Android SDE-1)

**Entities:**
- `Book(isbn, title, author)` is the title.
- `BookCopy(copyId, isbn, status)` is each physical copy. A library has 3 copies of one book, so separating title and copy is the key design point.
- `Member(id, name, maxBooks)`
- `Loan(copyId, memberId, issueDate, dueDate, returnDate)`
- `LibraryService` with `search(title/author)`, `issue(copyId, memberId)`, `return(copyId)`, and `fine(loan)`

**Rules:**
- A member can't exceed their book limit.
- A copy can't be issued twice.
- Fine = days late × rate, which is a `FinePolicy` strategy.

**Extension:** a waitlist, i.e. a queue of members per book, notified when a copy is returned (Observer).

## 5.5 LRU cache (very common, asked at Swiggy multiple times)

**What it is:** a fixed-size store that, when full, throws out the item **used least recently**.

**Real life:** your phone's recent-apps list. Open an app and it jumps to the front. Old ones fall off the end.

**The classic design:** a `HashMap` (key → node) for O(1) lookup, plus a **doubly linked list** to keep the usage order. On `get` or `put`, move the node to the front. When full, remove the node at the back. Both operations are O(1).

**Shortcut in Java:** `LinkedHashMap` with `accessOrder = true` already does this:

```java
class LRUCache<K, V> {
    private final int capacity;
    private final LinkedHashMap<K, V> map;
    LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new LinkedHashMap<>(16, 0.75f, true) {      // true = order by access
            @Override protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                return size() > LRUCache.this.capacity;
            }
        };
    }
    V get(K key) { return map.get(key); }
    void put(K key, V value) { map.put(key, value); }
}
// capacity 2: put a, put b, get a, put c  ->  b is evicted, cache holds [a, c]
```

**In an interview,** mention the shortcut, but be ready to write the HashMap + doubly-linked-list version by hand (LeetCode 146). It's in the DSA doc `02`.

## 5.6 Rate limiter as LLD (Glassdoor ASDE mention)

**What it is:** a gate that allows each user at most N requests per time window. It protects services from overload. You built one at Myntra to protect Accio.

**Token bucket, in plain words:** each user has a bucket that holds up to `capacity` tokens. Tokens drip back in at a fixed rate. Each request takes one token. If the bucket is empty, the request is rejected. Bursts are fine up to the capacity, and the long-run rate stays limited.

**Real life:** a coffee shop gives you 3 coupons, and you get one new coupon every minute. You can grab 3 coffees at once, but then you wait.

```java
interface RateLimiter { boolean allow(String clientId); }

class TokenBucketLimiter implements RateLimiter {
    private final int capacity;
    private final double refillPerMs;
    private final Map<String, double[]> buckets = new HashMap<>(); // [tokens, lastRefillMs]
    TokenBucketLimiter(int capacity, int refillPerSecond) {
        this.capacity = capacity; this.refillPerMs = refillPerSecond / 1000.0;
    }
    public synchronized boolean allow(String clientId) { return allowAt(clientId, System.currentTimeMillis()); }
    synchronized boolean allowAt(String clientId, long nowMs) {
        double[] b = buckets.computeIfAbsent(clientId, k -> new double[]{capacity, nowMs});
        b[0] = Math.min(capacity, b[0] + (nowMs - b[1]) * refillPerMs);   // add tokens earned since last time
        b[1] = nowMs;
        if (b[0] >= 1) { b[0] -= 1; return true; }
        return false;
    }
}
// capacity 3, refill 1/s: 5 requests at t=0 -> true true true false false; at t=2s -> true
```

**Design points to mention:**
- `RateLimiter` is an interface, so token bucket, fixed window and sliding window are interchangeable strategies.
- Passing the time in (`allowAt`) makes it testable.
- Across many servers, the counters live in Redis.

## 5.7 Calendar / meeting scheduler (asked in a 3-hour round, OLD)

**Requirements:**
- create a meeting with participants and a time slot
- show a user's calendar
- accept or reject an invite
- detect clashes

**Entities:**
- `User`
- `Meeting(id, organiser, start, end, participants)`
- `Invite(meetingId, userId, status: PENDING/ACCEPTED/REJECTED)`
- `Calendar` (per user: a `TreeMap<startTime, Meeting>`)
- `MeetingService`

**The core algorithm, a clash check:** two meetings overlap if `a.start < b.end && b.start < a.end`. With a `TreeMap` sorted by start time, you only check the meeting just before and just after the new one, which is O(log n). This is the "Merge Intervals" idea from DSA, which Swiggy also asks.

**Extensions:**
- **Find a free slot for everyone:** merge all participants' busy intervals and look for gaps.
- **Recurring meetings:** a `RecurrenceRule` strategy.
- **Meeting rooms:** a `Room` with its own calendar.

## 5.8 Other designs reported (one line each)
- **Instagram APIs (post, feed, follow):** the follow schema above, plus `posts(id, user_id, created_at)`. Build the feed by merging recent posts from people you follow, then paginating.
- **Bike rental:**
  - `Bike(status)`, `Station`, `Rental(start, end)`
  - `PricingStrategy` (per minute, per day)
  - state: AVAILABLE → RENTED → AVAILABLE / MAINTENANCE
- **Design a stack using an array (ASDE Mar 2026, "classes and interfaces"):**
  - Define a `Stack<T>` interface with `push`, `pop`, `peek`, `isEmpty` and `size`.
  - `ArrayStack<T>` holds an array and a `top` index. It **doubles** the array when full, and throws an exception on `pop` from an empty stack.
  - Mention that a `LinkedStack` could implement the same interface.

---

# Part 6: Final checklist and practice plan

## Practice plan (6 sessions)

| Day | Do | Time |
|---|---|---|
| 1 | Re-read Parts 1–3. Then type the patterns in Part 2 from memory. | 90 min |
| 2 | **Snake & Ladder** from scratch with a 60-minute timer. Compare with 4.1. | 75 min |
| 3 | **Splitwise** from scratch with a 60-minute timer. This is the most important one. Compare with 4.2. | 75 min |
| 4 | **Food cart with offers** (4.3), then talk through **follow/unfollow** (5.1) out loud. | 90 min |
| 5 | **Driver assignment** (4.4) including the dry run, then a parking lot outline (5.3). | 90 min |
| 6 | Redo Splitwise in **45 minutes**. Explain the code to a friend or to yourself out loud. | 60 min |

## The night before
- [ ] Can I explain the four pillars with a Swiggy example each?
- [ ] Interface vs abstract class: three differences?
- [ ] Strategy, Singleton (with double-checked locking), Factory, Observer: one example each?
- [ ] Do I have a project template ready in my IDE, so I'm not fighting setup in the round?
  - a Java project that compiles
  - a `Main` class
  - packages created
- [ ] Do I remember the time budget: **something running by the halfway mark**?
- [ ] Money in paise, validation before mutation, a demo in `main` that shows one rejected input?
