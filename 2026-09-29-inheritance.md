# Java Day 5: Inheritance

## What problem does it solve?

Say you have `SavingsAccount` and `CurrentAccount`. Both need an owner, a balance, `deposit()`, and `getBalance()`. Writing all of that twice means duplicated code, and duplicated bugs. **Inheritance** lets you put the shared parts in one **parent class** (also called a **superclass**) and have **child classes** (**subclasses**) reuse and extend them.

```java
public class Account {
    protected String owner;
    protected double balance;

    public Account(String owner, double balance) {
        this.owner = owner;
        this.balance = balance;
    }

    public void deposit(double amount) {
        balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}

public class SavingsAccount extends Account {
    private final double interestRate;

    public SavingsAccount(String owner, double balance, double interestRate) {
        super(owner, balance);        // run the parent's constructor first
        this.interestRate = interestRate;
    }

    public void addInterest() {
        balance += balance * interestRate;   // 'balance' is inherited
    }
}
```

`SavingsAccount` automatically has `owner`, `balance`, `deposit()`, and `getBalance()`, and it adds `addInterest()` on top. (Money as `double` is only for readability; real code uses `BigDecimal`.)

## "Is-a": the test for inheritance

Use inheritance only when the child genuinely **is a** kind of the parent. A `SavingsAccount` *is an* `Account`. ✅ A `Car` *is not an* `Engine`, it *has an* engine. ❌ That relationship is **composition**: `Car` holds an `Engine` field instead of extending it.

## `super`: talking to the parent

- **`super(...)`** calls the parent's constructor. It must be the **first line** in the child's constructor.
- **`super.method()`** calls the parent's version of a method you've overridden.

**Constructor chaining rule:** every constructor calls a parent constructor first. If you don't write `super(...)`, Java silently inserts `super()` (no arguments). So if the parent has **no** no-argument constructor, which is exactly the Java Day 3 "disappearing default constructor" rule, the child won't compile until you call `super(...)` explicitly with arguments.

## Method overriding

A child can **replace** a parent method with its own version:

```java
public class Account {
    public void withdraw(double amount) {
        if (amount > balance) {
            throw new IllegalArgumentException("Insufficient funds");
        }
        balance -= amount;
    }
}

public class CurrentAccount extends Account {
    private final double overdraftLimit;

    public CurrentAccount(String owner, double balance, double overdraftLimit) {
        super(owner, balance);
        this.overdraftLimit = overdraftLimit;
    }

    @Override
    public void withdraw(double amount) {
        if (amount > balance + overdraftLimit) {
            throw new IllegalArgumentException("Overdraft limit exceeded");
        }
        balance -= amount;
    }
}
```

**Always write `@Override`.** If you make a typo, like `withdrow`, or get the parameters wrong, the compiler catches it. Without `@Override`, you'd quietly create a brand-new method, and the parent's version would keep running.

Overriding rules:
- Same method name and parameter types.
- You can't make it **less** accessible (a `public` method can't become `private` in the child).
- `private`, `static`, and `final` methods can't be overridden.

## What's inherited and what isn't

- ✅ `public` and `protected` fields and methods, plus package-private ones if the child is in the same package (Java Day 4).
- ❌ `private` members: they exist inside the object, but the child can't access them directly.
- ❌ Constructors: never inherited. That's why the child must call `super(...)`.

## `final` and single inheritance

- A `final` **class** can't be extended. `String` is final, which is part of why its immutability can be trusted (Java Day 4).
- A `final` **method** can't be overridden.
- Java allows **only one parent class** per class: `class A extends B, C` doesn't compile. This avoids ambiguity when two parents define the same method. (Interfaces, coming soon, are how Java lets a class take on multiple types.)
- Every class ultimately extends `Object`. That's where `equals()`, `hashCode()`, and `toString()` come from (DSA Day 8).

## Composition over inheritance

A widely followed design principle: **prefer composition** (holding an object as a field) **over inheritance**, unless there's a true "is-a" relationship. Inheritance tightly couples the child to the parent's internals, so changing the parent can quietly break every child. Composition keeps the pieces independent and swappable.

A classic warning sign: `class Stack extends ArrayList`. Now your stack exposes `add(index, element)` and `remove(index)`, which let callers break stack behavior. A stack that *has an* internal list and exposes only `push`/`pop` is safer.

## Common pitfalls

- **Inheriting just to reuse code**, without a real "is-a" relationship.
- **Forgetting `super(...)`** when the parent has no no-argument constructor.
- **Skipping `@Override`**, and silently creating a new method because of a typo.
- **Calling overridable methods from a constructor.** The parent's constructor runs first, so if it calls a method the child overrides, the child's version runs *before* the child's own fields have been set.

## Interviewer follow-ups

**"Why doesn't Java support multiple inheritance of classes?"** To avoid the "diamond problem": if two parents both define `doWork()`, it's ambiguous which one the child inherits. Java lets a class implement multiple interfaces instead.

**"Overloading vs overriding?"** Overloading: same method name, **different parameters**, in the same class, resolved at compile time. Overriding: same name **and** same parameters, in a subclass, resolved at runtime based on the actual object type. That runtime choice is polymorphism, which is the next Java day.

## Retention Check

1. What problem does inheritance solve?
2. What's the "is-a" test, and how does "has-a" differ?
3. What does `super(...)` do, and where must it go?
4. Why won't a child compile if the parent has only a constructor that takes arguments?
5. Why should you always write `@Override`?
6. Which members are not inherited, or not directly accessible?
7. Why doesn't Java allow a class to extend two classes?
8. Why is `class Stack extends ArrayList` a bad design?

**Key points to self-check against:**
- It removes duplicated code by putting shared fields and behavior in a parent class that children reuse.
- Inherit only when the child *is a* kind of the parent; if it *has a* part, use composition (a field).
- It calls the parent's constructor; it must be the first line of the child's constructor.
- Java inserts `super()` by default, which fails if the parent has no no-argument constructor.
- The compiler catches typos or mismatched parameters that would otherwise create a new method.
- Constructors aren't inherited; `private` members can't be accessed directly.
- The diamond problem: two parents defining the same method would be ambiguous.
- Stack inherits `ArrayList` methods, like inserting at any index, that break stack behavior; composition is safer.
