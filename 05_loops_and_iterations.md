# Chapter 5: Loops - The Automation Engine

> "Never repeat yourself in code."

## The Goal: Batch Processing

You need to analyze 1,000 log files.
**Bad Way:** Write 1,000 commands.
**Good Way:** Write one loop.

---

## 5.1 The `for` Loop (Foreach)

In DevOps, we iterate over Lists (servers, files, logs).

```python
servers = ["web-01", "web-02", "db-01"]

for server in servers:
    print(f"Deploying to {server}")
    restart_service(server)
```

**What happens:**
1.  Python takes `servers`.
2.  Assigns "web-01" to `server`.
3.  Runs the block.
4.  Repeats for "web-02", "db-01".

---

## 5.2 The `range()` Function (Counters)

Need to retry a connection 3 times?
Use `range()`.

```python
for retries in range(3):
    print(f"Connecting... Attempt {retries + 1}")
    if connect():
        break
```

**Theory Note:** `range(3)` creates [0, 1, 2].

---

## 5.3 Under the Hood: Iterators

How does Python know which item is "next"?
-   **Iterable:** The container (List of Servers).
-   **Iterator:** The cursor pointing to current server.

The `for` loop asks the cursor "Next?" until it hits `StopIteration`.

---

## 5.4 The `while` Loop (Wait Until)

Typically used for polling or waiting for a service.

```python
attempts = 0
max_attempts = 5

while attempts < max_attempts:
    if check_health() == "OK":
        print("Service UP!")
        break
    attempts += 1
    print("Waiting...")
```

**Warning:** Infinite Loops (`while True`) are common in daemons, but dangerous in one-off scripts. Always have an exit condition (`break`).

---

## 5.5 Controlling the Flow

### `break` (Emergency Stop)
If the database connection fails, **stop** the entire batch.

```python
for log_file in logs:
    if "FATAL ERROR" in log_file:
        print("Stopping deployment!")
        break
```

### `continue` (Skip)
Ignore harmless warnings.

```python
for user in users:
    if user == "root":
        continue  # Don't delete root!
    delete_user(user)
```

---

## 5.6 Nested Loops (Matrix Configs)

Deploying across multiple regions and environments.

```python
regions = ["us-east", "eu-west"]
envs = ["dev", "prod"]

for region in regions:
    for env in envs:
        deploy(region, env)
```

**Performance:** Beware deeply nested loops (O(n^2)).

---

## 5.7 The Laws of Loops

1.  **Guard Your Exits:** Every `while` needs a state change.
2.  **Don't Touch the List:** Never `.remove()` while iterating.
3.  **Off-by-One Law:** `range(5)` stops at 4.
4.  **Break Early:** Don't waste CPU cycles.

---

## [!] Common Mistakes

### Error #1: Infinite Wait
```python
while service_down:
    print("Waiting...")
    # Forgot to check status again!
```
**Fix:** Update `service_down` inside the loop.

### Error #2: Modifying List
```python
servers = ["a", "b", "c"]
for s in servers:
    servers.remove(s)
# Result: Only "a" and "c" removed. "b" skipped.
```
**Fix:** `for s in servers[:]:` (Iterate over copy).

---

## Practice Problems

### Problem 1: Log Filter
Filter a list of log lines. Print only lines containing "ERROR".

### Problem 2: Retry with Backoff
Write a loop that retries a connection 5 times, waiting longer each time (1s, 2s, 4s...).

---

## Chapter Summary

-   **Automation:** Loops replace manual repetition.
-   **`for`:** For collections (servers, logs).
-   **`while`:** For conditions (waiting for UP).
-   **Control:** `break` / `continue` manage flow.
