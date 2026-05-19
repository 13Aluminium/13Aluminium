# Brain Corp Interview — Python, DS, OS & DBMS Cheat Sheet
### Quick-Reference for Ayush | Thursday May 21, 2026

---

## PART 1: PYTHON FUNDAMENTALS

### Data Types & Mutability

| Type | Mutable? | Example | Notes |
|------|----------|---------|-------|
| int, float, bool | No | `x = 5` | Immutable — reassignment creates new object |
| str | No | `s = "hello"` | Immutable — `s[0] = 'H'` raises TypeError |
| tuple | No | `t = (1, 2, 3)` | Hashable (can be dict key), faster than list |
| list | Yes | `a = [1, 2, 3]` | Dynamic array under the hood |
| dict | Yes | `d = {"a": 1}` | Hash table, O(1) avg lookup, insertion-ordered (3.7+) |
| set | Yes | `s = {1, 2, 3}` | Hash table, O(1) avg membership test, no duplicates |
| frozenset | No | `fs = frozenset({1,2})` | Immutable set, can be dict key |

### Memory Model — Everything is an Object

```python
a = [1, 2, 3]
b = a          # b points to SAME object as a
b.append(4)    # a is now [1, 2, 3, 4] — both changed!

c = a[:]       # SHALLOW COPY — new list, but elements are same refs
d = copy.deepcopy(a)  # DEEP COPY — fully independent

# id() gives memory address, is checks identity, == checks value
x = [1, 2]
y = [1, 2]
x == y   # True  (same value)
x is y   # False (different objects)
```

**Why this matters:** If they ask you to modify a list in a function, know whether you're mutating in-place or creating a new one.

### Passing: "Pass by Object Reference"

```python
def modify(lst):
    lst.append(4)     # Mutates the original — caller sees change
    lst = [99, 100]   # Rebinds LOCAL variable — caller does NOT see this

a = [1, 2, 3]
modify(a)
print(a)  # [1, 2, 3, 4] — append worked, reassignment didn't
```

Python doesn't do pass-by-value or pass-by-reference. It passes the reference by value. Mutable objects can be mutated through the reference. Reassigning the parameter only changes the local binding.

### List vs Tuple vs Set vs Dict — When to Use

- **List:** Ordered collection, need indexing, may have duplicates, will modify
- **Tuple:** Fixed collection, won't change, need as dict key, returning multiple values from function
- **Set:** Need fast membership testing, need to remove duplicates, set operations (union, intersection)
- **Dict:** Key-value mapping, fast lookup by key, counting (Counter), grouping

### List Comprehensions & Generators

```python
# List comp — builds entire list in memory
squares = [x**2 for x in range(1000000)]  # 1M items in memory

# Generator — lazy, yields one at a time, O(1) memory
squares_gen = (x**2 for x in range(1000000))  # Almost no memory

# Use generators when you're iterating once over large data
# Use lists when you need indexing or multiple passes

# Nested comprehension
matrix = [[1,2,3],[4,5,6],[7,8,9]]
flat = [x for row in matrix for x in row]  # [1,2,3,4,5,6,7,8,9]

# Dict comprehension
word_lengths = {w: len(w) for w in ["hello", "world"]}

# Conditional
evens = [x for x in range(20) if x % 2 == 0]
```

### *args and **kwargs

```python
def func(*args, **kwargs):
    # args is a tuple of positional arguments
    # kwargs is a dict of keyword arguments
    print(args)    # (1, 2, 3)
    print(kwargs)  # {'name': 'ayush', 'age': 24}

func(1, 2, 3, name="ayush", age=24)

# Unpacking
def add(a, b, c):
    return a + b + c

nums = [1, 2, 3]
add(*nums)  # Same as add(1, 2, 3)

config = {'a': 1, 'b': 2, 'c': 3}
add(**config)  # Same as add(a=1, b=2, c=3)
```

### Decorators

```python
def timer(func):
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time()-start:.3f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)

# @timer is syntactic sugar for: slow_function = timer(slow_function)
```

**When they come up:** "How would you benchmark different algorithms?" → decorator-based timing. Also: `@staticmethod`, `@classmethod`, `@property`.

### Context Managers

