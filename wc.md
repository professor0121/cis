# Complete Software Developer Interview Preparation — Questions & Answers

> Covers **Core Java, OOPs, SQL & DBMS (+ Practical Queries), DSA, Logical Coding (Java), JS/React/Node.js, Agile/SDLC, and HR Questions**.

---

## 1. Core Java Interview Questions

### 1.1 What is Java? What are its features?
Java is a high-level, object-oriented, platform-independent programming language developed by Sun Microsystems (now Oracle).
- **Key Features:**
  1. **Platform Independent:** Compiled into bytecode (`.class`) which runs on any OS with a JVM ("Write Once, Run Anywhere").
  2. **Object-Oriented:** Everything revolves around classes and objects.
  3. **Simple & Robust:** Automatic Garbage Collection, strong type checking, exception handling, no explicit pointers.
  4. **Secure:** Bytecode verification, sandboxed execution, no direct memory pointer manipulation.
  5. **Multithreaded:** Built-in support for concurrent execution (`Thread`, `ExecutorService`, Virtual Threads).
  6. **High Performance:** JIT (Just-In-Time) compiler optimizes hot bytecode to native machine code at runtime.

---

### 1.2 What is the difference between JDK, JRE, and JVM?
- **JVM (Java Virtual Machine):** Executes Java bytecode and manages memory/GC. Platform-dependent, makes Java programs platform-independent.
- **JRE (Java Runtime Environment):** `JVM + Core Class Libraries` required to **run** Java applications.
- **JDK (Java Development Kit):** `JRE + Development Tools` (`javac`, `jdb`, `jar`, `javadoc`) required to **write, compile, and run** Java applications.

---

### 1.3 What is the difference between C, C++, and Java?
| Feature | C | C++ | Java |
|---|---|---|---|
| **Paradigm** | Procedural | Procedural + Object-Oriented | Object-Oriented (plus functional lambdas) |
| **Platform** | Platform-dependent | Platform-dependent | Platform-independent (via JVM) |
| **Memory Management** | Manual (`malloc`/`free`) | Manual (`new`/`delete` + smart pointers) | Automatic (Garbage Collector) |
| **Pointers** | Full pointer support | Full pointer support | No explicit pointers (safe references) |
| **Multiple Inheritance** | N/A | Supported for classes | Only via Interfaces for classes |

---

### 1.4 What is the difference between `==` and `.equals()`?
- **`==`:** Compares **values** for primitives (`int`, `char`), and **memory addresses (references)** for objects.
- **`.equals()`:** Method in `Object` class used to compare **actual content/logical equality** of two objects (overridden in `String`, `Integer`, `List`, etc.).
```java
String s1 = new String("hello");
String s2 = new String("hello");
System.out.println(s1 == s2);      // false (different objects)
System.out.println(s1.equals(s2)); // true  (same text)
```

---

### 1.5 What are primitive and non-primitive data types?
- **Primitive Data Types (8 types):** Stored directly as values (on stack/inside objects), fixed size, not objects:
  `byte` (1B), `short` (2B), `int` (4B), `long` (8B), `float` (4B), `double` (8B), `char` (2B), `boolean` (JVM-dependent, logically 1 bit).
- **Non-Primitive (Reference) Data Types:** Objects stored on the Heap, variables hold references, can be `null`, have methods:
  `String`, Arrays, Classes, Interfaces, Enums, Collections (`ArrayList`, `HashMap`).

---

### 1.6 What is a constructor in Java?
A special block/method called automatically when an object is created with `new`, used to initialize the object's state. It has the **same name as the class** and **no return type**.
- **Types:** Default (no-arg), Parameterized, Copy constructor (user-defined in Java).

---

### 1.7 What is the difference between a constructor and a method?
| Feature | Constructor | Method |
|---|---|---|
| **Purpose** | Initializes a new object | Defines object behavior / performs operations |
| **Name** | Must match class name | Any valid identifier |
| **Return Type** | None (not even `void`) | Must specify a return type (or `void`) |
| **Invocation** | Implicitly called via `new` | Explicitly called on object/class |
| **Inherited?** | No (called via `super()`) | Yes |

---

### 1.8 What is the difference between `static` and non-static methods?
- **`static` method:** Belongs to the **class**, called as `ClassName.method()`. Cannot access non-static (instance) variables/methods directly because it has no `this` reference, and cannot be overridden (only hidden).
- **Non-static (instance) method:** Belongs to an **object**, called via `obj.method()`. Can access both static and instance members and supports runtime overriding.

---

### 1.9 What is the difference between `String`, `StringBuilder`, and `StringBuffer`?
- **`String`:** Immutable — every modification creates a new object in memory. Thread-safe.
- **`StringBuilder`:** Mutable, **not thread-safe** (unsynchronized), **fastest** — best for single-threaded loops/concatenation.
- **`StringBuffer`:** Mutable, **thread-safe** (`synchronized` methods), slightly slower than `StringBuilder`.

