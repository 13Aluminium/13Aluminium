# Brain Corp Interview — Python, DS, OS & DBMS Cheat Sheet (Expanded)
### Quick-Reference for Ayush | Thursday May 21, 2026

---

## TABLE OF CONTENTS
1. [Python Fundamentals](#part-1-python-fundamentals)
2. [Data Structures & Algorithms](#part-2-data-structures--algorithms)
3. [Operating System Concepts](#part-3-operating-system-concepts)
4. [Database Concepts](#part-4-database-concepts)
5. [OOP & Design Patterns](#part-5-oop--design-patterns)
6. [Quick-Fire Answers](#part-6-quick-fire-answers)

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
| bytes | No | `b = b"hello"` | Immutable byte sequence — used in network/file I/O |
| bytearray | Yes | `ba = bytearray(b"hi")` | Mutable byte sequence |

**Internals of list:** CPython implements `list` as a dynamic array (like C++ `vector`). Append is O(1) amortized — when the buffer fills, Python allocates a new buffer ~1.125× the current size and copies everything. This is why `list.append()` is fast but `list.insert(0, x)` is O(n) — it shifts every element.

**Internals of dict:** Python 3.6+ dicts are compact ordered hash tables. Keys are hashed to a slot. Collision is resolved by probing. Load factor is kept ≤ 2/3; beyond that, the table is resized (doubled). Lookup is O(1) average, O(n) worst case (all hashes collide — essentially impossible with good hash functions).

### Memory Model — Everything is an Object

```python
a = [1, 2, 3]
b = a          # b points to SAME object as a
b.append(4)    # a is now [1, 2, 3, 4] — both changed!

c = a[:]                  # SHALLOW COPY — new list, but elements are same refs
import copy
d = copy.deepcopy(a)      # DEEP COPY — fully independent tree of objects

# id() gives memory address; 'is' checks identity; '==' checks value
x = [1, 2]
y = [1, 2]
x == y   # True  (same value)
x is y   # False (different objects in memory)

# Small integer caching (-5 to 256) — CPython optimization
a = 100; b = 100; a is b  # True  (same cached object)
a = 300; b = 300; a is b  # False (different objects — outside cache range)

# String interning — short strings that look like identifiers are often interned
s1 = "hello"; s2 = "hello"; s1 is s2  # Often True
s1 = "hello world"; s2 = "hello world"; s1 is s2  # Often False
```

**Why this matters in interviews:** "Are two objects the same?" vs "Do they have the same value?" The `is` vs `==` distinction trips up many candidates. Also — if they ask you to modify a list inside a function, know whether you're mutating in-place or creating a new one.

### Passing: "Pass by Object Reference"

```python
def modify(lst):
    lst.append(4)     # Mutates the original — caller sees change
    lst = [99, 100]   # Rebinds LOCAL variable — caller does NOT see this

a = [1, 2, 3]
modify(a)
print(a)  # [1, 2, 3, 4] — append worked, reassignment didn't

# Why? Python passes a copy of the REFERENCE, not the object itself.
# lst.append(4) follows the reference and mutates the object.
# lst = [99, 100] makes 'lst' point to a NEW object; the original reference 'a' is unchanged.
```

**Analogy:** You give someone a copy of your house address (not your house). They can go to that address and rearrange the furniture (mutation). But if they write a new address on that paper, your house doesn't move.

### List vs Tuple vs Set vs Dict — When to Use

- **List:** Ordered collection, need indexing, may have duplicates, will modify. Use when order matters and you'll change the collection.
- **Tuple:** Fixed collection, won't change, need as dict key, returning multiple values from function. Use when data is read-only — also ~30% faster than list for iteration.
- **Set:** Need fast membership testing O(1), need to remove duplicates, set operations (`union`, `intersection`, `difference`). Unordered, no duplicates.
- **Dict:** Key-value mapping, fast lookup by key, counting (`Counter`), grouping. Use `defaultdict` to avoid `KeyError` on missing keys.

```python
# Set operations — extremely useful for intersection/union problems
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}
a & b   # {3, 4}      — intersection
a | b   # {1,2,3,4,5,6} — union
a - b   # {1, 2}      — difference (in a but not b)
a ^ b   # {1, 2, 5, 6} — symmetric difference

# Counter — fast frequency counting
from collections import Counter
words = ["apple", "banana", "apple", "cherry", "banana", "apple"]
c = Counter(words)   # Counter({'apple': 3, 'banana': 2, 'cherry': 1})
c.most_common(2)     # [('apple', 3), ('banana', 2)]
c['apple'] += 1      # You can increment counts directly
c.subtract(["apple"]) # Subtract counts

# OrderedDict — before Python 3.7 (now all dicts preserve order)
from collections import OrderedDict
od = OrderedDict()
od.move_to_end('key')   # Useful for LRU cache implementation
```

### List Comprehensions & Generators

```python
# List comp — builds entire list in memory
squares = [x**2 for x in range(1000000)]   # 1M items in memory (~8MB)

# Generator — lazy, yields one at a time, O(1) memory
squares_gen = (x**2 for x in range(1000000))  # Almost no memory used

# Use generators when you're iterating once over large data
# Use lists when you need indexing, len(), or multiple passes

# Nested comprehension
matrix = [[1,2,3],[4,5,6],[7,8,9]]
flat = [x for row in matrix for x in row]  # [1,2,3,4,5,6,7,8,9]

# Comprehension with condition
evens = [x for x in range(20) if x % 2 == 0]

# Dict and set comprehensions
word_lengths = {w: len(w) for w in ["hello", "world"]}   # {'hello': 5, 'world': 5}
unique_lengths = {len(w) for w in ["hi", "hello", "hey"]}  # {2, 5, 3}

# Generator function with yield
def chunked(lst, n):
    """Yield successive n-sized chunks from lst."""
    for i in range(0, len(lst), n):
        yield lst[i:i + n]

for chunk in chunked(range(10), 3):
    print(list(chunk))  # [0,1,2], [3,4,5], [6,7,8], [9]
```

### `*args` and `**kwargs`

```python
def func(*args, **kwargs):
    # args is a tuple of positional arguments
    # kwargs is a dict of keyword arguments
    print(args)    # (1, 2, 3)
    print(kwargs)  # {'name': 'ayush', 'age': 24}

func(1, 2, 3, name="ayush", age=24)

# Keyword-only arguments (after *)
def strict(a, b, *, verbose=False):
    # verbose can ONLY be passed by keyword
    pass

strict(1, 2, verbose=True)    # OK
strict(1, 2, True)            # TypeError

# Positional-only arguments (before /)  — Python 3.8+
def pos_only(a, b, /, c=0):
    # a and b can ONLY be passed positionally
    pass

# Unpacking
def add(a, b, c): return a + b + c
nums = [1, 2, 3]
add(*nums)               # Same as add(1, 2, 3)
config = {'a': 1, 'b': 2, 'c': 3}
add(**config)            # Same as add(a=1, b=2, c=3)

# Merging dicts (Python 3.5+)
merged = {**dict1, **dict2}       # dict2 values win on conflict
merged = dict1 | dict2            # Python 3.9+ syntax
```

### Decorators — Deep Dive

```python
import time
import functools

def timer(func):
    @functools.wraps(func)   # Preserves __name__, __doc__ etc.
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(0.1)

# @timer is exactly equivalent to: slow_function = timer(slow_function)
# Without @functools.wraps, slow_function.__name__ would be 'wrapper'

# Decorator with arguments — requires an extra layer
def retry(max_attempts=3, exceptions=(Exception,)):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt+1} failed: {e}")
        return wrapper
    return decorator

@retry(max_attempts=5, exceptions=(ConnectionError,))
def fetch_data(url):
    pass

# Class-based decorator
class memoize:
    def __init__(self, func):
        self.func = func
        self.cache = {}
        functools.update_wrapper(self, func)
    
    def __call__(self, *args):
        if args not in self.cache:
            self.cache[args] = self.func(*args)
        return self.cache[args]

@memoize
def expensive(n): return n * 2

# Built-in decorators
class MyClass:
    class_var = 0
    
    @staticmethod
    def helper(x):          # No access to class or instance
        return x * 2
    
    @classmethod
    def from_string(cls, s): # Gets class as first arg (cls), not instance
        return cls()         # Can create instances of subclasses correctly
    
    @property
    def value(self):         # Accessed as obj.value, not obj.value()
        return self._value
    
    @value.setter
    def value(self, v):      # obj.value = 5 triggers this
        if v < 0: raise ValueError("Value must be non-negative")
        self._value = v
```

### Context Managers — Full Picture

```python
# The 'with' statement — guarantees cleanup even if an exception occurs
with open("data.txt", "r") as f:
    data = f.read()
# f is automatically closed here, even if an exception occurred

# Custom context manager — class-based
class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self   # The value bound to 'as' variable
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        elapsed = time.perf_counter() - self.start
        print(f"Elapsed: {elapsed:.4f}s")
        return False  # False = don't suppress exceptions; True = suppress them

with Timer() as t:
    do_something()

# Context manager using contextlib — simpler
from contextlib import contextmanager

@contextmanager
def managed_resource():
    resource = acquire()    # Setup
    try:
        yield resource      # The 'as' value
    finally:
        release(resource)   # Teardown (runs even on exception)

# Multiple context managers
with open("in.txt") as fin, open("out.txt", "w") as fout:
    fout.write(fin.read())

# suppress — silently ignore specific exceptions
from contextlib import suppress
with suppress(FileNotFoundError):
    os.remove("might_not_exist.txt")
```

### Exception Handling — Complete Pattern

```python
try:
    result = risky_operation()
except ValueError as e:
    print(f"Bad value: {e}")
except (TypeError, KeyError) as e:
    print(f"Type or key error: {e}")
except Exception as e:
    print(f"Unexpected: {e}")   # Catch-all (avoid in production)
    raise                        # Re-raise the same exception
else:
    print("No exception occurred")  # Runs ONLY if try succeeded — rarely used but clean
finally:
    cleanup()    # ALWAYS runs — even if exception or return

# Custom exceptions — always inherit from Exception
class RobotNavigationError(Exception):
    def __init__(self, message, position=None):
        super().__init__(message)
        self.position = position

raise RobotNavigationError("Obstacle detected", position=(1.5, 2.3))

# Exception chaining — preserve original cause
try:
    raw_data = parse_sensor()
except ValueError as e:
    raise RobotNavigationError("Sensor parse failed") from e  # __cause__ set

# Exception groups (Python 3.11+)
try:
    async with asyncio.TaskGroup() as tg:
        tg.create_task(task1())
        tg.create_task(task2())
except* ValueError as eg:
    for exc in eg.exceptions:
        print(exc)
```

### GIL (Global Interpreter Lock) — Deep Dive

- **What it is:** A mutex in CPython that ensures only one thread executes Python bytecode at a time. It protects Python's internal state (reference counts, memory allocator).
- **Why it exists:** CPython's reference counting garbage collector is not thread-safe without it. Removing the GIL while keeping reference counting would require fine-grained locks everywhere — historically too slow.
- **Python 3.13+ (PEP 703):** Free-threaded CPython is available experimentally (`--disable-gil`). The ecosystem is adapting.

```python
# Threading — good for I/O-bound tasks (GIL released during I/O syscalls)
from concurrent.futures import ThreadPoolExecutor
import urllib.request

def fetch(url):
    with urllib.request.urlopen(url) as r:
        return r.read()

with ThreadPoolExecutor(max_workers=8) as pool:
    results = list(pool.map(fetch, urls))  # All fetching happens concurrently

# Multiprocessing — good for CPU-bound (separate processes, each with own GIL)
from concurrent.futures import ProcessPoolExecutor
import numpy as np

def process_image(img_array):
    return np.mean(img_array, axis=2)  # Grayscale conversion

with ProcessPoolExecutor(max_workers=4) as pool:
    results = list(pool.map(process_image, image_arrays))

# asyncio — cooperative multitasking on one thread (I/O bound)
import asyncio
import aiohttp

async def fetch_async(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_async(session, url) for url in urls]
        results = await asyncio.gather(*tasks)

# When to use what:
# I/O bound, many connections      → asyncio (single thread, event loop)
# I/O bound, simpler code          → ThreadPoolExecutor
# CPU bound, pure Python            → ProcessPoolExecutor
# CPU bound, numerical (numpy)     → numpy releases GIL, threading works fine
# CPU bound, C extensions          → they often release the GIL too
```

**Brain Corp relevance:** Robot sensor pipelines are CPU-heavy. Use `ProcessPoolExecutor` for parallel image processing or C/C++ extensions (which release the GIL). For sensor polling (I/O-bound), threading or asyncio is fine.

### Common Python Gotchas

```python
# 1. Mutable default argument — one of the most common interview traps
def bad(lst=[]):         # SAME list object shared across ALL calls!
    lst.append(1)
    return lst

bad()  # [1]
bad()  # [1, 1]  ← Bug! The default list persists between calls.

def good(lst=None):      # Correct pattern
    if lst is None:
        lst = []
    lst.append(1)
    return lst

# 2. Late binding closures
funcs = [lambda x: x + i for i in range(3)]
funcs[0](10)  # 12, not 10! — 'i' is looked up at call time, not creation time

# Fix: capture i as a default argument
funcs = [lambda x, i=i: x + i for i in range(3)]
funcs[0](10)  # 10 ✓

# 3. String concatenation in a loop — O(n²) because strings are immutable
# Bad:
s = ""
for word in words:
    s += word  # Creates a new string object each time

# Good: O(n)
s = "".join(words)

# 4. Checking empty collections
if len(lst) == 0:   # Works but not Pythonic
if not lst:          # Pythonic — empty collections are falsy
# Falsy: 0, 0.0, "", [], {}, set(), None, False

# 5. Integer division vs float division
7 / 2    # 3.5  (true division — always float in Python 3)
7 // 2   # 3    (floor division)
-7 // 2  # -4   (floors toward negative infinity, not toward zero!)
7 % 2    # 1
-7 % 2   # 1    (Python modulo always same sign as divisor)

# 6. Chained comparisons — actually work in Python
1 < x < 10  # True if x is between 1 and 10 (exclusive) — elegant!

# 7. Variable scope — LEGB rule
x = "global"
def outer():
    x = "enclosing"
    def inner():
        nonlocal x   # Without this, inner() can't rebind enclosing x
        x = "inner"
    inner()
    print(x)  # "inner" (because nonlocal was used)
```

### Type Hints & Dataclasses (Modern Python)

```python
from typing import Optional, List, Dict, Tuple, Union, Any
from typing import Callable, Generator, Iterator
from dataclasses import dataclass, field

# Type hints — not enforced at runtime, but great for documentation + mypy
def process(items: List[int], threshold: float = 0.5) -> Dict[str, int]:
    ...

# Optional is just Union[X, None]
def find(name: str) -> Optional[int]:
    return None

# Python 3.10+ — cleaner union syntax
def find(name: str) -> int | None:
    return None

# Dataclasses — less boilerplate than regular classes
@dataclass
class RobotPose:
    x: float
    y: float
    theta: float = 0.0
    timestamp: float = field(default_factory=float)   # mutable default
    
    def distance_to(self, other: 'RobotPose') -> float:
        return ((self.x - other.x)**2 + (self.y - other.y)**2) ** 0.5

pose = RobotPose(1.0, 2.0)  # __init__ auto-generated
print(pose)   # __repr__ auto-generated: RobotPose(x=1.0, y=2.0, theta=0.0, ...)

# @dataclass(frozen=True) makes it immutable and hashable (can use as dict key)
@dataclass(frozen=True)
class Point:
    x: float
    y: float
```

---

## PART 2: DATA STRUCTURES & ALGORITHMS

### Time & Space Complexity Quick Reference

| Operation | list | dict | set | heapq | deque | sorted list (bisect) |
|-----------|------|------|-----|-------|-------|----------------------|
| Access by index | O(1) | — | — | — | O(n) | O(1) |
| Search | O(n) | O(1) avg | O(1) avg | O(n) | O(n) | O(log n) |
| Insert at end | O(1)* | O(1) avg | O(1) avg | O(log n) | O(1) | O(log n) find + O(n) shift |
| Insert at front | O(n) | — | — | — | O(1) | — |
| Delete by value | O(n) | O(1) avg | O(1) avg | O(n) | O(n) | O(n) |
| Delete at end | O(1) | — | — | O(log n) | O(1) | O(1) |
| Min/Max | O(n) | — | — | O(1) peek | O(n) | O(1) |
| Sort | O(n log n) | — | — | — | — | already sorted |

*amortized O(1) — occasionally triggers resizing at O(n)

**Space complexity:** Most structures use O(n) space. Be explicit about it in interviews — "this solution is O(n) time and O(n) space due to the hash map."

### Arrays / Lists — Key Patterns

```python
# Pattern 1: Sliding Window — fixed size k
def max_sum_subarray(arr, k):
    window = sum(arr[:k])
    best = window
    for i in range(k, len(arr)):
        window += arr[i] - arr[i-k]   # Slide: add new, remove old
        best = max(best, window)
    return best

# Pattern 2: Variable Sliding Window — shrink/grow based on condition
def longest_subarray_with_sum_leq(arr, target):
    lo = window_sum = 0
    best = 0
    for hi in range(len(arr)):
        window_sum += arr[hi]
        while window_sum > target:
            window_sum -= arr[lo]
            lo += 1
        best = max(best, hi - lo + 1)
    return best

# Pattern 3: Two Pointers — sorted array, find pair summing to target
def two_sum_sorted(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo < hi:
        s = arr[lo] + arr[hi]
        if s == target: return (lo, hi)
        elif s < target: lo += 1
        else: hi -= 1
    return None

# Pattern 4: Prefix Sum — range sum queries in O(1) after O(n) setup
def build_prefix(arr):
    prefix = [0] * (len(arr) + 1)
    for i in range(len(arr)):
        prefix[i+1] = prefix[i] + arr[i]
    return prefix
# sum(arr[l:r+1]) == prefix[r+1] - prefix[l]

# Pattern 5: Kadane's Algorithm — maximum subarray sum O(n)
def max_subarray(nums):
    best = curr = nums[0]
    for n in nums[1:]:
        curr = max(n, curr + n)   # Either start fresh or extend
        best = max(best, curr)
    return best

# Pattern 6: Dutch National Flag — 3-way partition O(n)
def sort_colors(nums):
    lo = mid = 0
    hi = len(nums) - 1
    while mid <= hi:
        if nums[mid] == 0:
            nums[lo], nums[mid] = nums[mid], nums[lo]
            lo += 1; mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:
            nums[mid], nums[hi] = nums[hi], nums[mid]
            hi -= 1
```

### Hash Maps — The Most Important DS

```python
from collections import Counter, defaultdict, OrderedDict

# Counting occurrences — O(n)
counts = Counter([1, 2, 2, 3, 3, 3])  # Counter({3: 3, 2: 2, 1: 1})
counts.most_common(2)  # [(3, 3), (2, 2)]
counts['missing']      # 0, not KeyError (Counter is a defaultdict of int)

# Grouping with defaultdict — avoids KeyError on first access
groups = defaultdict(list)
for item in items:
    groups[item.category].append(item)

# Two Sum (unsorted) — O(n) time, O(n) space
def two_sum(nums, target):
    seen = {}   # value → index
    for i, n in enumerate(nums):
        comp = target - n
        if comp in seen:
            return [seen[comp], i]
        seen[n] = i

# Group anagrams — sort each word as key
def group_anagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(s))
        groups[key].append(s)
    return list(groups.values())

# Longest consecutive sequence — O(n) using set
def longest_consecutive(nums):
    num_set = set(nums)
    best = 0
    for n in num_set:
        if n - 1 not in num_set:   # Only start from sequence beginnings
            curr = n
            streak = 1
            while curr + 1 in num_set:
                curr += 1
                streak += 1
            best = max(best, streak)
    return best

# LRU Cache — dict + doubly linked list (or use OrderedDict)
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cap = capacity
        self.cache = OrderedDict()
    
    def get(self, key):
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)  # Mark as recently used
        return self.cache[key]
    
    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.cap:
            self.cache.popitem(last=False)  # Evict least recently used
```

### Stacks & Queues — Patterns

```python
from collections import deque

# Stack (LIFO) — use list
stack = []
stack.append(1)    # push O(1)
stack.pop()        # pop from top O(1)
stack[-1]          # peek O(1)

# Queue (FIFO) — use deque (O(1) both ends)
queue = deque()
queue.append(1)    # enqueue at right O(1)
queue.popleft()    # dequeue from left O(1)
# NEVER use list as queue — list.pop(0) is O(n) (shifts all elements)

# Monotonic stack — find next greater element to the right
def next_greater(nums):
    result = [-1] * len(nums)
    stack = []   # stores indices
    for i, num in enumerate(nums):
        # Pop anything smaller than current (current is their "next greater")
        while stack and nums[stack[-1]] < num:
            result[stack.pop()] = num
        stack.append(i)
    return result
# nums = [2, 1, 3, 4]
# result = [3, 3, 4, -1]

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

# Min stack — O(1) getMin()
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []   # Parallel stack tracking minimums
    
    def push(self, val):
        self.stack.append(val)
        min_val = min(val, self.min_stack[-1] if self.min_stack else val)
        self.min_stack.append(min_val)
    
    def pop(self):
        self.stack.pop()
        self.min_stack.pop()
    
    def getMin(self):
        return self.min_stack[-1]

# BFS / level-order traversal template (graphs, trees)
def bfs(start, get_neighbors, is_goal):
    queue = deque([(start, 0)])   # (node, distance)
    visited = {start}
    while queue:
        node, dist = queue.popleft()
        if is_goal(node):
            return dist
        for neighbor in get_neighbors(node):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
    return -1  # Not found
```

### Heaps (Priority Queue)

```python
import heapq

# Python heapq is a MIN heap (smallest item always at index 0)
nums = [3, 1, 4, 1, 5, 9]
heapq.heapify(nums)              # O(n) — transforms list into heap in-place
heapq.heappush(nums, 2)         # O(log n) — push item
smallest = heapq.heappop(nums)  # O(log n) — returns AND removes smallest
nums[0]                          # O(1) — peek at smallest WITHOUT removing

# For MAX heap: negate values
max_heap = [-x for x in [3, 1, 4]]
heapq.heapify(max_heap)
largest = -heapq.heappop(max_heap)   # Negate to get original value

# K largest/smallest — more efficient than sorting for large n
heapq.nlargest(3, nums)    # O(n log k) — top 3
heapq.nsmallest(3, nums)  # O(n log k) — bottom 3

# Heap with tuples — sorts by first element
tasks = [(5, "task A"), (1, "task B"), (3, "task C")]
heapq.heapify(tasks)
priority, name = heapq.heappop(tasks)  # (1, "task B") — highest priority

# Merge sorted arrays efficiently — O(n log k) for k arrays
merged = list(heapq.merge([1,3,5], [2,4,6], [0,7]))

# K closest points to origin — relevant to Brain Corp (nearest obstacles)
def k_closest(points, k):
    # Use max-heap of size k (negate distances so largest is on top)
    heap = []
    for x, y in points:
        dist = x*x + y*y  # No sqrt needed — monotonic
        if len(heap) < k:
            heapq.heappush(heap, (-dist, x, y))
        elif -dist > heap[0][0]:
            heapq.heapreplace(heap, (-dist, x, y))
    return [(x, y) for _, x, y in heap]

# Running median using two heaps
class MedianFinder:
    def __init__(self):
        self.lo = []   # Max-heap (negated) for lower half
        self.hi = []   # Min-heap for upper half
    
    def addNum(self, num):
        heapq.heappush(self.lo, -num)
        heapq.heappush(self.hi, -heapq.heappop(self.lo))
        if len(self.hi) > len(self.lo):
            heapq.heappush(self.lo, -heapq.heappop(self.hi))
    
    def findMedian(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2
```

### Trees — Complete Patterns

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

# BFS level order — iterative, most useful in interviews
def level_order(root):
    if not root: return []
    result = []
    queue = deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):   # Snapshot the level size
            node = queue.popleft()
            level.append(node.val)
            if node.left: queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level)
    return result

# DFS traversals — recursive
def inorder(root):    # Left, Root, Right — gives sorted order for BST
    if not root: return []
    return inorder(root.left) + [root.val] + inorder(root.right)

def preorder(root):   # Root, Left, Right — used for serialization/copying
    if not root: return []
    return [root.val] + preorder(root.left) + preorder(root.right)

def postorder(root):  # Left, Right, Root — used for deletion, evaluating expressions
    if not root: return []
    return postorder(root.left) + postorder(root.right) + [root.val]

# Iterative inorder — avoids stack overflow on deep trees
def inorder_iterative(root):
    result = []
    stack = []
    curr = root
    while curr or stack:
        while curr:          # Go as left as possible
            stack.append(curr)
            curr = curr.left
        curr = stack.pop()   # Process node
        result.append(curr.val)
        curr = curr.right    # Move to right subtree
    return result

# Max depth / height
def max_depth(root):
    if not root: return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))

# Lowest Common Ancestor (LCA) — important for robotics path queries
def lca(root, p, q):
    if not root or root == p or root == q:
        return root
    left = lca(root.left, p, q)
    right = lca(root.right, p, q)
    if left and right: return root   # p and q on opposite sides
    return left or right              # Both on one side

# Validate BST — track valid range
def is_valid_bst(root, lo=float('-inf'), hi=float('inf')):
    if not root: return True
    if not (lo < root.val < hi): return False
    return (is_valid_bst(root.left, lo, root.val) and
            is_valid_bst(root.right, root.val, hi))

# Path sum — does any root-to-leaf path sum to target?
def has_path_sum(root, target):
    if not root: return False
    if not root.left and not root.right:  # Leaf
        return root.val == target
    return (has_path_sum(root.left, target - root.val) or
            has_path_sum(root.right, target - root.val))

# Serialize / Deserialize binary tree
def serialize(root):
    if not root: return "null"
    return f"{root.val},{serialize(root.left)},{serialize(root.right)}"

def deserialize(data):
    vals = iter(data.split(','))
    def helper():
        v = next(vals)
        if v == "null": return None
        node = TreeNode(int(v))
        node.left = helper()
        node.right = helper()
        return node
    return helper()
```

### Graphs — Critical for Brain Corp (Navigation, SLAM, Path Planning)

```python
from collections import deque, defaultdict
import heapq

# Graph representations
# Adjacency list — O(V+E) space, good for sparse graphs (most real-world)
graph = defaultdict(list)
graph['A'].extend(['B', 'C'])
graph['B'].extend(['A', 'D'])

# Adjacency matrix — O(V²) space, good for dense graphs, O(1) edge lookup
V = 5
matrix = [[0]*V for _ in range(V)]
matrix[0][1] = 1   # Edge from 0 to 1

# BFS — shortest path in unweighted graph O(V+E)
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

# DFS — iterative (avoids Python recursion limit of 1000)
def dfs_iterative(graph, start):
    visited = set()
    stack = [start]
    order = []
    while stack:
        node = stack.pop()
        if node in visited: continue
        visited.add(node)
        order.append(node)
        for neighbor in reversed(graph[node]):  # reversed for consistent order
            if neighbor not in visited:
                stack.append(neighbor)
    return order

# Detect cycle in undirected graph
def has_cycle_undirected(graph, n):
    visited = set()
    def dfs(node, parent):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                if dfs(neighbor, node): return True
            elif neighbor != parent:   # Back edge (not the edge we came from)
                return True
        return False
    for node in range(n):
        if node not in visited:
            if dfs(node, -1): return True
    return False

# Topological sort (Kahn's algorithm — BFS-based)
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
    return order if len(order) == num_nodes else []  # Empty list = cycle detected

# Dijkstra — shortest path in weighted graph O((V+E) log V)
def dijkstra(graph, start):
    # graph[u] = [(weight, v), ...]
    dist = {start: 0}
    heap = [(0, start)]
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist.get(u, float('inf')):
            continue   # Stale entry
        for w, v in graph[u]:
            new_dist = d + w
            if new_dist < dist.get(v, float('inf')):
                dist[v] = new_dist
                heapq.heappush(heap, (new_dist, v))
    return dist

# A* search — Dijkstra + heuristic (Brain Corp: robot pathfinding on grid)
def astar(start, goal, neighbors_fn, heuristic):
    heap = [(0 + heuristic(start, goal), 0, start, [start])]
    visited = {}
    while heap:
        f, g, node, path = heapq.heappop(heap)
        if node in visited: continue
        visited[node] = g
        if node == goal: return path
        for neighbor, cost in neighbors_fn(node):
            if neighbor not in visited:
                new_g = g + cost
                heapq.heappush(heap, (new_g + heuristic(neighbor, goal), new_g, neighbor, path + [neighbor]))
    return None

def manhattan(a, b): return abs(a[0]-b[0]) + abs(a[1]-b[1])

# Union-Find (Disjoint Set Union) — for connected components, cycle detection
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n
        self.components = n
    
    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # Path compression
        return self.parent[x]
    
    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py: return False  # Already connected — cycle!
        # Union by rank
        if self.rank[px] < self.rank[py]: px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]: self.rank[px] += 1
        self.components -= 1
        return True
```

**Brain Corp graph connections:**
- **Grid navigation** → BFS for shortest path in unweighted grid, A* for weighted
- **SLAM** → graph of poses, loop closure detection
- **Task scheduling** → topological sort
- **Sensor fusion** → factor graphs, similar to Bayesian networks

### Sorting — Know the Internals

```python
# Python uses Timsort — hybrid merge sort + insertion sort, O(n log n), stable
# "Stable" = equal elements maintain their original relative order

# Custom sorting — key function
intervals = [(1,3), (2,1), (1,2)]
intervals.sort(key=lambda x: (x[0], x[1]))   # Sort by first, then second

# Sort descending
arr.sort(reverse=True)
arr.sort(key=lambda x: -x)    # Negate for descending on computed key

# sorted() returns new list; .sort() is in-place
new = sorted(students, key=lambda s: (-s.grade, s.name))

# Binary search — O(log n) on sorted data
import bisect
sorted_list = [1, 3, 5, 7, 9]
idx = bisect.bisect_left(sorted_list, 5)    # 2 — index of leftmost 5
idx = bisect.bisect_right(sorted_list, 5)   # 3 — index after rightmost 5
bisect.insort(sorted_list, 6)               # Inserts 6, maintains sorted order
bisect.insort_left(sorted_list, 5)          # Insert before existing 5s

# Count elements <= x in sorted array
count_leq = bisect.bisect_right(sorted_list, x)

# Sorting algorithms complexity
# Bubble sort:    O(n²) avg — useful as teaching tool only
# Selection sort: O(n²) avg — n swaps (good when write cost is high)
# Insertion sort: O(n²) avg, O(n) best — fast for nearly-sorted data (Timsort uses this)
# Merge sort:     O(n log n) always, O(n) space — stable, predictable
# Quick sort:     O(n log n) avg, O(n²) worst — in-place, cache-friendly
# Heap sort:      O(n log n) always, O(1) space — not stable, used in hybrid sorts
# Counting sort:  O(n+k) — for integer ranges [0, k]
# Radix sort:     O(d * (n+k)) — digit by digit, stable
```

### Recursion & Dynamic Programming

```python
# DP decision framework:
# 1. Does it have overlapping subproblems? (same sub-questions computed multiple times)
# 2. Does it have optimal substructure? (optimal solution uses optimal sub-solutions)
# If YES to both → DP. Otherwise → greedy or divide-and-conquer.

# Fibonacci — classic DP example
# Naive: O(2^n) time
@lru_cache(maxsize=None)    # Memoization — adds caching automatically
def fib(n):
    if n <= 1: return n
    return fib(n-1) + fib(n-2)

# Tabulation (bottom-up): O(n) time, O(1) space
def fib_dp(n):
    if n <= 1: return n
    a, b = 0, 1
    for _ in range(2, n+1):
        a, b = b, a + b
    return b

# Coin change — minimum coins to make amount
def coin_change(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    for a in range(1, amount + 1):
        for c in coins:
            if a - c >= 0:
                dp[a] = min(dp[a], 1 + dp[a-c])
    return dp[amount] if dp[amount] != float('inf') else -1

# 0/1 Knapsack — include or exclude each item
def knapsack(weights, values, capacity):
    n = len(weights)
    dp = [[0] * (capacity+1) for _ in range(n+1)]
    for i in range(1, n+1):
        for w in range(capacity+1):
            dp[i][w] = dp[i-1][w]   # Exclude item i
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w], values[i-1] + dp[i-1][w-weights[i-1]])
    return dp[n][capacity]

