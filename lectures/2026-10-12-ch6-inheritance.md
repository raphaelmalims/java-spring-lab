# Lecture 3 — Inheritance: common properties, `extends`, `super`, overriding, and changing hierarchies

**Date:** Mon 12 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — Ch.6 Inheritance (pp.61–90; read **§6.1–6.7** today; **§6.8** interfaces is **Tue** per `CURRICULUM.md` Week 2)

**Time plan (1 h):** 10 min recap (Week 1) · 20 min read this lecture + book §6.1–6.7 · 30 min hands-on (see §7 — curriculum prompts, no solutions here).

**Goal for today:** explain why subclasses exist, write a small hierarchy with `extends`, use `super` correctly, override methods on purpose (not by accident), and describe what happens when you move classes up or down a hierarchy.

> **Pacing note:** Week 2 in `CURRICULUM.md` schedules **Mon = ch.6 §6.1–6.7** (this file), **Tue = §6.8 interfaces**, **Wed = ch.7**, **Thu = ch.8**, **Fri = lambdas & streams**, **Sat = decision guide + review**. Do not invent schedule changes inside this lecture.

---

## 1. Recap hook (Week 1 — use the first 10 minutes)

Before reading §6.1, from memory in `notes/`:

1. **Three bullets from last week:** e.g. message = method call; encapsulation = private fields + public behaviour; abstract vs concrete class (ch.5).
2. **One self-check from Lecture 2:** e.g. why `CalculatorEngine` needs both `keep` and `toDo` for infix input.
3. **Skim** your last compile/test error (if any) from `exercises/week1/` or scratch folders.

If you did Week 1 **#4** (abstract hierarchy + diagram), keep that diagram handy — you will reuse a hierarchy this week (see curriculum **#4** later in the week).

---

## 2. Common properties and the case for inheritance (§6.1)

Ch.5 gave you **classification**: group objects that share traits into classes, then arrange classes into a **hierarchy** (superclass ↑, subclass ↓).

**Inheritance** is the mechanism that lets a **subclass** reuse and extend what a **superclass** already defines:

- **Common properties** = fields and methods shared by several classes. Instead of copying the same code into `Dog`, `Cat`, and `Horse`, you put shared state and behaviour in `Mammal` (or `Animal`) once.
- **Specialization** = each subclass adds or refines behaviour (ch.5 §5.4). Inheritance is how Java **implements** specialization in code.

Mental model:

| Idea (book) | Java |
|---|---|
| Subclass *is a kind of* superclass | `class Dog extends Mammal` |
| Subclass has everything the superclass has, plus more | inherited fields/methods + new ones |
| Outside code can treat a subclass as its superclass type | assignment and parameters (polymorphism — ch.7 Wed) |

