# Java Interview Preparation — Complete Questions & Answers

> Covers **Core Java, Strings, Collections, Exceptions, Multithreading, Java 8–21, JVM/GC, Design Patterns, Spring Boot**, plus all **Scenarios & Coding Round Solutions** in Java.

---

## 1. Core Java & OOP

### Concepts

#### 1.1 JDK vs JRE vs JVM?
- **JVM (Java Virtual Machine):** Abstract machine that executes Java bytecode (`.class` files), handles memory management (Heap/Stack/GC), and makes Java platform-independent ("Write Once, Run Anywhere"). *Note: JVM itself is platform-specific.*
- **JRE (Java Runtime Environment):** `JVM + Core Class Libraries` (`java.lang`, `java.util`, etc.) needed to **run** Java applications.
- **JDK (Java Development Kit):** `JRE + Development Tools` (`javac` compiler, `jdb` debugger, `jar`, `javadoc`, `jshell`) needed to **develop & compile** Java applications.
  *(Note: From Java 11+, Oracle stopped distributing a separate JRE — you ship JDK or a custom runtime built with `jlink`.)*

---

#### 1.2 `==` vs `.equals()`?
- **`==`:** Compares **primitive values** (e.g., `5 == 5` is `true`) OR **memory references** (whether two reference variables point to the exact same object on the heap).
- **`.equals()`:** Method defined in `Object` (default implementation uses `==`), meant to be overridden to check **logical/content equality** (e.g., `String`, `Integer`, `List`, custom classes).
```java
String a = new String("java");
String b = new String("java");
System.out.println(a == b);       // false (different heap objects)
System.out.println(a.equals(b));  // true  (same characters)
```

---

#### 1.3 Abstract class vs Interface (Java 8+ mein `default`/`static` methods ke saath)?
| Feature | Abstract Class | Interface |
|---|---|---|
| **Purpose** | "Is-a" base template with shared state | Contract / capability ("can-do", e.g., `Comparable`, `Serializable`) |
| **Fields** | Can have instance variables (mutable, `private`, `protected`, etc.) | Only `public static final` constants |
| **Constructors** | Yes (called via `super()` from subclass) | No |
| **Methods** | Abstract + concrete methods | Abstract, `default` (Java 8+), `static` (Java 8+), `private` helper methods (Java 9+) |
| **Multiple Inheritance** | A class can extend **only 1** abstract class | A class can implement **multiple** interfaces |

*(If two implemented interfaces have the same `default` method signature, the implementing class **must** override it and can pick one via `InterfaceA.super.method()`.)*

---

#### 1.4 Method Overloading vs Method Overriding?
- **Method Overloading (Compile-time polymorphism):** Same method name in the same class/hierarchy with **different parameter list** (count, type, or order). Return type alone cannot overload.
- **Method Overriding (Runtime polymorphism):** Subclass provides a specific implementation for an instance method already defined in its superclass with the **same signature** (return type can be covariant, access modifier cannot be more restrictive, cannot throw broader checked exceptions).
  *(Note: `static`, `private`, and `final` methods cannot be overridden — `static` methods are **hidden**, not overridden.)*

---

#### 1.5 `static`, `final`, `finally`, `finalize()` ka difference?
- **`static`:** Belongs to the **class**, not instances (shared variable, utility method without `this`, static initializer block, nested static class).
- **`final`:**
  - `final` variable → cannot be reassigned (constant reference).
  - `final` method → cannot be overridden.
  - `final` class → cannot be subclassed (e.g., `String`, `Integer`).
- **`finally`:** Block after `try`/`catch` that executes for cleanup (whether an exception occurred or not, except `System.exit()` or JVM crash).
- **`finalize()`:** Deprecated (removed/deprecated for removal since Java 9/18) `Object` method once called by GC before reclaiming an object. Unpredictable and dangerous — use **`try-with-resources`** (`AutoCloseable`) or **`Cleaner`** instead.

---

#### 1.6 Pass by value ya pass by reference?
**Java is ALWAYS strictly pass-by-value.**
- For **primitives**, a copy of the value is passed.
- For **objects**, a **copy of the reference (pointer)** is passed by value.
  - You *can* mutate the object's internal fields through the copied reference (`user.setName("A")`).
  - You *cannot* reassign the caller's original variable to a brand-new object (`user = new User(...)` inside the method has no effect outside).

---

