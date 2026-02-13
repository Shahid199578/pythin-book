# Chapter 8: File I/O - Persistence & Reports

> "Data in memory is a dream. Data on disk is reality."

## The Goal: Logs, Configs, and Reports

Scripts die when they finish.
To keep data, you must write it to a **File**.
-   **Input:** Reading `config.ini`, `access.log`, or `inventory.csv`.
-   **Output:** Writing `backup_report.txt` or `audit.csv`.

In DevOps, you aren't just reading text files. You are parsing **Paths**, manipulating **Directories**, and generating **Reports**.

---

## 8.1 Reading Files (`open` and `read`)

The `open()` function is your portal to the filesystem.

```python
# 'r' means Read Mode (default)
with open("server.conf", "r") as file:
    content = file.read()
    print(content)
```

**The `with` Statement (Context Manager):**
You *could* do `file = open(...)` and `file.close()`.
but if your script crashes in the middle, the file stays open (Resource Leak).
**With `with`**, Python auto-closes the file even if errors occur. **Always use `with`!**

---

## 8.2 Reading Line-by-Line (Log Parsing)

Don't use `.read()` on a 10GB log file. You will blow up your RAM.
Iterate line by line instead.

```python
with open("access.log", "r") as log_file:
    for line in log_file:
        if "ERROR" in line:
            print(f"Alert: {line.strip()}")
```

**DevOps Note:** This is "Lazy Loading". Python reads one line, processes it, forgets it, then reads the next. Memory usage stays low.

---

## 8.3 Writing Files (`w` and `a`)

-   `"w"`: **Write**. Overwrites the entire file! (Dangerous).
-   `"a"`: **Append**. Adds to the end of the file (Safe for logs).

```python
# Overwrite configuration
with open("config.txt", "w") as f:
    f.write("host=localhost\n")
    f.write("port=8080\n")

# Append to log
with open("deploy.log", "a") as f:
    f.write("[INFO] Deployment started...\n")
```

---

## 8.4 Modern Paths with `pathlib`

Old Python used `os.path.join()`. New Python uses `pathlib`.
It treats paths as **Objects**, not Strings.

```python
from pathlib import Path

# Create a path object (Works on Windows AND Linux)
log_dir = Path("/var/log/nginx")
log_file = log_dir / "access.log"  # Use / to join paths!

print(log_file)  # /var/log/nginx/access.log

# Check existence
if not log_file.exists():
    print("Log file missing!")

# Create directory if missing
log_dir.mkdir(parents=True, exist_ok=True)
```

**Why use Pathlib?**
-   **Cross-Platform:** Handles `\` vs `/` automatically.
-   **Methods:** `.stem` (filename), `.suffix` (extension), `.parent` (folder).

```python
filename = Path("data/backup.tar.gz")
print(filename.stem)   # backup.tar (Smart!)
print(filename.suffix) # .gz
```

---

## 8.5 Working with CSV (Spreadsheets)

DevOps Engineers live in Excel/Google Sheets.
Python's `csv` module lets you read/write them easily.

### Reading CSVs
```python
import csv

with open("inventory.csv", "r") as f:
    reader = csv.DictReader(f)  # Treating first row as Headers
    for row in reader:
        print(f"Server: {row['Hostname']}, IP: {row['IP']}")
```

### Writing CSVs
```python
data = [
    {"Hostname": "web01", "IP": "10.0.0.1"},
    {"Hostname": "db01", "IP": "10.0.0.5"}
]

with open("report.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["Hostname", "IP"])
    writer.writeheader()
    writer.writerows(data)
```

**Newline Note:** Always use `newline=""` when writing CSVs to work correctly on Windows.

---

## 8.6 File Operations (`shutil`)

Need to **Copy**, **Move**, or **Delete** files?
Use `shutil` (Shell Utilities).

```python
import shutil

# Copy
shutil.copy("source.txt", "dest.txt")

# Move (Rename)
shutil.move("old.txt", "new.txt")

# Delete Directory (Recursively - Dangerous!)
# shutil.rmtree("temp_dir")
```

---

## 8.7 Working with JSON (Configs)

Text files are hard to parse. JSON is structured.

```python
import json

# Writing JSON
data = {"host": "web01", "roles": ["api", "db"]}
with open("config.json", "w") as f:
    json.dump(data, f, indent=4)

# Reading JSON
with open("config.json", "r") as f:
    config = json.load(f)
    print(config["host"])
```

---

## 8.8 Under the Hood: File Descriptors & Buffers

When you `open()` a file, the OS gives you a number (File Descriptor). Use `lsof` in Linux to see them.
**Buffering:** Python doesn't write to disk immediately. It waits for the buffer (4KB or 8KB) to fill up to save CPU.
**Force Write:** `file.flush()` forces data to disk immediately (crucial for crash logs).

---

## 8.9 The Laws of File I/O

1.  **The Law of Closing:** Always use `with open(...)` to ensure files close.
2.  **The Law of Modes:**
    -   `r` = Read (Safe).
    -   `w` = Wipe & Write (Dangerous).
    -   `a` = Append (Safe).
3.  **The Law of Newlines:** `.write()` does NOT add `\n`. You must add it manually.
4.  **The Law of Paths:** Use `pathlib`, not string concatenation for file paths.

---

## [!] Common Mistakes

### Error #1: The Missing File
```python
open("ghost.txt", "r")
```
**Error:** `FileNotFoundError`.
**Fix:** Check `if Path("ghost.txt").exists():` before opening.

### Error #2: RAM Explosion
```python
open("massive.log").read()
```
**Error:** `MemoryError`.
**Fix:** Iterate line-by-line: `for line in file:`.

### Error #3: Windows Backslashes
```python
open("C:\Users\name") 
```
**Error:** `\n` is Newline, `\u` is Unicode.
**Fix:** Use Raw Strings `r"C:\Users\name"` or `Path("C:/Users/name")`.

---

## Practice Problems

### Problem 1: The Cleaner
Write a script that reads "dirty_log.txt", removes empty lines, and saves it to "clean_log.txt".

### Problem 2: The CSV Converter
Read a CSV "users.csv" (Name, Role).
Write a JSON file "users.json" containing the same data as a list of dictionaries.

### Problem 3: The Backup Script
Use `shutil` to copy all `.conf` files from `/etc/myapp` to `/backup/conf`.
(Hint: Use `pathlib.Path("/etc/myapp").glob("*.conf")` to find them).

---

## Chapter Summary

-   **`open()`:** Opens a file handle.
-   **`with`:** Auto-closes the handle (Safety).
-   **`pathlib`:** Object-oriented path handling (Modern Standard).
-   **`csv`:** Handling spreadsheets.
-   **`shutil`:** Copying/Moving files like a shell script.
-   **JSON:** The standard format for DevOps configs.
