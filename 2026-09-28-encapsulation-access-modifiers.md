# Java Day 4: Encapsulation & Access Modifiers

## The four pillars of OOP (the map for the next few days)

1. **Encapsulation**: hide internal data, expose controlled access. *(today)*
2. **Inheritance**: build a new class on top of an existing one.
3. **Polymorphism**: one interface, many behaviors.
4. **Abstraction**: expose *what* something does, hide *how*.

## What problem does encapsulation solve?

Take yesterday's `BankAccount`, where the fields had no access modifier:

```java
BankAccount acc = new BankAccount("Reeya", 1000);
acc.balance = -50000;   // nothing stops this
```

Any code anywhere in the same package can put the object into a nonsense state. There's no single place to enforce rules like "balance can't go negative," and if you later change how balance is stored, every piece of code touching it directly breaks.

**Encapsulation** means: make the data **private**, and allow changes only through **methods** that enforce the rules.

```java
public class BankAccount {
    private final String owner;
    private double balance;

    public BankAccount(String owner, double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance can't be negative");
        }
        this.owner = owner;
        this.balance = initialBalance;
    }

    public double getBalance() {
        return balance;
    }

    public void withdraw(double amount) {
        if (amount <= 0 || amount > balance) {
            throw new IllegalArgumentException("Invalid withdrawal");
        }
        balance -= amount;
    }
}
```

Now `acc.balance = -50000;` doesn't compile. The only way to change the balance is `withdraw()`, which enforces the rule. The object **protects its own validity**.

(Money as `double` is kept here only for readability. As in Day 5's backend read, real money code should use `BigDecimal`.)

## The four access levels

From most open to most restricted:

| Modifier | Same class | Same package | Subclass (other package) | Anywhere |
|---|---|---|---|---|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| *(none)*, called "package-private" | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

Two things people often get wrong:
- **No modifier is not the same as `public`.** It's package-private: visible only inside the same package.
- **`protected` is wider than it sounds.** It includes the whole package, not just subclasses.

**Rule of thumb: start with `private`, and loosen only when there's a real reason.** It's easy to open access later, but hard to take it back once other code depends on it.

## Getters and setters: not automatically encapsulation

A common misconception is that adding a getter and setter for every field *is* encapsulation:

```java
private double balance;
public void setBalance(double balance) { this.balance = balance; }   // no rules at all
```

This is barely better than a public field: anyone can still set any value. Real encapsulation means exposing **meaningful operations** (`deposit`, `withdraw`) that enforce rules, not raw setters for everything. Only add a setter when changing that field freely is actually valid.

## Immutability: the strongest form of encapsulation

An **immutable** object can't change after it's created. You already know one: `String` (DSA Day 3). To make your own:

1. Make all fields `private final`.
2. Provide no setters.
3. Make the class `final` so no subclass can add mutable behavior.
4. Don't leak mutable internals (see the pitfall below).

Why bother? Immutable objects are automatically **thread-safe** (nothing can change, so nothing can race), and they're safe as `HashMap` keys (DSA Day 8: a key that changes breaks the map).

Since Java 16, `record` gives you this in one line:

```java
public record Point(int x, int y) {}
```

This generates private final fields, a constructor, getters (`x()`, `y()`), and correct `equals()`, `hashCode()`, and `toString()`. That also solves the `Point` bug from today's DSA read.

## A subtle leak: returning a mutable internal object

```java
public class Team {
    private final List<String> members = new ArrayList<>();

    public List<String> getMembers() {
        return members;   // leak!
    }
}

team.getMembers().clear();   // wipes the private list from outside
```

`private final` stops anyone from pointing `members` at a different list. But `getMembers()` hands out the **reference** to the real list (Java Day 2: copying a reference means sharing the object), so callers can still modify its contents. Fix: return an unmodifiable view or a copy, e.g. `return List.copyOf(members);`.

## Common pitfalls

- **Assuming no modifier means `public`.** It's package-private.
- **Generating getters and setters for every field by default.** That exposes the data as much as public fields do.
- **Thinking `final` makes an object immutable.** `final` stops the *reference* from changing, not the object it points to.
- **Returning references to mutable internal collections.** Callers can modify your private state through them.

## Interviewer follow-ups

**"What's the difference between encapsulation and abstraction?"** Encapsulation is about *protecting* data: making it private and controlling access. Abstraction is about *simplifying*: exposing what an object does while hiding how it does it. They work together, but encapsulation is enforced with access modifiers, while abstraction is expressed through interfaces and abstract classes (coming later).

**"Does `private final List<String> list` make the list immutable?"** No. The field can't be reassigned, but the list's contents can still change. You need an unmodifiable list, like `List.copyOf(...)` or `List.of(...)`.

## Retention Check

1. What problem does encapsulation solve? Use the `BankAccount` example.
2. List the four access levels from most to least open.
3. What does "no modifier" mean, and who can see it?
4. Why isn't a public setter for every field real encapsulation?
5. What four steps make a class immutable?
6. Why are immutable objects automatically thread-safe?
7. In the `Team` example, how can outside code clear the private list, and how do you fix it?
8. What does `final` on a field guarantee, and what doesn't it guarantee?

**Key points to self-check against:**
- It prevents outside code from putting an object into an invalid state; all changes go through methods that enforce the rules.
- `public` → `protected` → package-private (no modifier) → `private`.
- Package-private: visible only to classes in the same package.
- It lets anyone set any value, so no rules are enforced; you've only renamed direct field access.
- `private final` fields, no setters, a `final` class, no leaking of mutable internals.
- Their state can never change, so concurrent threads can't interfere with each other.
- The getter returns a reference to the real list; return a copy or an unmodifiable view instead.
- The reference can't be reassigned; the object it points to can still be modified.
