# Chapter 16: Graphs - The Network Map

> "Everything is connected."

## The Goal: Modeling Connections

Trees have a strict hierarchy (Parent -> Child).
Real networks are messy.
-   Server A talks to Server B.
-   Server B talks to Server A.
-   Service C depends on Service D and E.
This is a **Graph**.

---

## 16.1 What is a Graph?

1.  **Vertices (Nodes):** The things (Servers, Cities, Tasks).
2.  **Edges (Links):** The connections (Cables, Roads, Dependencies).

**Types:**
-   **Undirected:** A cable connects A and B. (A <-> B).
-   **Directed (Digraph):** Traffic flows one way. (A -> B).
-   **Weighted:** Some links are "heavier" (Latency, Cost).

---

## 16.2 Representing Graphs (The Adjacency List)

The best way to store a graph in Python is a Dictionary of Lists (or Sets).

```python
network = {
    "WebLB": ["Web01", "Web02"],
    "Web01": ["DB01", "Cache01"],
    "Web02": ["DB01"],
    "DB01":  [],
    "Cache01": []
}
```
**Meaning:** `WebLB` is connected to `Web01` and `Web02`.

---

## 16.3 Graph Traversal (BFS vs DFS)

How do you find if `WebLB` can reach `DB01`?

### Breadth-First Search (BFS) - The Wave
Scan all neighbors, then neighbors' neighbors.
**Use Case:** Finding the **Shortest Path** (Fewest hops).

```python
def bfs(graph, start, target):
    queue = [start]
    visited = set()
    
    while queue:
        current = queue.pop(0) # FIFO
        if current == target:
            return True
        
        visited.add(current)
        for neighbor in graph[current]:
            if neighbor not in visited:
                queue.append(neighbor)
    return False
```

### Depth-First Search (DFS) - The Maze Runner
Go as deep as possible, then backtrack.
**Use Case:** Solving puzzles, detecting loops.

```python
def dfs(graph, current, visited=None):
    if visited is None: visited = set()
    print(f"Visiting: {current}")
    visited.add(current)
    
    for neighbor in graph[current]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
```

---

## 16.4 DevOps Use Case: Dependency Resolution

**Scenario:** You want to deploy "App A".
"App A" needs "Lib B". "Lib B" needs "Lib C".
This is a **Directed Acyclic Graph (DAG)**.
Tools like **Terraform**, **Docker Compose**, and **Systemd** use graphs to determine startup order.

**Topological Sort:**
An algorithm that flattens the graph into a linear list (Order of operations).
1.  Find node with no dependencies (Lib C).
2.  Remove it.
3.  Repeat.
Result: C -> B -> A.

---

## 16.5 The Shortest Path (Dijkstra)

If edges have weights (Latency in ms), BFS is not enough.
**Dijkstra's Algorithm** finds the fastest path, not just fewest hops.
It prioritizes exploring low-cost edges first (using a Priority Queue).

---

## 16.6 The Laws of Graphs

1.  **The Law of Connectivity:** Not all nodes are reachable. Islands exist.
2.  **The Law of Cycles:** Infinite loops are real. Always use a `visited` set to track where you've been.
3.  **The Law of Complexity:** Graph algorithms can be computationally expensive (O(V + E)).

---

## [!] Common Mistakes

### Error #1: Getting Stuck in a Loop
```python
A -> B -> A
```
If you don't check `if neighbor in visited`, your BFS/DFS will run forever.

### Error #2: Assuming Direction
In a directed graph, if A -> B, it **does not** mean B can talk to A. (Firewall Rules!).

---

## Practice Problems

### Problem 1: The Social Network
Create a graph representing friends.
Find if "Alice" is connected to "Kevin" within 3 degrees of separation (Degrees = Hops).

### Problem 2: The Critical Path
Given a list of tasks and dependencies.
`{"Build": ["Lint"], "Deploy": ["Build"], "Lint": []}`
Print the order they must run.

---

## Final Project: The Infrastructure Mapper
Write a script that takes a list of server connections and builds a Graph dictionary.
Then, write a function `ping_all(start_node)` that uses BFS to print every reachable server from the Gateway.

---

## Book Summary

You made it.
You started with variables and loops.
You built scripts with files and error handling.
You modeled systems with OOP.
You organized data with Trees and Graphs.

**You are no longer a Scripter.**
**You are a Developer.**

Go build something amazing.
