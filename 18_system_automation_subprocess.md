# Chapter 18: System Automation - Python as Glue
> "Python is a better Bash."

## The Goal: Replacing Shell Scripts
Bash is great for pipes (`|`). Python is great for logic.
When your Bash script needs arrays, loops, or JSON parsing, switch to Python.

But how do you run Linux commands (`ls`, `grep`, `systemctl`) from Python?
**Enter the `subprocess` module.**

---

## 18.1 Running Commands (`subprocess.run`)

The modern, safe way to execute commands.

```python
import subprocess

# Simple execution
subprocess.run(["ls", "-l"])
```

**Why the List?** `["ls", "-l"]`
Accessing the shell is dangerous. If you used a string `"ls -l"`, a hacker could inject commands.
By passing a list, Python creates the process directly, bypassing the shell. **Safe.**

---

## 18.2 Capturing Output (stdout)

Usually, you want to store the output in a variable, not just print it.

```python
result = subprocess.run(
    ["uname", "-a"],
    capture_output=True,
    text=True  # Decode bytes to string
)

print("Return Code:", result.returncode)
print("Output:", result.stdout)
```

-   **`capture_output=True`**: Redirects stdout/stderr to memory.
-   **`text=True`**: Converts generic Bytes (`b'Linux\n'`) to String (`"Linux\n"`).

---

## 18.3 Handling Errors (`check=True`)

If the command fails (e.g., `ls /nonexistent`), `subprocess.run` does *not* crash by default. It just sets `returncode=1`.

To force Python to raise an error (good for "Stop on Error" scripts):

```python
try:
    subprocess.run(["ls", "/ghost"], check=True)
except subprocess.CalledProcessError as e:
    print(f"Command failed with code {e.returncode}")
```

Use `check=True` when the command **must** succeed for the script to continue.

---

## 18.4 Environment Variables (`os.environ`)

Scripts need secrets (API Keys, DB Passwords).
**Never** hardcode them. Read them from the Environment.

```python
import os

# Reading
api_key = os.environ.get("AWS_ACCESS_KEY_ID")

if not api_key:
    print("Error: AWS_ACCESS_KEY_ID is missing!")
    exit(1)

# Writing (for child processes only)
os.environ["TEMP_DIR"] = "/tmp/my_script"
subprocess.run(["./backup.sh"]) # backup.sh sees TEMP_DIR
```

---

## 18.5 The "Shell=True" Danger

Sometimes you need pipes (`|`) or wildcards (`*`). `subprocess` doesn't support them by default.
You *can* use `shell=True`, but be careful.

```python
# Dangerous if 'user_input' comes from the web!
subprocess.run(f"grep {user_input} file.txt", shell=True)
```
If `user_input` is `"; rm -rf /"`, you are dead.

**Safe Alternative (Do the pipe in Python!):**
1.  Run `grep` via subprocess.
2.  Capture output.
3.  Filter it in Python code.

---

## 18.6 DevOps Use Case: Service Restarter

A script that checks Nginx and restarts it if dead.

```python
import subprocess
import time

def check_nginx():
    try:
        res = subprocess.run(
            ["systemctl", "is-active", "nginx"], 
            capture_output=True, text=True
        )
        return res.stdout.strip() == "active"
    except FileNotFoundError:
        # Handles cases where systemctl is missing (e.g. inside a container)
        print("Error: 'systemctl' command not found.")
        return False

def restart_nginx():
    print("Restarting Nginx...")
    subprocess.run(["sudo", "systemctl", "restart", "nginx"], check=True)

if not check_nginx():
    restart_nginx()
    time.sleep(2)
    if check_nginx():
        print("Success: Nginx is back online.")
    else:
        print("Critical: Nginx failed to restart.")
else:
    print("Nginx is healthy.")
```

---

## 18.7 The Laws of Subprocess

1.  **The Law of Lists:** Always pass commands as `["cmd", "arg"]`, not strings.
2.  **The Law of Checking:** Use `check=True` to catch failures automatically.
3.  **The Law of Sanitization:** Avoid `shell=True` unless absolutely necessary.
4.  **The Law of Secrets:** Use `os.environ`, not hardcoded strings.

---

## [!] Common Mistakes

### Error #1: Bytes vs Strings
```python
res = subprocess.run(["ls"], capture_output=True)
print(res.stdout) # b'file1\nfile2\n'
```
**Fix:** Pass `text=True` (or `universal_newlines=True` in older Python).

### Error #2: Blocking
`subprocess.run` **waits** for the command to finish. If you run a command that never ends (like `tail -f`), your Python script hangs forever.
**Fix:** Use `subprocess.Popen` for background tasks (Advanced).

---

## Practice Problem: The Backup

Write a script that:
1.  Uses `tar` (via subprocess) to compress a folder.
2.  Uses `mv` (via subprocess) to move it to a `/backup` folder.
3.  Reads `BACKUP_DIR` from an environment variable.

---

## Chapter Summary

-   **subprocess.run:** The standard way to run shell commands.
-   **Safety:** Avoid shell injection by using list arguments.
-   **Environment:** Use `os.environ` to configure your scripts dynamically.
-   **Power:** Combine Linux tools (CLI) with Python logic (Loops/Libs).
