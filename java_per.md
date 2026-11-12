1. Core Java & OOP

Concept

    JDK vs JRE vs JVM?
    == vs .equals()?
    Abstract class vs Interface (Java 8+ mein default/static methods ke saath)?
    Method overloading vs overriding?
    static, final, finally, finalize() ka difference?
    Pass by value ya pass by reference? (Java hamesha pass by value hai. Object ka reference copy hota hai.)
    Immutable class kaise banate hain?
    equals() aur hashCode() contract kya hai?

    Scenario

    Q: Tumne Employee class ko HashMap ka key banaya, lekin get() null return kar raha hai. Kyun?
    A: equals() aur hashCode() override nahi kiye, ya key mutable thi aur insert ke baad field change ho gayi.
    Q: Ek class ko thread-safe immutable banana hai jisme List field hai.
    A: Class final rakho, fields private final, constructor mein defensive copy lo, getter se Collections.unmodifiableList() return karo.
    java
    public final class Order {
        private final List<String> items;
        public Order(List<String> items) {
            this.items = List.copyOf(items); // defensive copy
        }
        public List<String> getItems() { return items; } // already immutable
    }
2. String

Concept

    String immutable kyun hai?
    String pool kya hai? new String("a") vs "a"?
    String vs StringBuilder vs StringBuffer?
    intern() kya karta hai?

Scenario

    Q: Loop mein 1 lakh baar string concat karna hai, performance slow hai. Kya karoge?
    A: StringBuilder use karo. + har baar naya object banata hai, jisse O(n²) ho jata hai.
    Q: Password store karne ke liye String ya char[]?
    A: char[], kyunki use use karke zero kar sakte ho. String pool mein zyada der memory mein rehti hai.

3. Collections Framework

Concept

    ArrayList vs LinkedList?
    HashMap internally kaise kaam karta hai? (array of buckets, hashing, linked list → tree after 8 collisions, load factor 0.75)
    HashMap vs Hashtable vs ConcurrentHashMap?
    HashSet vs TreeSet vs LinkedHashSet?
    Fail-fast vs fail-safe iterator?
    Comparable vs Comparator?
    Collections.synchronizedList vs CopyOnWriteArrayList?

Scenario

    Q: List iterate karte waqt element remove karna hai, ConcurrentModificationException aa raha hai.
    A: Iterator.remove() use karo, ya list.removeIf(...).
    Q: Insertion order maintain karna hai aur duplicates nahi chahiye.
    A: LinkedHashSet.
    Q: Ek cache banana hai jo sirf last 100 entries rakhe (LRU).
    java
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
    Q: 10 million records mein duplicate dhoondhna hai.
    A: HashSet (O(n) time). Memory tight ho to sort karke compare karo, ya Bloom filter.
4. Exception Handling

Concept

Checked vs Unchecked exceptions?
throw vs throws?
try-with-resources kya hai?
Kya finally hamesha run hota hai? (System.exit() ya JVM crash ko chhodke, haan)
Custom exception kaise banate hain?

Scenario

Q: try mein return 1 hai, finally mein return 2. Kya return hoga?
A: 2. finally ka return try ka return override kar deta hai. Isko avoid karna chahiye.
Q: File read karte waqt resource leak se kaise bachoge?
java
try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    return br.readLine();
} // auto-closed
5. Multithreading & Concurrency (sabse zyada poocha jata hai)

Concept

Thread kaise banate hain? (Thread, Runnable, Callable, ExecutorService)
synchronized vs ReentrantLock?
volatile kya karta hai? (visibility guarantee, atomicity nahi)
wait() / notify() vs sleep()?
Deadlock kya hai aur kaise avoid karte hain? (lock ordering, tryLock with timeout)
Thread pool types? (Fixed, Cached, Scheduled, Virtual threads in Java 21)
CompletableFuture kya hai?
Race condition aur atomic classes (AtomicInteger)?

Scenario

Q: Do threads ek shared counter increment kar rahe hain, result galat aa raha hai.
A: counter++ atomic nahi hai. AtomicInteger ya synchronized use karo.
Q: Producer-Consumer problem implement karo.
java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);

Runnable producer = () -> {
    try { for (int i = 0; i < 20; i++) queue.put(i); }
    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
};
Runnable consumer = () -> {
    try { while (true) System.out.println(queue.take()); }
    catch (InterruptedException e) { Thread.currentThread().interrupt(); }
};
new Thread(producer).start();
new Thread(consumer).start();
Q: Singleton ko thread-safe kaise banaoge?
java
public class Singleton {
    private static volatile Singleton instance;
    private Singleton() {}
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }
}

(Alternative: Enum singleton ya Bill Pugh holder pattern.)

