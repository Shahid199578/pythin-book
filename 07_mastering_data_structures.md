# Chapter 7: Data Structures - The System Inventory

> "Organizing data is half the battle."

## The Goal: Managing Collections

You manage 1,000 servers.
**Bad Way:** `server1 = "web01"`, `server2 = "web02"`...
**Good Way:** Put them in a **List**.

You need to know the IP, OS, and Uptime for each.
**Good Way:** Put them in a **Dictionary**.

---

## 7.1 Lists (Ordered Collections)

A list is like a rack of servers. Ordered and indexable.

```python
servers = ["web-01", "db-01", "cache-01"]
```

### Accessing Items
Zero-indexed (like rack units starting at 0).

```python
print(servers[0])  # "web-01"
print(servers[-1]) # "cache-01" (Last one)
```

### Modifying Lists (Mutable)
You can add or remove hardware.

```python
servers.append("web-02")      # Add to end
servers.insert(0, "lb-01")    # Add to front
servers.remove("db-01")       # Decommission
```

---

## 7.2 Tuples (Immutable Lists)

Sometimes you want a list that **cannot** be changed.
Example: Database Credentials, Geographic Coordinates.

```python
db_config = ("192.168.1.5", 5432)
# db_config[0] = "10.0.0.1"  # ERROR: TypeError
```

**DevOps Use:** Return multiple values from a function.
```python
def get_metrics():
    return (0.5, 2048)  # Load, RAM

load, ram = get_metrics()  # Unpacking
```

---

## 7.3 Dictionaries (Key-Value Stores)

The most useful structure for DevOps. It maps **Keys** (Names) to **Values** (Data).
Think of it like a JSON configuration file.

```python
server_config = {
    "hostname": "web-01",
    "ip": "10.0.0.5",
    "active": True,
    "roles": ["web", "api"]
}
```

### Accessing Data
```python
print(server_config["ip"])  # "10.0.0.5"
```

### Adding/Updating
```python
server_config["region"] = "us-east"  # Add new key
server_config["active"] = False      # Update existing
```

---

## 7.4 Sets (Unique Collections)

A bag of unique items. No duplicates allowed.
Great for combining lists of IPs from different logs.

```python
error_ips = {"10.0.0.1", "10.0.0.2", "10.0.0.1"}
print(error_ips)
# Output: {"10.0.0.1", "10.0.0.2"} (Duplicate removed)
```

**Set Math (Venn Diagrams):**
```python
group_a = {"web01", "web02"}
group_b = {"web02", "db01"}

# Intersection (Who is in BOTH?)
print(group_a & group_b)  # {"web02"}

# Union (Combine both)
print(group_a | group_b)  # {"web01", "web02", "db01"}
```

---

## 7.5 Under the Hood: Hash Maps (Dictionaries)

Why are Dictionaries so fast?
If you have a list of 1,000,000 items, searching it takes time (O(n)).
If you have a dictionary, looking up a key is instant (O(1)).

**How?** Python runs the key through a **Hash Function** (e.g., `hash("ip")` -> `12345`).
It uses that number to jump directly to the memory address. No searching required.

---

## 7.6 Nested Data Structures (Complex Configs)

Real-world configs are rarely flat. You put lists inside dictionaries, and dictionaries inside lists.

### List of Dictionaries (Server Inventory)
Commonly returned by cloud APIs (AWS/Azure).

```python
inventory = [
    {"hostname": "web-01", "ip": "10.0.0.1", "role": "web"},
    {"hostname": "db-01", "ip": "10.0.0.5", "role": "db"},
    {"hostname": "cache-01", "ip": "10.0.0.9", "role": "cache"}
]

# Accessing the IP of the second server
print(inventory[1]["ip"])  # "10.0.0.5"

# Looping through them
for server in inventory:
    print(f"Pinging {server['hostname']}...")
```

### Dictionary of Lists (Grouping)

```python
load_balancers = {
    "us-east": ["lb-01", "lb-02"],
    "eu-west": ["lb-eu-01", "lb-eu-02"]
}

# Accessing the first LB in eu-west
print(load_balancers["eu-west"][0])  # "lb-eu-01"
```

---

## 7.7 The Laws of Data Structures

1.  **The Law of Order:** Lists and Tuples preserve order. Sets and Dicts (mostly) do not.
2.  **The Law of Uniqueness:** Sets and Dictionary Keys must be unique.
3.  **The Law of Mutability:**
    -   Lists/Dicts/Sets -> Mutable (Changeable).
    -   Tuples/Strings -> Immutable (Frozen).

---

## [!] Common Mistakes

### Error #1: Key Error
```python
print(config["password"])
```
**Error:** `KeyError: 'password'`
**Fix:** Use `.get()` for safe access.
```python
print(config.get("password", "default"))
```

### Error #2: Modifying Tuples
```python
coords = (10, 20)
coords[0] = 5
```
**Error:** `TypeError`. Tuples are read-only.

---

## Practice Problems

### Problem 1: The Inventory Manager
Create a dictionary representing a server (hostname, ip, os).
Add a new key "uptime".
Print the OS.

### Problem 2: The Log Deduper
Given a list of IPs: `["1.1.1.1", "2.2.2.2", "1.1.1.1"]`.
Use a Set to remove duplicates and print the unique count.

### Problem 3: The Nested Config
Create a list of 3 dictionaries (each representing a user with `name` and `role`).
Loop through the list and print the name ONLY if the role is "admin".

---

## Chapter Summary

-   **Lists:** Ordered, mutable sequences (Server Racks).
-   **Tuples:** Immutable sequences (Coordinates).
-   **Dicts:** Key-Value stores (Configurations).
-   **Sets:** Unique bags of items (IP Whitelists).
-   **Nesting:** Combining these structures allows needed complexity.