# Longest Common Subsequence (LCS)
def lcs(s, t):
    m, n = len(s), len(t)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(1, m+1):
        for j in range(1, n+1):
            if s[i-1] == t[j-1]:
                dp[i][j] = 1 + dp[i-1][j-1]
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]

# DP on intervals — burst balloons / matrix chain multiplication pattern
# State: dp[i][j] = answer for subproblem on interval [i..j]
# Usually O(n²) states × O(n) transitions = O(n³) total

# Backtracking template
def backtrack(state, choices):
    if is_solution(state):
        result.append(state[:])
        return
    for choice in choices:
        if is_valid(state, choice):
            state.append(choice)
            backtrack(state, next_choices(state))
            state.pop()   # Undo choice
```

---

## PART 3: OPERATING SYSTEM CONCEPTS

### Processes vs Threads

| | Process | Thread |
|---|---------|--------|
| Memory space | Separate virtual address space | Shared address space within process |
| Creation cost | Expensive — `fork()` copies page tables, file descriptors | Cheap — just a new stack + registers |
| Communication | IPC (pipes, sockets, shared memory, signals) | Direct shared memory (fast but needs synchronization) |
| Crash isolation | One crash doesn't kill other processes | One thread crash (segfault) kills entire process |
| Context switch | Slow — must switch page tables, flush TLB | Fast — same address space, just switch stack/registers |
| Python | `multiprocessing` — bypasses GIL, true parallelism | `threading` — GIL limits CPU parallelism |
| Brain Corp | ROS nodes are processes — crash isolation important | Sensor streams within a node use threads |

**Process creation:** `fork()` creates a child that is an exact copy of the parent (copy-on-write — pages only actually copied when written to). `exec()` replaces the process image with a new program. Shell does `fork()` then `exec()` to run commands.

**Zombie process:** A process that has finished but whose parent hasn't called `wait()` to collect its exit status. The entry remains in the process table. Fix: parent calls `waitpid()`, or child is re-parented to `init`.

**Orphan process:** Parent exits before child. Orphan is adopted by `init`/`systemd` (PID 1), which periodically calls `wait()`.

### Concurrency vs Parallelism

- **Concurrency:** Multiple tasks make progress — may be interleaved on one core (context switching). Like one juggler with multiple balls.
- **Parallelism:** Multiple tasks literally execute simultaneously on multiple cores. Like multiple jugglers.
- Python threading = concurrency (GIL). Python multiprocessing = true parallelism. asyncio = concurrency via cooperative yielding (no GIL issue, single thread).

```python
# asyncio — event loop, cooperative multitasking
import asyncio

