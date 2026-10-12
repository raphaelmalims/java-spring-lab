# Lecture 4 — Interfaces: multiple inheritance of type, `implements`, and modern interface methods

**Date:** Tue 13 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — Ch.6 Inheritance (pp.61–90; read **§6.8** today; **§6.1–6.7** was **Mon** per `CURRICULUM.md` Week 2)

**Time plan (1 h):** 10 min recap (Mon §6.1–6.7) · 20 min read this lecture + book §6.8 · 30 min hands-on (see §12 — curriculum **#1**; problem statement in `CURRICULUM.md` only).

**Goal for today:** explain why Java uses interfaces for multiple *types*, declare and implement interfaces with `implements`, use default/static/private interface methods where they help, recognize a **functional interface** as setup for Friday's lambdas, and choose between an **interface** and an **abstract class** for a given design.

> **Pacing note:** Week 2 in `CURRICULUM.md` schedules **Mon = ch.6 §6.1–6.7**, **Tue = §6.8 interfaces** (this file), **Wed = ch.7**, **Thu = ch.8**, **Fri = lambdas & streams**, **Sat = decision guide + review**. Do not invent schedule changes inside this lecture.

---

## 1. Recap hook (Mon 12 Oct — use the first 10 minutes)

Before reading §6.8, from memory in `notes/`:

1. **Three bullets from yesterday:** e.g. `extends` gives single superclass inheritance; `super(...)` must be first in a subclass constructor; `@Override` marks a true instance-method override.
2. **One self-check from Lecture 3:** e.g. why Java allows only one superclass but still needs a story for "payable employee" *and* "payable invoice" in one list (preview today's topic).
3. **Skim** your scratch from `exercises/2026-10-12-ch6/` or notes on overriding vs overloading.

Mon ended with: **multiple inheritance of classes is forbidden**; **interfaces** carry **multiple inheritance of type**. Today closes that loop.

---

## 2. The multiple inheritance problem (§6.8 context)

If two superclasses both define the same instance method differently, a class that extended **both** would inherit conflicting implementations — the **diamond problem**. Java avoids this by allowing **at most one superclass** (`extends` once).

You still need unrelated classes to share a **role** or **capability**:

- A `CreditCardPayment`, an `Employee`, and a `UtilityBill` might all need to expose "amount due" behaviour without sharing a common superclass.
- API code should depend on **what something can do**, not on a single inheritance tree.

**Multiple inheritance of type** means a class can **implement several interfaces**. At compile time, the object **is** all of those types; at runtime, one object, one layout — no duplicate superclass state from interfaces.

| Mechanism | What you get |
|---|---|
| `extends OneClass` | One superclass: implementation + state inheritance |
| `implements I1, I2, …` | Zero or more interfaces: **contract** (method signatures), not duplicated class fields from parents |

---

## 3. Declaring and implementing interfaces (§6.8)

### 3.1 Interface syntax

```java
public interface Payable {
    double amountDue();   // implicitly public abstract (classic style)
}
```

```java
public class Invoice implements Payable {
    @Override
    public double amountDue() {
        return totalCents / 100.0;
    }
}
```

- **`implements`** lists one or more interface types (comma-separated).
- A class can **`extends` one class** and **`implements` any number of interfaces**.
- Interface methods are **public** by default; instance methods without a body are **abstract** (unless `default` or `static` — below).
- **Constants** in interfaces are `public static final` by default (use sparingly; enums or classes often fit better).

### 3.2 Unrelated implementors, one parameter type

The book's point: polymorphism through an **interface type** does not require an *is-a* class link between implementors.

```java
void printTotalDue(Payable item) {
    System.out.println(item.amountDue());
}
```

Any class that `implements Payable` can be passed — even if their class hierarchies are unrelated. That is how you build **mixed lists** (curriculum **#1** today): `List<Payable>` or a method taking `Payable`, holding many concrete types.

**Static type vs dynamic type (preview for your notes):**

- **Static type** — the type of the variable or parameter (`Payable`, `Employee`, …).
- **Dynamic type** — the actual class of the object at runtime (`Invoice`, `HourlyWorker`, …).

For **instance methods** on interfaces (and overrides in classes), which implementation runs is chosen by **dynamic binding** (ch.7 Wed). For **static** methods, **overloading**, and **private** methods, binding is **static** (compile-time). Curriculum **#1** asks you to **demonstrate with output** and explain static vs dynamic binding in `notes/` — no solution code in this lecture.

---

## 4. Default, static, and private interface methods (modern Java; book §6.8 + post-2008 language)

Older Java interfaces could only declare abstract methods and constants — adding a new method **broke every implementor**. Java 8+ added:

| Kind | Purpose | Called as |
|---|---|---|
| **`default`** instance method | Optional shared behaviour; implementors inherit unless they override | `ref.method()` on an instance |
| **`static`** method | Utility tied to the interface type; not inherited as instance API | `InterfaceName.method()` |
| **`private`** instance/static | Helper for default/static methods inside the interface | only inside the interface body |

Example shape (illustrative domain — not the Payable exercise):

```java
public interface Notifier {
    void send(String message);   // implementors must provide

    default void sendWithPrefix(String message) {
        send("[" + channelName() + "] " + message);
    }

    private String channelName() {
        return "APP";
    }

    static Notifier noop() {
        return message -> { /* Fri: lambda */ };
    }
}
```

**Design discipline:** defaults are for **backward-compatible** shared code, not for dumping large implementations into interfaces. Prefer **one abstract method** or a **class** when behaviour is heavy or stateful.

---

## 5. Functional interfaces — primer for Fri (lambdas & streams)

A **functional interface** is an interface with **exactly one abstract method** (SAM). Examples you will meet: `Runnable`, `Comparator`, `Predicate`, `Function`.

- Annotate with `@FunctionalInterface` (optional but recommended — compiler enforces SAM).
- **Friday** in `CURRICULUM.md` covers **lambdas**, method references, and streams; lambdas are **shorthand for implementing** a functional interface's single abstract method.

Today: when you write `interface X { void go(); }`, you are already holding a slot that a lambda can fill later. Do not rush lambda syntax until Fri; know **why** the SAM rule exists.

**Week 2 free resources** (functional interfaces): Oracle *Lambda Expressions*; Baeldung "Java 8 Functional Interfaces" — listed in `CURRICULUM.md` Week 2.

---

## 6. Interface vs abstract class (§6.8 decision)

Both can declare abstract behaviour and support polymorphism. Rough guide:

| Reach for… | When |
|---|---|
| **Interface** | Unrelated classes share a **role**; you need **multiple types**; you want to mock/swap in tests; behaviour is mostly contract, maybe small `default` helpers |
| **Abstract class** | Strong **is-a** in one hierarchy; **shared state** or substantial shared implementation in the superclass; single inheritance of implementation is enough |

**Both can coexist:** `abstract class Base implements Serializable, Cloneable` — shared implementation in the class, extra **types** from interfaces.

**Records** can implement interfaces (immutable carriers with interface contracts — Week 1 **#5**). **Sealed interfaces** (Fri/modern Week 2) restrict who may implement — closed variant sets.

Sat's `notes/class-vs-record-vs-interface.md` (curriculum proof artifact) will consolidate; today add one line in `notes/` when you chose interface over abstract class for **#1**.

---

## 7. What interfaces are not

- Interfaces do **not** replace constructors — you still `new` concrete classes.
- They do **not** give you fields with instance state (only `public static final` constants by default; Java allows `private static` helpers).
- They are **not** a workaround to extend two classes — only **one** class, many **interfaces**.
- **Skipping in this course:** desktop GUI toolkits, browser applets, and hand-written servlet APIs (book ch.13–15) — not on the Java API/Spring path per `CURRICULUM.md`.

---

## 8. Free resources (Week 2 — links only)

Use these alongside the book; do not paste proprietary book text into your repo.

- [Oracle Java Tutorials — Interfaces and Inheritance](https://docs.oracle.com/javase/tutorial/java/IandI/)
- [Oracle — Creating and Using Interfaces](https://docs.oracle.com/javase/tutorial/java/IandI/createinterface.html)
- [Oracle — Default Methods](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html)
- [Oracle — Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) (read lightly today; **Fri** primary)
- [dev.java — Learn Java](https://dev.java/learn/) (Streams, Lambdas, Sealed classes — Week 2 modern track)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
- Baeldung: [Guide to Java 8 Functional Interfaces](https://www.baeldung.com/java-8-functional-interfaces), [Composition, Aggregation, and Association](https://www.baeldung.com/java-composition-aggregation-association), [Favor Composition over Inheritance](https://www.baeldung.com/java-inheritance-composition)

*(Streams and aggregate operations: Oracle *Aggregate Operations* — **Fri** per `CURRICULUM.md`.)*

---

## 9. Glossary

interface · `implements` · multiple inheritance of type · contract · unrelated implementors · static type · dynamic type · static binding · dynamic binding · `default` method · `static` interface method · `private` interface method · functional interface · SAM · `@FunctionalInterface` · interface vs abstract class · mixed list · polymorphism (preview ch.7)

---

## 10. Common pitfalls

1. **Forgetting `public` on implementing methods** — interface methods are public; reducing visibility in the class is illegal.
2. **Assuming interfaces inherit from a class** — only other interfaces (`extends` between interfaces) and `Object` for classes.
3. **Diamond confusion** — two interfaces with the same `default` method require an explicit override in the class; resolve with `InterfaceName.super.method()`.
4. **Using an interface when you need shared mutable state** — use an abstract class or composition with a class.
5. **Fat interfaces** — ten unrelated methods force every implementor to stub; split interfaces (segregation).
6. **Calling interface static methods on an instance** — use `Payable.format(x)`, not `item.format(x)` (bad style / error).
7. **Expecting static methods to override** — static methods are not polymorphic; class hiding rules apply.
8. **Skipping `@Override` on implementors** — same benefit as with superclass overrides.
9. **Constants explosion in interfaces** — magic numbers shared everywhere; prefer enums or config types.
10. **Confusing "can pay" interface with "is an employee" class** — type role vs identity hierarchy (decision guide, Sat).

---

## 11. Class vs record vs interface (Week 2 anchor)

- **Interface** — today's focus: **capability** across unrelated classes; curriculum **#1** (`Payable`-style contract).
- **Class / abstract class** — Mon: *is-a* with shared implementation (`extends`).
- **Record** — immutable value; can `implements` interfaces when the contract fits read-only data.

---

## 12. Hands-on (Tue — curriculum **#1** only; you write all code)

**Primary hands-on for Tue 13 Oct** (per `CURRICULUM.md` Week 2 daily split): **exercise prompt #1** — problem statement only:

> Define an interface for "something that can be paid" and implement it in at least three unrelated classes. Write a method that accepts the interface type and processes a mixed list. Then demonstrate (with output) which method version runs and explain static vs dynamic binding in your notes.

- Full wording lives in **`CURRICULUM.md` Week 2** — do not treat this lecture as a second spec.
- **No solution code** in `lectures/`; type your implementation under `exercises/` (e.g. scratch `exercises/2026-10-13-ch6/` or toward `exercises/week2/`).
- Prompts **#2–#5** (orders/streams, sealed shapes, composition refactor, type labels) are **later in the week** — not today's primary slot.

Closing habit: one line in `notes/` — what clicked / what confused you about interface types vs class inheritance.

---

## 13. Self-check (answer in `notes/`, then check against the book)

1. Why does Java forbid extending two classes but allow `implements A, B, C`?
2. What is the difference between **static type** and **dynamic type** for a variable declared as an interface?
3. When would you add a **`default`** method to an interface instead of putting code in every implementor?
4. Name one abstract method that makes an interface **functional**; what will Fri add to implement it more briefly?
5. For curriculum **#1**, which calls are resolved at compile time vs runtime — sketch before you run your demo output.

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 14. Bridge to Wed (ch.7 Polymorphism)

Tomorrow: **polymorphism**, static vs dynamic binding in depth, overloading vs overriding in calls, and class (static) members — book ch.7. Today's binding notes for **#1** feed directly into Wed's reading.
