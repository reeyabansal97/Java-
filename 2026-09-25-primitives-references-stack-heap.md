# Java Day 2: Primitives vs References, Stack vs Heap

## Why this matters

This one idea explains a lot of Java behavior that otherwise looks random: why changing an object inside a method affects the caller but changing an `int` doesn't, why `==` fails on strings (Day 3 of DSA), and what `NullPointerException` actually means.

## Two kinds of types

**Primitive types** hold the actual value directly. Java has exactly eight:

| Type | Size | Example |
|---|---|---|
| `byte` | 8 bits | `byte b = 10;` |
| `short` | 16 bits | `short s = 1000;` |
| `int` | 32 bits | `int x = 42;` |
| `long` | 64 bits | `long big = 9_000_000_000L;` |
| `float` | 32 bits | `float f = 3.14f;` |
| `double` | 64 bits | `double d = 3.14;` |
| `char` | 16 bits | `char c = 'A';` |
| `boolean` | JVM-dependent | `boolean ok = true;` |

**Reference types** are everything else: `String`, arrays, `ArrayList`, your own classes. A reference variable doesn't hold the object. It holds the **address** of where the object lives, like a note saying "the object is over there."

## Where things live: stack vs heap

**Stack**
- One per thread.
- Holds method call frames (you saw these in DSA Day 6 on recursion).
- Each frame stores that method's local variables: primitive values directly, and reference variables (the addresses).
- Very fast, and cleaned up automatically when the method returns.

**Heap**
- Shared by all threads.
- Holds every object created with `new` (and arrays, and strings).
- Cleaned up by the **garbage collector** once nothing references an object anymore.

```java
public void example() {
    int count = 5;                         // stack: the value 5
    Person p = new Person("Reeya");        // stack: address of p; heap: the Person object
}
```

When `example()` returns, its frame is popped: `count` and the reference `p` disappear. The `Person` object stays on the heap until the garbage collector notices nothing points to it and removes it.

## The consequence: two variables can share one object

```java
int a = 10;
int b = a;       // b gets a COPY of the value
b = 20;
System.out.println(a);   // 10, unchanged

int[] x = {1, 2, 3};
int[] y = x;     // y gets a copy of the ADDRESS, so both point to the same array
y[0] = 99;
System.out.println(x[0]);   // 99! same array
```

Copying a primitive copies the value. Copying a reference copies the address, so both variables now point at the same object.

## Java is always pass-by-value

This is a classic interview trap. Java **always** passes a copy of the variable into a method. The twist is what gets copied:

- For a primitive, the copy is the value.
- For a reference, the copy is the address.

```java
void modify(int n, int[] arr) {
    n = 100;          // changes the local copy only; caller's int is unaffected
    arr[0] = 100;     // follows the copied address to the SAME array; caller sees this
    arr = new int[]{7, 7, 7};   // re-points the local copy only; caller's array is unaffected
}
```

The third line is the proof that Java is pass-by-value, not pass-by-reference. If it were pass-by-reference, reassigning `arr` inside the method would change the caller's variable too. It doesn't.

## `null` and NullPointerException

A reference variable can hold `null`, which means "points to nothing." Calling a method on it (`p.getName()` when `p` is `null`) has no object to follow the address to, so Java throws `NullPointerException`. Primitives can never be `null`.

## Wrapper classes and autoboxing (brief)

Collections like `ArrayList` can only hold objects, not primitives. So each primitive has a wrapper class: `int` → `Integer`, `double` → `Double`, and so on. Java converts between them automatically (**autoboxing** and **unboxing**):

```java
List<Integer> list = new ArrayList<>();
list.add(5);            // autoboxing: int 5 → Integer object
int first = list.get(0);   // unboxing: Integer → int
```

Watch out: unboxing a `null` `Integer` into an `int` throws `NullPointerException`.

## Common pitfalls

- **Assuming assignment copies an object.** `y = x` for arrays or objects creates a second name for the same object, not a copy.
- **Comparing wrappers with `==`.** `Integer` values are objects, so `==` compares addresses. It happens to work for small values (Java caches `Integer` objects from -128 to 127) and then silently breaks for larger ones. Use `.equals()`.
- **Believing Java passes objects by reference.** It passes references by value. Reassigning a parameter never affects the caller.

## Interviewer follow-ups

**"Is Java pass-by-value or pass-by-reference?"** Always pass-by-value. For objects, the value being copied is the reference (the address). That's why mutating an object's contents is visible to the caller, but reassigning the parameter isn't.

**"What lives on the stack vs the heap?"** Stack: method frames with local primitives and reference variables, one stack per thread. Heap: all objects and arrays, shared across threads, managed by the garbage collector.

## Retention Check

1. Name the eight primitive types.
2. What does a reference variable actually store?
3. What goes on the stack, and what goes on the heap?
4. After `int[] y = x; y[0] = 99;`, what is `x[0]`, and why?
5. Why is Java called pass-by-value even when passing objects?
6. In the `modify` example, why doesn't `arr = new int[]{7,7,7}` affect the caller?
7. What causes a `NullPointerException`?
8. Why might `Integer a = 127, b = 127; a == b` be `true` while the same comparison with `128` is `false`?

**Key points to self-check against:**
- `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`.
- The address of an object on the heap, not the object itself.
- Stack: method frames with local primitives and reference variables. Heap: objects and arrays.
- `99`, because `x` and `y` hold the same address and point to the same array.
- Java copies the variable's value into the method; for objects, that value is the address.
- It only re-points the method's local copy of the address; the caller's variable still points to the original array.
- Calling a method or accessing a field through a reference that is `null`.
- Java caches `Integer` objects from -128 to 127, so both variables point to the same cached object; 128 creates two separate objects.
