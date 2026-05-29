# Low Level Design — SOLID Principles & Design Patterns

---

## SOLID Principles

SOLID is a set of five design principles introduced by Robert C. Martin (Uncle Bob).
Following these principles makes your code easier to understand, extend, and maintain
without breaking existing functionality.

---

### S — Single Responsibility Principle (SRP)

**A class should have only one reason to change.**

Every class should be responsible for exactly one part of the system's functionality.
If a class handles multiple concerns (e.g., both saving data AND sending emails),
any change in one concern can accidentally break the other. Splitting them keeps each
class focused and independently changeable.

```java
// Bad: one class handles two unrelated responsibilities
class UserManager {
    public void saveToDatabase(User user) {
        // DB logic
    }
    public void sendWelcomeEmail(User user) {
        // Email logic
    }
}

// Good: each class has a single responsibility
class UserRepository {
    public void saveToDatabase(User user) {
        // DB logic only
    }
}

class EmailService {
    public void sendWelcomeEmail(User user) {
        // Email logic only
    }
}
```

---

### O — Open/Closed Principle (OCP)

**A class should be open for extension but closed for modification.**

Once a class is written and tested, you should not need to edit it to add new behavior.
Instead, extend it via inheritance or interfaces. This prevents regression bugs —
you're not touching working code, just adding to it.

```java
// Bad: every new shape requires modifying this method
class AreaCalculator {
    public double calculate(String shape, double value) {
        if (shape.equals("circle")) return Math.PI * value * value;
        if (shape.equals("square")) return value * value;
        return 0; // you keep adding ifs forever
    }
}

// Good: each shape knows how to calculate its own area
interface Shape {
    double area();
}

class Circle implements Shape {
    double radius;
    Circle(double radius) { this.radius = radius; }

    public double area() { return Math.PI * radius * radius; }
}

class Square implements Shape {
    double side;
    Square(double side) { this.side = side; }

    public double area() { return side * side; }
}

// Adding a Triangle later? Just create a new class — no existing code touched.
```

---

### L — Liskov Substitution Principle (LSP)

**A subclass should be fully substitutable for its parent class without breaking the program.**

If you have a method that accepts a `Bird`, and you pass a `Penguin` (which is a `Bird`),
the program should still work correctly. If the subclass throws errors or behaves
unexpectedly for inherited methods, it violates LSP. The fix is usually redesigning
the inheritance hierarchy.

```java
// Bad: Penguin IS-A Bird but cannot fly — substitution breaks the program
class Bird {
    public void fly() {
        System.out.println("Flying...");
    }
}

class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!"); // breaks LSP
    }
}

// Good: separate flying birds from non-flying birds
interface Bird {
    void eat();
}

interface FlyingBird extends Bird {
    void fly();
}

class Sparrow implements FlyingBird {
    public void eat() { System.out.println("Sparrow eating"); }
    public void fly() { System.out.println("Sparrow flying"); }
}

class Penguin implements Bird {
    public void eat() { System.out.println("Penguin eating"); }
    // no fly() — and that's perfectly fine
}
```

---

### I — Interface Segregation Principle (ISP)

**A class should not be forced to implement methods it doesn't need.**

When an interface is too large (fat interface), classes that implement it are forced
to provide empty or irrelevant implementations of methods they don't use. This creates
confusion and dead code. Split large interfaces into smaller, focused ones so each
class only implements what is relevant to it.

```java
// Bad: SimplePrinter is forced to implement fax() and scan() which it doesn't support
interface Machine {
    void print();
    void fax();
    void scan();
}

class SimplePrinter implements Machine {
    public void print() { System.out.println("Printing..."); }
    public void fax()   { /* not supported — forced empty impl */ }
    public void scan()  { /* not supported — forced empty impl */ }
}

// Good: split into focused interfaces
interface Printable {
    void print();
}

interface Faxable {
    void fax();
}

interface Scannable {
    void scan();
}

// SimplePrinter only implements what it actually does
class SimplePrinter implements Printable {
    public void print() { System.out.println("Printing..."); }
}

// AdvancedPrinter can implement all three
class AdvancedPrinter implements Printable, Faxable, Scannable {
    public void print() { System.out.println("Printing..."); }
    public void fax()   { System.out.println("Faxing..."); }
    public void scan()  { System.out.println("Scanning..."); }
}
```

---

### D — Dependency Inversion Principle (DIP)

**High-level modules should not depend on low-level modules. Both should depend on abstractions.**

If `OrderService` directly creates a `MySQLDatabase` object inside it, then swapping
MySQL for PostgreSQL or a mock in tests requires editing `OrderService` — that's wrong.
Instead, define an interface (`Database`) and inject the concrete implementation from
outside. This makes the code loosely coupled and easy to test or swap.

```java
// Bad: OrderService is tightly coupled to MySQLDatabase
class MySQLDatabase {
    public void save(String data) { System.out.println("Saving to MySQL: " + data); }
}

class OrderService {
    private MySQLDatabase db = new MySQLDatabase(); // hard dependency

    public void placeOrder(String order) {
        db.save(order);
    }
}

// Good: depend on an abstraction, inject the implementation
interface Database {
    void save(String data);
}

class MySQLDatabase implements Database {
    public void save(String data) { System.out.println("Saving to MySQL: " + data); }
}

class PostgreSQLDatabase implements Database {
    public void save(String data) { System.out.println("Saving to PostgreSQL: " + data); }
}

class OrderService {
    private Database db;

    OrderService(Database db) { this.db = db; } // dependency injected from outside

    public void placeOrder(String order) {
        db.save(order);
    }
}

// Usage: swap DB without touching OrderService
OrderService service     = new OrderService(new MySQLDatabase());
OrderService testService = new OrderService(new MockDatabase()); // easy to test
```

