# Java and Spring Boot from Zero (Swiggy Round 1)

> **Who this is for:** you know C++ from DSA, you have *used* Kotlin + Spring Boot at Atlassian and Go at Myntra, but nobody ever sat you down and explained how Java actually works. This file does that.
>
> **Why it matters for Swiggy:**
> - In the 2026 SDE-1 Round 1 (run by a third-party interviewer), the candidate coded for about 30 minutes and then got **20–25 rapid-fire Java questions in about 25 minutes**. The topics were OOP, JVM/JDK, garbage collection, Collections and multithreading.
> - The Dec 2025 round asked:
>   - interface vs abstract class
>   - checked vs unchecked exceptions
>   - the `final` keyword
>   - `@RestController`
>   - how Spring Boot manages configuration
>   - the Spring Bean lifecycle
> - Older rounds asked about `static`, `volatile`, inheritance vs composition, Singleton and polymorphism.
> - **Since 2026, the OA and DSA rounds allow only Java or Go.** So you must be able to write DSA in Java without fumbling.
>
> **Reading time:** about 2.5 hours the first time. Type out the code yourself; don't just read it.
>
> **How the file is laid out:** Parts 1–2 get you coding DSA in Java. Parts 3–8 are the theory interviewers ask about. Part 9 is Spring Boot. Part 10 is a 50-question rapid-fire bank: practise it out loud.

Every code block with a `main` method in this file was compiled and run with JDK 21.

---

## Part 1: Java for someone who knows C++

### 1.1 How Java code runs: JDK, JRE, JVM, bytecode, JIT

In C++, the compiler turns your code into **machine code**: instructions for one specific kind of CPU (say, Intel on Windows). Take that `.exe` to a Mac and it won't run.

Java adds a middle step:

```
Hello.java  --(javac compiler)-->  Hello.class (bytecode)  --(JVM)-->  runs on the CPU
```

- **Bytecode** is like machine code for an imaginary computer. It isn't for Intel or ARM; it is for the "Java Virtual Machine".
- **JVM (Java Virtual Machine)** is a program installed on each real computer. It reads bytecode and runs it on the real CPU. There is a JVM for Windows, one for Mac, one for Linux. The *same* `.class` file runs on all of them. This is the famous **"write once, run anywhere."**
- **JIT (Just-In-Time) compiler** is part of the JVM. At first the JVM *interprets* bytecode line by line, which is slow. When it notices a method running many times (a "hot" method, like your loop running a million times), the JIT compiles that method into real machine code and caches it. That's why Java is fast after a warm-up period.
- **JRE (Java Runtime Environment)** = JVM + the standard libraries (`java.util`, `java.lang`...). It's what you need to *run* Java programs.
- **JDK (Java Development Kit)** = JRE + developer tools (`javac` the compiler, `jar`, debugger). It's what you need to *write* Java programs.

**Real-life analogy (a Swiggy recipe):**
- Bytecode is a recipe written in a universal recipe language.
- Each kitchen (Windows, Mac, Linux) has its own chef: the JVM. The chef reads the universal recipe and cooks it with that kitchen's equipment.
- The JIT is the chef memorising a dish they make 500 times a day, so they stop reading the recipe for it.
- JRE = the chef + the kitchen's ingredients. JDK = all that + tools for *writing* new recipes.

> **Interview one-liner:** "JDK is for developing (has javac), JRE is for running (JVM + libraries), JVM executes bytecode. Bytecode makes Java platform-independent; the JIT compiles hot code to native machine code for speed."

### 1.2 What's different from C++

| Topic | C++ | Java |
|---|---|---|
| Memory | You call `new` and `delete`. Forget `delete` and you leak memory. | You call `new`. **There is no `delete`.** The **garbage collector (GC)** frees objects nobody uses any more. |
| Pointers | Real pointers, pointer arithmetic (`p++`) | No pointers you can touch. Object variables hold **references** (a safe, hidden pointer). No arithmetic. |
| Where code lives | Free functions allowed (`int main()`) | **Everything is inside a class.** Even `main` is `public static void main(String[] args)` inside a class. |
| Objects on stack? | `Car c;` creates a Car on the stack | Objects are **always on the heap**. `Car c;` only declares a reference (currently `null`). |
| Passing to functions | By value, by reference (`&`), by pointer | **Always by value**. For objects, the *value* is the reference, so the function can change the object's fields but cannot make your variable point to a different object. |
| Multiple inheritance | Allowed for classes | **Not allowed for classes.** A class extends one class but can implement many interfaces. |
| Operator overloading | Allowed | Not allowed (except `+` on Strings). |
| Strings | `std::string` is mutable | `String` is **immutable**: it can't change after creation. |
| Arrays | No bounds check | Bounds checked: out-of-range index throws `ArrayIndexOutOfBoundsException`. |
| Integer sizes | `int` size can vary | Fixed: `int` is always 32-bit, `long` is 64-bit. |

**"Java is pass-by-value" (a favourite trick question).** Look at this:

```java
public class PassByValue {
    static class Box { int value; }

    static void change(Box b) {
        b.value = 99;      // changes the SAME object the caller sees
        b = new Box();     // only changes our local copy of the reference
        b.value = -1;      // caller never sees this
    }

    public static void main(String[] args) {
        Box box = new Box();
        box.value = 1;
        change(box);
        System.out.println(box.value); // 99
    }
}
```

Think of a reference as a **slip of paper with a house address on it**. Java photocopies the slip and gives the copy to the function. The function can walk to the house and repaint the walls (`b.value = 99`), and you'll see the new paint. If it scribbles a new address on *its copy* (`b = new Box()`), your slip still has the old address.

### 1.3 The minimum syntax for interview DSA

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        // Reading input (OA platforms like HackerRank)
        Scanner sc = new Scanner("3\n10 20 30");  // in a real OA: new Scanner(System.in)
        int n = sc.nextInt();
        int[] arr = new int[n];
        for (int i = 0; i < n; i++) arr[i] = sc.nextInt();

        // Primitive types
        long big = 1_000_000_007L;       // L suffix for long
        double d = 3.5;
        char c = 'a';
        boolean ok = true;

        // 2D array
        int[][] grid = new int[3][4];     // all zeros by default
        grid[1][2] = 5;

        // Fill / copy arrays
        int[] dp = new int[n + 1];
        Arrays.fill(dp, -1);
        int[] copy = Arrays.copyOf(arr, n);

        // Loops
        for (int x : arr) System.out.print(x + " ");   // for-each
        System.out.println();

        // Char arithmetic
        int idx = 'c' - 'a';               // 2, used for 26-letter frequency arrays
        char next = (char) ('a' + 1);      // 'b'

        // Max/min values
        int inf = Integer.MAX_VALUE;
        long linf = Long.MAX_VALUE;

        // Math
        int m = Math.max(3, 7);
        int absVal = Math.abs(-4);

        // Ternary
        String parity = (n % 2 == 0) ? "even" : "odd";

        System.out.println(n + " " + big + " " + d + " " + c + " " + ok + " " + grid[1][2] + " "
                + dp[0] + " " + copy.length + " " + idx + " " + next + " " + inf + " " + linf
                + " " + m + " " + absVal + " " + parity);
    }
}
```

**Faster input for big OA inputs:** `Scanner` is slow for around 10^5+ numbers. Use `BufferedReader`:

```java
import java.io.*;
import java.util.*;

public class FastInput {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new StringReader("5\n1 2 3 4 5"));
        // in a real OA: new BufferedReader(new InputStreamReader(System.in))
        int n = Integer.parseInt(br.readLine().trim());
        StringTokenizer st = new StringTokenizer(br.readLine());
        long sum = 0;
        for (int i = 0; i < n; i++) sum += Integer.parseInt(st.nextToken());
        System.out.println(sum); // 15
    }
}
```

**In a LeetCode-style round** you only write the method inside `class Solution { ... }`. No input reading is needed.

---

## Part 2: The DSA toolkit in Java

In C++ you have the STL (`vector`, `map`, `unordered_map`, `stack`, `priority_queue`). Java has the **Collections Framework** in `java.util`. Here's the map:

| C++ | Java | Notes |
|---|---|---|
| `vector<int>` | `ArrayList<Integer>` | Resizable array |
| `unordered_map<K,V>` | `HashMap<K,V>` | O(1) average, no order |
| `map<K,V>` (sorted) | `TreeMap<K,V>` | O(log n), sorted keys |
| `unordered_set` / `set` | `HashSet` / `TreeSet` | |
| `stack<int>` | `ArrayDeque<Integer>` | Use `push`/`pop`/`peek` |
| `queue<int>` | `ArrayDeque<Integer>` | Use `offer`/`poll`/`peek` |
| `deque<int>` | `ArrayDeque<Integer>` | `offerFirst`/`offerLast`/`pollFirst`/`pollLast` |
| `priority_queue<int>` (max-heap) | `PriorityQueue<Integer>` | Java's is a **MIN-heap** by default! |
| `pair<int,int>` | `int[]{a, b}` or a small `record` | |

**Generics (`<Integer>`):** the angle brackets say what type the collection holds. Collections can only hold **objects**, not primitives. So you write `ArrayList<Integer>`, not `ArrayList<int>`. More on this in 2.6.

### 2.1 ArrayList (resizable array)

Like a `vector`. A row of lockers that automatically moves into a bigger room when full.

```java
import java.util.*;

public class ArrayListDemo {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();   // "List" is the interface, ArrayList the class
        list.add(5);                // append, amortised O(1)
        list.add(3);
        list.add(0, 9);             // insert at index 0, O(n) because elements shift
        int first = list.get(0);    // O(1)
        list.set(1, 7);             // replace index 1
        list.remove(Integer.valueOf(3)); // remove by VALUE
        list.remove(0);             // remove by INDEX  (careful: different overloads!)
        System.out.println(list + " size=" + list.size() + " first=" + first);
        System.out.println(list.contains(7));     // O(n) linear search

        // 2D: list of lists (adjacency list for graphs)
        int n = 4;
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        adj.get(0).add(1);
        adj.get(1).add(0);
        System.out.println(adj);
    }
}
```

**Trap:** `list.remove(3)` removes **index** 3, not the value 3. To remove the value, write `list.remove(Integer.valueOf(3))`.

### 2.2 HashMap and HashSet

A **HashMap** stores key → value pairs. Look-up is O(1) on average. Real life: a restaurant's **order counter with numbered pigeonholes**. Your order number (key) tells the staff exactly which hole holds your food (value). They don't search every shelf.

```java
import java.util.*;

public class HashMapDemo {
    public static void main(String[] args) {
        Map<String, Integer> freq = new HashMap<>();
        String[] dishes = {"biryani", "dosa", "biryani", "idli", "biryani"};

        for (String d : dishes) {
            freq.put(d, freq.getOrDefault(d, 0) + 1);  // classic counting line
            // shorter alternative: freq.merge(d, 1, Integer::sum);
        }
        System.out.println(freq.get("biryani"));         // 3
        System.out.println(freq.containsKey("pizza"));   // false
        System.out.println(freq.get("pizza"));           // null (not 0!)

        // Iterate
        for (Map.Entry<String, Integer> e : freq.entrySet()) {
            System.out.println(e.getKey() + " -> " + e.getValue());
        }

        // Map of lists (group anagrams style)
        Map<String, List<String>> groups = new HashMap<>();
        groups.computeIfAbsent("key", k -> new ArrayList<>()).add("value");
        System.out.println(groups);

        // HashSet: just keys, no values. "Have I seen this before?"
        Set<Integer> seen = new HashSet<>();
        System.out.println(seen.add(4));  // true (newly added)
        System.out.println(seen.add(4));  // false (already there)
        System.out.println(seen.contains(4));
    }
}
```

**Trap:** `map.get(missingKey)` returns `null`. If you unbox `null` into an `int` (`int x = map.get(k);`), you get a `NullPointerException`. Use `getOrDefault`.

### 2.3 TreeMap and TreeSet (sorted)

They keep keys **sorted** (internally a red-black tree, a self-balancing BST). Every operation is O(log n). Use them when you need "the smallest key ≥ x" and similar queries.

Real life: a **calendar of meeting slots**. You often ask "what's the next free slot after 3 pm?"

```java
import java.util.*;

public class TreeMapDemo {
    public static void main(String[] args) {
        TreeMap<Integer, String> slots = new TreeMap<>();
        slots.put(9, "standup");
        slots.put(14, "design review");
        slots.put(11, "1:1");

        System.out.println(slots.firstKey());       // 9  (smallest)
        System.out.println(slots.lastKey());        // 14 (largest)
        System.out.println(slots.ceilingKey(10));   // 11 (smallest key >= 10)
        System.out.println(slots.floorKey(10));     // 9  (largest key <= 10)
        System.out.println(slots.higherKey(11));    // 14 (strictly greater)
        System.out.println(slots.ceilingKey(15));   // null (none)
        System.out.println(slots.headMap(11));      // keys < 11: {9=standup}

        TreeSet<Integer> ts = new TreeSet<>(List.of(5, 1, 9));
        System.out.println(ts.first() + " " + ts.pollFirst() + " " + ts); // 1 1 [5, 9]
    }
}
```

### 2.4 ArrayDeque as a stack and as a queue

**Don't use the old `Stack` class.** It is synchronized (slow) and legacy. Use `ArrayDeque` for both stacks and queues.

- **Stack** = a pile of plates: last in, first out (LIFO).
- **Queue** = the line at a Swiggy pickup counter: first in, first out (FIFO).

```java
import java.util.*;

