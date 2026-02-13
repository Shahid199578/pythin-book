# Appendix A: DevOps Python Best Practices

## 1. Code Logic
- [ ] **Avoid Global Variables:** Keep state inside functions or classes.
- [ ] **Return vs Print:** Functions should return values, not print them.
- [ ] **Main Guard:** Always use `if __name__ == "__main__":`.

## 2. Formatting & Style
- [ ] **Snake Case:** `my_variable`, not `myVariable`.
- [ ] **Docstrings:** Document every function.
- [ ] **Formatting:** Run `black` before committing.
- [ ] **Linting:** Run `pylint` to catch bugs.

## 3. Reliability
- [ ] **Error Handling:** Use `try/except` blocks for I/O and Network calls.
- [ ] **Timeouts:** Never make a network request without a timeout.
- [ ] **Logging:** Use `logging` module, not `print` statements in production.

## 4. Security
- [ ] **Secrets:** NEVER commit API Keys or Passwords.
- [ ] **Env Vars:** Use `os.environ` for credentials.
- [ ] **Sanitization:** Never use `shell=True` with user input.

## 5. Project Structure
- [ ] **Requirements:** Always include a `requirements.txt`.
- [ ] **Gitignore:** Always ignore `.venv/`, `__pycache__/`, and `*.env`.
