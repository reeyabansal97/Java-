# Java Day 1: How Java Actually Runs — JDK, JRE, JVM & Bytecode

## Why start here?

You've written plenty of Java. But interviewers love asking what happens *between* typing `java MyApp` and your code running, and many experienced developers can't explain it cleanly. Everything later in this track (memory, garbage collection, class loading, performance) builds on this picture.

## The journey of a Java program

```
MyApp.java  --(javac)-->  MyApp.class  --(java / JVM)-->  running program
 source code             bytecode                        machine instructions
```

**Step 1: Compile.** `javac MyApp.java` turns your source code into **bytecode**, stored in `MyApp.class`. Bytecode is not machine code for any real CPU. It's a set of instructions for an imaginary machine: the Java Virtual Machine.

**Step 2: Run.** `java MyApp` starts a JVM, which loads `MyApp.class` and executes that bytecode on whatever real hardware you're on.

This two-step design is the whole secret behind Java's famous slogan, **"write once, run anywhere."** You compile once to bytecode, and that same `.class` file runs on Windows, Linux, or macOS, because each platform has its own JVM that knows how to execute bytecode on that hardware.

## JDK vs JRE vs JVM

These three get mixed up constantly. Think of them as nested boxes:

| Name | What it is | Who needs it |
|---|---|---|
| **JVM** (Java Virtual Machine) | The engine that executes bytecode | Every Java program, at runtime |
| **JRE** (Java Runtime Environment) | JVM + the standard class libraries (`java.lang`, `java.util`, ...) | Anyone who only *runs* Java programs |
| **JDK** (Java Development Kit) | JRE + development tools (`javac`, `jar`, `javadoc`, debugger) | Developers who *write* and *compile* Java |

So: **JDK ⊃ JRE ⊃ JVM.** As a developer, you install the JDK, which includes everything else.

(Side note: since Java 11, Oracle stopped shipping a separate standalone JRE download. You'll still hear the term constantly, and it's still a common interview question, so the concept is worth knowing.)

## What happens inside the JVM (simplified)

1. **Class loader** finds and loads `.class` files into memory, only when they're first needed.
2. **Bytecode verifier** checks the bytecode is safe and well-formed (e.g. no illegal memory access) before running it.
3. **Execution engine** runs the bytecode, using two strategies:
   - **Interpreter:** reads and executes bytecode one instruction at a time. Starts fast, but runs slowly.
   - **JIT compiler (Just-In-Time):** watches for "hot" code that runs often (like a loop executing millions of times) and compiles it into native machine code for your specific CPU. Slower to start, but much faster once running.

This is why Java programs often get faster after they've been running for a while: the JIT is optimizing the hot paths.

## Anatomy of `main`

```java
public class MyApp {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

Every keyword has a reason:
- `public`: the JVM calls this from outside your class, so it must be accessible.
- `static`: the JVM calls it *before* any object of `MyApp` exists, so it can't require an instance.
- `void`: it returns nothing to the JVM (exit codes go through `System.exit(...)` instead).
- `String[] args`: command-line arguments passed after the class name.

## Common pitfalls

- **Public class name must match the file name.** A `public class MyApp` must live in `MyApp.java`, or `javac` refuses to compile it.
- **Running with the `.class` extension.** It's `java MyApp`, not `java MyApp.class`. You give the JVM a class name, not a file name.
- **Confusing "Java is platform independent" with "the JVM is platform independent."** Bytecode is platform independent. The JVM is *not*; each OS needs its own JVM build.

## Interviewer follow-ups

**"Is Java compiled or interpreted?"** Both. Source is compiled to bytecode by `javac`. At runtime, the JVM interprets bytecode and JIT-compiles the hot parts into native machine code.

**"Why is `main` static?"** The JVM needs to call it before any object of the class exists. A non-static method would need an instance, and there's nothing to create one yet.

## Retention Check

1. What does `javac` produce, and why isn't it regular machine code?
2. Put JDK, JRE, and JVM in order from largest to smallest, and say what each adds.
3. What makes "write once, run anywhere" possible?
4. What is the difference between the interpreter and the JIT compiler?
5. Why do Java programs often speed up after running for a while?
6. Why must `main` be `static`?
7. Is the JVM itself platform independent? Explain.
8. Would `java MyApp.class` work? Why or why not?

**Key points to self-check against:**
- Bytecode in a `.class` file; it targets the JVM, not any real CPU.
- JDK (adds dev tools like `javac`) ⊃ JRE (adds standard libraries) ⊃ JVM (executes bytecode).
- Bytecode is the same everywhere; each platform has its own JVM that runs it.
- The interpreter executes bytecode instruction by instruction; the JIT compiles frequently-run code into native machine code.
- The JIT compiler optimizes hot code paths into native code once it has observed them running.
- The JVM calls it before any instance of the class exists.
- No. Bytecode is platform independent; each OS needs its own JVM build.
- No. The `java` command takes a class name, not a file name: `java MyApp`.