#### 1.7 Immutable class kaise banate hain?
1. Declare the class **`final`** (so subclasses can't override methods to add mutability).
2. Make all fields **`private final`**.
3. Do **not** provide setter methods.
4. Initialize all fields via constructor using **deep / defensive copies** for mutable inputs (like `List`, `Date`, arrays).
5. Return **defensive copies** or unmodifiable views (`List.copyOf()`, `Collections.unmodifiableList()`) from getters.
*(Java 16+ `record` automatically creates shallowly immutable data carriers—still defensively copy mutable components like `List`/`Date` in the compact constructor!)*

---

#### 1.8 `equals()` aur `hashCode()` contract kya hai?
1. **If `a.equals(b)` is `true`, then `a.hashCode() == b.hashCode()` MUST be `true`.**
2. **If `a.hashCode() == b.hashCode()`, `a.equals(b)` is NOT required to be `true`** (hash collision is legal).
3. Consistency: If fields used in `equals`/`hashCode` don't change, `hashCode()` must return the same value within a single execution.
- **Why it matters:** If you override `equals()` without `hashCode()`, two equal objects get different hash buckets in `HashMap`/`HashSet`, breaking `get()` and `contains()`!

---

### Scenarios

- **Q: Tumne `Employee` class ko `HashMap` ka key banaya, lekin `get()` `null` return kar raha hai. Kyun?**
  **A:** Either `equals()` and `hashCode()` were not overridden (so `HashMap` uses `Object`'s reference identity/hash), **OR** `Employee` was mutable and a field used in `hashCode()` changed after `put()` (so `get()` looks in the wrong bucket).

- **Q: Ek class ko thread-safe immutable banana hai jisme `List` field hai.**
  **A:** Make the class `final`, fields `private final`, take a defensive copy in constructor (`List.copyOf(items)`), and return it directly (already unmodifiable, rejects `null`s):
  ```java
  public final class Order {
      private final List<String> items;
      public Order(List<String> items) {
          this.items = List.copyOf(items); // defensive unmodifiable copy (Java 10+)
      }
      public List<String> getItems() {
          return items; // already immutable & thread-safe
      }
  }
  ```

---

## 2. String

### Concepts

#### 2.1 `String` immutable kyun hai?
1. **String Pool sharing:** Multiple variables point to the same literal in the String Constant Pool; if one mutated it, all others would corrupt silently.
2. **Thread safety:** Immutable objects are inherently thread-safe without synchronization.
3. **Security:** Class names, DB URLs, file paths, and network hosts passed as `String` cannot be tampered with after validation.
4. **HashCode caching:** `String` caches its `hashCode` once computed, making it super fast as a `HashMap`/`HashSet` key.

---

#### 2.2 String pool kya hai? `new String("a")` vs `"a"`?
- **String Constant Pool (SCP):** A special memory region inside the **Heap** (moved from PermGen to Heap in Java 7) that stores unique string literals to save memory.
- **`String s1 = "a";`**: Checks SCP. If `"a"` exists, reuses the reference; otherwise creates 1 object in SCP.
- **`String s2 = new String("a");`**: Always allocates **a new `String` object on the regular Heap** (and puts `"a"` in SCP if not already there — so 1 or 2 objects). `s1 == s2` is `false`.

---

#### 2.3 `String` vs `StringBuilder` vs `StringBuffer`?
| Feature | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| **Mutability** | Immutable | Mutable | Mutable |
| **Thread Safety** | Yes (immutable) | **No** (not synchronized) | **Yes** (`synchronized` methods) |
| **Performance** | Slow for repeated modifications | **Fastest** | Slower than `StringBuilder` due to lock overhead |
| **Use case** | Constants, map keys | Single-threaded loops/concatenation | Legacy multi-threaded string building |

---

#### 2.4 `intern()` kya karta hai?
`str.intern()` checks if an equal string already exists in the **String Pool**:
- If yes → returns the pooled reference.
- If no → adds `str` (reference) to the String Pool and returns it.
Useful to deduplicate huge numbers of identical strings parsed from files/APIs, though excessive interning can bloat the pool.

---

### Scenarios

- **Q: Loop mein 1 lakh baar string concat karna hai, performance slow hai. Kya karoge?**
  **A:** Use `StringBuilder`. Using `+` inside a loop creates a new `StringBuilder` + `String` object on every iteration, leading to $O(n^2)$ copying overhead.

- **Q: Password store karne ke liye `String` ya `char[]`?**
  **A:** `char[]`, because you can explicitly overwrite it with `' '` / `'\0'` (`Arrays.fill(pwd, '\0')`) immediately after authentication. `String` stays immutable in Heap/String Pool until GC runs and can show up in heap dumps/logs.

---

## 3. Collections Framework

### Concepts

#### 3.1 `ArrayList` vs `LinkedList`?
| Feature | `ArrayList` | `LinkedList` |
|---|---|---|
| **Underlying Structure** | Resizable dynamic array (grows 1.5x) | Doubly linked list (`Node` with `prev`/`next`) |
| **Random Access (`get(i)`)** | **$O(1)$** | **$O(n)$** |
| **Insert/Delete at end** | $O(1)$ amortized | $O(1)$ |
| **Insert/Delete at middle/start** | $O(n)$ (array shift) | $O(1)$ *only if you already hold the iterator/node*; $O(n)$ to search by index |
| **Memory & Cache** | Compact, CPU cache-friendly | High overhead (~24+ extra bytes per node for `prev`/`next` pointers), poor cache locality |

*(In practice, `ArrayList` outperforms `LinkedList` almost everywhere; use `ArrayDeque` for queues/stacks.)*

---

#### 3.2 `HashMap` internally kaise kaam karta hai?
1. Backed by an **array of buckets (`Node<K,V>[] table`)**, default initial capacity **16**, load factor **0.75** (resizes/doubles at `16 × 0.75 = 12` entries).
2. On `put(k, v)`:
   - Computes `k.hashCode()`, applies a **perturbation hash function** (`(h = key.hashCode()) ^ (h >>> 16)`) to mix high bits, then maps to bucket `index = (n - 1) & hash`.
   - Note: **`null` key** always goes to **bucket `0`**.
3. **Collision handling (Java 8+):**
   - If bucket has nodes, traverses the linked list checking `hash` and `equals()`. If key matches, overwrites value.
   - If bucket length exceeds **`TREEIFY_THRESHOLD = 8`** (and table capacity $\ge 64$), the linked list converts into a **Red-Black Tree** (improving worst-case lookup from $O(n)$ to **$O(\log n)$**; untreeifies back to list at `<= 6`).

---

#### 3.3 `HashMap` vs `Hashtable` vs `ConcurrentHashMap`?
| Feature | `HashMap` | `Hashtable` (Legacy) | `ConcurrentHashMap` |
|---|---|---|---|
| **Thread Safety** | No | Yes (coarse lock on entire map) | Yes (CAS + bucket-level `synchronized` on bin head in Java 8+) |
| **`null` keys / values** | 1 `null` key, many `null` values | **No `null` key or value** | **No `null` key or value** (avoids ambiguity in concurrent `get()`) |
| **Performance** | Fastest (single-threaded) | Slow under contention | High concurrency (lock-free reads, fine-grained writes) |
| **Iterator** | Fail-fast | Fail-fast / Enumerator | Fail-safe (weakly consistent) |

---

#### 3.4 `HashSet` vs `TreeSet` vs `LinkedHashSet`?
- **`HashSet`:** Backed by `HashMap`. Unordered, **$O(1)$** avg add/remove/contains, allows 1 `null`.
- **`LinkedHashSet`:** Backed by `LinkedHashMap`. Maintains **insertion order**, $O(1)$ avg, slightly more memory for doubly-linked entries.
- **`TreeSet`:** Backed by `TreeMap` (**Red-Black Tree**). Elements stored in **sorted order** (`Comparable` or `Comparator`), **$O(\log n)$** operations, **does NOT allow `null`** (throws `NullPointerException` on comparison).

---

#### 3.5 Fail-fast vs Fail-safe iterator?
- **Fail-Fast** (`ArrayList`, `HashMap`, `HashSet`): Iterates directly over the live collection using an internal **`modCount`**. If structure changes during iteration (outside `iterator.remove()`), immediately throws **`ConcurrentModificationException`**.
- **Fail-Safe (Weakly Consistent)** (`ConcurrentHashMap`, `CopyOnWriteArrayList`): Iterates over a **snapshot/copy** or handles concurrent modifications gracefully — **never throws `ConcurrentModificationException`**, though it may not reflect updates made after iteration started (and `CopyOnWrite` uses extra memory on writes).

---

#### 3.6 `Comparable` vs `Comparator`?
| Feature | `Comparable<T>` (`java.lang`) | `Comparator<T>` (`java.util`) |
|---|---|---|
| **Purpose** | Defines **natural / default** ordering inside the class itself | Defines **custom / multiple** external sortings (by age, salary, name) |
| **Method** | `int compareTo(T other)` | `int compare(T o1, T o2)` |
| **Modifies original class?** | Yes | No (great for 3rd-party/library classes) |
| **Lambda friendly?** | Implemented on class | Functional interface (`Comparator.comparing(Emp::getSalary)`) |

---

#### 3.7 `Collections.synchronizedList` vs `CopyOnWriteArrayList`?
- **`Collections.synchronizedList(list)`:** Wraps every method with a **single `synchronized` lock** (`mutex`). Reads block writes and reads block each other; **must manually synchronize** around iteration!
- **`CopyOnWriteArrayList`:** On every write (`add`/`set`/`remove`), copies the entire backing array under a lock and atomically swaps the reference. **Reads are completely lock-free** and iterators never throw `ConcurrentModificationException`. Best for **read-heavy, rare-write** lists (e.g., event listeners).

---

### Scenarios

- **Q: List iterate karte waqt element remove karna hai, `ConcurrentModificationException` aa raha hai.**
  **A:** Use `Iterator.remove()` inside a classic iterator loop, or `list.removeIf(item -> condition)` (Java 8+).

- **Q: Insertion order maintain karna hai aur duplicates nahi chahiye.**
  **A:** `LinkedHashSet`.

- **Q: Ek cache banana hai jo sirf last 100 entries rakhe (LRU).**
  **A:** Extend `LinkedHashMap` with `accessOrder = true` and override `removeEldestEntry`:
  ```java
  class LRUCache<K, V> extends LinkedHashMap<K, V> {
      private final int capacity;
      LRUCache(int capacity) {
          super(capacity, 0.75f, true); // accessOrder = true
          this.capacity = capacity;
      }
      @Override
      protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
          return size() > capacity;
      }
  }
  ```
  *(Note: Wrap with `Collections.synchronizedMap` or use `ConcurrentHashMap` + doubly-linked list/Caffeine if thread safety is needed.)*

- **Q: 10 million records mein duplicate dhoondhna hai.**
  **A:** `HashSet` gives $O(n)$ time (needs enough RAM). If memory is constrained: **sort** and check adjacent elements ($O(n \log n)$ time, $O(1)$ extra space), use external merge sort, or a **Bloom filter** if slight false positives are acceptable before DB check.

---

## 4. Exception Handling

### Concepts

#### 4.1 Checked vs Unchecked exceptions?
- **Checked Exceptions** (extend `Exception`, excluding `RuntimeException` — e.g., `IOException`, `SQLException`): Checked at **compile time**; must be caught (`try-catch`) or declared (`throws`). Used for recoverable external failures.
- **Unchecked Exceptions** (extend `RuntimeException` — e.g., `NullPointerException`, `IllegalArgumentException`, `IndexOutOfBoundsException`): Checked at **runtime**. Usually indicate programming bugs or invalid arguments.
- **`Error`** (e.g., `OutOfMemoryError`, `StackOverflowError`): Serious JVM failures that applications normally shouldn't catch.

---

#### 4.2 `throw` vs `throws`?
- **`throw`:** Used **inside a method** to explicitly throw a single exception instance (`throw new IllegalArgumentException("Invalid age");`).
- **`throws`:** Used in a **method signature** to declare checked exceptions the method may propagate to its caller (`void read() throws IOException`).

---

#### 4.3 `try-with-resources` kya hai?
Introduced in Java 7: automatically closes any resource implementing **`AutoCloseable`** (or `Closeable`) in reverse order of declaration when the `try` block exits—even if an exception is thrown! Also preserves the primary exception and attaches close exceptions as **suppressed exceptions** (`getSuppressed()`).

---

#### 4.4 Kya `finally` hamesha run hota hai?
Almost always **yes** (even if `try` or `catch` has a `return` or throws an exception).
**Exceptions where `finally` does NOT run:**
1. `System.exit(0)` / `Runtime.getRuntime().halt()` is called.
2. Fatal JVM crash (`SIGKILL` / power off).
3. An infinite loop or deadlock occurs before reaching `finally`, or a **daemon thread** is terminated when all non-daemon threads exit.

---

#### 4.5 Custom exception kaise banate hain?
Extend `Exception` (for checked) or `RuntimeException` (for unchecked) and pass message/cause to `super(...)`:
```java
public class InsufficientBalanceException extends RuntimeException {
    public InsufficientBalanceException(String message) {
        super(message);
    }
    public InsufficientBalanceException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

---

### Scenarios

- **Q: `try` mein `return 1` hai, `finally` mein `return 2`. Kya return hoga?**
  **A:** `2`. A `return` inside `finally` overrides any pending `return` (and even swallows exceptions thrown inside `try`!). Avoid returning from `finally`.

- **Q: File read karte waqt resource leak se kaise bachoge?**
  **A:** Use `try-with-resources`:
  ```java
  try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
      return br.readLine();
  } // br.close() called automatically
  ```

---

## 5. Multithreading & Concurrency

### Concepts

#### 5.1 Thread kaise banate hain? (`Thread`, `Runnable`, `Callable`, `ExecutorService`)
1. **Extend `Thread`:** Override `run()` (not preferred — wastes single inheritance).
2. **Implement `Runnable` (functional interface):** `void run()` — no return value, no checked exceptions. Pass to `new Thread(runnable).start()`. *(Never call `.run()` directly — that runs on the current thread!)*
3. **Implement `Callable<V>`:** `V call() throws Exception` — returns a `Future<V>` and can throw checked exceptions; submitted to an `ExecutorService`.
4. **`ExecutorService` (Preferred in production):** Decouples task submission from thread creation/management via thread pools (`Executors.newFixedThreadPool(4)`).

---

#### 5.2 `synchronized` vs `ReentrantLock`?
| Feature | `synchronized` | `ReentrantLock` (`java.util.concurrent.locks`) |
|---|---|---|
| **Locking/Unlocking** | Implicit (automatic at block/method exit) | Explicit (`lock()` / `unlock()` in `finally`!) |
| **Non-blocking / Timeout** | No | Yes: **`tryLock()`**, **`tryLock(timeout)`** (helps avoid deadlocks!) |
| **Interruptible wait** | No | Yes: **`lockInterruptibly()`** |
| **Fairness** | Unfair only | Configurable (`new ReentrantLock(true)` = fair) |
| **Condition variables** | Single (`wait`/`notify` on monitor) | Multiple (`newCondition()` — separate producer/consumer queues) |

---

#### 5.3 `volatile` kya karta hai?
`volatile` guarantees **visibility** and **prevents instruction reordering** (happens-before memory barrier) across threads: every read fetches from **main memory** and every write flushes directly to **main memory**, never stale CPU caches.
- **Limitation:** It does **NOT guarantee atomicity** for compound operations like `count++` (read → increment → write). Use `AtomicInteger` or `synchronized` for that.

---

#### 5.4 `wait()` / `notify()` vs `sleep()`?
| Feature | `wait()` / `notify()` | `Thread.sleep(ms)` |
|---|---|---|
| **Defined in** | `Object` class | `Thread` class (static) |
| **Lock behavior** | **Releases** the monitor lock | **Keeps/holds** all acquired locks |
| **Requires `synchronized`?** | **Yes** (else `IllegalMonitorStateException`) | No |
| **Wake up** | Needs `notify()` / `notifyAll()` (or timeout) | Wakes automatically after time elapses |

*(Always call `wait()` inside a **`while` loop**, not `if`, to guard against **spurious wakeups**!)*

---

#### 5.5 Deadlock kya hai aur kaise avoid karte hain?
Two or more threads blocked forever waiting for each other's locks (Circular Wait).
- **How to avoid:**
  1. **Consistent Lock Ordering:** Always acquire locks in the same global order (e.g., sort account IDs before locking both).
  2. **Use `tryLock(timeout)`:** Back off and retry if lock isn't acquired within timeout.
  3. **Minimize lock scope:** Keep synchronized blocks small and avoid calling external callbacks inside locks.
  4. **Use concurrent collections / lock-free atomics** instead of manual nested locks.

---

#### 5.6 Thread pool types? (`Fixed`, `Cached`, `Scheduled`, Virtual threads in Java 21)
- **`FixedThreadPool(n)`:** Fixed `n` worker threads + unbounded queue (`LinkedBlockingQueue`). Predictable thread count; watch out for OOM if tasks pile up → prefer `ThreadPoolExecutor` with a **bounded queue** in production!
- **`CachedThreadPool()`:** Creates new threads as needed, reuses idle threads (60s timeout). Good for short-lived async tasks, risky under burst load (can spawn thousands of threads).
- **`SingleThreadExecutor()`:** 1 thread, executes tasks strictly sequentially.
- **`ScheduledThreadPool(n)`:** Runs delayed or periodic tasks (`scheduleAtFixedRate`).
- **Virtual Threads (Java 21 — Project Loom):** Lightweight JVM-managed threads (`Executors.newVirtualThreadPerTaskExecutor()`). Millions can run concurrently because blocking I/O (HTTP/DB calls) unmounts the virtual thread from its OS carrier thread without wasting OS resources!

---

#### 5.7 `CompletableFuture` kya hai?
Non-blocking asynchronous programming API (Java 8+) that solves `Future.get()` blocking limitations:
- Supports chaining (`thenApply`, `thenCompose`, `thenAccept`), combining multiple async calls (`thenCombine`, `allOf`, `anyOf`), and clean exception handling (`exceptionally`, `handle`).
```java
CompletableFuture<String> cf = CompletableFuture
    .supplyAsync(() -> fetchUser())
    .thenApply(user -> user.getName())
    .exceptionally(ex -> "Guest");
```

---

#### 5.8 Race condition aur atomic classes (`AtomicInteger`)?
- **Race Condition:** Unpredictable bug when multiple threads read-modify-write shared mutable state concurrently without synchronization (`count++` loses updates).
- **`AtomicInteger` / `AtomicReference`:** Uses hardware **CAS (Compare-And-Swap)** instructions (`Unsafe`/VarHandle) to perform atomic updates lock-free (`incrementAndGet()`, `compareAndSet(expected, updated)`), much faster than `synchronized` under moderate contention.

---

### Scenarios

- **Q: Do threads ek shared counter increment kar rahe hain, result galat aa raha hai.**
  **A:** `counter++` is non-atomic (read + add + write). Use `AtomicInteger` (`incrementAndGet()`) or a `synchronized` block/method.

- **Q: Producer-Consumer problem implement karo.**
  **A:** Use `BlockingQueue` (internally handles locks and `Condition` wait/notify cleanly):
  ```java
  BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);

  Runnable producer = () -> {
      try {
          for (int i = 0; i < 20; i++) queue.put(i);
      } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
      }
  };

  Runnable consumer = () -> {
      try {
          while (!Thread.currentThread().isInterrupted()) {
              System.out.println(queue.take());
          }
      } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
      }
  };

  new Thread(producer).start();
  new Thread(consumer).start();
  ```

- **Q: Singleton ko thread-safe kaise banaoge?**
  **A:** Double-Checked Locking with `volatile` (prevents seeing a partially constructed object due to instruction reordering):
  ```java
  public class Singleton {
      private static volatile Singleton instance;
      private Singleton() {}
      public static Singleton getInstance() {
          if (instance == null) {
              synchronized (Singleton.class) {
                  if (instance == null) {
                      instance = new Singleton();
                  }
              }
          }
          return instance;
      }
  }
  ```
  *(Alternatives: **Bill Pugh static inner helper class** (lazy & thread-safe via classloader) or **Enum Singleton** (serialization & reflection safe).)*

- **Q: 3 API calls parallel chalani hain aur sab ka result chahiye.**
  **A:** Run 3 `CompletableFuture.supplyAsync(..., customExecutor)` calls, wait via `CompletableFuture.allOf(f1, f2, f3).join()`, then read `f1.join()`, `f2.join()`, `f3.join()`.

- **Q: Production mein thread pool ka queue full ho gaya, requests reject ho rahi hain.**
  **A:** Use a custom `ThreadPoolExecutor` with tuned `corePoolSize`/`maxPoolSize`, a **bounded queue**, and an appropriate `RejectedExecutionHandler` (like `CallerRunsPolicy` for backpressure instead of crashing with `AbortPolicy`), plus monitor pool metrics or switch I/O workloads to **Virtual Threads**.

---

## 6. Java 8+ Features

### Concepts

#### 6.1 Lambda, Functional Interface, Method Reference?
- **Functional Interface:** An interface with **exactly ONE abstract method** (annotated with `@FunctionalInterface` optionally; can still have `default`/`static` methods). Examples: `Runnable`, `Comparator<T>`, `Predicate<T>`, `Function<T,R>`, `Consumer<T>`, `Supplier<T>`.
- **Lambda Expression:** Concise anonymous implementation of a functional interface: `(a, b) -> a + b`.
- **Method Reference:** Shorthand for a lambda that just delegates to an existing method (`Employee::getName`, `System.out::println`, `ArrayList::new`).

---

#### 6.2 Stream API: `map`, `filter`, `reduce`, `collect`, `flatMap`?
Streams process data declaratively via **intermediate** (lazy) and **terminal** (triggers execution) operations without mutating the source collection:
- **`filter(Predicate)`** *(intermediate)*: Keeps elements matching condition.
- **`map(Function)`** *(intermediate)*: Transforms each element 1-to-1.
- **`flatMap(Function)`** *(intermediate)*: Transforms each element into a stream and flattens into a single stream.
- **`reduce(identity, accumulator)`** *(terminal)*: Combines elements into a single value (e.g., sum, max).
- **`collect(Collector)`** *(terminal)*: Gathers elements into `List`, `Set`, `Map`, `groupingBy`, `joining`, etc.

---

#### 6.3 `map` vs `flatMap`?
- **`map` (1-to-1):** Each input element produces **1 output element** (`Stream<List<String>>` remains nested).
- **`flatMap` (1-to-many + flatten):** Each input element produces a Stream of 0+ elements, then flattens all streams into one (`Stream<List<String>>` → `Stream<String>`).
```java
List<List<Integer>> nested = List.of(List.of(1, 2), List.of(3, 4));
List<Integer> flat = nested.stream().flatMap(Collection::stream).toList(); // [1, 2, 3, 4]
```

---

#### 6.4 `Optional` kaise use karte hain?
`Optional<T>` is a container object representing a value that may or may not be present, helping eliminate `NullPointerException`.
- **Create:** `Optional.of(val)` (non-null), `Optional.ofNullable(val)` (nullable), `Optional.empty()`.
- **Consume cleanly (avoid raw `.get()`!):**
  - `opt.orElse("default")` *(always evaluates `"default"` even if present!)*
  - `opt.orElseGet(() -> computeDefault())` *(lazy — preferred for expensive defaults)*
  - `opt.orElseThrow(() -> new NotFoundException(...))`
  - `opt.map(User::getEmail).ifPresent(System.out::println)`
*(Rule: Use `Optional` as a **method return type**, not for entity fields or method parameters.)*

---

#### 6.5 Records, Sealed classes, `var`, Text blocks, Switch expressions (Java 14–21)?
- **`var` (Java 10):** Local variable type inference at **compile time** (`var map = new HashMap<String, Integer>();`). Still statically typed!
- **Switch Expressions (Java 14):** Arrow syntax (`->`), no fall-through bugs, returns a value, enforces exhaustiveness.
  ```java
  int days = switch (month) {
      case "FEB" -> 28;
      case "APR", "JUN" -> 30;
      default -> 31;
  };
  ```
- **Text Blocks (Java 15):** Multi-line strings with `""" ... """` (great for SQL/JSON literals).
- **Records (Java 16):** Concise immutable data-carrier classes (`public record User(Long id, String name) {}`) — auto-generates `final` fields, canonical constructor, getters (`id()`, `name()`), `equals()`, `hashCode()`, `toString()`.
- **Sealed Classes (Java 17):** Restricts which classes can extend/implement a type:
  ```java
  public sealed interface Shape permits Circle, Rectangle {}
  ```
  Pairs with **Pattern Matching for `switch` (Java 21)** for compile-time exhaustive checks!

---

#### 6.6 Virtual Threads (Java 21)?
Introduced by **Project Loom**: lightweight threads scheduled by the **JVM** onto a small pool of OS carrier threads (`ForkJoinPool`), instead of 1-to-1 OS threads (~1 MB stack each).
- When a virtual thread hits a blocking I/O call (DB, HTTP, `sleep`), the JVM **unmounts** it and lets another virtual thread use that OS thread.
- Lets you write simple synchronous blocking code that scales to **hundreds of thousands/millions of concurrent I/O tasks** (`Executors.newVirtualThreadPerTaskExecutor()`). *(Note: Not faster for CPU-bound work; watch out for `synchronized` pinning in Java 21 — fixed in Java 24, or use `ReentrantLock` in Java 21.)*

---

### Scenarios (Common Coding Round Stream Queries)

```java
// 1. Employees ko department ke hisaab se group karo
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDept));

