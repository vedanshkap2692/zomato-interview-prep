# DSA From Zero: Swiggy Round 1

> **Who this is for:** you, starting from the basics.
> **What it does:** teaches every DSA pattern Swiggy has actually asked. It goes in order of how often each pattern came up, and it fully solves 31 real Swiggy problems.
> **How long:** 6–8 hours to read and understand properly. Do it over 3–4 days. Don't rush it.
> **Code language:** Java. In 2026, Swiggy restricted its OA and DSA rounds to **Java or Go** ([16], [17] in `01_Process_and_Past_Questions.md`). Every solution here compiled and passed tests on JDK 21.

**How to read each problem:**
1. Read the problem statement.
2. Stop and think for 5 minutes, even if you get nowhere. That struggle is how the pattern sticks.
3. Then read the intuition, the dry run and the code.
4. The next day, type the code from memory.

---

## What Swiggy actually asks (from ~38 interview reports, 2021–2026)

| Rank | Pattern | Asked problems (times reported) |
|---|---|---|
| 1 | **Stack** | Valid Parentheses (3), Min Stack, stack using an array, postfix evaluation, next greater element |
| 2 | **Dynamic programming** | Grid paths (5), Knapsack / Coin Change (5), Edit Distance (2), Unique BSTs (2) |
| 3 | **Intervals** | Merge Intervals (3), Meeting Rooms, Minimum Platforms |
| 4 | **Trees** | Lowest Common Ancestor (3), Build Tree from Preorder + Inorder (2), Sum Root-to-Leaf |
| 5 | **Graphs** | Rotting Oranges (2), Network Delay Time, Currency Conversion, BFS problems |
| 6 | **Sliding window / two pointers** | Longest Substring Without Repeating (2), 3Sum, Trapping Rain Water |
| 7 | **Binary search** | First/Last Position (2026 ASDE Round 1), Rotated Array, Integer Nth Root |
| 8 | **Heap** | Min Cost to Connect Sticks (2), Running Median, Task Scheduler |
| 9 | **Linked list** | Reverse in k-Groups, Cycle Start |
| 10 | **Hashing / prefix sums** | Subarrays Divisible by K (2) |
| 11 | **Design** | LRU Cache (3: "LRU cache is most asked") |

**Good news for ASDE:** the reports say ASDE DSA is easier than SDE-1. Phrases used were "two very simple DSA questions", "easy-medium", and "first and last occurrence + design a stack using an array". If you understand the first 15 problems in this doc well, you are in good shape.

---

# Part 0: How to approach a DSA interview

## 0.1 What the round looks like

- **Format:** usually **2 problems in about 60 minutes**. The interviewer shares a problem; you code it in your own IDE (VS Code), on a third-party platform such as InterviewVector, or in a shared editor.
- **The code must run.** In the 2026 SDE-1 report the candidate had about 30 minutes of coding and the code was executed.
- **You will be judged on four things:**
  1. Did you understand the problem? (Did you ask good questions?)
  2. Did you find a good approach and explain *why* it works?
  3. Is the code correct and clean?
  4. Did you test it yourself and know its complexity?

A candidate got **waitlisted** at Swiggy partly for confidently saying a stack *couldn't* solve Next Greater Element (it can — see P2). Another got a "weak hire" for not finishing Edit Distance in time. So two lessons: **think before claiming something is impossible**, and **practise typing the code fast**.

## 0.2 The 6-step routine (use it every single time)

Think of it like a delivery partner getting an order. They don't just start driving. They check the address, pick a route, then go.

**Step 1: Repeat the problem in your own words.**
"So I'm given an array of meeting times, and I need to return the merged list where overlapping meetings become one. Right?"
This catches misunderstandings in 20 seconds instead of 20 minutes.

**Step 2: Ask clarifying questions.** Good ones:
- Can the input be empty? Can numbers be negative? Can there be duplicates?
- How big is the input? (1,000 items allows a slow O(n²) solution. 10⁵ items needs O(n log n) or better.)
- Is the input sorted?
- What should I return if there is no answer: -1, null, or an empty list?

**Step 3: Say a brute-force solution first.** "Brute force" means the most obvious, slow way, such as trying every possibility. Say it out loud and give its complexity. Don't code it unless they ask. It shows you can solve the problem, and it gives you something to improve.

**Step 4: Optimise.** Ask yourself:
- What work am I repeating? (A HashMap or DP can remember it.)
- Would sorting help?
- Is there a pattern from this doc that fits?

Explain the better idea *before* coding and get a nod: "I'll use a stack because…, does that sound OK?"

**Step 5: Code it cleanly.**
- Use clear variable names (`left`, `right`, `count`, not `a`, `b`, `c`).
- Talk while you code, in short sentences: "Now I'm handling the case where the stack is empty."

**Step 6: Dry run and test.** "Dry run" means walking through your code by hand with a small example, tracking each variable's value. Then try edge cases:
- empty input
- one element
- all elements the same
- negative numbers
- the largest input size

**If you get stuck:** say what you're thinking. Interviewers give hints, and several Swiggy candidates said hints helped. Silence is the worst thing you can do.

## 0.3 Big-O from zero

**Big-O** describes how the **work grows when the input grows**. It doesn't measure seconds. It measures how the program behaves when the input gets 10× or 1000× bigger.

Real-life analogy: you are looking for a friend's order slip in a pile of `n` slips.

| Big-O | Name | Real-life version | Example code |
|---|---|---|---|
| **O(1)** | constant | You know the slip is at the top. One look, however big the pile. | `arr[5]`, `map.get(key)` |
| **O(log n)** | logarithmic | The slips are sorted by order number. Open the middle, throw away half, repeat. 1,000,000 slips → about 20 looks. | binary search |
| **O(n)** | linear | Look at every slip once. | one `for` loop |
| **O(n log n)** | "n log n" | Sort the pile first (good sorting is n log n). | `Arrays.sort` |
| **O(n²)** | quadratic | For every slip, compare it with every other slip. 1,000 slips → 1,000,000 comparisons. | two nested loops |
| **O(2ⁿ)** | exponential | Try every subset of slips. Even 40 slips is a trillion subsets. | plain recursion that tries "take or skip" |

**Rules for counting:**
- Drop constants. `2n` and `n + 100` are both **O(n)**.
- Keep only the biggest term: `n² + n` is **O(n²)**.
- Loops one after another **add**: O(n) + O(n) = O(n).
- Loops inside each other **multiply**: O(n) × O(n) = O(n²).

**Space complexity** is the same idea for memory. An extra array of size n is O(n) space. A few variables are O(1).

**Quick limits table.** A computer does roughly 10⁸ simple steps per second.

| Input size n | Approach that fits |
|---|---|
| ≤ 20 | O(2ⁿ) is OK (try all subsets) |
| ≤ 1,000 | O(n²) is OK |
| ≤ 10⁵ – 10⁶ | need O(n log n) or O(n) |
| ≥ 10⁹ | need O(log n) or O(1), so think binary search or math |

**A Swiggy question about this:** "Which is bigger, O(2ⁿ) or O(n!)?" was asked in one report. The answer is **n!**. Both multiply n numbers together: 2ⁿ = 2 × 2 × 2 × …, while n! = 1 × 2 × 3 × … × n. From n = 4 onwards, every factor in n! is bigger than 2, so n! grows faster.

## 0.4 The Java toolkit you need

You don't need much Java for DSA. These few tools cover every problem in this doc.

```java
import java.util.*;

int[] arr = new int[5];                    // fixed-size array, all zeros
Arrays.sort(arr);                          // sort ascending
Arrays.fill(arr, -1);                      // fill every slot with -1
int[][] grid = new int[3][4];              // 3 rows, 4 columns

List<Integer> list = new ArrayList<>();    // growable array
list.add(7); list.get(0); list.size();

Map<String, Integer> map = new HashMap<>();          // key -> value, O(1) average
map.put("biryani", 2);
map.getOrDefault("pizza", 0);              // 0 if missing
map.containsKey("biryani");

Set<Integer> seen = new HashSet<>();       // unique items, O(1) contains

Deque<Integer> stack = new ArrayDeque<>(); // use as STACK: push / pop / peek
stack.push(1); stack.pop(); stack.peek(); stack.isEmpty();

Deque<Integer> queue = new ArrayDeque<>(); // use as QUEUE: add / poll / peek
queue.add(1); queue.poll();

PriorityQueue<Integer> minHeap = new PriorityQueue<>();                          // smallest on top
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder()); // largest on top
PriorityQueue<int[]> byTime = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1])); // custom order

// Sort intervals by start time
int[][] iv = {{3, 5}, {1, 2}};
Arrays.sort(iv, (a, b) -> Integer.compare(a[0], b[0]));
```

**Why `ArrayDeque` and not `Stack`?** Java's old `Stack` class is synchronised (slow) and outdated. `ArrayDeque` is the modern choice. If an interviewer asks, saying this is a small plus.

**Why `Integer.compare(a, b)` and not `a - b`?** `a - b` can overflow when the numbers are huge or negative and give the wrong sign. `Integer.compare` is always safe.

---

# Part 1: Stack (the most-asked pattern)

## 1.1 What a stack is

A **stack** is a pile where you can only touch the **top**. You add to the top (**push**) and remove from the top (**pop**). The last thing in is the first thing out, called **LIFO** (Last In, First Out).

**Real-life analogy:** a stack of plates in a restaurant kitchen. The dishwasher puts clean plates on top. The cook takes from the top. Nobody pulls a plate from the middle.

**Another one:** the "back" button in the Swiggy app. You go Home → Restaurant → Dish → Cart. Pressing back takes you to Dish, then Restaurant. The last screen you opened is the first one you leave.

**When to think "stack":**
- Things must be **matched in pairs**, where the most recent opener matches first: brackets, HTML tags.
- "For each element, find the **next/previous greater/smaller** element."
- You need to **undo** or go back.
- Evaluating expressions (postfix, calculators).

Push, pop and peek are all **O(1)**.

## 1.2 Template

```java
Deque<Integer> stack = new ArrayDeque<>();
for (each item x) {
    while (!stack.isEmpty() && <top should be resolved by x>) {
        <resolve stack.pop() using x>;
    }
    stack.push(x);
}
```

---

### P1. Valid Parentheses (LC 20): asked 3 times

**Problem.** You get a string containing only `( ) [ ] { }`. Is it valid? Every opening bracket must be closed by the same kind of bracket, in the correct order.
- `"({[]})"` → true
- `"(]"` → false
- `"(("` → false

**Intuition.** Think of a waiter taking orders at a table. Each time someone says "open a tab", the waiter writes it on top of a notepad. When someone says "close", it must match **the most recent open tab**, which is the top of the notepad. That's a stack.

**Approach.**
1. Go through each character.
2. If it's an opening bracket, push it.
3. If it's a closing bracket:
   - If the stack is empty, there is nothing to close → false.
   - Pop the top. If it's not the matching opener → false.
4. At the end, the stack must be empty. Anything left means something was never closed.

**Dry run** on `"({[]})"`:

| char | action | stack (top on right) |
|---|---|---|
| `(` | push | `(` |
| `{` | push | `( {` |
| `[` | push | `( { [` |
| `]` | pop `[`, matches | `( {` |
| `}` | pop `{`, matches | `(` |
| `)` | pop `(`, matches | empty |

The stack is empty at the end → **true**.

```java
static boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);                       // opening: remember it
        } else {
            if (stack.isEmpty()) return false;   // closing with nothing open
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stack.isEmpty();                      // anything left open = invalid
}
```

**Complexity:** O(n) time, O(n) space (the worst case is all openers).

**Follow-ups:**
- *"Only one type of bracket `()`?"* You don't need a stack. Keep a counter: +1 for `(`, −1 for `)`. If it ever goes negative → false. It must end at 0. O(1) space.
- *"With `*` that can be `(`, `)` or empty?"* That's LC 678, asked once at Swiggy. Track the **range** of possible open counts `[low, high]`:
  - `(` → both +1
  - `)` → both −1
  - `*` → low−1, high+1
  - If high < 0 → false. Keep low ≥ 0. At the end, low must be 0.
- *"Longest valid parentheses substring?"* That's LC 32, asked once. Push **indexes**, with −1 as a starting base. When the stack empties after a pop, push the current index as the new base. Otherwise the length is `i - stack.peek()`.

