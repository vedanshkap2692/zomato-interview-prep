# Java and Go: The Questions for Round 1

> **Your round:** 45 minutes of coding and problem solving with data structures (PSDS), then 15 minutes of LLD. The invite says questions will cover **Java and Golang**.
> **What this file does:** the short version. These are the Java and Go questions most likely to come up, each explained simply with an answer you can say out loud. Read it in about 1.5 hours.
> **Need more depth on Java?** It's in `05_Java_and_Spring_From_Zero.md`. **LLD for the 15 minutes** is in `08_LLD_15_Minute_Round.md`.
> Every code snippet here was compiled and run (Java 21, Go 1.26).

---

## How to use this file

There are three kinds of questions you can get:

1. **"Solve this problem in Java or Go."** You need to be fast with each language's toolkit. See Part 1.
2. **Quick concept questions** like "what is a goroutine?" or "interface vs abstract class?". See Parts 2 and 3.
3. **"Java vs Go?"** You used Go at Myntra and Kotlin/Spring (which runs on the JVM, Java's runtime) at Atlassian, so expect it. See Part 4.

**Which language should you code DSA in?** Pick the one you're faster in, and stick with it. Java has a richer toolkit for DSA: a ready-made heap, sorted maps and deques. Go has less, but you used it daily at Myntra. If you aren't sure, use **Java** for DSA and be ready to *talk* Go.

---

## Part 1: The DSA toolkit side by side

At a restaurant, the same dish can be cooked in two kitchens with different tools. The recipe (the algorithm) stays the same; only the utensils change. Here are the utensils.

| What you need | Java | Go |
|---|---|---|
| Growing array | `List<Integer> a = new ArrayList<>(); a.add(5); a.get(0);` | `a := []int{}; a = append(a, 5); a[0]` |
| Hash map | `Map<String,Integer> m = new HashMap<>(); m.put("a",1); m.getOrDefault("a",0);` | `m := map[string]int{}; m["a"] = 1; v, ok := m["a"]` |
| Set | `Set<Integer> s = new HashSet<>(); s.add(3); s.contains(3);` | `s := map[int]bool{}; s[3] = true; s[3]` (Go has no set type) |
| Stack | `Deque<Integer> st = new ArrayDeque<>(); st.push(1); st.pop(); st.peek();` | slice: push `st = append(st, 1)`, top `st[len(st)-1]`, pop `st = st[:len(st)-1]` |
| Queue | `Deque<Integer> q = new ArrayDeque<>(); q.offer(1); q.poll();` | slice: push `q = append(q, 1)`, pop `x := q[0]; q = q[1:]` |
| Heap | `PriorityQueue<Integer> pq = new PriorityQueue<>();` (min-heap) | `container/heap`: you write 5 methods (shown below) |
| Sorted map | `TreeMap` (`floorKey`, `ceilingKey`) | Not built in. Sort the keys, then binary search with `sort.Search`. |
| Sort | `Arrays.sort(arr)` / `list.sort((x, y) -> x[0] - y[0])` | `sort.Ints(a)` / `sort.Slice(a, func(i, j int) bool { return a[i] < a[j] })` |
| String builder | `StringBuilder sb = new StringBuilder(); sb.append('a');` | `var sb strings.Builder; sb.WriteByte('a')` |
| Max int | `Integer.MAX_VALUE` | `math.MaxInt` |

### The one Go thing you must practise: a heap

Java hands you `PriorityQueue`. Go makes you describe your heap to the `container/heap` package by writing 5 small methods. Learn this once:

```go
package main

import (
	"container/heap"
	"fmt"
)

// MinHeap is a slice of ints that container/heap will manage.
type MinHeap []int

func (h MinHeap) Len() int           { return len(h) }
func (h MinHeap) Less(i, j int) bool { return h[i] < h[j] } // use > for a max-heap
func (h MinHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *MinHeap) Push(x any)        { *h = append(*h, x.(int)) }
func (h *MinHeap) Pop() any {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[:n-1]
	return x
}

func main() {
	h := &MinHeap{5, 2, 8}
	heap.Init(h)
	heap.Push(h, 1)
	for h.Len() > 0 {
		fmt.Print(heap.Pop(h), " ") // 1 2 5 8
	}
	fmt.Println()
}
```

**Important:** always call `heap.Push(h, x)` and `heap.Pop(h)` (the package functions), never `h.Push(x)` directly. The package function is what keeps the heap ordered.

The same thing in Java is a single line:

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();               // min-heap
PriorityQueue<Integer> maxPq = new PriorityQueue<>(Collections.reverseOrder());
PriorityQueue<int[]> byDist = new PriorityQueue<>((a, b) -> a[1] - b[1]); // sort pairs by 2nd value
```

### Traps that cost people time in the round

**Java**
- `Integer` objects compared with `==` compare *addresses*, not values. `list.get(0) == list.get(1)` can be false for 1000 and 1000. Use `.equals()`.
- `a[1] - b[1]` in a comparator can overflow for huge numbers. Use `Integer.compare(a[1], b[1])`.
- String `+` inside a loop is slow, because each `+` builds a new string. Use `StringBuilder`.

**Go**
- A map read of a missing key returns the **zero value** (0, "", false) and does not error. Use `v, ok := m[k]` when "missing" and "zero" mean different things.
- Map iteration order is **random on purpose**. You learned this at Myntra with the CASE-clause bug. If order matters, sort the keys first.
- `append` may create a new underlying array (Part 3, Q6). Always write `a = append(a, x)`, never just `append(a, x)`.
- Go has no `while`. Write `for cond { }`. There's no ternary (`? :`) either, so use if/else.

---

## Part 2: Java questions

Short answers here; full explanations in `05`.

**Q1. JDK vs JRE vs JVM?**
> "The JVM is the engine that runs Java bytecode. The JRE is the JVM plus the standard libraries, which is what you need to *run* Java. The JDK is the JRE plus the tools to *build* Java: the compiler `javac` and the debugger. I write code with the JDK, javac turns it into bytecode, and the JVM runs that bytecode on any OS."

*Real-life:* a recipe written in one standard format (bytecode) can be cooked in any kitchen that has the standard stove (a JVM).

**Q2. Interface vs abstract class?** (asked at Swiggy, Dec 2025)
> "An interface is a contract: 'anything that is a `PaymentMethod` must have `pay()`'. A class can implement many interfaces. An abstract class is a half-built parent: it can hold state (fields), constructors and shared code, but a class can extend only one. I use an interface for a capability shared by unrelated things, like `Notifier` for SMS and email. I use an abstract class when subclasses really share code and state, like a base `Vehicle` with a `numberPlate` field."

**Q3. Checked vs unchecked exceptions?** (asked at Swiggy)
> "Checked exceptions, like `IOException`, are problems outside your control that the compiler forces you to handle or declare with `throws`. Unchecked exceptions are `RuntimeException`s like `NullPointerException`: usually bugs, and the compiler doesn't force handling."

**Q4. `final`, `finally`, `finalize`?**
> "`final` means can't change: a final variable can't be reassigned, a final method can't be overridden, a final class can't be extended. `finally` is a block that always runs after try/catch, used for cleanup. `finalize` is an old, deprecated method the garbage collector might call. Don't use it."

**Q5. `static`?**
> "A static member belongs to the class, not to each object. Every object shares it. For example, a static counter of how many orders were created."

**Q6. `==` vs `.equals()`?**
> "`==` checks if two references point to the same object. `.equals()` checks if the contents are equal. For strings and wrapper types always use `.equals()`."

**Q7. Why must you override `hashCode()` when you override `equals()`?**
> "HashMap first uses `hashCode()` to pick a bucket, then `equals()` inside the bucket. If two equal objects had different hash codes, they would land in different buckets, and `map.get()` would fail to find a key that is 'equal'."

**Q8. How does HashMap work?** (the most common Java internals question)
> "It's an array of buckets. To store a key, Java calls `hashCode()`, mixes the bits, and picks a bucket index. If two keys land in the same bucket (a collision), they form a linked list; when a bucket gets more than 8 entries it turns into a red-black tree, so lookups stay fast. When the map is 75% full (the load factor), it doubles the array and redistributes entries. Average get/put is O(1)."

*Real-life:* a post office with numbered pigeonholes. The pin code decides the pigeonhole, and inside it you check names.

**Q9. HashMap vs ConcurrentHashMap?**
> "HashMap isn't thread-safe; two threads writing at once can corrupt it. `Hashtable` locks the whole map for every call, which is slow. `ConcurrentHashMap` locks only small parts, so many threads can read and write at once safely. In a multi-threaded service I use ConcurrentHashMap."

**Q10. String vs StringBuilder; why is String immutable?**
> "A String can never change; any 'change' creates a new String. That makes strings safe to share between threads, safe as HashMap keys, and allows the string pool to reuse them. StringBuilder is mutable and fast for building strings in a loop."

**Q11. Stack vs heap memory?**
> "Each thread has its own stack, holding method calls and local variables; it's cleaned up automatically when a method returns. Objects live on the heap, which all threads share. The garbage collector cleans the heap."

**Q12. What is garbage collection?**
> "The JVM automatically frees objects nobody refers to any more. Most objects die young, so the heap is split into a young generation that is cleaned often and cheaply, and an old generation for long-lived objects that is cleaned rarely. The idea is mark (find what's reachable) and sweep (free the rest)."

**Q13. Thread vs Runnable; what's a thread pool?**
> "Runnable is the *task*, Thread is the *worker*. Creating a new thread per task is expensive, so we use a thread pool via `ExecutorService`: a fixed number of workers pick tasks from a queue. That's exactly the worker pool I used at Atlassian with SQS."

**Q14. `synchronized` vs `volatile`?**
> "`synchronized` lets only one thread at a time run a block, which fixes race conditions like two threads doing `count++`. `volatile` only guarantees visibility: when one thread writes a variable, others immediately see the new value. It doesn't make `count++` safe, because `count++` is three steps: read, add, write. For a safe counter use `AtomicInteger` or `synchronized`."

**Q15. What is a deadlock, in Java terms?**
> "Thread 1 holds lock A and waits for B; thread 2 holds B and waits for A. Both wait forever. To prevent it, always take locks in the same order, or use `tryLock` with a timeout."

**Q16. Java 8 features?**
> "Lambdas (short functions), streams (`list.stream().filter(...).map(...).collect(...)`), `Optional` to avoid nulls, and default methods in interfaces."

**Q17. Overloading vs overriding?**
> "Overloading is the same method name with different parameters in one class, decided at compile time. Overriding is a subclass replacing a parent's method with the same signature, decided at runtime. That's runtime polymorphism."

---

## Part 3: Go questions

Go (or Golang) is Google's language for backend services. It was built to be **simple, fast to compile, and great at doing many things at once**. Swiggy uses it for backend services; you used it at Myntra.

### The basics

**Q1. What is Go, and why do companies use it for backends?**
> "Go is a compiled, statically typed language. It compiles to one binary with no runtime to install, starts fast, and has concurrency built in with goroutines and channels. The syntax is small, so codebases stay readable. It's good for I/O-heavy services, which call DBs, caches and other services, and that's what most backend work is. At Myntra our Ratings & Reviews service was in Go."

**Q2. Does Go have classes and inheritance?**
> "No. Go has **structs** (data) and **methods** attached to them. There's no inheritance. Instead Go uses **composition**: you *embed* one struct in another, and **interfaces** for polymorphism."

```go
type Restaurant struct {
	Name   string
	Rating float64
}

// A method with a pointer receiver: it can modify r.
func (r *Restaurant) UpdateRating(newRating float64) { r.Rating = newRating }

// Embedding: CloudKitchen "has" a Restaurant, and gets its fields and methods.
type CloudKitchen struct {
	Restaurant
	Brands []string
}
```

**Q3. How are interfaces different in Go?**
> "They're satisfied **implicitly**. In Java you write `implements PaymentMethod`. In Go, if your type has the methods the interface lists, it *automatically* satisfies it, with no keyword. That keeps packages loosely connected: you can define a small interface where you *use* it."

```go
type Notifier interface {
	Send(msg string) error
}

type SMS struct{}

func (SMS) Send(msg string) error { fmt.Println("SMS:", msg); return nil }

// SMS is a Notifier just because it has Send(). No "implements" needed.
var n Notifier = SMS{}
```

*Real-life:* Swiggy doesn't need a certificate saying "I am a delivery partner". Anyone with a bike, a phone and the app *can* deliver. If you have the abilities, you fit the role.

**Q4. Value receiver vs pointer receiver?**
> "A value receiver `func (r Restaurant)` gets a **copy**, so changes are lost. A pointer receiver `func (r *Restaurant)` works on the original. Use a pointer receiver when the method changes the struct or when the struct is big, so you don't copy it. Keep it consistent across a type's methods."

**Q5. How does Go handle errors? Why not exceptions?**
> "Errors are normal return values: a function returns `(result, error)` and the caller checks `if err != nil`. It's verbose, but every failure point is visible in the code; nothing jumps silently up the stack. You can wrap errors with context using `fmt.Errorf("fetch rating: %w", err)` and check them later with `errors.Is`. For truly broken situations there's `panic`, and `recover` can catch it inside a deferred function, but that's rare."

```go
func divide(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("division by zero")
	}
	return a / b, nil
}