public class DequeDemo {
    public static void main(String[] args) {
        // STACK
        Deque<Character> stack = new ArrayDeque<>();
        stack.push('(');
        stack.push('[');
        System.out.println(stack.peek());  // '[' (top)
        System.out.println(stack.pop());   // '['
        System.out.println(stack.isEmpty());

        // QUEUE (BFS)
        Queue<Integer> q = new ArrayDeque<>();
        q.offer(1);
        q.offer(2);
        System.out.println(q.poll());      // 1 (front)
        System.out.println(q.peek());      // 2

        // Queue of pairs for grid BFS
        Queue<int[]> bfs = new ArrayDeque<>();
        bfs.offer(new int[]{0, 0});
        int[] cell = bfs.poll();
        System.out.println(cell[0] + "," + cell[1]);
    }
}
```

**Trap:** `ArrayDeque` does **not allow `null`** elements. Also, `pop()` on an empty deque throws an exception, but `poll()` returns `null`.

### 2.5 PriorityQueue (heap) with comparators

A **heap** always gives you the smallest (or largest) item fast. Real life: the **hospital emergency room**. It doesn't serve in arrival order; it always takes the most critical patient next.

Java's `PriorityQueue` is a **min-heap by default**. For a max-heap, pass a comparator.

A **comparator** is a little rule that says how to order two things. `(a, b) -> a - b` means: if the result is negative, `a` comes first.

```java
import java.util.*;

public class HeapDemo {
    record Order(String id, int priority, int distanceKm) {}

    public static void main(String[] args) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
        for (int x : new int[]{5, 1, 8, 3}) { minHeap.offer(x); maxHeap.offer(x); }
        System.out.println(minHeap.poll() + " " + maxHeap.poll()); // 1 8

        // Heap of arrays: [node, distance], smallest distance first (Dijkstra)
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
        pq.offer(new int[]{2, 10});
        pq.offer(new int[]{3, 4});
        System.out.println(pq.peek()[0]); // 3

        // Heap of objects: highest priority first, then nearest
        PriorityQueue<Order> orders = new PriorityQueue<>(
            Comparator.comparingInt(Order::priority).reversed()
                      .thenComparingInt(Order::distanceKm));
        orders.offer(new Order("A", 1, 2));
        orders.offer(new Order("B", 5, 9));
        orders.offer(new Order("C", 5, 3));
        System.out.println(orders.poll().id()); // C (priority 5, closer than B)
    }
}
```

**Trap:** `(a, b) -> a - b` can **overflow** when the values are huge (for example `Integer.MIN_VALUE - 1`). `Integer.compare(a, b)` is always safe. Use it.

### 2.6 LinkedHashMap (remembers order) and the LRU cache

A `LinkedHashMap` is a HashMap plus a doubly linked list through the entries. It remembers **insertion order**, or **access order** if you ask it to. That makes LRU Cache (asked at Swiggy, and called "most asked" in one ASDE review) almost free:

```java
import java.util.*;

public class LRUDemo {
    static class LRUCache<K, V> extends LinkedHashMap<K, V> {
        private final int capacity;
        LRUCache(int capacity) {
            super(16, 0.75f, true);    // true = order by ACCESS, not insertion
            this.capacity = capacity;
        }
        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return size() > capacity;  // auto-evict least recently used
        }
    }

    public static void main(String[] args) {
        LRUCache<Integer, String> cache = new LRUCache<>(2);
        cache.put(1, "one");
        cache.put(2, "two");
        cache.get(1);            // 1 is now most recently used
        cache.put(3, "three");   // evicts 2
        System.out.println(cache.keySet()); // [1, 3]
    }
}
```

> In an interview, show this, but **be ready to build it by hand** with a HashMap plus your own doubly linked list. That's what they usually want to see. The DSA doc covers the hand-built version.

### 2.7 Sorting with lambdas

A **lambda** is a short, unnamed function: `(a, b) -> a[0] - b[0]`. Part 8 explains lambdas properly.

```java
import java.util.*;

public class SortDemo {
    public static void main(String[] args) {
        int[] nums = {5, 2, 9, 1};
        Arrays.sort(nums);                               // primitives: ascending only
        System.out.println(Arrays.toString(nums));

        // Descending needs Integer[] (objects), not int[]
        Integer[] boxed = {5, 2, 9, 1};
        Arrays.sort(boxed, Collections.reverseOrder());
        System.out.println(Arrays.toString(boxed));

        // Sort intervals by start (Merge Intervals, asked at Swiggy)
        int[][] intervals = {{8, 10}, {1, 3}, {2, 6}};
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
        System.out.println(Arrays.deepToString(intervals));

        // Sort a List of Strings by length, then alphabetically
        List<String> words = new ArrayList<>(List.of("dosa", "idli", "biryani", "vada"));
        words.sort(Comparator.comparingInt(String::length).thenComparing(Comparator.naturalOrder()));
        System.out.println(words);   // [dosa, idli, vada, biryani]

        Collections.sort(words);     // natural (alphabetical) order
        System.out.println(words);
    }
}
```

### 2.8 String vs StringBuilder

`String` is **immutable**: once created, it can never change. `s = s + "x"` doesn't modify `s`. It builds a **brand-new String** and copies every character into it. Doing that in a loop of n steps costs O(n²).

`StringBuilder` is a **mutable** buffer. Appending is amortised O(1).

Real life:
- A `String` is a **printed menu**. To change a price, you reprint the whole menu.
- A `StringBuilder` is a **whiteboard menu**. You just rub out and rewrite.

```java
public class StringDemo {
    public static void main(String[] args) {
        String s = "swiggy";
        System.out.println(s.length() + " " + s.charAt(0) + " " + s.substring(1, 4)); // 6 s wig
        System.out.println(s.indexOf('g') + " " + s.toUpperCase() + " " + s.contains("gg"));
        char[] arr = s.toCharArray();           // to modify characters
        arr[0] = 'S';
        System.out.println(new String(arr));     // Swiggy
        String[] parts = "a,b,c".split(",");
        System.out.println(parts.length);        // 3

        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 5; i++) sb.append(i).append(' ');
        sb.reverse();
        sb.setCharAt(0, '#');
        sb.deleteCharAt(sb.length() - 1);
        System.out.println(sb.toString());

        // Comparing strings: ALWAYS use equals, never ==
        String a = new String("hi");
        String b = "hi";
        System.out.println(a == b);       // false (different objects)
        System.out.println(a.equals(b));  // true  (same characters)
    }
}
```

### 2.9 int vs Integer, autoboxing, and the `==` trap

- `int` is a **primitive**: a raw 32-bit number stored directly. Fast, can't be `null`.
- `Integer` is a **wrapper class**: an object that contains an int. It can be `null`, and it's what collections store.
- **Autoboxing:** Java converts `int → Integer` automatically when needed (`list.add(5)`). **Unboxing** is the reverse (`int x = list.get(0)`).

**The famous trap:** `==` on two objects checks "**same object in memory?**", not "same value?". Java caches `Integer` objects for -128..127, so small values *happen* to be the same object:

```java
import java.util.*;

public class IntegerTrap {
    public static void main(String[] args) {
        Integer a = 127, b = 127;
        System.out.println(a == b);        // true  (cached, same object)
        Integer c = 128, d = 128;
        System.out.println(c == d);        // false (two different objects!)
        System.out.println(c.equals(d));   // true  (correct way)

        // Real bug: comparing values from two lists or maps
        List<Integer> x = List.of(1000), y = List.of(1000);
        System.out.println(x.get(0) == y.get(0));             // false
        System.out.println(x.get(0).intValue() == y.get(0));  // true (one side unboxed)

        // Null unboxing
        Map<String, Integer> m = new HashMap<>();
        try {
            int v = m.get("missing");      // NullPointerException
            System.out.println(v);
        } catch (NullPointerException e) {
            System.out.println("NPE from unboxing null");
        }
    }
}
```

This bug really bites in "Min Stack" or "two stacks equal?" problems, where you compare `stack.peek() == other.peek()`. Use `.equals()`, or store values in `int` variables first.

### 2.10 Common Java DSA pitfalls (checklist)

1. **Integer overflow.** `int` goes up to about 2.1 × 10^9. Use `long` for sums and products: `long ans = (long) a * b;`. Note that the cast must come **before** the multiply.
2. **Mid overflow in binary search.** Write `int mid = lo + (hi - lo) / 2;`, not `(lo + hi) / 2`.
3. **`==` on Strings or Integers.** Use `.equals()`.
4. **`list.remove(int)` vs `list.remove(Object)`.** See 2.1.
5. **Modifying a collection while iterating it with for-each** throws `ConcurrentModificationException` (see Part 6). Use an `Iterator` and `it.remove()`, or collect the changes first.
6. **`Arrays.sort(int[], comparator)` doesn't exist.** Comparators work only on object arrays (`Integer[]`, `int[][]`).
7. **Default PriorityQueue is a min-heap.**
8. **Char vs String:** `'a'` is a char and `"a"` is a String. `s.charAt(i) == 'x'` is fine, because chars are primitives.
9. **Modulo of negatives:** `-7 % 3` is `-1` in Java. For a positive mod, use `((x % m) + m) % m`.
10. **Recursion depth.** Java's default stack can overflow at around 10^4–10^5 deep recursion. For huge inputs, prefer iterative DFS.

---

## Part 3: OOP in Java

**OOP (Object-Oriented Programming)** means modelling a program as **objects** that hold data (fields) and behaviour (methods), the way the real world has things that *have* properties and *do* stuff.

### 3.1 Classes, objects, constructors, `this`, `super`

- A **class** is a blueprint, like the *design* of a delivery bike.
- An **object** is one actual thing built from the blueprint: *this* bike, registration KA-01-1234.
- A **constructor** is a special method that runs when you create an object with `new`. It sets up the initial state. It has the same name as the class and no return type.
- **`this`** means "the current object". You use it when a parameter has the same name as a field.
- **`super`** means "my parent class". `super(...)` calls the parent's constructor; `super.method()` calls the parent's version of a method.

```java
public class ConstructorDemo {
    static class Vehicle {
        protected String regNo;
        Vehicle(String regNo) { this.regNo = regNo; }   // this.regNo = field, regNo = parameter
        String describe() { return "Vehicle " + regNo; }
    }

    static class Bike extends Vehicle {
        private final int engineCc;
        Bike(String regNo, int engineCc) {
            super(regNo);                 // must be the FIRST line: build the parent part first
            this.engineCc = engineCc;
        }
        Bike(String regNo) { this(regNo, 110); }   // constructor chaining with this(...)
        @Override
        String describe() { return super.describe() + " (bike, " + engineCc + "cc)"; }
    }

    public static void main(String[] args) {
        Vehicle v = new Bike("KA01-1234");
        System.out.println(v.describe()); // Vehicle KA01-1234 (bike, 110cc)
    }
}
```

If you write **no** constructor, Java gives you a free no-argument **default constructor**. As soon as you write any constructor, that free one disappears.

### 3.2 The four pillars, each with a Swiggy example

**1. Encapsulation: hide the data, expose safe methods.**
A Swiggy wallet object keeps `balance` **private**. Outsiders can't write `wallet.balance = 1000000`. They must call `debit(amount)`, which checks the rules: no negative balance, amount > 0. Real life: an ATM. You never touch the cash vault; you use the buttons.

**2. Abstraction: show *what* something does, hide *how*.**
`PaymentService.pay(order)`: the caller doesn't care whether it's UPI, card or wallet underneath. Real life: you drive a car with a steering wheel and pedals, not by controlling fuel injection.
*Encapsulation vs abstraction:* encapsulation is about **protecting data** (private fields). Abstraction is about **hiding complexity** behind a simple interface.

**3. Inheritance: a child class reuses and extends a parent class.**
`Bike extends Vehicle`. The bike gets `regNo` and `describe()` for free and adds `engineCc`. It's an **"is-a"** relationship: a Bike *is a* Vehicle.

**4. Polymorphism: "many forms". One call behaves differently depending on the actual object.**
- **Compile-time (static) polymorphism = method overloading.** Same method name, different parameters. The compiler picks which one to call.
- **Run-time (dynamic) polymorphism = method overriding.** A child redefines a parent method. The JVM picks which version to run **while the program runs**, based on the real object type (this is called **dynamic dispatch**).

```java
import java.util.*;

