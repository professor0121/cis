# Complete OOPs Concepts — Interview & Revision Guide (with C++ Examples)

> Covers every core, intermediate, advanced C++ internal, and OOP design concept from basics to SOLID & tricky interview edge cases.

---

## 1. Procedural Programming (POP) vs Object-Oriented Programming (OOP)

| Feature | Procedural (POP — e.g., C) | Object-Oriented (OOP — e.g., C++, Java) |
|---|---|---|
| **Focus** | Functions / step-by-step procedures | Objects (data + behavior bundled together) |
| **Approach** | Top-down | Bottom-up |
| **Data Security** | Global data is openly accessible (less secure) | Data hiding via `private` / `protected` access specifiers |
| **Code Reuse** | Function calls | Inheritance, Composition, Polymorphism, Templates |
| **Scalability** | Hard to maintain in large systems | Modular, easier to extend and maintain |

---

## 2. Class & Object

- **Class:** A user-defined blueprint/template describing properties (data members) and behaviors (member functions). **No memory is allocated** when a class is defined.
- **Object:** A concrete **instance** of a class. Memory is allocated when an object is created.

```cpp
#include <iostream>
#include <string>
using namespace std;

class Car {
public:
    string brand;
    int speed;

    void drive() {
        cout << brand << " driving at " << speed << " km/h\n";
    }
};

int main() {
    Car c1;                 // Stack allocation (automatically destroyed at scope end)
    c1.brand = "Toyota";
    c1.speed = 80;
    c1.drive();

    Car* c2 = new Car();    // Heap allocation (must manually `delete c2;`)
    c2->brand = "Honda";
    c2->speed = 100;
    c2->drive();
    delete c2;
}
```

### Important C++ Memory Facts:
1. **Empty Class Size:** `sizeof(Empty)` is **`1 byte`** (never `0`), so two distinct objects (`Empty a, b;`) have unique memory addresses (`&a != &b`).
2. **Functions don't add to `sizeof(object)`:** Member functions live in the code segment and are shared by all objects. Only non-static data members (+ alignment padding + `vptr` if virtual functions exist) contribute to `sizeof`.

---

## 3. Access Specifiers & Data Hiding

Control visibility of class members:
- **`private`:** Accessible **only inside the same class** (and `friend`s). Default for `class`.
- **`protected`:** Accessible inside the same class **and derived classes**.
- **`public`:** Accessible from anywhere. Default for `struct`.

### `class` vs `struct` in C++:
The **only** language differences are:
1. Default member access: `private` in `class`, `public` in `struct`.
2. Default inheritance mode: `private` in `class`, `public` in `struct`.
*(Both support constructors, destructors, inheritance, virtual functions, and templates!)*

---

## 4. Encapsulation

**Encapsulation** = Wrapping data (variables) and the methods that operate on that data into a single unit (`class`), while restricting direct access to internal state (**Data Hiding**) so invariants cannot be broken.

```cpp
class BankAccount {
private:
    double balance = 0;   // Hidden from outside

public:
    void deposit(double amount) {
        if (amount > 0) balance += amount;   // Enforces valid state
    }

    bool withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }

    double getBalance() const { return balance; }
};
```
- **Encapsulation vs Data Hiding:** Encapsulation is the structural bundling + controlled interface; Data Hiding (`private`) is the security mechanism used inside encapsulation to protect data integrity.

---

## 5. Abstraction

**Abstraction** = Showing only essential features ("**what** an object does") and hiding complex internal implementation details ("**how** it does it").

- **Achieved in C++ via:**
  1. **Abstract classes / Interfaces** (pure virtual functions).
  2. **Access specifiers + public API methods** (caller calls `car.startEngine()` without knowing fuel injection or ignition internals).