---

### 1.10 What is exception handling?
Exception handling is a mechanism (`try`, `catch`, `finally`, `throw`, `throws`) to gracefully handle runtime errors (like division by zero, missing files, invalid input) so the program doesn't crash abruptly and can recover or release resources cleanly.

---

### 1.11 What is the difference between checked and unchecked exceptions?
- **Checked Exceptions** (extend `Exception`, not `RuntimeException` — e.g., `IOException`, `SQLException`): Checked at **compile time**; must be handled with `try-catch` or declared with `throws`.
- **Unchecked Exceptions** (extend `RuntimeException` — e.g., `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException`): Occur at **runtime**; usually represent coding bugs, not mandatory to declare.

---

### 1.12 What is the use of the `final` keyword?
- **`final` variable:** Value/reference cannot be reassigned (constant).
- **`final` method:** Cannot be overridden by subclasses.
- **`final` class:** Cannot be inherited/extended (e.g., `String`).

---

### 1.13 What is method overloading and method overriding?
- **Method Overloading (Compile-time):** Multiple methods in the same class with the **same name** but **different parameters** (number, type, or order).
- **Method Overriding (Runtime):** A subclass provides its own implementation for an inherited method with the **exact same name and parameters**.

---

### 1.14 What is the difference between an abstract class and an interface?
- **Abstract Class:** Can have instance fields (state), constructors, abstract methods, and concrete methods. A class can `extends` **only one** abstract class.
- **Interface:** Has `public static final` constants, no constructors, abstract methods (plus `default` and `static` methods since Java 8, `private` helpers since Java 9). A class can `implements` **multiple** interfaces.

---

### 1.15 What is the difference between `Array` and `ArrayList`?
| Feature | Array (`int[]`) | `ArrayList<Integer>` |
|---|---|---|
| **Size** | Fixed once created | Dynamic (resizes automatically) |
| **Data Types** | Stores both primitives and objects | Stores objects only (uses wrapper classes like `Integer`) |
| **Methods / Utilities** | Only `.length` | Rich API (`add`, `remove`, `contains`, `size`, Streams) |
| **Performance** | Slightly faster, less memory | Slight boxing/resizing overhead |

---

## 2. OOPs Concepts (Very Important)

### 2.1 What is OOPs?
**Object-Oriented Programming (OOP)** is a programming paradigm that models software around **objects** (real-world entities combining data/attributes and methods/behavior), improving modularity, code reusability, security, and maintainability.

---

### 2.2 What are the four pillars of OOPs? (with Real-Life Examples & Java Code)
1. **Encapsulation:** Wrapping data + methods together and hiding internal fields (`private`) behind controlled getters/setters.
   - *Real-life example:* A **Bank ATM / Capsule** — you can't directly edit your bank balance in memory; you go through `deposit()` / `withdraw()` validation.
2. **Abstraction:** Showing only essential functionality and hiding internal complexity.
   - *Real-life example:* **Car accelerator & steering** — you press the pedal to speed up without knowing how fuel injection or pistons work.
3. **Inheritance:** A child class acquiring properties and methods of a parent class (`extends`) for code reuse and "is-a" hierarchy.
   - *Real-life example:* `Vehicle` (has `speed`, `start()`) → `Car` and `Bike` inherit those features and add their own.
4. **Polymorphism:** One interface/method name behaving differently depending on context or object type.
   - *Real-life example:* A **Payment button (`pay()`)** works differently for UPI, Credit Card, or NetBanking.

```java
// Combined Java Example of all 4 OOPs Pillars:
abstract class Payment {                         // Abstraction
    private double amount;                       // Encapsulation (data hiding)
    public Payment(double amount) { this.amount = amount; }
    public double getAmount() { return amount; }
    public abstract void pay();                  // Abstract contract
}

class UpiPayment extends Payment {               // Inheritance
    public UpiPayment(double amount) { super(amount); }
    @Override
    public void pay() {                          // Polymorphism (Overriding)
        System.out.println("Paid Rs." + getAmount() + " via UPI");
    }
}

class CardPayment extends Payment {
    public CardPayment(double amount) { super(amount); }
    @Override
    public void pay() {
        System.out.println("Paid Rs." + getAmount() + " via Card");
    }
}

// Usage:
// Payment p = new UpiPayment(500);
// p.pay(); // Runtime polymorphism -> calls UpiPayment.pay()
```

---

### 2.3 Explain inheritance with an example.
Inheritance allows a subclass (`Derived`) to reuse fields and methods of a superclass (`Base`) using `extends`:
```java
class Animal {
    void eat() { System.out.println("Eating..."); }
}
class Dog extends Animal {
    void bark() { System.out.println("Barking..."); }
}
// Dog d = new Dog(); d.eat(); d.bark();
```

---

