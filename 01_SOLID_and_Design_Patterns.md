# Low Level Design — SOLID Principles & Design Patterns

## SOLID Principles

Five rules to keep code clean and maintainable.

---

### S — Single Responsibility
A class should do one thing only.

```python
# Bad: one class does everything
class UserManager:
    def save_to_db(self): ...
    def send_email(self): ...

# Good: split the concerns
class UserRepository:
    def save_to_db(self): ...

class EmailService:
    def send_email(self): ...
```

---

### O — Open/Closed
Open for extension, closed for modification.

```python
# Bad: you modify this every time a new shape is added
def area(shape):
    if shape == "circle": ...
    if shape == "square": ...

# Good: add new shapes without touching existing code
class Circle:
    def area(self): return 3.14 * r * r

class Square:
    def area(self): return s * s
```

---

### L — Liskov Substitution
A subclass should be usable wherever the parent is used, without breaking things.

```python
class Bird:
    def fly(self): ...

class Penguin(Bird):
    def fly(self): raise Exception("Can't fly!")  # Violates LSP
```

Penguin shouldn't extend Bird if Bird promises `fly()`.

---

### I — Interface Segregation
Don't force a class to implement methods it doesn't need.

```python
# Bad: Printer is forced to implement fax()
class Machine:
    def print(self): ...
    def fax(self): ...      # Printer doesn't need this

# Good: split into focused interfaces
class Printable:
    def print(self): ...

class Faxable:
    def fax(self): ...
```

---

### D — Dependency Inversion
Depend on abstractions, not concrete classes.

```python
# Bad: tightly coupled to MySQL
class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()  # hard dependency

# Good: inject any database
class OrderService:
    def __init__(self, db: Database):  # depends on abstraction
        self.db = db
```

---

## UML (Unified Modeling Language)

A visual language to draw your design before coding it.

| Diagram | What it shows |
|---|---|
| **Class Diagram** | Classes, attributes, methods, relationships |
| **Sequence Diagram** | How objects talk to each other over time |

**Key relationships in Class Diagrams:**

```
A ──────> B     Association   (A uses B)
A ──◇───> B     Aggregation   (B can exist without A)
A ──◆───> B     Composition   (B dies if A dies)
A ──▷───  B     Inheritance   (A is-a B)
A ·····▷  B     Implements    (A implements interface B)
```

---

## Singleton Pattern

**One instance only, ever.**

Use when: config manager, logger, DB connection pool — things that should exist exactly once.

```python
class Config:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

c1 = Config()
c2 = Config()
print(c1 is c2)  # True — same object
```

> **Pitfall:** Singletons make testing harder (global state). Use sparingly.

---

## Factory Pattern

**Hide the "which class to create" decision.**

Use when: the caller shouldn't know (or care) which subclass they're getting.

```python
class Dog:
    def speak(self): return "Woof"

class Cat:
    def speak(self): return "Meow"

class AnimalFactory:
    def create(self, animal_type):
        if animal_type == "dog": return Dog()
        if animal_type == "cat": return Cat()

# Caller just asks for an animal — doesn't know Dog/Cat exists
factory = AnimalFactory()
animal = factory.create("dog")
animal.speak()  # Woof
```

The caller is decoupled from the concrete classes.

---

## Strategy Pattern

**Swap algorithms/behaviors at runtime.**

Use when: you have multiple ways to do the same thing and want to switch between them.

```python
class BubbleSort:
    def sort(self, data): ...

class QuickSort:
    def sort(self, data): ...

class Sorter:
    def __init__(self, strategy):
        self.strategy = strategy  # inject any sorting strategy

    def sort(self, data):
        return self.strategy.sort(data)

# Switch strategy without changing Sorter
s = Sorter(QuickSort())
s.sort([3, 1, 2])

s.strategy = BubbleSort()  # swap at runtime
s.sort([3, 1, 2])
```

The key insight: behavior is passed in, not baked in.

---

## How They Relate

```
SOLID      →  the rules your design should follow
UML        →  how you draw/communicate that design
Singleton  →  pattern for "one instance" problems
Factory    →  pattern for "object creation" problems
Strategy   →  pattern for "swappable behavior" problems
```
