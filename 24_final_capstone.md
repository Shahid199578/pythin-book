# Chapter 24: Capstone Project - The Inspector CLI
> "Time to build."

## The Mission
You have been hired as a DevOps Engineer.
Your manager wants a single tool called `inspector` that can:
1.  Check **Disk Space**.
2.  Check **Website Health** (API Status).
3.  Report results to **JSON** or **Terminal**.
4.  Clean up **Old Logs**.

We will use everything we've learned: `argparse`, `shutil`, `requests`, `logging`, and `OOP`.

---

## 24.1 Project Structure

```text
inspector/
├── main.py        # The CLI Entrypoint
├── checks/        # Module Folder
│   ├── __init__.py
│   ├── system.py  # Disk/CPU checks
│   └── web.py     # API checks
└── requirements.txt
```

**requirements.txt:**
```text
requests==2.28.0
```

---

## 24.2 The System Check Module (`checks/system.py`)

Using `shutil` to check disk usage.

```python
import shutil

class SystemCheck:
    def check_disk(self, mount_point="/"):
        total, used, free = shutil.disk_usage(mount_point)
        percent_used = (used / total) * 100
        return {
            "check": "disk_usage",
            "mount": mount_point,
            "percent": round(percent_used, 2),
            "status": "ALARM" if percent_used > 80 else "OK"
        }
```

---

## 24.3 The Web Check Module (`checks/web.py`)

Using `requests` to monitor endpoints.

```python
import requests

class WebCheck:
    def check_site(self, url):
        try:
            r = requests.get(url, timeout=3)
            return {
                "check": "website",
                "url": url,
                "code": r.status_code,
                "latency": r.elapsed.total_seconds(),
                "status": "OK" if r.status_code == 200 else "ERROR"
            }
        except requests.exceptions.RequestException:
            return {
                "check": "website",
                "url": url,
                "status": "DOWN"
            }
```

---

## 24.4 The CLI Logic (`main.py`)

Using `argparse` to wire it together.

```python
import argparse
import json
import sys
from checks.system import SystemCheck
from checks.web import WebCheck

def main():
    # 1. Setup Arguments
    parser = argparse.ArgumentParser(description="Inspector: DevOps Health Tool")
    parser.add_argument("--check-disk", action="store_true", help="Check Disk Usage")
    parser.add_argument("--check-web", help="Check a specific Website URL")
    parser.add_argument("--json", action="store_true", help="Output as JSON")
    
    args = parser.parse_args()
    results = []

    # 2. Run Checks
    if args.check_disk:
        sys_checker = SystemCheck()
        results.append(sys_checker.check_disk())

    if args.check_web:
        web_checker = WebCheck()
        results.append(web_checker.check_site(args.check_web))

    if not results:
        parser.print_help()
        sys.exit(1)

    # 3. Output
    if args.json:
        print(json.dumps(results, indent=2))
    else:
        print("--- REPORT ---")
        for res in results:
            status = res.get('status')
            print(f"[{status}] {res['check']} | Data: {res}")
        print("--------------")

if __name__ == "__main__":
    main()
```

---

## 24.5 Running The Tool

### Scenario 1: Human Readable Check
```bash
$ python main.py --check-disk --check-web https://google.com

--- REPORT ---
[OK] disk_usage | Data: {'check': 'disk_usage', 'percent': 45.2, 'status': 'OK'}
[OK] website | Data: {'check': 'website', 'url': 'https://google.com', 'code': 200, 'status': 'OK'}
--------------
```

### Scenario 2: JSON for Automation
```bash
$ python main.py --check-disk --json
[
  {
    "check": "disk_usage",
    "mount": "/",
    "percent": 45.2,
    "status": "OK"
  }
]
```

---

## 24.6 Challenge: Extensions

You built V1. Now build V2.
1.  **Add Logging:** Write output to `inspector.log`.
2.  **Add Email Alerts:** Use `smtplib` to email if Status == "ALARM".

### 3. Configuration via YAML
Hardcoding URLs is bad. Let's use `PyYAML`.

**config.yaml**
```yaml
websites:
  - https://google.com
  - https://github.com
disk: /
```

**Modified `main.py`**
```python
import yaml # pip install pyyaml

def load_config(path="config.yaml"):
    try:
        with open(path, "r") as f:
            return yaml.safe_load(f)
    except FileNotFoundError:
        print(f"Error: Config file '{path}' not found.")
        sys.exit(1)
    except yaml.YAMLError as e:
        print(f"Error: Invalid YAML format. {e}")
        sys.exit(1)

# Inside main():
config = load_config()
for url in config['websites']:
    # check_site(url)...
```

---

## Book Conclusion

You started with `print("Hello World")`.
You ended with a Modular, Object-Oriented, CLI Tool that performs real-world checks.

**You are a Python DevOps Engineer.**
Go automate the world.