async def fetch(session, url):
    async with session.get(url) as r:
        return await r.json()   # 'await' yields control back to event loop

async def main():
    async with aiohttp.ClientSession() as session:
        tasks = [asyncio.create_task(fetch(session, url)) for url in urls]
        results = await asyncio.gather(*tasks)   # Run all concurrently

# asyncio.gather vs asyncio.wait
# gather: returns results in same order as tasks, raises on first exception
# wait: returns completed/pending sets, more control

# asyncio + threading bridge
loop = asyncio.get_event_loop()
result = loop.run_in_executor(None, blocking_function, arg)  # Run in thread pool
```

### Synchronization Primitives — With Examples

```python
import threading

# Mutex (Lock) — only one thread can hold it at a time
lock = threading.Lock()
with lock:
    shared_counter += 1   # Thread-safe increment

# RLock (Reentrant Lock) — same thread can acquire it multiple times
rlock = threading.RLock()

# Semaphore — allows N threads simultaneously
sem = threading.Semaphore(value=5)   # Max 5 concurrent threads
with sem:
    access_limited_resource()

# Event — one thread signals another
event = threading.Event()
def producer():
    produce_data()
    event.set()          # Signal that data is ready

def consumer():
    event.wait()         # Block until producer signals
    process_data()

# Condition — wait for a condition with a lock
condition = threading.Condition()
buffer = []

