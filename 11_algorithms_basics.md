# Chapter 11: Algorithms - Efficiency at Scale

> "Premature optimization is the root of all evil." — Donald Knuth

## The Goal: Handling Scale

If you have 10 logs, any script is fast.
If you have 10,000,000 logs (common in Prod), a bad script will hang for hours.

**Algorithms** are about solving problems efficiently.
In DevOps, "Efficient" means:
1.  **Time:** How long does it take? (CPU)
2.  **Space:** How much RAM does it eat? (Memory)

---

## 11.1 Big O Notation (The Speedometer)

We don't measure speed in seconds (because hardware varies). We measure in **Operations** relative to Input Size (`n`).

### The Big O Tier List

| Notation | Name | Speed | Example |
| :--- | :--- | :--- | :--- |
| **O(1)** | Constant | ⚡ Instant | Dictionary Lookup (`config["ip"]`) |
| **O(log n)** | Logarithmic | 🏎️ Very Fast | Binary Search (Sorted Logs) |
| **O(n)** | Linear | 🏃 Fast-ish | Looping through a List |
| **O(n log n)** | Log Linear | 🚶 Decent | Sorting a List |
| **O(n^2)** | Quadratic | 🐢 Slow | Nested Loops (Compare All vs All) |
| **O(2^n)** | Exponential | 🛑 Impossible | Cracking Passwords |

### 1. O(1) - Constant Time
No matter how much data, it takes 1 step.

```python
# Accessing an element by index or key
ip = servers[50]
config = settings["db_host"]
```
It takes the same time whether `servers` has 100 or 1,000,000 items.

### 2. O(n) - Linear Time
If you have N items, it takes N steps.

```python
# Finding a value in an unsorted list
for server in servers_list:
    if server == "prod-db-01":
        print("Found!")
```
**Scaling:**
-   10 servers -> 10 ms
-   1 million servers -> 1,000 seconds (16 mins!)

### 3. O(n^2) - Quadratic Time (The Danger Zone)
Usually caused by **Nested Loops**.

```python
# Finding duplicates (Bad Approach)
for line1 in log_lines:
    for line2 in log_lines:
        if line1 == line2:
            print("Duplicate!")
```
**Scaling:**
-   1,000 lines -> 1,000,000 steps.
-   1,000,000 lines -> **1 Trillion steps** (Script hangs forever).

**DevOps Rule:** Never use O(n^2) on Production logs.

---

## 11.2 Space Complexity (RAM is Expensive)

Time isn't the only cost. If you load a 50GB log file into RAM on a 4GB VM, your script dies (`MemoryError` or OOM Kill).

### O(1) Space (Streaming)
You hold only one line in memory at a time.

```python
with open("massive.log") as f:
    for line in f:  # Lazy Loading
        process(line)
```

### O(n) Space (Loading All)
You load everything into a list.

```python
with open("massive.log") as f:
    lines = f.readlines()  # BOOM! 50GB RAM needed.
```

---

## 11.3 Search Algorithms

### Linear Search (Brute Force)
Start at the beginning, check every item.
-   **Pros:** Works on unsorted data.
-   **Cons:** Slow (O(n)).

```python
def find_error(logs, target="ERROR"):
    for i, line in enumerate(logs):
        if target in line:
            return i
    return -1
```

### Binary Search (Divide & Conquer)
Only works on **Sorted Data** (e.g., Logs sorted by Timestamp).
-   **Pros:** Blazing FAST (O(log n)).
-   **Cons:** Data MUST be sorted first.

**How it works:**
1.  Pick the middle item.
2.  Is target lower? Discard the right half.
3.  Is target higher? Discard the left half.
4.  Repeat.

**Power of O(log n):**
Searching **4 Billion** items takes only **32 steps**.

---

## 11.4 DevOps Case Study: The IP Whitelist

**Scenario:** You have 10,000 Whitelisted IPs. You process 1,000,000 incoming requests.

### Bad Solution (List - O(n))
```python
whitelist = ["1.1.1.1", "2.2.2.2", ...] # List

for ip in incoming_requests:
    if ip in whitelist:  # O(n) scan inside the loop!
        allow(ip)
```
Total Complexity: O(n * m).
1M requests * 10k whitelist = **10 Billion Steps**.

### Good Solution (Set - O(1))
```python
whitelist_set = set(["1.1.1.1", "2.2.2.2", ...]) # Hash Map

for ip in incoming_requests:
    if ip in whitelist_set:  # O(1) Instant Lookup
        allow(ip)
```
Total Complexity: O(m).
1M requests * 1 step = **1 Million Steps**.
**Result:** The script runs 10,000x faster.

---

## 11.5 The Laws of Optimization

1.  **The Law of Measurement:** Don't guess which part is slow. Use `time` or `cProfile` to measure.
2.  **The Law of Datatypes:** Dictionaries/Sets are O(1). Lists are O(n). Know the difference.
3.  **The Law of Memory:** Streaming (O(1) Space) is always safer than Loading All (O(n) Space).

---

## [!] Common Mistakes

### Error #1: Premature Optimization
Spending 3 days optimizing a script that runs once a month and takes 5 seconds.
**Fix:** Only optimize scripts that run frequently or handle massive data.

### Error #2: The Hidden Loop
```python
if request_ip in list_of_ips:
```
This looks like one line, but it is a **hidden loop** (O(n)). If `list_of_ips` is huge, this line is slow.

---

## Practice Problems

### Problem 1: The RAM Saver
Refactor this code to use O(1) Space:
```python
# Bad
logs = open("server.log").read().splitlines()
errors = [line for line in logs if "ERROR" in line]
print(len(errors))
```

### Problem 2: The Binary Search Implementation
Write a function `binary_search(sorted_list, target)` that returns the index.
Handle the edge case where the target is not found.

---

## Chapter Summary

-   **Big O:** The standard language for describing performance.
-   **O(1) vs O(n):** The difference between "Instant" and "Wait for it".
-   **Space Complexity:** Managing RAM is just as important as CPU.
-   **Sets/Dicts:** The secret weapon for O(1) lookups.