### 2.4 What is polymorphism?
Polymorphism ("many forms") allows a single method or reference to take multiple forms—either at compile time (**overloading**) or at runtime (**overriding** via parent reference pointing to child objects).

---

### 2.5 What is encapsulation?
Encapsulation binds data (`private` variables) and methods (`public` getters/setters) into a single class, protecting internal state from unauthorized or invalid direct modification.

---

### 2.6 What is abstraction?
Abstraction exposes only necessary operations to the user while hiding implementation details, achieved in Java using **`abstract` classes** and **`interface`s**.

---

### 2.7 What is the difference between abstraction and encapsulation?
- **Abstraction** hides **implementation complexity** (focuses on *what* an object does — design level, using interfaces/abstract classes).
- **Encapsulation** hides **internal data** (focuses on *how* data is protected and bundled — implementation level, using `private` + getters/setters).

---

### 2.8 What are the different types of inheritance?
1. **Single:** `A → B`
2. **Multilevel:** `A → B → C`
3. **Hierarchical:** `A → B` and `A → C`
4. **Multiple:** `A, B → C` *(in Java, only via Interfaces)*
5. **Hybrid:** Combination of two or more types *(in Java, via Interfaces)*

---

### 2.9 Does Java support multiple inheritance?
- **For classes: No**, Java does NOT support multiple class inheritance (`class C extends A, B` is illegal) to avoid the **Diamond Problem** (ambiguity when both parents have the same method/state).
- **For interfaces: Yes**, a class can implement multiple interfaces (`class C implements A, B`). If two interfaces share a `default` method, the class must explicitly override it.

---

### 2.10 What is runtime polymorphism?
Also called **Dynamic Method Dispatch**: when a parent class reference points to a child class object (`Animal a = new Dog();`), the JVM decides **at runtime** (based on the actual object, not reference type) which overridden method (`Dog.sound()`) to execute.

---

### 2.11 What is compile-time polymorphism?
Resolved by the compiler at compile time based on method signature—implemented in Java via **Method Overloading** (`add(int, int)` vs `add(double, double)`).

---

### 2.12 What is the difference between an interface and an abstract class?
*(See Section 1.14)* — Key takeaway: Use an **abstract class** when closely related classes share common state/code; use an **interface** to define a common capability contract across unrelated classes (and when multiple inheritance is needed).

---

### 2.13 What is the use of the `super` keyword?
`super` refers to the immediate **parent class**:
1. `super()` — calls parent class constructor (must be first statement in child constructor).
2. `super.method()` — calls overridden parent method.
3. `super.field` — accesses hidden parent field.

---

### 2.14 What is the use of the `this` keyword?
`this` refers to the **current object**:
1. Distinguishes instance variables from parameters (`this.name = name;`).
2. `this(...)` — calls another constructor in the same class (constructor chaining).
3. Passes current object as an argument or returns `*this` for method chaining.

---

### 2.15 What is data hiding?
Data hiding restricts direct external access to class fields using `private` (or `protected`), forcing all reads/writes through validated public methods so internal invariants can't be broken.

---

## 3. SQL and DBMS Interview Questions

### Theory Questions

#### 3.1 What is DBMS?
**Database Management System (DBMS)** is software used to store, retrieve, update, and manage data in databases efficiently (e.g., MySQL, PostgreSQL, Oracle, MongoDB).

#### 3.2 What is RDBMS?
**Relational DBMS** is a type of DBMS based on the relational model (E.F. Codd) that stores data in structured **tables (rows & columns)** linked via **keys (Primary/Foreign)** and enforces ACID transactions. Examples: MySQL, PostgreSQL.

#### 3.3 What is the difference between DBMS and RDBMS?
- **DBMS** can store data as files/hierarchies/documents without enforcing table relationships or strict normalization (e.g., XML/file-based DBMS, older systems).
- **RDBMS** stores data strictly in **tables**, enforces **relationships (Foreign Keys)**, supports **SQL**, **normalization**, and **ACID** properties.

#### 3.4 What is SQL?
**Structured Query Language (SQL)** is the standard language for creating, querying, updating, and managing data in relational databases.

#### 3.5 What are DDL, DML, DCL, and TCL?
- **DDL (Data Definition Language):** Defines schema — `CREATE`, `ALTER`, `DROP`, `TRUNCATE`
- **DML (Data Manipulation Language):** Manipulates row data — `INSERT`, `UPDATE`, `DELETE` *(plus `SELECT` as DQL)*
- **DCL (Data Control Language):** Permissions — `GRANT`, `REVOKE`
- **TCL (Transaction Control Language):** Transactions — `COMMIT`, `ROLLBACK`, `SAVEPOINT`

#### 3.6 What is a primary key?
A column (or combination of columns) that **uniquely identifies every row** in a table. It must be **unique** and **`NOT NULL`**, and a table can have only one primary key.

#### 3.7 What is a foreign key?
A column (or set of columns) in one table that references the **Primary Key** (or Unique Key) of another table, enforcing **referential integrity** between tables.

