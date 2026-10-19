# Lecture 10 — Exception handling: cleanup (§9.6), try-with-resources, and resource safety

**Date:** Tue 20 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — Ch.9 Exception Handling (pp.119–134; read **§9.6** today; **try-with-resources** is modern Java — not in the 2008 book, scheduled **Tue** per `CURRICULUM.md` Week 3)

**Time plan (1 h):** 10 min recap (Mon ch.9 §9.1–9.5) · 20 min read this lecture + book §9.6 · 30 min hands-on (see §12 — curriculum **#2** and/or **#3**; problem statements in `CURRICULUM.md` only).

**Goal for today:** understand why **cleanup** must run even when operations fail; treat the book’s **finalization** discussion (§9.6) as **historical context** and prefer **`try-with-resources`** for anything that implements **`AutoCloseable`**; connect **checked `IOException`** to **closing streams and files** so curriculum **#3** and Wed’s ch.10 read feel natural — without leaking handles or swallowing causes.

> **Pacing note:** Week 3 in `CURRICULUM.md` schedules **Mon = ch.9 §9.1–9.5**, **Tue = §9.6 + try-with-resources** (this file), **Wed = ch.10 streams & files**, **Thu = `java.nio.file` + `Scanner`**, **Fri = Spring Boot "hello REST"**, **Sat = REST + error handling**. Do not invent schedule changes inside this lecture.

---

## 1. Recap hook (Mon 19 Oct — use the first 10 minutes)

Before reading §9.6, from memory in `notes/`:

1. **Three bullets from yesterday:** e.g. **checked** vs **unchecked** and when the compiler forces `throws`/`catch`; **`try` / `catch` / `finally`** execution order; **multi-catch** when recovery is identical; **custom exceptions** + **chaining** with `getCause()`.
2. **One self-check from Lecture 9:** e.g. why **subclass** `catch` clauses must come before superclass catches, or what **exception chaining** preserves for log readers.
3. **Skim** your curriculum **#2** work (or scratch) — did your **checked** vs **unchecked** rationale still feel right after a night’s sleep? Note one sentence to tighten in `notes/`.

Mon taught **failure as objects** and **handler placement**. Tue adds **failure while holding scarce resources** (open files, sockets, DB connections later in Week 4): the JVM will not fix forgotten cleanup for you — your structure must.

*(Book ch.13–15 — GUI, applets, servlets — remain **skipped**; see `CURRICULUM.md` [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose).)*

---

## 2. Why cleanup belongs in the exception story (§9.6; pp.130–132)

Many operations **acquire resources** that must be **released** whether the happy path finishes or an exception flies out:

| Resource (examples) | Risk if not closed |
|---|---|
| File streams (`InputStream`, `Reader`, …) | Locked files, descriptor leaks |
| `Connection` / `Statement` (JDBC, Week 4) | Pool exhaustion, DB-side locks |
| Network sockets | Hung connections |

The book’s §9.6 discusses **finalization** — the idea that the garbage collector might eventually run a **`finalize()`** method on an unreachable object so it can release native handles.

**Modern stance (paraphrase curriculum + JDK direction):**

- **`finalize()` is deprecated for removal** — timing is **non-deterministic**, ordering is fragile, and it **does not** replace explicit close in application code.
- **Week 6** revisits §9.6 again as a **historical contrast** with **`Cleaner`** and leak patterns; today’s job is the **simple rule**: *close what you open, in a `finally` or (preferably) try-with-resources*.

**Link to Mon:** `finally` was introduced as “runs almost always” cleanup. §9.6 explains *why* the language authors cared about cleanup at all; **try-with-resources** (below) is the syntax Java added so you do not hand-write error-prone `finally` blocks for every stream.

---

## 3. Manual cleanup with `try` / `finally` (bridge from Mon)

Before automatic resource management, the pattern looked like:

```java
InputStream in = null;
try {
    in = openStream();
    use(in);
} catch (IOException ex) {
    handle(ex);
} finally {
    if (in != null) {
        try {
            in.close();
        } catch (IOException closeEx) {
            // often logged; easy to get wrong
        }
    }
}
```

Pain points (why Tue exists):

1. **`null` checks** and reassignment bugs.
2. **`close()` itself throws** — nested try/catch in `finally` obscures the original failure.
3. **Multiple resources** — copy-paste and ordering mistakes (close in reverse order of open).
4. **Return or throw from `try`/`catch`** — `finally` still runs, but readability suffers.

You should **recognize** this shape in older code and book listings; new code in this lab should prefer **try-with-resources** unless you have a rare reason not to.

---

## 4. Try-with-resources (Java 7+; curriculum Tue topic)

Any type that implements **`AutoCloseable`** (including **`Closeable`** streams) can appear in a **try-with-resources** header:

```java
try (InputStream in = openStream();
     OutputStream out = openOut()) {
    copyBytes(in, out);
} catch (IOException ex) {
    throw new DataCopyException("Copy failed", ex);  // chain, Mon §6.3
}
// in.close() and out.close() run automatically — normal or exceptional exit
```

Rules to remember:

| Rule | Detail |
|---|---|
| **Declaration in parentheses** | One or more resources, separated by `;` |
| **Implicit `final`** | Resource variables are effectively final in the block |
| **Close order** | Closed in **reverse** order of declaration |
| **`close()` exceptions** | If both body and `close()` throw, body exception is primary; close exception is **suppressed** (attach with `addSuppressed`, visible on stack trace) |
| **Checked exceptions** | `close()` on many streams declares **`IOException`** — still part of the API contract |

**Custom resources:** your own types can implement `AutoCloseable` and release locks, temp files, or pooled connections in `close()` — useful for domain objects that wrap I/O (keep `close()` idempotent where possible).

**vs `finally`:** try-with-resources is **`finally` + boilerplate + suppression rules**, generated by the compiler. Curriculum **#3** asks you to copy a file **using try-with-resources**, then rewrite with **`Files.copy`** — same week, different API layer (Thu NIO.2 goes deeper).

---

## 5. Relating cleanup to checked vs unchecked (Mon + Tue)

| Situation | Typical exception shape |
|---|---|
| **Opening / reading / closing files** | **`IOException`** (checked) — callers plan or propagate |
| **Bad argument to your API** | **`IllegalArgumentException`** or your **unchecked** subtype (**#2**) |
| **Business rule the caller should handle** | Your **checked** subtype (**#2**) |
| **Wrap low-level I/O at a service boundary** | Catch `IOException`, throw domain exception **with cause** |

When you implement **#3**, decide whether your utility **declares `throws IOException`** or wraps into an unchecked domain type — either is valid if **documented** and **consistent** with Week 3 proof code.

---

## 6. Pitfalls specific to resources (feeds §10)

1. **Relying on GC / finalize** — files stay locked until explicit close.
2. **Swallowing `close()` failures** — empty inner catch in manual `finally`; prefer try-with-resources or log suppressed exceptions.
3. **Opening without try-with-resources** in new lab code — reviewers will flag it.
4. **Single resource, long method** — still use try-with-resources; scope stays tight.
5. **Forgetting that `Files.copy` also uses internal resources** — API is higher-level; **#3** timing comparison is about convenience and implementation, not permission to ignore errors.
6. **Chaining dropped on wrap** — `throw new MyException("copy failed")` without `ex` when catching **`IOException`** (repeat Mon pitfall in I/O context).

---

## 7. Free resources (Week 3 — links only)

**Tue focus:** cleanup + try-with-resources. Full Week 3 list lives in `CURRICULUM.md`; use these today:

- [Oracle Java Tutorials — Exceptions (try-with-resources)](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)
- [Oracle Java Tutorials — Basic I/O](https://docs.oracle.com/javase/tutorial/essential/io/) (preview for **Wed** ch.10)
- Baeldung: [Java try-with-resources](https://www.baeldung.com/java-try-with-resources)

*(NIO.2 `Files`, Spring error handling: same **Week 3** curriculum list — scheduled **Wed–Sat** per daily split.)*

---

## 8. Glossary

cleanup · resource · finalization · `finalize()` (deprecated) · `AutoCloseable` · `Closeable` · try-with-resources · suppressed exception · `addSuppressed` · descriptor leak · `IOException` · explicit close · reverse close order

---

## 9. Common pitfalls (combined Mon + Tue)

1. **Catching `Exception` too early** — hides programming bugs (Mon).
2. **Empty catch** — swallows I/O failures (Mon + Tue).
3. **Manual `finally` close** when try-with-resources fits — noise and bug surface (Tue).
4. **Wrong checked/unchecked for domain rules** — curriculum **#2** rationale should still hold (Mon).
5. **No cause when wrapping `IOException`** — loses line-level detail for **#1** / **#3** (Mon).
6. **Assuming `Files.copy` means “no exceptions to handle”** — still throws **`IOException`** (Tue/**#3**).
7. **Reading ch.10 streams before Wed** — skim Oracle *Basic I/O* only if time; **Wed** is the primary ch.10 day per curriculum.

---

## 10. Self-check (answer in `notes/`, then check against the book / Oracle tutorial)

1. Why is **`finalize()`** a poor substitute for **`close()`** on a file stream?
2. In try-with-resources, if the `try` block throws and `close()` also throws, which exception do you usually see first, and where does the other go?
3. Name two JDK types that implement **`AutoCloseable`** you expect to use this week.
4. What problem does **`finally`** solve that a plain **`try`/`catch`** does not?
5. For curriculum **#3**, what would you **measure** when comparing your copy utility to **`Files.copy`**?

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 11. Hands-on (Tue — curriculum prompts only; you write all code)

Week 3 proof set: **five prompts** in `CURRICULUM.md` Week 3 (**#1–#5**). Problem statements live in the curriculum only — **no solution code** in this lecture.

**Tue 20 Oct (after §9.6 + try-with-resources):** per the suggested daily split, today’s focus is curriculum **#2** and/or **#3**. Finish **#2** if Mon left it incomplete; **#3** needs today’s resource patterns.

### Curriculum **#2** (problem statement only)

Define your own **checked exception** and your own **unchecked exception** for a **domain rule** (e.g. insufficient funds, invalid email). Write a **short rationale** for each choice (in `notes/` or beside the exercise).

### Curriculum **#3** (problem statement only)

Write a utility that **copies a file** using **try-with-resources**, then rewrite it with **`Files.copy`**. **Time both** on a large file and note differences (in `notes/`).

| Other prompts | Tue guidance |
|---|---|
| **#1** | Continue if started Mon; full stream/file depth aligns with **Wed–Thu** ch.10 / NIO.2 |
| **#4–#5** | **Fri–Sat** — Spring Boot REST and centralized error handling |

Work in scratch under e.g. `exercises/2026-10-20-ch9-try-with-resources/` or extend Mon’s folder until you consolidate into Week 3 proof paths per curriculum **Proof artifact**.

Closing habit (2 min): one line in `notes/` — what clicked / what confused you about **automatic close** or **`IOException`** handling.

---

## 12. Bridge to Wed (per `CURRICULUM.md` Week 3)

**Tomorrow:** book ch.10 **Input and Output Operations** — **byte vs character streams**, files, and the stream stack that **`try-with-resources`** closes for you (focus **§10.1–10.4**, **10.7–10.10** per curriculum; skim **10.5–10.6** and **10.11**). Curriculum **#1** (malformed number lines) and **#3** (copy) both sit on the same I/O foundation; Wed makes the **stream types** explicit.

**Later this week:** **Thu** adds **`java.nio.file`** (`Path`, `Files`) and **`Scanner`**; **Fri–Sat** maps failures to **HTTP** status codes and **`@RestControllerAdvice`**. Your **#2** exception choices should still align with “client error” vs “server error” when you reach REST.

Do not jump to servlet chapters (book ch.15 skipped) or Swing (ch.13); the web path stays **Spring Boot** per `CURRICULUM.md`.