def producer():
    with condition:
        buffer.append(item)
        condition.notify()   # Wake up one waiting consumer

def consumer():
    with condition:
        while not buffer:
            condition.wait()  # Release lock and wait; re-acquire when notified
        item = buffer.pop(0)
```

**Deadlock — Four Conditions (all must hold simultaneously):**
1. **Mutual exclusion** — resource can't be shared (lock)
2. **Hold and wait** — holding one resource while waiting for another
3. **No preemption** — can't forcibly take a resource from another thread
4. **Circular wait** — Thread A waits for B, Thread B waits for A

**Prevention (break any one condition):**
- Break circular wait: Always acquire locks in the **same global order** across all threads. Lock A then Lock B — never B then A.
- Break hold-and-wait: Try to acquire all locks atomically (`try_lock` with timeout).

**Race condition:** Two threads access shared data, at least one writes, no synchronization. Fix: use a lock. "Check-then-act" is a classic race:
```python
# RACE: another thread can change balance between check and debit
if account.balance >= amount:       # Check
    account.balance -= amount       # Act

# FIX: hold the lock for the entire check-then-act
with account.lock:
    if account.balance >= amount:
        account.balance -= amount
```

### Memory Management

**Stack vs Heap:**

| | Stack | Heap |
|---|-------|------|
| What's stored | Function frames, local variables, return addresses | Dynamically allocated objects |
| Size | Small — typically 1–8MB per thread | Large — limited by RAM |
| Allocation | Automatic — push/pop on function call/return | Manual (C/C++) or garbage-collected |
| Speed | Fast — just move stack pointer | Slower — allocator bookkeeping |
| Lifetime | Variable lives until function returns | Variable lives until freed/GC'd |

**Python memory management:**
- Everything lives on the **heap** (even small ints)
- Python's **private heap** is managed by `pymalloc` (small objects) and the OS allocator (large objects)
- **Reference counting** (primary): every object has a `ob_refcnt`. When it hits 0, object is immediately freed.
- **Cyclic garbage collector** (`gc` module): handles reference cycles (`a → b → a`)

```python
import sys, gc