```python
# The 'with' statement — guarantees cleanup (closing files, releasing locks)
with open("data.txt", "r") as f:
    data = f.read()
# f is automatically closed here, even if an exception occurred

# Custom context manager
class Timer:
    def __enter__(self):
        self.start = time.time()
        return self
    def __exit__(self, *args):
        print(f"Elapsed: {time.time()-self.start:.3f}s")

with Timer():
    do_something()
```

### Exception Handling

```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"Bad value: {e}")
except (TypeError, KeyError) as e:
    print(f"Type or key error: {e}")
except Exception as e:
    print(f"Unexpected: {e}")  # Catch-all (avoid in production)
else:
    print("No exception occurred")  # Runs only if try succeeded
finally:
    cleanup()  # ALWAYS runs — even if exception or return
```

### GIL (Global Interpreter Lock)

- CPython has a GIL — only one thread executes Python bytecode at a time
- **Threading** is still useful for I/O-bound tasks (network, file reads) because the GIL is released during I/O
- **Multiprocessing** bypasses the GIL — separate processes, separate memory, true parallelism for CPU-bound work
- **Why it matters for Brain Corp:** Robot perception pipelines are CPU-heavy. You'd use multiprocessing or C++ extensions, not threading, for parallel image processing

```python
# Threading — good for I/O
from concurrent.futures import ThreadPoolExecutor
with ThreadPoolExecutor(max_workers=4) as pool:
    results = pool.map(fetch_url, urls)

# Multiprocessing — good for CPU
from concurrent.futures import ProcessPoolExecutor
with ProcessPoolExecutor(max_workers=4) as pool:
    results = pool.map(process_image, images)
```

### Common Gotchas They Might Ask

```python
# 1. Mutable default argument
def bad(lst=[]):       # SAME list shared across calls!
    lst.append(1)
    return lst
bad()  # [1]
bad()  # [1, 1]  ← Bug!

def good(lst=None):    # Correct pattern
    if lst is None:
        lst = []
    lst.append(1)
    return lst

# 2. Late binding closures
funcs = [lambda x: x + i for i in range(3)]
funcs[0](10)  # 12, not 10! — i is looked up at call time, not creation time

# Fix: capture i as default arg
funcs = [lambda x, i=i: x + i for i in range(3)]
funcs[0](10)  # 10 ✓

# 3. String concatenation in a loop
# Bad: O(n²) because strings are immutable
s = ""
for word in words:
    s += word  # Creates new string each time

# Good: O(n)
s = "".join(words)

# 4. Checking empty collections
if len(lst) == 0:   # Works but not Pythonic
if not lst:          # Pythonic — empty collections are falsy
```

---

## PART 2: DATA STRUCTURES & ALGORITHMS

### Time Complexity Quick Reference

| Operation | list | dict | set | heapq | deque |
|-----------|------|------|-----|-------|-------|
| Access by index | O(1) | — | — | — | O(n) |
| Search | O(n) | O(1) avg | O(1) avg | O(n) | O(n) |
| Insert at end | O(1)* | O(1) avg | O(1) avg | O(log n) | O(1) |
| Insert at front | O(n) | — | — | — | O(1) |
| Delete | O(n) | O(1) avg | O(1) avg | O(log n) | O(1) ends |
| Sort | O(n log n) | — | — | — | — |

*amortized

### Arrays / Lists

```python
# Sliding window — fixed size k
def max_sum_subarray(arr, k):
    window = sum(arr[:k])
    best = window
    for i in range(k, len(arr)):
        window += arr[i] - arr[i-k]
        best = max(best, window)
    return best

# Two pointers — sorted array, find pair with target sum
def two_sum_sorted(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo < hi:
        s = arr[lo] + arr[hi]
        if s == target: return (lo, hi)
        elif s < target: lo += 1
        else: hi -= 1
    return None

# Prefix sum — range sum queries in O(1) after O(n) setup
def prefix_sums(arr):
    prefix = [0] * (len(arr) + 1)
    for i in range(len(arr)):
        prefix[i+1] = prefix[i] + arr[i]
    return prefix
# sum(arr[l:r+1]) == prefix[r+1] - prefix[l]
```

### Hash Maps (dict) — The Most Important DS

```python
from collections import Counter, defaultdict

# Counting occurrences
counts = Counter([1, 2, 2, 3, 3, 3])  # {3: 3, 2: 2, 1: 1}
counts.most_common(2)  # [(3, 3), (2, 2)]

# Grouping
groups = defaultdict(list)
for item in items:
    groups[item.category].append(item)

# Two Sum (unsorted) — classic interview problem
def two_sum(nums, target):
    seen = {}
    for i, n in enumerate(nums):
        comp = target - n
        if comp in seen:
            return [seen[comp], i]
        seen[n] = i
```

