# Lecture 7 — Lambdas, method references, functional interfaces, `Optional`, and Stream API basics

**Date:** Fri 16 Oct 2026 · **Book:** none (modern Java track per `CURRICULUM.md` Week 2 — book stops at ch.8; **ch.13–15 skipped** on purpose)

**Time plan (1 h):** 10 min recap (Thu ch.8 + **#2** loop versions if started) · 20 min read this lecture + free resources below · 30 min hands-on (see §13 — curriculum **#2** primary; problem statements in `CURRICULUM.md` only).

**Goal for today:** express behaviour as **lambdas** and **method references**, recognize **functional interfaces** (Tue §6.8), use **`Optional`** for absent values without `null` sprawl, and answer list questions with the **Stream API** (`filter`, `map`, `collect`, `reduce`) — then compare stream vs loop **readability** in `notes/` for curriculum **#2**.

> **Pacing note:** Week 2 in `CURRICULUM.md` schedules **Mon = ch.6 §6.1–6.7**, **Tue = §6.8 interfaces**, **Wed = ch.7**, **Thu = ch.8**, **Fri = lambdas & streams** (this file), **Sat = decision guide + review**. Do not invent schedule changes inside this lecture. **Sealed interfaces** (curriculum **#3**) are introduced here as modern context; full **#3** hands-on can continue today or Sat if time is tight.

---

## 1. Recap hook (Thu 15 Oct — use the first 10 minutes)

Before today’s modern track, from memory in `notes/`:

1. **Three bullets from yesterday:** e.g. **package** = namespace + directory layout; **package-private** hides types within a module; **encapsulation** = invariants behind a narrow **`public`** API.
2. **One self-check from Lecture 6:** e.g. why a subclass in **another package** cannot access a **package-private** member of the superclass.
3. **Skim** curriculum **#2** — do you have **≥10 orders** (id, customer, amount, status) in named packages? If Thu left **loop** answers for totals / top 3 / unpaid check, keep those outputs — Fri repeats the **same questions** with streams for comparison.

Thu’s clean **`List<Order>`** in a **`model`** package is the input for today’s streams. Tue’s **functional interface** idea is the type lambdas implement.

---

## 2. Why lambdas and streams (post-2008 Java; curriculum Week 2)

The course book predates Java 8. Modern Java adds **first-class behaviour** (lambdas) and **declarative bulk operations** (streams) on collections.

| Without (loops + anonymous classes) | With (lambdas + streams) |
|---|---|
| Verbose `new Comparator<>() { … }` | `Comparator.comparing(Order::amount)` |
| Index/`for` loops with mutable accumulators | Pipeline: source → intermediate ops → terminal op |
| Easy to mix “what” and “how” in one block | Often clearer **intent** when the pipeline stays short |

Streams are **not** magic faster loops by default: they excel at **expressing transformations and aggregations** over in-memory data. Heavy I/O and Spring APIs come later; today stays on **`List<Order>`**-style examples.

*(Book ch.13–15 — GUI, applets, servlets — are **not** on this path; see `CURRICULUM.md` [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose).)*

---

## 3. Functional interfaces (Tue §6.8 + modern Java)

A **functional interface** has **exactly one abstract method** (SAM). Lambdas and method references are values of that type.

| Concept | Meaning |
|---|---|
| `@FunctionalInterface` | Optional annotation; compiler error if SAM rule broken |
| SAM | The one method your lambda body implements |
| Common JDK types | `Predicate<T>`, `Function<T,R>`, `Consumer<T>`, `Supplier<T>`, `Comparator<T>` |

Examples of **shape** (not exercise solutions):

```java
Predicate<Order> isUnpaid = o -> o.status() == OrderStatus.UNPAID;
Function<Order, String> customerOf = Order::customer;
Comparator<Order> byAmountDesc = Comparator.comparing(Order::amount).reversed();
```

**Polymorphism link (Wed):** a method parameter typed `Predicate<Order>` accepts any lambda or method reference that matches the SAM — same “depend on the contract” story as `Payable`.

**Tue recap:** interface **default** and **static** methods do not add extra abstract methods; an interface can stay functional if only one abstract method remains.

---

## 4. Lambda expressions

**Syntax:** `(parameters) -> expression` or `(parameters) -> { statements; }`

| Form | When |
|---|---|
| `(x) -> x * 2` | Single expression; type often inferred |
| `(Order o) -> o.amount() > 100` | Explicit parameter type when inference fails |
| `() -> System.out.println("hi")` | No parameters |
| `(a, b) -> a.compareTo(b)` | Multiple parameters |

**Rules of thumb:**

- Lambdas capture **effectively final** locals from the enclosing scope (no reassignment of captured variables).
- Prefer **small** lambdas; extract a **private method** or **named** `Predicate` if the block grows past a few lines.
- Lambdas implement the SAM of the **target type** (context where the lambda appears), not “a generic lambda type.”

**Pitfall preview:** using a lambda where a **raw** overload expects `Object` — fix the **target functional interface** or parameter type.

---

## 5. Method references

Shorthand when a lambda only **delegates** to an existing method.

| Kind | Syntax | Example idea |
|---|---|---|
| Static | `Type::method` | `Integer::parseInt` |
| Instance on particular object | `obj::method` | `System.out::println` |
| Instance on first parameter | `Type::method` | `Order::amount` (getter) |
| Constructor | `Type::new` | `ArrayList::new` |

Same SAM rules as lambdas: `Order::amount` fits `Function<Order, Double>` if `amount()` returns `double`/`Double` with boxing where needed.

Use method references when they **read as English**; use a lambda when you need **extra logic** (e.g. compare with a threshold).

---

## 6. `Optional<T>`

**Purpose:** model “value may be absent” in **return types** and chains without scattering **`null`** checks (still not a general-purpose field type).

| API (basics) | Use |
|---|---|
| `Optional.of(value)` | Non-null value; NPE if null |
| `Optional.ofNullable(value)` | Nullable source |
| `Optional.empty()` | No value |
| `isPresent()` / `isEmpty()` | Imperative checks (prefer functional style when learning) |
| `orElse(default)` / `orElseGet(Supplier)` | Default if empty |
| `map(fn)` / `flatMap(fn)` | Transform if present |
| `ifPresent(Consumer)` | Side effect if present |

**Stream link:** `stream.findFirst()` returns `Optional<T>`; terminal **reductions** that may find nothing often pair with `Optional`.

**Style for this course:** use `Optional` on **method returns** (e.g. “highest order if any”); avoid `Optional` fields on entities unless you have a strong reason. For **#2** “top 3 by amount”, a **`List`** or stream **limit** is natural; for “single max order”, `Optional` fits.

---

## 7. Stream API basics

A **stream** is a **sequence of elements** supporting sequential (and later parallel) bulk operations. Streams are **not** collections: they are **one-shot** pipelines (do not reuse a consumed stream).

### 7.1 Pipeline shape

```
source → intermediate* → terminal
```

- **Source:** `collection.stream()`, `Stream.of(...)`, `Arrays.stream(...)`, etc.
- **Intermediate (lazy):** `filter`, `map`, `sorted`, `distinct`, `flatMap`, …
- **Terminal (eager):** `collect`, `reduce`, `forEach`, `count`, `findFirst`, …

### 7.2 Core operations for curriculum **#2**

| Operation | Role | **#2** mapping (conceptual) |
|---|---|---|
| **`filter(Predicate)`** | Keep elements matching a test | Orders with a given status; unpaid check |
| **`map(Function)`** | Transform each element | Order → customer name; Order → amount |
| **`collect(Collector)`** | Mutable reduction into container | **`groupingBy`** customer → total amount; **`toList()`** for top-N after sort |
| **`reduce`** | Combine to one value | Sum amounts (often `mapToDouble` + `sum` is clearer for numbers) |

**Sorting / top 3:** `sorted(Comparator)` then `limit(3)`, or partial sort patterns — implement with streams **and** with loops, then compare in `notes/`.

**Grouping:** `Collectors.groupingBy` + downstream `summingDouble` (or `mapping` + `reducing`) expresses “total per customer” declaratively; loop version uses a `Map` and explicit accumulation.

### 7.3 Primitive streams (awareness)

`mapToDouble(Order::amount)` → `DoubleStream` avoids boxing in numeric sums. Know it exists; either style is fine in Week 2 proof work.

### 7.4 When loops still win

- Complex **multi-step** logic with early exits and many local variables.
- **Indexed** access or mutation of parallel arrays.
- **Performance-sensitive** hot paths (measure later; not Week 2 focus).

Curriculum **#2** explicitly asks for **both** and a **readability** paragraph in `notes/` — honesty beats “streams always better.”

---

## 8. Sealed interfaces (modern; curriculum **#3** preview)

**Sealed types** (JEP 409) restrict **which classes/interfaces** may implement or extend a type — a **closed set** of variants.

| Idea | Benefit |
|---|---|
| `sealed interface Shape permits Circle, Rect, …` | Compiler knows all subtypes |
| **`record`** implementors | Compact immutable variants |
| **`switch` pattern matching** (JEP 441 awareness) | Exhaustive `switch` without `default` when permitted set is complete |

Full **#3** (shapes + area via exhaustive `switch`) is the sealed + records exercise; skim dev.java **Sealed classes** today if you touch **#3**, or defer to Sat review.

---

## 9. Free resources (Week 2 — links only)

Use these alongside this lecture; paraphrase only — no long verbatim book or paywalled excerpts in your repo.

- [Oracle — Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- [Oracle — Method References](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)
- [Oracle — Aggregate Operations (Streams)](https://docs.oracle.com/javase/tutorial/collections/streams/index.html)
- [Oracle — Interfaces and Inheritance](https://docs.oracle.com/javase/tutorial/java/IandI/) (functional interfaces recap)
- [dev.java — Learn Java](https://dev.java/learn/) → **Lambdas**, **Streams**, **Sealed classes**
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
- [JEP 441: Pattern Matching for switch](https://openjdk.org/jeps/441) (awareness for **#3**)
- Baeldung: [Guide to Java 8 Streams](https://www.baeldung.com/java-8-streams), [Java 8 Functional Interfaces](https://www.baeldung.com/java-8-functional-interfaces)
- [Java SE 21 API — `java.util.stream`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/package-summary.html), [`Optional`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html)

*(Full Week 2 resource list: `CURRICULUM.md` Week 2 **Free resources**.)*

---

## 10. Glossary

lambda expression · functional interface · SAM · `@FunctionalInterface` · method reference · capture (effectively final) · `Optional` · stream · source · intermediate operation · terminal operation · `filter` · `map` · `flatMap` · `collect` · `Collector` · `groupingBy` · `reduce` · lazy evaluation · boxing · `mapToDouble` · sealed interface · permits · exhaustive switch · readability · declarative vs imperative

---

## 11. Common pitfalls

1. **Reusing a stream** after a terminal operation — create a new stream from the collection.
2. **Mutating external state** from inside `forEach` when `collect`/`reduce` expresses the same goal — harder to test and not thread-safe later.
3. **Null in stream elements** — `NullPointerException` in pipeline; validate at boundaries or filter explicitly.
4. **Wrong `collect`** — e.g. expecting a `Map` but using `toList()`; learn `groupingBy` signatures for **#2** totals per customer.
5. **`Optional.get()` without check** — use `orElse`, `orElseThrow`, or `ifPresent`.
6. **Using `Optional` as a field** everywhere — obscures simple data models; prefer for returns.
7. **Over-long pipelines** — one giant chain with no names; split or use loops for clarity (counts for **#2** notes).
8. **Confusing `map` vs `flatMap`** — `map` one-to-one; `flatMap` flattens nested streams (Week 4+ lists-of-lists).
9. **Assuming parallel streams are free speed** — not on curriculum path; default **sequential** today.
10. **Forgetting Tue’s type** — lambda must match SAM (parameter types, return type).
11. **Side effects in `sorted`** — comparator must be consistent; do not mutate orders while sorting.
12. **Skipping loop version for #2** — curriculum requires **both** and comparison in `notes/`.

---

## 12. Class vs record vs interface (Week 2 anchor — Fri lens)

| Construct | Today’s use |
|---|---|
| **`record Order(...)`** | Immutable row for **#2**; getters match method references (`Order::amount`) |
| **Interface (`Payable`, sealed `Shape`)** | Contract + closed variants (**#1**, **#3**) |
| **Lambda / method ref** | Behaviour passed into `filter`, `sorted`, `forEach`, APIs taking `Predicate` etc. |
| **Static utility** | Pure helpers (e.g. format money) — no stream required unless batching |

Sat’s `notes/class-vs-record-vs-interface.md` consolidates the week; note **one** place a **record** + stream read clearer than a mutable class.

---

## 13. Hands-on (Fri — curriculum **#2** primary; you write all code)

**Reading:** this lecture + Oracle/dev.java/Baeldung links in §9 (at least **Lambda** + **Aggregate Operations** stream sections).

**Primary Fri focus** (per `CURRICULUM.md` Week 2 **Fri = lambdas & streams**):

### Curriculum **#2** (problem statement only)

Given a list of **at least 10 orders** (each with **id**, **customer**, **amount**, **status**):

1. **Total amount per customer** — implement once with **streams** (e.g. grouping + summing) and once with **plain loops** and a `Map`.
2. **Top 3 orders by amount** — stream path (sort/limit or equivalent) and loop path.
3. **Whether any order is unpaid** — stream path (any-match style) and loop path.

Write a short **readability comparison** in `notes/`: when the stream version was easier to read, when the loop was clearer, and one rule you will use next week.

**Thu carry-over:** reuse the same data set and packages from Lecture 6 **#2** prep; do not change the business questions between loop and stream versions.

| Other prompts | Fri guidance (not a second spec) |
|---|---|
| **#1** | Continue if unfinished; optional: pass a `Predicate` or `Comparator` into your processor method to practice functional types. |
| **#3** | Sealed **`Shape`** + records + area `switch` — good Sat stretch; skim §8 if you start. |
| **#4** | Composition refactor — any day; unrelated to streams unless you stream over a collection of composed objects. |
| **#5** | Label types in `notes/` as you build **#2**; Sat proof **`class-vs-record-vs-interface.md`**. |

Full wording of **#1–#5** lives in **`CURRICULUM.md` Week 2** only.

Work under `exercises/2026-10-16-lambdas-streams/`, continue `exercises/2026-10-15-ch8/` / earlier scratch, or consolidate toward `exercises/week2/`.

Closing habit: one line in `notes/` — what clicked / what confused you about `collect` vs a hand-rolled loop.

---

## 14. Self-check (answer in `notes/`, then verify against §9 links)

1. What makes an interface a **functional interface**?
2. Rewrite in words: `(Order o) -> o.amount() > 50` as a **method reference** on a **bound** vs **unbound** receiver — when is `Order::amount` enough?
3. When should **`Optional.of`** vs **`Optional.ofNullable`** be used?
4. Name one **intermediate** and one **terminal** operation you used for **#2**.
5. Why must you not reuse a stream after **`collect`** or **`reduce`**?
6. (Stretch) How does a **sealed interface** help exhaustive **`switch`** for curriculum **#3**?

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 15. Bridge to Sat (decision guide + review)

Tomorrow per `CURRICULUM.md`: **decision guide + review** — consolidate **`notes/class-vs-record-vs-interface.md`**, revisit weak self-checks, and advance any unfinished **#1–#5**.

Proof artifact: `exercises/week2/` with all five exercises + your decision guide; commit tagged **`week2`**.

Bring today’s **#2** loop vs stream notes to Sat: they feed the decision guide’s “when is a static function / record / interface enough?” table. If **#3** sealed shapes are done, check exhaustiveness with no **`default`** branch.

Week 3 opens **exceptions and I/O** then Spring Boot — streams on in-memory lists remain the pattern for many service-layer transformations later.