```cpp
class PaymentGateway { // Abstract interface
public:
    virtual void pay(double amount) = 0; // Only "what", not "how"
    virtual ~PaymentGateway() = default;
};

class UPIPayment : public PaymentGateway {
public:
    void pay(double amount) override {
        cout << "Processing UPI payment of Rs. " << amount << "\n";
    }
};
```

### Encapsulation vs Abstraction:
| Encapsulation | Abstraction |
|---|---|
| Focuses on **how data & methods are bundled and protected** | Focuses on **what is exposed vs hidden** at the design/interface level |
| Implemented using `private`/`protected` + getters/setters | Implemented using abstract classes, interfaces, and public APIs |
| Solves **data security / integrity** | Solves **design complexity** |

---

## 6. Constructors

A **constructor** is a special member function with the same name as the class and **no return type**, automatically called when an object is created to initialize it.

### Types of Constructors:

1. **Default Constructor:** Takes no arguments (compiler generates one if no user constructor is defined).
2. **Parameterized Constructor:** Initializes object with passed arguments.
3. **Copy Constructor:** Creates a new object from an existing object of the same class (`ClassName(const ClassName& other)`).
   - **Why `const T&` (by reference, not by value)?** Passing by value would try to copy the argument, which calls the copy constructor again → **infinite recursion**!
4. **Move Constructor (C++11):** Transfers ownership of resources (pointers/handles) from a temporary/rvalue object without deep copying (`ClassName(ClassName&& other) noexcept`).
5. **Delegating Constructor (C++11):** One constructor calls another constructor of the same class in its initializer list to avoid code duplication.
6. **`explicit` Constructor:** Prevents unintended implicit type conversions for single-argument constructors.

```cpp
class Buffer {
private:
    int* data;
    size_t size;

public:
    // 1. Parameterized + explicit (prevents `Buffer b = 10;` implicit conversion)
    explicit Buffer(size_t n) : data(new int[n]{}), size(n) {}

    // 2. Delegating Constructor
    Buffer() : Buffer(10) {}

    // 3. Copy Constructor (Deep Copy)
    Buffer(const Buffer& other) : data(new int[other.size]), size(other.size) {
        copy(other.data, other.data + size, data);
    }

    // 4. Move Constructor (steals resource, leaves `other` in valid empty state)
    Buffer(Buffer&& other) noexcept : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }

    ~Buffer() { delete[] data; }
};
```

### Shallow Copy vs Deep Copy:
- **Shallow Copy (Compiler Default):** Copies member values bit-by-bit. If the class holds a raw pointer (`int* data`), both objects point to the **same heap memory** → modifying one mutates the other, and destruction causes **double-free crash**!
- **Deep Copy:** Allocates separate heap memory and copies the actual pointed-to values.

### Member Initializer List (`: a(x), b(y)`):
- Faster than assigning inside `{ a = x; }` (avoids default-constructing then re-assigning).
- **Mandatory when initializing:**
  1. `const` data members
  2. Reference (`&`) data members
  3. Base class constructors without a default constructor
  4. Member objects whose class lacks a default constructor
- **Gotcha:** Members are initialized in **the order they are declared in the class**, NOT the order written in the initializer list!

---

## 7. Destructors & Virtual Destructors

A **destructor** (`~ClassName()`) has no parameters, no return type, **cannot be overloaded**, and runs automatically when an object goes out of scope or is `delete`d.

### Virtual Destructor (Crucial Interview Question!):
If you delete a `Derived` object through a `Base*` pointer and `Base`'s destructor is **NOT `virtual`**, only `~Base()` runs → **Derived's destructor is skipped → memory/resource leak (Undefined Behavior)!**
```cpp
class Base {
public:
    virtual ~Base() { cout << "~Base\n"; } // ALWAYS make base destructor virtual!
};

class Derived : public Base {
    int* arr = new int[100];
public:
    ~Derived() override {
        delete[] arr;
        cout << "~Derived\n";
    }
};

int main() {
    Base* p = new Derived();
    delete p; // Output: ~Derived  then  ~Base
}
```
- **Pure Virtual Destructor:** Allowed in C++ (`virtual ~Base() = 0;`) to make a class abstract when it has no other pure virtual methods, **but you MUST still provide a body** (`Base::~Base() {}`) because derived destructors always call the base destructor!

