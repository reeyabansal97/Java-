# Java Day 9: Comparable vs Comparator

## What problem does it solve?

`Arrays.sort()` on an `int[]` just works: numbers have an obvious order. But what about this?

```java
List<Employee> employees = ...;
Collections.sort(employees);   // sort by what? name? salary? age?
```

Java has no idea how to order your own objects. You have to tell it, and there are two ways to do that.

## Comparable: the object's "natural" order

A class implements `Comparable<T>` to define its **one default ordering**, built into the class itself:

```java
public class Employee implements Comparable<Employee> {
    private final String name;
    private final int salary;

    public Employee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }

    public String getName() { return name; }
    public int getSalary() { return salary; }

    @Override
    public int compareTo(Employee other) {
        return this.name.compareTo(other.name);   // natural order: by name
    }
}

Collections.sort(employees);   // now works: sorts by name
```

`String`, `Integer`, and `LocalDate` all implement `Comparable`, which is why sorting them "just works."

## The compareTo contract

`a.compareTo(b)` returns:
- **negative** if `a` should come **before** `b`
- **zero** if they're equal in ordering
- **positive** if `a` should come **after** `b`

Only the **sign** matters, not the exact number.

**A classic bug: comparing by subtraction**

```java
return this.salary - other.salary;   // looks clever, but buggy
```

If `salary` is a large positive number and `other.salary` is a large negative one, the subtraction **overflows** `int` and flips sign, so the order comes out wrong. It's the same overflow problem as `(low + high) / 2` in binary search (Day 12). Use the safe helper instead:

```java
return Integer.compare(this.salary, other.salary);
```

## Comparator: any order you want, defined outside the class

What if you sometimes need to sort by name, and sometimes by salary? A class gets only **one** `compareTo`. And what if you can't edit the class at all, because it's from a library?

A **`Comparator<T>`** is a separate object that defines an ordering. You can make as many as you like:

```java
Comparator<Employee> bySalary = Comparator.comparingInt(Employee::getSalary);
Comparator<Employee> byName = Comparator.comparing(Employee::getName);

employees.sort(bySalary);              // ascending salary
employees.sort(bySalary.reversed());   // descending salary
```

`Comparator` is a **functional interface** (Java Day 7: exactly one abstract method, `compare`), which is why lambdas and method references work here.

## Sorting by multiple keys

```java
employees.sort(
    Comparator.comparing(Employee::getDepartment)
              .thenComparing(Employee::getName)
);
```

This sorts by department, then by name within each department. `thenComparing` only breaks ties left by the first comparator.

(This is the readable alternative to the "two stable sort passes" trick from today's DSA read. And `List.sort` is stable, so both approaches agree.)

## Handling nulls

If a sort key can be `null`, a plain comparator throws `NullPointerException`. Decide explicitly where nulls should go:

```java
Comparator.comparing(Employee::getManager, Comparator.nullsLast(Comparator.naturalOrder()));
```

## Comparable vs Comparator at a glance

| | Comparable | Comparator |
|---|---|---|
| Where it lives | Inside the class (`implements`) | A separate object |
| Method | `compareTo(T other)` | `compare(T a, T b)` |
| How many orders | One (the natural order) | As many as you need |
| Can use on classes you can't edit? | No | Yes |
| Typical use | The obvious default: dates by time, accounts by ID | Context-specific sorts: by salary, by name descending |

## Where else this shows up

Ordering isn't just for `sort()`:
- **`TreeMap` / `TreeSet`** keep keys sorted using `Comparable` or a `Comparator` you pass in.
- **`PriorityQueue`** (a heap, a later DSA topic) uses them to decide which element comes out first.

```java
PriorityQueue<Employee> highestPaidFirst =
    new PriorityQueue<>(Comparator.comparingInt(Employee::getSalary).reversed());
```

## Consistency with equals (good to know)

If `compareTo` returns 0, ideally `equals` should return `true` as well. Sorted collections like `TreeSet` use `compareTo`, **not** `equals`, to detect duplicates. So if you compare employees only by name, adding two *different* employees who share a name to a `TreeSet` silently keeps just one of them.

## Common pitfalls

- **Subtraction in compareTo:** overflow can flip the sign. Use `Integer.compare` or `Comparator.comparingInt`.
- **Sorting objects that don't implement Comparable** without a comparator: throws `ClassCastException` at runtime.
- **Null keys** without `nullsFirst` or `nullsLast`: `NullPointerException`.
- **compareTo inconsistent with equals**, used in a `TreeSet` or `TreeMap`: distinct objects silently get treated as duplicates.

## Interviewer follow-ups

**"Comparable or Comparator?"** Comparable for the one obvious natural ordering, built into the class. Comparator for any other ordering, multiple orderings, or classes you can't modify.

**"Why is `return a - b;` a bad compare implementation?"** Integer overflow can flip the sign for large values, producing an incorrect order. `Integer.compare(a, b)` avoids that.

## Retention Check

1. Why can't Java sort a list of your own objects without extra information?
2. What do negative, zero, and positive results from `compareTo` mean?
3. Why is `return this.salary - other.salary;` buggy?
4. When should you use a `Comparator` instead of `Comparable`?
5. How do you sort by department, then by name?
6. Why can you write a `Comparator` as a lambda?
7. How do you handle `null` sort keys?
8. What goes wrong in a `TreeSet` if `compareTo` only compares names?

**Key points to self-check against:**
- Your class has no built-in ordering; Java needs `Comparable` or a `Comparator` to decide.
- Negative: `this` comes first. Zero: equal in order. Positive: `this` comes after.
- The subtraction can overflow `int` and flip the sign.
- For multiple orderings, context-specific orderings, or classes you can't edit.
- `Comparator.comparing(Employee::getDepartment).thenComparing(Employee::getName)`.
- It's a functional interface: it has exactly one abstract method, `compare`.
- Wrap the key comparator with `Comparator.nullsFirst(...)` or `Comparator.nullsLast(...)`.
- Two different employees with the same name compare as 0, so the set keeps only one.