**When inheritance fits:** shared **structure and behaviour**, stable *is-a* relationship (`SalesPerson` *is an* `Employee`). **When it does not:** two classes share only a few helpers, or the relationship is *has-a* (a `Car` *has an* `Engine`) — composition (curriculum **#4**, Baeldung link below).

---

## 3. Inheritance in Java: `extends` (§6.2–6.3)

### 3.1 Syntax

```java
class SubclassName extends SuperclassName {
    // additional fields, methods, constructors
}
```

- Java allows **single inheritance of classes**: each class extends **at most one** superclass. (Multiple inheritance of *types* comes via **interfaces** — §6.8 Tue.)
- **`Object`** is the root of every class hierarchy. If you write `class Foo { }`, you implicitly extend `Object`.
- A subclass **inherits** accessible members of the superclass (fields and methods), subject to visibility (§6.6).

### 3.2 What the subclass gets

- **Instance fields** declared in the superclass (each subclass instance has the superclass part + the subclass part — one object, layered layout).
- **Methods** defined on the superclass, callable on subclass instances unless **overridden** (§6.5).
- **Constructors are not inherited.** The subclass must define its own constructors and usually call a superclass constructor with `super(...)` (§6.4).

### 3.3 Subclass references

A variable typed as the superclass can refer to a subclass instance:

```java
Employee e = new SalesPerson(...);   // legal when SalesPerson extends Employee
```

Which methods run when you call `e.doSomething()` is the subject of **dynamic binding** (ch.7). Today: know that the *static type* (`Employee`) vs *dynamic type* (`SalesPerson`) distinction exists.

Illustrative hierarchy (different domain from weekly exercises):

```java
class Appliance {
    protected final String model;
    protected boolean poweredOn;

    Appliance(String model) {
        this.model = model;
    }

    void powerToggle() {
        poweredOn = !poweredOn;
    }
}

class Microwave extends Appliance {
    private int wattage;

    Microwave(String model, int wattage) {
        super(model);           // must run superclass constructor first
        this.wattage = wattage;
    }

    void defrost(int minutes) {
        if (!poweredOn) powerToggle();
        // ...
    }
}
```

---

## 4. The `super` keyword (§6.4)

`super` refers to the **superclass portion** of the current object.

| Use | Meaning |
|---|---|
| `super(...)` | Call a **superclass constructor** — must be the **first** statement in the subclass constructor (if used). |
| `super.methodName(...)` | Call the **superclass version** of a method (often inside an override). |
| `super.fieldName` | Access a superclass field hidden by a subclass field (rare; prefer not to hide fields). |

**Constructor chain:** when you `new Microwave(...)`, Java runs superclass constructors before subclass instance initialization completes. If the superclass has no no-arg constructor and you omit `super(...)`, the compiler inserts a call to the no-arg superclass constructor — and fails if none exists.

**Why call `super.method()`?** When overriding, you often **extend** behaviour (run superclass logic, then add steps) rather than replace it entirely.

---

## 5. Method overriding (§6.5)

**Overriding** = subclass provides a **new implementation** for an **instance method** already declared in a superclass, with the **same signature** (name, parameter types, return type — with covariant return allowed on overrides).

```java
class Kettle extends Appliance {
    @Override
    void powerToggle() {
        super.powerToggle();
        System.out.println(model + (poweredOn ? " heating" : " idle"));
    }
}
```

Rules and contrasts:

| | Overloading (Week 1) | Overriding (today) |
|---|---|---|
| Where | Same class, different parameter lists | Subclass vs superclass, **same** signature |
| Which is chosen | Compile time (static) | Runtime for instance methods (ch.7) |
| Annotation | none required | `@Override` strongly recommended (compiler checks) |

**Not overrideable the same way:** `static` methods are **hidden**, not overridden; **`private`** methods in the superclass are not visible to the subclass for override; **`final`** methods cannot be overridden.

**Pitfall:** confusing **overloading** in a subclass with overriding — same name but different parameters is still overloading.

Book tie-in: the animal hierarchy (Mammal, Bird, …) is the running example for shared behaviour (e.g. movement, reproduction) specialized in subclasses.

---

## 6. Visibility and `protected` (§6.6)

Inheritance interacts with **access modifiers** (ch.8 Thu goes deeper on packages):

| Modifier | Same class | Same package | Subclass (other package) | Anywhere |
|---|---|---|---|---|
| `private` | yes | no | no | no |
| *(package-private)* | yes | yes | no | no |
| `protected` | yes | yes | yes | no |
| `public` | yes | yes | yes | yes |

- **`protected`** is the inheritance-friendly default when you want subclasses outside the package to see a member but not the whole world.
- **Subclass visibility:** an override may not **reduce** access (e.g. cannot override `public` with `package-private`).
- **Encapsulation still wins:** prefer `private` fields + `protected`/`public` methods over `protected` fields unless the book's pattern explicitly exposes state to subclasses.

---

## 7. Changing class hierarchies (§6.7)

Real designs change. The book discusses **refactoring hierarchies**:

1. **Pull up** — move common code from siblings into a new or existing superclass (**generalization**, ch.5).
2. **Push down** — move specialized code from a bloated superclass into subclasses (**specialization**).
3. **Replace inheritance with delegation** — when *is-a* was wrong, use composition (has-a field implementing behaviour).

Consequences to watch:

- **Constructors** must still chain correctly after you move classes.
- **Callers** typed on a superclass may break if you remove or rename public API.
- **Overridden methods** change runtime behaviour for all subclasses at once when you edit the superclass version.

Curriculum **#4** (later this week) asks you to refactor one Week 1 relationship toward **composition** — today's §6.7 is the conceptual setup.

---

## 8. Free resources (Week 2 — links only)

Use these alongside the book; do not paste proprietary book text into your repo.

- [Oracle Java Tutorials — Interfaces and Inheritance](https://docs.oracle.com/javase/tutorial/java/IandI/)
- [Oracle — Controlling Access to Members](https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html)
- [dev.java — Sealed classes](https://dev.java/learn/) (preview for Fri/modern topics in Week 2)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409)
- Baeldung: [Composition, Aggregation, and Association](https://www.baeldung.com/java-composition-aggregation-association), [Favor Composition over Inheritance](https://www.baeldung.com/java-inheritance-composition)

*(Interfaces, lambdas, streams: Oracle *Lambda Expressions* and *Aggregate Operations*, Baeldung streams/functional interfaces — scheduled **Tue–Fri** per `CURRICULUM.md`.)*

---

## 9. Glossary

inheritance · superclass · subclass · `extends` · common properties · specialization · `super` · constructor chaining · method overriding · `@Override` · hiding (static) · `protected` · access modifier · pull up / push down · generalization · composition vs inheritance · single inheritance · `Object`

## 10. Common pitfalls

1. **Forgetting `super(...)`** when the superclass has no no-arg constructor — compile error on the subclass constructor.
2. **Calling `super()` after other statements** in a constructor — illegal; `super(...)` must be first.
3. **Trying to extend two classes** — Java allows one superclass only; use interfaces for multiple types (Tue).
4. **Shadowing instead of overriding** — typo in method name or wrong parameters → surprise: superclass method runs.
5. **Skipping `@Override`** — you think you overrode but you accidentally overloaded; compiler won't catch it.
6. **Overriding `static` methods** — you hide, not override; polymorphic dispatch does not apply.
7. **`private` methods "overridden"** — the subclass method is unrelated; no polymorphism.
8. **Copy-paste duplication across siblings** before pulling up to a superclass — maintenance debt §6.1 warns against.
9. **Deep inheritance for tiny differences** — prefer fields/enums or composition (curriculum decision guide, Sat).
10. **Breaking invariants** — subclass constructor must leave the object valid for both superclass and subclass rules.

---

## 11. Class vs record vs interface (Week 2 anchor)

Today's focus is **classes in a hierarchy**:

- **Subclass `extends` superclass** when *is-a* holds and you share implementation.
- **Records** do not participate in deep mutable hierarchies the same way (immutable carriers — Week 1 **#5**).
- **Interfaces** (Tue §6.8) express *can-do* contracts when implementations are unrelated classes.

Sat's `notes/class-vs-record-vs-interface.md` will summarize; keep today's notes on *when `extends` helped vs when composition would be cleaner*.

---

## 12. Hands-on (Mon — curriculum prompts only; you write all code)

Week 2 proof set: **five prompts** in `CURRICULUM.md` Week 2 (**#1–#5**). This lecture does **not** duplicate that list as a second exercise set — problem statements live in the curriculum only:

1. **Payable interface** — three unrelated implementors; mixed list; static vs dynamic binding in `notes/`.
2. **Orders + streams** — totals, top 3, unpaid check; streams vs loops; readability in `notes/`.
3. **Sealed shapes** — sealed interface + records; area via exhaustive `switch`.
4. **Composition refactor** — Week 1 three-level hierarchy; one relationship moved to composition; 5-sentence justification.
5. **Type labels** — tiny app; each type marked class / record / interface / enum / static utility with one-line reason.

**Mon 12 Oct (after §6.1–6.7):** the suggested daily split is **reading**, not a numbered prompt day. Use the 30-minute slot for **optional inheritance prep** that supports later prompts — e.g. extend your Week 1 **#4** hierarchy with `extends`, at least one **`super(...)`** constructor chain, and one deliberate **`@Override`**, in scratch under `exercises/2026-10-12-ch6/` or `notes/`. **Do not** start curriculum **#1** today unless you are ahead; **#1** aligns with **interfaces** (**Tue §6.8**). Proof artifacts for the week land in `exercises/week2/` with tag `week2` when Sat review is done.

Closing habit: one line in `notes/` — what clicked / what confused you about `super` vs overriding.

## 13. Self-check (answer in `notes/`, then check against the book)

1. What problem does inheritance solve that copy-paste into every subclass does not?
2. Write the constructor call order when `new Sub(...)` runs and both super and sub define constructors.
3. Give one example where you would call `super.method(...)` inside an override.
4. How does overriding differ from overloading? Which one depends on the runtime type of the object (preview ch.7)?
5. When might you **remove** an inheritance link and use composition instead (tie to §6.7)?

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*