// 2. Har department ka average salary
Map<String, Double> avgSalary = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDept,
             Collectors.averagingDouble(Employee::getSalary)));

// 3. Second highest salary (use distinct() if duplicate salaries exist!)
Optional<Double> secondSalary = employees.stream()
    .map(Employee::getSalary)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst();

// Or second highest Employee (when salaries are distinct):
Optional<Employee> secondEmp = employees.stream()
    .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
    .skip(1)
    .findFirst();

// 4. Duplicate elements in a list
Set<Integer> seen = new HashSet<>();
List<Integer> dups = nums.stream()
    .filter(n -> !seen.add(n))
    .distinct()
    .toList();

// 5. String mein har character ka frequency count
Map<Character, Long> freq = str.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```

---

## 7. JVM, Memory & Garbage Collection

### Concepts

#### 7.1 JVM architecture? (ClassLoader, Runtime Data Areas, Execution Engine)
1. **ClassLoader Subsystem:**
   - **Loading** (Bootstrap → Platform/Extension → Application ClassLoader, following Parent Delegation Model)
   - **Linking** (Verify bytecode → Prepare static fields with defaults → Resolve symbolic references)
   - **Initialization** (Runs `static` blocks and assigns static values)
2. **Runtime Data Areas:**
   - **Shared across threads:** **Heap** (objects/arrays), **Method Area / Metaspace** (class metadata, static vars, constant pool).
   - **Per-thread:** **JVM Stack** (stack frames for method calls, local vars, operand stack), **PC Register**, **Native Method Stack**.
3. **Execution Engine:** Interpreter + **JIT (Just-In-Time) Compiler** (compiles hot bytecode paths into native machine code) + **Garbage Collector**.

---

#### 7.2 Heap vs Stack? Metaspace kya hai?
- **Stack:** Per-thread, LIFO, stores local primitives and **object references**, fast allocation/deallocation, throws `StackOverflowError` if recursion goes too deep.
- **Heap:** Shared across all threads, stores **all actual objects & arrays**, managed by GC, throws `OutOfMemoryError: Java heap space` when full.
- **Metaspace (Java 8+):** Replaced PermGen. Lives in **native OS memory** (not JVM heap!) and stores class definitions, method bytecode metadata, and runtime constant pool metadata. Auto-grows up to `MaxMetaspaceSize`.

---

#### 7.3 GC kaise kaam karta hai? (Young/Old Gen, Minor/Major GC)
Based on the **Weak Generational Hypothesis** (most objects die young):
- **Young Generation:** `Eden` + two Survivor spaces (`S0`, `S1`). New objects allocate in `Eden`. When full → **Minor GC** copies live objects to `S0`/`S1` and increments their age (fast, frequent).
- **Old (Tenured) Generation:** Objects surviving multiple Minor GCs (age threshold) are promoted here. When Old Gen fills → **Major / Full GC** runs (slower, longer Stop-The-World pause).
- **Algorithm:** Mark-and-Sweep / Mark-Compact / Copying from **GC Roots** (active thread stacks, static fields, JNI references).

---

#### 7.4 GC types: Serial, Parallel, G1, ZGC?
- **Serial GC:** Single-threaded GC; best for tiny single-core containers/CLI tools.
- **Parallel GC:** Multi-threaded Young & Old GC; maximizes **throughput** (batch jobs), higher pause times.
- **G1 GC (Default since Java 9):** Splits heap into equal regions, collects regions with most garbage first ("Garbage First") to meet configurable pause-time targets (`-XX:MaxGCPauseMillis`). Balanced throughput + latency.
- **ZGC (Java 15+ / Generational in Java 21):** Concurrent low-latency collector keeping pauses **under 1 ms** even on multi-TB heaps using colored pointers & load barriers.

---

#### 7.5 Memory leak Java mein kaise hota hai?
When objects are **no longer needed by business logic**, yet remain **reachable from GC Roots** so GC cannot reclaim them:
1. Unbounded `static` collections / caches (`static List<User> cache = new ArrayList<>()`).
2. Unclosed resources (`Connection`, `InputStream`, `ResultSet`).
3. Listeners/callbacks registered and never deregistered.
4. `ThreadLocal` variables in thread pools without calling `.remove()`.
5. Non-`static` inner classes holding implicit references to outer class instances.
6. Mutable keys in `HashMap` / missing `equals` & `hashCode`.

---

#### 7.6 Strong, Weak, Soft, Phantom references?
- **Strong (`Object o = new Object()`):** Never GC'd while reachable.
- **Soft (`SoftReference<T>`):** Cleared **only when JVM is running low on memory** (before throwing `OOM`) — great for memory-sensitive caches.
- **Weak (`WeakReference<T>`):** Cleared on the **next GC cycle** if no strong references exist — used in `WeakHashMap` (e.g., canonicalizing maps / metadata caches).
- **Phantom (`PhantomReference<T>`):** `.get()` always returns `null`; enqueued into `ReferenceQueue` when object is finalized/ready to reclaim — used for off-heap/native resource cleanup (`Cleaner`).

---

### Scenarios

- **Q: Production mein `OutOfMemoryError: Java heap space` aa raha hai.**
  **A:** Enable `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/path`, analyze the `.hprof` dump in **Eclipse MAT** or **VisualVM** to inspect Dominator Tree / biggest retained heap objects, and fix leaks (static maps, `ThreadLocal`, unclosed streams, missing pagination on DB queries).

- **Q: App mein baar-baar "Stop the world" pauses aa rahe hain.**
  **A:** Enable/check GC logs (`-Xlog:gc*`), verify heap sizing (`-Xms`/`-Xmx`), tune or switch to **G1GC** or **ZGC** (Java 21 Generational ZGC), and reduce short-lived allocation churn in hot loops.

---

## 8. Design Patterns & Principles

### 8.1 SOLID Principles (with Quick Java Examples)
1. **S — Single Responsibility:** One class = one reason to change (`InvoiceCalculator` vs `InvoiceRepository` vs `InvoicePrinter`).
2. **O — Open/Closed:** Open for extension, closed for modification (add new `DiscountStrategy` classes instead of editing `if/else`).
3. **L — Liskov Substitution:** Subtypes must honor base class contracts (`Square` shouldn't break `Rectangle`'s independent width/height contract).
4. **I — Interface Segregation:** Small focused interfaces (`Printer`, `Scanner`) instead of one bloated `AllInOneMachine`.
5. **D — Dependency Inversion:** Depend on abstractions (`OrderService` takes `PaymentGateway` interface, not hardcoded `RazorpayClient`).

---

### 8.2 Key Design Patterns (with Java Examples)

1. **Singleton (Creational):** Ensures only one instance exists globally (DB pool, config manager).
   ```java
   public enum AppConfig {
       INSTANCE; // Thread-safe, serialization & reflection safe
       public String getEnv() { return "prod"; }
   }
   ```
2. **Factory (Creational):** Centralizes object creation behind a common interface without exposing instantiation logic.
   ```java
   interface Notification { void send(); }
   class NotificationFactory {
       public static Notification create(String type) {
           return switch (type) {
               case "EMAIL" -> new EmailNotification();
               case "SMS"   -> new SmsNotification();
               default -> throw new IllegalArgumentException("Unknown type");
           };
       }
   }
   ```
3. **Builder (Creational):** Constructs complex objects with many optional fields step-by-step cleanly.
   ```java
   User u = new User.Builder("Abhishek").age(22).city("Indore").build();
   ```
4. **Strategy (Behavioral):** Swaps algorithms/behaviors at runtime cleanly via composition (`PaymentStrategy` for UPI/Card/NetBanking).
5. **Observer (Behavioral):** One-to-many pub-sub event notification (when Order is placed → notify EmailService, InventoryService, AnalyticsService).
6. **Decorator (Structural):** Dynamically adds behavior to an object by wrapping it with the same interface (e.g., `new BufferedReader(new FileReader(...))`).
7. **Proxy (Structural):** Controls access to another object (lazy loading, caching, logging, security, **Spring `@Transactional` AOP proxies**).

---

### 8.3 Dependency Injection (DI) vs Inversion of Control (IoC)?
- **IoC (Principle):** Broad architectural principle where **control of object creation and lifecycle is inverted** from your code to a framework/container ("Don't call us, we'll call you").
- **DI (Implementation Pattern):** A specific way to achieve IoC where a class receives its dependencies from outside (constructor/setter) instead of doing `new Dependency()` inside itself.

---

### Scenarios

- **Q: Payment gateway (Razorpay, Stripe, PayPal) ke liye extensible design batao.**
  **A:** **Strategy + Factory Pattern:** Define a `PaymentStrategy` interface (`pay(amount)`), implement `RazorpayStrategy`, `StripeStrategy`, `PayPalStrategy`, and inject/select the right one dynamically via a Factory or Spring `Map<String, PaymentStrategy>` bean map.

- **Q: Object mein 10 optional fields hain, constructor messy hai.**
  **A:** Use the **Builder Pattern** (e.g., static inner `Builder` class or Lombok `@Builder`).

---

## 9. Spring Boot

### Concepts

#### 9.1 `@SpringBootApplication` kya hai? Auto-configuration kaise kaam karta hai?
`@SpringBootApplication` is a meta-annotation combining 3 annotations:
1. `@SpringBootConfiguration` (marks it as a `@Configuration` source)
2. `@EnableAutoConfiguration` (activates Spring Boot auto-configuration)
3. `@ComponentScan` (scans current package & sub-packages for `@Component`, `@Service`, etc.)
- **How Auto-Configuration works:** On startup, Spring Boot reads candidate auto-config classes from `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Spring Boot 3+; `spring.factories` in Boot 2) and applies **`@Conditional*` annotations** (`@ConditionalOnClass`, `@ConditionalOnMissingBean`, `@ConditionalOnProperty`) — e.g., if HikariCP & Postgres driver are on the classpath and you haven't defined a custom `DataSource`, it auto-configures one!

