# Appendix C: Common Beginner Mistakes

> "Experience is the name everyone gives to their mistakes." — Oscar Wilde

## 1. Mutable Default Arguments
**The Bug:**
```python
def add_server(server, list=[]):
    list.append(server)
    return list

print(add_server("srv1")) # ['srv1']
print(add_server("srv2")) # ['srv1', 'srv2'] -- Wait, what?
```
**The Why:** The list is created *once* when the function is defined, not every time it runs.
**The Fix:** Use `None`.
```python
def add_server(server, list=None):
    if list is None:
        list = []
```

## 2. The Broad Exception
**The Bug:**
```python
try:
    do_something()
except:
    pass # "Silence is golden"
```
**The Why:** This catches *everything*, including `KeyboardInterrupt` (Ctrl+C). You can't stop your script!
**The Fix:** Catch specific errors.
```python
except (ValueError, FileNotFoundError):
    logging.error("File missing")
```

## 3. Shell Injection
**The Bug:**
```python
subprocess.run(f"ping {user_input}", shell=True)
```
**The Why:** If user inputs `; rm -rf /`, your server is gone.
**The Fix:** Use lists, not strings.
```python
subprocess.run(["ping", user_input])
```

## 4. Shadowing Built-ins
**The Bug:**
```python
list = [1, 2, 3] # You just killed the 'list()' function
str = "Hello"    # You just killed 'str()'
```
**The Why:** Python lets you overwrite standard functions.
**The Fix:** Use synonyms like `my_list` or `data_str`.

## 5. Path Hardcoding
**The Bug:**
```python
path = "C:\\Users\\Admin\\file.txt" # Breaks on Linux
```
**The Why:** Windows uses `\`, Linux uses `/`.
**The Fix:** Use `pathlib` or `os.path.join`.
```python
from pathlib import Path
path = Path.home() / "file.txt"
```