---

## 8. `this` Pointer

- Every non-static member function receives an implicit hidden pointer **`this`** pointing to the invoking object (`ClassName* const this`, or `const ClassName* const this` inside `const` functions).
- **Uses:**
  1. Disambiguate parameter names from member variables (`this->x = x;`).
  2. **Method Chaining** (`return *this;`).
  3. Self-assignment check in `operator=` (`if (this == &other) return *this;`).

```cpp
class Builder {
    int val = 0;
public:
    Builder& add(int x) { val += x; return *this; }
    Builder& mul(int x) { val *= x; return *this; }
    int get() const { return val; }
};
// Usage: int res = Builder().add(5).mul(3).get(); // 15
```

---

## 9. `static` Members in a Class

- **`static` Data Member:** Shared by **all objects** of the class (one single copy in static memory, exists even if 0 objects are created).
  - Pre-C++17: must be defined outside the class (`int Counter::count = 0;`).
  - C++17+: can use `inline static int count = 0;` inside the class.
- **`static` Member Function:** Belongs to the class, called via `ClassName::func()`.
  - Has **no `this` pointer** → can **only** access `static` members, cannot be `const`, and cannot be `virtual`.

```cpp
class User {
public:
    inline static int activeUsers = 0;
    User()  { ++activeUsers; }
    ~User() { --activeUsers; }
    static int getCount() { return activeUsers; }
};
```

---

## 10. `const` & `mutable` in OOP

- **`const` Member Function (`void print() const`):** Promises not to modify any non-static data members (`this` becomes pointer-to-`const`).
- **`const` Object (`const User u;`):** Can **only** call `const` member functions.
- **`const` Overloading:** You can overload a function based on `const`ness (`int& get()` vs `const int& get() const`).
- **`mutable`:** Allows a specific member variable (e.g., cache, mutex, call counter) to be modified even inside a `const` member function (**logical constness**).

```cpp
class Logger {
    mutable int readCount = 0;
    string msg = "Hello";
public:
    const string& read() const {
        ++readCount; // Allowed because readCount is mutable!
        return msg;
    }
};
```

---

## 11. `friend` Function & `friend` Class

A `friend` function or class (declared inside a class with `friend`) can access its `private` and `protected` members.

```cpp
class Box {
    int width = 10;
    friend void printWidth(const Box& b);
    friend class Inspector;
};

void printWidth(const Box& b) {
    cout << b.width << "\n"; // Accessing private member
}
```
- **Properties of Friendship:**
  1. **Not mutual:** If `A` declares `B` as friend, `B` can access `A`, not vice versa.
  2. **Not inherited:** Friend of Base is NOT friend of Derived.
  3. **Not transitive:** Friend of a friend is NOT a friend.
