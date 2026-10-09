# Java Day 12: Lambdas & Functional Interfaces

## What problem do they solve?

Sometimes you want to pass **behavior** into a method: "sort these *by salary*," "remove items *that are empty*," "run this *later*." Before Java 8, passing behavior meant creating an object of an anonymous class:

```java
employees.sort(new Comparator<Employee>() {
    @Override
    public int compare(Employee a, Employee b) {
        return Integer.compare(a.getSalary(), b.getSalary());
    }
});
```

Six lines of ceremony for one line of logic. A **lambda** writes the same thing as just the logic:

```java
employees.sort((a, b) -> Integer.compare(a.getSalary(), b.getSalary()));
```

## Lambda syntax

```
(parameters) -> expression
(parameters) -> { statements; return value; }
```

| Form | Example |
|---|---|
| No parameters | `() -> System.out.println("hi")` |
| One parameter (parentheses optional) | `s -> s.isEmpty()` |
| Several parameters | `(a, b) -> a + b` |
| Block body | `(a, b) -> { int sum = a + b; return sum * 2; }` |

Parameter types are usually **inferred** by the compiler from context, so you rarely write them.

## Functional interfaces: what a lambda actually is

A lambda isn't a standalone thing. It's always an implementation of a **functional interface**: an interface with **exactly one abstract method** (Java Day 7). The lambda's body becomes that one method's body.

`Comparator` has one abstract method, `compare`, so a lambda can stand in for a `Comparator`. That's all there is to it.

You can mark your own with `@FunctionalInterface`. The compiler then errors if someone later adds a second abstract method:

```java
@FunctionalInterface
public interface DiscountRule {
    double apply(double price);
}

DiscountRule festive = price -> price * 0.9;
double finalPrice = festive.apply(1000);   // 900.0
```

(`default` and `static` methods don't count toward the one-abstract-method limit.)

## The built-in functional interfaces (`java.util.function`)

You rarely need to write your own. These four cover most cases:

| Interface | Method | Takes → Returns | Example use |
|---|---|---|---|
| `Predicate<T>` | `test` | `T → boolean` | `s -> s.isEmpty()` (a condition) |
| `Function<T, R>` | `apply` | `T → R` | `s -> s.length()` (a transformation) |
| `Consumer<T>` | `accept` | `T → nothing` | `s -> System.out.println(s)` (an action) |
| `Supplier<T>` | `get` | `nothing → T` | `() -> new ArrayList<>()` (a factory) |

Plus two-argument versions like `BiFunction<T, U, R>`, the type behind `freq.merge(c, 1, Integer::sum)`.

There are also primitive versions, like `IntPredicate` and `IntFunction`, that avoid boxing `int` into `Integer` (Java Day 2).

## Method references: an even shorter lambda

When a lambda only calls an existing method, you can refer to that method directly with `::`

| Kind | Method reference | Equivalent lambda |
|---|---|---|
| Static method | `Integer::sum` | `(a, b) -> Integer.sum(a, b)` |
| Instance method of the argument | `String::isEmpty` | `s -> s.isEmpty()` |
| Instance method of a specific object | `System.out::println` | `x -> System.out.println(x)` |
| Constructor | `ArrayList::new` | `() -> new ArrayList<>()` |

You've already written several: `names.removeIf(String::isEmpty)` (Java Day 10), `Comparator.comparing(Employee::getName)` (Java Day 9).

## Composing behavior

Functional interfaces have `default` methods for combining them:

```java
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> shortWord = s -> s.length() < 5;
Predicate<String> both = notEmpty.and(shortWord);

Function<Integer, Integer> doubleIt = x -> x * 2;
Function<Integer, Integer> plusOne = x -> x + 1;
doubleIt.andThen(plusOne).apply(5);   // (5 * 2) + 1 = 11
```

`Comparator.comparing(...).thenComparing(...).reversed()` from Java Day 9 is this same idea.

## The "effectively final" rule

A lambda can use local variables from the surrounding method, but only if they're **never reassigned** after being set. These are called **effectively final** variables:

```java
int threshold = 10;
list.removeIf(x -> x < threshold);   // fine

int count = 0;
list.forEach(x -> count++);          // compile error: count is modified
```

Why? A lambda can run later, even on another thread, after the method has returned. Local variables live in the method's stack frame (Java Day 2), which is gone by then. So Java **copies** the value into the lambda. If the variable could change afterward, the copy and the original would silently disagree, so Java forbids changing it.

(Fields and objects on the heap are different. You *can* modify an object a lambda refers to, like adding to a list. But doing that from lambdas running on several threads is unsafe, which is a concurrency topic.)

## Common pitfalls

- **Modifying a local variable inside a lambda.** It won't compile; it must be effectively final.
- **Huge multi-line lambdas.** If a lambda grows past a few lines, move the logic into a named method and use a method reference. It's easier to read and test.
- **Checked exceptions inside lambdas.** `Function.apply` doesn't declare `throws IOException`, so calling a method that throws a checked exception (Java Day 8) inside a lambda forces an awkward `try/catch` there.
- **`this` means different things.** Inside a lambda, `this` refers to the enclosing object. Inside an anonymous class, `this` refers to the anonymous class instance itself.

## Interviewer follow-ups

**"What is a functional interface?"** An interface with exactly one abstract method. Any lambda or method reference whose shape matches that method can be used as an instance of it. Examples: `Runnable`, `Comparator`, `Predicate`, `Function`.

**"Why must variables used in a lambda be effectively final?"** The lambda may run after the method returns, when the local variable's stack frame no longer exists, so Java captures a copy of the value. Forbidding changes guarantees the copy and the original can never disagree.

## Retention Check

1. What problem do lambdas solve compared with anonymous classes?
2. What is a functional interface?
3. Why can a lambda be passed wherever a `Comparator` is expected?
4. What do `Predicate`, `Function`, `Consumer`, and `Supplier` each take and return?
5. Rewrite `s -> s.toUpperCase()` as a method reference.
6. What does `@FunctionalInterface` do?
7. Why can't a lambda modify a local variable from the enclosing method?
8. What does `doubleIt.andThen(plusOne).apply(5)` return, and why?

**Key points to self-check against:**
- They pass behavior without the boilerplate of creating an anonymous class.
- An interface with exactly one abstract method.
- `Comparator` has one abstract method, `compare`, and the lambda supplies its body.
- Predicate: T → boolean. Function: T → R. Consumer: T → nothing. Supplier: nothing → T.
- `String::toUpperCase`.
- It makes the compiler reject the interface if it ever has more than one abstract method.
- The lambda captures a copy of the value, and may run after the method's stack frame is gone; changes would make the copy and the original disagree.
- 11: double first (10), then add one.