Q: 3 API calls parallel chalani hain aur sab ka result chahiye.
A: CompletableFuture.supplyAsync(...) x3, phir CompletableFuture.allOf(...).join().
Q: Production mein thread pool ka queue full ho gaya, requests reject ho rahi hain.
A: Pool size aur queue capacity tune karo, RejectedExecutionHandler set karo (CallerRunsPolicy), aur bounded queue use karo.
6. Java 8+ Features

Concept

Lambda, Functional Interface, Method reference?
Stream API: map, filter, reduce, collect, flatMap?
map vs flatMap?
Optional kaise use karte hain?
Records, Sealed classes, var, Text blocks, Switch expressions (Java 14-21)?
Virtual threads (Java 21)?

Scenario (yeh coding round mein bahut aate hain)

java
// Employees ko department ke hisaab se group karo
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDept));

// Har department ka average salary
Map<String, Double> avgSalary = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDept,
             Collectors.averagingDouble(Employee::getSalary)));

// Second highest salary
Optional<Employee> second = employees.stream()
    .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
    .skip(1).findFirst();

// Duplicate elements
Set<Integer> seen = new HashSet<>();
List<Integer> dups = nums.stream().filter(n -> !seen.add(n)).toList();

// String mein har char ka count
Map<Character, Long> freq = str.chars().mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
7. JVM, Memory & Garbage Collection

Concept

JVM architecture? (ClassLoader, Runtime Data Areas, Execution Engine)
Heap vs Stack? Metaspace kya hai?
GC kaise kaam karta hai? (Young/Old gen, Minor/Major GC)
GC types: Serial, Parallel, G1, ZGC?
Memory leak Java mein kaise hota hai?
Strong, Weak, Soft, Phantom references?

Scenario

Q: Production mein OutOfMemoryError: Java heap space aa raha hai.
A: Heap dump lo (-XX:+HeapDumpOnOutOfMemoryError), Eclipse MAT ya VisualVM se analyze karo. Static collections, unclosed resources aur listeners mein leak dhoondho.
Q: App mein baar-baar “Stop the world” pauses aa rahe hain.
A: GC logs dekho, G1 ya ZGC try karo, aur heap size tune karo.
8. Design Patterns & Principles
SOLID principles (har ek ka example ready rakho)
Singleton, Factory, Builder, Observer, Strategy, Decorator, Proxy
Dependency Injection vs Inversion of Control?

Scenario

Q: Payment gateway (Razorpay, Stripe, PayPal) ke liye extensible design batao.
A: Strategy pattern. PaymentStrategy interface, har gateway ek implementation, aur Factory se select karo.
Q: Object mein 10 optional fields hain, constructor messy hai.
A: Builder pattern.
9. Spring Boot (Java roles mein almost hamesha)

Concept

@SpringBootApplication kya hai? Auto-configuration kaise kaam karta hai?
@Component vs @Service vs @Repository vs @Controller?
Bean scopes? Default scope? (singleton)
@Autowired vs constructor injection (constructor preferred)?
@RestController vs @Controller?
Spring Data JPA: @Entity, @OneToMany, Lazy vs Eager loading?
@Transactional kaise kaam karta hai?
Spring Security + JWT flow?

Scenario

Q: N+1 query problem kya hai aur kaise fix karoge?
A: Ek parent fetch karne par har child ke liye alag query chalti hai. JOIN FETCH, @EntityGraph ya batch size se fix karo.
Q: @Transactional method same class ke andar se call karne par kaam nahi kar raha.
A: Spring proxy-based AOP use karta hai. Self-invocation proxy bypass kar deta hai. Method alag bean mein daalo.
Q: API ko 1000 req/sec handle karna hai.
A: Redis caching, DB indexing, pagination, connection pooling (HikariCP), async processing (Kafka/BullMQ jaisa queue), horizontal scaling.
Q: Global exception handling kaise karoge?
java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<Map<String, String>> handle(ResourceNotFoundException e) {
        return ResponseEntity.status(404).body(Map.of("error", e.getMessage()));
    }
}
10. Coding Round (Java mein practice karo)

Yeh sab fresher interviews mein bahut common hain:

String / Number reverse, Palindrome, Anagram
Fibonacci, Factorial, Prime check
Array mein duplicate / missing number / second largest
Two Sum (HashMap approach)
Linked List reverse, cycle detection (Floyd)
Binary Search aur uske variants
Valid Parentheses (Stack)
LRU Cache, Producer-Consumer
Merge intervals, Sliding window (longest substring without repeating chars)
Tree traversals, BFS/DFS
java
// Two Sum - O(n)
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int need = target - nums[i];
        if (map.containsKey(need)) return new int[]{map.get(need), i};
        map.put(nums[i], i);
    }
    return new int[]{};
}