---

## UML (Unified Modeling Language)

UML is a standardized visual language used to design and communicate software structure
before writing code. It helps you think through the design, discuss it with teammates,
and document the system. In LLD, you mainly use two types of diagrams.

| Diagram | What it shows |
|---|---|
| **Class Diagram** | Classes, their attributes, methods, and how they relate to each other |
| **Sequence Diagram** | The order in which objects interact with each other over time (who calls whom) |

**Key relationships in Class Diagrams:**

```
A ──────> B     Association   (A has a reference to B, e.g. Order has a Customer)
A ──◇───> B     Aggregation   (A contains B, but B can exist independently)
A ──◆───> B     Composition   (A owns B; if A is destroyed, B is too)
A ──▷───  B     Inheritance   (A is a type of B — extends)
A ·····▷  B     Implements    (A implements interface B)
```

---

## Singleton Pattern

**Ensure a class has only one instance throughout the application's lifetime.**

Some objects should never be created more than once — a configuration loader, a logger,
or a database connection pool. Creating multiple instances could cause inconsistency
(two loggers writing to different files) or waste resources (multiple DB pools).
The Singleton pattern controls instantiation so only one object ever exists,
and provides a global access point to it.

```java
class AppConfig {
    private static AppConfig instance = null;

    // private constructor prevents direct instantiation from outside
    private AppConfig() {
        System.out.println("Config loaded.");
    }

    public static AppConfig getInstance() {
        if (instance == null) {
            instance = new AppConfig(); // created only on first call
        }
        return instance;
    }

    public String getDbUrl() { return "jdbc:mysql://localhost/mydb"; }
}

// Usage
AppConfig c1 = AppConfig.getInstance();
AppConfig c2 = AppConfig.getInstance();
System.out.println(c1 == c2); // true — same object
```

> **Thread-safe version** (for multi-threaded apps): use `synchronized` or initialize
> the instance eagerly (`private static AppConfig instance = new AppConfig();`).

> **Pitfall:** Singletons carry global state, which makes unit testing harder.
> Use them only when a single shared instance is a genuine requirement.

---

## Factory Pattern

**Delegate the responsibility of object creation to a separate factory class.**

When a caller needs an object, it often shouldn't know (or care) which specific subclass
to instantiate — that decision may depend on runtime input, configuration, or business rules.
The Factory pattern centralizes this "which class to create" logic in one place.
If you add a new type later, you only change the factory, not every caller.

```java
// Product types
interface Notification {
    void send(String message);
}

class EmailNotification implements Notification {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

class SMSNotification implements Notification {
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

class PushNotification implements Notification {
    public void send(String message) {
        System.out.println("Push: " + message);
    }
}

// Factory: centralizes the creation decision
class NotificationFactory {
    public Notification create(String type) {
        switch (type) {
            case "email": return new EmailNotification();
            case "sms":   return new SMSNotification();
            case "push":  return new PushNotification();
            default: throw new IllegalArgumentException("Unknown type: " + type);
        }
    }
}

// Caller doesn't know or care which class is returned
NotificationFactory factory = new NotificationFactory();
Notification n = factory.create("sms");
n.send("Your OTP is 4821"); // SMS: Your OTP is 4821
```

Adding a new `WhatsAppNotification` later? Add the class and one `case` in the factory.
No existing caller changes.

---

## Strategy Pattern

**Define a family of algorithms, encapsulate each one, and make them interchangeable at runtime.**

Sometimes you have multiple ways to perform the same operation — sorting by different
algorithms, calculating discounts by different rules, or paying by different payment methods.
Instead of hardcoding the logic or using if-else chains, extract each behavior into
its own class and inject the desired one at runtime. This follows OCP too — you add new
strategies without modifying the context class.

```java
// Strategy interface
interface SortStrategy {
    void sort(int[] data);
}

// Concrete strategies
class BubbleSort implements SortStrategy {
    public void sort(int[] data) {
        System.out.println("Sorting using Bubble Sort");
        // bubble sort logic
    }
}

class QuickSort implements SortStrategy {
    public void sort(int[] data) {
        System.out.println("Sorting using Quick Sort");
        // quick sort logic
    }
}

class MergeSort implements SortStrategy {
    public void sort(int[] data) {
        System.out.println("Sorting using Merge Sort");
        // merge sort logic
    }
}

// Context class: holds a strategy and delegates to it
class Sorter {
    private SortStrategy strategy;

    Sorter(SortStrategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy; // swap strategy at runtime
    }

    public void sort(int[] data) {
        strategy.sort(data); // delegates — doesn't know the algorithm details
    }
}

// Usage
int[] data = {5, 3, 8, 1};

Sorter sorter = new Sorter(new QuickSort());
sorter.sort(data); // Sorting using Quick Sort

sorter.setStrategy(new MergeSort());
sorter.sort(data); // Sorting using Merge Sort — no change to Sorter class
```

The key insight: **behavior is injected, not hardcoded**. The context class stays the same;
you swap what it does by swapping the strategy.

---

## How They All Relate

```
SOLID      →  the rules/principles your design should follow at all times
UML        →  the visual language to design and communicate your solution
Singleton  →  creational pattern  — controls how many instances exist (answer: one)
Factory    →  creational pattern  — controls who decides which class to instantiate
Strategy   →  behavioral pattern  — controls which algorithm/behavior runs at runtime
```

> **Creational patterns** deal with object creation.
> **Behavioral patterns** deal with how objects communicate and share responsibility.
> **Structural patterns** (coming next) deal with how objects are composed together.