---

### P2. Next Greater Element: the one a candidate got wrong

**Problem.** For each element of an array, find the first element **to its right** that is bigger. If there is none, return -1.
`[4, 5, 2, 25]` → `[5, 25, 25, -1]`.

**Brute force:** for each element, scan to the right. O(n²).

**Intuition.** Imagine people standing in a queue at a Swiggy pickup counter, all facing left, and each person wants to know who is the first taller person behind them. Walk from left to right. Keep a "waiting list" of people who haven't found their answer yet. When a new person arrives, everyone on the waiting list who is **shorter** than them has found their answer: it's this new person. Those people leave the list.

The waiting list is always in **decreasing order** from bottom to top. If a shorter person were below a taller one, the shorter one would already have been answered. This is called a **monotonic stack**: a stack kept in sorted order.

**Approach.**
1. Store **indexes** in the stack, so you know where to write each answer.
2. For each `i`, while the top's value < `a[i]`: pop it and set its answer to `a[i]`.
3. Push `i`.
4. Anything left at the end has no greater element (-1).

**Dry run** on `[4, 5, 2, 25]`:
- i=0 (4): stack empty, push → [0]
- i=1 (5): a[0]=4 < 5 → ans[0]=5, pop. Push → [1]
- i=2 (2): a[1]=5 is not < 2. Push → [1, 2]
- i=3 (25): a[2]=2 < 25 → ans[2]=25. a[1]=5 < 25 → ans[1]=25. Push → [3]
- End: ans[3] stays -1. Result: [5, 25, 25, -1] ✓

```java
static int[] nextGreater(int[] a) {
    int n = a.length;
    int[] ans = new int[n];
    Arrays.fill(ans, -1);                        // default: no greater element
    Deque<Integer> stack = new ArrayDeque<>();   // holds INDEXES still waiting
    for (int i = 0; i < n; i++) {
        // a[i] is the answer for every waiting element smaller than it
        while (!stack.isEmpty() && a[stack.peek()] < a[i]) {
            ans[stack.pop()] = a[i];
        }
        stack.push(i);
    }
    return ans;
}
```

**Complexity:** O(n) time. It looks like a loop inside a loop, but every index is pushed once and popped at most once, so the total work is about 2n. O(n) space.

**Follow-ups:**
- *Circular array (LC 503):* loop `i` from 0 to 2n−1 and use `i % n`.
- *Next smaller:* flip `<` to `>`.
- *"Daily temperatures" (LC 739):* same code, but store `i - popped` (the distance) instead of the value.
- **Why is this O(n)?** Be ready to explain the "pushed once, popped once" argument. That's exactly where the waitlisted candidate went wrong.

---

### P3. Min Stack (LC 155): with an O(1)-extra-space follow-up

**Problem.** Build a stack that supports `push`, `pop`, `top` and `getMin()`, **all in O(1)**. `getMin` returns the smallest element currently in the stack.

**Intuition (simple version).** A stack of plates where each plate has a sticky note: "the lightest plate from here down weighs X." When you add a plate, write `min(its weight, the note on the plate below)`. When you remove a plate, the plate below already has the correct note. Nothing needs recomputing.

```java
// Min stack, simple version: every entry stores (value, min so far)
static class MinStack {
    private final Deque<int[]> st = new ArrayDeque<>();
    void push(int x) {
        int min = st.isEmpty() ? x : Math.min(x, st.peek()[1]);
        st.push(new int[]{x, min});
    }
    void pop() { st.pop(); }
    int top() { return st.peek()[0]; }
    int getMin() { return st.peek()[1]; }
}
```

All operations are O(1). Space is O(n) **extra**, because there is one extra number per element.

**The Swiggy follow-up (asked in a 2021 report): do it with O(1) extra space.** You may keep only **one** extra variable, `min`.

**The trick.** When a new value `x` is smaller than the current `min`, don't push `x`. Push a coded number, `2x − min`, and set `min = x`.
- Why this works: `x < min` means `x − min < 0`, so `2x − min < x`. The coded value is **smaller than the new min**. A normal value is never smaller than the min, so the coded one is easy to spot.
- When you pop and the popped value `t < min`, you know it was coded: the element you are removing *was* the min. The previous min was `2·min − t`. (Check: `2x − (2x − oldMin) = oldMin`.)

```java
// Min stack with O(1) extra space: store an "encoded" value when a new min arrives
static class MinStackO1 {
    private final Deque<Long> st = new ArrayDeque<>();
    private long min;
    void push(int xi) {
        long x = xi;
        if (st.isEmpty()) { st.push(x); min = x; }
        else if (x >= min) { st.push(x); }
        else {                                    // new minimum
            st.push(2 * x - min);                 // encoded: always < x
            min = x;
        }
    }
    void pop() {
        long t = st.pop();
        if (t < min) min = 2 * min - t;           // we popped the min: recover the old one
    }
    int top() {
        long t = st.peek();
        return (int) (t < min ? min : t);
    }
    int getMin() { return (int) min; }
}
```

**Dry run:**
- push 5 → stack [5], min 5
- push 3 → 3 < 5, so push 2·3−5 = **1**, min = 3. Stack [5, 1].
- getMin → 3 ✓
- pop → popped 1 < min 3, so this was the coded entry. min = 2·3 − 1 = **5** ✓

**Why `long`?** `2x − min` can overflow `int` when values are near ±2 billion.

---

### P4. Design a stack using an array: asked in Round 1 of ASDE 2026

**Problem.** Implement `push`, `pop`, `peek`, `isEmpty` and `size` using a plain array, without Java collections. The interviewer also wanted **classes and interfaces** ("basic LLD concepts"), so show clean design.

**Intuition.** An array is a row of numbered lockers. Keep one counter, `size`, that tells you how many lockers are in use. The top of the stack is locker `size − 1`.
- **Push:** put the item in locker `size`, then `size++`.
- **Pop:** `size--`, then return locker `size`.
- **Full?** Get a row twice as long and move everything over. This is how `ArrayList` grows.

```java
// Stack built on a plain array
static class ArrayStack {
    private int[] data;
    private int size = 0;                         // also = index of next free slot
    ArrayStack(int capacity) { data = new int[capacity]; }
    void push(int x) {
        if (size == data.length) data = Arrays.copyOf(data, data.length * 2); // grow
        data[size++] = x;
    }
    int pop() {
        if (size == 0) throw new RuntimeException("Stack is empty");
        return data[--size];
    }
    int peek() {
        if (size == 0) throw new RuntimeException("Stack is empty");
        return data[size - 1];
    }
    boolean isEmpty() { return size == 0; }
    int size() { return size; }
}
```

**Complexity:** push, pop and peek are O(1). Push becomes O(n) on the rare call that grows the array. Averaged over many pushes it is still O(1), which is called **amortised O(1)**.

**How to show "LLD" in this question** (the interviewer asked for classes and interfaces):

```java
interface Stack<T> {          // the contract: what any stack must do
    void push(T item);
    T pop();
    T peek();
    boolean isEmpty();
    int size();
}
// class ArrayStack<T> implements Stack<T> { ... }   // one implementation
// class LinkedListStack<T> implements Stack<T> { ... } // another, same contract
```

Say: "I code against the `Stack` interface. Callers don't care whether it's backed by an array or a linked list, so I can swap implementations without changing them." That is the **"program to an interface"** principle (see the machine coding doc, `03_...`, in this folder).

**Follow-ups:**
- *Stack using a linked list:* push = add at head, pop = remove head. Always O(1), no resizing needed.
- *Two stacks in one array:* one grows from the left end, one from the right end. The array is full when they meet.
- *Make it generic:* use `Object[]` and cast to `T`, or use `List<T>`.
- *Throw or return -1 on an empty pop?* Throwing an exception is cleaner. Say why: -1 could be a real value.

---

### P5. Evaluate Postfix Expression: asked at ASDE (Glassdoor)

**Problem.** Evaluate an expression written in **postfix**, where the operator comes after its two numbers.
`2 3 1 * + 9 -` means `2 + (3 × 1) − 9` = **−4**.

**Why postfix exists:** it needs no brackets and no precedence rules. Computers like it.

**Intuition.** A cashier with a calculator that has a "stack" of numbers. When you read a number, put it on the pile. When you read an operator, take the top two numbers, combine them, and put the result back.

**The one trap:** the **first** number you pop is the **right** operand. For `9 3 -`, pop 3 (b), then 9 (a), and compute a − b = 6, not 3 − 9.

```java
static int evalPostfix(String[] tokens) {
    Deque<Integer> st = new ArrayDeque<>();
    for (String t : tokens) {
        switch (t) {
            case "+": case "-": case "*": case "/": {
                int b = st.pop();                 // right operand comes off first
                int a = st.pop();
                if (t.equals("+")) st.push(a + b);
                else if (t.equals("-")) st.push(a - b);
                else if (t.equals("*")) st.push(a * b);
                else st.push(a / b);
                break;
            }
            default: st.push(Integer.parseInt(t));
        }
    }
    return st.pop();
}
```

**Dry run** on `2 3 1 * + 9 -`:

| token | stack |
|---|---|
| 2 | [2] |
| 3 | [2, 3] |
| 1 | [2, 3, 1] |
| * | 3 × 1 = 3 → [2, 3] |
| + | 2 + 3 = 5 → [5] |
| 9 | [5, 9] |
| - | 5 − 9 = −4 → [−4] |

**Complexity:** O(n) time, O(n) space.

**Follow-ups:** *Infix to postfix* (the Shunting-yard idea): keep operators on a stack and pop the ones with higher or equal precedence before pushing a new operator. Also: *division by zero?* Check it and throw.

---

# Part 2: Intervals

## 2.1 What an interval problem is

An **interval** is a start and an end: `[9:00, 9:30]`. Interval problems ask about **overlaps**: merge overlapping ones, check whether any overlap, or count how many overlap at once.

**Real-life analogy:** a restaurant's delivery slots, meeting-room bookings, train platforms, or delivery partners' shifts.

**The golden trick: sort by start time first.** Once sorted, an interval can only overlap with the ones right before it. You never have to look far back.

**Two intervals `[a, b]` and `[c, d]` (with a ≤ c) overlap if `c ≤ b`**, meaning the second starts before the first ends. (Whether "touching", `c == b`, counts as overlap depends on the problem. Ask!)

**Template:**
```java
Arrays.sort(iv, (x, y) -> Integer.compare(x[0], y[0]));
for (int[] cur : iv) {
    if (<cur overlaps the last kept one>) <extend / count conflict>;
    else <start a new one>;
}
```

---

### P6. Merge Intervals (LC 56): asked 3 times (ASDE HM round, SDE-1 2026 "variation")

**Problem.** Merge all overlapping intervals.
`[[1,3],[2,6],[8,10],[15,18]]` → `[[1,6],[8,10],[15,18]]`.

**Intuition.** You're a restaurant manager with booking slips for the private dining room, all mixed up. Sort them by start time. Go through them. If the next booking starts before the current block ends, extend the block. Otherwise close the block and start a new one.

**Approach.**
1. Sort by start.
2. Keep a result list.
3. For each interval:
   - If the result is empty, or the last block ends **before** this one starts → add it as a new block.
   - Otherwise → the last block's end = max(last end, this end).

**Dry run** (sorted: [1,3], [2,6], [8,10], [15,18]):
- [1,3] → result [[1,3]]
- [2,6]: 2 ≤ 3, overlap → end = max(3,6) = 6 → [[1,6]]
- [8,10]: 8 > 6 → new → [[1,6],[8,10]]
- [15,18]: new → [[1,6],[8,10],[15,18]] ✓

```java
static int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (x, y) -> Integer.compare(x[0], y[0])); // sort by start
    List<int[]> res = new ArrayList<>();
    for (int[] cur : intervals) {
        if (res.isEmpty() || res.get(res.size() - 1)[1] < cur[0]) {
            res.add(new int[]{cur[0], cur[1]});   // no overlap: new block
        } else {
            int[] last = res.get(res.size() - 1); // overlap: stretch the end
            last[1] = Math.max(last[1], cur[1]);
        }
    }
    return res.toArray(new int[0][]);
}
```