res, err := divide(10, 0)
if err != nil {
	fmt.Println("error:", err) // handle it right here
}
```

**Q6. Array vs slice? What happens on `append`?** (a classic)
> "An array has a fixed size that's part of its type: `[3]int`. A slice is a flexible *view* over an array. It's a small header with three things: a pointer to the array, a length, and a capacity. When you `append` and there's spare capacity, it writes into the same array. When it's full, Go allocates a bigger array (roughly double for small slices), copies everything over, and returns a slice pointing to the new one. That's why you must write `a = append(a, x)`."

**The sharing trap:** two slices can share one array, so changing one changes the other.
```go
a := []int{1, 2, 3, 4}
b := a[0:2]      // b views a's array: [1 2]
b[0] = 99
fmt.Println(a)   // [99 2 3 4]  ← a changed too!
```
*Real-life:* two people looking through different windows at the same room. Move the furniture and both see it.

**Q7. `make` vs `new`?**
> "`make` is only for slices, maps and channels. It creates them *ready to use*: `make(map[string]int)`, `make([]int, 0, 10)`, `make(chan int)`. `new(T)` allocates a zeroed T and returns a pointer `*T`. In practice you mostly use `make` and `&Struct{}`."

**Q8. What does `defer` do?**
> "`defer` schedules a call to run when the surrounding function returns, even if it returns early or panics. It's used for cleanup: `defer file.Close()`, `defer mu.Unlock()`. Multiple defers run in reverse order (last in, first out). The arguments are evaluated when the defer line runs, not at the end."

*Real-life:* when you enter a kitchen, you put a note on the door: "turn off the gas when leaving". Whichever way you leave, the note gets followed.

**Q9. Are Go maps safe for concurrent use?**
> "No. If two goroutines write to a map at the same time, the program crashes with 'fatal error: concurrent map writes'. Protect it with a `sync.Mutex` (or `RWMutex` if reads dominate), or use `sync.Map` for special cases like caches written once and read many times."

**Q10. What's the nil interface trap?**
> "An interface value holds two things: a type and a value. It's `nil` only if *both* are nil. If you return a nil pointer of a concrete type, like `var e *MyErr = nil; return e`, as an `error`, the interface now has a type, so `err != nil` is **true**. Fix: return a literal `nil` when there's no error."

### Concurrency (the most important Go topic)

**Q11. What is a goroutine? How is it different from a thread?**
> "A goroutine is a function running concurrently, started with `go f()`. It's much lighter than an OS thread: it starts with about a 2 KB stack that grows as needed, while a thread typically reserves around 1 MB. So you can run thousands, even millions. Go's runtime scheduler multiplexes many goroutines onto a few OS threads; this is called M:N scheduling. When a goroutine blocks on I/O, the scheduler runs another one on that thread."

*Real-life:* OS threads are full-time chefs: expensive to hire, and you can only have a few. Goroutines are order tickets. A few chefs (threads) work through thousands of tickets (goroutines), and when one dish is waiting in the oven (I/O), the chef picks up the next ticket.

**Q12. What is a channel? Buffered vs unbuffered?**
> "A channel is a typed pipe that goroutines use to pass data safely. Go's motto: *don't communicate by sharing memory; share memory by communicating.* An **unbuffered** channel (`make(chan int)`) blocks the sender until a receiver takes the value, like a hand-to-hand delivery. A **buffered** channel (`make(chan int, 10)`) lets the sender drop up to 10 values without waiting, like a restaurant's pickup shelf with 10 slots; the sender only waits when the shelf is full."

**Channel rules interviewers love:**

| Action | Open channel | Closed channel | nil channel |
|---|---|---|---|
| Send | waits if no space | **panic** | blocks forever |
| Receive | waits if empty | returns zero value immediately (`v, ok := <-ch`, `ok=false`) | blocks forever |
| Close | ok | **panic** | **panic** |

Rule of thumb: **only the sender closes** a channel, and only to say "no more values". `for v := range ch` stops when the channel is closed.

**Q13. What is `select`?**
> "`select` waits on several channel operations and runs whichever is ready first. It's commonly used with a timeout."

```go
select {
case res := <-resultCh:
	fmt.Println("got", res)
case <-time.After(2 * time.Second):
	fmt.Println("timeout") // the call took too long: give up
}
```

**Q14. How do you wait for many goroutines to finish?** Use `sync.WaitGroup`.

```go
var wg sync.WaitGroup
for i := 1; i <= 3; i++ {
	wg.Add(1)          // one more worker to wait for
	go func(id int) {
		defer wg.Done() // tell the WaitGroup this one finished
		fmt.Println("worker", id, "done")
	}(i)
}
wg.Wait() // block until all three called Done
```

**Q15. What's a race condition and how do you fix it in Go?**
> "When two goroutines touch the same variable and at least one writes, the result depends on timing. For example, two goroutines doing `count++` can lose updates. Fixes: a `sync.Mutex` around the update, the `sync/atomic` package for simple counters, or a channel so only one goroutine owns the data. Go also has a race detector: run `go test -race` or `go run -race`."

```go
var mu sync.Mutex
count := 0
// inside each goroutine:
mu.Lock()
count++
mu.Unlock()
```

**Q16. Write a worker pool in Go.** (very likely, since you built worker pools at Atlassian)

Think of a Swiggy kitchen: orders come in on a rail (the `jobs` channel), 3 chefs (workers) take tickets, and finished dishes go on the pass (the `results` channel).

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, results chan<- string, wg *sync.WaitGroup) {
	defer wg.Done()
	for orderID := range jobs { // keeps taking jobs until the channel is closed
		results <- fmt.Sprintf("worker %d cooked order %d", id, orderID)
	}
}

func main() {
	jobs := make(chan int, 10)
	results := make(chan string, 10)
	var wg sync.WaitGroup

	for w := 1; w <= 3; w++ { // start 3 workers
		wg.Add(1)
		go worker(w, jobs, results, &wg)
	}
	for order := 1; order <= 5; order++ { // send 5 jobs
		jobs <- order
	}
	close(jobs) // no more jobs: workers' range loops will end

	go func() { wg.Wait(); close(results) }() // close results once all workers finish

	count := 0
	for r := range results {
		fmt.Println(r)
		count++
	}
	fmt.Println("total:", count) // total: 5
}
```

