# Chapter 12: Sorting and Recursion - Organization

> "Chaos is merely order waiting to be deciphered."

## The Goal: Ordered Data

Logs come in random order (due to network latency).
To analyze an incident, you need to **Sort** them by timestamp.
To find a file in a deep folder structure, you need **Recursion**.

---

## 12.1 Sorting in Python (`sort` vs `sorted`)

Python uses **Timsort**, a hybrid algorithm (O(n log n)). It's very fast.

### 1. `sorted()` function (Safe)
Returns a **new** sorted list. The original list is untouched.

```python
ips = ["10.0.0.5", "10.0.0.1", "10.0.0.3"]
sorted_ips = sorted(ips)

print(sorted_ips) # ['10.0.0.1', '10.0.0.3', '10.0.0.5']
print(ips)        # ['10.0.0.5', '10.0.0.1', '10.0.0.3'] (Unchanged)
```

### 2. `.sort()` method (In-Place)
Modifies the **existing** list. Saves memory (no copy), but destroys original order.

```python
ips.sort()
print(ips) # ['10.0.0.1', '10.0.0.3', '10.0.0.5']
```

---

## 12.2 Custom Sorting (Key Functions)

Default sorting works on strings/numbers.
What if you want to sort items by specific criteria?

**Scenario:** Sort logs by severity (INFO < WARNING < ERROR).

```python
logs = [
    {"msg": "Disk Full", "level": "ERROR"},
    {"msg": "Login OK", "level": "INFO"},
    {"msg": "High Latency", "level": "WARNING"}
]

# Helper function to define the order
def get_severity(item):
    priority = {"INFO": 0, "WARNING": 1, "ERROR": 2}
    return priority[item["level"]]

# Sort using the 'key'
sorted_logs = sorted(logs, key=get_severity, reverse=True)
# Result: ERROR first, then WARNING, then INFO
```

**Lambda (Anonymous Function):**
For simple keys, use a `lambda` (one-line function).

```python
# Sort servers by CPU usage
servers = [("web01", 80), ("db01", 10), ("cache01", 50)]
servers.sort(key=lambda s: s[1]) # Sort by the second item (CPU)
```

---

## 12.3 Recursion (Functions Calling Themselves)

Recursion is when a function calls itself to solve a smaller piece of the problem.
**DevOps Use Case:** Walking through a directory tree (`/var/www/html/...`).

### The Structure of Recursion
1.  **Base Case:** The condition to STOP (e.g., "No more subfolders").
2.  **Recursive Step:** The function calling itself.

```python
import os

def walk_dir(path):
    print(f"Scanning: {path}")
    
    # Get all items in directory
    items = os.listdir(path)
    
    for item in items:
        full_path = os.path.join(path, item)
        
        if os.path.isdir(full_path):
            # RECURSION: Function calls itself for the subfolder
            walk_dir(full_path)
        else:
            print(f"Found File: {item}")

# walk_dir("/var/log")
```

**Visualizing the Stack:**
1. `walk_dir("/log")`
    2. `walk_dir("/log/nginx")`
        3. `walk_dir("/log/nginx/old")` ... returns
    2. ... returns
1. Done.

---

## 12.4 Recursive vs Iterative

Should you always use recursion? **No.**
Recursion uses the **Call Stack** (O(n) Space). If the directory is 10,000 folders deep, Python will crash (`RecursionError`).

**Iterative (Stack-Saving) Approach:**
Use a list as a "ToDo" stack.

```python
def walk_iterative(start_path):
    todo = [start_path]
    
    while todo:
        current = todo.pop()
        # process current...
        # add subfolders to todo list
```

---

## 12.5 The Laws of Sorting

1.  **The Law of Stability:** Python's sort is **Stable**. If two items have the same key, their original order is preserved.
2.  **The Law of Keys:** Always use `key=` for complex objects (dicts/tuples).
3.  **The Law of Depth:** Be careful with recursion limit (default 1000). Use `sys.setrecursionlimit()` if you dare.

---

## [!] Common Mistakes

### Error #1: Sorting None
```python
x = ips.sort()
print(x) # None
```
**Why:** `.sort()` modifies in-place and returns `None`. **Do not assign the result.**

### Error #2: Infinite Recursion
```python
def loop():
    print("Run")
    loop() # No Base Case!
```
**Result:** `RecursionError: maximum recursion depth exceeded`.

---

## Practice Problems

### Problem 1: The Late Logs
Sort a list of log strings based on the timestamp at the beginning of the line.
`"2023-10-01 Error..."`
(Hint: `key=lambda x: x.split()[0]`)

### Problem 2: The Directory Size
Write a recursive function `get_size(path)` that calculates the total size (bytes) of a directory and all its subfolders.

---

## Chapter Summary

-   **Sorting:** `sorted()` (Copy) vs `.sort()` (In-Place).
-   **Keys:** `lambda` functions define custom sort orders.
-   **Recursion:** Elegant for hierarchical data (File systems).
-   **Base Case:** The most important part of a recursive function.
