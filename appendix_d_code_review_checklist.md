# Appendix D: Code Review Checklist

> "Code is a liability. The less of it you have, the better."

Use this checklist before merging any Pull Request.

## 1. Functionality & Logic
- [ ] **Does it work?** Has the author provided proof (screenshots/logs)?
- [ ] **Edge Cases:** What happens if the file is empty? If the API is down?
- [ ] **Complexity:** Can this 50-line function be 5 lines?
- [ ] **DRY (Don't Repeat Yourself):** Is there copied logic? Make it a function.

## 2. Style (PEP 8)
- [ ] **Naming:** `snake_case` for variables/functions, `PascalCase` for classes.
- [ ] **Docstrings:** Does every function explain *what* it does and *what* it returns?
- [ ] **Formatting:** Has `black` been run?
- [ ] **Imports:** Are imports at the top? Are unused imports removed?

## 3. Security
- [ ] **Secrets:** Are there API keys/passwords in the code? (REJECT IMMEDIATELY).
- [ ] **Input Validation:** Is user input sanitized?
- [ ] **Shell Safety:** Is `shell=True` avoided in `subprocess`?
- [ ] **Dependencies:** Are we pinning versions in `requirements.txt`?

## 4. Reliability & DevOps
- [ ] **Logging:** Are there `print()` statements? (Ask to change to `logging`).
- [ ] **Error Handling:** Are there bare `except:` blocks?
- [ ] **Configuration:** Are parameters (URLs, timeouts) configurable (Env Vars/YAML)?
- [ ] **Cleanup:** Does the script close files and DB connections (using `with`)?

## 5. Testing
- [ ] **Unit Tests:** Are there tests for the new logic?
- [ ] **Happy Path:** Does it work when everything is right?
- [ ] **Sad Path:** Does it fail gracefully when things go wrong?
