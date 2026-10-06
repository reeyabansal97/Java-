# Java Day 10: The Collections Framework — Choosing the Right Collection

## Why this matters

You've already used `ArrayList`, `HashMap`, `ArrayDeque`, `LinkedList`, and `TreeSet` in earlier reads. This read puts them on one map, so you can answer the question interviewers actually ask: **"Which collection would you use here, and why?"**

## The big picture

```
Iterable
 └── Collection
      ├── List    → ordered, allows duplicates, access by index
      ├── Set     → no duplicates
      └── Queue   → ordered for processing (FIFO, priority, ...)
           └── Deque → both ends

Map (separate hierarchy) → key → value pairs, unique keys
```

`Map` is **not** a `Collection`. It's its own interface, because it stores pairs rather than single elements.

These are all **interfaces** . You declare variables with the interface type and choose an implementation, which is polymorphism :

```java
List<String> names = new ArrayList<>();
Map<String, Integer> counts = new HashMap<>();
```

## Step 1: pick the interface

Ask what you need:

| Need | Interface |
|---|---|
| Ordered items, duplicates allowed, access by position | `List` |
| Unique items, fast "does it contain x?" | `Set` |
| Look up a value by a key | `Map` |
| Process items in arrival order, or from both ends | `Queue` / `Deque` |
| Always take out the smallest (or highest-priority) item | `PriorityQueue` |

## Step 2: pick the implementation

### Lists

| | `ArrayList` | `LinkedList` |
|---|---|---|
| Backed by | Resizable array (DSA Day 2) | Doubly linked nodes (DSA Day 9) |
| `get(i)` | O(1) | O(n) |
| Add at end | Amortized O(1) | O(1) |
| Add/remove at front | O(n) (shifts) | O(1) |
| Memory | Compact | Extra node objects and references |

**Default to `ArrayList`.** Even where `LinkedList` wins on paper, `ArrayList` is often faster in practice, because contiguous memory is much friendlier to CPU caches. If you need fast operations at both ends, `ArrayDeque` is usually better than `LinkedList` too.

### Sets and Maps (they're built the same way)

A `HashSet` is actually a `HashMap` underneath, using only its keys. So the three choices match:

| | `HashSet` / `HashMap` | `LinkedHashSet` / `LinkedHashMap` | `TreeSet` / `TreeMap` |
|---|---|---|---|
| Ordering | None (don't rely on it) | Insertion order | Sorted |
| `add` / `get` / `contains` | O(1) average | O(1) average | O(log n) |
| Built on | Hash table  | Hash table + linked list | Red-black tree (balanced BST) |
| Needs | `equals` + `hashCode` | `equals` + `hashCode` | `Comparable` or a `Comparator` |

**Default to `HashMap` / `HashSet`.** Switch to:
- `LinkedHashMap` when output order should match insertion order. (It can also be configured for access order, which turns it into a simple LRU cache, the eviction policy from backend Day 6.)
- `TreeMap` when you need keys sorted, or "range" questions like "the first key ≥ 50" (`ceilingKey`) or "all keys between a and b" (`subMap`).

### Queues

- **`ArrayDeque`**: the default for stacks and FIFO queues (DSA Days 10–11).
- **`PriorityQueue`**: a min-heap. `poll()` always returns the smallest element (or the "first" by your `Comparator`). `offer` and `poll` are O(log n). Heaps get their own DSA day later.

## Immutable collections

```java
List<String> fixed = List.of("a", "b", "c");
Map<String, Integer> fixedMap = Map.of("x", 1, "y", 2);
List<String> copy = List.copyOf(someList);
```

These throw `UnsupportedOperationException` if you try to modify them, and they **don't allow `null`**. They're the safe way to return internal data without leaking it (the `Team.getMembers()` leak from Java Day 4).

## Iterating safely

```java
for (String name : names) {
    if (name.isEmpty()) names.remove(name);   // throws ConcurrentModificationException
}
```

Modifying a collection while a for-each loop is walking it breaks the loop's internal iterator, so Java throws `ConcurrentModificationException`. This is called "fail-fast" behavior. Safe alternatives:

```java
names.removeIf(String::isEmpty);   // clean and correct
```

or use an explicit `Iterator` and call `iterator.remove()`.

Despite the name, this exception happens in completely **single-threaded** code. It's about modifying during iteration, not about threads.

## Thread safety (preview)

`ArrayList`, `HashMap`, and the rest are **not thread-safe**. If several threads modify one at the same time, data can be corrupted. The fixes, like `ConcurrentHashMap`, come in the concurrency days of this track. The old `Vector` and `Hashtable` classes are synchronized, but they're considered legacy (the same reason `Stack` was, in DSA Day 10).

## Common pitfalls

- **Picking `LinkedList` "for fast inserts"** without measuring. `ArrayList` or `ArrayDeque` is usually faster.
- **Expecting `HashMap` to keep order.** Use `LinkedHashMap` or `TreeMap`.
- **Mutable keys in a `HashMap`, or a missing `hashCode`.** Lookups silently fail (DSA Day 8).
- **Removing inside a for-each loop.** `ConcurrentModificationException`; use `removeIf`.
- **Modifying `List.of(...)`.** `UnsupportedOperationException`.
- **`TreeSet` with a `compareTo` that's inconsistent with `equals`.** Distinct items are treated as duplicates (Java Day 9).

## Interviewer follow-ups

**"ArrayList vs LinkedList?"** `ArrayList` gives O(1) access by index and better cache performance, and is the default. `LinkedList` gives O(1) insertion and removal at the ends or at an iterator position, but O(n) access by index and more memory. In practice, `ArrayList` (or `ArrayDeque` for both ends) usually wins.

**"HashMap vs TreeMap?"** `HashMap`: O(1) average, no ordering, needs `equals`/`hashCode`. `TreeMap`: O(log n), keys always sorted, supports range queries, needs `Comparable` or a `Comparator`.

## Retention Check

1. Why isn't `Map` a subtype of `Collection`?
2. Which interface would you choose for: unique tags, a to-do list in order, and looking up a user by email?
3. Why is `ArrayList` usually the better default over `LinkedList`?
4. What's the relationship between `HashSet` and `HashMap`?
5. When would you use `LinkedHashMap`? When would you use `TreeMap`?
6. What does `PriorityQueue.poll()` return, and how fast is it?
7. Why does removing inside a for-each loop throw an exception, and what's the fix?
8. What happens if you call `add` on a list created with `List.of(...)`?

**Key points to self-check against:**
- A map stores key-value pairs, not single elements, so it has its own interface.
- `Set`; `List`; `Map`.
- O(1) index access and contiguous memory that's cache-friendly; `LinkedList`'s theoretical wins rarely show up in practice.
- A `HashSet` is backed by a `HashMap` and uses only its keys.
- `LinkedHashMap` for insertion order (or an LRU cache); `TreeMap` for sorted keys and range queries.
- The smallest element (by natural order or your `Comparator`), in O(log n).
- It breaks the loop's internal iterator (fail-fast, `ConcurrentModificationException`); use `removeIf` or `Iterator.remove()`.
- It throws `UnsupportedOperationException`, because the list is immutable.
