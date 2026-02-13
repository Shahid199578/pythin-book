# Chapter 13: Stacks and Queues - Managing Flow

> "First In, First Out. Or Last In, First Out. Pick one."

## The Goal: Job Management

How do you handle 1,000 users trying to upload logs at once?
You can't process them all instantly. You need a **Buffer**.
In DevOps, these buffers are **Queues** (RabbitMQ, Kafka, Celery).

---

## 13.1 Stacks (LIFO - Last In, First Out)

Think of a stack of plates.
-   You put a plate on top (**Push**).
-   You take the top plate off (**Pop**).
The **Last** one you put in is the **First** one out.

### Python Implementation
Python lists are perfect stacks.

```python
stack = []

# Push (Append)
stack.append("Page 1")
stack.append("Page 2")
stack.append("Page 3")

# Pop (Remove from end)
current = stack.pop()
print(current) # "Page 3"
```

### DevOps Use Case: The "Undo" Button
Deployments often work like stacks.
1.  Deploy Database.
2.  Deploy API.
3.  Deploy Frontend.
**Error!** Frontend fails.
**Rollback:**
1.  Undo Frontend (Pop).
2.  Undo API (Pop).
3.  Undo Database (Pop).
(Reverse order of operations).

---

## 13.2 Queues (FIFO - First In, First Out)

Think of a line at a ticket counter.
-   People join the back (**Enqueue**).
-   People are served from the front (**Dequeue**).

### The Performance Trap of Lists
**Bad:** Using `list.pop(0)`.
Removing item 0 forces Python to **shift** all other items left. This is slow **O(n)**.

**Good:** Using `collections.deque` (Double-Ended Queue).
It allows O(1) adds and removes from both ends.

```python
from collections import deque

job_queue = deque()

# Enqueue (Add to right)
job_queue.append("Job 1: Resize Image")
job_queue.append("Job 2: Send Email")

# Dequeue (Remove from left)
task = job_queue.popleft() # Fast! O(1)
print(task) # "Job 1..."
```

---

## 13.3 Priority Queues (VIP Lines)

Sometimes a "Critical Security Patch" should jump the line ahead of "Daily Backup".
Python uses `heapq` for this.

```python
import heapq

# List of (Priority, Task)
# Priority 1 is highest (served first)
tasks = []

heapq.heappush(tasks, (3, "Backup"))
heapq.heappush(tasks, (1, "Security Patch"))
heapq.heappush(tasks, (2, "Update Docs"))

# Process (heappop always gives the smallest number first)
while tasks:
    priority, task = heapq.heappop(tasks)
    print(f"Doing: {task}")
    
# Output:
# Doing: Security Patch
# Doing: Update Docs
# Doing: Backup
```

---

## 13.4 DevOps System Design: Worker Pools

Standard pattern for scaling: **Producers** and **Consumers**.

1.  **Web Server (Producer):** Receives user request. Puts it in a **Queue** (Redis/RabbitMQ).
2.  **Worker Server (Consumer):** Reads from Queue. Processes it.

If load increases, the Queue grows. The Worker keeps chugging at a steady pace.
**Solution:** Auto-scale more Workers to drain the Queue faster.

---

## 13.5 The Laws of Queues

1.  **The Law of Fairness:** FIFO Queues are fair. Everyone waits their turn.
2.  **The Law of Performance:** Never use `list.pop(0)` for queues. Use `deque`.
3.  **The Law of Backpressure:** If the queue grows faster than you can pop, you will run out of RAM. (Implement limits).

---

## [!] Common Mistakes

### Error #1: Using List as Queue
```python
queue = []
queue.pop(0) # O(n) - Slow for large lists!
```
**Fix:** `from collections import deque`.

### Error #2: Stack Overflow
Recursion uses the internal Call Stack. Too deep = Crash.
A Stack Data Structure (on the heap) can hold millions of items without crashing.

---

## Practice Problems

### Problem 1: The Browser History
Implement a `History` class using a Stack.
`visit(url)` pushes to stack.
`back()` pops and returns the previous URL.

### Problem 2: The Printer Spooler
Implement a print queue using `deque`.
Add 5 documents.
Print them in FIFO order.

---

## Chapter Summary

-   **Stack (LIFO):** Undo operations, Function calls. Use `list`.
-   **Queue (FIFO):** Buffer for jobs/tasks. Use `deque`.
-   **Priority Queue:** sorting tasks by urgency. Use `heapq`.
-   **Architecture:** Decoupling systems using Queues is a core DevOps pattern.
