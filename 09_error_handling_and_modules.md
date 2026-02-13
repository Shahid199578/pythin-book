# Chapter 9: Error Handling - Reliability Engineering

> "Failures are inevitable. Crashing is optional."

## The Goal: Resilience

In DevOps, networks flake, disks fill up, and APIs time out.
**Bad Script:** Crashes with a traceback when `curl` fails.
**Good Script:** Catches the error, retries, or alerts the on-call engineer.

This is **Exception Handling**.

---

## 9.1 The `try-except` Block (The Safety Net)

Wrap dangerous code in a `try` block.

```python
try:
    # Dangerous operation
    conn = connect_to_db("10.0.0.5")
    print("Combined!")
except:
    # Safety net
    print("Connection Failed. Check VPN.")
```

**Result:** The script doesn't crash. It prints the error message and *continues running*.

---

## 9.2 Catching Specific Errors

Catching *everything* (`except:`) is bad practice. You might hide real bugs (like a syntax error).
Catch only what you expect.

```python
try:
    with open("config.json", "r") as f:
        data = f.read()
except FileNotFoundError:
    print("Config missing! Using defaults.")
except PermissionError:
    print("Access Denied! Run as sudo.")
    exit(1) # Stop script with error code
```

**Common DevOps Exceptions:**
-   `TimeoutError`: Server didn't respond.
-   `ConnectionRefusedError`: Port closed.
-   `KeyError`: Missing config key.
-   `ValueError`: Letters in a port number.
-   `KeyboardInterrupt`: User hit Ctrl+C.

---

## 9.3 The `logging` Module (Beyond `print`)

Scripts running in the background shouldn't `print()`. Where does the text go?
They should **Log**.

```python
import logging

# Configure formatting
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    filename='deploy.log' # Optional: Write to file
)

logging.debug("Starting connection...") # Won't show (level is INFO)
logging.info("Deploying v1.0")
logging.warning("Disk space low: 10%")
logging.error("Database connection failed")
logging.critical("SYSTEM DOWN")
```

**Levels:**
1.  **DEBUG:** Detailed info for diagnosing problems.
2.  **INFO:** Confirmation that things are working as expected.
3.  **WARNING:** Indication that something unexpected happened.
4.  **ERROR:** A more serious problem.
5.  **CRITICAL:** A serious error, the program may be unable to continue.

---

## 9.4 Raising Errors (The Alarm)

Sometimes you *want* to crash (or signal failure to a higher function).

```python
def deploy(version):
    if version < 1.0:
        raise ValueError("Cannot deploy beta version to Prod!")
    print("Deploying...")
```

**DevOps Context:** If a health check fails, `raise SystemExit(1)` to tell standard CI/CD pipelines (Jenkins/GitLab) that the job failed.

---

## 9.5 Custom Exceptions

Why use generic errors? Create your own domain-specific errors.

```python
class DeployError(Exception):
    """Raised when deployment fails"""
    pass

class MaintenanceModeError(Exception):
    """Raised when server is in maintenance mode"""
    pass

# Usage
def verify_server(server):
    if server.status == "maintenance":
        raise MaintenanceModeError(f"{server.name} is in maintenance")
```

**Benefit:** You can catch *only* your specific errors.
```python
try:
    verify_server(s)
except MaintenanceModeError:
    print("Skipping server...")
except DeployError:
    print("CRITICAL: Deployment halted!")
```

---

## 9.6 The `traceback` Module (Deep Debugging)

Sometimes you want to handle the error, but still see *where* it happened.

```python
import traceback

try:
    risky_logic()
except Exception:
    print("Something went wrong, but I'm staying alive.")
    # Print the stack trace string without crashing
    print(traceback.format_exc())
```

This is vital for **Daemon** processes that must never crash but need to report bugs.

---

## 9.7 The `finally` Block (Cleanup)

Code that runs **no matter what** (Success or Failure).
Crucial for closing connections or deleting temporary files.

```python
lock_file = open("deploy.lock", "w")
try:
    lock_file.write("LOCKED")
    perform_long_task()
except:
    print("Failed!")
finally:
    # This runs even if we crashed above
    lock_file.close()
    print("Lock released.")
```

---

## 9.8 Under the Hood: The Stack Trace

When an error is raised, Python unwinds the **Call Stack**.
It looks for a `try` block in the current function.
Not found? It goes to the caller.
Not found? It goes to `main`.
Not found? **Crash & Print Traceback.**

---

## 9.9 The Laws of Handling

1.  **The Law of Specificity:** Catch `ValueError`, not `Exception`.
2.  **The Law of Silence:** Don't just `pass` in an `except` block. Log the error!
3.  **The Law of Cleanup:** Use `finally` or `with` to release resources.
4.  **The Law of Logging:** Production scripts log to files, not stdout.

---

## [!] Common Mistakes

### Error #1: The Empty Except
```python
try:
    upload()
except:
    pass  # EVIL!
```
**Why:** If `upload()` fails because of a typo (NameError), you will never know. The script will just silently do nothing.

### Error #2: Catching Interrupts
```python
except Exception:
```
This usually doesn't catch `Ctrl+C` (`KeyboardInterrupt`). That's a good thing! You want to be able to stop your script.

---

## Practice Problems

### Problem 1: The Retry Loop
Write a function `connect()` that randomly raises `ConnectionError`.
Write a loop that calls it. If it fails, `logging.warning()` and retry up to 3 times. If 3 failures, `raise`.

### Problem 2: The Custom Validator
Define `class ConfigError(Exception)`.
Write a function that checks a dictionary for "API_KEY". If missing, raise `ConfigError`.

---

## Chapter Summary

-   **`try/except`:** Catching errors to prevent crashes.
-   **`logging`:** Structured output for production monitoring.
-   **Custom Exceptions:** making errors descriptive (`DeployError`).
-   **Traceback:** Debugging without crashing.
-   **Reliability:** Essential for long-running services.