### Stacks & Queues

```python
from collections import deque

# Stack (LIFO) — use list
stack = []
stack.append(1)    # push
stack.pop()        # pop from top

# Queue (FIFO) — use deque (O(1) both ends)
queue = deque()
queue.append(1)    # enqueue at right
queue.popleft()    # dequeue from left
# NEVER use list as queue — list.pop(0) is O(n)

# Monotonic stack — find next greater element
def next_greater(nums):
    result = [-1] * len(nums)
    stack = []  # stores indices
    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            result[stack.pop()] = num
        stack.append(i)
    return result

# Valid parentheses
def is_valid(s):
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}
    for c in s:
        if c in '([{':
            stack.append(c)
        elif not stack or stack.pop() != pairs[c]:
            return False
    return len(stack) == 0
```

### Heaps (Priority Queue)

```python
import heapq

# Python heapq is a MIN heap
nums = [3, 1, 4, 1, 5, 9]
heapq.heapify(nums)          # O(n) — turns list into heap in-place
heapq.heappush(nums, 2)      # O(log n)
smallest = heapq.heappop(nums)  # O(log n) — returns smallest

# For MAX heap: negate values
max_heap = [-x for x in nums]
heapq.heapify(max_heap)
largest = -heapq.heappop(max_heap)

# K largest/smallest
heapq.nlargest(3, nums)   # Top 3
heapq.nsmallest(3, nums)  # Bottom 3

# K closest points to origin — DIRECTLY relevant to Brain Corp (nearest obstacles)
def k_closest(points, k):
    heap = []
    for x, y in points:
        dist = x*x + y*y
        if len(heap) < k:
            heapq.heappush(heap, (-dist, x, y))  # max heap of size k
        elif -dist > heap[0][0]:
            heapq.heapreplace(heap, (-dist, x, y))
    return [(x, y) for _, x, y in heap]
```

### Trees

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# BFS level order — most useful in interviews
def level_order(root):
    if not root: return []
    result = []
    queue = deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left: queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level)
    return result

# DFS traversals
def inorder(root):    # Left, Root, Right — gives sorted order for BST
    if not root: return []
    return inorder(root.left) + [root.val] + inorder(root.right)

def preorder(root):   # Root, Left, Right — used for serialization
    if not root: return []
    return [root.val] + preorder(root.left) + preorder(root.right)

# Max depth
def max_depth(root):
    if not root: return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))
```

### Graphs — CRITICAL for Brain Corp (navigation, SLAM, path planning)

```python
from collections import deque, defaultdict

# Adjacency list representation
graph = defaultdict(list)
graph['A'].extend(['B', 'C'])
graph['B'].extend(['A', 'D'])

# BFS — shortest path in unweighted graph
def bfs(graph, start, target):
    queue = deque([(start, [start])])
    visited = {start}
    while queue:
        node, path = queue.popleft()
        if node == target:
            return path
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))
    return None

# DFS — iterative (avoids recursion limit)
def dfs(graph, start):
    visited = set()
    stack = [start]
    while stack:
        node = stack.pop()
        if node in visited:
            continue
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)
    return visited

# Topological sort — dependency ordering
def topo_sort(graph, num_nodes):
    in_degree = defaultdict(int)
    for node in graph:
        for neighbor in graph[node]:
            in_degree[neighbor] += 1
    
    queue = deque([n for n in range(num_nodes) if in_degree[n] == 0])
    order = []
    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
    return order if len(order) == num_nodes else []  # empty = cycle

# Dijkstra — shortest path in weighted graph
def dijkstra(graph, start):
    dist = {start: 0}
    heap = [(0, start)]
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist.get(u, float('inf')):
            continue
        for v, w in graph[u]:  # (neighbor, weight)
            new_dist = d + w
            if new_dist < dist.get(v, float('inf')):
                dist[v] = new_dist
                heapq.heappush(heap, (new_dist, v))
    return dist
```

### Sorting — Know the Internals

```python
# Python uses Timsort — hybrid merge sort + insertion sort, O(n log n), stable
# Stable = equal elements maintain original order