---

#### 9.2 `@Component` vs `@Service` vs `@Repository` vs `@Controller`?
All 4 are stereotypes scanned as Spring Beans (`@Service`, `@Repository`, `@Controller` are specializations of `@Component`):
- **`@Component`:** Generic Spring-managed bean.
- **`@Service`:** Business logic layer (semantic clarity + AOP pointcut target).
- **`@Repository`:** Data access (DAO) layer — **additionally translates vendor-specific SQL/JPA exceptions into Spring's unified `DataAccessException` hierarchy!**
- **`@Controller`:** Web MVC controller (returns view names; combine with `@ResponseBody` or use `@RestController` for JSON APIs).

---

#### 9.3 Bean scopes? Default scope?
- **`singleton` (Default):** **One shared instance per Spring IoC container** (note: should be stateless for thread safety!).
- **`prototype`:** New instance created every time the bean is requested/injected.
- Web scopes: **`request`** (per HTTP request), **`session`** (per HTTP session), **`application`** (per `ServletContext`), **`websocket`**.

---

#### 9.4 `@Autowired` vs Constructor Injection?
**Constructor Injection is strongly preferred:**
1. Allows dependencies to be **`final`** (immutable & guaranteed initialized — no NPEs).
2. Makes dependencies explicit and **easy to unit test** without Spring reflection (`new OrderService(mockRepo)`).
3. Fails fast on **circular dependencies** at startup rather than hiding design smells.
*(Note: If a class has a single constructor, `@Autowired` is optional since Spring 4.3.)*

