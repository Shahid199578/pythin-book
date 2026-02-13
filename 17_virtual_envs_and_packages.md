# Chapter 17: Virtual Environments & Packages - The Sandbox
> "Works on my machine" is not a valid excuse.

## The Goal: Isolation

You have Project A requiring `requests==2.0`.
You have Project B requiring `requests==3.0`.
If you install them globally on your laptop, one will break.

**Virtual Environments (`venv`)** solve this by creating a self-contained folder for each project, with its own independent Python and libraries.

---

## 17.1 Creating a Virtual Environment

Navigate to your project folder and run:

```bash
# Linux/Mac
python3 -m venv .venv

# Windows
python -m venv .venv
```

This creates a hidden folder `.venv/` containing a copy of the Python binary.

### Activating the Environment
You must "enter" the sandbox to use it.

```bash
# Linux/Mac
source .venv/bin/activate

# Windows (PowerShell)
.venv\Scripts\Activate.ps1
```

**How do you know it worked?**
Your terminal prompt will change:
`(.venv) user@host:~/project$`

---

## 17.2 The Package Manager (`pip`)

`pip` is the tool to install external libraries from **PyPI** (The Python Package Index).

### Installing Packages
```bash
pip install requests
pip install boto3
```

### Listing Installed Packages
```bash
pip list
# Package    Version
# ---------- -------
# boto3      1.26.0
# requests   2.28.0
```

---

## 17.3 Dependency Management (`requirements.txt`)

You need to share your code with a teammate. They need the exact same libraries.

### Freezing Dependencies
Save your current environment to a file:
```bash
pip freeze > requirements.txt
```

**Content of `requirements.txt`:**
```text
boto3==1.26.0
requests==2.28.0
urllib3==1.26.13
```

### Installing from Requirements
Your teammate clones your code and runs:
```bash
pip install -r requirements.txt
```
Now their environment matches yours exactly.

---

## 17.4 Project Structure

Professional Python projects follow a standard layout.

```text
my_project/
├── .venv/              # The Sandbox (Gitignored!)
├── src/                # Source Code
│   ├── __init__.py
│   └── main.py
├── tests/              # Unit Tests
├── .gitignore          # Ignore .venv and __pycache__
├── README.md           # Documentation
└── requirements.txt    # Dependencies
```

**The `.gitignore` File:**
Never commit your virtual environment to Git. It's huge and OS-specific.
Add this `swp` to `.gitignore`:
```text
.venv/
__pycache__/
*.pyc
```

---

## 17.5 Under the Hood: The `sys.path`

How does Python find imports?
It looks in a list of directories called `sys.path`.

When you activate a `venv`, it modifies `sys.path` to look inside `.venv/lib/python3.X/site-packages` *before* looking in the system folders.

```python
import sys
for path in sys.path:
    print(path)
```

---

## 17.6 The Laws of Packaging

1.  **The Law of Isolation:** Never `pip install` globally (unless it's a system tool). Always use a venv.
2.  **The Law of Freezing:** Always pin your versions in `requirements.txt`.
3.  **The Law of Ignoring:** Add `.venv` to `.gitignore` immediately.

---

## [!] Common Mistakes

### Error #1: Forgetting to Activate
```bash
$ pip install requests
$ python main.py
ModuleNotFoundError: No module named 'requests'
```
**Cause:** You installed `requests` in the venv, but forgot to run `source .venv/bin/activate`.

---

## Practice Problem: The Setup

1.  Create a folder `ops_tools`.
2.  Create a `venv` inside it.
3.  Activate it.
4.  Install `requests`.
5.  Generate a `requirements.txt`.
6.  Deactivate (`deactivate` command) and delete the folder.

---

## Chapter Summary

-   **venv:** Creates isolated Python sandboxes.
-   **pip:** Installs packages from PyPI.
-   **requirements.txt:** Tracks dependencies for reproducibility.
-   **Isolation:** Essential for managing multiple projects on one machine.
