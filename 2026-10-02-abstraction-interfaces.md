# Java Day 7: Abstraction — Abstract Classes & Interfaces

## The last OOP pillar

1. Encapsulation: protect data (Day 4) ✅
2. Inheritance: reuse a parent's code (Day 5) ✅
3. Polymorphism: one type, many behaviors (Day 6) ✅
4. **Abstraction: expose *what* something does, hide *how*** *(today)*

You use abstraction constantly. You call `list.add(x)` without knowing whether it's an `ArrayList` copying arrays or a `LinkedList` linking nodes. You only know the contract: "add puts it in the list."

## Abstract classes

An **abstract class** is a parent that's incomplete on purpose. It can't be instantiated, and it can declare **abstract methods** (no body) that every child must implement.

```java
public abstract class Account {
    protected double balance;

    public Account(double balance) {
        this.balance = balance;
    }

    public double getBalance() {           // concrete: shared by all children
        return balance;
    }

    public abstract double monthlyFee();   // abstract: each child must define it
}

public class SavingsAccount extends Account {
    public SavingsAccount(double balance) { super(balance); }

    @Override
    public double monthlyFee() { return 0; }
}

public class CurrentAccount extends Account {
    public CurrentAccount(double balance) { super(balance); }

    @Override
    public double monthlyFee() { return 200; }
}
```

- `new Account(100)` doesn't compile. A generic "account" with no fee rule makes no sense.
- A child that doesn't implement `monthlyFee()` won't compile, unless it's abstract too.
- An abstract class **can** have fields, constructors, and fully written methods. It's a partially built class.

Use one when related classes share **state and code**, but some behavior must differ.

## Interfaces

An **interface** is a pure contract: "anything that implements me can do these things." Classes `implement` it:

```java
public interface Payable {
    void pay(double amount);
}

public class CreditCard implements Payable {
    @Override
    public void pay(double amount) { /* charge the card */ }
}

public class UpiWallet implements Payable {
    @Override
    public void pay(double amount) { /* debit the wallet */ }
}
```

Code that only needs payment works with any `Payable`, through polymorphism (Day 6):

```java
void checkout(Payable method, double total) {
    method.pay(total);
}
```

Key facts:
- Interface methods are `public` and `abstract` by default.
- Interfaces can't hold instance state. Any fields are automatically `public static final` constants.
- **A class can implement many interfaces**, even though it can extend only one class (Day 5). This is how Java gives you "multiple types" without the diamond problem.

```java
public class SmartCard implements Payable, Comparable<SmartCard>, Serializable { ... }
```

## Modern interface features

- **`default` methods (Java 8+):** an interface method *with* a body. This lets you add a new method to an interface without breaking every class that already implements it. `List.sort()` and `Map.getOrDefault()` (which you use in your frequency maps) are default methods.
- **`static` methods:** helpers that belong to the interface itself, like `List.of(...)` and `Comparator.comparing(...)`.
- **`private` methods (Java 9+):** shared helper code for the default methods.

If a class implements two interfaces with the **same default method**, it must override that method and choose explicitly. Java refuses to guess, which is how it avoids the diamond problem even with default methods.

## Abstract class vs interface

| | Abstract class | Interface |
|---|---|---|
| Can a class extend/implement more than one? | No, one parent class only | Yes, many |
| Instance fields (state) | Yes | No (only constants) |
| Constructors | Yes | No |
| Methods with bodies | Yes | Only `default`, `static`, `private` |
| Relationship | "is-a" (a `SavingsAccount` *is an* `Account`) | "can-do" (a `CreditCard` *can* `pay`) |

**Rule of thumb:** default to an **interface**, since it's more flexible. Choose an **abstract class** when the related classes genuinely share state and implementation code.

## Functional interfaces (preview)

An interface with **exactly one** abstract method is a **functional interface**, and can be written as a lambda. You've already written one:

```java
freq.merge(c, 1, Integer::sum);
```

`Integer::sum` is a method reference passed as a `BiFunction`. Lambdas and streams get their own Java day later.

## Common pitfalls

- **Trying to instantiate an abstract class or interface.** Neither compiles with `new`.
- **Forgetting to implement every abstract method.** The class must implement all of them, or be declared abstract itself.
- **Reaching for an abstract class when an interface would do.** It uses up the class's only parent slot.
- **Putting state in an interface.** Interface fields are constants, not per-object data.

## Interviewer follow-ups

**"Abstract class or interface?"** Interface by default, for flexibility and because a class can implement many. Abstract class when subclasses share real state and code, such as common fields and constructor logic.

**"Why were default methods added to interfaces?"** To evolve existing interfaces without breaking every implementation. That's how Java 8 added methods like `forEach` and `sort` to collection interfaces that millions of existing classes already implemented.

## Retention Check

1. What is abstraction, in one sentence?
2. Why can't you instantiate an abstract class?
3. What happens if a subclass doesn't implement an abstract method?
4. How many interfaces can a class implement, versus how many classes it can extend?
5. Can an interface hold per-object state? What are interface fields?
6. Why were `default` methods introduced?
7. What must a class do if two interfaces give it the same default method?
8. When should you choose an abstract class over an interface?

**Key points to self-check against:**
- Exposing what something does while hiding how it does it.
- It's intentionally incomplete; its abstract methods have no implementation.
- It won't compile, unless the subclass is also declared abstract.
- Many interfaces, but only one class.
- No; interface fields are automatically `public static final` constants.
- To add methods to existing interfaces without breaking the classes that already implement them.
- Override the method and choose explicitly.
- When related classes share real state and implementation code.