---

#### 9.5 `@RestController` vs `@Controller`?
- **`@Controller`:** Returns a **view name** resolved by ViewResolver (HTML/Thymeleaf/JSP) unless methods have `@ResponseBody`.
- **`@RestController`:** `@Controller + @ResponseBody` — every handler method serializes the return object directly to the HTTP response body (JSON/XML via Jackson).

---

#### 9.6 Spring Data JPA: `@Entity`, `@OneToMany`, Lazy vs Eager loading?
- **`@Entity`:** Maps a Java class to a DB table (`@Id`, `@GeneratedValue`).
- **`@OneToMany`:** Maps a parent-to-children relationship (always keep `mappedBy` on the parent side so JPA doesn't create an extra join table!).
- **Fetch Types:**
  - **`LAZY` (Default for `@OneToMany` / `@ManyToMany`):** Loads child collection only when accessed inside an active session/transaction. Best for performance!
  - **`EAGER` (Default for `@ManyToOne` / `@OneToOne`):** Always loads related entity immediately on parent fetch — often causes unwanted joins or N+1 queries! *(Best practice: set `@ManyToOne(fetch = FetchType.LAZY)` too, and fetch intentionally via `JOIN FETCH` when needed.)*

---

#### 9.7 `@Transactional` kaise kaam karta hai?
Spring wraps the bean in a **dynamic proxy** (JDK Dynamic Proxy or CGLIB). When an external caller invokes a `@Transactional` method:
1. Proxy opens/joins a DB transaction (`setAutoCommit(false)`).
2. Calls the target method.
3. **Commits** if normal return; **rolls back** automatically on **unchecked exceptions (`RuntimeException`/`Error`)** by default (use `rollbackFor = Exception.class` to also rollback on checked exceptions).

---

#### 9.8 Spring Security + JWT flow?
1. Client sends `POST /login` with username/password.
2. `AuthenticationManager` delegates to `DaoAuthenticationProvider` → loads `UserDetails` (`UserDetailsService`) and verifies `BCrypt` hash via `PasswordEncoder`.
3. On success, server generates a signed **JWT** (`header.payload.signature` with `sub`, roles, `exp`) and returns it.
4. For subsequent requests (`Authorization: Bearer <token>`), a custom **`OncePerRequestFilter`** (`JwtAuthFilter`) extracts & validates the JWT signature/expiry, builds an `Authentication` object, and stores it in **`SecurityContextHolder`** (stateless session: `SessionCreationPolicy.STATELESS`).
5. Spring Security checks `@PreAuthorize` / route rules and executes the controller.

---

### Scenarios

- **Q: N+1 query problem kya hai aur kaise fix karoge?**
  **A:** Fetching 1 parent list (`SELECT * FROM orders` → 1 query) causes Hibernate to fire `N` extra queries (`SELECT * FROM order_items WHERE order_id = ?`) for each order's lazy children. **Fix:** Use JPQL **`JOIN FETCH`**, **`@EntityGraph`**, or `@BatchSize(size = 50)`.

- **Q: `@Transactional` method same class ke andar se call karne par kaam nahi kar raha.**
  **A:** Spring AOP is **proxy-based**. Calling `this.methodB()` inside `methodA()` is a direct self-invocation that **bypasses the proxy**, so transaction advice never triggers. **Fix:** Move `methodB()` to a separate `@Service` bean (or inject self / use `TransactionTemplate`).

- **Q: API ko 1000 req/sec handle karna hai.**
  **A:** Add **Redis caching** for hot reads, proper **DB indexes** + **keyset pagination**, tune **HikariCP** connection pool, offload slow work (emails/analytics) to async queues (**Kafka/RabbitMQ**), use **Java 21 Virtual Threads** (`spring.threads.virtual.enabled=true`), and scale horizontally behind a load balancer.

- **Q: Global exception handling kaise karoge?**
  **A:** Use `@RestControllerAdvice` + `@ExceptionHandler`:
  ```java
  @RestControllerAdvice
  public class GlobalExceptionHandler {
      @ExceptionHandler(ResourceNotFoundException.class)
      public ResponseEntity<Map<String, String>> handle(ResourceNotFoundException e) {
          return ResponseEntity.status(HttpStatus.NOT_FOUND)
                               .body(Map.of("error", e.getMessage()));
      }
  }
  ```

---

## 10. Coding Round Solutions (Java)

### 10.1 String Reverse, Palindrome, Anagram
```java
// Reverse String — O(n) time, O(n) space
public String reverse(String s) {
    char[] arr = s.toCharArray();
    for (int l = 0, r = arr.length - 1; l < r; l++, r--) {
        char tmp = arr[l]; arr[l] = arr[r]; arr[r] = tmp;
    }
    return new String(arr);
}

// Reverse Number (handles negative & overflow check) — O(log10 n)
public int reverseInt(int x) {
    int rev = 0;
    while (x != 0) {
        int pop = x % 10;
        x /= 10;
        if (rev > Integer.MAX_VALUE / 10 || rev < Integer.MIN_VALUE / 10) return 0;
        rev = rev * 10 + pop;
    }
    return rev;
}

// Palindrome Check — O(n) time, O(1) space
public boolean isPalindrome(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        if (s.charAt(l++) != s.charAt(r--)) return false;
    }
    return true;
}

// Anagram Check (lowercase ASCII) — O(n) time, O(1) space
public boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < a.length(); i++) {
        count[a.charAt(i) - 'a']++;
        count[b.charAt(i) - 'a']--;
    }
    for (int c : count) if (c != 0) return false;
    return true;
}
```

---

### 10.2 Fibonacci, Factorial, Prime Check
```java
// Iterative Fibonacci (nth) — O(n) time, O(1) space
public long fib(int n) {
    if (n <= 1) return n;
    long a = 0, b = 1;
    for (int i = 2; i <= n; i++) {
        long c = a + b; a = b; b = c;
    }
    return b;
}

// Factorial — O(n) time, O(1) space
public long factorial(int n) {
    long res = 1;
    for (int i = 2; i <= n; i++) res *= i;
    return res;
}

// Prime Check — O(sqrt(n)) time, O(1) space
public boolean isPrime(int n) {
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;
    for (int i = 5; i * i <= n; i += 6) {
        if (n % i == 0 || n % (i + 2) == 0) return false;
    }
    return true;
}
```

---

### 10.3 Array: Duplicate, Missing Number, Second Largest
```java
// Missing Number in [0..n] — O(n) time, O(1) space (XOR avoids overflow!)
public int missingNumber(int[] nums) {
    int xor = nums.length;
    for (int i = 0; i < nums.length; i++) xor ^= i ^ nums[i];
    return xor;
}

// Find Duplicate (Floyd's Cycle Detection for array of [1..n] size n+1) — O(n) time, O(1) space
public int findDuplicate(int[] nums) {
    int slow = nums[0], fast = nums[0];
    do { slow = nums[slow]; fast = nums[nums[fast]]; } while (slow != fast);
    slow = nums[0];
    while (slow != fast) { slow = nums[slow]; fast = nums[fast]; }
    return slow;
}

// Second Largest in single pass — O(n) time, O(1) space
public int secondLargest(int[] nums) {
    int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int n : nums) {
        if (n > first) { second = first; first = n; }
        else if (n > second && n != first) { second = n; }
    }
    return second;
}
```

---

### 10.4 Two Sum (HashMap — $O(n)$ time, $O(n)$ space)
```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int need = target - nums[i];
        if (map.containsKey(need)) return new int[]{map.get(need), i};
        map.put(nums[i], i);
    }
    return new int[]{};
}
```

---

### 10.5 Linked List Reverse & Cycle Detection (Floyd)
```java
// Reverse Linked List — O(n) time, O(1) space
public ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}

// Cycle Detection (Floyd's Tortoise & Hare) — O(n) time, O(1) space
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

---

### 10.6 Binary Search & Variants ($O(\log n)$ time, $O(1)$ space)
```java
// Classic Binary Search
public int binarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2; // prevents overflow
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

// Lower Bound (first index >= target)
public int lowerBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
```

---

### 10.7 Valid Parentheses (Stack — $O(n)$ time, $O(n)$ space)
```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(') stack.push(')');
        else if (c == '{') stack.push('}');
        else if (c == '[') stack.push(']');
        else if (stack.isEmpty() || stack.pop() != c) return false;
    }
    return stack.isEmpty();
}
```

---

### 10.8 LRU Cache (LinkedHashMap) & Producer-Consumer (`BlockingQueue`)
- See **Section 3 Scenario (`LRUCache`)** and **Section 5 Scenario (`BlockingQueue` Producer-Consumer)** above for complete implementations.
*(If interviewer asks LRU Cache **without** `LinkedHashMap`, implement using a **`HashMap<Integer, Node>` + custom Doubly Linked List** with dummy `head` and `tail` nodes for $O(1)$ `get` and `put`.)*

---

### 10.9 Merge Intervals & Sliding Window (Longest Substring Without Repeating Chars)
```java
// Merge Intervals — O(n log n) time, O(n) space
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
    List<int[]> res = new ArrayList<>();
    for (int[] cur : intervals) {
        if (res.isEmpty() || res.get(res.size() - 1)[1] < cur[0]) {
            res.add(cur);
        } else {
            res.get(res.size() - 1)[1] = Math.max(res.get(res.size() - 1)[1], cur[1]);
        }
    }
    return res.toArray(new int[0][]);
}

// Longest Substring Without Repeating Characters — O(n) time, O(min(n, charset)) space
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int left = 0, best = 0;
    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (lastSeen.containsKey(c)) {
            left = Math.max(left, lastSeen.get(c) + 1);
        }
        lastSeen.put(c, right);
        best = Math.max(best, right - left + 1);
    }
    return best;
}
```

---

### 10.10 Tree Traversals (DFS & BFS) ($O(n)$ time)
```java
// DFS: Inorder (Left, Root, Right) — Preorder (Root, L, R) — Postorder (L, R, Root)
public void inorder(TreeNode root, List<Integer> res) {
    if (root == null) return;
    inorder(root.left, res);
    res.add(root.val);
    inorder(root.right, res);
}

// BFS: Level Order Traversal using Queue
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> q = new ArrayDeque<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = q.poll();
            level.add(node.val);
            if (node.left != null) q.offer(node.left);
            if (node.right != null) q.offer(node.right);
        }
        res.add(level);
    }
    return res;
}
```