public class PolymorphismDemo {
    interface Payment { String pay(int amount); }               // abstraction
    static class UpiPayment implements Payment {
        public String pay(int amount) { return "UPI paid " + amount; }
    }
    static class CardPayment implements Payment {
        public String pay(int amount) { return "Card paid " + amount + " (+2% fee)"; }
    }

    static class Wallet {                                        // encapsulation
        private int balance;                                     // hidden
        void credit(int amt) { if (amt <= 0) throw new IllegalArgumentException(); balance += amt; }
        boolean debit(int amt) { if (amt > balance) return false; balance -= amt; return true; }
        int getBalance() { return balance; }
    }

    // Overloading: same name, different parameter lists (compile-time)
    static int total(int a, int b) { return a + b; }
    static int total(int a, int b, int c) { return a + b + c; }
    static double total(double a, double b) { return a + b; }

    public static void main(String[] args) {
        List<Payment> methods = List.of(new UpiPayment(), new CardPayment());
        for (Payment p : methods) System.out.println(p.pay(250));   // run-time polymorphism

        Wallet w = new Wallet();
        w.credit(500);
        System.out.println(w.debit(800) + " " + w.getBalance());   // false 500

        System.out.println(total(1, 2) + " " + total(1, 2, 3) + " " + total(1.5, 2.5));
    }
}
```

### 3.3 Overloading vs overriding

| | Overloading | Overriding |
|---|---|---|
| Where | Same class (or a subclass) | Child class redefines a parent's method |
| Signature | **Different** parameter lists | **Same** name, parameters and (compatible) return type |
| Decided | At compile time | At run time (dynamic dispatch) |
| Example | `total(int,int)` and `total(int,int,int)` | `CardPayment.pay()` overrides `Payment.pay()` |
| Rules | Return type alone can't distinguish overloads | Can't reduce visibility (public → private is not allowed); can't override `static`, `final` or `private` methods |

The `@Override` annotation asks the compiler to check that you really are overriding. If you misspell the method name, you get a compile error instead of a silent bug. Always use it.

**Can you override a static method?** No. If a child declares the same static method, it **hides** the parent's version. Which one runs depends on the *reference type*, not the object. That's why it isn't polymorphism.

### 3.4 Interface vs abstract class (asked in Dec 2025)

- An **abstract class** is a *partly built* class. It can have fields, constructors and normal methods, plus some **abstract** methods (declared with no body) that children must fill in. You can't do `new` on it.
  - Real life: a **franchise kitchen template**. The rent, staff and billing system are already set up; each outlet must decide its own `specialDish()`.
- An **interface** is a pure **contract**: "anything that implements me promises to have these methods." It's about *capability*.
  - Real life: an **FSSAI certification**. Any kitchen can get it, whatever kind of business it is, as long as it meets the listed rules.

```java
public class AbstractVsInterface {
    // Interface: a capability, no state (constants only)
    interface Trackable {
        String location();                       // abstract by default
        default String eta() { return "ETA based on " + location(); }  // Java 8 default method
        static Trackable fixed(String loc) { return () -> loc; }       // static helper
    }
    interface Ratable { double rating(); }

    // Abstract class: shared state + partial behaviour
    static abstract class DeliveryPartner {
        protected final String name;
        protected int completedOrders;
        DeliveryPartner(String name) { this.name = name; }          // abstract classes CAN have constructors
        void completeOrder() { completedOrders++; }                  // concrete shared method
        abstract int maxOrdersAtOnce();                              // each type decides
    }

    // A class extends ONE class but implements MANY interfaces
    static class BikePartner extends DeliveryPartner implements Trackable, Ratable {
        BikePartner(String name) { super(name); }
        int maxOrdersAtOnce() { return 2; }
        public String location() { return "Koramangala"; }
        public double rating() { return 4.7; }
    }

    public static void main(String[] args) {
        BikePartner p = new BikePartner("Ravi");
        p.completeOrder();
        System.out.println(p.name + " " + p.completedOrders + " " + p.maxOrdersAtOnce()
                + " " + p.eta() + " " + p.rating());
        System.out.println(Trackable.fixed("HSR").eta());
    }
}
```

| | Abstract class | Interface |
|---|---|---|
| Methods | Abstract + concrete | Abstract; plus `default` and `static` (Java 8+) and `private` (Java 9+) |
| Fields | Any (instance state allowed) | Only `public static final` constants |
| Constructor | Yes | No |
| Inheritance | `extends` **one** | `implements` **many** |
| Access modifiers | Any | Methods are `public` by default |
| Use when | Related classes share code and state ("is-a" with common base) | Unrelated classes share a capability ("can-do") |

**Why default methods were added (Java 8):** to add new methods to existing interfaces (like `List.sort` or `Collection.stream`) without breaking every class that already implemented them.

**Diamond problem:** a class implements two interfaces with the same default method. It **must override** that method to resolve the clash, and can call a specific one with `InterfaceName.super.method()`.

**Answer to say out loud:** "Use an abstract class when subclasses are closely related and share state or code, like a base `DeliveryPartner`. Use an interface to define a capability that unrelated classes can have, like `Trackable`. A class can implement many interfaces but extend only one class. Since Java 8, interfaces can have default methods, but they still can't hold instance state."

### 3.5 `static`

`static` means "**belongs to the class, not to any one object**."

- A **static field** has one copy shared by all objects. Real life: the **restaurant's total orders today** counter on the wall. Every waiter updates the same counter. A non-static field is each waiter's own notepad.
- A **static method** can be called without an object (`Math.max(a, b)`). It **cannot use `this`** or non-static fields directly, because there's no "current object".
- A **static block** runs once, when the class is first loaded. It's used to initialise static data.
- A **static nested class** is a class inside a class that doesn't need an outer object.

```java
public class StaticDemo {
    static int totalOrders = 0;          // shared by all
    final int orderId;                   // per object

    static { System.out.println("Class loaded once"); }   // static block

    StaticDemo() { totalOrders++; orderId = totalOrders; }

    static int getTotal() { return totalOrders; }          // no 'this' allowed here

    public static void main(String[] args) {
        new StaticDemo(); new StaticDemo();
        StaticDemo third = new StaticDemo();
        System.out.println(third.orderId + " " + StaticDemo.getTotal()); // 3 3
    }
}
```

**Why is `main` static?** The JVM must call `main` before any object exists, so `main` must belong to the class itself.

### 3.6 `final`, `finally`, `finalize` (three different things)

- **`final`** is a keyword meaning "can't be changed":
  - `final` **variable**: assigned once only. A constant: `final int MAX = 5;`.
  - `final` **method**: can't be overridden by a child.
  - `final` **class**: can't be extended. `String` is a final class.
  - **Trap:** a `final` *reference* can't point to another object, but the object itself can still change. `final List<Integer> l = new ArrayList<>(); l.add(1);` is fine. `l = new ArrayList<>();` is a compile error.
- **`finally`** is a block after `try/catch` that **always runs**, exception or not. It's used for cleanup like closing a file or releasing a lock. (It doesn't run only if the JVM itself dies, for example on `System.exit()`.)
- **`finalize()`** was an old method the GC *might* call before destroying an object. It is unreliable, **deprecated since Java 9**, and you shouldn't use it. Use try-with-resources instead (Part 4).

### 3.7 Access modifiers

| Modifier | Same class | Same package | Subclass (other package) | Everyone |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(none)* "package-private" | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

A **package** is a folder of related classes, like `com.swiggy.orders`.

Rule of thumb: make **fields private** and give methods the least access that works.

### 3.8 `equals()` and `hashCode()`: the contract

Every Java class inherits from `Object`, which provides:
- `equals(other)`: by default it means "same object in memory", like `==`.
- `hashCode()`: a number used by HashMap and HashSet to decide which **bucket** an object goes in.

**The contract:** if `a.equals(b)` is true, then `a.hashCode() == b.hashCode()` **must** be true. (The reverse need not hold: two different objects may share a hash code. That's a collision, and it's allowed.)

**What breaks if you override `equals` but not `hashCode`:** two "equal" objects land in different buckets. The HashSet then happily stores duplicates, and `map.get()` can't find your key.

Real life: a library shelves books by the **first letter of the title** (the hash code), then checks the exact title (equals). If two copies of the same book were filed under different letters, the librarian would never find the second one.

```java
import java.util.*;

public class EqualsHashCode {
    static class BadPoint {
        int x, y;
        BadPoint(int x, int y) { this.x = x; this.y = y; }
        @Override public boolean equals(Object o) {
            if (!(o instanceof BadPoint p)) return false;
            return x == p.x && y == p.y;
        }
        // hashCode NOT overridden -> contract broken
    }

    static class GoodPoint {
        final int x, y;
        GoodPoint(int x, int y) { this.x = x; this.y = y; }
        @Override public boolean equals(Object o) {
            if (this == o) return true;
            if (!(o instanceof GoodPoint p)) return false;
            return x == p.x && y == p.y;
        }
        @Override public int hashCode() { return Objects.hash(x, y); }
    }

    // A record (Java 16+) auto-generates equals, hashCode, toString and getters
    record Cell(int r, int c) {}

    public static void main(String[] args) {
        Set<BadPoint> bad = new HashSet<>();
        bad.add(new BadPoint(1, 2)); bad.add(new BadPoint(1, 2));
        System.out.println("bad size = " + bad.size());   // usually 2 (wrong!)

        Set<GoodPoint> good = new HashSet<>();
        good.add(new GoodPoint(1, 2)); good.add(new GoodPoint(1, 2));
        System.out.println("good size = " + good.size()); // 1

        Set<Cell> visited = new HashSet<>();
        visited.add(new Cell(0, 0));
        System.out.println(visited.contains(new Cell(0, 0))); // true; great for grid BFS
    }
}
```

**DSA tip:** use a `record` (`record Cell(int r, int c) {}`) as a HashMap/HashSet key for pairs. It's the cleanest Java way to get a C++-style `pair`.

### 3.9 Immutability, and why String is immutable

An **immutable** object can't change after it is created. To "change" it, you create a new one.

How to make a class immutable:
1. Make the class `final`, so no one can subclass it and add mutable behaviour.
2. Make all fields `private final`.
3. Provide no setters.
4. If a field is a mutable object (like a List), store a **copy** and return copies, so outsiders can't modify it through a reference.

**Why is `String` immutable? Four reasons:**
1. **String pool / memory saving.** Java stores string literals in a shared pool (Part 5). If `"biryani"` is used in 1,000 places, they all share one object. That's only safe if nobody can change it.
2. **Security.** Strings hold file paths, URLs, DB usernames and class names. If a String could change after a security check, an attacker could swap `"/safe/file"` for `"/etc/passwd"` after validation.
3. **Thread safety.** Immutable objects can be shared between threads without locks; nobody can change them.
4. **Caching the hash code.** A String computes its `hashCode` once and caches it. That's why Strings are fast HashMap keys. A mutable key would break the map if it changed.

### 3.10 Composition vs inheritance

- **Inheritance ("is-a"):** `Bike extends Vehicle`.
- **Composition ("has-a"):** an object **holds** other objects as fields and uses them. `Order` *has a* `PaymentMethod`, *has a* `List<Item>`.

**Prefer composition** (classic advice, from the *Effective Java* book):
- Inheritance ties the child tightly to the parent. If the parent changes, children can break (the "fragile base class" problem).
- You can't change a parent at run time, but you *can* swap a component: give an Order a different `DiscountStrategy` object.
- Java allows only one parent class, but you can compose many parts.

Real life: a Swiggy **restaurant** *has a* kitchen, *has* chefs, *has a* menu. It isn't *a* kitchen. Modelling `Restaurant extends Kitchen` would be wrong.

```java
public class CompositionDemo {
    interface DiscountStrategy { int apply(int amount); }
    static class NoDiscount implements DiscountStrategy { public int apply(int a) { return a; } }
    static class FlatOff implements DiscountStrategy {
        private final int off;
        FlatOff(int off) { this.off = off; }
        public int apply(int a) { return Math.max(0, a - off); }
    }

    static class Cart {                          // Cart HAS-A discount strategy
        private DiscountStrategy discount = new NoDiscount();
        private int subtotal;
        void add(int price) { subtotal += price; }
        void setDiscount(DiscountStrategy d) { this.discount = d; }  // swap at run time
        int total() { return discount.apply(subtotal); }
    }

    public static void main(String[] args) {
        Cart cart = new Cart();
        cart.add(300); cart.add(200);
        System.out.println(cart.total());          // 500
        cart.setDiscount(new FlatOff(100));
        System.out.println(cart.total());          // 400
    }
}
```

This is exactly the **Strategy pattern** that the Dec 2025 candidate used for discounts in the food-ordering LLD round. The machine-coding doc covers patterns in depth.

### 3.11 Singleton (asked in an older round)

**Singleton** = a class that allows **exactly one object** in the whole program. Examples: a config manager, a connection pool, a logger. Real life: a country has **one** central bank.

```java
public class SingletonDemo {
    // Thread-safe lazy singleton: "double-checked locking"
    static class ConfigManager {
        private static volatile ConfigManager instance;   // volatile: see Part 7
        private ConfigManager() {}                        // private: no one else can call new
        static ConfigManager getInstance() {
            if (instance == null) {                       // 1st check, no lock (fast path)
                synchronized (ConfigManager.class) {
                    if (instance == null) {               // 2nd check, inside the lock
                        instance = new ConfigManager();
                    }
                }
            }
            return instance;
        }
    }

