# Chapter 10: OOP - The Infrastructure Blueprints

> "Code format reflects mental models."

## The Goal: Modeling Complex Systems

In small scripts, functions are enough: `restart_server()`.
In large systems (like writing a Cloud API wrapper), you need **Object-Oriented Programming (OOP)**.

OOP allows you to structure code by modeling real-world objects. You don't just have "data" and "logic" floating around; you have **Objects** that hold their own data and know how to perform their own actions.

---

## 10.1 Classes vs Objects (The Theory)

Think of a **Class** as a Blueprint (The Architect's Drawing).
Think of an **Object** as the Building (The Physical Structure).

-   **Class:** `Server` (Defines that servers have an IP and can restart).
-   **Object:** `web-01` (A specific server at 10.0.0.1).

```python
class Server:
    # 1. The Constructor (__init__)
    # This runs automatically when you create a new object.
    # It sets up the initial state.
    def __init__(self, hostname, cpu_cores):
        self.hostname = hostname    # Attribute (Data)
        self.cpu_cores = cpu_cores  # Attribute (Data)
        self.is_on = False          # Attribute (Default State)

    # 2. Instance Methods (Behavior)
    # Functions inside a class are called Methods.
    # They MUST take 'self' as the first argument.
    def power_on(self):
        self.is_on = True
        print(f"{self.hostname} is booting...")

# Creating Instances (Instantiation)
s1 = Server("web-01", 4)
s2 = Server("db-01", 16)

# Each object has its own separate memory
print(s1.hostname)  # "web-01"
print(s2.hostname)  # "db-01"

s1.power_on()       # Only s1 turns on. s2 stays off.
```

---

## 10.2 Encapsulation (Protecting Data)

In DevOps, you have rules. "You cannot set CPU cores to -5."
Encapsulation allows you to enforce these rules by hiding data behind a protective layer.

### The `@property` Decorator (Getters and Setters)
Instead of letting users touch `self.ram` directly, we force them to go through a "Gatekeeper" method.

```python
class VM:
    def __init__(self):
        self._ram = 0  # The "_" means "Internal/Protected" (Convention only)

    # 1. The Getter to read the value
    @property
    def ram(self):
        return f"{self._ram} GB"

    # 2. The Setter to write the value
    @ram.setter
    def ram(self, value):
        if value < 0:
            raise ValueError("RAM cannot be negative!")
        if value > 128:
            raise ValueError("Too much RAM for this host!")
        self._ram = value

# Usage
my_vm = VM()
my_vm.ram = 64      # Goes through Setter -> OK
print(my_vm.ram)    # Goes through Getter -> "64 GB"

# my_vm.ram = -10   # CRASH: ValueError (The logic protected the variable!)
```

---

## 10.3 Inheritance (The Hierarchy)

Don't Repeat Yourself (DRY).
If you have `WebServer` and `DatabaseServer`, they share 90% of the same code (IP, Hostname, Restart).
Put the shared code in a **Parent Class** (`Server`).

### 1. Single Inheritance
One Parent, One Child.
The Child gets everything the Parent has, plus its own extras.

```python
class Server:
    def ping(self): print("Pong from Server")

class WebServer(Server):
    def serve_html(self): print("Serving <html>...")

w = WebServer()
w.ping()        # Works! Inherited from Server.
w.serve_html()  # Works! Specific to WebServer.
```

### 2. Multilevel Inheritance
Grandparent -> Parent -> Child.

```python
class Machine:
    def has_electricity(self): return True

class Server(Machine):
    def boot(self): print("Booting OS...")

class Database(Server):
    def query(self): print("SELECT * FROM data")

db = Database()
db.has_electricity() # From Machine
db.boot()            # From Server
db.query()           # From Database
```

---

## 10.4 Abstraction (The Interface)

Abstraction means enforcing a structure.
If you are building a Cloud Tool, every provider (AWS, Azure) MUST have a `connect()` method. You can't let a developer forget to write it.

### The Abstract Base Class (ABC)

We use the `abc` module to create a template. You cannot create an object from a template; you must inherit from it.

```python
from abc import ABC, abstractmethod

# 1. The Abstract Contract
class CloudProvider(ABC):
    @abstractmethod
    def connect(self):
        pass

    @abstractmethod
    def get_status(self):
        pass

# 2. The Implementation (AWS)
class AWS(CloudProvider):
    def connect(self):
        print("Connecting to AWS via boto3...")

    def get_status(self):
        return "AWS is Online"

# 3. The Broken Implementation (Azure)
class Azure(CloudProvider):
    def connect(self):
        print("Connecting to Azure...")
    # Oops! We forgot 'get_status'

# Usage
aws = AWS()        # Works!
# azure = Azure()  # CRASH! TypeError: Can't instantiate abstract class Azure with abstract method get_status
```

**Why this matters:** Abstraction guarantees that all your plugins/providers follow the exact same rules. 

---

## 10.4 Multiple Inheritance & The Diamond Problem

This is where it gets complex. Python allows a class to have **Two Parents**.
This is powerful (Mixins) but dangerous.

### The Mixin Pattern (The Good Use Case)
A **Mixin** is a small class that adds one specific feature (like Logging) to any class.

```python
class LoggableMixin:
    def log(self, msg):
        print(f"[LOG] {msg}")

class Server:
    def start(self): print("Starting...")

# Inherits from BOTH
class SecureServer(Server, LoggableMixin):
    pass

s = SecureServer()
s.start()        # From Server
s.log("Secure")  # From LoggableMixin
```

### The Diamond Problem (The Danger)
What if **both** parents define the same method? Which one wins?

Structure:
```text
      A (Root)
     / \
    B   C
     \ /
      D (Child)
```

```python
class A:
    def save(self): print("Saved to A (Root)")

class B(A):
    def save(self): print("Saved to B (File)")

class C(A):
    def save(self): print("Saved to C (Database)")

class D(B, C):
    pass
    # D inherits from B and C. Both have 'save()'.
    # Who wins?
```

### The Solution: Method Resolution Order (MRO)
Python uses the **C3 Linearization Algorithm** to decide.
**The Rule:** Children before Parents. Left Parents before Right Parents.

For `class D(B, C)`:
1.  Is it in D? No.
2.  Is it in B (Left)? Yes! **(Stop here).**
3.  (Ignores C).

```python
d = D()
d.save()
# Output: "Saved to B (File)"

# Viewing the Decision Tree
print(D.mro())
# [<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>]
```

**Why this matters:** If you connect a class to two Libraries that both have a method called `connect()`, Python will silently pick the left one. You must know your MRO!

---

## 10.5 Polymorphism (Many Forms)

Polymorphism allows you to treat different objects as if they were the same.
You don't care *what* the object is, only that it can do the job.

### 1. Method Overriding
We have a generic `Server.restart()`.
The `Database` needs a special restart (flush to disk first).
The `Database` **overrides** the parent method.

```python
class Server:
    def restart(self):
        print("Rebooting OS...")

class Database(Server):
    def restart(self):
        print("Flushing SQL transactions...")
        print("Rebooting OS...")

# Polymorphic Loop
infrastructure = [Server(), Database(), Server()]

for unit in infrastructure:
    # "unit" changes type in every loop iteration!
    # Python figures out the correct method at Runtime.
    unit.restart()
```

### 2. Duck Typing
Python is dynamic. It doesn't check if you inherit from `Server`.
It checks: **"Do you have a restart() method?"**

```python
class Car:
    def restart(self): print("Turning ignition key...")


### 3. Compile Time vs Run Time Polymorphism (The Interview Question)

In languages like C++ or Java, there are two types:
1.  **Compile Time (Static Binding):** Method **Overloading**. Same function name, different arguments. (e.g., `add(int, int)` vs `add(str, str)`).
2.  **Run Time (Dynamic Binding):** Method **Overriding**. Parent has `restart()`, Child has new `restart()`.

**Python's Reality:**
Python is **Dynamically Typed**, so it does **NOT** support Compile Time Polymorphism (Overloading) in the traditional sense.
If you define two methods with the same name, the second one simply overwrites the first.

**How to "Simulate" Overloading in Python:**
Use default arguments or variable-length arguments (`*args`).

```python
class Calculator:
    # "Overloading" using default None
    def add(self, a, b, c=None):
        if c:
            return a + b + c
        return a + b

calc = Calculator()
print(calc.add(1, 2))       # 3
print(calc.add(1, 2, 3))    # 6
```

**Key Takeaway:** Python relies almost exclusively on **Run Time Polymorphism** (Overriding & Duck Typing). The interpreter figures out which method to call *while the code is running*, not beforehand.


---

## 10.6 Magic Methods (Connecting to Python)

How does `len(my_object)` work? Or `my_object + other_object`?
You define **Magic Methods** (Double Underscore -> Dunder).

| Method | Triggered By | Example |
| :--- | :--- | :--- |
| `__init__` | Creation | `x = Server()` |
| `__str__` | `print()` | `print(x)` |
| `__len__` | `len()` | `len(x)` |
| `__add__` | `+` Operator | `x + y` |
| `__eq__` | `==` Operator | `x == y` |

```python
class LoadBalancer:
    def __init__(self):
        self.servers = []

    def __len__(self):
        # Telling Python how to count this object
        return len(self.servers)

    def __add__(self, other_lb):
        # Telling Python how to add two LoadBalancers
        new_lb = LoadBalancer()
        new_lb.servers = self.servers + other_lb.servers
        return new_lb
```

---

## 10.7 The Laws of Advanced OOP

1.  **Liskov Substitution Principle:**
    -   **Rule:** If you inherit from a Parent, you must be able to replace the Parent without breaking anything.
    -   **Example:** If `Server.restart()` takes 0 arguments, `Database.restart()` cannot require 5 arguments.

2.  **Favor Composition over Inheritance:**
    -   **Rule:** Multiple inheritance is messy. Instead of "Is A" (Inheritance), use "Has A" (Composition).
    -   **Good:** `Server` **has a** `Logger`.
    -   **Bad:** `Server` **is a** `Logger`.

3.  **Know your MRO:**
    -   **Rule:** When in doubt with Multiple Inheritance, print `ClassName.mro()` to see exactly what Python will call.

---

## Practice Problem: The Cloud API

Create a base class `CloudResource` with `cost()`.
Create subclasses `VM` (cost = $10) and `Storage` (cost = $0.10/GB).
Create a `Project` class that holds a list of resources.
Add a magic method `__int__` to `Project` that returns the total cost of all resources.

---

## Chapter Summary

-   **Classes/Objects:** Blueprints vs Instances.
-   **Encapsulation:** `@property` protects your data.
-   **Polymorphism:** `restart()` behaves differently for Web vs DB.
-   **Multiple Inheritance:** Understand MRO (Left to Right) to solve collisions.
-   **Magic Methods:** `__str__`, `__len__` make your objects feel native.