#### 3.8 What is the difference between Primary Key and Unique Key?
- **Primary Key:** Cannot be `NULL`, only **one** per table, creates a clustered index by default (in MySQL InnoDB / SQL Server).
- **Unique Key:** Allows `NULL` values, can have **multiple** unique keys in a table, creates a non-clustered unique index.

#### 3.9 What is the difference between `DELETE`, `DROP`, and `TRUNCATE`?
- **`DELETE`:** DML; deletes specific rows (with `WHERE`) or all rows; can be rolled back; slower (row-by-row logging); does not reset auto-increment.
- **`TRUNCATE`:** DDL; removes **all** rows at once; faster; resets auto-increment in MySQL; cannot use `WHERE`.
- **`DROP`:** DDL; deletes the **entire table structure + data + indexes** permanently.

#### 3.10 What are Joins in SQL?
Joins combine rows from two or more tables based on a related column (usually Primary Key = Foreign Key).

#### 3.11 Explain `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, and `FULL JOIN`.
- **`INNER JOIN`:** Returns only matching rows present in **both** tables.
- **`LEFT JOIN`:** Returns **all rows from left table** + matching rows from right table (`NULL` if no match).
- **`RIGHT JOIN`:** Returns **all rows from right table** + matching rows from left table (`NULL` if no match).
- **`FULL JOIN` (`FULL OUTER JOIN`):** Returns all rows from **both** tables (`NULL` where no match; supported natively in Postgres, emulated via `UNION` in MySQL).

#### 3.12 What is normalization?
Normalization is the process of organizing tables and columns to **reduce data redundancy** and **eliminate insertion, update, and deletion anomalies**.

#### 3.13 What are the different types of normalization?
- **1NF:** Atomic (single) values per cell, no repeating groups.
- **2NF:** In 1NF + no partial dependency (every non-key attribute depends on the *entire* composite primary key).
- **3NF:** In 2NF + no transitive dependency (non-key column depends only on primary key, not another non-key column).
- **BCNF:** Stricter 3NF where every determinant is a candidate/super key.

#### 3.14 What is a stored procedure?
A precompiled, named block of SQL statements stored inside the database that can accept parameters, contain control logic (`IF`, loops), and be reused via `CALL proc_name()`.