**Why `max` and not just `cur[1]`?** In `[1,10]` followed by `[2,3]`, the second interval is fully inside the first. The end must stay 10.

**Complexity:** O(n log n) for the sort, plus O(n) for the scan. O(n) for the output.

**Follow-ups (the "variations"):**
- *Insert a new interval into a sorted list (LC 57):* add the intervals ending before it, merge the overlapping ones, then add the rest. O(n).
- *Minimum intervals to remove so none overlap (LC 435):* sort by **end** and greedily keep the one that ends earliest.
- *Total time covered:* merge, then sum `end − start`.
- *Intersection of two interval lists (LC 986):* two pointers.

---

### P7. Meeting Rooms (LC 252) and Minimum Platforms (asked 2024)

**Meeting Rooms problem.** Can one person attend all the meetings? That is, do any two overlap?

Sort by start. If any meeting starts before the previous one ends → false.

```java
static boolean canAttendMeetings(int[][] meetings) {
    Arrays.sort(meetings, (x, y) -> Integer.compare(x[0], y[0]));
    for (int i = 1; i < meetings.length; i++) {
        if (meetings[i][0] < meetings[i - 1][1]) return false; // starts before previous ends
    }
    return true;
}
```

**Minimum Platforms problem** (the same idea as Meeting Rooms II, LC 253). Given train arrival and departure times, how many platforms are needed so no train waits? In Swiggy terms: **how many delivery partners are needed at a restaurant's pickup counter at the busiest moment?**

**Intuition.** Picture a doorman with a clicker counter. Every time a train **arrives**, click +1. Every time one **leaves**, click −1. The highest number the counter ever shows is your answer. You don't care *which* train is on which platform, only how many there are at once.

So: sort the arrivals and the departures **separately**, and walk through both with two pointers, like merging two sorted lists of events.

```java
static int minPlatforms(int[] arr, int[] dep) {
    int[] a = arr.clone(), d = dep.clone();
    Arrays.sort(a);
    Arrays.sort(d);
    int i = 0, j = 0, cur = 0, best = 0;
    while (i < a.length) {
        if (a[i] <= d[j]) { cur++; i++; }        // a train arrives before the next one leaves
        else { cur--; j++; }                      // a train leaves first
        best = Math.max(best, cur);
    }
    return best;
}
```

**Dry run.**
- Arrivals [900, 940, 950, 1100, 1500, 1800], departures [910, 1200, 1120, 1130, 1900, 2000].
- Sorted departures: [910, 1120, 1130, 1200, 1900, 2000].

| event | count |
|---|---|
| 900 arrive | 1 |
| 910 leave | 0 |
| 940 arrive | 1 |
| 950 arrive | 2 |
| 1100 arrive | **3** |
| 1120 leave | 2 |
| … | … |

Answer: **3**.

**`<=` vs `<`:** with `<=`, a train arriving at exactly the minute another leaves needs a **new** platform. That's the usual convention for this problem. For meeting rooms, ending at 10:00 and starting at 10:00 usually *is* allowed, so use `<`. **Ask the interviewer.**

**Complexity:** O(n log n) time, O(n) space for the sorted copies.

**Alternative:** sort the meetings by start time and keep a **min-heap of end times**. For each meeting, if the earliest-ending room is free (heap top ≤ start), pop it. Then push this meeting's end. The heap size at the end is the answer. Learn it too; it's the "which room" version.

---

# Part 3: Trees

## 3.1 What a tree is

A **tree** is a structure where each item (a **node**) can point to children, and there are no loops. The top node is the **root**. Nodes with no children are **leaves**.

A **binary tree** gives each node at most 2 children: `left` and `right`.

**Real-life analogy:** Swiggy's menu. "Menu" is the root. Its children are "Starters", "Mains" and "Desserts". "Mains" has the children "Biryani" and "Curries", and so on. Or a company org chart: CEO → VPs → managers → engineers.

A **binary search tree (BST)** is a binary tree where, for every node, everything on the left is **smaller** and everything on the right is **bigger**. Like a sorted list arranged as a tree, so you can search by going left or right.

```java
static class TreeNode {
    int val; TreeNode left, right;
    TreeNode(int v) { val = v; }
}
```

## 3.2 Traversals: the four ways to visit every node

| Name | Order | Memory trick | Use |
|---|---|---|---|
| **Preorder** | Root, Left, Right | root comes **pre** (first) | copy a tree, serialise |
| **Inorder** | Left, Root, Right | root **in** the middle | in a BST this gives **sorted order** |
| **Postorder** | Left, Right, Root | root comes **post** (last) | delete a tree, compute heights (children first) |
| **Level order** | level by level | BFS with a queue | "top view", "right view", min depth |

```java
static void inorder(TreeNode t, List<Integer> out) {
    if (t == null) return;
    inorder(t.left, out); out.add(t.val); inorder(t.right, out);
}
static void preorder(TreeNode t, List<Integer> out) {
    if (t == null) return;
    out.add(t.val); preorder(t.left, out); preorder(t.right, out);
}
```

**How to think recursively about trees.** This is the single most important idea in the chapter. **Trust that the function works on the children.** Don't trace the whole tree in your head. Ask three questions:
1. What do I return for an empty tree (`null`)? This is the **base case**.
2. If my left child's call gives me X and my right child's call gives me Y, what do I return?
3. Done.

Example, the height of a tree: `height(null) = 0`, and `height(node) = 1 + max(height(left), height(right))`.

**Recursion explained once:** a function that calls itself on a smaller piece of the problem. It's like a manager who asks two team leads for their numbers, adds their own, and reports up. Each team lead does the same with their teams. The "base case" is the intern with no team, who just reports their own number.

---

### P8. Lowest Common Ancestor of a Binary Tree (LC 236): asked 3 times

**Problem.** Given two nodes `p` and `q` in a binary tree, find their **lowest common ancestor (LCA)**: the deepest node that has both p and q underneath it. A node counts as its own ancestor.

**Real-life analogy:** a family tree. The LCA of two cousins is the grandparent they share. The LCA of a parent and their child is the parent.

**Intuition.** Ask each subtree a question: "Did you find p or q in there?"
- If the **left** finds one and the **right** finds the other, **I'm** the place where their paths meet. I'm the LCA.
- If only one side found something, pass that up. It's either one of the nodes or their LCA, already found lower down.
- If I *am* p or q, return myself. Even if the other is below me, I'm still the answer.

**Approach.**
1. Base: if root is null, p or q → return root.
2. `l = lca(left)`, `r = lca(right)`.
3. Both non-null → return root. Otherwise return whichever is non-null.

**Dry run** on the tree below, with p = 15 and q = 7:

```
        3
       / \
      9   20
         /  \
        15   7
```

- At 3: the left (9) subtree has neither → null.
- At 20: left = 15 (found p), right = 7 (found q). Both non-null → return 20.
- At 3: left null, right 20 → return 20. ✓

```java
static TreeNode lca(TreeNode root, TreeNode p, TreeNode q) {
    if (root == null || root == p || root == q) return root;
    TreeNode l = lca(root.left, p, q);
    TreeNode r = lca(root.right, p, q);
    if (l != null && r != null) return root;      // one on each side: I am the meeting point
    return l != null ? l : r;                     // pass up whatever was found
}
```