Things to *say* while writing it:
- `<-chan int` means "receive-only" and `chan<- string` means "send-only". The compiler stops a worker from sending on `jobs` by mistake.
- Closing `jobs` is how the workers learn to stop.
- A separate goroutine closes `results` after `wg.Wait()`. If `main` waited first, the workers could get stuck sending into a full `results` channel: a deadlock.
- The number of workers caps how much load you put on a downstream service. That's the same reason you used a rate limiter at Myntra.

**Q17. What is `context` and why does every Go backend function take one?**
> "`context.Context` carries a deadline, a cancellation signal and request-scoped values through a chain of calls. If a user's request times out or they close the app, cancelling the context tells every goroutine and DB call working for that request to stop, so we don't waste resources. Functions take `ctx` as their first parameter and check `ctx.Done()`."

```go
ctx, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
defer cancel()
rating, err := fetchRating(ctx, styleID) // gives up if it takes > 500 ms
```

**Q18. What is a goroutine leak?**
> "A goroutine that never ends, usually because it's blocked forever on a channel nobody will send to or receive from. Leaked goroutines pile up and eat memory. Prevent it by always giving goroutines a way to exit: close channels, use `context` cancellation, use buffered channels where a result might be abandoned."

**Q19. How does Go's garbage collector differ from Java's?**
> "Go's GC is concurrent mark-and-sweep (tri-colour). It runs alongside your program, aiming for very short pauses, and it isn't split into generations like Java's main collectors. Java gives you several collectors and lots of tuning knobs. Go keeps it simple with essentially one knob, `GOGC`."