#### 3.15 What is a view in SQL?
A virtual table created from a saved `SELECT` query. It stores no physical data (unless it's a Materialized View), simplifies complex queries, and improves security by exposing only specific columns/rows.

#### 3.16 What is an index?
A database data structure (usually a **B-Tree**) built on one or more columns that speeds up `SELECT`/`WHERE`/`ORDER BY` lookups from $O(n)$ full table scans to $O(\log n)$, at the cost of extra disk space and slightly slower `INSERT`/`UPDATE`/`DELETE`.

#### 3.17 What is the difference between `WHERE` and `HAVING`?
- **`WHERE`** filters individual rows **before** `GROUP BY` and cannot use aggregate functions.
- **`HAVING`** filters grouped results **after** `GROUP BY` and can use aggregate functions (`SUM`, `COUNT`, `AVG`).

#### 3.18 What is `GROUP BY`?
Groups rows that have identical values in specified columns into summary rows, typically used with aggregate functions (`COUNT`, `SUM`, `AVG`, `MAX`, `MIN`).

#### 3.19 What is the difference between `UNION` and `UNION ALL`?
- **`UNION`:** Combines results of two queries and **removes duplicates** (slower).
- **`UNION ALL`:** Combines results and **keeps all duplicates** (faster).

#### 3.20 What is a subquery?
A query nested inside another SQL query (`SELECT`, `FROM`, or `WHERE`), used to pass an intermediate result to the outer query.

---

### SQL Practical Questions (Q1 – Q8)

```sql
-- Q1. Find the second-highest salary from an Employee table.
SELECT MAX(salary) AS second_highest_salary
FROM Employee
WHERE salary < (SELECT MAX(salary) FROM Employee);

-- Q2. Display employees whose salary is greater than 30,000.
SELECT * FROM Employee WHERE salary > 30000;

-- Q3. Find duplicate records in a table (e.g., duplicate email).
SELECT email, COUNT(*) AS cnt
FROM Employee
GROUP BY email
HAVING COUNT(*) > 1;

-- Q4. Count the total number of employees in each department.
SELECT dept_id, COUNT(*) AS emp_count
FROM Employee
GROUP BY dept_id;

-- Q5. Find the highest salary in each department.
SELECT dept_id, MAX(salary) AS max_salary
FROM Employee
GROUP BY dept_id;

-- Q6. Display employees whose names start with 'A'.
SELECT * FROM Employee WHERE name LIKE 'A%';

-- Q7. Find the total salary of all employees.
SELECT SUM(salary) AS total_salary FROM Employee;

-- Q8. Join Employee and Department tables to display employee names with department names.
SELECT e.name AS employee_name, d.name AS department_name
FROM Employee e
LEFT JOIN Department d ON e.dept_id = d.id;
```

---

## 4. Data Structures and Algorithms

### 4.1 What is a data structure?
A systematic way of organizing, storing, and managing data in memory so operations like searching, inserting, updating, and deleting can be performed efficiently.

### 4.2 What is an array?
A contiguous block of memory storing elements of the **same data type**, supporting $O(1)$ random index access (`arr[i]`). Fixed size in C++/Java.

### 4.3 What is a linked list?
A linear data structure made of **nodes**, where each node stores data and a pointer/reference to the next node (non-contiguous memory).

### 4.4 What is the difference between an array and a linked list?
- **Array:** Contiguous memory, $O(1)$ random access, $O(n)$ insertion/deletion in middle, fixed size, better CPU cache locality.
- **Linked List:** Non-contiguous memory, $O(n)$ access/search, $O(1)$ insert/delete given a node pointer, dynamic size, extra memory for pointers.

### 4.5 What is a stack?
A linear **LIFO (Last In, First Out)** data structure where elements are inserted (`push`) and removed (`pop`) from the same end (`top`). Used in function call stacks, undo/redo, and balanced parentheses.

### 4.6 What is a queue?
A linear **FIFO (First In, First Out)** data structure where elements are inserted at the `rear` (`enqueue`) and removed from the `front` (`dequeue`). Used in scheduling, BFS, and task queues.

### 4.7 What is a linear search?
Checks every element sequentially from start to end until the target is found. Works on unsorted arrays; **$O(n)$** time complexity.

### 4.8 What is binary search?
Efficient search on a **sorted array** that repeatedly divides the search space in half by comparing the middle element with the target. **$O(\log n)$** time complexity.

### 4.9 What is sorting?
Arranging elements in a specific order (ascending or descending) to make searching, deduplication, and merging faster (e.g., Merge Sort, Quick Sort, Bubble Sort).

### 4.10 Explain bubble sort.
Repeatedly steps through the array, compares adjacent elements, and swaps them if they are in the wrong order—the largest element "bubbles up" to the end after each pass. Time: **$O(n^2)$** worst/average, $O(n)$ best (if already sorted with swapped flag); Space: $O(1)$.

### 4.11 What is recursion?
A technique where a function calls itself directly or indirectly to solve smaller subproblems until reaching a **base case** (which stops recursion to prevent `StackOverflowError`).

### 4.12 What is time complexity?
A measure of how the **execution time (number of operations)** of an algorithm grows as the input size $n$ increases.

### 4.13 What is Big O notation?
Mathematical notation describing the **upper bound (worst-case growth rate)** of time or space complexity as $n \to \infty$ (e.g., $O(1)$, $O(\log n)$, $O(n)$, $O(n \log n)$, $O(n^2)$).

### 4.14 What is the difference between stack and queue?
- **Stack:** **LIFO** (Last In, First Out) — single open end (`top`); operations: `push`, `pop`, `peek`.
- **Queue:** **FIFO** (First In, First Out) — two ends (`front` and `rear`); operations: `enqueue`/`offer`, `dequeue`/`poll`.

### 4.15 What is a `HashMap`?
A key-value data structure using a **hash function** to map keys to array bucket indexes, giving **$O(1)$ average time** for `put`, `get`, and `remove` (and $O(\log n)$ worst-case per bucket in Java 8+ via Red-Black Tree conversion on high collisions).

---

## 5. Programming and Logical Coding Questions (Java)

```java
import java.util.*;

public class CodingPrep {

    // 1. Reverse a string — O(n) time, O(n) space
    public static String reverseString(String s) {
        char[] arr = s.toCharArray();
        for (int l = 0, r = arr.length - 1; l < r; l++, r--) {
            char tmp = arr[l]; arr[l] = arr[r]; arr[r] = tmp;
        }
        return new String(arr);
    }

    // 2. Check whether a number is prime — O(sqrt(n)) time, O(1) space
    public static boolean isPrime(int n) {
        if (n <= 1) return false;
        if (n <= 3) return true;
        if (n % 2 == 0 || n % 3 == 0) return false;
        for (int i = 5; i * i <= n; i += 6) {
            if (n % i == 0 || n % (i + 2) == 0) return false;
        }
        return true;
    }

    // 3. Check whether a string is palindrome — O(n) time, O(1) space
    public static boolean isPalindrome(String s) {
        int l = 0, r = s.length() - 1;
        while (l < r) {
            if (s.charAt(l++) != s.charAt(r--)) return false;
        }
        return true;
    }

    // 4. Find factorial of a number — O(n) time, O(1) space
    public static long factorial(int n) {
        long res = 1;
        for (int i = 2; i <= n; i++) res *= i;
        return res;
    }

    // 5. Print Fibonacci series — O(n) time, O(1) space
    public static void printFibonacci(int count) {
        long a = 0, b = 1;
        for (int i = 0; i < count; i++) {
            System.out.print(a + (i < count - 1 ? " " : "\n"));
            long next = a + b;
            a = b;
            b = next;
        }
    }

    // 6. Find the largest element in an array — O(n) time, O(1) space
    public static int findLargest(int[] arr) {
        int max = arr[0];
        for (int i = 1; i < arr.length; i++) {
            if (arr[i] > max) max = arr[i];
        }
        return max;
    }

    // 7. Find the second-largest element in an array — O(n) time, O(1) space
    public static int findSecondLargest(int[] arr) {
        int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
        for (int x : arr) {
            if (x > first) {
                second = first;
                first = x;
            } else if (x > second && x != first) {
                second = x;
            }
        }
        return second;
    }

    // 8. Reverse a number — O(log10 n) time, O(1) space
    public static int reverseNumber(int n) {
        int rev = 0;
        while (n != 0) {
            rev = rev * 10 + (n % 10);
            n /= 10;
        }
        return rev;
    }

    // 9. Check whether a number is Armstrong (e.g., 153 = 1^3 + 5^3 + 3^3)
    public static boolean isArmstrong(int n) {
        if (n < 0) return false;
        int digits = String.valueOf(n).length();
        int temp = n, sum = 0;
        while (temp > 0) {
            int d = temp % 10;
            sum += (int) Math.pow(d, digits);
            temp /= 10;
        }
        return sum == n;
    }

    // 10. Count vowels in a string — O(n) time, O(1) space
    public static int countVowels(String s) {
        int count = 0;
        for (char c : s.toLowerCase().toCharArray()) {
            if (c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u') count++;
        }
        return count;
    }

    // 11. Find duplicate elements in an array — O(n) time, O(n) space
    public static List<Integer> findDuplicates(int[] arr) {
        Set<Integer> seen = new HashSet<>();
        Set<Integer> dups = new LinkedHashSet<>();
        for (int x : arr) {
            if (!seen.add(x)) dups.add(x);
        }
        return new ArrayList<>(dups);
    }

    // 12. Sort an array without using built-in methods (Bubble Sort) — O(n^2) time, O(1) space
    public static void sortArray(int[] arr) {
        for (int i = 0; i < arr.length - 1; i++) {
            boolean swapped = false;
            for (int j = 0; j < arr.length - 1 - i; j++) {
                if (arr[j] > arr[j + 1]) {
                    int tmp = arr[j]; arr[j] = arr[j + 1]; arr[j + 1] = tmp;
                    swapped = true;
                }
            }
            if (!swapped) break;
        }
    }

    // 13. Swap two numbers without using a third variable
    public static void swapWithoutTemp(int a, int b) {
        a = a ^ b; // or: a = a + b; b = a - b; a = a - b;
        b = a ^ b;
        a = a ^ b;
        System.out.println("a=" + a + ", b=" + b);
    }

    // 14. Find the sum of digits of a number — O(log10 n) time, O(1) space
    public static int sumOfDigits(int n) {
        n = Math.abs(n);
        int sum = 0;
        while (n > 0) {
            sum += n % 10;
            n /= 10;
        }
        return sum;
    }

    // 15. Remove duplicate characters from a string (preserving order) — O(n) time
    public static String removeDuplicateChars(String s) {
        Set<Character> seen = new LinkedHashSet<>();
        for (char c : s.toCharArray()) seen.add(c);
        StringBuilder sb = new StringBuilder();
        for (char c : seen) sb.append(c);
        return sb.toString();
    }
}
```

---

## 6. JavaScript, React, Angular, and Node.js

### 6.1 What is JavaScript?
JavaScript is a high-level, single-threaded, dynamically typed, interpreted/JIT-compiled programming language used to make web pages interactive on the browser and build backend services with Node.js.

### 6.2 What is the difference between `var`, `let`, and `const`?
| Feature | `var` | `let` | `const` |
|---|---|---|---|
| **Scope** | Function-scoped | Block-scoped (`{}`) | Block-scoped (`{}`) |
| **Hoisting** | Hoisted & initialized as `undefined` | Hoisted in **Temporal Dead Zone (TDZ)** (throws `ReferenceError`) | Hoisted in **TDZ** |
| **Reassignable** | Yes | Yes | **No** *(note: object/array properties can still be mutated)* |
| **Redeclare in same scope** | Yes | No | No |

### 6.3 What is the difference between `==` and `===`?
- **`==` (Loose Equality):** Performs **type coercion** before comparing (`5 == "5"` → `true`).
- **`===` (Strict Equality):** Compares **both value AND type** without coercion (`5 === "5"` → `false`). Always prefer `===`.

### 6.4 What is DOM?
The **Document Object Model (DOM)** is a tree-like programming representation of an HTML document created by the browser, allowing JavaScript to dynamically read, add, update, or delete elements, attributes, and styles.

### 6.5 What is an event in JavaScript?
An **event** is an action or occurrence detected by the browser (e.g., `click`, `submit`, `keydown`, `input`, `load`) to which JavaScript can respond using event listeners (`element.addEventListener('click', handler)`).

### 6.6 What is an arrow function?
A concise ES6 function syntax (`const add = (a, b) => a + b;`) that **does not bind its own `this`** (inherits `this` lexically from its enclosing scope) and cannot be used as a constructor with `new`.

### 6.7 What is the difference between `null` and `undefined`?
- **`undefined`:** Variable is declared but **not yet assigned** a value (default state by JS engine). `typeof undefined === "undefined"`.
- **`null`:** **Intentional assignment** representing "no value / empty". `typeof null === "object"` (historical JS quirk).

### 6.8 What is an array in JavaScript?
A dynamic, ordered list-like object capable of holding mixed data types (`[1, "hello", true]`), with built-in higher-order methods like `map()`, `filter()`, `reduce()`, `forEach()`, and `find()`.

### 6.9 What is React?
React is an open-source JavaScript **library** (by Meta) for building fast, reusable, component-based user interfaces using a **Virtual DOM** and declarative rendering.

### 6.10 What are components in React?
Independent, reusable UI building blocks (modern React uses **Functional Components** with Hooks like `useState`, `useEffect`) that accept inputs (`props`) and return JSX describing what appears on screen.

### 6.11 What are `props` and `state`?
- **`props` (Properties):** Read-only data passed **from parent to child** component (immutable inside the child).
- **`state`:** Mutable data **managed internally** by a component (`useState`); updating state triggers a re-render of the component.

### 6.12 What is Node.js?
Node.js is an open-source, cross-platform JavaScript **runtime environment** built on Chrome's V8 engine that lets you run JS outside the browser to build scalable, non-blocking, event-driven backend APIs and servers.

### 6.13 What is the difference between frontend and backend?
- **Frontend (Client-side):** Everything the user sees and interacts with in the browser (HTML, CSS, JavaScript, React/Angular).
- **Backend (Server-side):** Business logic, authentication, APIs, and database communication running on the server (Java/Spring Boot, Node.js/Express, MySQL/PostgreSQL).

### 6.14 What is an API?
**Application Programming Interface (API)** is a contract/set of rules that allows two software applications to communicate. In web development, REST APIs exchange data over HTTP (`GET`, `POST`, `PUT`, `DELETE`) typically in JSON format.

### 6.15 What is JSON?
**JavaScript Object Notation (JSON)** is a lightweight, human-readable, language-independent text format for storing and exchanging structured data using key-value pairs (`{"name": "Abhishek", "role": "Developer"}`).

---

## 7. Agile and Software Development Questions

### 7.1 What is SDLC?
**Software Development Life Cycle (SDLC)** is a structured process followed by software teams to plan, design, build, test, deploy, and maintain high-quality software within time and budget.

### 7.2 Explain the different phases of SDLC.
1. **Requirement Gathering & Analysis** — Understand client needs and define SRS.
2. **Planning & Feasibility** — Estimate timeline, cost, tech stack, and resources.
3. **Design (HLD & LLD)** — Architecture, database schema, UI/UX, and API contracts.
4. **Development (Coding)** — Engineers write code following coding standards.
5. **Testing (QA)** — Unit, integration, system, and UAT testing to catch bugs.
6. **Deployment** — Release to production via CI/CD pipelines.
7. **Maintenance** — Monitoring, bug fixes, security patches, and updates.

### 7.3 What is Agile methodology?
An iterative, incremental approach to software development where work is delivered in small, working releases with continuous customer feedback, collaboration, and adaptability to changing requirements (unlike rigid Waterfall).

### 7.4 What is Scrum?
A popular **Agile framework** with defined roles (**Product Owner, Scrum Master, Development Team**), ceremonies (**Sprint Planning, Daily Stand-up, Sprint Review, Sprint Retrospective**), and artifacts (**Product Backlog, Sprint Backlog, Increment**).

### 7.5 What is a sprint?
A fixed, short time-box (typically **1 to 4 weeks**, most commonly 2 weeks) during which the Scrum team completes an agreed set of backlog tasks to produce a potentially shippable product increment.

### 7.6 What is software testing?
The process of verifying and validating that a software application meets requirements, works correctly across normal and edge cases, and is free of critical bugs before reaching users.

### 7.7 What is the difference between manual and automation testing?
- **Manual Testing:** A human tester executes test cases manually without scripts — great for exploratory, UI/UX, and usability testing.
- **Automation Testing:** Test scripts (e.g., JUnit, Selenium, Playwright, Cypress) run tests automatically — fast, repeatable, ideal for regression and CI/CD pipelines.

### 7.8 What is debugging?
The systematic process of locating, reproducing, analyzing root causes, and fixing bugs/errors in code using logs, breakpoints, stack traces, and debuggers.

### 7.9 What is Git?
A free, open-source **distributed version control system (DVCS)** used to track code changes locally and collaboratively via commits, branches, merges, and rollbacks.

### 7.10 What is GitHub?
A cloud-based hosting platform built around Git for storing remote repositories, managing Pull Requests (PRs), code reviews, issue tracking, and CI/CD (GitHub Actions).

### 7.11 What is code review?
A peer inspection of code changes (usually via a Pull Request) before merging, to catch bugs early, ensure clean readable code, maintain design/security standards, and share knowledge across the team.

### 7.12 Why is documentation important in software development?
Good documentation (READMEs, API specs like Swagger/OpenAPI, architecture notes) speeds up **onboarding**, reduces dependency on individual team members, prevents misunderstandings across frontend/backend teams, and makes long-term maintenance much easier.

---

## 8. HR Interview Questions (Tailored Sample Answers)

### 8.1 Tell me about yourself.
> "Good morning/afternoon. My name is **Abhishek Kushwaha**, and I am currently based in **Indore**. I completed my **B.Tech in Computer Science** from Gyan Sagar College of Engineering, Sagar in 2026 with a CGPA of **7.91**.  
> Technically, I have hands-on experience in **Java, JavaScript, React**, and databases like **MySQL, PostgreSQL, and MongoDB**, along with strong fundamentals in **OOPs and DSA**.  
> I also completed a Full-Stack Web Developer internship at **Madquick**, where I worked on real-world projects involving REST APIs, authentication, databases, and deployment. I'm a quick and consistent learner looking to start my career as a Software Developer and contribute to impactful production projects."

---

### 8.2 Why do you want to become a software developer?
> "I enjoy solving logical problems and turning ideas into working products that people actually use. During college and my internship at Madquick, building APIs, integrating databases, and seeing features go live gave me a strong sense of satisfaction. Software development keeps me learning continuously, which matches my personality."

---

### 8.3 Why should we hire you?
> "As a fresher, I bring both **strong core fundamentals** (Java, OOPs, SQL, DSA) and **practical internship experience** from Madquick where I worked on APIs, authentication, and deployment. I learn new stacks quickly, take ownership of tasks from start to finish, and can ramp up fast on your team's codebase."

---

### 8.4 What are your strengths and weaknesses?
- **Strength:** "I learn new technologies quickly, stay consistent, and take feedback constructively so I improve fast."
- **Weakness:** "Earlier, I sometimes used to take critical feedback personally or spend too long perfecting small details. Now I actively focus on separating the work from myself, asking clarifying questions, and prioritizing deliverables on time."

---

### 8.5 What do you know about our company?
> *(Customize before interview — check company website/LinkedIn!)*  
> "I researched **[Company Name]** and saw that you specialize in **[web/mobile/enterprise software solutions / tech stack / clients]**, and have a strong engineering presence here in **Indore**. I'm excited about the opportunity to work in a collaborative engineering environment where I can grow my Java/full-stack skills on real client projects."

---

### 8.6 Are you comfortable working in a team?
> "Yes, completely. During my internship at Madquick and college projects, I collaborated using Git/GitHub, participated in task planning, and worked closely with teammates on frontend-backend API integration. I value clear communication and helping teammates whenever needed."

---

### 8.7 How do you handle pressure?
> "I stay calm, break the problem down into smaller actionable steps, and prioritize what needs to be fixed first. During tight deadlines, I avoid rushing blindly—I verify root causes, communicate progress honestly with my mentor or team, and focus on delivering a clean solution."

---

### 8.8 Are you willing to learn new technologies?
> "Definitely. In fact, my goal as an early-career developer is to build strong versatility. Even though my primary comfort is in Java, React, and SQL/MongoDB, I am very open to learning Angular, Spring Boot, cloud tools, or whatever stack the project requires."

---

### 8.9 Where do you see yourself in five years?
> "In the next 2–3 years, I want to become a reliable, well-rounded Software Engineer who owns end-to-end features with clean, scalable code. Within 5 years, I see myself stepping into a Senior Developer role, mentoring juniors, and contributing to system architecture and large-scale projects."

---

### 8.10 Are you willing to relocate or work from our Indore office?
> "Yes, I am **already based in Indore**, so working from the Indore office is completely convenient for me—and I am also flexible if project travel or relocation is ever required."

---

### 8.11 Do you have any internship or project experience?
> "Yes. I completed a Full-Stack Web Developer internship at **Madquick**, where I worked on real-world web applications involving **REST APIs, user authentication, database integration, and deployment**. That experience taught me production workflows, debugging real issues, and writing maintainable code."

---

### 8.12 What are your salary expectations?
> "As a fresher, my primary focus is learning, contributing meaningfully to the team, and growing with the company. I am comfortable with your company's standard compensation structure for this role based on my skills and interview performance."
