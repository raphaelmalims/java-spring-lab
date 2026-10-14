# Lecture 6 — Modularity: packages, `import`, visibility, and encapsulation trade-offs

**Date:** Thu 15 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — Ch.8 Modularity (pp.103–117; full chapter today per `CURRICULUM.md` Week 2)

**Time plan (1 h):** 10 min recap (Wed ch.7 + curriculum **#1** if still open) · 20 min read this lecture + book ch.8 · 30 min hands-on (see §11 — ch.8 package layout + curriculum **#2** prep / **#4** / finish **#1**; problem statements in `CURRICULUM.md` only).

**Goal for today:** organize code into **packages**, use **`import`** cleanly, apply **access modifiers** across compilation units, relate **instance vs class (static) properties** to modular design, and choose **encapsulation** boundaries with deliberate trade-offs — so Week 2 polymorphic APIs expose a small **public surface** and hide implementation detail.

> **Pacing note:** Week 2 in `CURRICULUM.md` schedules **Mon = ch.6 §6.1–6.7**, **Tue = §6.8 interfaces**, **Wed = ch.7**, **Thu = ch.8** (this file), **Fri = lambdas & streams**, **Sat = decision guide + review**. Do not invent schedule changes inside this lecture. **Do not** treat Friday’s lambdas/streams lesson as today’s topic — only brief Week 2 context (e.g. **#2** order model feeds streams **Fri**).

---

## 1. Recap hook (Wed 14 Oct — use the first 10 minutes)

Before reading ch.8, from memory in `notes/`:

1. **Three bullets from yesterday:** e.g. **dynamic binding** picks overridden **instance** methods at runtime; **static binding** picks overloads, `static` methods, and fields; **static type** vs **dynamic type** explain mixed `List<Payable>` behaviour.
2. **One self-check from Lecture 5:** e.g. why `Super s = new Sub(); s.staticMethod();` uses the superclass static method.
3. **Skim** curriculum **#1** — binding table in `notes/` done or note the gap; polymorphic `Payable`-style API is the **client** side; today is the **module** side (who may call what from which package).

Wed’s public methods on supertypes/interfaces become maintainable when **implementation classes** stay **package-private** or **private** where possible.

---

## 2. Why modularity matters (ch.8 opening; pp.103–104)

**Modularity** means splitting a system into **cohesive units** with **clear boundaries**. In Java, the primary unit above a single class is the **package**.

Benefits the book emphasizes (paraphrase):

- **Manage complexity** — large programs are navigable when related types live together.
- **Control coupling** — other parts of the system depend on a **small, stable API**, not every field.
- **Reuse and deployment** — packages map to directory trees and (with tools like Maven) to **artifacts** you ship and version.
- **Team boundaries** — naming and visibility conventions reduce accidental cross-module breakage.

OOP modularity pairs with what you already know: **encapsulation** (Week 1) hides internal state; **packages + visibility** extend that idea **across files**.

*(Book ch.13–15 — GUI, applets, servlets — are **not** on this course path; see `CURRICULUM.md` [Book chapters skipped on purpose](#book-chapters-skipped-on-purpose). Spring Boot replaces hand-written servlets from Week 3 onward.)*

---

## 3. Object properties vs class properties (§8.2; pp.103–108)

Ch.8 revisits **where state lives**, tying Mon/Wed **static vs instance** to modular design.

| | **Object (instance) properties** | **Class (static) properties** |
|---|---|---|
| **Storage** | One copy **per object** on the heap | One copy **per class** (shared by all instances) |
| **Typical use** | Identity, mutable state with invariants | Constants, counters, shared caches (use carefully) |
| **Access** | Through instance methods (preferred) | `ClassName.field` or via static getters |
| **Modularity note** | Encapsulate with `private` fields | **Global within the class loader** — visible side effects across the app |

**Design link:** a **package-private** class with **private** instance fields exposes behaviour through **`public`** methods (or **`public`** / **`protected`** types in an API package). **Static** fields that mutate are **module-wide globals** — document invariants; concurrency weeks revisit thread safety.

**Book point (pp.105–108):** distinguish **behaviour that belongs to each object** from **behaviour or data that belongs to the type as a whole**. Wrong placement (everything `static`, or shared mutable static counters without thought) breaks encapsulation as surely as public fields.

---

## 4. Packages (§8.3; pp.108–111)

A **package** is a namespace for types: `package com.example.payments;` at the top of a `.java` file declares membership.

### 4.1 Naming and layout

- **Convention:** reverse DNS style (`com.yourname.project.feature`) to avoid name clashes.
- **Filesystem rule:** file path must match package (`com/example/payments/Payable.java` for `com.example.payments.Payable`).
- **Default package:** no `package` line — only for tiny scratch; **avoid** in exercises/week proof folders; use a named package early.

### 4.2 Package cohesion

Group classes that **change together** and **serve one purpose**:

| Cohesive package (example shape) | Holds |
|---|---|
| `…model` or `…domain` | Entities, records, value types |
| `…service` or `…app` | Orchestration, `main`, use cases |
| `…util` | Pure helpers (prefer **static** functions when stateless) |

**Unrelated** classes dumped in one package erode modularity — callers import clutter and visibility becomes meaningless.

### 4.3 Package-level privacy

Types and members with **no modifier** are **package-private** (also called *default* or *package* access):

- Visible to all types in the **same package**.
- **Invisible** to other packages — even subclasses in another package cannot see package-private members (contrast **`protected`**, Mon §6.6).

This is your main tool for **implementation hiding** without making everything `public`.

---

## 5. The `import` statement (§8.4; pp.111–113)

**Import** tells the compiler which **short names** you use for types defined elsewhere.

```java
import com.example.payments.Payable;
import java.util.List;
```

| Form | Meaning |
|---|---|
| `import a.b.C;` | Use simple name `C` for `a.b.C` |
| `import a.b.*;` | Import all types from package `a.b` (**on-demand**; use sparingly in production style) |
| *(no import for `java.lang`)* | `String`, `Object`, etc. always available |

**Rules of thumb:**

- Prefer **single-type imports** over `*` — clearer in code review and merges.
- **Fully qualified name** once (`com.example.payments.Payable p`) avoids an import but hurts readability if repeated — pick one style per file.
- Import **does not** affect runtime loading by itself; it is compile-time name resolution. (Class loading is JVM/classpath — Week 5+.)

**Static import** (post-book, modern Java): `import static java.lang.Math.max;` — use for well-known constants/methods only; avoid static-importing your own types unless it genuinely clarifies tests.

---

## 6. Controlling access: visibility modifiers (§8.5; pp.113–116)

Java’s four access levels govern **members** (fields, methods, nested types) and **top-level classes** (only `public` or package-private).

| Modifier | Same class | Same package | Subclass (other package) | Anywhere |
|---|---|---|---|---|
| **`private`** | yes | no | no | no |
| *(package-private)* | yes | yes | no | no |
| **`protected`** | yes | yes | yes | no |
| **`public`** | yes | yes | yes | yes |

### 6.1 Top-level classes

- At most **one** `public` class per file; filename must match that class.
- Other classes in the same file may be package-private **support** types (helpers not meant for export).

### 6.2 Inheritance + packages (Mon §6.6 + ch.8)

- **`protected`** lets subclasses **outside the package** call or override members — but **not** arbitrary unrelated code.
- **Subclass cannot reduce** visibility when overriding (e.g. cannot override `public` with package-private).
- **Package-private** superclass members are **not** inherited as accessible API in a subclass **in another package** — design hierarchies with package placement in mind.

### 6.3 Interface and API surface

For curriculum **#1**-style designs:

- **`public interface`** in an `api` or root package — what clients depend on.
- **`public` concrete classes** only where construction or extension is intended; otherwise **package-private** classes + **`public` factory** or interface return types.

---

## 7. Encapsulation trade-offs (§8.6; pp.116–117)

**Encapsulation** = bundle data with the methods that preserve **invariants**, and **hide** representation choices.

| Stronger encapsulation | Weaker / leaky encapsulation |
|---|---|
| `private` fields, accessors only where needed | `public` fields, getters/setters for every field by habit |
| Package-private implementation types | Everything `public` “for testing” |
| Narrow `public` API on interfaces | Concrete types leaked into every method signature |

**Trade-offs the book invites you to notice:**

1. **Getters/setters everywhere** — sometimes a **record** or **behaviour-only** API is clearer (Week 1 **#5**, Sat decision guide).
2. **`protected` fields** — convenient for subclasses, but **freeze** internal layout; prefer `protected` **methods** or `private` fields + protected accessors.
3. **Friend-style access** — Java has no `friend`; **same package** is the built-in compromise — use package structure intentionally.
4. **Testing** — package-private types are testable from **`…` test package** with same package name in `src/test/java` (JUnit weeks) or via **public** behaviour under test; do not make production types `public` only for tests without thought.
5. **Composition (curriculum #4)** — hiding a delegate behind a **private final** field is encapsulation across objects, not just across fields (Baeldung links below).

**Composition vs inheritance (Thu hands-on #4):** when a Week 1 **is-a** link was wrong, **has-a** with a **private** collaborator keeps the **public** superclass/interface stable while you move behaviour into another class — modularity win.

---

## 8. Putting it together: a modular Week 2 shape

Illustrative layout only — **not** solution code for exercises:

```
src/
  com/raphael/week2/
    payable/          // contract + maybe public types
      Payable.java
    impl/             // package-private implementors OK
      ...
    orders/           // #2 domain — prep today, streams Fri
      Order.java
      OrderStatus.java
```

- **`Payable`** — likely `public` interface; implementors may be package-private if created inside the module.
- **`Order`** for **#2** — define **id, customer, amount, status** with sensible visibility; **Fri** adds streams over a `List<Order>`.
- **#4** — refactor one Week 1 hierarchy: mark which types stay **`public`**, which become package-private, which relationship becomes a **private** field.

---

## 9. Free resources (Week 2 — links only)

Use these alongside the book; do not paste proprietary book text into your repo.

- [Oracle Java Tutorials — Creating and Using Packages](https://docs.oracle.com/javase/tutorial/java/package/)
- [Oracle — Controlling Access to Members of a Class](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html)
- [Oracle — Interfaces and Inheritance](https://docs.oracle.com/javase/tutorial/java/IandI/) (visibility + inheritance recap)
- [Oracle — Aggregate Operations (Streams)](https://docs.oracle.com/javase/tutorial/collections/streams/index.html) (**Fri** primary — skim **#2** domain today)
- [Oracle — Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) (**Fri**)
- [dev.java — Learn Java](https://dev.java/learn/) (Streams, Lambdas — Week 2 modern track)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409) (**Fri** / prompt **#3**)
- Baeldung: [Composition, Aggregation, and Association](https://www.baeldung.com/java-composition-aggregation-association), [Favor Composition over Inheritance](https://www.baeldung.com/java-inheritance-composition), [Guide to Java 8 Streams](https://www.baeldung.com/java-8-streams) (**Fri**)

*(Full Week 2 resource list: `CURRICULUM.md` Week 2 **Free resources**.)*

---

## 10. Glossary

modularity · package · package-private · default access · namespace · compilation unit · `import` · on-demand import (`*`) · access modifier · `public` · `protected` · `private` · encapsulation · invariant · API surface · implementation hiding · object property · class property · static field · cohesion · coupling · composition · top-level class

---

## 11. Common pitfalls

1. **Mismatching package and directory** — compile errors or wrong class loaded; fix path or `package` line.
2. **Everything `public`** — no module boundary; refactors break unknown callers.
3. **Assuming `import` brings in static fields/methods** — import is for **types**; call static members on the declaring class unless using static import.
4. **Subclass in another package expecting access to package-private superclass members** — illegal; use `protected`/`public` deliberately or co-locate packages.
5. **`protected` data fields** — subclasses everywhere couple to your layout; prefer private fields.
6. **Mutable `public static` fields** — global variables; hard to test and reason about (ch.8 + Wed static members).
7. **Default package in real projects** — fine for five-line scratch; use named packages in `exercises/week2/`.
8. **Circular package dependencies** — types in `a` importing `b` and vice versa; extract shared API or merge cohesion.
9. **Leaking concrete types in polymorphic API** — method returns `ArrayList` instead of `List`; callers bind to implementation (generics Week 4).
10. **Skipping #4 justification** — composition refactor needs **five sentences** in `notes/` linking encapsulation/modularity to the change.

---

## 12. Class vs record vs interface (Week 2 anchor)

- **Package + `public` interface** — stable **contract** for polymorphism (**#1**).
- **Package-private class** — hidden implementation; **public** factory or interface-typed returns.
- **Record** — immutable **model** rows (**#2** orders, **#3** shapes later); often `public` in a `model` package.
- **Static utility** — stateless helpers in a `util` package; no instance state.

Sat’s `notes/class-vs-record-vs-interface.md` consolidates; today note **one** type you made package-private and why.

---

## 13. Hands-on (Thu — curriculum **#1–#5** pointers only; you write all code)

**Reading:** book ch.8 (pp.103–117) + this lecture.

**Primary Thu focus** (per `CURRICULUM.md` Week 2 **Thu = ch.8**): apply **packages and visibility** while advancing the weekly proof set — **no solution code** in `lectures/`.

| Prompt | Thu guidance (not a second spec) |
|---|---|
| **#1** | If Wed left gaps, finish **Payable** demo + binding notes; move types into at least **two packages** (e.g. contract vs impl) and shrink **`public`** surface. |
| **#2** | **Prep for Fri streams:** model **≥10 orders** (id, customer, amount, status) in named packages; **plain loops** for totals / top 3 / unpaid check are OK today — repeat with streams **Fri**. Compare readability in `notes/` when both exist. |
| **#3** | Optional scratch only — **sealed shapes** are **Fri**/modern-heavy. |
| **#4** | Strong Thu fit: **composition refactor** on Week 1 three-level hierarchy — use **private** delegate fields, package-private helpers, **5-sentence** justification in `notes/`. |
| **#5** | Light labeling in `notes/` as types appear; Sat proof **`class-vs-record-vs-interface.md`**. |

Full wording of **#1–#5** lives in **`CURRICULUM.md` Week 2** only.

Work under `exercises/2026-10-15-ch8/`, continue earlier scratch, or consolidate toward `exercises/week2/`.

Closing habit: one line in `notes/` — what clicked / what confused you about package-private vs `public`.

---

## 14. Self-check (answer in `notes/`, then check against the book pp.103–117)

1. What is the difference between an **object property** and a **class property**? Where does each live at runtime (preview Week 5 heap)?
2. Why must the directory path match the **`package`** declaration?
3. Which access level lets unrelated code in **another package** call a method? Which level hides a member from subclasses **in another package**?
4. When is **`protected`** appropriate instead of **`private`** + public API?
5. Name one **encapsulation trade-off** when you expose a **`public` getter/setter** for every field.

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 15. Bridge to Fri (lambdas & streams)

Tomorrow per `CURRICULUM.md`: **lambdas**, method references, **`Optional`**, **Stream API** basics (`filter` / `map` / `collect` / `reduce`), and **sealed interfaces** — modern Week 2 track.

Today’s **#2** order list and clean **model** packages give you a **`List<Order>`** to stream; **functional interfaces** (Tue) are the types lambdas implement. Finish loop versions first if you can — Fri’s comparison in `notes/` is richer with both.

Sat: **decision guide** + Week 2 review; proof artifact `exercises/week2/` + `notes/class-vs-record-vs-interface.md`, tag `week2`.
