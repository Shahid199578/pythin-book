# Chapter 20: Web APIs - Talking to the World
> "The Internet is just other people's computers talking JSON."

## The Goal: Integration
Your server needs to alert Slack.
Your CI/CD pipeline needs to trigger a build on Jenkins.
Your monitoring script needs to query AWS IP ranges.

All of these use **HTTP APIs**.
Python's `requests` library is the gold standard for this.

---

## 20.1 GET Requests (Reading Data)

Browsers send GET requests to view pages. Scripts do the same to get JSON.

```python
import requests

# 1. Send Request
response = requests.get("https://api.github.com/zen")

# 2. Check Status
print(f"Status: {response.status_code}") # 200 = OK

# 3. Read Content
print(f"Message: {response.text}")
```

---

## 20.2 Working with JSON Payloads

Most APIs return JSON. Requests has a built-in `.json()` decoder.

```python
r = requests.get("https://jsonplaceholder.typicode.com/todos/1")

if r.status_code == 200:
    data = r.json() # Converts JSON string -> Python Dict
    print(f"Task: {data['title']}")
    print(f"Completed: {data['completed']}")
else:
    print("Error fetching data")
```

---

## 20.3 POST Requests (Sending Data)

To create data (e.g., Sending a Slack message), uses POST.

```python
url = "https://hooks.slack.com/services/T000/B000/XXXX"
payload = {"text": "Deployment Started!"}

# Requests auto-sets Content-Type: application/json
r = requests.post(url, json=payload)

if r.status_code == 200:
    print("Message Sent!")
```

---

## 20.4 Handling Headers (Authentication)

Most APIs require an API Key. This goes in the **Header**.

```python
headers = {
    "Authorization": "Bearer MY_SECRET_TOKEN",
    "User-Agent": "MyPythonScript/1.0"
}

r = requests.get("https://api.example.com/secure-data", headers=headers)
```

---

## 20.5 DevOps Use Case: The Health Check

A script that monitors a list of websites.

```python
import requests
import time

urls = [
    "https://google.com",
    "https://github.com",
    "https://nonexistent-site-123.com"
]

def check_sites():
    for url in urls:
        try:
            r = requests.get(url, timeout=5) # Don't hang forever!
            if r.status_code == 200:
                print(f"[OK] {url}")
            else:
                print(f"[ERR] {url} returned {r.status_code}")
        except requests.exceptions.ConnectionError:
            print(f"[DOWN] {url} unreachable")
        except requests.exceptions.Timeout:
            print(f"[SLOW] {url} timed out")

check_sites()
```

---

## 20.6 The Laws of Requests

1.  **The Law of Timeouts:** Always set `timeout=5`. Default is infinite! If the server hangs, your script hangs.
2.  **The Law of Status:** Don't assume success. Check `if r.status_code == 200`.
3.  **The Law of Secrets:** Never put API keys in the URL. Use Headers.
4.  **The Law of Rate Limiting:** Don't loop a request 1,000 times a second. You will get banned.

---

## [!] Common Mistakes

### Error #1: Generic Except
```python
try:
    requests.get(url)
except:
    print("Fail")
```
This hides bugs. Catch specific errors: `requests.exceptions.RequestException`.

### Error #2: JSON Decode Error
If the server returns 500 (HTML Error Page), `r.json()` will crash because it's not JSON.
**Fix:** Check status code first.

---

## Practice Problem: The Weather Bot

1.  Use a free weather API (like `open-meteo.com`).
2.  Ask the user for a Latitude/Longitude (using `input()` or `argparse`).
3.  Fetch the current temperature.
4.  Print "Bring a Jacket" if temp < 15°C.

---

## Chapter Summary

-   **requests:** The most popular Python library (for a reason).
-   **Methods:** GET (Read), POST (Write).
-   **JSON:** The native language of APIs.
-   **Status Codes:** 200 (Good), 404 (Missing), 500 (Server Broken).
-   **Automation:** APIs allow scripts to control other systems.