**Q20. `sync.Once`?**
> "It runs a function exactly once, even if many goroutines call it. It's the Go way to build a thread-safe Singleton, such as a single DB connection pool."

```go
var (
	once     sync.Once
	instance *DB
)

func GetDB() *DB {
	once.Do(func() { instance = connect() })
	return instance
}
```

---

## Part 4: Java vs Go (expect at least one comparison question)

| | Java | Go |
|---|---|---|
| How it runs | Compiled to bytecode, run by the JVM (JIT makes it fast after warm-up) | Compiled straight to a native binary; starts instantly |
| OOP | Classes, inheritance, `implements` | Structs + methods, embedding (composition), implicit interfaces |
| Errors | Exceptions (try/catch, checked and unchecked) | Errors as return values (`if err != nil`) |
| Concurrency | OS threads, thread pools (`ExecutorService`), `synchronized`; virtual threads since Java 21 | Goroutines + channels built into the language |
| Generics | Yes, for a long time | Yes, since Go 1.18 (2022) |
| Memory | Garbage collected; many collectors and tuning options | Garbage collected; one low-pause concurrent collector |
| Ecosystem | Huge: Spring Boot, mature libraries | Smaller but strong for cloud tools: Docker and Kubernetes are written in Go |
| Typical use | Large enterprise services, Android | Microservices, infrastructure, high-concurrency I/O services |

