# Chapter 21: Python & Git - The DevOps Gatekeeper
> "Bad code should never reach production."

## The Goal: Automated Quality
You are tired of:
1.  Running `git commit` with messy code.
2.  Reviewing PRs with syntax errors.
3.  Deploying broken scripts.

The solution is **CI/CD** (Continuous Integration/Deployment).
And Python is the glue that makes it work.

---

## 21.1 Code Quality Tools (The Robot Reviewer)

Before you commit, you must clean your code.

### 1. Formatting with `black`
Stop arguing about spaces vs tabs. Let `black` decide.
```bash
pip install black
black my_script.py
# Output: "reformatted my_script.py. All done!"
```

### 2. Linting with `pylint`
Find bugs before running the code.
```bash
pip install pylint
pylint my_script.py
# Output: "Rate: 8.5/10. Error: Undefined variable 'x'"
```

---

## 21.2 Git Hooks (The Police)

Git has a hidden folder `.git/hooks`.
You can put Python scripts there to **block bad commits**.

### Example: The "No TODOs" Hook
Create `.git/hooks/pre-commit` (and `chmod +x` it):

```python
#!/usr/bin/env python3
import sys
import subprocess

# 1. Get list of staged files
result = subprocess.run(
    ["git", "diff", "--cached", "--name-only"],
    capture_output=True, text=True
)
files = result.stdout.splitlines()

# 2. Check each file for "TODO"
for file in files:
    if file.endswith(".py"):
        with open(file, "r") as f:
            if "TODO" in f.read():
                print(f"Error: You left a TODO in {file}!")
                sys.exit(1) # Block the commit!

print("Code looks clean. Committing...")
sys.exit(0)
```

Now, if you try to commit a "TODO", Git will say **NO**.

---

## 21.3 CI Pipelines (GitHub Actions)

In the cloud (GitHub/GitLab), we define pipelines using YAML.
Python scripts are the *steps* in that pipeline.

### Example: `.github/workflows/test.yml`

```yaml
name: Python DevOps Pipeline

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
          
      - name: Install Dependencies
        run: |
          pip install -r requirements.txt
          pip install pylint black
          
      - name: Check Formatting
        run: black --check .
        
      - name: Run Linter
        run: pylint src/
        
      - name: Run Tests
        run: python -m unittest discover tests/
```

If `black` or `pylint` fails (exit code 1), the **Merge Button turns Red**.
This is how we protect Production.

---

## 21.4 DevOps Use Case: Version Bumping

A script to auto-increment your app version (Semantic Versioning).
e.g., `v1.0.2` -> `v1.0.3`.

```python
import argparse

def bump_version(path):
    with open(path, "r") as f:
        version = f.read().strip()
    
    major, minor, patch = map(int, version.split("."))
    new_version = f"{major}.{minor}.{patch + 1}"
    
    with open(path, "w") as f:
        f.write(new_version)
    
    print(f"Bumped: {version} -> {new_version}")

if __name__ == "__main__":
    bump_version("VERSION")
```

---

## 21.5 The Laws of CI/CD

1.  **The Law of Formatting:** Use `black` locally. Use `black --check` in CI.
2.  **The Law of Hooks:** Don't rely on humans to remember checks. Use `pre-commit`.
3.  **The Law of Pipelines:** Deployment should be a button press (or Git Push), not a manual command.

---

## [!] Common Mistakes

### Error #1: Committing Secrets
You accidentally committed `AWS_KEY="123"`.
**Fix:** Use `git-secrets` or a Python hook to scan for regex patterns like `AKIA...`.

### Error #2: Ignoring CI Failures
"It works on my machine!"
**Fix:** If CI fails, assume *your machine* is wrong (environment drift).

---

## Chapter Summary

-   **Linting:** Tools like `pylint` find bugs early.
-   **Git Hooks:** Python scripts that run before `commit` or `push`.
-   **CI/CD:** Using Python inside YAML pipelines (GitHub Actions) to automate testing.
-   **Quality:** Automation protects you from your own mistakes.