    // Simplest thread-safe version: an enum
    enum Registry { INSTANCE; int value = 42; }

    public static void main(String[] args) {
        System.out.println(ConfigManager.getInstance() == ConfigManager.getInstance()); // true
        System.out.println(Registry.INSTANCE.value);
    }
}
```

**Why two checks?** Without the outer check, every call takes the lock, which is slow. Without the inner check, two threads could both pass the outer check and each create an object. **Why `volatile`?** It stops another thread from seeing a half-built object (Part 7.4).

---

## Part 4: Exceptions

An **exception** is an event that breaks the normal flow of a program: a file missing, a null object used, dividing by zero. Java turns it into an object that is "thrown" up the call stack until some code "catches" it.

Real life: a waiter finds the kitchen is out of paneer. They don't silently serve nothing. They **raise** it to the manager (throw), who decides what to do (catch): offer a substitute or refund.

### 4.1 The hierarchy

```
Throwable
├── Error            (serious JVM problems: don't catch)
│     ├── OutOfMemoryError
│     └── StackOverflowError
└── Exception
      ├── IOException, SQLException, ...   ← CHECKED
      └── RuntimeException                 ← UNCHECKED
            ├── NullPointerException
            ├── ArrayIndexOutOfBoundsException
            ├── ArithmeticException (divide by zero)
            ├── IllegalArgumentException
            └── ConcurrentModificationException
```

### 4.2 Checked vs unchecked (asked in Dec 2025)

- **Checked exceptions** are checked by the **compiler**. If a method can throw one, you **must** either catch it or declare it with `throws`. They represent problems a good program should expect and handle: the network is down, a file is missing (`IOException`), the DB is unreachable (`SQLException`).
- **Unchecked exceptions** (`RuntimeException` and its children) are not forced by the compiler. They usually mean a **programming bug**: null access, bad index, bad argument. You fix the code rather than catch them.
- **Errors** are JVM-level disasters (out of memory). You normally don't catch them.

Real life:
- Checked = **"the restaurant might be closed"**. It's a normal, expected situation. The app *must* plan for it.
- Unchecked = **"the developer wrote quantity = -3"**. That's a bug to fix, not a situation to plan for.

```java
import java.io.*;

public class ExceptionDemo {
    // Custom checked exception
    static class RestaurantClosedException extends Exception {
        RestaurantClosedException(String msg) { super(msg); }
    }
    // Custom unchecked exception
    static class InvalidQuantityException extends RuntimeException {
        InvalidQuantityException(String msg) { super(msg); }
    }

    static void placeOrder(boolean open, int qty) throws RestaurantClosedException {
        if (qty <= 0) throw new InvalidQuantityException("qty must be > 0, got " + qty);
        if (!open) throw new RestaurantClosedException("Restaurant closed");
        System.out.println("Order placed: " + qty);
    }

    public static void main(String[] args) {
        try {
            placeOrder(true, 2);
            placeOrder(false, 1);
        } catch (RestaurantClosedException e) {        // compiler FORCED us to handle this
            System.out.println("Handled: " + e.getMessage());
        } finally {
            System.out.println("finally always runs");
        }

        try {
            placeOrder(true, -3);
        } catch (InvalidQuantityException | RestaurantClosedException e) {  // multi-catch
            System.out.println("Caught: " + e.getMessage());
        }

        // try-with-resources: auto-closes anything that implements AutoCloseable
        try (BufferedReader br = new BufferedReader(new StringReader("line1"))) {
            System.out.println(br.readLine());
        } catch (IOException e) {
            System.out.println("IO problem");
        }   // br.close() called automatically, even if an exception happened
    }
}
```

### 4.3 try-with-resources

Anything that must be **closed** (files, DB connections, network sockets) should go in `try ( ... )`. Java calls `close()` automatically at the end, in reverse order of opening, even if an exception is thrown. This replaces the old ugly `finally { if (x != null) x.close(); }` code and is why `finalize()` isn't needed.

### 4.4 `throw` vs `throws`

- `throw new X()` **actually throws** an exception object (a statement, inside a method).
- `throws X` **declares**, in the method signature, that this method might throw X.

### 4.5 Good practices to mention

- Catch **specific** exceptions, not `Exception`.
- Never swallow exceptions silently (an empty `catch {}`). At least log them.
- Don't use exceptions for normal control flow; they are slow.
- In Spring, map exceptions to HTTP responses centrally with `@ControllerAdvice` (Part 9.8).

---

## Part 5: Memory and the JVM

### 5.1 Stack vs heap

The JVM uses two main memory areas.

- **Stack:** one per **thread**. Each method call pushes a **frame** holding its local variables and parameters. When the method returns, the frame is popped and gone. It's fast and automatic. It holds **primitives** (`int x = 5`) and **references** (the "address slips").
  - Too-deep recursion → `StackOverflowError`.
- **Heap:** **shared** by all threads. **All objects** live here (`new Order()`, arrays, Strings). The garbage collector cleans it up.
  - Too many live objects → `OutOfMemoryError`.

```java
// illustrative
void process() {
    int count = 3;                   // 'count' (value 3) is on the STACK
    Order o = new Order();           // reference 'o' on the STACK -> Order object on the HEAP
}                                    // frame popped; the Order on the heap is now unreachable -> GC can free it
```

Real life:
- The **stack** is your **desk**. You put papers on it for the current task and clear them when the task is done.
- The **heap** is the **office storeroom**. Big items live there; the desk only holds a note saying which shelf. A cleaner (GC) periodically removes items no note points to any more.

**Other JVM areas you may hear about:**
- **Metaspace** (Java 8+, replacing "PermGen"): stores class metadata (the blueprints themselves).
- **PC register:** per thread, records which instruction the thread is on.

### 5.2 The String pool

When you write a **literal** like `"swiggy"`, Java checks a special area of the heap called the **String pool** (or string constant pool). If `"swiggy"` is already there, it reuses that object. `new String("swiggy")` forces a **new** object outside the pool.

```java
public class StringPool {
    public static void main(String[] args) {
        String a = "swiggy";
        String b = "swiggy";
        String c = new String("swiggy");
        System.out.println(a == b);           // true  (same pooled object)
        System.out.println(a == c);           // false (c is a separate heap object)
        System.out.println(a == c.intern());  // true  (intern() returns the pooled copy)
        System.out.println(a.equals(c));      // true  (always compare with equals)
    }
}
```

**"How many objects does `new String("abc")` create?"** Up to **two**: one `"abc"` in the pool (if it isn't there already), plus one new String object on the heap.

### 5.3 Garbage collection basics

**Garbage collection (GC)** = the JVM automatically finding objects that **no one can reach any more** and freeing their memory.

**"Reachable"** means you can get to the object by following references starting from **GC roots**. GC roots are things that are definitely alive: local variables on any thread's stack, static fields, active threads.

**Mark and sweep (the basic idea):**
1. **Mark:** start from the GC roots and follow every reference, ticking each object you reach as "alive".
2. **Sweep:** everything not ticked is garbage. Free it.
3. **(Compact):** optionally slide the live objects together so free memory is one big block, not scattered holes.

Real life: closing time at a restaurant.
1. The manager walks from the **active order tickets** (roots) to every plate those tickets mention and tags them "keep".
2. Every untagged plate goes to the dishwasher.
3. The tagged plates get pushed together on one counter.

**Generations: why the heap is split.** An observed fact, called the **weak generational hypothesis**, is that **most objects die young**. Temporary Strings, loop objects and request objects are created and forgotten within milliseconds. So the heap is divided:

- **Young generation:** where new objects are born. It has **Eden** plus two small **Survivor** spaces (S0, S1).
  - When Eden fills up, a fast **minor GC** runs. Live objects are copied to a survivor space and everything else is wiped at once. This is cheap because most objects are already dead.
  - Objects that survive several minor GCs get **promoted** to the old generation.
- **Old generation:** long-lived objects (caches, singletons, connection pools). When it fills, a **major/full GC** runs. That's slower.

Real life: a hospital has a **triage ward** (young generation) for walk-ins, most of whom go home quickly, and a **long-term ward** (old generation) for patients who stay. The hospital checks the triage ward often because it's quick, and the long-term ward rarely.

**Stop-the-world pause:** during parts of GC, the JVM pauses your application threads. In a Swiggy API, a long pause shows up as a latency spike (slow p99). Modern collectors keep pauses short:
- **G1** is the default since Java 9. It splits the heap into regions and cleans the most-garbage regions first, aiming for a pause-time target.
- **ZGC** and **Shenandoah** have pauses of a few milliseconds, even on huge heaps.

You can *ask* for GC with `System.gc()`, but it's only a hint. The JVM may ignore it.

### 5.4 Can Java have memory leaks?

Yes. Java memory leaks look different from C++ ones. In C++, a leak is memory you forgot to `delete`. In Java, a leak is an object you **don't need any more but still hold a reference to**. The GC thinks it's alive and never frees it.

Common causes:
1. **Static collections that only grow.** For example, `static Map<String, Session> cache` where nobody ever removes entries. Fix: use eviction (an LRU cache or TTL) or a `WeakHashMap`.
2. **Listeners/callbacks registered and never unregistered.**
3. **Unclosed resources:** connections or streams. Fix: try-with-resources.
4. **ThreadLocal values in thread pools.** Pool threads live forever, so their ThreadLocal values do too, unless you call `remove()`.
5. **Inner-class objects holding a hidden reference** to their big outer object.

Link to your Myntra work: an in-memory cache with no size limit or TTL is the classic leak. That's why caches like Aerospike/Redis use TTLs (your 7-day TTL).

---

## Part 6: Collections internals

### 6.1 How HashMap works (asked in some form in almost every Java interview)

**The setup.** A HashMap is an **array of "buckets"**, 16 by default. Each bucket can hold a small chain of entries `(key, value, hash, next)`.

**`put(key, value)`, step by step:**
1. **Hash the key.** Call `key.hashCode()` to get an int. Java then mixes the high bits into the low bits (`h ^ (h >>> 16)`) so keys that differ only in their high bits still spread out.
2. **Find the bucket.** `index = hash & (capacity - 1)`. Because capacity is always a power of 2, this is a fast way of doing `hash % capacity`.
3. **If the bucket is empty**, put the entry there.
4. **If the bucket already has entries** (a **collision**: two keys landed in the same bucket), walk the chain:
   - If some entry's key `equals()` the new key, **replace its value**.
   - Otherwise, **append** a new entry to the chain.
5. **Resize if needed.** If `size > capacity × loadFactor`, the array **doubles** (16 → 32) and every entry is redistributed.

**`get(key)`:** hash the key, go to the bucket, walk the chain using `equals()` to find the match. With a good spread, chains are short and this is O(1).

**Load factor (default 0.75):** "how full can the map get before it grows?" 16 × 0.75 = 12, so the 13th entry triggers a resize to 32.
- A lower load factor means fewer collisions but more wasted memory.
- A higher one means less memory but longer chains.
- 0.75 is the balance.

**Resize cost:** O(n) for that one put, but it happens rarely (each time the size doubles), so `put` is still **amortised O(1)**. Tip: if you know you'll store a million entries, call `new HashMap<>(capacity)` up front to avoid repeated resizes.

**Treeify (Java 8):** if many keys collide into one bucket, the chain becomes a linked list and look-up degrades to O(n). Java 8 fixed this. When a single bucket's chain reaches **8 entries** (and the table has at least 64 buckets), that bucket's list is converted into a **red-black tree**, so the worst case becomes **O(log n)**. When it shrinks back to 6, it turns back into a list. (If the table is smaller than 64, Java resizes instead of treeifying.)

**Null:** HashMap allows **one null key** (it always goes to bucket 0) and any number of null values.

Real life: a **post office** with numbered pigeonholes.
- The PIN code (hash) picks the pigeonhole (bucket).
- Several letters can share a hole (collision), so the postman checks names (equals).
- If one hole gets crammed (8+ letters), they get a sorted organiser for it (treeify).
- When the office is 75% full, they move to a building with twice the holes (resize).

**Why must keys be immutable?** If you put a key in, then change a field that affects `hashCode`, the map looks in the **wrong bucket** later and can't find it. That's why `String` and `Integer` are perfect keys.

**Time complexity summary:** average O(1) for get/put/remove; worst case O(log n) since Java 8 (O(n) before).

### 6.2 HashMap vs Hashtable vs ConcurrentHashMap

| | HashMap | Hashtable | ConcurrentHashMap |
|---|---|---|---|
| Thread-safe? | ❌ No | ✅ Yes, but **every** method is `synchronized`: one big lock, so slow | ✅ Yes, with fine-grained locking: fast |
| Null keys/values | 1 null key, null values OK | ❌ none | ❌ none |
| Status | Default choice for single-threaded code | Legacy, don't use | Use when multiple threads share a map |

**How ConcurrentHashMap is fast (Java 8+):**
- **Reads take no lock at all.**
- A write locks **only the one bucket** it touches (using `synchronized` on that bucket's first node), or uses a lock-free **CAS** (compare-and-swap) instruction if the bucket is empty.
- So two threads writing to different buckets never block each other. (Before Java 8 it used "segments", about 16 sub-maps each with its own lock.)

**Why no nulls in ConcurrentHashMap?** If `get(k)` returns null, you can't tell "key missing" from "value is null". In single-threaded code you could check `containsKey`, but another thread might change the map between your two calls. So nulls are simply banned.

Real life:
- A `Hashtable` is a library with **one librarian and one door**: everyone queues, even people who only want to read.
- A `ConcurrentHashMap` is a library where **anyone can read freely**, and you only need permission for the **one shelf** you're re-arranging.

**Also know:** `Collections.synchronizedMap(map)` wraps any map with one big lock. It's like Hashtable, and just as slow.

### 6.3 ArrayList vs LinkedList

| Operation | ArrayList (resizable array) | LinkedList (doubly linked nodes) |
|---|---|---|
| `get(i)` | **O(1)** jump straight to index | O(n) walk from the head or tail |
| Add at end | Amortised O(1) | O(1) |
| Insert/remove in the middle | O(n): shift elements | O(1) *once you're at the node*, but finding the node is O(n) |
| Memory | Compact | Each node has 2 extra pointers, so more memory |
| CPU cache | Very friendly (contiguous memory) | Unfriendly (nodes scattered) |

**In practice ArrayList wins almost always.** Even middle inserts are often faster on an ArrayList, because shifting a contiguous array is very fast on modern CPUs. Use `ArrayDeque` (not LinkedList) for queues and stacks.

**How ArrayList grows:** when full, it creates a new array **1.5×** the size and copies the elements over. That's why `add` is amortised O(1).

### 6.4 Fail-fast vs fail-safe iterators

An **iterator** is an object that walks through a collection one element at a time.

**Fail-fast** (HashMap, ArrayList and the other normal collections): each collection keeps a counter called `modCount` that increases on every structural change. The iterator remembers the count when it starts. If the collection was changed by anything other than the iterator itself, the next step throws `ConcurrentModificationException`. That's the collection shouting "someone changed me while you were reading!" rather than silently giving wrong results.

**Fail-safe / weakly consistent** (ConcurrentHashMap, CopyOnWriteArrayList): these iterate over a snapshot, or tolerate changes, and never throw. Their trade-off is that you may not see the very latest changes.

```java
import java.util.*;
import java.util.concurrent.*;

public class FailFastDemo {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>(List.of(1, 2, 3, 4));
        try {
            for (Integer x : list) if (x == 2) list.remove(x);   // modifying while iterating
        } catch (ConcurrentModificationException e) {
            System.out.println("CME thrown");
        }

        // Correct way 1: Iterator.remove()
        Iterator<Integer> it = list.iterator();
        while (it.hasNext()) if (it.next() % 2 == 0) it.remove();
        System.out.println(list);

        // Correct way 2: removeIf
        List<Integer> l2 = new ArrayList<>(List.of(1, 2, 3, 4));
        l2.removeIf(x -> x % 2 == 0);
        System.out.println(l2);

        // Fail-safe collection: no exception
        Map<String, Integer> chm = new ConcurrentHashMap<>(Map.of("a", 1, "b", 2));
        for (String k : chm.keySet()) chm.put(k + "x", 0);    // allowed
        System.out.println(chm.size() >= 4);
    }
}
```

**Interview caveat:** in the first loop above, if you remove the **second-to-last** element, no exception is thrown and the loop simply ends early. That's because `hasNext()` returns false before the check runs. Fail-fast is a "best effort" safety net, not a guarantee.

### 6.5 Comparable vs Comparator

- **Comparable**: the class itself defines its **natural order** by implementing `compareTo(other)`. `String` and `Integer` do this. It's one order, built into the class.
- **Comparator**: a **separate** object that defines an order (`(a, b) -> ...`). You can have many: sort restaurants by rating, by distance, by delivery time.

```java
import java.util.*;

