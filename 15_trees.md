# Chapter 15: Trees - Hierarchical Data

> "Roots, Trunks, Branches, Leaves. Nature knows best."

## The Goal: Structured Data

Lists are flat.
Real life is nested.
-   **Filesystems:** `/` -> `home` -> `user` -> `docs`.
-   **HTML/XML:** `<body>` -> `<div>` -> `<p>`.
-   **JSON/YAML:** `config` -> `network` -> `interfaces`.

These are **Trees**.

---

## 15.1 The Value of Hierarchies

A Tree is just a Node (Root) that points to multiple Children.
Each Child is also a Tree (Subtree).

```python
class TreeNode:
    def __init__(self, name):
        self.name = name
        self.children = [] # List of Nodes

    def add(self, child_node):
        self.children.append(child_node)

    def __repr__(self):
        return f"Dir({self.name})"
```

---

## 15.2 Building a Directory Tree

Let's model a Linux filesystem.

```python
root = TreeNode("/")
home = TreeNode("home")
bin_dir = TreeNode("bin")

root.add(home)
root.add(bin_dir)

user = TreeNode("ubuntu")
home.add(user)

# Structure:
# /
# ├── home
# │   └── ubuntu
# └── bin
```

---

## 15.3 Traversing a Tree (Recursion Again!)

To print the whole tree, we visit the Node, then **Recursively** visit its children.

```python
def print_tree(node, level=0):
    indent = "  " * level
    print(f"{indent}- {node.name}")
    
    for child in node.children:
        print_tree(child, level + 1)

print_tree(root)
# - /
#   - home
#     - ubuntu
#   - bin
```

**DevOps Use Case:** This is exactly how `tree` command or `ls -R` works.

---

## 15.4 Binary Search Trees (BST)

A special tree where:
-   Left Child < Parent
-   Right Child > Parent

**Why?**
Searching is O(log n).
Is 50 > 10? Go Right.
Is 50 < 100? Go Left.
found.

**Database Indexes:** Databases use B-Trees (Balanced Trees) to find "User ID 500" instantly without scanning the whole table.

---

## 15.5 Parsing JSON config (The Real Tree)

When you load a JSON file, Python creates a Tree of Dictionaries/Lists.

```json
{
    "server": {
        "ports": [80, 443],
        "admin": {
            "name": "admin",
            "email": "root@localhost"
        }
    }
}
```

**Searching unique keys in JSON:**
You can write a recursive function to find *any* key named "email", no matter how deep it is nested.

```python
def find_key(data, target):
    if isinstance(data, dict):
        for key, value in data.items():
            if key == target:
                print(f"Found: {value}")
            find_key(value, target) # Recurse!
    
    elif isinstance(data, list):
        for item in data:
            find_key(item, target) # Recurse!

# find_key(config, "email")
```

---

## 15.6 The Laws of Trees

1.  **The Law of Roots:** Every tree has exactly one Root.
2.  **The Law of Leaves:** Nodes with no children are Leaves.
3.  **The Law of Depth:** Deep trees risk Recursion Limits.
4.  **The Law of Balance:** An unbalanced tree (a line) is just a Linked List (slow).

---

## [!] Common Mistakes

### Error #1: Cycles (Not a Tree)
If a child points back to the parent, it's not a Tree anymore. It's a **Graph**.
Trees must be **Acyclic**.

### Error #2: Modifying while Traversing
Deleting nodes while recursing through them is dangerous.
Collect "nodes to delete" in a list, then delete them after the loop.

---

## Practice Problems

### Problem 1: The File Finder
Write a function that takes a `TreeNode` (directory) and searches for a file named "passwords.txt".

### Problem 2: Depth Calculator
Write a function `get_max_depth(node)` that returns how many layers deep the tree goes.

---

## Chapter Summary

-   **Tree:** Hierarchical Nodes (Root -> Branches -> Leaves).
-   **Recursion:** The standard way to walk trees.
-   **DOM/JSON:** Real-world tree examples.
-   **Databases:** Use Trees for fast indexing.