a = [1, 2, 3]
sys.getrefcount(a)   # Returns count + 1 (the call itself adds a reference)

# Reference cycle — neither gets freed by refcount alone
class Node:
    def __init__(self): self.self_ref = self

n = Node()   # n.self_ref → n — cycle!
del n        # refcount goes from 2 → 1, not 0 — NOT freed
gc.collect() # Cyclic GC detects and frees it

# Memory profiling
import tracemalloc
tracemalloc.start()
# ... code ...
snapshot = tracemalloc.take_snapshot()
for stat in snapshot.statistics('lineno')[:5]:
    print(stat)
```

### Virtual Memory & Paging

- **Virtual memory:** Each process gets its own virtual address space — appears to have all memory to itself. Actual physical memory is shared.
- **Page:** Fixed-size block (typically 4KB). Virtual pages map to physical frames via the page table.
- **Page fault (minor):** Page is in memory but not mapped — kernel maps it. Fast.
- **Page fault (major):** Page is not in memory — kernel must load it from disk (swap). Slow — milliseconds.
- **TLB (Translation Lookaside Buffer):** Hardware cache for page table entries. Cache miss = must walk the page table. TLB flush on context switch (one reason context switches are expensive).
- **Huge pages (2MB / 1GB):** Reduce TLB pressure for large allocations. Used in HPC and databases.

**Memory layout of a process (low to high addresses):**
```
[ Text (code) | Data (globals) | BSS (uninit globals) | Heap ↑ | ... | Stack ↓ ]
```

### Scheduling Algorithms

| Algorithm | How It Works | Pros | Cons |
|-----------|-------------|------|------|
| FCFS | First come, first served | Simple | Convoy effect — short jobs wait behind long ones |
| SJF (non-preemptive) | Shortest burst time runs first | Optimal avg wait time | Starvation of long jobs; must predict burst time |
| SRTF (preemptive SJF) | Always runs shortest remaining time | Optimal avg wait | Preemption overhead, starvation |
| Round Robin | Each process gets a fixed time quantum, then preempted | Fair, responsive | High context switch overhead if quantum too small |
| Priority | Higher priority runs first | Important tasks run first | Starvation of low-priority — fix with **aging** |
| MLFQ | Multiple queues; processes demoted based on CPU usage | Balances interactive and batch | Complex, tunable parameters |
| CFS (Linux) | Completely Fair Scheduler — tracks "virtual runtime," runs least-run process | Fair on multicore | Complex, overhead |

**Context switch cost:** Save registers, PC, stack pointer; switch page tables; flush TLB; load new process state. Typically 1–10 microseconds. At 1ms quantum and 1μs switch overhead, that's 0.1% overhead — acceptable.

**Priority inversion:** Low-priority task holds a lock that a high-priority task needs. Medium-priority tasks preempt the low one → high-priority task starved. **Fix: Priority inheritance** — low-priority task temporarily borrows the waiting high-priority task's priority.

### File Systems & I/O

- **Inode:** Metadata about a file (size, permissions, uid/gid, timestamps, block pointers). Does NOT contain the filename.
- **Directory entry:** Maps filenames → inode numbers. A directory is just a file containing (name, inode) pairs.
- **Hard link:** Another directory entry pointing to the same inode. `ln file hardlink`. The file's data is not deleted until ALL hard links are removed. Both names are equal — no "original."
- **Symbolic link (symlink):** A separate file whose content is a path. `ln -s target symlink`. Can span filesystems; can break if target deleted.
- **File descriptors (FDs):** Per-process integer handles for open files. Table maps FD → file table entry → inode. FD 0=stdin, 1=stdout, 2=stderr.
- **`mmap`:** Maps file contents into the process's virtual address space. Reads/writes to memory automatically sync to the file. Used for shared memory between processes and for fast file I/O.

**Linux commands (you have Linux on every resume):**
```bash
ps aux                # List all processes (user, PID, CPU%, MEM%, command)
top / htop            # Live monitor of CPU/memory
kill -9 <pid>         # SIGKILL — cannot be caught or ignored
kill -15 <pid>        # SIGTERM — polite shutdown request (default)
strace ./program      # Trace system calls — gold for debugging
ltrace ./program      # Trace library calls
lsof -i :8080         # What process is using port 8080?
netstat -tlnp         # Listening ports and processes
grep -r "text" .      # Recursive search for text
grep -n "pattern" f   # Show line numbers
find . -name "*.py" -mtime -1   # .py files modified in last day
chmod 755 file        # rwxr-xr-x
chown user:group file # Change ownership
df -h                 # Disk usage (human readable)
du -sh ./dir          # Directory size
free -m               # RAM usage in MB
vmstat 1              # CPU/memory/IO stats every second
iostat -x 1           # Disk I/O stats
nvidia-smi            # GPU status, memory, processes (Jetson/CUDA)
journalctl -u service # Service logs (systemd)
systemctl status ros2 # Check ROS2 service status
```

### Interprocess Communication (IPC)

| Method | How it works | Latency | Brain Corp use |
|--------|-------------|---------|----------------|
| Pipe (unnamed) | One-way byte stream, kernel buffer, parent↔child | Fast | Basic command piping |
| Named Pipe (FIFO) | Like pipe but has a filesystem path, any processes | Fast | Simple inter-node comm |
| Shared Memory | Processes map same physical pages into their address space. Must synchronize with semaphores. | Fastest | High-throughput sensor data between processes |
| Message Queue | Kernel-managed queue of structured messages, async | Moderate | Decoupled producer/consumer |
| Unix Domain Socket | Socket-based, local only, faster than TCP | Fast | ROS 2 intra-host transport |
| TCP/UDP Socket | Network socket, works across machines | Slower (RTT) | Multi-machine robot fleet comms |
| Signal | Integer notification (SIGTERM, SIGUSR1, SIGALRM) | Immediate | Simple events, shutdown |
| D-Bus | Message bus for Linux desktop/system services | Moderate | System-level notifications |

**ROS 2 specifics:** ROS 2 uses **DDS (Data Distribution Service)** middleware. Nodes communicate via:
- **Topics** — publish/subscribe, async, one-to-many
- **Services** — request/reply, synchronous
- **Actions** — long-running tasks with feedback and preemption
- **Parameters** — runtime configuration

DDS can use shared memory transport (FastDDS SHM) for zero-copy intra-process or intra-host communication — critical for high-frequency sensor data (LiDAR, cameras).

---

## PART 4: DATABASE CONCEPTS

### SQL — Full Patterns

```sql
-- SELECT with multiple JOINs
SELECT e.name, d.dept_name, m.name AS manager
FROM employees e
JOIN departments d ON e.dept_id = d.id
LEFT JOIN employees m ON e.manager_id = m.id   -- LEFT because some may have no manager
WHERE e.salary > 50000
  AND e.hire_date >= '2020-01-01'