public class CompareDemo {
    static class Restaurant implements Comparable<Restaurant> {
        String name; double rating; int distance;
        Restaurant(String n, double r, int d) { name = n; rating = r; distance = d; }
        public int compareTo(Restaurant o) { return name.compareTo(o.name); }  // natural: by name
        public String toString() { return name; }
    }

    public static void main(String[] args) {
        List<Restaurant> list = new ArrayList<>(List.of(
            new Restaurant("Meghana", 4.5, 5), new Restaurant("Empire", 4.1, 2),
            new Restaurant("Truffles", 4.5, 3)));
        Collections.sort(list);                                       // Comparable
        System.out.println(list);                                     // [Empire, Meghana, Truffles]
        list.sort(Comparator.comparingDouble((Restaurant r) -> r.rating).reversed()
                            .thenComparingInt(r -> r.distance));      // Comparator
        System.out.println(list);                                     // [Truffles, Meghana, Empire]
    }
}
```

---

## Part 7: Multithreading

### 7.1 Words first

- A **process** is a running program with its own memory. Swiggy's order service JVM is one process.
- A **thread** is a path of execution *inside* a process. Threads in the same process **share the heap** but each has its **own stack**.
  - Real life: a restaurant is the **process** and its cooks are the **threads**. They share one kitchen and fridge (the heap), and each has their own cutting board (the stack).
- **Concurrency** means dealing with many tasks *at once* by switching between them. **Parallelism** means *literally* running at the same instant on multiple CPU cores.
- **Why multithreading?** A Swiggy server handles thousands of requests at once. If one request is waiting on the DB, other threads keep working.
- **Race condition:** two threads read and write shared data at the same time, and the result depends on unlucky timing.

### 7.2 Thread vs Runnable vs Callable

- **`Thread`** is the class representing a thread. You *can* extend it, but that wastes your one inheritance slot.
- **`Runnable`** is an interface with `void run()`: "a task to run". You can't return a value or throw a checked exception. **Prefer this over extending Thread** (composition over inheritance again).
- **`Callable<V>`** is an interface with `V call() throws Exception`: a task that **returns a result** and may throw. You submit it to an executor and get back a **`Future<V>`**, an IOU you can redeem later with `future.get()`.

**`start()` vs `run()`:** `start()` creates a **new thread** that then calls `run()`. Calling `run()` directly just runs the code on the **current** thread, with no new thread at all. This is a classic trick question.

```java
import java.util.concurrent.*;

public class ThreadBasics {
    public static void main(String[] args) throws Exception {
        // 1) Extending Thread (not preferred)
        Thread t1 = new Thread() {
            @Override public void run() { System.out.println("t1 on " + Thread.currentThread().getName()); }
        };

        // 2) Runnable (preferred), as a lambda
        Runnable task = () -> System.out.println("runnable on " + Thread.currentThread().getName());
        Thread t2 = new Thread(task, "worker-2");

        t1.start(); t2.start();      // start() -> new thread
        t1.join();  t2.join();       // join(): wait for them to finish

        // 3) Callable returns a value, through a Future
        ExecutorService pool = Executors.newFixedThreadPool(2);
        Callable<Integer> computeBill = () -> 250 + 40;     // items + delivery fee
        Future<Integer> future = pool.submit(computeBill);
        System.out.println("bill = " + future.get());       // blocks until the result is ready
        pool.shutdown();
    }
}
```

**Thread lifecycle (states):**
- `NEW` (created, not started)
- `RUNNABLE` (running or ready to run)
- `BLOCKED` (waiting for a lock)
- `WAITING` / `TIMED_WAITING` (`wait()`, `join()`, `sleep()`)
- `TERMINATED`

### 7.3 ExecutorService and thread pools (connect this to Atlassian)

Creating a thread is **expensive**: the OS must allocate a stack and register it. Creating one per request would melt the server under load. A **thread pool** creates a fixed number of threads **once** and reuses them. Tasks wait in a **queue**, and each free thread picks up the next task.

Real life: Swiggy doesn't hire a new delivery partner for every order. It has a **fleet** of partners (the pool) and a **queue of orders**. When a partner finishes, they take the next order. If orders pile up, they wait in the queue instead of the city being flooded with new hires.

**Your Atlassian worker pool was exactly this.** Messages arrived on AWS SQS (a queue). A fixed set of workers (threads, or coroutines in Kotlin) pulled messages and processed app-publishing requests. A **bounded** pool protects downstream services, and **horizontal scaling** means adding more machines, each with its own pool.

```java
import java.util.*;
import java.util.concurrent.*;

public class PoolDemo {
    public static void main(String[] args) throws Exception {
        ExecutorService workers = Executors.newFixedThreadPool(3);   // 3 reusable threads
        List<Future<String>> results = new ArrayList<>();

        for (int i = 1; i <= 6; i++) {
            final int requestId = i;
            results.add(workers.submit(() -> {
                Thread.sleep(50);                                    // pretend to call an API
                return "request " + requestId + " done by " + Thread.currentThread().getName();
            }));
        }
        for (Future<String> f : results) System.out.println(f.get());

        workers.shutdown();                                          // stop accepting new tasks
        workers.awaitTermination(5, TimeUnit.SECONDS);               // wait for running ones
    }
}
```

**Pool types (from the `Executors` factory):**

| Factory | What you get | Notes |
|---|---|---|
| `newFixedThreadPool(n)` | n threads | Unbounded queue, which can grow forever under load |
| `newCachedThreadPool()` | Creates threads as needed, reuses idle ones | Can create too many threads |
| `newSingleThreadExecutor()` | One thread | Tasks run in order |
| `newScheduledThreadPool(n)` | Runs tasks after a delay or periodically | |
| `newVirtualThreadPerTaskExecutor()` (Java 21) | A lightweight **virtual thread** per task | Cheap like Go goroutines, good for I/O-heavy work |

**Production tip worth saying:** in real services, build a `ThreadPoolExecutor` directly with a **bounded queue** and a **rejection policy**, so that under overload you reject or slow down instead of running out of memory. It's the same idea as your rate limiter: protect the system under high traffic.

**How many threads?** For CPU-heavy work, about the number of cores. For I/O-heavy work (waiting on DB/HTTP), more, because threads spend most of their time waiting.

### 7.4 Race conditions, `synchronized`, `volatile`, atomics, locks

**The bug:** `count++` *looks* like one step but is three: **read** count, **add** 1, **write** it back. Two threads can both read 5, both write 6, and one increment is lost.

Real life: two cashiers update the same paper ledger. Both see "₹500", both add their ₹100 sale, both write "₹600". ₹100 vanished.

**`synchronized`:** only one thread at a time can run a synchronized block for a given **lock object** (the "monitor"). Others wait. Every Java object has a built-in lock.
- `synchronized` on an instance method locks `this`.
- On a static method it locks the class object.
- You can also write `synchronized (someObject) { ... }` to lock just a block. Smaller blocks mean less waiting.

**`volatile`:** solves **visibility**, not atomicity.
- Each CPU core may keep a **cached copy** of a variable. Without `volatile`, one thread may set `running = false` and another thread may keep reading its stale cached `true` forever.
- `volatile` guarantees every read sees the latest write, and it stops the compiler/CPU from **reordering** instructions around it.
- But `volatile int count; count++;` is **still broken**: the read-add-write steps can still interleave.
- Use `volatile` for simple **flags** written by one thread and read by others (like a "stop" flag), and in double-checked locking (3.11).

**Atomic classes** (`AtomicInteger`, `AtomicLong`, `AtomicReference`): lock-free, thread-safe operations like `incrementAndGet()`. They use the CPU's **CAS (compare-and-swap)** instruction: "set the value to 6 *only if* it's still 5; otherwise retry." This is faster than locking for simple counters.

**`ReentrantLock`:** an explicit lock with extras that `synchronized` doesn't have:
- `tryLock()` (give up if the lock is busy, which helps avoid deadlock)
- timeouts
- fairness
- multiple wait conditions

You **must** unlock in `finally`. "**Reentrant**" means the same thread can take the lock again without deadlocking itself. (`synchronized` is reentrant too.)

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;
import java.util.concurrent.locks.*;

public class SyncDemo {
    static int unsafeCount = 0;
    static int syncCount = 0;
    static final Object lock = new Object();
    static AtomicInteger atomicCount = new AtomicInteger();
    static int lockCount = 0;
    static final ReentrantLock rlock = new ReentrantLock();
    static volatile boolean running = true;   // visibility flag

    public static void main(String[] args) throws Exception {
        ExecutorService pool = Executors.newFixedThreadPool(4);
        for (int t = 0; t < 4; t++) {
            pool.submit(() -> {
                for (int i = 0; i < 100_000; i++) {
                    unsafeCount++;                            // race condition
                    synchronized (lock) { syncCount++; }      // safe: one thread at a time
                    atomicCount.incrementAndGet();            // safe: CAS, lock-free
                    rlock.lock();
                    try { lockCount++; } finally { rlock.unlock(); }  // safe: explicit lock
                }
            });
        }
        pool.shutdown();
        pool.awaitTermination(10, TimeUnit.SECONDS);
        System.out.println("unsafe=" + unsafeCount + " (usually < 400000)");
        System.out.println("sync=" + syncCount + " atomic=" + atomicCount.get() + " lock=" + lockCount);

        // volatile flag: the background thread sees the update and stops
        Thread bg = new Thread(() -> { while (running) { /* busy work */ } System.out.println("bg stopped"); });
        bg.start();
        Thread.sleep(50);
        running = false;
        bg.join();
    }
}
```