# Custom sorting
intervals = [(1,3), (2,1), (1,2)]
intervals.sort(key=lambda x: (x[0], x[1]))  # Sort by first, then second

# Sort by multiple criteria
students = [("Alice", 90), ("Bob", 85), ("Alice", 85)]
students.sort(key=lambda s: (-s[1], s[0]))  # Descending grade, ascending name

# Binary search — O(log n) on sorted data
import bisect
sorted_list = [1, 3, 5, 7, 9]
idx = bisect.bisect_left(sorted_list, 5)   # 2 — leftmost position
bisect.insort(sorted_list, 6)              # Insert maintaining order
```

### Recursion & Dynamic Programming

```python
# Fibonacci — classic DP example
# Naive recursion: O(2^n) — terrible
def fib_bad(n):
    if n <= 1: return n
    return fib_bad(n-1) + fib_bad(n-2)

# Memoization (top-down): O(n)
from functools import lru_cache
@lru_cache(maxsize=None)
def fib_memo(n):
    if n <= 1: return n
    return fib_memo(n-1) + fib_memo(n-2)

# Tabulation (bottom-up): O(n) time, O(1) space
def fib_dp(n):
    if n <= 1: return n
    a, b = 0, 1
    for _ in range(2, n+1):
        a, b = b, a + b
    return b

# When to use DP: overlapping subproblems + optimal substructure
# Common patterns: 0/1 knapsack, longest common subsequence, coin change
```

---

## PART 3: OPERATING SYSTEM CONCEPTS

### Processes vs Threads

| | Process | Thread |
|---|---------|--------|
| Memory | Separate address space | Shared address space within process |
| Creation cost | Expensive (fork) | Cheap |
| Communication | IPC (pipes, sockets, shared memory) | Direct shared memory |
| Crash isolation | One crash doesn't kill others | One crash can kill all threads in process |
| Python relevance | multiprocessing — bypasses GIL | threading — limited by GIL for CPU work |

**Brain Corp context:** A robot runs multiple processes — perception, planning, control, communication. Each might be a separate ROS node (process). Within a node, threads handle different sensor streams.

### Concurrency vs Parallelism

- **Concurrency:** Multiple tasks make progress (may be interleaved on one core). Like juggling.
- **Parallelism:** Multiple tasks literally execute at the same time on multiple cores. Like multiple jugglers.
- Python threading = concurrency (GIL). Python multiprocessing = parallelism.

### Synchronization Primitives

```
Mutex (Lock):     Only one thread can hold it. Protects shared data.
Semaphore:        Allows N threads to access a resource simultaneously (e.g., connection pool of size 10).
Condition Variable: Thread waits until a condition is met, another thread signals it.
Read-Write Lock:  Multiple readers OR one writer. Good when reads >> writes.
```

**Deadlock** — Four conditions (ALL must hold):
1. Mutual exclusion — resource can't be shared
2. Hold and wait — holding one resource, waiting for another
3. No preemption — can't forcibly take a resource
4. Circular wait — A waits for B, B waits for A

**Prevention:** Break any one condition. Common: always acquire locks in the same global order.

**Race Condition:** Two threads access shared data, at least one writes, no synchronization. Result depends on timing. Fix: use locks.

### Memory Management

**Stack vs Heap:**
- Stack: function call frames, local variables, automatic cleanup, fixed size (~8MB), fast, LIFO
- Heap: dynamically allocated objects (malloc/new), manual or GC cleanup, large, slower

**Python memory:** Everything lives on the heap. Python's memory manager handles allocation. Garbage collection uses reference counting (primary) + cyclic garbage collector (for reference cycles).

```python
import sys
a = [1, 2, 3]
sys.getrefcount(a)  # Shows reference count (includes the call's own reference)