ORDER BY e.name
LIMIT 10 OFFSET 20;  -- Pagination (skip first 20)

-- Aggregation with HAVING
SELECT dept_id,
       COUNT(*) AS cnt,
       AVG(salary) AS avg_sal,
       MAX(salary) AS max_sal,
       MIN(salary) AS min_sal
FROM employees
WHERE status = 'active'
GROUP BY dept_id
HAVING COUNT(*) > 5   -- HAVING filters on aggregates; WHERE filters on rows BEFORE grouping
ORDER BY avg_sal DESC;

-- Subquery vs JOIN — subquery often slower (may execute once per row)
-- Subquery:
SELECT name FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Correlated subquery — executes once per outer row (usually slow!)
SELECT e.name FROM employees e
WHERE e.salary > (SELECT AVG(e2.salary) FROM employees e2 WHERE e2.dept_id = e.dept_id);
-- Better as a JOIN with a subquery in FROM:
SELECT e.name FROM employees e
JOIN (SELECT dept_id, AVG(salary) AS avg_sal FROM employees GROUP BY dept_id) dept_avg
  ON e.dept_id = dept_avg.dept_id
WHERE e.salary > dept_avg.avg_sal;

-- CTEs (Common Table Expressions) — readable, reusable subqueries
WITH dept_stats AS (
    SELECT dept_id, AVG(salary) AS avg_sal, COUNT(*) AS cnt
    FROM employees GROUP BY dept_id
),
top_depts AS (
    SELECT dept_id FROM dept_stats WHERE cnt > 10
)
SELECT e.name, e.salary
FROM employees e
JOIN top_depts t ON e.dept_id = t.dept_id
JOIN dept_stats ds ON e.dept_id = ds.dept_id
WHERE e.salary > ds.avg_sal;

