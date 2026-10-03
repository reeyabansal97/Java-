# Java Day 8: Exception Handling

## What is an exception?

An exception is Java's way of saying "something went wrong, and normal flow can't continue." You've already met several:
- `ArrayIndexOutOfBoundsException` (your missing-number bug, DSA Day 2)
- `NullPointerException` (unboxing `null`, Java Day 2)
- `NoSuchElementException` (popping an empty `ArrayDeque`, DSA Day 10)
- `ArithmeticException` (modulo by zero, in your vowels problem)

When an exception is thrown, Java stops the current method and **unwinds the call stack** (Java Day 2, DSA Day 6), looking for code that handles it. If nothing does, the thread dies and prints the stack trace.

## The hierarchy

```
Throwable
├── Error                    (serious JVM problems; don't catch these)
│     e.g. OutOfMemoryError, StackOverflowError
└── Exception
      ├── checked exceptions (e.g. IOException, SQLException)
      └── RuntimeException   (unchecked)
            e.g. NullPointerException, IllegalArgumentException,
                 ArrayIndexOutOfBoundsException
```

## Checked vs unchecked: the key distinction

**Checked exceptions** (subclasses of `Exception`, but not of `RuntimeException`): the compiler **forces** you to handle them, either by catching them or by declaring them with `throws`. They represent problems a well-written program should expect and recover from, like a file being missing or a network failure.

```java
public String readConfig(String path) throws IOException {   // must declare
    return Files.readString(Path.of(path));
}
```

**Unchecked exceptions** (`RuntimeException` and its subclasses): the compiler doesn't force anything. They usually signal **bugs**, such as `null` where it shouldn't be, a bad index, or an invalid argument. The fix is usually correcting the code, not catching the exception.

**`Error`s** (`OutOfMemoryError`, `StackOverflowError` from your recursion read) mean the JVM itself is in trouble. Don't catch them.

## try / catch / finally

```java
try {
    riskyOperation();
} catch (FileNotFoundException e) {
    // handle the specific case
} catch (IOException e) {
    // handle the broader case
} finally {
    // ALWAYS runs: success, exception, or even a return inside try
}
```

- **Order catch blocks from most specific to most general.** `FileNotFoundException` is a subclass of `IOException`. If `IOException` came first, it would catch everything, and the compiler rejects the unreachable specific block.
- **`finally` always runs.** That's why it was traditionally used for cleanup, like closing files and connections.
- **Multi-catch:** `catch (IOException | SQLException e)` handles several unrelated types with one block.

## try-with-resources: the modern way to clean up

You saw this in the connection pooling read (Day 10):

```java
try (Connection conn = dataSource.getConnection();
     PreparedStatement ps = conn.prepareStatement(sql)) {
    // use them
}   // both closed automatically, in reverse order, even if an exception is thrown
```

Any class that implements `AutoCloseable` works here. It replaces the error-prone manual `finally { conn.close(); }` pattern, which is exactly how connection leaks happen.

## Throwing your own

```java
public void withdraw(double amount) {
    if (amount <= 0) {
        throw new IllegalArgumentException("Amount must be positive: " + amount);
    }
    // ...
}
```

This is the encapsulation pattern from Java Day 4: reject invalid input loudly, at the boundary. Custom exceptions are just classes:

```java
public class InsufficientFundsException extends RuntimeException {
    public InsufficientFundsException(String message) {
        super(message);
    }
}
```

**Wrapping exceptions:** when you catch a low-level exception and throw a higher-level one, pass the original along as the **cause**, so the stack trace keeps the root problem:

```java
catch (SQLException e) {
    throw new OrderSaveException("Could not save order " + orderId, e);   // 'e' preserved
}
```

## The Spring connection

Two connections back to earlier reads:
- **`@Transactional` rolls back only on unchecked exceptions by default** (Day 5). A checked exception lets the transaction commit, unless you use `rollbackFor = Exception.class`. Now you know what "checked" and "unchecked" mean there.
- **`@ControllerAdvice` + `@ExceptionHandler`** turn exceptions into clean HTTP responses in one place, instead of `try/catch` in every controller:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(InsufficientFundsException.class)
    public ResponseEntity<String> handle(InsufficientFundsException e) {
        return ResponseEntity.badRequest().body(e.getMessage());   // 400, not 500
    }
}
```

## Common pitfalls

- **Swallowing exceptions:** `catch (Exception e) {}` hides real failures. At minimum, log them.
- **Catching `Exception` (or `Throwable`) everywhere.** It catches bugs you should fix, and can hide `Error`s.
- **Losing the cause** when wrapping. Always pass the original exception into the new one.
- **Using exceptions for normal control flow.** They're slower and harder to read than a simple `if`.
- **Returning from `finally`.** It silently discards any exception thrown in `try`.

## Interviewer follow-ups

**"Checked vs unchecked exceptions?"** Checked ones must be caught or declared, and represent recoverable, expected conditions like I/O failures. Unchecked ones (`RuntimeException`) aren't enforced by the compiler, and usually indicate programming bugs.

**"Does `finally` always run?"** Essentially yes: after normal completion, after an exception, and even after a `return` in `try`. The exceptions are extreme cases, like `System.exit()` or the JVM crashing.

## Retention Check

1. What happens to the call stack when an exception is thrown and not caught?
2. What's the difference between checked exceptions, unchecked exceptions, and `Error`s?
3. Why must specific catch blocks come before general ones?
4. When does `finally` run?
5. What does try-with-resources do, and what must a class implement to use it?
6. Why should you pass the original exception as the cause when wrapping it?
7. By default, which exceptions make `@Transactional` roll back?
8. Why is `catch (Exception e) {}` dangerous?

**Key points to self-check against:**
- Java unwinds the stack looking for a handler; if none exists, the thread ends and prints the stack trace.
- Checked: must be caught or declared, recoverable conditions. Unchecked: `RuntimeException`, usually bugs. `Error`: serious JVM problems, not to be caught.
- A general catch placed first would catch everything, making the specific block unreachable (a compile error).
- Always: after success, after an exception, and even after a `return` in `try` (barring extreme cases like `System.exit`).
- It closes resources automatically at the end of the block, even on exceptions; the class must implement `AutoCloseable`.
- So the stack trace still shows the root cause.
- Unchecked exceptions (`RuntimeException` and `Error`).
- It silently hides real failures, including bugs you should fix.