**Complexity:** O(n) time, since each node is visited once. O(h) space for the recursion, where h is the height (log n if balanced, n if it's a straight line).

**Follow-ups:**
- **BST version (LC 235), also discussed at Swiggy.** Use the ordering, no full search needed:
  - if both values < root, go left
  - if both > root, go right
  - otherwise root is the split point → that's the LCA
  - O(h) time
- *What if p or q might not be in the tree?* Do a full traversal that counts how many of the two were found, and only trust the answer if the count is 2.
- *With parent pointers?* It becomes "intersection of two linked lists": walk up from both.
- *Distance between two nodes:* depth(p) + depth(q) − 2·depth(LCA).

---

### P9. Construct a Binary Tree from Preorder and Inorder (LC 105): asked twice

**Problem.** Given the preorder and inorder traversal arrays of a tree with unique values, rebuild the tree.
`preorder = [3,9,20,15,7]`, `inorder = [9,3,15,20,7]`.

A candidate was waitlisted partly for **running out of time** on this one. Learn it until it's automatic.

**Intuition, from two facts:**
1. **Preorder's first element is always the root** (root comes first).
2. **In inorder, the root splits the array:** everything to its left is the left subtree, everything to its right is the right subtree.

So: take 3 from preorder, which is the root. Find 3 in inorder: `[9] 3 [15,20,7]`. The left subtree has values {9}; the right has {15,20,7}. Recurse. The next preorder values (9, then 20…) are the roots of those subtrees, in order, because preorder does the root, then the **whole left side**, then the whole right side.

**Approach.**
1. Put every inorder value's index in a HashMap, so finding the root is O(1) instead of a scan.
2. Keep a global pointer `preIdx` into preorder.
3. `build(lo, hi)` builds the tree from inorder range `[lo..hi]`:
   - if lo > hi → null
   - the root is `pre[preIdx++]`; find its position `mid`
   - left = `build(lo, mid−1)`, right = `build(mid+1, hi)`. **Left first**, because preorder lists the left subtree first.

```java
static int preIdx;
static Map<Integer, Integer> inPos;
static TreeNode buildTree(int[] preorder, int[] inorder) {
    preIdx = 0;
    inPos = new HashMap<>();
    for (int i = 0; i < inorder.length; i++) inPos.put(inorder[i], i);
    return build(preorder, 0, inorder.length - 1);
}
private static TreeNode build(int[] pre, int lo, int hi) {
    if (lo > hi) return null;                     // no values for this subtree
    int rootVal = pre[preIdx++];                  // next preorder value is this subtree's root
    TreeNode root = new TreeNode(rootVal);
    int mid = inPos.get(rootVal);                 // split inorder into left | root | right
    root.left = build(pre, lo, mid - 1);          // left first (preorder is root, left, right)
    root.right = build(pre, mid + 1, hi);
    return root;
}
```

**Dry run.**
- build(0,4): root 3, mid 1
  - left = build(0,0): root 9, mid 0, so its children are build(0,−1) = null and build(1,0) = null
  - right = build(2,4): root 20, mid 3
    - left = build(2,2) → 15
    - right = build(4,4) → 7
- The tree matches the picture in P8. ✓

**Complexity:** O(n) time with the HashMap (O(n²) without it). O(n) space.

**Follow-ups:**
- *Postorder + inorder (LC 106):* the root is the **last** element of postorder. Walk the postorder pointer backwards and build **right first**.
- *Why can't you rebuild from preorder + postorder alone?* If a node has only one child, you can't tell whether it's the left or the right child.
- *Duplicates?* Then a value's position in inorder is ambiguous and this method fails.

---

### P10. Count Unique BSTs (LC 96): asked twice ("count binary trees from 1 to n")

**Problem.** How many structurally different BSTs can be built from the values 1..n?
- n = 3 → 5
- n = 5 → 42

**Intuition.** Pick which number is the root. Say root = `r`. Then 1..r−1 **must** go on the left (they're smaller), and r+1..n on the right. The left side can be arranged in `count(r−1)` ways and the right side in `count(n−r)` ways. Each left shape can pair with each right shape, so the number of trees with root `r` is **left × right**. Add this up over all roots.

It only matters **how many** nodes are on each side, not which numbers. 3 nodes {4,5,6} make the same number of shapes as {1,2,3}.

```
dp[0] = 1                 (empty tree: exactly one way, "nothing")
dp[n] = sum over root r = 1..n of dp[r-1] * dp[n-r]
```

These numbers are called **Catalan numbers**: 1, 1, 2, 5, 14, 42…

```java
static int numTrees(int n) {
    int[] dp = new int[n + 1];
    dp[0] = 1;                                    // empty tree: one way
    dp[1] = 1;
    for (int nodes = 2; nodes <= n; nodes++) {
        for (int root = 1; root <= nodes; root++) {
            dp[nodes] += dp[root - 1] * dp[nodes - root]; // left size * right size
        }
    }
    return dp[n];
}
```

**Dry run for n = 3:**
- root 1 → dp[0]·dp[2] = 1·2 = 2
- root 2 → dp[1]·dp[1] = 1
- root 3 → dp[2]·dp[0] = 2
- Total **5** ✓

**Complexity:** O(n²) time, O(n) space.

**Follow-ups:**
- *Why is dp[0] = 1?* With root 1 and an empty left side, there is exactly one way to arrange "nothing". If it were 0, every product would become 0.
- *Generate all the trees (LC 95):* the same recursion, but returning lists of trees.
- *Closed form:* C(2n, n) / (n+1).
- The same numbers count valid bracket strings with n pairs, and ways to triangulate a polygon.

**Also asked once (2024 SDE-1 OA): Sum Root-to-Leaf Numbers (LC 129).** Each root-to-leaf path such as 1→2→3 forms the number 123. Return the sum of all of them. Recurse with `cur = cur*10 + node.val`, and at a leaf, return `cur`.

---

# Part 4: Dynamic Programming from zero

DP sounds scary. It isn't. It is **recursion plus remembering answers**. Swiggy asked DP in at least 12 reports (grid paths, knapsack, coin change, edit distance), so this is the biggest chapter.

## 4.1 The idea in one example

**Climbing stairs:** you can climb 1 or 2 steps at a time. How many ways are there to reach step n?

**Think backwards.** To stand on step n, your last move was either from step n−1 (a 1-step) or from step n−2 (a 2-step). So:

```
ways(n) = ways(n-1) + ways(n-2),   ways(0) = ways(1) = 1
```

**Plain recursion is slow.** `ways(5)` calls `ways(4)` and `ways(3)`. `ways(4)` calls `ways(3)` **again**. The same questions get answered over and over, which is exponential, O(2ⁿ).

**Real-life analogy:** someone asks you "what's 1+1+1+1+1+1+1?" You count: 7. Then they add "+1". You don't recount from scratch. You remember 7 and say 8. **DP is remembering.**

### Step 1 → 2 → 3: how to build every DP solution

**Step 1: write the recursion** (what are the choices at each step?).

**Step 2: memoisation (top-down).** Keep an array `memo`. Before computing, check whether the answer is already saved. After computing, save it.

```java
static long[] memo;
static long climb(int n) {
    if (n <= 1) return 1;
    if (memo[n] != 0) return memo[n];
    return memo[n] = climb(n - 1) + climb(n - 2);
}
```

Now each `n` is computed once, so it's O(n).

**Step 3: tabulation (bottom-up).** Fill an array from the smallest case upwards, with no recursion:

```java
long[] dp = new long[n + 1];
dp[0] = dp[1] = 1;
for (int i = 2; i <= n; i++) dp[i] = dp[i - 1] + dp[i - 2];
```

**When is a problem DP?** It needs two things:
1. **Overlapping subproblems:** the same smaller question gets asked repeatedly.
2. **Optimal substructure:** the best answer for the big problem is built from the best answers for smaller ones.

Keywords: "number of ways", "minimum cost", "maximum value", "is it possible", "longest/shortest" combined with choices at each step.

**The 4 things to write down for every DP problem:**
1. **State:** what does `dp[i]` (or `dp[i][j]`) *mean*, in words? This is the most important step.
2. **Transition:** how do you compute it from smaller states?
3. **Base case:** the smallest states you know directly.
4. **Answer:** which cell holds the final answer?

---

### P11. Unique Paths II, grid with obstacles (LC 63): grid DP was asked 5 times

**Problem.** A delivery partner starts at the top-left of an m×n grid of city blocks and must reach the bottom-right, moving only **right** or **down**. Some cells are blocked (1). How many different routes are there?

```
0 0 0
0 1 0      → 2 routes
0 0 0
```

**Intuition.** The only ways to arrive at a cell are **from above** or **from the left**. So `routes to here = routes to the cell above + routes to the cell on the left`. A blocked cell has 0 routes.

**The 4 things:**
1. **State:** `dp[i][j]` = number of routes from the start to cell (i, j).
2. **Transition:** `dp[i][j] = dp[i−1][j] + dp[i][j−1]`, or 0 if the cell is blocked.
3. **Base:** `dp[0][0] = 1` (if it's not blocked).
4. **Answer:** `dp[m−1][n−1]`.

```java
static int uniquePathsWithObstacles(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dp = new int[m][n];
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 1) { dp[i][j] = 0; continue; }   // blocked cell
            if (i == 0 && j == 0) { dp[i][j] = 1; continue; }  // start
            int fromTop = i > 0 ? dp[i - 1][j] : 0;
            int fromLeft = j > 0 ? dp[i][j - 1] : 0;
            dp[i][j] = fromTop + fromLeft;
        }
    }
    return dp[m - 1][n - 1];
}
```

**Dry run** (the filled dp table):

```
1 1 1
1 0 1
1 1 2   ← answer 2
```

**Complexity:** O(m·n) time and space. Space can drop to O(n) with a single row, since each row only needs the row above it.

---

### P12. Minimum Path Sum (LC 64) and its Swiggy variants

**Problem.** Each cell has a cost (say, minutes of traffic). Moving only right or down, find the **cheapest** route from the top-left to the bottom-right.

It's the same shape as P11. Change "+" to "min of the two, plus my own cost":

```java
static int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dp = new int[m][n];
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (i == 0 && j == 0) dp[i][j] = grid[0][0];
            else if (i == 0) dp[i][j] = dp[i][j - 1] + grid[i][j];
            else if (j == 0) dp[i][j] = dp[i - 1][j] + grid[i][j];
            else dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
        }
    }
    return dp[m - 1][n - 1];
}
```

`[[1,3,1],[1,5,1],[4,2,1]]` → **7** (1→3→1→1→1).

**The Swiggy variants seen in reports, and how to adapt:**

| Variant | Change |
|---|---|
| Max points, with some negative cells | `max` instead of `min` |
| Can I reach B from A moving **right, up-right, down-right** in a 0/1 grid? | Go **column by column**. Cell (i, j) is reachable if it's open and any of (i, j−1), (i−1, j−1) or (i+1, j−1) is reachable. |
| "Medium-hard 2D grid DP" in the ASDE OA | Same method: define the state, list where you can come *from*, fill in the right order |

**Golden rule for grid DP:** fill the cells in an order where the cells you depend on are **already filled**.

---

### P13. 0/1 Knapsack: the knapsack family was asked 5 times

**Problem.** A delivery bag can carry weight `W`. There are items, each with a weight and a value. Each item can be taken **at most once** (that's the "0/1": take it or don't). Maximise the total value.

`weights [1,3,4,5]`, `values [1,4,5,7]`, `W = 7` → **9** (take the weights 3 and 4: values 4 + 5).

**Why greedy fails:** "take the highest value per kg first" can leave wasted space. You must consider combinations, which is where DP comes in.

**Intuition.** For each item there are exactly two choices: **skip it** or **take it** (if it fits). If you take it, you get its value plus the best you can do with the remaining capacity, using the earlier items.

**The 4 things (2D version first, to understand it):**
1. **State:** `dp[i][c]` = the best value using only the first `i` items with capacity `c`.
2. **Transition:** `dp[i][c] = max(dp[i−1][c], val[i] + dp[i−1][c − wt[i]])`. The first option skips item i; the second takes it, if `wt[i] ≤ c`.
3. **Base:** `dp[0][c] = 0` (no items, no value).
4. **Answer:** `dp[n][W]`.

**Space trick (1D).** Each row only uses the row before it, so keep one array. Loop the capacity **downwards**, from W to wt[i]. Why downwards? With an upward loop, `dp[c − wt[i]]` might already include item i from this same round, which would use the item twice. Going down means you always read **old** values (without item i).

```java
// 0/1 knapsack: each item at most once
static int knapsack01(int[] wt, int[] val, int W) {
    int[] dp = new int[W + 1];                    // dp[c] = best value with capacity c
    for (int i = 0; i < wt.length; i++) {
        for (int c = W; c >= wt[i]; c--) {        // go DOWN so item i is used once
            dp[c] = Math.max(dp[c], dp[c - wt[i]] + val[i]);
        }
    }
    return dp[W];
}
```

**Complexity:** O(n·W) time, O(W) space.

**Related problems asked:**
- **Subset sum** ("select weights so the sum ≤ capacity", or "can you make exactly S?"). It's the same DP with booleans: `dp[c] = dp[c] || dp[c − w]`.
- **LC 2035 "partition into two arrays, minimise the sum difference"** (asked 2025). The easy version: subset-sum up to total/2, and the best reachable sum s gives the difference `total − 2s`. LC 2035 itself has negative numbers and exact halves, which needs "meet in the middle". Know that it exists; it's a hard problem.

---

### P14. Coin Change II (LC 518): unbounded knapsack, asked in a 2025 HM round

**Problem.** Coins `[1, 2, 5]`, amount 5. How many **combinations** make the amount? Each coin can be used **unlimited** times.

The answer is 4: 5 | 2+2+1 | 2+1+1+1 | 1+1+1+1+1.

**Intuition.** It's like the knapsack, but items can repeat. `dp[a]` = number of ways to make amount `a`. For each coin, every amount `a` can be made by "a way to make `a − coin`, plus this coin".

**Two details to understand:**
1. **Loop the amount upwards.** Now we *want* to reuse the coin within the same round. The opposite of 0/1 knapsack.
2. **Coins in the outer loop, amount in the inner loop.** This counts **combinations**: 1+2 and 2+1 count once, because coins are added in a fixed order. If you swap the loops (amount outside), you count **orderings** (permutations), which is a different problem (LC 377). Interviewers love this question.

```java
// Coin Change II: number of combinations (unbounded knapsack)
static int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1;                                    // one way to make 0: pick nothing
    for (int coin : coins) {                      // coins OUTER => combinations, not orders
        for (int a = coin; a <= amount; a++) {    // go UP so a coin can be reused
            dp[a] += dp[a - coin];
        }
    }
    return dp[amount];
}
```

**Dry run** (amount 5):

| after coin | dp[0..5] |
|---|---|
| start | 1 0 0 0 0 0 |
| 1 | 1 1 1 1 1 1 |
| 2 | 1 1 2 2 3 3 |
| 5 | 1 1 2 2 3 **4** |

**Complexity:** O(coins × amount) time, O(amount) space.

**Follow-ups:**
- **Coin Change I (LC 322): the *fewest* coins.** `dp[a] = min(dp[a], dp[a − coin] + 1)`. Initialise dp to "infinity" (amount + 1), with dp[0] = 0. If dp[amount] is still infinity → −1.
- *Packets of 6, 9, 20: can N be made, and with the fewest packets?* (asked once). Exactly Coin Change I with coins [6, 9, 20].
- *Why does greedy fail?* Coins [1, 3, 4], amount 6: greedy takes 4+1+1 (3 coins), but 3+3 is 2 coins.

---

### P15. Edit Distance (LC 72): asked twice; one candidate didn't finish it in time

**Problem.** The minimum number of operations (**insert**, **delete**, **replace** one character) to turn word `a` into word `b`.
- `horse → ros` = 3
- `intention → execution` = 5

**Real-life use:** Swiggy's search box. A user types "biryni" and you want "biryani". Edit distance measures how close two words are.

**Intuition.** Compare the words from the end.
- If the last letters **match**, they cost nothing. Solve for the rest.
- If they don't match, try all 3 operations and take the cheapest:
  - **Replace** a's last letter with b's → both shrink by one: `dp[i−1][j−1] + 1`
  - **Delete** a's last letter → a shrinks: `dp[i−1][j] + 1`
  - **Insert** b's last letter at the end of a → now they match, so b shrinks: `dp[i][j−1] + 1`

**The 4 things:**
1. **State:** `dp[i][j]` = the minimum operations to turn the first `i` letters of a into the first `j` letters of b.
2. **Transition:** as above.
3. **Base:** `dp[i][0] = i` (delete all i letters); `dp[0][j] = j` (insert all j letters).
4. **Answer:** `dp[m][n]`.

```java
static int minDistance(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];           // dp[i][j]: first i chars of a -> first j of b
    for (int i = 0; i <= m; i++) dp[i][0] = i;    // delete everything
    for (int j = 0; j <= n; j++) dp[0][j] = j;    // insert everything
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];      // same letter: free
            } else {
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1],      // replace
                               Math.min(dp[i - 1][j],          // delete from a
                                        dp[i][j - 1]));        // insert into a
            }
        }
    }
    return dp[m][n];
}
```

**Dry run** (`horse → ros`, the final table):

```
      ""  r  o  s
""     0  1  2  3
h      1  1  2  3
o      2  2  1  2
r      3  2  2  2
s      4  3  3  2
e      5  4  4  3   ← answer 3
```

**Watch the index shift:** `dp` has size (m+1)×(n+1) because row 0 means "the empty prefix". So letter i of the word is `charAt(i − 1)`. This off-by-one is the most common bug.

**Complexity:** O(m·n) time and space. This can drop to two rows, O(n).

**Follow-ups:**
- *Longest Common Subsequence (LC 1143):* same table. Match → `dp[i−1][j−1] + 1`, otherwise `max(dp[i−1][j], dp[i][j−1])`.
- *Print the actual operations:* walk back from `dp[m][n]` and follow which choice gave the minimum.

---

# Part 5: Graphs

## 5.1 What a graph is

A **graph** is a set of **nodes** (also called vertices) connected by **edges**. A tree is a special graph with no loops. A general graph can have loops, and a node can be reached in many ways.

**Real-life analogy:** a city map. Junctions are nodes and roads are edges.
- Roads can be **one-way** (a **directed** graph) or two-way (**undirected**).
- Roads can have a travel time (a **weighted** graph).

Swiggy is basically one big graph problem: restaurants, customers and delivery partners on a road network.

**How to store a graph: an adjacency list.** For each node, keep a list of its neighbours.

```java
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
adj.get(u).add(v);            // edge u -> v
adj.get(v).add(u);            // add this too if the road is two-way
```

A **grid** is also a graph. Each cell is a node, and it connects to its 4 neighbours (up, down, left, right). You don't build a list; you compute the neighbours with a `dirs` array.

## 5.2 BFS: Breadth-First Search

**BFS** explores in **rings**: first every node 1 step away, then every node 2 steps away, and so on. It uses a **queue** (first in, first out, like a bank queue).

**Real-life analogy:** you drop a stone in a pond. The ripple reaches the nearest points first, then the next ring. Or news spreading: you tell your friends (round 1), they tell theirs (round 2).

**The key fact:** in a graph **without weights**, BFS finds the **shortest path** (the fewest steps), because it reaches every node at the earliest possible ring.

**BFS template (memorise this):**
```java
Deque<Integer> q = new ArrayDeque<>();
boolean[] visited = new boolean[n];
q.add(start); visited[start] = true;
int steps = 0;
while (!q.isEmpty()) {
    int size = q.size();              // everything currently in the queue = one ring
    for (int k = 0; k < size; k++) {
        int u = q.poll();
        for (int v : adj.get(u)) {
            if (!visited[v]) { visited[v] = true; q.add(v); }  // mark when ADDING, not when polling
        }
    }
    steps++;
}
```

**Why mark visited when adding?** If you wait until polling, the same node can be added many times by different neighbours, which wastes time and can blow up memory.

**DFS (Depth-First Search)** goes as deep as possible down one path before backing up. It uses recursion, which is a stack under the hood. Use it for "explore everything", counting islands, or detecting cycles. Use **BFS when the question says "minimum steps / shortest / nearest"**.

---

### P16. Rotting Oranges (LC 994): asked twice

**Problem.** A grid: 0 = empty, 1 = fresh orange, 2 = rotten orange. Every minute, each rotten orange rots its 4 neighbours. How many minutes until no fresh orange is left? Return −1 if some can never rot.

**Swiggy version:** "Some dark stores (Instamart warehouses) have stock. Every hour, stock moves to neighbouring stores. When will every store have stock?"

**Intuition.** All the rotten oranges spread **at the same time**. So put **all** of them in the queue at the start. This is called **multi-source BFS**. Each BFS ring is one minute.

**Approach.**
1. Add every rotten cell to the queue and count the fresh ones.
2. Do BFS level by level. Each level rots its fresh neighbours (mark them as 2, which also marks them visited) and is one minute.
3. Stop when the queue is empty or no fresh oranges are left.
4. If any fresh orange remains → −1. Otherwise return the minutes.

```java
static int orangesRotting(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    Deque<int[]> q = new ArrayDeque<>();
    int fresh = 0;
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 2) q.add(new int[]{i, j});   // all rotten start together
            else if (grid[i][j] == 1) fresh++;
        }
    int minutes = 0;
    int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
    while (!q.isEmpty() && fresh > 0) {
        int size = q.size();                      // one "minute" = one BFS level
        for (int k = 0; k < size; k++) {
            int[] cell = q.poll();
            for (int[] d : dirs) {
                int r = cell[0] + d[0], c = cell[1] + d[1];
                if (r < 0 || c < 0 || r >= m || c >= n || grid[r][c] != 1) continue;
                grid[r][c] = 2;                   // rot it (also marks visited)
                fresh--;
                q.add(new int[]{r, c});
            }
        }
        minutes++;
    }
    return fresh == 0 ? minutes : -1;
}
```

**Dry run** on `[[2,1,1],[1,1,0],[0,1,1]]`:
- Minute 1: (0,1) and (1,0) rot.
- Minute 2: (0,2) and (1,1) rot.
- Minute 3: (2,1) rots.
- Minute 4: (2,2) rots.
- Fresh = 0 → **4**.

**Why `fresh > 0` in the while condition?** Without it you'd count one extra minute: the last ring would be processed even though it rots nothing.

**Complexity:** O(m·n) time and space.

**Follow-ups:**
- *Distance of each cell from the nearest delivery hub (LC 542 "01 Matrix"):* the same multi-source BFS.
- *What if the spread time differs per cell?* Then it's weighted, so use Dijkstra (next problem).
- *A Swiggy OA trick, "delivery zone coverage":* doing a BFS from **every** node timed out. The fix was to **reverse the edges** and do **one** BFS from the targets. If many sources point to a few targets, start from the targets.

---

### P17. Network Delay Time (LC 743): Dijkstra, asked in a 2025 OA

**Problem.** There are `n` nodes and directed edges `(u, v, w)`, meaning a signal from u reaches v in w time. A signal starts at node `k`. How long until **all** nodes receive it? Return −1 if some never do.

**Why not BFS?** BFS counts **steps**, but here the roads have different travel times. A 3-road route can be faster than a 1-road route.

**Dijkstra's algorithm, intuition.** Imagine Swiggy dispatch spreading out from a restaurant.
- Keep a "best known time" for every junction, starting at infinity, with 0 at the start.
- Always **finalise the junction with the smallest known time next**. Nothing can beat it later, because every other route is already longer and roads only add time.
- From it, update its neighbours: "Can I reach you faster through here?"

To always grab the smallest time quickly, use a **min-heap (PriorityQueue)**.

**Approach.**
1. Build the adjacency list of `(neighbour, weight)`.
2. `dist[]` = infinity, `dist[k] = 0`. Push `(k, 0)`.
3. Pop the smallest. If it's outdated (a shorter time was already found), skip it.
4. For each edge: if `dist[u] + w < dist[v]`, update and push.
5. The answer is the max of `dist`, or −1 if any is still infinity.

```java
static int networkDelayTime(int[][] times, int n, int k) {
    List<List<int[]>> adj = new ArrayList<>();
    for (int i = 0; i <= n; i++) adj.add(new ArrayList<>());
    for (int[] t : times) adj.get(t[0]).add(new int[]{t[1], t[2]});
    int[] dist = new int[n + 1];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[k] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>((x, y) -> Integer.compare(x[1], y[1]));
    pq.add(new int[]{k, 0});
    while (!pq.isEmpty()) {
        int[] top = pq.poll();
        int u = top[0], d = top[1];
        if (d > dist[u]) continue;                // stale entry, a shorter path was found already
        for (int[] e : adj.get(u)) {
            int v = e[0], w = e[1];
            if (d + w < dist[v]) {
                dist[v] = d + w;
                pq.add(new int[]{v, dist[v]});
            }
        }
    }
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == Integer.MAX_VALUE) return -1; // someone never got the signal
        ans = Math.max(ans, dist[i]);
    }
    return ans;
}
```

**Dry run.** Edges 2→1 (1), 2→3 (1), 3→4 (1), start at 2.
- pop (2,0) → dist[1] = 1, dist[3] = 1
- pop (1,1): no edges
- pop (3,1) → dist[4] = 2
- pop (4,2)
- max = **2** ✓

**Complexity:** O(E log V), where E = number of edges and V = number of nodes.

**Follow-ups:**
- *Negative weights?* Dijkstra breaks. Use Bellman-Ford (relax all edges V−1 times).
- *Print the path:* keep `parent[v] = u` whenever you update, then walk back from the target.
- *"Cheapest flight with at most K stops" (LC 787):* BFS by number of stops, or Bellman-Ford limited to K+1 rounds.

---

### P18. Currency Conversion (like LC 399): asked as a custom Swiggy problem (2024)

**Problem (the Swiggy phrasing).** You're given rates like `['USD','GBP',0.77]`, `['GBP','EUR',1.15]`, `['USD','INR',83]`. Return the rate from any currency to another. Return −1 if it's not possible.

**Intuition.** Currencies are nodes and rates are weighted edges. If 1 USD = 0.77 GBP, then 1 GBP = 1/0.77 USD, so add a **reverse edge** with weight 1/rate. To convert USD → EUR, find a path and **multiply** the rates along it: 0.77 × 1.15 = 0.8855.

**Approach.**
1. Build the graph with both directions.
2. BFS from `from`, carrying "how many units of this currency 1 unit of `from` buys".
3. When you reach `to`, return that value.
4. If the BFS ends without reaching it → −1.

```java
// Currency conversion: rates like {"USD","GBP",0.77}
static double convert(String[][] pairs, double[] rates, String from, String to) {
    Map<String, Map<String, Double>> g = new HashMap<>();
    for (int i = 0; i < pairs.length; i++) {
        String a = pairs[i][0], b = pairs[i][1];
        g.computeIfAbsent(a, x -> new HashMap<>()).put(b, rates[i]);
        g.computeIfAbsent(b, x -> new HashMap<>()).put(a, 1.0 / rates[i]); // reverse edge
    }
    if (!g.containsKey(from) || !g.containsKey(to)) return -1.0;
    Deque<String> q = new ArrayDeque<>();
    Map<String, Double> val = new HashMap<>();     // val[c] = how many c for 1 unit of 'from'
    q.add(from);
    val.put(from, 1.0);
    while (!q.isEmpty()) {
        String cur = q.poll();
        if (cur.equals(to)) return val.get(cur);
        for (Map.Entry<String, Double> e : g.get(cur).entrySet()) {
            if (val.containsKey(e.getKey())) continue;   // visited
            val.put(e.getKey(), val.get(cur) * e.getValue());
            q.add(e.getKey());
        }
    }
    return -1.0;                                   // no chain of rates connects them
}
```

**Note:** `computeIfAbsent(key, x -> new HashMap<>())` means "get the inner map for this key, creating an empty one first if it's missing". It's a handy Java idiom.

**Complexity:** O(V + E) per query.

**Follow-ups:**
- *Many queries?* Precompute with **Union-Find with weights**, or cache the results.
- *Best rate when there are several paths?* That's a "maximum product path". Take logs (log of a product = sum of logs), negate them, and run shortest path.
- *Arbitrage detection* (a cycle whose product is > 1): negative-cycle detection with Bellman-Ford on −log(rate).

**Also asked once:** **Alien Dictionary (LC 269)**, a **topological sort**. Compare each pair of adjacent words. The first differing letter gives an edge "a comes before b". Then repeatedly output letters that have no remaining incoming edges (Kahn's algorithm). Also: **detect a cycle in a dependency graph** ("Spring beans depend on each other"), which is the same topological sort. If not every node gets output, there's a cycle.

---

# Part 6: Sliding Window and Two Pointers

## 6.1 The ideas

**Two pointers:** two index variables move through an array, often from both ends towards the middle, or both left to right at different speeds. They replace a nested loop (O(n²)) with a single pass (O(n)).

**Real-life analogy:** two people searching a sorted shelf of books from both ends for two books whose prices add to ₹500. If the sum is too big, the right person moves left (to cheaper books). If it's too small, the left person moves right.

**Sliding window:** a special kind of two pointers. `left` and `right` mark a **window**, a continuous stretch of the array or string.
- Grow it by moving `right`.
- Shrink it by moving `left` when it breaks a rule.

**Real-life analogy:** a train window. As the train moves, new scenery enters on one side and old scenery leaves on the other. You never re-look at the whole view.

**Recognise it when the problem says:** "longest / shortest **substring / subarray** such that…", "at most k distinct…", "sum of a window of size k".

**Template:**
```java
int left = 0, best = 0;
for (int right = 0; right < n; right++) {
    <add s[right] to the window>;
    while (<window is invalid>) {
        <remove s[left] from the window>;
        left++;
    }
    best = Math.max(best, right - left + 1);
}
```

---

### P19. Longest Substring Without Repeating Characters (LC 3): asked twice (OA 2025)

**Problem.** The length of the longest substring with all different characters.
- `"abcabcbb"` → 3 ("abc")
- `"pwwkew"` → 3 ("wke")

**Intuition.** Grow the window to the right. When the new character already appears **inside the window**, move `left` to just after that earlier copy. Remember the last index of each character in a HashMap, so you can jump straight there.

```java
static int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
            left = lastSeen.get(c) + 1;           // jump past the earlier copy
        }
        lastSeen.put(c, right);
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

**Why `>= left`?** The earlier copy might be **outside** the window already. In "abba", when we reach the last 'a', the first 'a' (index 0) is already left of the window (left = 2). Jumping to 1 would move `left` backwards. Test "abba"; the answer is 2.

**Dry run** on "abcabcbb":
- a, b, c → window "abc", best 3
- a again (seen at 0) → left = 1, window "bca"
- b → left = 2, and so on
- best stays **3**

**Complexity:** O(n) time, O(alphabet) space.

**Follow-ups:**
- *At most k distinct characters (LC 340), asked once:* keep a count map. While `map.size() > k`, shrink from the left and remove keys whose count hits 0.
- *Count "nice" substrings* (an unspecified OA medium): usually "count subarrays with **at most** k X" − "at most k−1 X" = **exactly** k X. The asked "subarrays with at most k odd elements" uses exactly this window.

---

### P20. 3Sum (LC 15): asked once (SDE-1 2021)

**Problem.** Find all **unique** triplets that sum to 0.
`[-1,0,1,2,-1,-4]` → `[[-1,-1,2],[-1,0,1]]`.

**Intuition.**
1. **Sort** first.
2. Fix the first number `nums[i]`. Now you need two numbers in the rest that sum to `−nums[i]`.
3. In a sorted array, that's the two-people-on-a-shelf trick: if the sum is too small, move `l` right; if too big, move `r` left.

**The tricky part is duplicates.** Skip a number that's the same as the previous one, both for `i` and after finding a triplet.

```java
static List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;   // skip duplicate first numbers
        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum < 0) l++;                     // too small: need a bigger number
            else if (sum > 0) r--;                // too big: need a smaller number
            else {
                res.add(Arrays.asList(nums[i], nums[l], nums[r]));
                while (l < r && nums[l] == nums[l + 1]) l++;  // skip duplicates
                while (l < r && nums[r] == nums[r - 1]) r--;
                l++; r--;
            }
        }
    }
    return res;
}
```

**Complexity:** O(n²) time (the sort is O(n log n), which is smaller). O(1) extra space, not counting the output.

**Follow-ups:**
- *2Sum on an unsorted array, returning indexes:* use a HashMap of value → index. O(n).
- *4Sum:* fix two numbers, then two pointers, O(n³).
- *Count pairs with a given sum* (asked once): a HashMap of counts, adding `count[target − x]` for each x.

---

### P21. Trapping Rain Water (LC 42): asked 2024

**Problem.** The bar heights of an elevation map. How much rain water is trapped?
- `[0,1,0,2,1,0,1,3,2,1,2,1]` → 6
- `[4,2,0,3,2,5]` → 9

**Key fact.** The water above bar `i` = `min(tallest bar on its left, tallest bar on its right) − height[i]`. Water rises to the **shorter** of the two walls around it, like water in a bucket that spills over the lower edge.

**Simple approach:** precompute `leftMax[]` and `rightMax[]` arrays, then sum. O(n) time, O(n) space. It's fine to say this first.

**Two-pointer approach (O(1) space).**
1. Put pointers at both ends and track `leftMax` and `rightMax`.
2. If `h[l] < h[r]`: the right side has a wall at least as tall as `h[r]`, which is taller than `h[l]`. So the **left side is the limiting wall** for position `l`, and the water there is `leftMax − h[l]`. Move `l` in.
3. Otherwise, do the mirror image on the right.

```java
static int trap(int[] h) {
    int l = 0, r = h.length - 1, leftMax = 0, rightMax = 0, water = 0;
    while (l < r) {
        if (h[l] < h[r]) {                        // the left wall is the limit
            leftMax = Math.max(leftMax, h[l]);
            water += leftMax - h[l];
            l++;
        } else {                                  // the right wall is the limit
            rightMax = Math.max(rightMax, h[r]);
            water += rightMax - h[r];
            r--;
        }
    }
    return water;
}
```

**Dry run** on `[4,2,0,3,2,5]`:
- h[l] = 4 < h[r] = 5 → leftMax 4, water += 0. l = 1.
- h[1] = 2 → water += 2.
- h[2] = 0 → water += 4.
- h[3] = 3 → water += 1.
- h[4] = 2 → water += 2.
- Total **9** ✓

**Complexity:** O(n) time, O(1) space.

---

# Part 7: Binary Search

## 7.1 The idea

**Binary search** finds something in a **sorted** range by checking the middle and throwing away half each time. That's O(log n).

**Real-life analogy:** finding the word "paneer" in a paper dictionary. You open it in the middle and see "M". "P" comes after "M", so you ignore the first half. Open the middle of the second half… A million words take about 20 steps.

**The bigger idea: "binary search on the answer".** Even when there's no sorted array, if the question is "find the smallest/largest X such that some condition is true", and the condition is **monotonic** (false, false, false, true, true, true), you can binary search over the X values.

Swiggy example: "what's the minimum number of delivery partners so all orders go out within 30 minutes?" If 10 partners works, 11 also works. So binary search on the count.

**Template (find the boundary):**
```java
int lo = 0, hi = n - 1, ans = -1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;      // NOT (lo+hi)/2: that can overflow for big numbers
    if (<mid is a valid answer>) { ans = mid; <move to search for a better one>; }
    else <move the other way>;
}
```

---

### P22. First and Last Position in a Sorted Array (LC 34): Swiggy ASDE Round 1, 2026

**Problem.** In a sorted array, find the first and last index of `target`. Return [−1, −1] if it's absent. It must be O(log n).
`[5,7,7,8,8,10]`, target 8 → `[3,4]`.

**Intuition.** Normal binary search stops as soon as it finds the target. Here, when you find it, **write it down and keep searching**:
- for the **first** position, keep going **left** (there may be an earlier copy)
- for the **last** position, keep going **right**

**Why not find one copy and then scan left and right?** If the array is all 8s, the scan is O(n). Interviewers ask exactly this.

```java
static int[] searchRange(int[] nums, int target) {
    return new int[]{findEdge(nums, target, true), findEdge(nums, target, false)};
}
private static int findEdge(int[] a, int target, boolean first) {
    int lo = 0, hi = a.length - 1, ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;             // avoids overflow of (lo+hi)
        if (a[mid] == target) {
            ans = mid;                            // record, then keep looking
            if (first) hi = mid - 1; else lo = mid + 1;
        } else if (a[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return ans;
}
```

**Dry run** (finding the first 8 in `[5,7,7,8,8,10]`):
- lo 0, hi 5 → mid 2 (7 < 8) → lo 3
- mid 4 (8): ans = 4, hi = 3
- mid 3 (8): ans = 3, hi = 2
- Stop → **3**

**Complexity:** O(log n) time, O(1) space.

**Follow-ups:**
- *Count the occurrences of x:* last − first + 1.
- *Single non-duplicate in a sorted array (LC 540), asked once:* pairs sit at (even, odd) indexes before the single element, and at (odd, even) after it. Binary search on that.
- *Search a row- and column-sorted matrix (LC 240), asked once:* start at the **top-right** corner. If it's bigger than the target, go left; if smaller, go down. O(m + n).

---

### P23. Search in a Rotated Sorted Array (LC 33)

**Problem.** A sorted array was rotated at an unknown point: `[4,5,6,7,0,1,2]`. Find a target's index in O(log n).

**Intuition.** When you cut a rotated array in the middle, **at least one half is still properly sorted**. Check which one: if `a[lo] ≤ a[mid]`, the left half is sorted. If the target lies within the sorted half's range, search there. Otherwise search the other half.

```java
static int searchRotated(int[] a, int target) {
    int lo = 0, hi = a.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] == target) return mid;
        if (a[lo] <= a[mid]) {                    // left half is sorted
            if (a[lo] <= target && target < a[mid]) hi = mid - 1;
            else lo = mid + 1;
        } else {                                  // right half is sorted
            if (a[mid] < target && target <= a[hi]) lo = mid + 1;
            else hi = mid - 1;
        }
    }
    return -1;
}
```

**Dry run** (target 0):
- mid 3 (7). Left half [4..7] is sorted, and 0 isn't in it → lo = 4.
- mid 5 (1). a[4] = 0 ≤ 1, so the left half [0..1] is sorted, and 0 is in it → hi = 4.
- mid 4 → found **4**.

**Complexity:** O(log n).

**Follow-ups:** *find the minimum (LC 153)*; *with duplicates (LC 81)*: when `a[lo] == a[mid] == a[hi]` you can't tell which half is sorted, so do `lo++`, and the worst case becomes O(n).

---

### P24. Integer Nth Root: asked in a December 2025 OA

**Problem.** Given `m` and `n`, find the largest integer `x` with `xⁿ ≤ m`.
- (27, 3) → 3
- (30, 3) → 3
- (10¹⁸, 2) → 10⁹

**Intuition.** This is binary search on the answer: as x grows, xⁿ only grows, so the condition `xⁿ ≤ m` goes true, true, …, false, false. Find the last true.

**The trap is overflow.** `mid` can be huge, and `midⁿ` overflows `long` very fast. Compute the power step by step, and **stop early** as soon as it exceeds m. The check `result > m / base` means "multiplying again would pass m", and it's done without actually multiplying.

```java
// Integer Nth root: largest x with x^n <= m
static long nthRoot(long m, int n) {
    long lo = 0, hi = m, ans = 0;
    while (lo <= hi) {
        long mid = lo + (hi - lo) / 2;
        if (powCompare(mid, n, m) <= 0) { ans = mid; lo = mid + 1; }
        else hi = mid - 1;
    }
    return ans;
}
// returns -1 if base^n < m, 0 if equal, 1 if greater (stops early to avoid overflow)
private static int powCompare(long base, int n, long m) {
    long result = 1;
    for (int i = 0; i < n; i++) {
        if (base != 0 && result > m / base) return 1;
        result *= base;
    }
    return Long.compare(result, m);
}
```

**Complexity:** O(n · log m).

**Follow-ups:**
- *Return −1 if m is not a perfect nth power:* check whether `powCompare(ans, n, m) == 0`.
- *Square root with decimals:* binary search on doubles for about 100 iterations, or until `hi − lo < 1e-6`.

---

# Part 8: Heaps (Priority Queues)

## 8.1 The idea

A **heap** is a structure that always gives you the **smallest** item (a min-heap) or the **largest** (a max-heap) in O(1). Adding or removing an item costs O(log n). In Java it's `PriorityQueue`.

**Real-life analogy:** a hospital emergency room. Patients don't go in arrival order; the most serious goes first. Or Swiggy dispatch: "always assign the order that has waited the longest". When new orders arrive, you don't re-sort the whole list. The heap keeps the top one ready.

**Recognise it when the problem says:** "k largest/smallest", "repeatedly take the smallest two", "merge k sorted lists", "median of a stream", or "always process the most urgent next".

**Template:**
```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.add(x);          // O(log n)
minHeap.peek();          // smallest, O(1)
minHeap.poll();          // remove smallest, O(log n)
```

---

### P25. Min Cost to Connect Sticks (LC 1167): asked twice (SDE-1 and SDE-2)

**The Swiggy phrasing:** "Pick two numbers, replace them with their sum, and add that sum to a total. Repeat until one number is left. Minimise the total."
- [2,4,3] → 14
- [1,8,3,5] → 30

**Intuition.** Whatever you combine early gets **added again** every time its result is combined later. So big numbers should be combined as **late** as possible. That means always combining the **two smallest**. That's a greedy choice, and a min-heap gives you the two smallest instantly. (This is the same idea as **Huffman coding**, used in compression.)

```java
static int connectSticks(int[] sticks) {
    PriorityQueue<Integer> pq = new PriorityQueue<>();
    for (int s : sticks) pq.add(s);
    int cost = 0;
    while (pq.size() > 1) {
        int a = pq.poll(), b = pq.poll();         // two smallest
        cost += a + b;
        pq.add(a + b);                            // the joined stick goes back
    }
    return cost;
}
```

**Dry run** on [1,8,3,5]:
- 1+3 = 4 (cost 4) → [4,5,8]
- 4+5 = 9 (cost 13) → [8,9]
- 8+9 = 17 (cost **30**)

**Complexity:** O(n log n).

**Follow-ups:** *why does greedy work?* Each number's contribution = its value × how many times it gets merged (its depth). Putting small numbers deeper minimises the total. If they ask for more, use the exchange argument: swapping a smaller number deeper never hurts.

---

### P26. Find the Median from a Data Stream (LC 295): asked once (campus)

**Problem.** Numbers keep arriving. At any time, return the median of everything seen so far.

**Swiggy angle:** "the median delivery time of today's orders, live on a dashboard".

**Intuition.** Split the numbers into two halves:
- **low:** the smaller half, in a **max-heap** (so you can see its largest)
- **high:** the bigger half, in a **min-heap** (so you can see its smallest)

Keep their sizes equal, or `low` one bigger. Then the median is either low's top, or the average of the two tops. It's like keeping a sorted line of people and only ever looking at the two in the middle.

**Add a number** in three steps:
1. Push it into low.
2. Move low's largest to high. This guarantees everything in low ≤ everything in high.
3. If high is now bigger, move high's smallest back to low.

```java
static class MedianFinder {
    private final PriorityQueue<Integer> low = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
    private final PriorityQueue<Integer> high = new PriorityQueue<>();                          // min-heap
    void addNum(int x) {
        low.add(x);
        high.add(low.poll());                     // move the biggest of low to high
        if (high.size() > low.size()) low.add(high.poll()); // keep low >= high in size
    }
    double findMedian() {
        if (low.size() > high.size()) return low.peek();
        return (low.peek() + high.peek()) / 2.0;
    }
}
```

**Dry run.**
- add 5 → low [5] → median 5
- add 15 → low [5], high [15] → 10
- add 1 → low [5,1], high [15] → 5
- add 3 → low [3,1], high [5,15] → (3+5)/2 = **4**

**Complexity:** add is O(log n), median is O(1).

**Follow-ups:**
- *A sliding-window median:* you also need to delete, so use TreeMaps or lazy deletion.
- *If all numbers are in 0..100:* use a count array of size 101 and walk it for O(100) per median.

---

### P27. Task Scheduler (LC 621): asked as a variant (cooldown k)

**Problem.** Tasks are letters. The same task needs a gap of at least `n` time units between runs. You may idle. What is the minimum total time?
- `AAABBB`, n = 2 → 8 (`A B idle A B idle A B`)

**Intuition (math version).** The most frequent task decides the shape. If A appears `maxF` times, you need `maxF − 1` "frames" of length `n+1` (A, then n slots), plus a last frame containing every task that has the max frequency. Other tasks fill the idle slots. If there are so many tasks that no idling is needed, the answer is just the number of tasks.

```java
static int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;
    int maxF = 0, countMax = 0;
    for (int f : freq) maxF = Math.max(maxF, f);
    for (int f : freq) if (f == maxF) countMax++;
    // (maxF - 1) full "frames" of size n+1, then one last partial frame
    int frames = (maxF - 1) * (n + 1) + countMax;
    return Math.max(frames, tasks.length);
}
```

For AAABBB, n=2: (3−1)·3 + 2 = **8**.

**The Swiggy variant asked for the actual order as well as the time.** For that, simulate with a **max-heap** of remaining counts plus a **cooldown queue**:
- Each time unit, run the task with the most remaining work.
- Put it in the queue with "ready at time t + n + 1".
- Move tasks from the queue back to the heap when they're ready.
- O(total time × log 26).

Say this approach out loud if they want the sequence.

---

# Part 9: Linked Lists

## 9.1 The idea

A **linked list** is a chain of nodes. Each node holds a value and a pointer to the **next** node. There's no index access: to reach the 5th node, you walk from the head.

**Real-life analogy:** a treasure hunt. Each clue tells you where the next clue is. Or a train: each coach is hooked to the next.

**Strengths:** inserting or removing a node is O(1) **if you already have a pointer** to the spot. You just re-hook the links. (In an array you'd shift everything.)

```java
static class ListNode {
    int val; ListNode next;
    ListNode(int v) { val = v; }
}
```

**Three tricks cover almost every problem:**
1. **Dummy head:** create a fake node before the head so the first node isn't a special case.
2. **Reversing:** `prev = null, cur = head; while (cur != null) { next = cur.next; cur.next = prev; prev = cur; cur = next; }`. Draw it the first few times.
3. **Fast and slow pointers:** one moves 2 steps, the other 1. Use them to find the middle, detect cycles, or find the k-th node from the end.

**Always draw boxes and arrows on paper.** Linked-list bugs are pointer-order bugs.

---

### P28. Reverse Nodes in k-Group (LC 25): asked (Android SDE-1)

**Problem.** Reverse the list in blocks of k. A leftover block shorter than k stays as it is.
- 1→2→3→4→5, k = 2 → 2→1→4→3→5
- k = 3 → 3→2→1→4→5

**Intuition.** Process one block at a time. Remember:
- `groupPrev`: the node just before the block (it starts as the dummy)
- `kth`: the last node of the block
- `groupNext`: the node right after the block

Reverse the block, then plug it back in: `groupPrev` should now point to `kth` (the new first), and the old first node becomes the new `groupPrev` for the next block.

**The neat trick:** start the reversal with `prev = groupNext` instead of null, so the reversed block's tail automatically connects to the rest of the list.

```java
static ListNode reverseKGroup(ListNode head, int k) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode groupPrev = dummy;
    while (true) {
        ListNode kth = groupPrev;                 // find the k-th node ahead
        for (int i = 0; i < k && kth != null; i++) kth = kth.next;
        if (kth == null) break;                   // fewer than k left: stop
        ListNode groupNext = kth.next;
        // reverse the k nodes between groupPrev and groupNext
        ListNode prev = groupNext, cur = groupPrev.next;
        while (cur != groupNext) {
            ListNode nxt = cur.next;
            cur.next = prev;
            prev = cur;
            cur = nxt;
        }
        ListNode oldFirst = groupPrev.next;       // becomes last of the group
        groupPrev.next = kth;                     // kth is now first
        groupPrev = oldFirst;
    }
    return dummy.next;
}
```

**Dry run** (k = 2):
- Block [1,2]: reverse → 2→1→(3…). dummy→2. groupPrev = 1.
- Block [3,4] → 4→3→5. 1→4. groupPrev = 3.
- Next: only [5] is left → stop.

Result: **2→1→4→3→5**.

**Complexity:** O(n) time, O(1) space.

**Also asked once:**
- **Reverse the odd-indexed nodes** (ASDE): split into two lists by position, reverse one, and weave them back together.
- **Add two numbers stored as lists without reversing (LC 445):** push both lists' digits onto two stacks, then pop and add with a carry.

---

### P29. Linked List Cycle II, find where the loop starts (LC 142): asked with Floyd's

**Problem.** If the list has a loop, return the node where the loop begins. Otherwise return null.

**Intuition (Floyd's tortoise and hare).**
- **Step 1, detect:** the slow pointer moves 1 step and the fast pointer moves 2. If there's a loop, fast eventually laps slow and they meet inside the loop, like two runners on a circular track. If fast hits null, there's no loop.
- **Step 2, find the start:** put one pointer back at the head and leave the other at the meeting point. Move **both 1 step** at a time. They meet exactly at the loop's start.

**Why step 2 works, in plain terms.** Let:
- `a` = distance from the head to the loop start
- `b` = distance from the loop start to the meeting point
- `L` = the loop length

At the meeting point, slow has walked `a + b` and fast has walked twice that. Fast's extra distance, `a + b`, is a whole number of laps. So `a + b` is a multiple of `L`, which means that walking `a` more steps from the meeting point lands you back at the loop start. The head is also `a` steps from the loop start, so both pointers arrive there together.

```java
static ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {                       // they met inside the loop
            ListNode p = head;
            while (p != slow) { p = p.next; slow = slow.next; }
            return p;                             // meeting point = loop start
        }
    }
    return null;
}
```

**Complexity:** O(n) time, O(1) space. The easy alternative is a HashSet of visited nodes: O(n) space. Mention it first, then optimise.

**Follow-ups:** *loop length:* from the meeting point, walk until you're back and count the steps. *Remove the loop:* find the start, then find the node whose `next` is the start and set its `next` to null.

---

# Part 10: Hashing and Prefix Sums

## 10.1 The ideas

**Hashing:** a `HashMap` or `HashSet` gives O(1) average lookup. Use one whenever you catch yourself asking "have I seen this before?" or "how many times have I seen this?".

**Real-life analogy:** a restaurant's coat-check counter. You hand in your coat and get token #47. Later, token #47 takes them straight to your coat; they don't search every hanger. The hash function turns the key into a hanger number.

**Prefix sum:** `prefix[i]` = the sum of the first i elements. Then the **sum of any subarray** from i to j is `prefix[j+1] − prefix[i]`, in O(1).

**Real-life analogy:** your bank statement shows a running balance after each transaction. To know how much you spent between the 5th and the 20th, you don't add up every transaction. You subtract the two balances.

**The combo pattern: "count subarrays with property X".** Walk through the array keeping a running prefix value. Use a HashMap to count how many earlier prefixes would pair with the current one to make X. This turns O(n²) into O(n). Example: subarray sum = k (LC 560): `ans += count[prefix − k]`.

---

### P30. Subarray Sums Divisible by K (LC 974): asked twice

**Problem.** Count the continuous subarrays whose sum is divisible by k.
`[4,5,0,-2,-3,1]`, k = 5 → **7**.

**Intuition.** The sum of subarray (i..j) = `prefix[j] − prefix[i−1]`. It's divisible by k exactly when **both prefixes have the same remainder** mod k. For example, 17 and 7 both leave remainder 2 when divided by 5, and 17 − 7 = 10 is divisible by 5.

So count the remainders you've seen. At each step, every earlier prefix with the **same remainder** forms a valid subarray ending here.

**Two details:**
1. **Start with `count[0] = 1`.** That's the empty prefix, so subarrays starting at index 0 get counted.
2. **Negative numbers:** in Java, `-2 % 5 = -2`, not 3. Fix it with `((x % k) + k) % k`.

```java
static int subarraysDivByK(int[] nums, int k) {
    int[] count = new int[k];                     // count[r] = prefixes with remainder r
    count[0] = 1;                                 // the empty prefix
    int prefix = 0, ans = 0;
    for (int x : nums) {
        prefix = ((prefix + x) % k + k) % k;      // keep remainder non-negative
        ans += count[prefix];                     // pair with every earlier same remainder
        count[prefix]++;
    }
    return ans;
}
```

**Dry run.**
- Remainders after each element: 4, 4, 4, 2, 4, 0.
- Counting as we go: 0 + 1 + 2 + 0 + 3 + 1 = **7** ✓

**Complexity:** O(n) time, O(k) space.

**Follow-ups:**
- *Print the subarrays* (asked in an old Swiggy report): store the **list of indexes** per remainder instead of a count. Each pair gives a subarray. The output can be O(n²).
- *Subarray sum equals k (LC 560):* use a HashMap of prefix-sum counts.
- *First non-repeating character with one array of 26* (asked once): count the frequencies in pass 1, then find the first char with count 1 in pass 2.

---

# Part 11: Design: LRU Cache

### P31. LRU Cache (LC 146): "LRU cache is most asked" (ASDE Glassdoor), asked 3 times

**What a cache is.** A small, fast storage that keeps recently used data so you don't fetch it from the slow source (a database) again. You used Aerospike as a cache at Myntra, so connect this to your experience.

**Why LRU?** The cache has limited space. When it's full, something must be thrown out. **LRU = Least Recently Used**: throw out the item that hasn't been touched for the longest time.

**Real-life analogy:** the "Recently ordered" row in the Swiggy app shows your last 5 restaurants. When you order from a new one, it goes to the front, and the one you ordered from longest ago falls off the end. Ordering again from an old one moves it back to the front.

**Problem.** Build `get(key)` and `put(key, value)`, both in **O(1)**, with a fixed capacity.

**Why two structures?**
- A **HashMap** finds any key in O(1), but it has no idea which key is oldest.
- A **doubly linked list** keeps the order (most recent at the front, oldest at the back) and can remove any node in O(1) **if you have a pointer to it**. "Doubly" means each node points to both next and prev, so removing it needs no search.
- **Combined:** the map stores key → node pointer. To "touch" a key, find its node through the map, unlink it and move it to the front, all in O(1). To evict, remove the node at the back.

**Dummy head and tail nodes** mean you never deal with null at the ends.

```java
static class LRUCache {
    private class Node {
        int key, val; Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }
    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0), tail = new Node(0, 0); // dummy ends

    LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }
    private void remove(Node n) { n.prev.next = n.next; n.next.prev = n.prev; }
    private void addFront(Node n) {
        n.next = head.next; n.prev = head;
        head.next.prev = n; head.next = n;
    }
    int get(int key) {
        Node n = map.get(key);
        if (n == null) return -1;
        remove(n); addFront(n);                   // just used: move to front
        return n.val;
    }
    void put(int key, int value) {
        Node n = map.get(key);
        if (n != null) { n.val = value; remove(n); addFront(n); return; }
        if (map.size() == capacity) {             // full: throw out the least recent
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);
        }
        Node fresh = new Node(key, value);
        map.put(key, fresh);
        addFront(fresh);
    }
}
```

**Dry run** (capacity 2):
- put(1), put(2) → list: 2, 1
- get(1) → 1, and 1 moves to the front: 1, 2
- put(3) → full, so evict the back (2): 3, 1
- get(2) → −1 ✓

**Why does the node store the key?** When evicting the back node, you must also remove it from the map, and the map is keyed by key.

**Complexity:** O(1) for both operations. O(capacity) space.

**Follow-ups (these come up a lot):**
- *Use Java's built-in:* `LinkedHashMap` with `accessOrder = true` and an overridden `removeEldestEntry`. Mention it, but code the manual version unless they say otherwise.
- *Thread-safe?* Simplest: make `get`/`put` `synchronized`. Better: a `ReentrantLock`, or segment the cache into shards, each with its own lock.
- *LFU (Least Frequently Used, LC 460):* evict the least-used key. Keep a map of key → (value, freq), a map of freq → a LinkedHashSet of keys, and track `minFreq`.
- *LRU in OS page replacement* (asked in a campus report): the same idea for memory pages. Compare with FIFO and Optimal (see the CS fundamentals doc).
- *Add expiry (TTL)?* Store an expiry time in each node. On `get`, if it has expired, remove it and return −1.

---

# Part 12: Practice list and order

## 12.1 Suggested order (about 10 days, 2–3 hours a day)

| Day | Do | Why this order |
|---|---|---|
| 1 | Part 0 (Big-O, the routine, the Java toolkit) + Stack: P1, P2, P4 | Stacks are the most asked and the easiest start |
| 2 | P3, P5 + Intervals: P6, P7 | All short and very common |
| 3 | Binary search: P22, P23, P24 | P22 was in the 2026 ASDE Round 1 |
| 4 | Sliding window / two pointers: P19, P20, P21 | Very common in OAs |
| 5 | Trees: traversals, P8, P10 | Recursion warm-up before DP |
| 6 | P9 (practise until you can write it in 10 minutes) + Hashing: P30 | |
| 7 | DP: 4.1, P11, P12, P13 | Build the "4 things" habit |
| 8 | DP: P14, P15 | Classic HM-round questions |
| 9 | Graphs: P16, P17, P18 | |
| 10 | Heap: P25, P26, P27 + Linked list: P28, P29 + LRU: P31 | |
| Then | Practice list below, 3–4 problems a day; re-type P1–P31 from memory | |

**How to practise each problem:**
1. Set a 20-minute timer.
2. Talk out loud as if the interviewer is there.
3. Write the code in a plain editor first (VS Code, no autocomplete help), then run it on LeetCode.
4. If you couldn't solve it, read the solution, then **redo it 2 days later** without looking.

## 12.2 The other problems Swiggy asked (by pattern)

**Stack**
- [Valid Parenthesis String (LC 678)](https://leetcode.com/problems/valid-parenthesis-string/)
- [Longest Valid Parentheses (LC 32)](https://leetcode.com/problems/longest-valid-parentheses/)
- [Evaluate Reverse Polish Notation (LC 150)](https://leetcode.com/problems/evaluate-reverse-polish-notation/), which is the postfix problem

**Intervals / greedy**
- [Maximum Length of Pair Chain (LC 646)](https://leetcode.com/problems/maximum-length-of-pair-chain/), asked twice
- [Minimum Number of Taps to Open to Water a Garden (LC 1326)](https://leetcode.com/problems/minimum-number-of-taps-to-open-to-water-a-garden/), the "fountains" OA problem
- [Balanced Team (Codeforces 1133C)](https://codeforces.com/problemset/problem/1133/C), sort + sliding window

**Trees**
- [LCA of a BST (LC 235)](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)
- [Sum Root to Leaf Numbers (LC 129)](https://leetcode.com/problems/sum-root-to-leaf-numbers/)
- [Binary Tree Right Side View (LC 199)](https://leetcode.com/problems/binary-tree-right-side-view/), level order; "top view" was asked once

**DP**
- [Unique Paths II (LC 63)](https://leetcode.com/problems/unique-paths-ii/)
- [Coin Change (LC 322)](https://leetcode.com/problems/coin-change/)
- [Partition Equal Subset Sum (LC 416)](https://leetcode.com/problems/partition-equal-subset-sum/), the subset-sum family
- [Partition Array Into Two Arrays to Minimize Sum Difference (LC 2035)](https://leetcode.com/problems/partition-array-into-two-arrays-to-minimize-sum-difference/), hard, asked 2025
- [Maximum Subarray (LC 53)](https://leetcode.com/problems/maximum-subarray/), Kadane's algorithm
- Count binary strings without consecutive 1s ([GfG](https://www.geeksforgeeks.org/count-number-binary-strings-without-consecutive-1s/)), the Fibonacci pattern
- Building Bridges ([GfG](https://www.geeksforgeeks.org/dynamic-programming-building-bridges/)), asked twice: sort by one bank, then LIS (Longest Increasing Subsequence) on the other
- [Longest Increasing Subsequence (LC 300)](https://leetcode.com/problems/longest-increasing-subsequence/), needed for Building Bridges
- [Best Time to Buy and Sell Stock with Cooldown (LC 309)](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/), the "modified stock" problem

**Graphs**
- [Evaluate Division (LC 399)](https://leetcode.com/problems/evaluate-division/)
- [Alien Dictionary (LC 269)](https://leetcode.com/problems/alien-dictionary/) (premium; free version: [GfG](https://www.geeksforgeeks.org/problems/alien-dictionary/1))
- [Course Schedule II (LC 210)](https://leetcode.com/problems/course-schedule-ii/), topological sort practice for Alien Dictionary
- [Stepping Numbers (LC 1215)](https://leetcode.com/problems/stepping-numbers/), BFS
- [Shortest Path in Binary Matrix (LC 1091)](https://leetcode.com/problems/shortest-path-in-binary-matrix/)

**Sliding window / two pointers / arrays**
- [Longest Substring with At Most K Distinct Characters (LC 340)](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/) (or the free [LC 904 Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/))
- [Count Number of Nice Subarrays (LC 1248)](https://leetcode.com/problems/count-number-of-nice-subarrays/), the "at most k odd" OA problem
- [Next Permutation (LC 31)](https://leetcode.com/problems/next-permutation/)
- [Rotate Image (LC 48)](https://leetcode.com/problems/rotate-image/)
- [Reverse Words in a String III (LC 557)](https://leetcode.com/problems/reverse-words-in-a-string-iii/)

**Binary search**
- [Single Element in a Sorted Array (LC 540)](https://leetcode.com/problems/single-element-in-a-sorted-array/)
- [Search a 2D Matrix II (LC 240)](https://leetcode.com/problems/search-a-2d-matrix-ii/)

**Heap**
- [Task Scheduler (LC 621)](https://leetcode.com/problems/task-scheduler/)
- [Meeting Rooms II (LC 253)](https://leetcode.com/problems/meeting-rooms-ii/) (premium; the same idea as Minimum Platforms)

**Linked list**
- [Add Two Numbers II (LC 445)](https://leetcode.com/problems/add-two-numbers-ii/)

**Design**
- [LFU Cache (LC 460)](https://leetcode.com/problems/lfu-cache/), the LRU follow-up

## 12.3 Fifteen similar problems for extra practice (Swiggy-style)

These weren't in the reports, but they use the same patterns and often appear in Indian product-company rounds. Do them after the lists above.

| # | Problem | Pattern |
|---|---|---|
| 1 | [Daily Temperatures (LC 739)](https://leetcode.com/problems/daily-temperatures/) | Monotonic stack |
| 2 | [Largest Rectangle in Histogram (LC 84)](https://leetcode.com/problems/largest-rectangle-in-histogram/) | Monotonic stack (harder) |
| 3 | [Insert Interval (LC 57)](https://leetcode.com/problems/insert-interval/) | Intervals |
| 4 | [Non-overlapping Intervals (LC 435)](https://leetcode.com/problems/non-overlapping-intervals/) | Intervals greedy |
| 5 | [Binary Tree Level Order Traversal (LC 102)](https://leetcode.com/problems/binary-tree-level-order-traversal/) | BFS on a tree |
| 6 | [Diameter of Binary Tree (LC 543)](https://leetcode.com/problems/diameter-of-binary-tree/) | Tree recursion |
| 7 | [House Robber (LC 198)](https://leetcode.com/problems/house-robber/) | 1D DP |
| 8 | [Longest Common Subsequence (LC 1143)](https://leetcode.com/problems/longest-common-subsequence/) | 2D DP like Edit Distance |
| 9 | [Number of Islands (LC 200)](https://leetcode.com/problems/number-of-islands/) | Grid DFS/BFS |
| 10 | [Course Schedule (LC 207)](https://leetcode.com/problems/course-schedule/) | Cycle detection / topological sort |
| 11 | [Minimum Window Substring (LC 76)](https://leetcode.com/problems/minimum-window-substring/) | Sliding window (hard) |
| 12 | [Koko Eating Bananas (LC 875)](https://leetcode.com/problems/koko-eating-bananas/) | Binary search on the answer |
| 13 | [Top K Frequent Elements (LC 347)](https://leetcode.com/problems/top-k-frequent-elements/) | Heap / bucket |
| 14 | [Merge k Sorted Lists (LC 23)](https://leetcode.com/problems/merge-k-sorted-lists/) | Heap + linked list |
| 15 | [Subarray Sum Equals K (LC 560)](https://leetcode.com/problems/subarray-sum-equals-k/) | Prefix sum + HashMap |

## 12.4 Before the interview: a one-page checklist

- [ ] I can explain Big-O, and I know which complexity fits which input size.
- [ ] I can write BFS, binary search (boundary version), the sliding window and the monotonic stack templates without looking.
- [ ] I can do the "4 things" (state, transition, base, answer) for any DP problem out loud.
- [ ] I can type Valid Parentheses, Merge Intervals, First/Last Position, the stack using an array, and LRU Cache in under 10 minutes each.
- [ ] I can explain *why* the monotonic stack is O(n), *why* Dijkstra needs a heap, and *why* the coin-change loop order matters.
- [ ] For every solution I say the complexity before the interviewer asks.
- [ ] I've practised in **Java** (2026 rounds were Java/Go only). If I'm allowed Go, I know Go's `slice`, `map` and `container/heap` basics.