| Tool | Solves | Use for |
|---|---|---|
| `volatile` | Visibility (and ordering) | Stop flags, status flags, double-checked locking |
| `AtomicInteger` | Atomic single-variable updates | Counters, sequence numbers |
| `synchronized` | Mutual exclusion + visibility | Simple critical sections |
| `ReentrantLock` | The same + tryLock, timeouts, fairness | Complex locking, avoiding deadlocks |
| `ConcurrentHashMap` etc. | Thread-safe data structures | Shared maps/queues |

### 7.5 Deadlock in Java

**Deadlock** = two or more threads each hold a lock and wait forever for a lock the other holds.

Real life: two delivery partners on a one-lane bridge, facing each other. Each waits for the other to reverse. Nobody moves, ever.

All **four conditions** (Coffman conditions) must hold for deadlock:
1. **Mutual exclusion:** a resource is held by only one thread.
2. **Hold and wait:** a thread holds one lock while waiting for another.
3. **No preemption:** you can't forcibly take a lock away.
4. **Circular wait:** T1 waits for T2, and T2 waits for T1.

Break **any one** of them and deadlock can't happen. The easiest in code is to break circular wait: **always acquire locks in the same global order**.

```java
import java.util.concurrent.*;
import java.util.concurrent.locks.*;

public class DeadlockDemo {
    static final ReentrantLock walletLock = new ReentrantLock();
    static final ReentrantLock orderLock  = new ReentrantLock();

    // BAD: thread A takes wallet->order, thread B takes order->wallet => possible deadlock.
    // We demo it safely with tryLock + timeout so the program can still finish.
    static void lockBoth(ReentrantLock first, ReentrantLock second, String name) {
        try {
            if (first.tryLock(100, TimeUnit.MILLISECONDS)) {
                try {
                    Thread.sleep(50);   // make the bad interleaving likely
                    if (second.tryLock(100, TimeUnit.MILLISECONDS)) {
                        try { System.out.println(name + " got both locks"); }
                        finally { second.unlock(); }
                    } else {
                        System.out.println(name + " gave up: would have DEADLOCKED");
                    }
                } finally { first.unlock(); }
            }
        } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }

    public static void main(String[] args) throws Exception {
        Thread a = new Thread(() -> lockBoth(walletLock, orderLock, "A"));
        Thread b = new Thread(() -> lockBoth(orderLock, walletLock, "B"));   // opposite order
        a.start(); b.start(); a.join(); b.join();

        // FIX: same lock order everywhere (wallet always before order)
        Thread c = new Thread(() -> lockBoth(walletLock, orderLock, "C"));
        Thread d = new Thread(() -> lockBoth(walletLock, orderLock, "D"));
        c.start(); d.start(); c.join(); d.join();
    }
}
```

With plain `synchronized` blocks in the opposite order, threads A and B would **hang forever**. The demo uses `tryLock` with a timeout so it can show the problem and still exit. Using a timeout is itself a fix: it breaks "hold and wait / no preemption".

**How to detect deadlock in a live Java app:** take a **thread dump** (`jstack <pid>`). The JVM prints "Found one Java-level deadlock" along with the threads and locks involved.

### 7.6 wait / notify and producer–consumer with BlockingQueue

**The producer–consumer problem:**
- **Producers** create work and put it in a shared buffer. Customers placing orders are producers.
- **Consumers** take work out. Kitchen staff cooking the orders are consumers.
- Rules: producers must wait if the buffer is **full**; consumers must wait if it's **empty**.

**Low-level way: `wait()` / `notify()`.**
- They are called inside `synchronized` on the shared object.
- `wait()` releases the lock and sleeps until notified.
- `notify()` wakes one waiting thread; `notifyAll()` wakes all of them.
- Always call `wait()` inside a **`while` loop** that re-checks the condition, never an `if`. Threads can wake up "spuriously" (for no reason), or another consumer may grab the item first.

```java
import java.util.*;

public class WaitNotifyDemo {
    static class OrderBuffer {
        private final Queue<Integer> q = new LinkedList<>();
        private final int capacity;
        OrderBuffer(int capacity) { this.capacity = capacity; }

        synchronized void put(int order) throws InterruptedException {
            while (q.size() == capacity) wait();     // full -> wait (releases the lock)
            q.add(order);
            notifyAll();                             // wake consumers
        }
        synchronized int take() throws InterruptedException {
            while (q.isEmpty()) wait();              // empty -> wait
            int o = q.poll();
            notifyAll();                             // wake producers
            return o;
        }
    }

    public static void main(String[] args) throws Exception {
        OrderBuffer buf = new OrderBuffer(2);
        Thread producer = new Thread(() -> {
            try { for (int i = 1; i <= 5; i++) buf.put(i); } catch (InterruptedException ignored) {}
        });
        Thread consumer = new Thread(() -> {
            try { for (int i = 1; i <= 5; i++) System.out.println("cooked order " + buf.take()); }
            catch (InterruptedException ignored) {}
        });
        producer.start(); consumer.start(); producer.join(); consumer.join();
    }
}
```

**Modern way: `BlockingQueue`.**
- `put()` blocks when the queue is full; `take()` blocks when it's empty.
- All the locking is handled for you. In real code, use this.
- This is the **in-process version of SQS/Kafka**: a queue that decouples who makes the work from who does it.

```java
import java.util.concurrent.*;

public class BlockingQueueDemo {
    public static void main(String[] args) throws Exception {
        BlockingQueue<String> orders = new ArrayBlockingQueue<>(3);   // bounded buffer
        ExecutorService pool = Executors.newFixedThreadPool(3);

        pool.submit(() -> {                                    // producer
            for (int i = 1; i <= 6; i++) orders.put("order-" + i);
            orders.put("STOP"); orders.put("STOP");            // "poison pills" to stop consumers
            return null;
        });
        for (int c = 1; c <= 2; c++) {                          // 2 consumers = a worker pool
            final int id = c;
            pool.submit(() -> {
                while (true) {
                    String o = orders.take();
                    if (o.equals("STOP")) break;
                    System.out.println("worker " + id + " processed " + o);
                }
                return null;
            });
        }
        pool.shutdown();
        pool.awaitTermination(5, TimeUnit.SECONDS);
    }
}
```

**`sleep()` vs `wait()` (common question):**
- `Thread.sleep()` pauses but **keeps** any locks held. It's a method of `Thread`.
- `wait()` **releases** the lock and must be called inside `synchronized`. It's a method of `Object`, because every object can be a lock.

### 7.7 CompletableFuture (briefly)

`Future.get()` blocks your thread while waiting. **`CompletableFuture`** lets you say "when this finishes, then do that" without blocking. You chain steps and combine parallel calls.

Real life, the Swiggy checkout page: fetch **cart**, **offers** and **delivery ETA** at the same time, then combine them when all three are ready.

```java
import java.util.concurrent.*;

public class CompletableFutureDemo {
    static int sleepAndReturn(int ms, int value) {
        try { Thread.sleep(ms); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        return value;
    }

    public static void main(String[] args) {
        CompletableFuture<Integer> cart   = CompletableFuture.supplyAsync(() -> sleepAndReturn(100, 500));
        CompletableFuture<Integer> offer  = CompletableFuture.supplyAsync(() -> sleepAndReturn(80, 50));
        CompletableFuture<Integer> etaMin = CompletableFuture.supplyAsync(() -> sleepAndReturn(60, 28));

        CompletableFuture<String> page = cart
            .thenCombine(offer, (c, o) -> c - o)                     // combine two results
            .thenCombine(etaMin, (total, eta) -> "Pay ₹" + total + ", arrives in " + eta + " min")
            .exceptionally(ex -> "Something failed: " + ex.getMessage());   // fallback on error

        System.out.println(page.join());   // total time ≈ 100 ms (parallel), not 240 ms
    }
}
```

Other useful methods:
- `thenApply` transforms a result.
- `thenCompose` chains another async call.
- `allOf` waits for many futures.

### 7.8 Other concurrency tools (name-drop level)

- **`CountDownLatch`**: wait until N events have happened. Example: start the app only after 3 caches have warmed up.
- **`Semaphore`**: allow at most N threads into a section at once. Real life: a parking lot with N spots. It's useful to limit concurrent calls to a downstream service, a cousin of your rate limiter.
- **`ThreadLocal`**: each thread gets its own copy of a variable. Used for request IDs and DB connections per thread.
- **Virtual threads (Java 21):** very cheap threads managed by the JVM, similar to Go's goroutines (which you used at Myntra). You can have millions of them. They're great for I/O-heavy services.

---

## Part 8: Java 8+ features

### 8.1 Functional interfaces and lambdas

- A **functional interface** has exactly **one abstract method**. Examples: `Runnable` (`run`), `Comparator` (`compare`), `Callable` (`call`).
- A **lambda** is a short way to create an object of a functional interface without writing a whole class: `(a, b) -> a + b`.
- **`@FunctionalInterface`** makes the compiler check that there's only one abstract method.

Built-in ones in `java.util.function`, worth knowing by name:

| Interface | Shape | Example |
|---|---|---|
| `Function<T,R>` | takes T, returns R | `s -> s.length()` |
| `Predicate<T>` | takes T, returns boolean | `o -> o.amount() > 500` |
| `Consumer<T>` | takes T, returns nothing | `o -> System.out.println(o)` |
| `Supplier<T>` | takes nothing, returns T | `() -> new ArrayList<>()` |
| `BiFunction<T,U,R>` | takes T and U, returns R | `(a, b) -> a * b` |

A **method reference** (`String::length`, `System.out::println`) is an even shorter lambda that just calls an existing method.

### 8.2 Streams

A **stream** is a pipeline for processing a collection: **source → intermediate operations → terminal operation**.
- Intermediate ops (`filter`, `map`, `sorted`) are **lazy**: nothing runs until a terminal op (`collect`, `sum`, `count`, `forEach`) is called.
- Streams **don't modify** the original collection.

Real life: an **assembly line**. Orders go in, a filter removes cancelled ones, a mapper pulls out the amount, a summer adds them up.

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

public class StreamDemo {
    record Order(String id, String city, int amount, boolean delivered) {}

    @FunctionalInterface
    interface DeliveryFee { int compute(int distanceKm); }

    public static void main(String[] args) {
        List<Order> orders = List.of(
            new Order("o1", "BLR", 450, true), new Order("o2", "DEL", 1200, true),
            new Order("o3", "BLR", 800, false), new Order("o4", "BLR", 300, true));

        int blrRevenue = orders.stream()
            .filter(o -> o.city().equals("BLR"))          // keep BLR
            .filter(Order::delivered)                      // keep delivered
            .mapToInt(Order::amount)                       // to int stream
            .sum();
        System.out.println(blrRevenue);                    // 750

        Map<String, Long> countByCity = orders.stream()
            .collect(Collectors.groupingBy(Order::city, Collectors.counting()));
        System.out.println(countByCity);                   // e.g. {DEL=1, BLR=3}

        List<String> bigIds = orders.stream()
            .filter(o -> o.amount() > 400)
            .sorted(Comparator.comparingInt(Order::amount).reversed())
            .map(Order::id)
            .collect(Collectors.toList());
        System.out.println(bigIds);                        // [o2, o3, o1]

        // Functional interfaces
        Predicate<Order> expensive = o -> o.amount() > 1000;
        Function<Order, String> label = o -> o.id() + ":" + o.amount();
        Supplier<List<String>> fresh = ArrayList::new;
        DeliveryFee fee = km -> km <= 3 ? 20 : 20 + (km - 3) * 8;
        List<String> labels = fresh.get();
        orders.stream().filter(expensive).map(label).forEach(labels::add);
        System.out.println(labels + " fee(5km)=" + fee.compute(5));
    }
}
```

**In DSA rounds, avoid streams in hot loops.** They're slower than plain loops and harder to debug. They're great for clean business logic in LLD rounds.

### 8.3 Optional

`Optional<T>` is a box that **may or may not contain a value**. It's a polite replacement for returning `null`. It forces the caller to think about the "missing" case, which avoids NullPointerExceptions.

```java
import java.util.*;