# When refcount hits 0 → immediately freed
# Cyclic references (a→b→a) caught by gc module
import gc
gc.collect()  # Force garbage collection cycle
```

### Virtual Memory & Paging

- **Virtual memory:** Each process gets its own virtual address space (appears to have all memory to itself)
- **Page:** Fixed-size block (typically 4KB). Virtual pages map to physical frames.
- **Page table:** Maps virtual page numbers → physical frame numbers
- **Page fault:** Process accesses a page not in physical memory → OS loads it from disk (swap)
- **TLB (Translation Lookaside Buffer):** Cache for page table entries — makes address translation fast

### Scheduling Algorithms

| Algorithm | How It Works | Pros | Cons |
|-----------|-------------|------|------|
| FCFS | First come, first served | Simple | Convoy effect — short jobs wait behind long ones |
| SJF | Shortest job first | Optimal avg wait time | Starvation of long jobs, need to predict job length |
| Round Robin | Each process gets a time slice (quantum) | Fair, responsive | High context switch overhead if quantum too small |
| Priority | Higher priority runs first | Important tasks run first | Starvation — fix with aging |
| MLFQ | Multiple queues with different priorities | Balances responsiveness and throughput | Complex |

**Context switch:** Save current process state (registers, PC, stack pointer), load another's. Costs time — 1-10 microseconds typically.

### File Systems & I/O

- **Inode:** Metadata about a file (size, permissions, timestamps, pointers to data blocks). NOT the filename.
- **Directory:** Maps filenames → inode numbers
- **Hard link:** Another filename pointing to same inode. `ln file hardlink`
- **Symbolic link:** Separate file pointing to a path. `ln -s file symlink`. Can break if target deleted.
- **File descriptors:** Integer handles for open files. 0=stdin, 1=stdout, 2=stderr.

**Linux commands they might expect you to know** (you listed Linux in every resume):
```bash
ps aux          # List all processes
top / htop      # Monitor CPU/memory usage
kill -9 <pid>   # Force kill a process
grep -r "text" .   # Search recursively
find . -name "*.py"  # Find files
chmod 755 file     # Set permissions (rwxr-xr-x)
lsof -i :8080     # What process is using port 8080
df -h              # Disk usage
free -m            # Memory usage
nvidia-smi         # GPU status (relevant for Jetson/CUDA work)
```

### Interprocess Communication (IPC)

| Method | Use Case |
|--------|----------|
| Pipe | Simple one-way byte stream between parent-child |
| Named Pipe (FIFO) | Between unrelated processes |
| Shared Memory | Fastest IPC — processes map same physical memory. Need synchronization. |
| Message Queue | Structured messages, asynchronous |
| Socket | Network communication (also works local via Unix sockets) |
| Signal | Simple notifications (SIGTERM, SIGKILL) |

**Brain Corp / ROS context:** ROS nodes communicate via topics (publish/subscribe, like message queues), services (request/reply), and actions (long-running tasks). Under the hood, ROS 2 uses DDS (Data Distribution Service) middleware.

---

## PART 4: DATABASE CONCEPTS

### SQL Basics

```sql
-- SELECT with JOIN
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.id
WHERE e.salary > 50000
ORDER BY e.name
LIMIT 10;

-- Aggregation
SELECT dept_id, COUNT(*) as cnt, AVG(salary) as avg_sal
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 5;

-- Subquery
SELECT name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Window functions
SELECT name, salary,
       RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) as dept_rank
