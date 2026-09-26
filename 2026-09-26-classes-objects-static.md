# Java Day 3: Classes, Objects, Constructors & Static vs Instance

## The core idea

A **class** is a blueprint. An **object** is a real thing built from that blueprint.

```java
public class BankAccount {
    String owner;        // field: data each object holds
    double balance;

    void deposit(double amount) {   // method: behavior each object has
        balance += amount;
    }
}

BankAccount a = new BankAccount();   // object 1
BankAccount b = new BankAccount();   // object 2, completely separate
```

Connecting to Java Day 2: `new BankAccount()` creates the object **on the heap**, and `a` is a reference (an address) stored **on the stack**. `a` and `b` point to two different objects, each with its own `owner` and `balance`.

## Constructors

A constructor runs once, when the object is created, to set it up. It has the same name as the class and no return type.

```java
public class BankAccount {
    String owner;
    double balance;

    public BankAccount(String owner, double balance) {
        this.owner = owner;
        this.balance = balance;
    }
}

BankAccount acc = new BankAccount("Reeya", 1000);
```

**The default constructor rule:** if you write **no** constructor at all, Java quietly gives you an empty no-argument one. The moment you write **any** constructor yourself, that free one disappears. So after adding the constructor above, `new BankAccount()` no longer compiles. This trips people up constantly, especially with frameworks like JPA/Hibernate, which require a no-argument constructor.

## `this`

`this` means "the current object": the one this method or constructor was called on.

In the constructor above, `this.owner = owner;` is needed because the parameter `owner` has the same name as the field `owner`. Inside the constructor, plain `owner` means the parameter (the closer one wins). `this.owner` explicitly means the field. Leave out `this.` and you've assigned the parameter to itself, so the field stays `null`. The compiler won't warn you.

## Static vs instance

**Instance members** belong to each object. Every object has its own copy.
**Static members** belong to the **class itself**. There is exactly one copy, shared by all objects.

```java
public class BankAccount {
    static int totalAccounts = 0;   // ONE copy, shared by every account
    String owner;                   // each account has its own

    public BankAccount(String owner) {
        this.owner = owner;
        totalAccounts++;            // every new account bumps the shared counter
    }

    static int getTotalAccounts() { // static method: called on the class
        return totalAccounts;
    }
}

new BankAccount("A");
new BankAccount("B");
System.out.println(BankAccount.getTotalAccounts());   // 2
```

**The key rule:** a static method **cannot** use instance fields or `this` directly. It isn't running on any particular object, so there's no "current object" to read `owner` from. (An instance method, on the other hand, *can* use static members, since there's only one copy and it always exists.)

This is also why `main` is `static` (Java Day 1). The JVM calls it before any object exists, and that's also why you can't call a non-static method directly from `main` without first creating an object.

## When to use static

Good uses:
- **Constants:** `static final double INTEREST_RATE = 0.04;`
- **Utility methods that don't depend on object state:** like `Math.max(a, b)` or your `isVowel(char c)` helper. Notice you already made `isVowel` static in your sliding window solutions, because it only depends on its input.
- **Counters or shared data** that genuinely belong to the class as a whole.

Bad use: making everything static just to avoid creating objects. That turns your class into a bag of global variables, which makes code hard to test and unsafe when multiple threads touch the same shared data.

## Common pitfalls

- **"Non-static method cannot be referenced from a static context."** You called an instance method from `main` (or another static method) without an object. Either create an object first, or make the method static if it really doesn't need object state.
- **Forgetting `this.` when names clash.** `owner = owner;` assigns the parameter to itself and leaves the field unset.
- **Losing the default constructor.** Adding any constructor removes the free no-arg one.
- **Mutable static fields in multithreaded code.** One shared copy means every thread sees and modifies the same value. That's a race condition waiting to happen (concurrency comes later in this track).

## Interviewer follow-ups

**"Can a static method access instance variables?"** Not directly. A static method has no `this`, because it isn't called on any object. It would need an object reference passed in or created.

**"Where are static variables stored?"** Once per class, not per object. In modern JVMs, the static fields live with the class's data on the heap (since Java 8, class metadata moved from the old "PermGen" area to "Metaspace"). The key idea for interviews: one copy per class, created when the class is loaded.

## Retention Check

1. What's the difference between a class and an object?
2. After `BankAccount a = new BankAccount("A", 0);`, what lives on the stack and what lives on the heap?
3. When does Java provide a default constructor, and when does it stop providing one?
4. What goes wrong with `owner = owner;` inside a constructor?
5. What's the difference between a static field and an instance field?
6. Why can't a static method read an instance field directly?
7. Why did your `isVowel` helper make sense as a static method?
8. What does "non-static method cannot be referenced from a static context" mean, and how do you fix it?

**Key points to self-check against:**
- A class is a blueprint; an object is an instance built from it, on the heap.
- Stack: the reference `a`. Heap: the `BankAccount` object with its fields.
- Only when you write no constructor at all; writing any constructor removes it.
- The parameter is assigned to itself; the field stays at its default (`null`).
- A static field has one copy shared by the whole class; each object has its own instance fields.
- A static method isn't called on any object, so there's no `this` to read instance fields from.
- It depends only on its input character, not on any object's state.
- You called an instance method without an object; create an object first, or make the method static if it doesn't need object state.