**"Which would you pick for a new Swiggy service?"** There isn't a single right answer; the point is to show you understand the trade-off:
> "For a high-throughput, I/O-heavy service, like one that fans out to a cache and a few downstream APIs per request, I'd lean towards Go. Goroutines make concurrent calls cheap, the binary is small, and startup is fast, which helps autoscaling. If the team already has strong Spring Boot tooling, or the domain has lots of complex business rules where Java's ecosystem helps, Java is a fine choice. At Myntra the service was in Go; at Atlassian it was Kotlin on Spring Boot, and both worked well for their jobs."

---

## Part 5: Last-minute checklist

- [ ] I can write a stack, a queue, a map counter and a sort with a custom comparator in my DSA language without looking anything up.
- [ ] If coding in Go: I can write the `container/heap` boilerplate from memory.
- [ ] Java: interface vs abstract class, checked vs unchecked, `final`/`static`, `==` vs `equals`, how HashMap works, `synchronized` vs `volatile`.
- [ ] Go: goroutine vs thread, buffered vs unbuffered channels (plus the closed/nil table), `select`, `WaitGroup`, `Mutex`, `context`, `defer`, slices and `append`, implicit interfaces, `if err != nil`.
- [ ] I can write a worker pool in Go and explain each line.
- [ ] I can tell my Myntra Go map-order bug story in 60 seconds.
- [ ] I can say "Java vs Go" in 4–5 sentences, with a reason.
