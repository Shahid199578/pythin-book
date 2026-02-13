# Chapter 4: Logic - The Decision Engine

> "A smart script is one that knows when to stop."

## The Goal: Automated Decisions

Your deployment script is running.
-   **IF** the tests pass -> Deploy to Prod.
-   **ELSE** -> Rollback and Slack the team.

This is **Flow Control**. It turns a dumb list of commands into an intelligent agent.

---

## 4.1 Boolean Logic (Status Checks)

In DevOps, everything is a status check. Pass (`True`) or Fail (`False`).

```python
build_success = True
server_online = False
```

### Comparison Operators

| Operator | Meaning | DevOps Example |
| :--- | :--- | :--- |
| `==` | Equal | `status_code == 200` |
| `!=` | Not Equal | `error_count != 0` |
| `>` | Greater | `latency_ms > 500` |
| `<` | Less | `disk_usage < 90` |
| `>=` | Greater or Equal | `uptime_days >= 30` |
| `<=` | Less or Equal | `packet_loss <= 1` |

---

## 4.2 The `if` Statement (The Gatekeeper)

Execute code only if a condition is met.

```python
http_status = 500

if http_status == 500:
    print("CRITICAL: Server Error")
    restart_service()
```

**Indentation is Law:**
Python needs those 4 spaces to know what code belongs to the "Critical" block.

---

## 4.3 The `else` Clause (The Backup Plan)

```python
ping_result = "OK"

if ping_result == "OK":
    print("Host is up")
else:
    print("Host is down! Paging on-call...")
```

---

## 4.4 The `elif` Ladder (Multiple Statuses)

```python
code = 404

if code == 200:
    print("Success")
elif code == 404:
    print("Not Found")
elif code == 500:
    print("Server Error")
else:
    print(f"Unknown Status: {code}")
```

**The One-Path Rule:**
Only ONE block runs. Even if `code` matches multiple conditions, Python stops after the first True.

---

## 4.5 Advanced Operators (The Power Tools)

Beyond `and`/`or`, typical DevOps scripts leverage these advanced tools.

### Logical Operators (`and`, `or`, `not`)
These combine boolean conditions.

-   **Why not `&&` and `||`?** Python prioritizes readability (English words).
-   **Short-Circuiting:**
    -   `A and B`: If A is False, B is never checked. (Safety mechanism).
    -   `A or B`: If A is True, B is never checked.

### Bitwise Operators (`&`, `|`, `^`, `~`)
These work on **bits** (1s and 0s). Rarely used for logic, but CRITICAL for **File Permissions (chmod)**.

| Operator | Name | DevOps Use Case |
| :--- | :--- | :--- |
| `&` | Bitwise AND | Checking permission masking (e.g., `mode & 0o777`). |
| `|` | Bitwise OR | Adding permissions (e.g., `READ | WRITE`). |
| `^` | Bitwise XOR | Toggling bits. |
| `~` | Bitwise NOT | Inverting bits. |
| `<<` | Left Shift | Multiplying by 2 (binary shift). |
| `>>` | Right Shift | Dividing by 2. |

**Example: Checking Permissions**
```python
READ = 4   # 100
WRITE = 2  # 010
EXEC = 1   # 001

current_perm = 6  # 110 (Read + Write)

# Check if Write is allowed (Bitwise AND)
if current_perm & WRITE:
    print("Writable!")
```

### The Identity Operator (`is`)
Checks if two variables point to the **same object in memory**.

```python
a = [1, 2]
b = [1, 2]

print(a == b)  # True (Values are equal)
print(a is b)  # False (Different memory addresses)
```
**Usage:** Only use `is` for `None`. (`if x is None:`).

### The Walrus Operator (`:=`)
Python 3.8+ feature. Assigns a value *inside* an expression.

```python
# Old way
data = get_data()
if data:
    process(data)

# Walrus way
if (data := get_data()):
    process(data)
```
**Benefit:** Saves lines of code in looping through file chunks.

---

## 4.6 The Rules of Logic

1.  **Indentation is Law:** 4 spaces. No tabs.
2.  **Comparison (`==`) vs Assignment (`=`):**
    -   `if cpu = 100:` (SyntaxError)
    -   `if cpu == 100:` (Correct)
3.  **The Colon:** Don't forget the `:` at the end.
4.  **Boolean Simplicity:** Don't check `if x == True`. Just use `if x`.

---

## [!] Common Mistakes

### Error #1: Confusing `&` and `and`
```python
if user_exists & password_correct:
```
**Issue:** `&` is bitwise. It works on numbers/sets. If you use it on Booleans, it works but it is NOT short-circuited.
**Fix:** Always use `and` for logic.

### Error #2: Indentation
```python
if True:
print("Run")
```
**Fix:** Indent.

---

## Practice Problems

### Problem 1: The Permission Checker
Given a permission code `7` (RWX), use bitwise `&` to check if `EXEC` (1) is enabled.

### Problem 2: Load Balancer Logic
Write an `if/elif/else` chain:
-   Requests < 100: "Low"
-   100-500: "Normal"
-   > 500: "High"
-   **Bonus:** Use the Walrus operator to simulate reading the request count.

---

## Chapter Summary

-   **Decisions:** Scripts adapt using `if/elif/else`.
-   **Operators:** `and`/`or` for logic, `&`/`|` for permissions (bitwise).
-   **`is` vs `==`:** Memory identity vs Value equality.
-   **DevOps Context:** Essential for monitoring thresholds and permission management.