- **Common valid use case:** Operator overloading (`operator<<`, `operator+` when left operand isn't your class), testing utilities. Overuse weakens encapsulation.

---

## 12. Inheritance, Types, Diamond Problem & Relationships

**Inheritance ("is-a" relationship):** A derived (child) class acquires properties and behaviors of a base (parent) class.

### Visibility Modes in C++ Inheritance:
| Base Member Access | `public` Inheritance | `protected` Inheritance | `private` Inheritance |
|---|---|---|---|
| `public` in Base | `public` in Derived | `protected` in Derived | `private` in Derived |
| `protected` in Base | `protected` in Derived | `protected` in Derived | `private` in Derived |
| `private` in Base | Inaccessible directly | Inaccessible directly | Inaccessible directly |

*(Note: `private` members of Base ARE still part of Derived's object memory layout, just not directly accessible by name in Derived.)*

### 5 Types of Inheritance:
1. **Single:** `A → B`
2. **Multilevel:** `A → B → C`
3. **Multiple:** `A, B → C` (Supported in C++; Java uses `interface`s for this)
4. **Hierarchical:** `A → B` and `A → C`
5. **Hybrid:** Combination (e.g., Hierarchical + Multiple → causes Diamond Problem)

### Diamond Problem & Virtual Inheritance:
When `B` and `C` both inherit from `A`, and `D` inherits from both `B` and `C`, `D` ends up with **two separate copies of `A`** → calling `d.show()` is **ambiguous**!
- **Fix:** Use **Virtual Inheritance** (`class B : virtual public A`, `class C : virtual public A`), so `D` shares **one single copy** of `A`.
- **Important:** In virtual inheritance, the **most derived class (`D`)** is responsible for directly calling the virtual base (`A`)'s constructor!

```cpp
class A {
public:
    int x = 10;
    A(int v = 10) : x(v) {}
};
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {
public:
    D() : A(99) {} // Most derived class initializes virtual base A directly
};
```

### Constructor & Destructor Execution Order in Inheritance:
- **Constructors:** Base → Member objects (in declaration order) → Derived. *(Virtual base classes are constructed first!)*
- **Destructors:** Exact reverse: Derived → Member objects → Base.

### What is NOT Inherited?
- Constructors & Destructors
- Assignment operator (`operator=`)
- `friend` functions / classes

### Object Relationships (Association, Aggregation, Composition, Inheritance):
1. **Association ("uses-a"):** Independent objects communicating (e.g., `Teacher` ↔ `Student`). Neither owns the other's lifetime.
2. **Aggregation ("has-a", weak ownership):** Whole contains Part, **Part can survive without Whole** (passed via pointer/reference). E.g., `Department` has `Professor*` — if `Department` closes, `Professor` still exists.
3. **Composition ("part-of", strong ownership):** Whole owns Part's lifetime — **Part dies when Whole dies** (member by value or `unique_ptr`). E.g., `House` has `Room`.
4. **Inheritance ("is-a"):** Subtyping (`Dog` is an `Animal`).
*(Design Principle: **Prefer Composition over Inheritance** when you only need code reuse without tight coupling or fragile base-class bugs.)*

---

## 13. Polymorphism (Compile-Time & Runtime)

**Polymorphism** ("many forms") allows the same function/operator/interface to behave differently depending on types or runtime objects.

### A. Compile-Time Polymorphism (Static / Early Binding)
Resolved at compile time → zero runtime overhead.

#### 1. Function Overloading
Same function name in the same scope with **different parameter list** (number, types, or order of parameters — or `const` qualification on member functions).
- **Return type alone CANNOT overload** a function.
- **How C++ implements it:** **Name Mangling** (compiler encodes parameter types into internal symbol names, e.g., `_Z3addii` vs `_Z3adddd`).
- **Gotcha:** Watch out for default arguments causing ambiguity (`void f(int)` vs `void f(int, int = 0)`).

#### 2. Operator Overloading
Gives user-defined meaning to C++ operators for custom types.
- **Cannot overload:** `::` (scope), `.` (member access), `.*` (member pointer), `?:` (ternary), `sizeof`, `typeid`.
- **Must be member function (not global):** `=`, `[]`, `()`, `->`.
- **Must be non-member (`friend`):** `<<` and `>>` (because left operand is `ostream`/`istream`).

```cpp
class Complex {
public:
    double r, i;
    Complex(double r = 0, double i = 0) : r(r), i(i) {}

    Complex operator+(const Complex& o) const {
        return Complex(r + o.r, i + o.i);
    }

    // Prefix ++a vs Postfix a++ (dummy `int` differentiates postfix)
    Complex& operator++() { ++r; return *this; }
    Complex operator++(int) { Complex old = *this; ++r; return old; }

    friend ostream& operator<<(ostream& os, const Complex& c) {
        return os << c.r << " + " << c.i << "i";
    }
};
```

#### 3. Parametric Polymorphism (Templates)
Writing type-independent code resolved at compile time:
```cpp
template <typename T>
T maxVal(T a, T b) { return (a > b) ? a : b; }
```

---

### B. Runtime Polymorphism (Dynamic / Late Binding)
Resolved at runtime using **`virtual` functions** and a **Base pointer or reference**.

#### 1. Function Overriding, `override`, & `final`
- **`virtual`** in Base allows Derived to override the behavior dynamically.
- **`override` (C++11):** Compiler checks that you actually match a `virtual` base signature (catches typos like missing `const`!).
- **`final` (C++11):** Prevents further overriding on a virtual method (`void f() override final;`) OR prevents inheriting from a class (`class Secret final {};`).

```cpp
class Shape {
public:
    virtual double area() const = 0; // Pure virtual
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double r;
public:
    Circle(double r) : r(r) {}
    double area() const override { return 3.14159 * r * r; }
};
```

#### 2. How Virtual Functions Work Internally (`vtable` & `vptr`)
- **`vtable` (Virtual Table):** A per-class static lookup table of function pointers to the class's virtual functions. Created by the compiler for any class with virtual functions (or inheriting them).
- **`vptr` (Virtual Pointer):** A hidden pointer added to **every object** of that class (adds `sizeof(void*)` — 8 bytes on 64-bit systems), initialized by the constructor to point to that class's `vtable`.
- **Runtime call:** `basePtr->area()` looks up `basePtr->vptr[index_of_area]` and calls the derived version dynamically.
- **Trade-off:** Slight memory overhead (`vptr` per object) and one extra pointer dereference per call (cannot easily inline).

#### 3. Pure Virtual Functions, Abstract Classes & Interfaces
- **Pure Virtual Function:** `virtual void draw() = 0;`
- **Abstract Class:** Any class with **at least one pure virtual function**. Cannot be instantiated directly (`Shape s;` is a compile error), though you can hold `Shape*` / `Shape&` to derived objects.
- **Interface in C++:** C++ has no `interface` keyword — an interface is modeled as an **abstract class with only pure virtual functions, no data members, and a `virtual ~Interface() = default;`**.

#### 4. Function Hiding (Name Hiding)
If Derived defines a function with the **same name** as a Base function (non-virtual, or different parameters), it **hides all overloads of that name in Base**!
```cpp
class Base {
public:
    void show(int x) { cout << "Base int\n"; }
};
class Derived : public Base {
public:
    using Base::show; // Unhides Base::show(int) so overloading works!
    void show(double x) { cout << "Derived double\n"; }
};
```

#### 5. Object Slicing
If you assign or pass a `Derived` object **by value** to a `Base` parameter (`void process(Base b)`), the `Derived`-specific members and `vptr` are **sliced off** — only the `Base` part is copied!
- **Fix:** Always pass polymorphic objects by **reference (`const Base&`)** or **pointer (`Base*` / `unique_ptr<Base>`)**.

---

## 14. Rule of Three, Rule of Five, Rule of Zero & RAII

### Resource Management Rules:
- **Rule of Three (Pre-C++11):** If a class manually manages a resource (raw pointer, file handle, socket) and defines any one of:
  1. Destructor (`~T()`)
  2. Copy Constructor (`T(const T&)`)
  3. Copy Assignment Operator (`T& operator=(const T&)`)
  → It almost certainly needs **all three**.
- **Rule of Five (C++11+):** Add move semantics for performance:
  4. Move Constructor (`T(T&&) noexcept`)
  5. Move Assignment Operator (`T& operator=(T&&) noexcept`)
- **Rule of Zero (Modern C++ Best Practice):** Don't manage raw memory manually at all! Use **`std::unique_ptr`**, **`std::shared_ptr`**, **`std::vector`**, **`std::string`**, so the compiler-generated defaults handle everything safely.
- **`= delete` / `= default`:** Explicitly disable copying (`Buffer(const Buffer&) = delete;`) or request compiler default (`= default;`).

### Copy-and-Swap Idiom (Exception-safe `operator=`):
```cpp
class MyVec {
    int* arr;
    size_t n;
public:
    MyVec(size_t n = 0) : arr(n ? new int[n]{} : nullptr), n(n) {}
    ~MyVec() { delete[] arr; }
    MyVec(const MyVec& o) : arr(o.n ? new int[o.n] : nullptr), n(o.n) {
        copy(o.arr, o.arr + n, arr);
    }
    friend void swap(MyVec& a, MyVec& b) noexcept {
        std::swap(a.arr, b.arr);
        std::swap(a.n, b.n);
    }
    // Pass by value makes copy; swap steals it; old data dies with `other`:
    MyVec& operator=(MyVec other) noexcept {
        swap(*this, other);
        return *this;
    }
};
```

### RAII (Resource Acquisition Is Initialization) & Smart Pointers:
- **RAII:** Tie resource lifetime (heap memory, mutex lock, file handle) to an object's scope — acquire in constructor, release in destructor (guaranteed even if exceptions occur!).
- **`std::unique_ptr<T>`:** Exclusive ownership, zero overhead, non-copyable, movable.
- **`std::shared_ptr<T>`:** Shared ownership via atomic reference counting.
- **`std::weak_ptr<T>`:** Non-owning observer of `shared_ptr` — **breaks circular references** (e.g., doubly linked list / parent-child graph) that would otherwise leak memory.

---

## 15. RTTI & C++ Type Casting

- **RTTI (Run-Time Type Information):** Allows checking an object's dynamic type at runtime (`dynamic_cast`, `typeid`) — only works on **polymorphic classes** (classes with at least one `virtual` function).
- **4 C++ Casts:**
  1. **`static_cast<T>(x)`:** Compile-time checked conversions (`int` to `double`, upcasting, or unchecked downcasting if you are 100% sure of type).
  2. **`dynamic_cast<T>(x)`:** Safe runtime-checked downcasting on polymorphic types. Returns **`nullptr`** for failed pointer casts, throws **`std::bad_cast`** for failed reference casts.
  3. **`const_cast<T>(x)`:** Adds/removes `const` or `volatile` (undefined behavior if modifying an originally `const` variable!).
  4. **`reinterpret_cast<T>(x)`:** Low-level bit/pointer reinterpretation (hardware/serialization, unsafe).

```cpp
Base* bp = new Derived();
if (Derived* dp = dynamic_cast<Derived*>(bp)) {
    // Safe downcast succeeded
}
```

---

## 16. SOLID Design Principles (with C++)

1. **S — Single Responsibility Principle (SRP):**
   A class should have **only one reason to change**.
   - *Bad:* `Invoice` class calculates total, prints PDF, AND saves to DB.
   - *Good:* Separate `Invoice`, `InvoicePrinter`, and `InvoiceRepository`.
2. **O — Open/Closed Principle (OCP):**
   Open for **extension**, closed for **modification**.
   - *Example:* Instead of editing an `if/else` chain in `DiscountCalculator` for every new discount type, define a `DiscountStrategy` interface and add new subclasses (`FestivalDiscount`, `VIPDiscount`).
3. **L — Liskov Substitution Principle (LSP):**
   Subtypes must be substitutable for their base types **without breaking expected behavior**.
   - *Classic violation:* `Square` inheriting from `Rectangle` where `setWidth(5); setHeight(4);` breaks `area() == 20` if `Square` forces width = height. Fix: both inherit from `Shape`.
4. **I — Interface Segregation Principle (ISP):**
   Clients shouldn't be forced to depend on methods they don't use — prefer **small, focused interfaces** over one fat interface.
   - *Example:* Split `IMachine` (`print`, `scan`, `fax`) into `IPrinter`, `IScanner`, `IFax` so a basic printer isn't forced to throw "Not supported" on `fax()`.
5. **D — Dependency Inversion Principle (DIP):**
   High-level modules shouldn't depend on low-level modules; **both should depend on abstractions**.
   - *Example:* `OrderService` depends on `IPaymentProcessor*` (injected via constructor — Dependency Injection), NOT directly on hardcoded `StripeAPI`.

```cpp
class INotifier {
public:
    virtual void send(const string& msg) = 0;
    virtual ~INotifier() = default;
};
class EmailNotifier : public INotifier {
public:
    void send(const string& msg) override { cout << "Email: " << msg << "\n"; }
};
class OrderService {
    INotifier& notifier; // Depends on abstraction (DIP + OCP)
public:
    OrderService(INotifier& n) : notifier(n) {}
    void placeOrder() { notifier.send("Order placed!"); }
};
```

---

## 17. Coupling, Cohesion & Special Class Types

- **Cohesion:** How focused and unified a single class's responsibilities are → aim for **High Cohesion**.
- **Coupling:** How strongly one class depends on the internals of another → aim for **Low (Loose) Coupling** (via interfaces & dependency injection).
- **Nested Class:** A class declared inside another class. In C++, a nested class can access `private` members of the enclosing class (since C++11), and helps group helper types cleanly (e.g., `LinkedList::Node`).
- **Local Class:** A class declared inside a function body. Cannot have `static` data members and can only access `static`/enum variables of the enclosing function.

### Overloading vs Overriding vs Hiding Summary:
| Feature | Overloading | Overriding | Hiding |
|---|---|---|---|
| **Scope** | Same class/scope | Base & Derived | Base & Derived |
| **Signature** | Different parameters | Exact same signature + `virtual` | Same name, non-virtual OR different params |
| **Binding** | Compile-time (Static) | Runtime (Dynamic) | Compile-time (Static) |

---

## 18. Tricky C++ OOP Interview Questions (Rapid-Fire)

1. **Can a constructor be `virtual`? Why not?**
   - **No.** Because `vptr` is initialized *inside* the constructor, and to create an object you must already know its exact concrete type at compile time. *(Workaround: Virtual Constructor idiom via `virtual Base* clone() const = 0;` Factory method.)*

2. **Can a constructor be `private`?**
   - **Yes!** Used in **Singleton**, **Factory Method**, and **Builder** patterns to control object creation:
     ```cpp
     class Singleton {
         Singleton() = default;
     public:
         Singleton(const Singleton&) = delete;
         Singleton& operator=(const Singleton&) = delete;
         static Singleton& instance() {
             static Singleton s; // Thread-safe in C++11+ (Meyers' Singleton)
             return s;
         }
     };
     ```

3. **What happens if you call a `virtual` function inside a Constructor or Destructor?**
   - It calls the **current class's own version** (NOT the derived override)! Why? During Base's constructor, the Derived part hasn't been constructed yet; during Base's destructor, the Derived part is already destroyed. So `vptr` points to Base's `vtable`.

4. **Default arguments with virtual functions — what happens?**
   - **Virtual functions are dynamically bound, while default arguments are statically bound!**
     ```cpp
     class B { public: virtual void f(int x = 10) { cout << x; } };
     class D : public B { public: void f(int x = 20) override { cout << x; } };
     B* p = new D();
     p->f(); // Calls D::f(), BUT prints 10 (from B's static type)!
     ```

5. **Can a `virtual` function be `private`?**
   - **Yes!** Derived classes can override a `private` virtual function of Base, and Base can call it internally (**NVI — Non-Virtual Interface pattern** / Template Method pattern).

6. **Can `main()` be inside a class or can an object be created before `main()`?**
   - `main()` must be a global function, **but constructors of global/`static` objects run BEFORE `main()`** (and their destructors run AFTER `main()` returns).
