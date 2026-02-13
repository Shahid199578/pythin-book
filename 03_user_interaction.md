# Chapter 3: User Interaction - The CLI Experience

> "Tools that don't talk back are dangerous."

## The Goal: Interactive Scripts

Most DevOps scripts run silently in the background (cron jobs).
But sometimes you need to build **Interactive CLI Tools** for other developers.
-   "Which database do you want to restore?"
-   "Are you sure you want to delete production? (yes/no)"

---

## 3.1 Getting Input: `input()`

The `input()` function pauses the script and waits for the admin to type.

```python
env = input("Target Environment (dev/prod): ")
print(f"Deploying to {env}...")
```

**What happens:**
1.  Script prints "Target Environment (dev/prod): "
2.  Script **BLOCKS** (freezes) until Enter is pressed.
3.  Admin types "dev".
4.  Variable `env` becomes `"dev"`.

---

## 3.2 Under the Hood: stdin & stdout

In Linux, everything is a file stream.
-   **stdin (Standard Input):** Keyboard -> Python
-   **stdout (Standard Output):** Python -> Screen

**DevOps Trick:** You can Pipe data into your script!
```bash
echo "prod" | python deploy.py
```
Because `input()` reads from stdin, it will see "prod" and continue without stopping. This is how you automate interactive scripts!

---

## 3.3 The Law of Strings: All Input is Text

Even if you type a port number, it comes in as text.

```python
port = input("Enter Port: ")  # User types 8080
# type(port) is <class 'str'>
```

**The Crash:**
```python
next_port = port + 1
# TypeError: can only concatenate str (not "int") to str
```

**The Fix (Casting):**
```python
port = int(input("Enter Port: "))
next_port = port + 1  # 8081
```

---

## 3.4 Handling Validation (Sanitizing Input)

Admins make typos.
If you ask for "yes", they might type "  YES  ".

```python
confirm = input("Delete Database? (yes/no): ")

# Bad
if confirm == "yes": ...

# Good (Sanitized)
clean_confirm = confirm.strip().lower()
if clean_confirm == "yes": ...
```

---

## 3.5 The Toolkit: 20+ Essential String Methods

In DevOps, 90% of your work is parsing text (logs, configs, outputs).
Python has a massive toolkit for this.

### Group 1: The Cleaners
Sanitize your input.

| Method | Description | Example | Result |
| :--- | :--- | :--- | :--- |
| `.strip()` | Removes whitespace from both ends | `"  Hi  ".strip()` | `"Hi"` |
| `.lstrip()` | Removes whitespace from left | `"  Hi".lstrip()` | `"Hi"` |
| `.rstrip()` | Removes whitespace from right | `"Hi  ".rstrip()` | `"Hi"` |
| `.replace(old, new)` | Replaces substring | `"v1.0".replace(".", "-")` | `"v1-0"` |
| `.lower()` | Converts to lowercase | `"Admin".lower()` | `"admin"` |
| `.upper()` | Converts to uppercase | `"dev".upper()` | `"DEV"` |
| `.title()` | Capitalizes first letter of words | `"john doe".title()` | `"John Doe"` |
| `.capitalize()` | Capitalizes only first letter | `"hello world".capitalize()` | `"Hello world"` |
| `.swapcase()` | Swaps case | `"Hi".swapcase()` | `"hI"` |

### Group 2: The Inspectors
Check what the string contains (returns True/False).

| Method | Description | Example | Result |
| :--- | :--- | :--- | :--- |
| `.startswith(x)` | Checks start | `"error.log".startswith("err")` | `True` |
| `.endswith(x)` | Checks end | `"sys.conf".endswith(".conf")` | `True` |
| `.isdigit()` | Checks if only numbers | `"123".isdigit()` | `True` |
| `.isalpha()` | Checks if only letters | `"abc".isalpha()` | `True` |
| `.isalnum()` | Checks if letters+numbers | `"web01".isalnum()` | `True` |
| `.isspace()` | Checks if only whitespace | `"   ".isspace()` | `True` |
| `.islower()` | Checks if lowercase | `"abc".islower()` | `True` |
| `.isupper()` | Checks if uppercase | `"ABC".isupper()` | `True` |

### Group 3: The Searchers & Splitters
Find data inside strings.

| Method | Description | Example | Result |
| :--- | :--- | :--- | :--- |
| `.find(x)` | Returns index of x (or -1) | `"Error: Disk".find(":")` | `5` |
| `.count(x)` | Counts occurrences | `"10.0.0.1".count(".")` | `3` |
| `.split(x)` | Splits into a list | `"a,b,c".split(",")` | `['a', 'b', 'c']` |
| `.splitlines()` | Splits by newline | `"Line1\nLine2".splitlines()` | `['Line1', 'Line2']` |
| `.join(list)` | Joins list into string | `"-".join(['a', 'b'])` | `"a-b"` |
| `.zfill(width)` | Pads with zeros | `"5".zfill(3)` | `"005"` |
| `.center(width)` | Centers text | `"Menu".center(10)` | `"   Menu   "` |

---

## 3.6 Use F-Strings for Logging

Building log messages with `+` is slow and ugly.
Use **f-strings** (formatted string literals).

**Bad:**
```python
print("Deploying version " + str(ver) + " to server " + host)
```

**Good:**
```python
print(f"Deploying version {ver} to server {host}")
```

**DevOps Pro Tip:** You can format numbers too!
```python
success_rate = 0.9567
print(f"Success Rate: {success_rate:.1%}")
# Output: Success Rate: 95.7%
```

---

## 3.7 The 5 Laws of Input

1.  **The Law of Casting:** Input is *always* a string. Cast it (`int()`).
2.  **The Law of Hygiene:** Always `.strip()` input.
3.  **The Law of Skepticism:** Never trust input. Validate it.
4.  **The Law of Clarity:** Use f-strings.
5.  **The Law of Resilience:** Expect the user to hit Enter without typing anything.

---

## [!] Common Mistakes

### Error #1: The Hanging Script
You wrote `input()` in a script meant for a cron job.
**Result:** The script runs forever, waiting for a ghost to type something.
**Fix:** Use command-line arguments (sys.argv) or environment variables for non-interactive scripts.

### Error #2: The Newline Trap
When reading from a file, `readline()` returns `"server01\n"`.
If you don't `.strip()`, your code breaks:
```python
if line == "server01":  # False because of \n
```

---

## Practice Problems

### Problem 1: The Log Parser
Given a log line: `"[ERROR] Connection Refused - 10.0.0.5"`.
-   Use methods to extract the IP address.
-   Check if it starts with `[ERROR]`.

### Problem 2: The ID Generator
Ask for a username (e.g., " john ").
-   Strip whitespace.
-   Convert to lowercase.
-   Replace spaces with dots.
-   Pad with zeros to make it at least 8 chars long (`zfill` won't work perfectly on strings, try `.rjust` or logic!).

---

## Chapter Summary

-   **`input()`**: Reads from stdin (Keyboard or Pipe).
-   **Sanitization:** The 20+ methods are your toolkit for cleaning messy data.
-   **Casting:** Always convert numeric input (`int()`).
-   **Streams:** Understanding stdin helps you automate automation.
