# Java Day 6: Polymorphism

## What does it mean?

"Polymorphism" means "many forms." In Java: **one reference type, many actual behaviors.** A variable of a parent type can point to any child object, and when you call a method, the **child's** version runs.

```java
Account a1 = new SavingsAccount("Reeya", 1000, 0.04);
Account a2 = new CurrentAccount("Asha", 500, 2000);

a1.withdraw(100);   // runs SavingsAccount's version (inherited from Account)
a2.withdraw(100);   // runs CurrentAccount's overridden version, with overdraft
```

Both variables are declared as `Account`, but each call does the right thing for the actual object. (These classes come from yesterday's inheritance read.)

## Why that's useful

You can write code once against the parent type, and it works for every current *and future* child:

```java
void processWithdrawals(List<Account> accounts, double amount) {
    for (Account acc : accounts) {
        acc.withdraw(amount);   // each account applies its own rules
    }
}
```

Add a `FixedDepositAccount` next month with its own `withdraw` rules, and this method keeps working unchanged. No `if (acc instanceof SavingsAccount) ... else if ...` chains. This is the idea behind "open for extension, closed for modification."

## Two kinds of polymorphism

**Compile-time (static) polymorphism: overloading.** The same method name with different parameters. The compiler picks which one to call, based on the argument types it sees.

```java
int add(int a, int b)          { return a + b; }
double add(double a, double b) { return a + b; }
```

**Runtime (dynamic) polymorphism: overriding.** The same method signature in parent and child. The JVM picks which one to run **at runtime**, based on the actual object, not the declared variable type. This is called **dynamic dispatch**, and it's what people usually mean by "polymorphism."

## The key rule: declared type vs actual type

```java
Account acc = new SavingsAccount("Reeya", 1000, 0.04);
```

- **Declared type** (`Account`) decides what you're **allowed to call**. The compiler only lets you call methods that `Account` has.
- **Actual type** (`SavingsAccount`) decides **which version runs**.

So this doesn't compile:

```java
acc.addInterest();   // compile error: Account has no addInterest()
```

...even though the object really is a `SavingsAccount`. The compiler only knows the declared type.

## Casting (and why to avoid needing it)

If you truly need child-only methods, you can **downcast**:

```java
if (acc instanceof SavingsAccount s) {   // pattern matching, Java 16+
    s.addInterest();
}
```

Casting without checking is dangerous: `(SavingsAccount) someCurrentAccount` throws `ClassCastException` at runtime.

Frequent `instanceof` checks are usually a design smell. They mean the behavior should probably be a method on the parent that each child overrides.

## What is *not* polymorphic

- **Fields.** If a parent and child both declare a field named `x`, the declared type decides which one you read. Fields are never dispatched at runtime. (Another reason to keep fields `private`, Java Day 4.)
- **Static methods.** They belong to the class (Java Day 3), so they're chosen by declared type. A child "redefining" a static method is called **hiding**, not overriding.
- **Private and final methods.** They can't be overridden, so there's nothing to dispatch.

## You already use this every day

```java
List<String> names = new ArrayList<>();
Map<Character, Integer> freq = new HashMap<>();
Deque<Integer> stack = new ArrayDeque<>();
```

You declare the **interface** type (`List`, `Map`, `Deque`) and pick an implementation. Switch `ArrayList` to `LinkedList` later, and no other code has to change. That's polymorphism. Interfaces are the next Java day.

## Common pitfalls

- **Calling child-only methods through a parent reference.** Compile error: the declared type limits what you can call.
- **Casting without `instanceof`.** Risks a `ClassCastException`.
- **Expecting fields or static methods to behave polymorphically.** They don't; only overridden instance methods do.
- **Long `instanceof` chains.** Usually means a method should be overridden instead.

## Interviewer follow-ups

**"Overloading vs overriding?"** Overloading: same name, different parameters, chosen at compile time. Overriding: same name and parameters in a subclass, chosen at runtime by the actual object. Only overriding is runtime polymorphism.

**"Why declare `List<String> list = new ArrayList<>()` instead of `ArrayList<String>`?"** Code depends only on the `List` contract, so you can swap the implementation later without changing callers. It's programming to an interface, not an implementation.

## Retention Check

1. What does runtime polymorphism mean, in one sentence?
2. In `Account acc = new SavingsAccount(...)`, what decides which methods you can call, and what decides which version runs?
3. Why doesn't `acc.addInterest()` compile?
4. How does polymorphism remove the need for `instanceof` chains?
5. What's the difference between compile-time and runtime polymorphism?
6. Are fields polymorphic? Are static methods?
7. What's the safe way to downcast?
8. Why is `List<String> names = new ArrayList<>()` an example of polymorphism?

**Key points to self-check against:**
- A parent-type reference can point to a child object, and the child's overridden method runs.
- Declared type decides what's callable; actual object type decides which version runs.
- The declared type `Account` has no `addInterest()` method.
- Each child overrides the method with its own behavior, so one call works for all types.
- Compile-time is overloading, chosen by argument types; runtime is overriding, chosen by the actual object.
- No and no; both are chosen by the declared type.
- Check with `instanceof` first (or use pattern matching `instanceof SavingsAccount s`).
- Code depends on the `List` interface, and the actual `ArrayList` behavior is chosen at runtime, so implementations are swappable.