-- Recursive CTE — hierarchy traversal (org charts, file systems)
WITH RECURSIVE subordinates AS (
    SELECT id, name, manager_id, 1 AS level
    FROM employees WHERE manager_id IS NULL  -- Root (CEO)
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, s.level + 1
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates ORDER BY level, name;

-- Window Functions — without collapsing rows (unlike GROUP BY)
SELECT name, salary, dept_id,
    RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS dept_rank,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS global_rank,
    LAG(salary, 1) OVER (PARTITION BY dept_id ORDER BY hire_date) AS prev_sal,
    LEAD(salary, 1) OVER (PARTITION BY dept_id ORDER BY hire_date) AS next_sal,
    SUM(salary) OVER (PARTITION BY dept_id) AS dept_total,
    AVG(salary) OVER (PARTITION BY dept_id ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS rolling_avg
FROM employees;

-- RANK vs DENSE_RANK vs ROW_NUMBER
-- Scores: 90, 85, 85, 80
-- RANK:        1,  2,  2,  4  (gaps)
-- DENSE_RANK:  1,  2,  2,  3  (no gaps)
-- ROW_NUMBER:  1,  2,  3,  4  (always unique)

-- UPSERT (INSERT or UPDATE)
-- PostgreSQL:
INSERT INTO sensors (id, value, ts) VALUES (1, 42.5, NOW())
ON CONFLICT (id) DO UPDATE SET value = EXCLUDED.value, ts = EXCLUDED.ts;

-- MySQL:
INSERT INTO sensors VALUES (1, 42.5, NOW())
ON DUPLICATE KEY UPDATE value = VALUES(value), ts = VALUES(ts);
```

### JOIN Types — Visual

```
Table A: {1, 2, 3}   Table B: {2, 3, 4}

INNER JOIN:  {2, 3}          — only matching rows
LEFT JOIN:   {1, 2, 3}       — all of A; B columns NULL for 1
RIGHT JOIN:  {2, 3, 4}       — all of B; A columns NULL for 4
FULL OUTER:  {1, 2, 3, 4}    — all rows; NULLs where no match
CROSS JOIN:  3×3 = 9 rows    — cartesian product (every pair)
SELF JOIN:   Table joined to itself (e.g., employees and their managers)
```

### ACID Properties

| Property | What it means | Example |
|----------|--------------|---------|
| **Atomicity** | Transaction is all-or-nothing — every statement succeeds, or the whole thing rolls back | Bank transfer: debit AND credit must both succeed; partial execution is impossible |
| **Consistency** | DB moves from one valid state to another — all constraints upheld | Foreign key constraints, CHECK constraints, NOT NULL — never violated by a commit |
| **Isolation** | Concurrent transactions don't interfere with each other | Two simultaneous transfers don't double-spend or see partial updates |
| **Durability** | Committed data survives crashes | Written to WAL (Write-Ahead Log) before confirming commit; survives power loss |

**How durability works:** PostgreSQL (and most RDBMS) uses WAL. Before modifying data pages, the change is written to a sequential log file. On crash, the DB replays the WAL to recover. This is why SSDs and RAID matter for databases.

### Indexing — Deep Dive

```sql
-- B-Tree index (default): balanced tree, O(log n) lookup
-- Good for: equality (=), range (>, <, BETWEEN), ORDER BY, GROUP BY
CREATE INDEX idx_salary ON employees(salary);

-- Hash index: O(1) average lookup — equality only, no range queries
CREATE INDEX idx_email_hash ON employees USING HASH (email);

-- Composite index — column ORDER matters (leftmost prefix rule)
CREATE INDEX idx_last_first ON employees(last_name, first_name);
-- Supports: WHERE last_name = ?
-- Supports: WHERE last_name = ? AND first_name = ?
-- Does NOT help: WHERE first_name = ?     (not leftmost)
-- Does NOT help: WHERE last_name LIKE '%son' (leading wildcard)

-- Covering index — all needed columns in the index (no table lookup needed)
CREATE INDEX idx_covering ON employees(dept_id, salary, name);
-- Query: SELECT name, salary FROM employees WHERE dept_id = 5 ORDER BY salary
-- Can be answered entirely from the index — no heap access ("index-only scan")

-- Partial index — index only a subset of rows
CREATE INDEX idx_active ON employees(last_name) WHERE status = 'active';
-- Smaller index, faster inserts, ideal when querying a specific subset

-- Expression index
CREATE INDEX idx_lower_email ON employees(LOWER(email));
-- Supports: WHERE LOWER(email) = 'user@example.com'

-- When to add an index:
-- Columns in WHERE clauses that are highly selective (many distinct values)
-- Columns in JOIN ON conditions (foreign keys)
-- Columns in ORDER BY / GROUP BY
-- When: query is slow AND EXPLAIN shows a sequential scan on a large table

-- When NOT to add an index:
-- Small tables (sequential scan faster — no index overhead)
-- Low-selectivity columns (e.g., boolean 'is_active' with 90% true)
-- Frequently updated columns (each write must update the index)
-- You already have too many indexes — writes become slow

-- EXPLAIN ANALYZE — understand query execution
EXPLAIN ANALYZE SELECT * FROM employees WHERE last_name = 'Luhar';
-- Look for: Seq Scan (bad on large table) vs Index Scan (good) vs Index Only Scan (best)
-- Look for: actual rows vs estimated rows (large difference = stale statistics → ANALYZE)
```

### Normalization — With Practical Examples

| Form | Rule | Violation Example | Fix |
|------|------|-------------------|-----|
| **1NF** | Atomic values, no repeating groups | Column `skills = "Python, C++, Java"` | Separate `employee_skills` table |
| **2NF** | 1NF + no partial dependencies on composite key | Table `(student_id, course_id, student_name)` — `student_name` depends only on `student_id` | Separate students table |
| **3NF** | 2NF + no transitive dependencies | `employees` has `dept_id` AND `dept_name` — `dept_name` depends on `dept_id`, not employee | Move `dept_name` to `departments` table |
| **BCNF** | Every determinant is a candidate key | Stronger than 3NF — eliminates some remaining anomalies |
| **4NF** | No multi-valued dependencies | Rarely needed in practice |

**When to denormalize:** Analytics/data warehouses read far more than they write. Star schema (fact table + dimension tables) is denormalized by design. Trade-off: faster reads, data redundancy, harder to update.

**Robot telemetry example:**
```sql
-- Normalized (OLTP) — correct and consistent, but lots of joins
robots(robot_id, model, fleet_id)
fleets(fleet_id, name, region)
sensor_readings(reading_id, robot_id, sensor_type, value, ts)

-- Denormalized (analytics) — fast queries on billions of rows
sensor_readings_flat(reading_id, robot_id, model, fleet_name, region, sensor_type, value, ts)
```

### SQL vs NoSQL

| | SQL (Relational) | Document (MongoDB) | Key-Value (Redis) | Column (Cassandra) | Graph (Neo4j) |
|---|---|---|---|---|---|
| Schema | Fixed, predefined | Flexible, per-document | None | Flexible per row | Property-based |
| Scaling | Vertical (bigger machine) | Horizontal sharding | Horizontal | Horizontal | Horizontal (limited) |
| Query language | SQL | MQL (JSON-like) | Commands | CQL (SQL-like) | Cypher |
| Joins | Yes, powerful | No (embed or $lookup) | No | No | Graph traversal |
| ACID | Yes | Single-doc ACID | Yes (single key) | Eventual (tunable) | Yes |
| Best for | Structured data, complex queries, transactions | JSON documents, flexible schemas | Caching, sessions, leaderboards | Time-series, write-heavy, distributed | Social networks, recommendations, paths |

**Robot telemetry context:** Thousands of robots sending position/sensor data → time-series DB (InfluxDB, TimescaleDB) or Cassandra for ingestion at scale. Configuration and fleet management → PostgreSQL (relational, strong consistency). Session caching → Redis.

### CAP Theorem

You can guarantee at most 2 of 3 during a **network partition**:

- **Consistency (C):** Every read returns the most recent write (or an error)
- **Availability (A):** Every request gets a (non-error) response — not necessarily the latest data
- **Partition Tolerance (P):** System continues working despite network splits between nodes

In practice, network partitions happen, so P is mandatory. The real trade-off is **CP vs AP**:

| | CP systems | AP systems |
|---|---|---|
| During partition | Reject reads/writes to maintain consistency | Serve possibly stale data |
| Examples | HBase, ZooKeeper, etcd, PostgreSQL (with sync replication) | DynamoDB, Cassandra, CouchDB |
| Robot fleet | Use CP for critical state (robot assignment) | Use AP for telemetry (slight staleness OK) |

**PACELC** (extension of CAP): Even without partitions, there's a latency vs consistency trade-off. Systems that are PC also tend toward high latency; PA systems tend toward low latency.

### Transactions & Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Typical Use |
|-------|-----------|-------------------|-------------|-------------|
| Read Uncommitted | Yes | Yes | Yes | Rare — logging, approximate analytics |
| Read Committed | No | Yes | Yes | Most databases default (PostgreSQL) |
| Repeatable Read | No | No | Yes | MySQL InnoDB default; financial reads |
| Serializable | No | No | No | Strictest — high contention, slowest |

- **Dirty read:** Reading uncommitted data from another transaction (which might roll back)
- **Non-repeatable read:** Same `SELECT` returns different rows within one transaction (another committed `UPDATE`)
- **Phantom read:** Same `SELECT` with `WHERE` returns different set of rows (another committed `INSERT` or `DELETE`)

**Optimistic vs Pessimistic locking:**
- **Pessimistic:** Lock the row before reading (`SELECT ... FOR UPDATE`). Prevents conflicts but reduces throughput.
- **Optimistic:** No lock; check at commit time if data was modified (compare version/timestamp). High throughput when conflicts are rare. Retry on conflict.

```sql
-- Pessimistic locking
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;  -- Locks the row
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- Optimistic locking pattern (application-side)
-- Add 'version' column to table
UPDATE accounts SET balance = balance - 100, version = version + 1
WHERE id = 1 AND version = 5;  -- If version changed, 0 rows updated → retry
```

---

## PART 5: OOP & DESIGN PATTERNS

### OOP Pillars with Python Examples

```python
# ENCAPSULATION — hide internal state, expose clean interface
class SensorBuffer:
    def __init__(self, maxsize=100):
        self._buffer = []          # Convention: _ = "private"
        self.__maxsize = maxsize   # Name-mangled: __x → _SensorBuffer__x
    
    def add(self, reading):
        if len(self._buffer) >= self.__maxsize:
            self._buffer.pop(0)   # Oldest out
        self._buffer.append(reading)
    
    @property
    def latest(self):
        return self._buffer[-1] if self._buffer else None
    
    @property
    def size(self): return len(self._buffer)

# INHERITANCE — reuse and extend behavior
class Robot:
    def __init__(self, name, max_speed):
        self.name = name
        self.max_speed = max_speed
    
    def describe(self):
        return f"Robot {self.name}, max speed {self.max_speed} m/s"
    
    def move(self, velocity):
        raise NotImplementedError("Subclasses must implement move()")

class CleaningRobot(Robot):
    def __init__(self, name):
        super().__init__(name, max_speed=1.5)
        self.brush_speed = 0
    
    def move(self, velocity):
        # Override with cleaning-robot-specific movement
        clipped = min(velocity, self.max_speed)
        print(f"Moving at {clipped} m/s while cleaning")
    
    def describe(self):
        base = super().describe()   # Extend parent method
        return f"{base}, cleaning robot"

# POLYMORPHISM — same interface, different behavior
robots = [CleaningRobot("Bot1"), SecurityRobot("Bot2")]
for robot in robots:
    robot.move(1.0)   # Each implements move() differently

# ABSTRACTION — abstract base classes enforce interface contracts
from abc import ABC, abstractmethod

class Navigator(ABC):
    @abstractmethod
    def plan_path(self, start, goal) -> list: ...
    
    @abstractmethod
    def is_path_blocked(self, path) -> bool: ...
    
    def navigate(self, start, goal):   # Template method — concrete!
        path = self.plan_path(start, goal)
        if self.is_path_blocked(path):
            path = self.plan_path(start, goal)  # Re-plan
        return path

class AStarNavigator(Navigator):
    def plan_path(self, start, goal):
        return astar(start, goal, ...)   # Concrete implementation
    
    def is_path_blocked(self, path):
        return any(is_obstacle(p) for p in path)
```

### Design Patterns — Most Common in Interviews

```python
# SINGLETON — only one instance (e.g., RobotConfig, Logger)
class RobotConfig:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        if not self._initialized:
            self.max_speed = 1.5
            self._initialized = True

# FACTORY — create objects without specifying exact class
class SensorFactory:
    _registry = {}
    
    @classmethod
    def register(cls, name):
        def decorator(sensor_cls):
            cls._registry[name] = sensor_cls
            return sensor_cls
        return decorator
    
    @classmethod
    def create(cls, name, **kwargs):
        if name not in cls._registry:
            raise ValueError(f"Unknown sensor: {name}")
        return cls._registry[name](**kwargs)

@SensorFactory.register("lidar")
class LidarSensor:
    def __init__(self, range_m=10): self.range_m = range_m

sensor = SensorFactory.create("lidar", range_m=15)

# OBSERVER — pub/sub (like ROS topics)
class EventBus:
    def __init__(self):
        self._subscribers = defaultdict(list)
    
    def subscribe(self, event_type, callback):
        self._subscribers[event_type].append(callback)
    
    def publish(self, event_type, data):
        for callback in self._subscribers[event_type]:
            callback(data)

bus = EventBus()
bus.subscribe("obstacle_detected", lambda d: print(f"Alert: {d}"))
bus.publish("obstacle_detected", {"position": (1.5, 2.0), "size": 0.3})

# STRATEGY — swap algorithms at runtime
class PathPlanner:
    def __init__(self, strategy):
        self._strategy = strategy
    
    def set_strategy(self, strategy):
        self._strategy = strategy
    
    def plan(self, start, goal):
        return self._strategy.plan(start, goal)

class AStarStrategy:
    def plan(self, start, goal): ...

class DijkstraStrategy:
    def plan(self, start, goal): ...

planner = PathPlanner(AStarStrategy())
planner.set_strategy(DijkstraStrategy())  # Swap strategy at runtime

# DECORATOR PATTERN (structural, not Python decorator)
class SensorBase:
    def read(self): return []

class NoisyLidar(SensorBase):
    def read(self): return [1.5, 2.0, 1.8]

class FilteredSensor(SensorBase):   # Wrapper
    def __init__(self, sensor, window=3):
        self._sensor = sensor
        self._window = window
    
    def read(self):
        readings = self._sensor.read()
        return sorted(readings)[len(readings)//4 : -len(readings)//4] or readings

# Compose: filtered lidar
filtered = FilteredSensor(NoisyLidar(), window=5)
```

### Python-Specific OOP Details

```python
# MRO (Method Resolution Order) — Python 3 uses C3 linearization
class A:
    def method(self): return "A"

class B(A):
    def method(self): return "B"

class C(A):
    def method(self): return "C"

class D(B, C):   # Multiple inheritance
    pass

D.mro()  # [D, B, C, A, object] — C3 linearization
D().method()  # "B" — follows MRO

# __slots__ — fix allowed attributes, reduce memory (no __dict__ per instance)
class Point:
    __slots__ = ('x', 'y')   # Saves ~50 bytes per instance
    def __init__(self, x, y):
        self.x = x
        self.y = y
# point.z = 1  → AttributeError

# __repr__ vs __str__
class Pose:
    def __repr__(self):
        return f"Pose(x={self.x}, y={self.y})"  # For developers, unambiguous
    def __str__(self):
        return f"Position: ({self.x:.2f}, {self.y:.2f})"  # For users, readable

# Comparison methods
from functools import total_ordering

@total_ordering
class Priority:
    def __init__(self, level):
        self.level = level
    def __eq__(self, other): return self.level == other.level
    def __lt__(self, other): return self.level < other.level
    # @total_ordering fills in >, >=, <= automatically
```

---

## PART 6: QUICK-FIRE ANSWERS

**"What happens when you type `python script.py`?"**
> Shell finds the Python interpreter in `$PATH` → reads the `.py` file → **lexer** tokenizes the source → **parser** builds an AST (Abstract Syntax Tree) → **compiler** generates bytecode (`.pyc`, cached in `__pycache__/`) → **PVM** (Python Virtual Machine) executes the bytecode instruction by instruction. If the `.pyc` is cached and the source hasn't changed (mtime check), compilation is skipped.

---

**"What's the difference between `__init__` and `__new__`?"**
> `__new__` allocates and returns a new instance (controls creation). `__init__` initializes the already-created instance (controls initialization). `__new__` is called first. You override `__new__` for singletons, immutable types (like subclassing `tuple` or `str`), or metaclasses. `__init__` is what you override almost every other time.

---

**"What are Python's dunder/magic methods?"**
> Methods surrounded by double underscores that Python calls automatically for built-in operations:
> - `__init__` / `__new__` / `__del__` — lifecycle
> - `__str__` / `__repr__` — string conversion
> - `__len__` / `__getitem__` / `__setitem__` / `__iter__` / `__next__` — container protocol
> - `__eq__` / `__lt__` / `__hash__` — comparison and hashing
> - `__enter__` / `__exit__` — context manager
> - `__call__` — makes instances callable: `obj()`
> - `__add__` / `__mul__` / etc. — operator overloading

---

**"Explain `yield` and generators"**
> `yield` turns a function into a generator factory. When called, it returns a generator object — it doesn't execute anything yet. Calling `next()` (or iterating) runs the function until the next `yield`, pauses, and returns the value. The local state (variables, instruction pointer) is preserved between calls. Memory-efficient for large data: generates values one at a time rather than building the entire result in memory. `yield from` delegates to a sub-generator. Generators can also receive values via `gen.send(value)`.

---

**"What is a deadlock? How would you detect one in your robot code?"**
> A deadlock is when two or more threads/processes each hold a resource the other needs, so none can proceed. In a robot: the perception thread holds the camera lock and waits for the planner lock; the planner thread holds the planner lock and waits for the camera lock. **Detection:** Lock acquisition timeout (`threading.Lock().acquire(timeout=5)`) — if it times out, assume deadlock and log/alert. Or build a resource allocation graph and check for cycles (can't do this easily at runtime; better to prevent). **Prevention:** Enforce a global lock ordering — always acquire locks in alphabetical or numerical order, never reverse.

---

**"Process vs thread for robot perception?"**
> Use **separate processes** for independent subsystems (perception, planning, control) — crash isolation is critical. A buggy perception process shouldn't kill the control loop. Use **threads** within a process for lightweight tasks sharing memory — multiple camera streams feeding the same perception node. In Python specifically: use `ProcessPoolExecutor` for CPU-bound image processing (bypasses GIL). Threads are fine for I/O-bound work (serial ports, network calls). ROS 2 naturally maps each node to a process, with intra-process communication for zero-copy data sharing.

---

**"How does a robot handle real-time constraints?"**
> Real-time means **deterministic and bounded latency**, not necessarily fast. **Hard real-time** (motor control, safety) requires RTOS or RT-PREEMPT Linux, C/C++, bounded memory allocation, no dynamic allocation in loops. **Soft real-time** (perception, planning) can use Python — fixed-rate loops with deadline tracking, dropping stale frames rather than queuing them (to avoid unbounded latency growth), using `time.monotonic()` for timing, offloading heavy computation to separate processes.

---

**"What's the difference between deep copy and shallow copy?"**
> **Shallow copy** creates a new container object but references the same elements. Modifying a nested object affects both. `list[:]`, `list.copy()`, `copy.copy()` are shallow. **Deep copy** (`copy.deepcopy()`) recursively copies all objects — fully independent. Use deep copy when you have nested mutable structures and need full independence. Caveat: deep copy is slow and can fail on objects with custom `__copy__` behavior.

---

**"How does Python's GC handle reference cycles?"**
> CPython's primary GC is reference counting — when `refcount` hits 0, the object is immediately freed. But cycles (`a.ref = b; b.ref = a`) prevent refcount from reaching 0. The cyclic GC (`gc` module) periodically scans objects in "generations" (gen 0, 1, 2) looking for groups of objects that are only reachable from each other (net refcount = 0 accounting for internal references). It frees those groups. You can trigger it with `gc.collect()`. Disable it (carefully) in tight loops for performance: `gc.disable()`.

---

**"Explain B-Tree vs Hash index"**
> **B-Tree** (balanced n-ary search tree): O(log n) lookup, supports equality, range queries, ORDER BY, and LIKE 'prefix%'. The default index type in PostgreSQL and MySQL. Leaf nodes store actual data (or pointers); inner nodes are routing keys. Self-balancing via rotations/splits. **Hash index**: O(1) average lookup for exact equality. No range queries — the hash gives no information about adjacent keys. Faster for point lookups but inflexible. In PostgreSQL, hash indexes are now WAL-logged (crash-safe since PG10). MySQL InnoDB doesn't support explicit hash indexes (only adaptive hash index — automatic, internal).

---

**"What is database sharding?"**
> Sharding is horizontal partitioning — splitting a large table/database across multiple servers. Each **shard** holds a subset of rows (e.g., shard by `user_id % 10`). Enables horizontal scaling — you can add shards. Challenges: cross-shard queries (joins across shards are expensive), rebalancing when adding shards, maintaining ACID across shards (requires distributed transactions). Alternatives: read replicas (for read scaling), partitioning (splits within one server), vertical scaling.

---

**"What is the difference between `is` and `==` in Python?"**
> `==` calls `__eq__` and checks **value equality** — do the objects have the same content? `is` checks **identity** — are they literally the same object in memory (same `id()`)? Use `==` to compare values (almost always what you want). Use `is` only for `None` checks (`if x is None`) and identity checks. Never use `is` to compare integers, strings, or lists — CPython caches small integers (-5 to 256) and interns some strings, so `is` might return `True` for equal small ints, but this is an implementation detail, not guaranteed behavior.

---

**"Explain eventual consistency"**
> In a distributed system, **eventual consistency** means that if no new updates are made to a piece of data, all replicas will eventually converge to the same value. There's no guarantee about **when** — only that it will happen. During the convergence window, different nodes may return different values. Used in AP systems (Cassandra, DynamoDB, DNS). Contrast with **strong consistency**: every read reflects the most recent write, but this requires coordination between nodes (slower, less available). For robot fleet telemetry, eventual consistency is acceptable — slightly stale position data is fine. For robot task assignment, strong consistency is required — you don't want two robots assigned the same task.

---

*Good luck Thursday, Ayush. You've got this.*
