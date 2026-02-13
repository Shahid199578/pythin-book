# Chapter 6: Functions - Building Tools

> "The Linux philosophy: Do one thing and do it well."

## The Goal: Modular Scripts

You copy-pasted the backup code 10 times. Now you need to change the path.
**Pain:** You have to edit 10 places.
**Solution:** Wrap it in a function.

A function is a **Reusable Tool**.

---

## 6.1 Defining a Function (`def`)

```python
def check_disk_space():
    percent = get_usage("/")
    print(f"Disk Usage: {percent}%")
```

**Calling it:**
```python
check_disk_space()
```

---

## 6.2 Parameters (Inputs)

Make the tool flexible.

```python
def check_disk_space(mount_point):
    percent = get_usage(mount_point)
    print(f"Usage for {mount_point}: {percent}%")

check_disk_space("/var/log")
check_disk_space("/home")
```

---

## 6.3 Return Values (Outputs)

Scripts need Data, not Print statements.

```python
def get_uptime(server):
    uptime_seconds = fetch_metric(server, "uptime")
    return uptime_seconds / 3600  # Return hours

hours = get_uptime("web-01")
if hours < 1:
    print("Server recently rebooted")
```

**Why Return?**
Because you can't do math on `print()`.

---

## 6.4 UNDER THE HOOD: The Call Stack

When you run `deploy()`, Python creates a workspace (Stack Frame).
-   Variables (`server_ip`, `status`) live here.
-   When `deploy()` finishes, the Frame is destroyed.

**Recursion:** A function calling itself (e.g., traversing directories).
Too deep? `RecursionError` (Stack Overflow).

---

## 6.5 Variable Scope (Global vs Local)

```python
api_key = "12345"  # Global (Read-only usually)

def connect():
    socket = create_connection() # Local (Destroyed after function)
```

**Rule:** Passed parameters are better than Globals.

---

## 6.6 Default Parameters (Configuration)

```python
def connection(host, port=22, timeout=10):
    print(f"SSH to {host}:{port} (timeout {timeout}s)")

connection("web-01")        # Uses 22, 10
connection("db-01", 5432)   # Overrides port
```

**Warning:** Mutable Defaults (`list=[]`) persist between calls. Avoid them!

---

## 6.7 Function Design Laws

1.  **Rule of Singularity:** One job per function (`parse_log`, not `parse_and_upload`).
2.  **Rule of Descriptiveness:** `backup_db()` is better than `run()`.
3.  **Rule of Return:** Return data for the caller to handle.
4.  **Rule of Scope:** Don't rely on global state.

---

## [!] Common Mistakes

### Error #1: The Phantom Return
```python
def add(a, b):
    res = a + b
    # Forgot return!

total = add(10, 20)
print(total) # None
```
**Fix:** `return res`.

### Error #2: Global Modification
```python
count = 0
def inc():
    count = 1  # Creates a NEW local variable 'count'

inc()
print(count) # 0
```
**Fix:** `return count + 1`.

---

## Practice Problems

### Problem 1: URL Builder
Write `build_url(host, path, ssl=True)`.
Returns `https://host/path` or `http://host/path`.

### Problem 2: Log Formatter
Write `log(level, message)` that prints `[TIMESTAMP] [LEVEL] Message`.

---

## Chapter Summary

-   **Functions:** Wrap logic into named blocks.
-   **Parameters:** Inputs make functions flexible.
-   **Return:** Outputs allow chaining logic.
-   **Scope:** Local variables keep memory clean.
-   **Design:** Keep functions small (Unix Philosophy).
