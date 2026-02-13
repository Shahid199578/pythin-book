# Chapter 2: Variables - Infrastructure Resources

> "If you can't measure it, you can't manage it." — Peter Drucker

## The Goal: Monitoring Metrics

Imagine you are monitoring a web server. You have constant streams of data:
-   CPU Usage: 45%
-   Available RAM: 16 GB
-   Hostname: "prod-web-01"
-   Service Status: Active

You need to store these values so your script can check them (e.g., "If CPU > 90%, send alert").

In Python, **Variables** are the labels for these system resources.

---

## 2.1 What is a Variable? Labels, Not Boxes

In DevOps, we often use the container metaphor. However, in Python, variables function closer to **labels** than containers.

```python
hostname = "web-01"
```

**The Process:**
1.  Python instantiates a string object `"web-01"` in memory.
2.  Python creates the label `hostname`.
3.  Python binds the label to the object.

If you reassign it:
```python
hostname = "db-01"
```
Python simply moves the `hostname` label to the new object `"db-01"`. The original `"web-01"` object remains in memory, unreferenced.

---

## 2.2 Under the Hood: Memory & Garbage Collection

When you move a label, what happens to the old data?

**The Garbage Collector (GC):**
Python's internal process that acts like a janitor.
1.  It scans memory for objects that have **zero** labels attached.
2.  If an object (like the old `"web-01"`) has no labels, the GC deletes it to free up RAM.

**DevOps Note:** This is why Python is "memory safe." You don't have to manually `free()` memory like in C++, preventing memory leaks in your long-running automation scripts.

---

## 2.3 The 4 Basic Data Types (Resource Types)

Different metrics require different data types.

### 1. Integers (`int`) - Whole Counts
Used for discrete resources.
```python
request_count = 5000
cpu_cores = 8
failed_logins = 0
```
**Feature:** Python ints can grow as large as memory allows (no 32-bit overflow limit).

### 2. Floats (`float`) - Percentages/Averages
Used for continuous metrics.
```python
cpu_load = 45.5
memory_usage = 12.4
uptime_days = 99.9
```
**Precision Warning:** `0.1 + 0.2` might equal `0.30000000000000004` due to binary floating-point math. For precise financial calculations, use `Decimal`.

### 3. Strings (`str`) - Configuration / Logs
Text data.
```python
server_name = "nginx-proxy"
error_msg = "Connection Refused"
api_key = "xy78-bb92"  # Even though it has numbers, it's text!
```

### 4. Booleans (`bool`) - Flags
Status checks.
```python
is_active = True
maintenance_mode = False
```

---

## 2.4 Dynamic Typing: Flexibility vs Risk

Python allows a variable to switch types.

```python
config = 10        # int
config = "default" # str (Allowed!)
```

**DevOps Pro/Con:**
-   **Pro:** Great for quick scripts where parsing loose log data is messy.
-   **Con:** Dangerous in large systems. Did you expect `config` to be a number? Now it's text.
-   **Fix:** Use **Type Hinting** (modern Python standard):
    ```python
    port: int = 8080
    host: str = "localhost"
    ```

---

## 2.5 Naming Rules: Configuration Standards

Your variables should look like standard configuration keys (Snake Case).

1.  **Snake Case (`snake_case`):**
    -   All lowercase, underscores between words.
    -   **Good:** `max_connections`, `retry_count`, `admin_email`
    -   **Bad:** `maxConnections` (Java style), `MaxConn` (Go style).

2.  **Descriptive Names:**
    -   **Bad:** `t = 5` (What is t? Time? Timeout? Threads?)
    -   **Good:** `timeout_seconds = 5`

3.  **No Keywords:**
    -   Don't name a variable `list`, `str`, or `min`. These are built-in Python tools.
    -   **Bad:** `str = "Log line"` (You just broke the string converter!)

---

## 2.6 The Type Trap (Casting)

Logs always come in as Strings. You cannot do math on strings.

```python
# Reading from a config file usually returns text
cpu_limit = "80"  
current_usage = 85

# print(current_usage > cpu_limit) 
# ERROR: TypeError: '>' not supported between instances of 'int' and 'str'
```

**The Fix:** Cast (convert) the data first.

```python
cpu_limit = int("80")  # Convert text "80" to integer 80
if current_usage > cpu_limit:
    print("ALARM: High CPU")
```

**Casting Functions:**
-   `int("404")` -> `404` (HTTP Status)
-   `float("10.5")` -> `10.5` (Load Average)
-   `str(200)` -> `"200"` (Stringified Status)

---

## [!] Common Mistakes

### Error #1: The Uninitialized Variable
```python
print(cluster_status)
```
**Error:** `NameError: name 'cluster_status' is not defined`.
**Fix:** Assign a default value first: `cluster_status = "unknown"`.

### Error #2: The Math Trap
```python
version = "3.10"
new_version = version + 1
```
**Error:** `TypeError`. Strings generally cannot be added to numbers.

### Error #3: Variable Shadowing
```python
id = 5
print(id(x))  # Crash!
```
**Analysis:** `id` is a built-in function that gets memory addresses. You overwrote it with the number 5. Never name a variable `id`. Use `user_id` or `server_id`.

---

## Practice Problems

### Problem 1: Resource Calculation
-   Total RAM: 32 GB
-   Used RAM: 14.5 GB
Calculate `free_ram` and print it.

### Problem 2: Config Builder
Create three variables: `host`, `port`, `protocol`.
Add them together to form a URL string: `https://api.server.com:8080`
(Hint: Use `str(port)` if port is an integer).

---

## Chapter Summary

-   **Variables:** Labels for system resources (CPU, Memory, Config).
-   **Dynamic Typing:** Variables can change type, but be careful.
-   **Garbage Collection:** Automatic memory cleanup prevents leaks.
-   **Casting:** Always convert textual inputs (logs/configs) to numbers before math.
-   **Naming:** Use `snake_case` (e.g., `server_status`).