FROM employees;
```

### JOIN Types

```
INNER JOIN:  Only matching rows from both tables
LEFT JOIN:   All rows from left + matching from right (NULL if no match)
RIGHT JOIN:  All rows from right + matching from left
FULL OUTER:  All rows from both (NULL where no match)
CROSS JOIN:  Every row from A paired with every row from B (cartesian product)
```

### ACID Properties

| Property | Meaning | Example |
|----------|---------|---------|
| Atomicity | Transaction is all-or-nothing | Transfer $100: debit AND credit both succeed or both roll back |
| Consistency | DB moves from one valid state to another | Constraints (foreign keys, checks) are never violated |
| Isolation | Concurrent transactions don't interfere | Two transfers happening simultaneously produce correct results |
| Durability | Committed data survives crashes | Written to disk/WAL before confirming commit |

### Indexing

- **B-Tree index (default):** Balanced tree, O(log n) lookup. Good for range queries (`WHERE age > 25`) and equality.
- **Hash index:** O(1) lookup for exact equality only. No range queries.
- **Composite index:** Index on multiple columns `(a, b, c)`. Supports queries filtering on a, or a+b, or a+b+c (leftmost prefix).

```sql
CREATE INDEX idx_name ON employees(last_name, first_name);
-- Speeds up: WHERE last_name = 'Luhar'
-- Speeds up: WHERE last_name = 'Luhar' AND first_name = 'Ayush'
-- Does NOT help: WHERE first_name = 'Ayush' (not leftmost prefix)
```

**When to index:** Columns in WHERE, JOIN ON, ORDER BY that are queried frequently. Don't over-index — each index slows down writes (INSERT/UPDATE/DELETE).

### Normalization

| Form | Rule | Example Violation |
|------|------|-------------------|
| 1NF | No repeating groups, atomic values | A column containing "Python, C++, Java" as one string |
| 2NF | 1NF + no partial dependencies on composite key | In (student_id, course_id) → student_name depends only on student_id |
| 3NF | 2NF + no transitive dependencies | dept_id → dept_name through employee table instead of dept table |

**Denormalization:** Intentionally breaking normalization for read performance. Common in analytics/data warehouses. Trade-off: faster reads, slower writes, data redundancy.

### SQL vs NoSQL

| | SQL (Relational) | NoSQL |
|---|---|---|
| Schema | Fixed, predefined | Flexible, schema-less |
| Scaling | Vertical (bigger machine) | Horizontal (more machines) |
| Joins | Yes, powerful | Limited or none |
| ACID | Yes | Usually eventual consistency (BASE) |
| Best for | Structured data, complex queries, transactions | High volume, flexible schema, real-time, distributed |
| Examples | PostgreSQL, MySQL, SQLite | MongoDB (document), Redis (key-value), Cassandra (column), Neo4j (graph) |

**Brain Corp context:** Robot telemetry data (thousands of robots reporting position, status, sensor data) might use a time-series database or NoSQL for ingestion at scale. Configuration and fleet management likely uses relational.

### CAP Theorem (for distributed systems)

You can only guarantee 2 of 3:
- **Consistency:** Every read gets the most recent write
- **Availability:** Every request gets a response
- **Partition tolerance:** System works despite network splits

In practice, partitions happen, so you choose CP (consistent but may be unavailable during partition) or AP (available but may return stale data).

### Transactions & Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|-------------------|-------------|
| Read Uncommitted | Yes | Yes | Yes |
| Read Committed | No | Yes | Yes |
| Repeatable Read | No | No | Yes |
| Serializable | No | No | No |

- **Dirty read:** Reading uncommitted data from another transaction
- **Non-repeatable read:** Same query returns different results within one transaction
- **Phantom read:** New rows appear between two identical queries in one transaction

---

## PART 5: QUICK-FIRE ANSWERS

**"What happens when you type `python script.py`?"**
> Shell finds Python interpreter → interpreter reads .py file → lexer tokenizes → parser builds AST → compiler generates bytecode (.pyc) → PVM (Python Virtual Machine) executes bytecode line by line.

**"What's the difference between `__init__` and `__new__`?"**
> `__new__` creates the instance (allocates memory), `__init__` initializes it (sets attributes). `__new__` is called first. You rarely override `__new__` unless doing singleton pattern or immutable types.

**"What are Python's magic/dunder methods?"**
> Methods surrounded by double underscores that define object behavior: `__init__` (constructor), `__str__` (print), `__repr__` (debug), `__len__` (len()), `__eq__` (==), `__lt__` (<), `__hash__` (hashing), `__iter__`/`__next__` (iteration), `__enter__`/`__exit__` (context manager).

**"Explain `yield` and generators"**
> `yield` pauses a function and returns a value. Next call resumes from where it left off. The function becomes a generator — produces values lazily, one at a time, without storing all in memory. Perfect for large datasets or infinite sequences.

**"What is a deadlock? How would you detect one in your robot code?"**
> Two or more processes/threads each holding a resource the other needs. In a robot: perception thread holds camera lock waiting for planner lock, planner holds planner lock waiting for camera lock. Detection: timeout on lock acquisition, or build a resource allocation graph and check for cycles.

**"Process vs thread for robot perception?"**
> Use separate processes for independent subsystems (perception, planning, control) — crash isolation, true parallelism. Use threads within a process for lightweight tasks sharing memory (e.g., multiple camera streams feeding the same perception node). In Python specifically, use multiprocessing for CPU-bound work due to GIL.

**"How does a robot handle real-time constraints?"**
> Real-time doesn't mean fast — it means deterministic and predictable timing. Hard real-time (motor control) uses C/C++ with RTOS or RT-PREEMPT Linux. Soft real-time (perception, planning) can use Python with careful pipeline design — fixed-rate loops, async I/O, dropping frames if behind schedule rather than queuing up latency.
