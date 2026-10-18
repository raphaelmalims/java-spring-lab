# Lecture 9 — Exception handling: terminology, checked vs unchecked, try/catch/finally, multi-catch, custom exceptions, chaining

**Date:** Mon 19 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — Ch.9 Exception Handling (pp.119–134; read **§9.1–9.5** today; **§9.6** and **try-with-resources** are **Tue** per `CURRICULUM.md` Week 3)

**Time plan (1 h):** 10 min recap (Week 2) · 20 min read this lecture + book §9.1–9.5 · 30 min hands-on (see §12 — curriculum **#2** primary; optional start **#1**; problem statements in `CURRICULUM.md` only).

**Goal for today:** use exception **terminology** precisely, distinguish **checked** from **unchecked** exceptions and justify when each fits, handle failures with **`try` / `catch` / `finally`**, use **multi-catch** where it simplifies code, define **custom exceptions**, and **chain** causes so callers and logs tell a full story — without letting errors crash the whole program when you can recover or report.

> **Pacing note:** Week 3 in `CURRICULUM.md` schedules **Mon = ch.9 §9.1–9.5** (this file), **Tue = §9.6 + try-with-resources**, **Wed = ch.10 streams & files**, **Thu = `java.nio.file` + `Scanner`**, **Fri = Spring Boot "hello REST"**, **Sat = REST + error handling**. Do not invent schedule changes inside this lecture.

---

## 1. Recap hook (Week 2 — use the first 10 minutes)

Before reading §9.1, from memory in `notes/`:

1. **Three bullets from last week (Sat review):** e.g. when to reach for **record** vs **class**; **sealed interface + records** for closed variants; **`week2`** proof in `exercises/week2/` + `notes/class-vs-record-vs-interface.md`.
2. **One self-check from Lecture 8:** e.g. when **dynamic binding** chooses the method implementation, or why **#3** avoids a `default` branch on a complete sealed `switch`.
3. **Skim** your last compile/test error from **`exercises/week2/`** or scratch — note one pattern you fixed (visibility, binding, streams) that might reappear when exceptions wrap I/O later this week.

Week 2 taught you to **label types deliberately** (class, record, interface, enum, static utility). Week 3 adds **failure paths**: the same domain types (`BankAccount`, orders, todos) now need rules for *what happens when input or state is invalid* — often expressed as **exceptions** or structured error responses (Spring, Fri/Sat).

*(Book ch.13–15 — GUI, applets, servlets — remain **skipped**; see `CURRICULUM.md` [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose).)*

---

## 2. Exception terminology (§9.1; pp.119–121)

An **exception** is an object that represents a **disruptive condition** during program execution — something that prevents the normal next step from completing safely (invalid argument, missing resource, violated business rule, I/O failure).

Core vocabulary (book + Java):

| Term | Meaning |
|---|---|
| **Throw** | Code **creates and throws** an exception object (`throw new SomeException("…")`) to signal the condition. |
| **Catch** | A **`catch`** block **handles** an exception of a given type (or subtype) after a **`try`** block. |
| **Propagate** | If no handler catches the exception, it **unwinds** the call stack — each frame’s `finally` runs, then the caller gets a chance to catch, until the thread dies or a top-level handler catches it. |
| **Handler** | The `catch` block (or caller’s catch) that runs recovery, logging, or rethrow logic. |
| **Stack trace** | The JVM’s report of **which methods were active** when the exception was thrown — essential for debugging. |

**Normal control flow vs exceptional flow:** most methods express the **happy path**; exceptions express **“cannot continue this operation as requested.”** That is not the same as using exceptions for ordinary branching on expected cases (see pitfalls §11).

**Link to OOP:** exceptions are **objects** (usually subclasses of `Throwable`). You can attach messages, custom fields (in your own exception classes), and **causes** (§9.5).

---

## 3. The exception hierarchy: checked vs unchecked (§9.2; pp.121–124)

Java splits exceptions into two families the **compiler** treats differently:

```
Throwable
├── Error (e.g. OutOfMemoryError)     — serious JVM problems; do not catch for “business logic”
└── Exception
    ├── RuntimeException (unchecked)  — compiler does not require declare/catch
    │   └── IllegalArgumentException, NullPointerException, …
    └── other Exception (checked)     — compiler enforces handling or declaration
        └── IOException, SQLException, …
```

| | **Checked exception** | **Unchecked exception** |
|---|---|---|
| **Supertype** | `Exception` but **not** `RuntimeException` | `RuntimeException` (and `Error`) |
| **Compiler** | Callers must **catch** or **declare** (`throws`) | No `throws` requirement |
| **Typical meaning** | **Recoverable external** condition caller should plan for (file missing, network down) | **Programming bug** or **precondition violation** (null where forbidden, bad index) |
| **Book emphasis** | API contracts: “this operation may fail in ways you must acknowledge” | Often “should not happen if caller uses API correctly” |

**Custom types:** you usually extend **`Exception`** for checked custom types and **`RuntimeException`** for unchecked custom types (§9.5).

**Design tension (paraphrase ch.9):** overusing checked exceptions forces every caller to boilerplate; overusing unchecked hides recoverable failures. Curriculum **#2** asks you to **define one of each** for a **domain rule** and write a **short rationale** — that is today’s primary hands-on focus.

**Modern note (not in 2008 book):** many libraries and Spring APIs lean on **unchecked** exceptions for programming errors and wrap I/O in runtime wrappers in some stacks; you still need to **know** checked rules for `IOException` and JDBC (Week 4). This week’s **#1** and Wed–Thu I/O days will mix **catch** vs **propagate** explicitly.

---

## 4. `try`, `catch`, and `finally` (§9.3; pp.124–128)

### 4.1 Basic shape

```java
try {
    // code that may throw
} catch (SpecificException ex) {
    // handle or recover
} finally {
    // almost always runs (cleanup)
}
```

- **`try`:** encloses statements that might throw.
- **`catch`:** runs when a thrown exception **matches** the catch parameter type (assignment compatible).
- **`finally`:** runs **whether or not** an exception occurred — use for **cleanup** that must happen (close in older style; **try-with-resources** on **Tue** is the modern default for closable resources).

### 4.2 Execution order (mental model)

1. Enter `try`.
2. If something throws, matching **`catch`** runs (if any); if no match, propagate after `finally`.
3. **`finally`** runs before control leaves the `try`/`catch` region (normal return, catch return, or propagate).
4. If both `catch` and `finally` exist, **`finally`** still runs after `catch` completes unless the JVM exits abruptly.

**Pitfall preview:** returning from `try` or `catch` while `finally` also returns — `finally` wins; avoid relying on that (§11).

### 4.3 Multiple catch blocks

You may list several **`catch`** clauses for **different** exception types. Order matters: **more specific types before general ones** (subclass before superclass), or the compiler rejects unreachable catches.

### 4.4 Declaring `throws`

When you **do not** catch a checked exception inside the method, the method **signature** must declare it:

```java
void readConfig() throws IOException {
    // caller must handle or declare further
}
```

That is how checked exceptions become part of the **API contract**. Curriculum **#1** (file of numbers) asks you to **decide where to catch and where to propagate** — you can start sketching that design today even if full I/O lands Wed.

---

## 5. Multi-catch (§9.4; pp.128–130)

**Multi-catch** (Java 7+) lets one **`catch`** handle **several unrelated exception types** with the same recovery logic:

```java
try {
    // ...
} catch (IOException | SQLException ex) {
    // shared handler — ex is effectively final
    logFailure(ex);
}
```

Rules to remember:

- Types are separated by **`|`**; the caught parameter type is the **common supertype** for the block (often `Exception` if the union has no tighter shared type — prefer not to over-generalize).
- The exception variable in multi-catch is **implicitly final** — you cannot assign to `ex` inside the block.
- Use multi-catch when handlers **duplicate** the same five lines; keep **separate** catches when recovery **differs** by type.

Book tie-in: multi-catch reduces noise when several failure modes share one recovery path (log and abort, retry once, substitute default).

---

## 6. Custom exceptions and exception chaining (§9.5; pp.130–134)

### 6.1 Why define your own exception types?

A **domain-specific** exception class makes **catch** clauses precise and documents **business rules** in the type system:

- **`InsufficientFundsException`** (example shape) — withdrawal would violate balance rules.
- **`InvalidEmailFormatException`** — string fails validation before persistence.

Minimal patterns:

```java
// Checked — caller must acknowledge
public class InsufficientFundsException extends Exception {
    public InsufficientFundsException(String message) {
        super(message);
    }
}

// Unchecked — signals programmer error or unrecoverable rule break
public class InvalidEmailFormatException extends RuntimeException {
    public InvalidEmailFormatException(String message) {
        super(message);
    }
}
```

Add constructors that take **`cause`** when wrapping lower-level failures (chaining below).

### 6.2 Checked vs unchecked for *your* domain (feeds curriculum **#2**)

Ask:

1. **Can a reasonable caller recover or retry?** → often **checked** (or return a result type in modern APIs — out of scope today).
2. **Is it a bug or violated precondition on bad input?** → often **unchecked** (`IllegalArgumentException` or your subtype).
3. **Will every caller need the same handling?** If only top-level handles it, unchecked + centralized handler (Spring **Sat**) may suffice later.

Write the **rationale in `notes/`** in plain language — curriculum **#2** requires it.

### 6.3 Exception chaining (cause)

When you catch a low-level exception and throw a **higher-level** one, preserve the original with **`initCause`** or the **`Throwable(String message, Throwable cause)`** constructor:

```java
try {
    lowLevelOperation();
} catch (IOException ex) {
    throw new DataLoadException("Failed to load user profile", ex);
}
```

Effects:

- Callers see **your** message and type at the boundary.
- Logs and stack traces retain the **root cause** (`getCause()`).
- Avoid **empty** catch blocks that swallow the original — always log or chain.

Chaining matters for curriculum **#1** when a malformed line wraps parsing errors: report **line number** at your layer, keep **cause** for detail in logs.

---

## 7. Relating exceptions to Week 2 constructs

| Construct | Exception role |
|---|---|
| **Class** | Throw from methods that enforce **mutable invariants** (account balance, state machine). |
| **Record** | Often validate in **compact constructor**; throw **unchecked** for impossible states at construction. |
| **Interface** | Interface methods may **declare** checked exceptions; implementations must honor the contract. |
| **Static utility** | Pure parsers/validators throw or return — if they throw, document checked vs unchecked clearly. |

Sat Week 3 will map failures to **HTTP status codes** and **`@ExceptionHandler`** — today’s Java-layer choices should align with “client error” vs “server error” you will express in REST later.

---

## 8. Free resources (Week 3 — links only)

Use these alongside the book; do not paste proprietary book text into your repo. **Mon focus:** exceptions; I/O and Spring links are for later days in the same week list.

- [Oracle Java Tutorials — Exceptions](https://docs.oracle.com/javase/tutorial/essential/exceptions/)
- [Oracle Java Tutorials — Essential Java Classes](https://docs.oracle.com/javase/tutorial/essential/) (exceptions live under this trail)
- Baeldung: [Exception Handling in Java](https://www.baeldung.com/java-exceptions)

*(Try-with-resources, NIO.2, and Spring error handling: same **Week 3** list in `CURRICULUM.md` — Oracle *Basic I/O*, *File I/O (NIO.2)*, Baeldung try-with-resources / `java.nio.file`, Spring *Building a RESTful Web Service* and *Error handling* — scheduled **Tue–Sat** per daily split.)*

---

## 9. Glossary

exception · throw · catch · propagate · handler · stack trace · checked exception · unchecked exception · `RuntimeException` · `Error` · `try` · `finally` · `throws` · multi-catch · custom exception · exception chaining · cause · `getCause()` · recoverable vs programming error

---

## 10. Common pitfalls

1. **Catching `Exception` or `Throwable` too early** — hides bugs; catch **specific** types first.
2. **Empty catch blocks** — swallows failures; at minimum **log** or **chain** the cause.
3. **Using exceptions for normal control flow** — e.g. loop until success by catching end-of-file; prefer return values or iterators where appropriate.
4. **Wrong checked/unchecked choice** — forcing callers to handle “email has no @" as checked everywhere; curriculum **#2** rationale should defend your choice.
5. **Catch order wrong** — superclass before subclass → compile error or unreachable handler.
6. **Losing the cause** — `throw new MyException("fail")` without `ex` when wrapping **IOException**.
7. **Finally that throws** — masks original exception; keep `finally` minimal.
8. **Declaring `throws Exception`** on public API — too vague; declare specific checked types.
9. **Ignoring stack trace in `#1` prep** — when reporting bad lines, include **line number** in *your* message; use cause for parse detail.
10. **Reading ahead into try-with-resources today** — **Tue §9.6**; today master manual **try/finally** cleanup concept so Tue’s syntax makes sense.

---

## 11. Self-check (answer in `notes/`, then check against the book)

1. What is the difference between **throwing** and **propagating** an exception?
2. Name one **checked** and one **unchecked** exception type from the JDK (not your custom types).
3. When does a **`finally`** block **not** run? (Think JVM exit vs return paths.)
4. Why must **subclass** `catch` clauses appear **before** superclass catches?
5. What problem does **exception chaining** solve for maintainers reading logs?
6. For curriculum **#2**, state one **domain rule** you might model as checked and one as unchecked — before you code, in one sentence each.

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 12. Hands-on (Mon — curriculum prompts only; you write all code)

Week 3 proof set: **five prompts** in `CURRICULUM.md` Week 3 (**#1–#5**). Problem statements live in the curriculum only — **no solution code** in this lecture.

**Mon 19 Oct (after §9.1–9.5):** per the suggested daily split, today’s **primary** focus is curriculum **#2**; you may **start** **#1** if time remains (full file I/O depth aligns with **Wed–Thu** ch.10).

### Curriculum **#2** (primary — problem statement only)

Define your own **checked exception** and your own **unchecked exception** for a **domain rule** (e.g. insufficient funds, invalid email). Write a **short rationale** for each choice (in `notes/` or beside the exercise).

### Curriculum **#1** (optional start — problem statement only)

Write a program that reads a text file of numbers (some lines intentionally malformed), sums the valid ones, reports each bad line with its **line number**, and **never crashes**. Decide **where to catch** and **where to propagate**.

| Other prompts | Mon guidance |
|---|---|
| **#3** | **Tue+** — try-with-resources and file copy (`CURRICULUM.md` daily split) |
| **#4–#5** | **Fri–Sat** — Spring Boot REST and centralized error handling |

Work in scratch under e.g. `exercises/2026-10-19-ch9-exceptions/` or `notes/` until you consolidate into **`exercises/week3-todo-api/`** and related Week 3 proof folders per curriculum **Proof artifact**.

Closing habit (2 min): one line in `notes/` — what clicked / what confused you about **checked vs unchecked** or **chaining**.

---

## 13. Bridge to Tue (per `CURRICULUM.md` Week 3)

**Tomorrow:** book ch.9 **§9.6** plus **try-with-resources** — automatic close of `AutoCloseable` resources, and how that replaces easy-to-get-wrong manual `finally` cleanup. Revisit curriculum **#3** (file copy: manual try-with-resources vs `Files.copy`) after Tue’s read.

**Later this week:** ch.10 **I/O** (Wed–Thu) completes the story for **#1**; **Spring Boot** (Fri–Sat) maps exceptions to **HTTP responses** and **`@RestControllerAdvice`** — your **#2** type choices should feel compatible with “400 bad input” vs “404 not found” vs “500 unexpected.”

Do not jump to servlet chapters (book ch.15 skipped) or Swing (ch.13); the web path stays **Spring Boot** per `CURRICULUM.md`.
