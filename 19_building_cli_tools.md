# Chapter 19: Building CLI Tools - The Interface
> "A tool without documentation is a puzzle."

## The Goal: Professional UX
You wrote a script `backup.py`.
User A runs it: `python backup.py` -> Crashes.
User B runs it: `python backup.py /tmp` -> "Unknown argument".

You want your script to behave like `ls` or `git`.
-   `--help` menu.
-   Flags like `--verbose` or `--dry-run`.
-   Required vs Optional arguments.

**`argparse`** is the standard library for this.

---

## 19.1 specific vs Generic (`argparse` vs `sys.argv`)

**The Amateur Way (`sys.argv`):**
```python
import sys
filename = sys.argv[1] # Crashes if user forgets the argument!
```

**The Pro Way (`argparse`):**
Auto-generates help menus and handles errors gracefully.

---

## 19.2 Your First Robust CLI

```python
import argparse

# 1. Create the Parser
parser = argparse.ArgumentParser(description="A robust backup tool.")

# 2. Add Arguments
parser.add_argument("source", help="Directory to back up")
parser.add_argument("dest", help="Destination folder")
parser.add_argument("--compress", action="store_true", help="Enable GZIP compression")

# 3. Parse Args
args = parser.parse_args()

# 4. Use Logic
print(f"Backing up {args.source} to {args.dest}...")
if args.compress:
    print("Compression Enabled.")
```

**Try running it:**
-   `python cli.py --help` -> Show nice menu.
-   `python cli.py src/` -> Error: the following arguments are required: dest.
-   `python cli.py src/ /bak --compress` -> Works!

---

## 19.3 Argument Types

### 1. Positional Arguments
Mandatory. Order matters. (Like `source` and `dest` above).

### 2. Optional Flags (`--name`)
Optional. Order doesn't matter.
```python
parser.add_argument("-v", "--verbose", action="store_true", help="Talkative mode")
```
Usage: `python script.py -v`

### 3. Application with Values
Taking an integer or a specific string choice.
```python
parser.add_argument("--port", type=int, default=80, help="Server Port")
parser.add_argument("--env", choices=["dev", "prod"], default="dev")
```
Python automatically validates:
-   `--port ABC` -> Error (expects int).
-   `--env mylaptop` -> Error (must be dev/prod).

---

## 19.4 DevOps Use Case: The Deploy Tool

```python
import argparse

def main():
    parser = argparse.ArgumentParser(description="Deploy App V1")
    
    # Target Selection
    parser.add_argument("service", help="Service name (e.g. 'web', 'db')")
    
    # Configuration
    parser.add_argument("--region", default="us-east-1", help="AWS Region")
    parser.add_argument("--replicas", type=int, default=1, help="Container Count")
    parser.add_argument("--dry-run", action="store_true", help="Simulate only")

    args = parser.parse_args()

    if args.dry_run:
        print(f"[DRY RUN] Would deploy {args.service} to {args.region} x {args.replicas}")
        return

    print(f"Deploying {args.service}...")
    # Real logic here

if __name__ == "__main__":
    main()
```

---

## 19.5 The Laws of CLIs

1.  **The Law of Help:** Every tool must have a useful `--help` (argparse does this for you).
2.  **The Law of Defaults:** Provide sensible defaults (e.g., port 80).
3.  **The Law of Feedback:** If `--verbose` is on, tell the user what is happening.
4.  **The Law of Validation:** Use `type=int` and `choices=[]` to stop bad input early.

---

## [!] Common Mistakes

### Error #1: Confusing `-` and `--`
-   `-` (One dash) for single letters: `-v`.
-   `--` (Two dashes) for words: `--verbose`.

### Error #2: Hardcoding Paths
Don't write `filename = "backup.txt"` inside your code.
Make it an argument: `parser.add_argument("--file", default="backup.txt")`.
Now your script is reusable!

---

## Practice Problem: The Log Searcher

Create a tool `search.py` that takes:
1.  `file`: Positional argument (Path to log).
2.  `--keyword`: Optional argument (Default: "ERROR").
3.  `--limit`: Optional Integer (Max lines to print).

It should print lines from `file` containing `keyword`, stopping after `limit` matches.

---

## Chapter Summary

-   **argparse:** The standard solution for Python CLIs.
-   **Positional:** Mandatory inputs (`cp source dest`).
-   **Optional:** Flags (`ls -l`).
-   **Validation:** Automatic type checking and help generation.
-   **Professionalism:** CLIs distinguish Scripts from Tools.
