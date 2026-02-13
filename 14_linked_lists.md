# Chapter 14: Linked Lists - Chains of Custody

> "Strong links make a strong chain."

## The Goal: Dynamic Structures

Array (List) in memory is a solid block.
`[1][2][3]`
If you want to insert a new item at the beginning `[0][1][2][3]`, you have to shift EVERYTHING.
This is O(n).

A **Linked List** is different.
Items can be anywhere in RAM. Each item points to the next one.
`[1]->[2]->[3]`
To insert `[0]`, you just change one pointer.
`[0]->[1]->[2]->[3]`
This is O(1).

---

## 14.1 The Node (The Building Block)

A Linked List is made of **Nodes**.
Each Node has:
1.  **Data** (The content).
2.  **Next** (Pointer to the next node).

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None  # Initially points to nothing

    def __repr__(self):
        return f"Node({self.data})"
```

---

## 14.2 Building the Chain

```python
# Create independent nodes
head = Node("Access Log")
middle = Node("Error Log")
tail = Node("Debug Log")

# Link them together
head.next = middle
middle.next = tail

# The Chain: Access -> Error -> Debug -> None
```

### Traversing the List
To read the list, you start at the Head and follow the pointers until you hit `None`.

```python
current = head

while current:
    print(current.data)
    current = current.next
```

---

## 14.3 DevOps Use Case: The Blockchain (Log Integrity)

Why uses Connected Lists in real life?
**Filesystem Inodes:** Files are often fragmented on disk. One block points to the next.
**Git Commits:** Each commit points to its parent.
**Blockchain:** Each block includes the Hash of the previous block.

```python
import hashlib

class Block:
    def __init__(self, data, previous_hash):
        self.data = data
        self.previous_hash = previous_hash
        self.hash = self.calculate_hash()

    def calculate_hash(self):
        # Unique ID based on data + parent
        content = f"{self.data}{self.previous_hash}".encode()
        return hashlib.sha256(content).hexdigest()

# Genesis Block
genesis = Block("Initial Commit", "0")
block1 = Block("Added Login", genesis.hash)
block2 = Block("Fixed Bug", block1.hash)

print(f"Block 2 points to: {block2.previous_hash}")
print(f"Block 1 Hash:    {block1.hash}")
# They Match! The chain is unbroken.
```

If a hacker changes `block1.data`, `block1.hash` changes.
Then `block2.previous_hash` won't match. **The chain breaks.**
This is how **Git** ensures history integrity.

---

## 14.4 Python's Reality Check

Does Python use Linked Lists?
**No.** Python's `list` is a **Dynamic Array** (C Array).
-   Access `list[5]` is O(1).
-   Linked List `node.next.next...` is O(n).

You rarely implement Linked Lists manually in Python unless strictly for an algorithm interview or a very specific data structure (like a Graph).

---

## 14.5 The Laws of Linked Lists

1.  **The Law of Pointers:** If you lose the `head` pointer, you lose the whole list (Garbage Collection eats it).
2.  **The Law of Insertion:** Inserting at the beginning is O(1). (Lists are O(n)).
3.  **The Law of Access:** Finding the 100th item is O(n). (Lists are O(1)).

---

## [!] Common Mistakes

### Error #1: Infinite Loops
```python
node.next = node # Points to itself
```
If you traverse this, `while current:` never ends. Cycle Detection is a classic problem.

### Error #2: Breaking the Chain
To remove `B` from `A->B->C`.
**Correct:** `A.next = C`
**Incorrect:** Just deleting `B` without updating `A`. (A still points to dead memory or keeps B alive).

---

## Practice Problem: The Reversal

Write a function `reverse_list(head)` that takes a Linked List `A->B->C` and turns it into `C->B->A`.
(Hint: You need 3 pointers: `prev`, `curr`, `next`).

---

## Chapter Summary

-   **Node:** Data + Pointer.
-   **Linked List:** Chain of nodes.
-   **Pros:** Fast insertions.
-   **Cons:** Slow lookups.
-   **Real World:** Git History, Blockchains, Filesystems.