public class OptionalDemo {
    static Map<String, String> users = Map.of("u1", "Vaideeka");

    static Optional<String> findUser(String id) { return Optional.ofNullable(users.get(id)); }

    public static void main(String[] args) {
        System.out.println(findUser("u1").map(String::toUpperCase).orElse("GUEST"));  // VAIDEEKA
        System.out.println(findUser("u9").orElse("GUEST"));                           // GUEST
        findUser("u1").ifPresent(n -> System.out.println("Hello " + n));
        System.out.println(findUser("u9").isPresent());                              // false
    }
}
```

Use Optional for **return values**. Don't use it for fields or method parameters.

### 8.4 Other modern bits worth knowing

- **`var`** (Java 10): local type inference. `var map = new HashMap<String, Integer>();`.
- **Records** (Java 16): one-line immutable data classes (see 3.8).
- **Switch expressions** (Java 14): `int fee = switch (zone) { case "A" -> 20; default -> 40; };`.
- **Text blocks** (Java 15): multi-line strings with `"""`.
- **Virtual threads** (Java 21): see 7.8.

---

## Part 9: Spring Boot from zero (linked to your Atlassian work)

At Atlassian you built the app-publishing service in **Kotlin + Spring Boot**. Interviewers will assume you know *why* Spring does things, not just that you added annotations. Kotlin and Java Spring work the same way; only the syntax differs.

### 9.1 What is a framework?

A **library** is code *you* call: `Math.max()`. A **framework** is code that **calls you**. It runs the main program, and you plug your pieces into slots it defines. This is called **Inversion of Control**: control of the program flow is inverted, moving from your code to the framework's.

Real life: cooking at home with your own utensils is using a **library**: you're in charge. Working as a chef in a **restaurant chain kitchen** is using a **framework**: the kitchen decides when orders arrive, what station you're at and how plates go out. You just cook your dish when called.

**Spring** is a Java framework for building applications, especially backends. **Spring Boot** is Spring with:
- sensible **defaults** and **auto-configuration**
- an **embedded web server** (Tomcat), so you run one `.jar` and the server starts
- **starter** dependencies (`spring-boot-starter-web` pulls in everything for REST APIs)

With plain Spring you had to write lots of XML/config; Spring Boot removes most of that.

### 9.2 IoC and dependency injection (the core idea)

A **dependency** is an object your class needs to do its job. For example, `OrderService` needs a `PaymentClient` and an `OrderRepository`.

**Without DI**, the class creates its own dependencies:

```java
// illustrative
class OrderService {
    private PaymentClient payment = new RazorpayClient("hardcoded-key");  // tightly glued
}
```

This is bad for three reasons:
- You can't swap in a fake client for tests.
- You can't change the implementation without editing this class.
- Every class builds its own copies.

**With dependency injection (DI)**, the class just **declares** what it needs, and someone else **gives (injects)** it:

```java
// illustrative
@Service
class OrderService {
    private final PaymentClient payment;
    private final OrderRepository repo;

    OrderService(PaymentClient payment, OrderRepository repo) {   // constructor injection
        this.payment = payment;
        this.repo = repo;
    }
}
```

The "someone else" is the **Spring IoC container**, also called the **ApplicationContext**. At start-up, Spring:
1. finds all your classes marked as components,
2. creates one object of each,
3. works out who needs what, and
4. passes the right objects into constructors.

Real life: a **restaurant manager** (the container). A chef doesn't go out and buy their own knives and stove. They just say "I need a knife and a stove", and the manager provides them when the chef is hired. To test the chef, the manager can hand over a practice knife (a mock).

**Three kinds of injection:**
1. **Constructor injection** (✅ recommended):
   - dependencies can be `final`
   - the object is never half-built
   - it's easy to test (just call `new OrderService(mockPayment, mockRepo)`)
   - circular dependencies fail loudly at start-up
2. **Setter injection:** for optional dependencies.
3. **Field injection** (`@Autowired` on a private field): short, but hides dependencies and is hard to test. Avoid it.

**`@Autowired`** tells Spring "inject this". On a class with a **single constructor**, Spring injects automatically, so you don't even need the annotation.

**If two beans match one type** (two `PaymentClient` implementations), Spring can't choose and fails at start-up. Fix it with:
- `@Primary` on the default one, or
- `@Qualifier("upiClient")` at the injection point.

### 9.3 Beans and scopes

A **bean** is simply **an object that Spring creates and manages** inside its container. Your `@Service` classes, controllers and repositories are all beans.

**Two ways to create beans:**
- **Annotate the class** (`@Component`, `@Service`...). Spring finds it through **component scanning** (it scans your package for annotated classes).
- **Write a `@Bean` method inside a `@Configuration` class.** Use this for objects from libraries you can't annotate, like an `SqsClient` or a `RestTemplate`.

```java
// illustrative
@Configuration
public class AwsConfig {
    @Bean
    public SqsClient sqsClient() {
        return SqsClient.builder().region(Region.US_EAST_1).build();
    }
}
```

**Bean scopes** decide how many objects are created:

| Scope | Meaning | Example |
|---|---|---|
| **singleton** (default) | **One** instance for the whole container, shared by everyone | Services, repositories, clients |
| **prototype** | A **new** instance every time it's requested | A stateful helper object |
| **request** (web) | One per HTTP request | Per-request context |
| **session** (web) | One per user HTTP session | A shopping cart in old-style web apps |

**Important consequence:** singleton beans are shared across **all request threads**. So a singleton service **must be stateless** or thread-safe. Never store request data in a field of a `@Service`; two concurrent requests would overwrite each other. This ties back to Part 7.

### 9.4 The bean lifecycle (asked in Dec 2025)

The order of what Spring does with each bean:

1. **Instantiate**: call the constructor (constructor injection happens here).
2. **Populate properties**: setter or field injection.
3. **Aware callbacks**: if the bean implements `BeanNameAware` or `ApplicationContextAware`, Spring tells it its name or context. Rarely used.
4. **`BeanPostProcessor` before-initialisation**: framework hooks run.
5. **Initialise**: your `@PostConstruct` method runs (or `afterPropertiesSet()` from `InitializingBean`, or a custom `initMethod`). Use it for one-time setup after dependencies are injected: warm a cache, validate config, open a connection pool.
6. **`BeanPostProcessor` after-initialisation**: this is where Spring may wrap your bean in a **proxy** to add features like `@Transactional`, `@Async` and AOP logging.
7. **Ready**: the bean is used by the app.
8. **Destroy** (on shutdown): your `@PreDestroy` method runs (or `destroy()` from `DisposableBean`). Use it to close connections, flush buffers and stop worker threads gracefully.

```java
// illustrative
@Service
public class PublishWorkerPool {
    private ExecutorService workers;

    @PostConstruct
    void start() {                       // after injection: start the SQS polling workers
        workers = Executors.newFixedThreadPool(8);
    }

    @PreDestroy
    void stop() throws InterruptedException {   // on shutdown: let in-flight messages finish
        workers.shutdown();
        workers.awaitTermination(30, TimeUnit.SECONDS);
    }
}
```

Real life: hiring a new delivery partner.
1. Hire them (instantiate).
2. Give them a bike and phone (inject).
3. Run orientation (`@PostConstruct`).
4. They deliver orders (ready).
5. On their last day, they return the equipment and finish current deliveries (`@PreDestroy`).

> **Interview answer (30 sec):** "Spring instantiates the bean, injects dependencies, runs aware callbacks and BeanPostProcessors, then calls the init method like `@PostConstruct`. After-init post-processors may wrap it in a proxy for things like `@Transactional`. Then it's in use. On context shutdown, `@PreDestroy` runs for cleanup. Singletons live as long as the container. For prototype beans, Spring doesn't call the destroy callbacks."

### 9.5 The stereotype annotations

| Annotation | Meaning |
|---|---|
| `@Component` | Generic "this is a Spring bean" |
| `@Service` | A bean holding **business logic**. Same as `@Component`; the name documents intent |
| `@Repository` | A bean for **data access** (DB). Also translates DB-specific exceptions into Spring's `DataAccessException` |
| `@Controller` | A web controller whose methods return **view names** (HTML pages via a template engine) |
| `@RestController` | `@Controller` + `@ResponseBody`: return values are written **directly into the HTTP response body as JSON**. Used for REST APIs |

**`@Controller` vs `@RestController` (asked in Dec 2025):**
- With `@Controller`, returning `"orderPage"` means "render the template called orderPage.html".
- With `@RestController`, returning an `Order` object means "convert it to JSON and send it back". The conversion is done by the **Jackson** library through an `HttpMessageConverter`.
- Backend APIs (like yours at Atlassian) use `@RestController`.

### 9.6 How Spring Boot manages configuration (asked in Dec 2025)

**Configuration** = values that change between environments without changing code: DB URLs, queue names, timeouts, feature flags, API keys.

**Where config lives:** `src/main/resources/application.properties` or `application.yml`:

```yaml
# application.yml
server:
  port: 8080
app:
  publish:
    queue-name: marketplace-publish-requests
    worker-count: 8
    max-retries: 3
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/marketplace
```

**Profiles** let you keep a separate config per environment:
- `application-dev.yml`, `application-staging.yml`, `application-prod.yml`
- Activate one with `spring.profiles.active=prod`, using an env var `SPRING_PROFILES_ACTIVE=prod` or a command-line flag.
- The profile file **overrides** the base `application.yml`.
- Link to Myntra: your **production-parity staging environment** is exactly what profiles and per-environment config enable. The same code runs with stage DB, Kafka and Aerospike endpoints.

**Reading config in code:**

```java
// illustrative
@Service
class PublishService {
    @Value("${app.publish.max-retries:3}")     // single value, with a default of 3
    private int maxRetries;
}

@ConfigurationProperties(prefix = "app.publish")   // a whole group, type-safe
record PublishProperties(String queueName, int workerCount, int maxRetries) {}
```

`@ConfigurationProperties` is preferred for groups of settings:
- it's type-safe
- it can be validated
- `queue-name` maps to `queueName` automatically (this is called "relaxed binding")

**Priority order (higher wins), simplified:**
1. Command-line arguments (`--server.port=9090`)
2. OS environment variables (`SERVER_PORT=9090`)
3. Profile-specific files (`application-prod.yml`)
4. `application.yml` / `application.properties`
5. Defaults in code (`@Value("${x:default}")`)

This order lets the same `.jar` run everywhere. In Kubernetes or AWS, you inject env vars or mounted config, and **secrets** (passwords, API keys) come from a secret manager (AWS Secrets Manager, Vault), never from Git.

### 9.7 Auto-configuration

**Auto-configuration** is Spring Boot's "smart defaults". When your app starts, Boot looks at **what's on the classpath** (which libraries you included) and **which beans you've already defined**, then configures the rest for you.

Examples:
- `spring-boot-starter-web` is on the classpath, so Boot starts an embedded Tomcat on port 8080 and sets up JSON conversion with Jackson.
- A MySQL driver is present and `spring.datasource.url` is set, so Boot creates a `DataSource` with a **HikariCP connection pool**.
- If you define your own `DataSource` bean, **yours wins**. Boot's auto-config "backs off".

How it works:
- `@SpringBootApplication` = `@Configuration` + `@ComponentScan` + `@EnableAutoConfiguration`.
- The auto-config classes use conditions like `@ConditionalOnClass` ("only if this library exists") and `@ConditionalOnMissingBean` ("only if the user didn't define one").

Real life: a **hotel room that sets itself up** based on what you packed. You brought a laptop, so the room puts out a desk and power strip. If you brought your own lamp, it doesn't put out a second one.

### 9.8 A request's journey through a REST controller

What happens when a client calls `POST /api/v1/apps/publish`:

1. **Network → embedded Tomcat.** The request arrives on port 8080. Tomcat takes a thread from its **thread pool** (200 threads by default) to handle it.
2. **Filters.** The servlet filter chain runs first: logging, CORS, and **Spring Security**, which checks the JWT or auth token. A filter can reject the request here (401).
3. **DispatcherServlet.** Spring's single "front controller". It receives every request and routes it.
4. **Handler mapping.** It finds which controller method matches `POST /api/v1/apps/publish`.
5. **Interceptors** (`HandlerInterceptor.preHandle`), if any: for example, adding a request ID.
6. **Argument binding and validation.**
   - Jackson converts the JSON body into a Java object (`@RequestBody`).
   - `@PathVariable` and `@RequestParam` are filled in.
   - `@Valid` triggers validation rules like `@NotBlank` and `@Size`. If they fail, the response is 400.
   - Link to Atlassian: your **automated request validation** happened at this step.
7. **Controller method runs** and calls the **service** layer (business logic). The service calls the **repository** (DB) or clients (SQS).
8. **Return value → response.**
   - The object is converted back to JSON.
   - The status code is set (`ResponseEntity.status(202)`, for example).
   - Headers are written.
9. **Exceptions.** If anything throws, `@ControllerAdvice` (next section) turns it into a clean error response.
10. The response goes back through the filters to the client, and the Tomcat thread returns to the pool.

```java
// illustrative: the Atlassian flow in Spring terms
@RestController
@RequestMapping("/api/v1/apps")
public class PublishController {
    private final PublishService service;
    public PublishController(PublishService service) { this.service = service; }

    @PostMapping("/publish")
    public ResponseEntity<PublishResponse> publish(@Valid @RequestBody PublishRequest req) {
        String requestId = service.enqueue(req);          // validate + push to SQS, don't wait
        return ResponseEntity.accepted()                  // 202 Accepted: "got it, processing async"
                             .body(new PublishResponse(requestId, "QUEUED"));
    }

    @GetMapping("/publish/{requestId}")
    public PublishStatus status(@PathVariable String requestId) {
        return service.status(requestId);                 // client polls for the result
    }
}
```

Why **202 Accepted**? The work happens later, in the SQS worker pool. So the API says "accepted for processing" instead of 200 "done". That's the "don't make the caller wait" idea from your Atlassian internship.

### 9.9 Exception handling with `@ControllerAdvice`

Without it, every controller would need try/catch blocks, and error responses would be inconsistent. **`@RestControllerAdvice`** is one global class that catches exceptions from **all** controllers and turns each into a proper HTTP response.

```java
// illustrative
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(AppNotFoundException.class)
    public ResponseEntity<ErrorBody> notFound(AppNotFoundException ex) {
        return ResponseEntity.status(404).body(new ErrorBody("APP_NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)      // thrown when @Valid fails
    public ResponseEntity<ErrorBody> badRequest(MethodArgumentNotValidException ex) {
        return ResponseEntity.badRequest().body(new ErrorBody("VALIDATION_FAILED", "Invalid fields"));
    }

    @ExceptionHandler(Exception.class)                            // safety net
    public ResponseEntity<ErrorBody> fallback(Exception ex) {
        // log the full stack trace internally; don't leak it to clients
        return ResponseEntity.status(500).body(new ErrorBody("INTERNAL_ERROR", "Something went wrong"));
    }
}
```

Real life: a restaurant's **customer-complaints desk**. Waiters don't each invent their own apology. Every complaint goes to one desk, which responds in a consistent, polite way.

### 9.10 A few more Spring terms you may hear

- **`@Transactional`**: wraps a method in a DB transaction, so all its DB writes succeed or all roll back. It works through a **proxy** (lifecycle step 6). Gotcha: calling a `@Transactional` method from **another method in the same class** skips the proxy, so no transaction is applied.
- **Spring Data JPA**: you write an interface like `interface OrderRepo extends JpaRepository<Order, Long>`, and Spring generates the SQL code. Method names like `findByCityAndStatus` become queries.
- **Actuator**: built-in endpoints like `/actuator/health` and `/actuator/metrics` for health checks and monitoring. Kubernetes uses health endpoints for liveness and readiness probes.
- **AOP (Aspect-Oriented Programming)**: adds cross-cutting behaviour (logging, timing, security) around methods without touching their code.

---

## Part 10: Rapid-fire bank (50 questions, in interview order)

Practise answering each in **under 20 seconds**, out loud. The 2026 round packed 20–25 of these into about 25 minutes.

**JVM / basics**

1. **JDK vs JRE vs JVM?** The JVM runs bytecode. The JRE is the JVM plus the standard libraries, for running programs. The JDK is the JRE plus dev tools like `javac`, for building programs.
2. **Why is Java platform-independent?** `javac` produces bytecode, not machine code. Each OS has its own JVM that runs the same bytecode.
3. **What is the JIT compiler?** Part of the JVM that compiles frequently run ("hot") bytecode into native machine code at run time, so long-running Java is fast.
4. **Is Java pass-by-value or pass-by-reference?** Always pass-by-value. For objects, the value copied is the reference, so a method can mutate the object but can't re-point the caller's variable.
5. **Why is `main` static?** The JVM must call it before any object exists.
6. **Stack vs heap?** The stack is per thread and holds method frames, local primitives and references; it's freed automatically on return. The heap is shared and holds all objects; it's cleaned by the GC.
7. **What causes `StackOverflowError` vs `OutOfMemoryError`?** Too-deep (usually infinite) recursion fills the stack. Too many live objects fill the heap.

**OOP**

8. **The four pillars?** Encapsulation (private data, public methods), abstraction (hide how, show what), inheritance (is-a reuse), polymorphism (one interface, many behaviours).
9. **Overloading vs overriding?** Overloading: same name, different parameters, chosen at compile time. Overriding: a subclass redefines a method with the same signature, chosen at run time by dynamic dispatch.
10. **Interface vs abstract class?** An abstract class can have state, constructors and concrete methods, and you extend only one. An interface is a contract with no instance state (default/static methods allowed), and you can implement many. Abstract class for related classes sharing code; interface for a capability.
11. **Can an interface have method bodies?** Yes, since Java 8: `default` and `static` methods, and `private` ones since Java 9.
12. **Why no multiple inheritance of classes?** To avoid the diamond problem: ambiguity when two parents have the same method or state. Interfaces give multiple *type* inheritance, and default-method conflicts must be resolved explicitly.
13. **Can we override static or private methods?** No. Static methods are hidden, not overridden. Private methods aren't visible to subclasses.
14. **`static` keyword?** Belongs to the class, not to instances: one shared copy of the field, and methods callable without an object that can't use `this`.
15. **`final` vs `finally` vs `finalize`?** `final` means can't reassign, override or extend. `finally` always runs after try/catch, for cleanup. `finalize` is a deprecated GC hook; don't use it.
16. **Is a `final` List immutable?** No. The reference can't change, but you can still add and remove elements. Use `List.of()` or `Collections.unmodifiableList()` for that.
17. **Composition vs inheritance?** Inheritance is "is-a" and tightly coupled. Composition is "has-a" and lets you swap parts at run time. Prefer composition (example: the Strategy pattern for discounts).
18. **Why is String immutable?** String-pool sharing, security (paths and credentials can't change after checks), thread safety, and a cached hashCode that makes it a good HashMap key.
19. **String vs StringBuilder vs StringBuffer?** String is immutable. StringBuilder is mutable and fast but not thread-safe. StringBuffer is mutable and synchronized (thread-safe but slower).
20. **`==` vs `equals()`?** `==` compares references (the same object). `equals()` compares logical content when the class overrides it.
21. **equals/hashCode contract?** Equal objects must have equal hash codes. If you override `equals` without `hashCode`, HashMap and HashSet break (duplicates, failed look-ups).
22. **What is a Singleton and how do you make it thread-safe?** A class with exactly one instance. Use a private constructor plus double-checked locking with `volatile`, or simply an `enum`.

**Exceptions**

23. **Checked vs unchecked?** Checked (`IOException`, `SQLException`) must be caught or declared, and represent recoverable external problems. Unchecked (`RuntimeException`s like NPE) aren't enforced and usually mean bugs.
24. **Error vs Exception?** `Error` is a serious JVM problem (OOM, StackOverflow) that apps shouldn't catch. `Exception` is a condition the app can handle.
25. **`throw` vs `throws`?** `throw` actually throws an exception object. `throws` declares in the signature that a method may throw.
26. **What is try-with-resources?** A `try (resource)` block that auto-closes anything implementing `AutoCloseable`, even when an exception occurs.
27. **Does `finally` run if there's a `return` in `try`?** Yes. It doesn't run only if the JVM exits (`System.exit`) or crashes.

**GC / memory**

28. **How does garbage collection work?** It marks every object reachable from GC roots (stack variables, statics, threads), sweeps the unreachable ones, and optionally compacts memory.
29. **Young vs old generation?** New objects go to young (Eden plus survivor spaces), where frequent, cheap minor GCs run because most objects die young. Survivors are promoted to old, which has rarer, costlier major GCs.
30. **Can Java leak memory?** Yes: objects still referenced but no longer needed. Examples: ever-growing static maps, unremoved listeners, unclosed resources, ThreadLocals in pools.
31. **Can you force GC?** `System.gc()` is only a request; the JVM may ignore it.

**Collections**

32. **How does HashMap work internally?** An array of buckets. The bucket is found from `hash & (n-1)`. Collisions are chained in a list, which becomes a red-black tree at 8 entries. It resizes (doubles) when size > capacity × 0.75. `equals` finds the exact key.
33. **HashMap time complexity?** Average O(1). Worst case O(log n) since Java 8, thanks to treeify (O(n) before).
34. **What is the load factor?** The fill ratio (default 0.75) that triggers a resize. It's a trade-off between memory and collisions.
35. **HashMap vs ConcurrentHashMap vs Hashtable?** HashMap isn't thread-safe and allows one null key. Hashtable is fully synchronized, legacy and slow. ConcurrentHashMap is thread-safe with lock-free reads and per-bucket locks/CAS, and allows no nulls.
36. **ArrayList vs LinkedList?** ArrayList has O(1) random access, is cache-friendly and is usually faster. LinkedList has O(n) access and only gains O(1) insert/remove at a known node. Use ArrayList (and ArrayDeque for queues).
37. **Fail-fast vs fail-safe iterator?** Fail-fast iterators (ArrayList, HashMap) throw `ConcurrentModificationException` if the collection is modified during iteration. Fail-safe ones (ConcurrentHashMap, CopyOnWriteArrayList) don't throw and may not show the latest changes.
38. **Comparable vs Comparator?** Comparable is the class's own natural order (`compareTo`). Comparator is an external, pluggable ordering, and you can have many.
39. **HashSet internally?** It's backed by a HashMap: elements are the keys, with a dummy constant as the value.
40. **TreeMap vs HashMap?** TreeMap is a red-black tree: sorted keys, O(log n), with `floor`/`ceiling` queries. HashMap is unordered with O(1) average.

**Multithreading**

41. **Process vs thread?** A process has its own memory space. Threads live inside a process and share its heap, each with its own stack. Threads are lighter and switch faster.
42. **Runnable vs Callable?** `Runnable.run()` returns nothing and can't throw checked exceptions. `Callable.call()` returns a value (via `Future`) and can throw.
43. **`start()` vs `run()`?** `start()` creates a new thread that calls `run()`. Calling `run()` directly executes on the current thread.
44. **Why use a thread pool?** Creating threads is expensive. A pool reuses a fixed set, queues extra tasks and bounds resource use, like my Atlassian SQS worker pool.
45. **What does `synchronized` do?** Only one thread at a time holds an object's monitor lock for that block or method. It gives mutual exclusion and memory visibility.
46. **What does `volatile` do, and is `volatile count++` safe?** It guarantees visibility of the latest value across threads and prevents reordering. No, `count++` isn't safe: read-modify-write isn't atomic. Use `AtomicInteger` or a lock.
47. **What is deadlock and how do you prevent it?** Threads wait forever on each other's locks. It needs mutual exclusion, hold-and-wait, no preemption and circular wait. Prevent it with a consistent lock ordering, or `tryLock` with a timeout.
48. **`wait()` vs `sleep()`?** `wait()` releases the lock, must be inside `synchronized`, and is woken by notify. `sleep()` keeps the lock and just pauses for a time.

**Spring (if your resume or the round mentions it)**

49. **What is dependency injection, and what is a bean?** DI means the container creates objects and passes in their dependencies instead of classes building them, giving loose coupling and easy testing. A bean is an object created and managed by the Spring container (singleton by default).
50. **`@Controller` vs `@RestController`, and how does Spring Boot handle config?** `@RestController` = `@Controller` + `@ResponseBody`: it returns JSON directly instead of a view name. Config comes from `application.yml`/`.properties`, overridden by profile files, env vars and command-line args, and is read with `@Value` or `@ConfigurationProperties`. Auto-configuration sets sensible defaults based on the classpath.

**Bonus (one line each, if time allows):**
- **Bean lifecycle?** Instantiate → inject → aware callbacks → BeanPostProcessor before-init → `@PostConstruct` → BeanPostProcessor after-init (proxies) → in use → `@PreDestroy` on shutdown.
- **What is auto-configuration?** Boot configures beans automatically, based on the libraries on the classpath and the beans you haven't defined yourself (`@ConditionalOnClass`, `@ConditionalOnMissingBean`).
- **`@Transactional` self-call gotcha?** Calling a transactional method from the same class bypasses the proxy, so no transaction is applied.

---

## How to practise this file (3 sessions)

1. **Session 1 (90 min):** Parts 1–2. Re-type every DSA toolkit snippet into an IDE and run it. Then solve 3 easy LeetCode problems **in Java only**: Valid Parentheses, Two Sum, Merge Intervals.
2. **Session 2 (90 min):** Parts 3–6. After each part, close the file and explain it aloud to an imaginary interviewer: HashMap internals, interface vs abstract class, why String is immutable, GC generations.
3. **Session 3 (60 min):** Parts 7–9. Run the multithreading demos and watch the unsafe counter give wrong answers. Then do the 50-question bank **out loud with a timer**: 20 seconds per answer. Mark the ones you stumble on and re-read just those sections.
