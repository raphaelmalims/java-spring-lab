# Lecture 5 — Polymorphism: static vs dynamic binding, overloading vs overriding, and class members

**Date:** Wed 14 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — Ch.7 Polymorphism (pp.93–102; full chapter today per `CURRICULUM.md` Week 2)

**Time plan (1 h):** 10 min recap (Tue §6.8 + curriculum **#1** if started) · 20 min read this lecture + book ch.7 · 30 min hands-on (see §11 — curriculum **#1** binding notes; problem statement in `CURRICULUM.md` only).

**Goal for today:** use **polymorphism** confidently — assign subclass/interface implementors to supertype variables, predict **which method runs** via static vs **dynamic binding**, distinguish **overloading** from **overriding** at call sites, and treat **class (static) members** separately from instance polymorphism.

> **Pacing note:** Week 2 in `CURRICULUM.md` schedules **Mon = ch.6 §6.1–6.7**, **Tue = §6.8 interfaces**, **Wed = ch.7** (this file), **Thu = ch.8**, **Fri = lambdas & streams**, **Sat = decision guide + review**. Do not invent schedule changes inside this lecture. **Do not** treat Friday’s lambdas/streams lesson as today’s topic — only brief Week 2 context where it naturally connects (e.g. functional interfaces as *types* for polymorphism later).

---

## 1. Recap hook (Tue 13 Oct — use the first 10 minutes)

Before reading ch.7, from memory in `notes/`:

1. **Three bullets from yesterday:** e.g. `implements` gives multiple inheritance of **type**; mixed lists use the **interface** (or superclass) as the static type; **default** methods share behaviour without breaking old implementors.
2. **One self-check from Lecture 4:** e.g. why interface **instance** methods follow dynamic binding but **static** interface methods do not.
3. **Skim** output from curriculum **#1** if you already coded it — or note where you stopped (compile error, missing third implementor, binding paragraph not written yet).

Mon gave **overriding**; Tue gave **interface types**. Today explains **what happens at the call** when static type and dynamic type differ.

---

## 2. Polymorphism in OOP (ch.7 opening; pp.93–94)

**Polymorphism** (*many forms*) means one **interface** to the outside world can refer to objects of **several concrete classes**, and the **right behaviour** still runs.

In Java you see it as:

| Form | Idea (book) | Typical Java shape |
|---|---|---|
| **Inclusion polymorphism** | Treat a subtype as its supertype | `Employee e = new SalesPerson(...);` |
| **Interface polymorphism** | Depend on a **role**, not a class tree | `Payable p = new Invoice(...);` (Tue §6.8) |

Benefits the book stresses:

- **Flexibility** — methods accept supertypes; callers pass any compatible implementor.
- **Extensibility** — new subclasses/implementors without rewriting every caller (as long as the contract holds).
- **Clarity** — client code depends on **what** objects do, not on every concrete class name.

**Static type** (compile-time type of variable/parameter) vs **dynamic type** (actual class of the object at runtime) is the pair you will use all day. Ch.7 makes that distinction the key to **binding**.

---

## 3. Static binding vs dynamic binding (pp.94–97)

**Binding** = linking a **method call** to the **code** that runs.

| Binding | When decided | What it applies to (typical) |
|---|---|---|
| **Static (early) binding** | **Compile time** | Overloaded methods; `static` methods; `private` methods; `final` instance methods; field access |
| **Dynamic (late) binding** | **Runtime** | **Overridden instance methods** on objects (virtual dispatch) |

### 3.1 Dynamic binding for overridden instance methods

When you call an **instance method** through a reference whose **static type** is a superclass or interface, the JVM uses the object’s **dynamic type** to choose the **most specific override** in the inheritance chain.

Mental trace (book’s employee / animal style examples):

```java
Employee ref = new HourlyEmployee(...);
ref.calculatePay();   // runs HourlyEmployee’s override if present
```

The compiler checks that `calculatePay()` exists on `Employee` (static type). At runtime, the dispatch table for the **actual object** selects the subclass implementation.

Same story for **interface** references (Tue): `Payable item = new UtilityBill(...);` → `item.amountDue()` uses the implementor’s method body.

### 3.2 Static binding cases (still ch.7)

These do **not** swap implementations when the dynamic type changes:

- **Overloading** — same name, **different parameter lists** in one class (or inherited overloads visible at compile time). The compiler picks the best match from **static types of arguments** and the **static type** of the receiver for instance overload resolution.
- **`static` methods** — **hidden**, not overridden (Mon §6.5). Call `Super.staticM()` vs `Sub.staticM()` is resolved by the **declared type** of the reference, not the object’s class.
- **`private` and `final` instance methods** — no dynamic override chain for those calls in the usual sense.
- **Fields** — access is by **declared type** of the reference (fields are not polymorphic); prefer methods for behaviour.

Curriculum **#1** asks you to **print** which version runs and **explain** static vs dynamic binding in `notes/` — use ch.7 vocabulary here.

### 3.3 Why it matters for design

APIs that take **`Payable`**, **`Shape`**, or **`Employee`** stay stable when you add new concrete classes. **Dynamic binding** is what makes “one method on the interface/superclass” fair for every implementor. **Static binding** is what makes overloads and utilities predictable at compile time — and what bites you if you expect `static` methods to behave like overrides.

**Week 2 modern preview (context only):** a **functional interface** (Tue primer) is still a **type** for polymorphism; **Friday** adds lambda syntax to implement it. **Sealed** hierarchies (Fri/modern) narrow *who* can implement/subclass while keeping the same dispatch rules for overrides.

---

## 4. Overloading vs overriding at call sites (pp.97–99)

Ch.7 revisits the contrast Mon introduced; now tie each to **binding**.

| | **Overloading** | **Overriding** |
|---|---|---|
| **Signature** | Same name, **different parameters** | Same name, **same parameters** (return may covary) |
| **Location** | Usually one class; subclasses may add overloads | Superclass/subclass (or interface/implementor) |
| **Binding** | **Static** — compiler chooses overload | **Dynamic** — JVM chooses override (instance methods) |
| **Annotation** | none | `@Override` on implementors/overrides |

**Pitfall combo:** subclass declares `void pay()` and you call `pay()` on a `Employee` reference to a `SalesPerson` — if you meant to override but changed parameters, you **overloaded** instead; the superclass method may still run.

**Pitfall:** `static void report()` in superclass and subclass — call through `Super ref = new Sub()` invokes **Super’s** static method (static type wins).

Book exercises often ask you to **trace** a short program line by line: write predicted output **before** running, then reconcile with binding rules.

---

## 5. Polymorphism and class (static) members (pp.99–102)

Ch.7 separates **per-object state** from **class-wide** state and behaviour.

### 5.1 Instance vs class members

| | **Instance** | **Class (static)** |
|---|---|---|
| **Belongs to** | Each object | The **class** (one copy shared) |
| **Keyword** | (default for fields/methods) | `static` |
| **Access** | Needs an instance (or object reference) | Can use `ClassName.member` or reference (discouraged for static methods) |
| **Polymorphism** | Overridden **instance** methods — dynamic binding | **No** true override; **hiding** — static binding |

Examples the book uses: counting instances, shared constants, factory-style static methods, utility helpers on the class.

### 5.2 Static fields and invariants

- **`static` fields** are shared — all instances see the same value; useful for counts, caches, shared configuration (watch thread-safety later in the course).
- **Instance fields** live in each object; `static` methods may only access instance members through a **specific object** reference passed in.

### 5.3 When to use static methods

Reach for **static** when the operation does not need instance state (aligns with curriculum decision guide: **static function** for pure computation). Do **not** use `static` on a method you want subclasses to **polymorphically** replace — use an instance override or an interface.

Thu (ch.8) adds **packages** and **visibility**; today’s static/instance split feeds **where** shared state lives in a module.

---

## 6. Upcasting, downcasting, and `instanceof` (pp.100–102 — book emphasis)

**Upcasting** — assign a subclass instance to a superclass/interface variable (implicit, safe):

```java
SalesPerson sp = new SalesPerson(...);
Employee e = sp;   // upcast; dynamic type unchanged
```

**Downcasting** — treat a reference as a **more specific** type to call subclass-only API:

```java
Employee e = getEmployee();
if (e instanceof SalesPerson sp) {
    sp.getSalesQuota();   // pattern matching style (modern Java; book may use classic cast)
}
```

Rules:

- Downcast only when the **dynamic type** actually supports it — otherwise `ClassCastException`.
- Prefer **polymorphic instance methods** on the supertype so callers avoid excessive casting (design toward ch.8 encapsulation).

---

## 7. Free resources (Week 2 — links only)

Use these alongside the book; do not paste proprietary book text into your repo.

- [Oracle Java Tutorials — Interfaces and Inheritance](https://docs.oracle.com/javase/tutorial/java/IandI/)
- [Oracle — Overriding and Hiding Methods](https://docs.oracle.com/javase/tutorial/java/IandI/override.html)
- [Oracle — Polymorphism](https://docs.oracle.com/javase/tutorial/java/IandI/polymorphism.html)
- [Oracle — Lambda Expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) (**Fri** primary — skim only if curious)
- [Oracle — Aggregate Operations (Streams)](https://docs.oracle.com/javase/tutorial/collections/streams/index.html) (**Fri**)
- [dev.java — Learn Java](https://dev.java/learn/) (Streams, Lambdas, Sealed classes — Week 2 modern track)
- [JEP 409: Sealed Classes](https://openjdk.org/jeps/409) (closed hierarchies — **Fri** / prompt **#3**)
- Baeldung: [Composition, Aggregation, and Association](https://www.baeldung.com/java-composition-aggregation-association), [Favor Composition over Inheritance](https://www.baeldung.com/java-inheritance-composition)

*(Functional interfaces and streams: Oracle *Lambda Expressions* and *Aggregate Operations*; Baeldung "Guide to Java 8 Streams" — **Fri** per `CURRICULUM.md`.)*

---

## 8. Glossary

polymorphism · inclusion polymorphism · interface polymorphism · static type · dynamic type · static (early) binding · dynamic (late) binding · virtual dispatch · overloading · overriding · hiding (`static`) · class member · instance member · `static` field · `static` method · upcasting · downcasting · `instanceof` · `ClassCastException` · contract · mixed list

---

## 9. Common pitfalls

1. **Expecting fields to override** — subclass field “hides” superclass field; access follows **reference declared type**, not dynamic type.
2. **Expecting `static` methods to override** — they **hide**; call resolves at compile time from reference type.
3. **Accidental overload instead of override** — parameter list drift; missing `@Override` hides the mistake.
4. **Calling subclass-only API through a supertype reference** without cast — compile error; fix with better polymorphic method on supertype or safe downcast.
5. **Unsafe downcast** — no `instanceof` check → runtime `ClassCastException`.
6. **Confusing overload resolution with override dispatch** — overload picked from **argument static types** at compile time; override picked from **receiver object** at runtime.
7. **Interface `default` vs class override** — diamond/default conflicts need explicit resolution (`InterfaceName.super.method()` — Tue).
8. **Mutating shared `static` state** from many instances — surprising global effects; document invariants.
9. **Using polymorphism where a `record` or static utility suffices** — decision guide Sat; not every variation needs a deep hierarchy.
10. **Skipping the written binding explanation in #1** — output alone is not enough; ch.7 wants you to **name** static vs dynamic for each call in your demo.

---

## 10. Class vs record vs interface (Week 2 anchor)

- **Polymorphic receiver** — usually **`interface`** or **superclass** type for parameters/collections (Tue **#1**, today’s binding).
- **Class hierarchy** — shared implementation + overrides (`extends`, Mon).
- **Record** — value type; can implement interfaces but is not your go-to for mutable polymorphic hierarchies.
- **Static methods** — utilities and factories, not polymorphic hooks.

---

## 11. Hands-on (Wed — curriculum **#1** focus; you write all code)

**Primary hands-on for Wed 14 Oct** (per `CURRICULUM.md` Week 2 daily split **Wed = ch.7**): finish or deepen **exercise prompt #1** — especially the **binding write-up** if Tue ran out of time.

> Define an interface for "something that can be paid" and implement it in at least three unrelated classes. Write a method that accepts the interface type and processes a mixed list. Then demonstrate (with output) which method version runs and explain static vs dynamic binding in your notes.

**Wed-specific guidance (not a second spec):**

- If **#1** code already runs from Tue, spend the 30 minutes on a **binding table** in `notes/`: for each `println` or method call in your demo, label **static binding** vs **dynamic binding** and cite ch.7 (pp.94–97).
- Add **one deliberate contrast**: e.g. an overloaded helper vs an overridden `amountDue()` (or similar), or a **`static`** method on a class vs an **instance** override — predict output using today’s rules, then run.
- Full wording of **#1** lives in **`CURRICULUM.md` Week 2** — no solution code in `lectures/`.
- Prompts **#2–#5** remain **later in the week** (#2 streams **Fri**; #3 sealed **Fri**/modern; #4 composition; #5 type labels / Sat guide).

Work under `exercises/2026-10-14-ch7/`, continue Tue scratch, or consolidate toward `exercises/week2/`.

Closing habit: one line in `notes/` — what clicked / what confused you about static vs dynamic binding.

---

## 12. Self-check (answer in `notes/`, then check against the book pp.93–102)

1. In one sentence each, define **static type** and **dynamic type** for `Payable p = new Invoice(...)`.
2. Which binding applies to an **overridden instance method** call through a superclass reference? Which applies to an **overload** chosen by argument types?
3. Why does `Super s = new Sub(); s.staticMethod();` **not** invoke `Sub.staticMethod()` if both define static methods?
4. Can two unrelated classes appear in the same `List<Payable>`? What binding rule makes each `amountDue()` call correct?
5. When is a **downcast** necessary, and what check should precede it?

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*

---

## 13. Bridge to Thu (ch.8 Modularity)

Tomorrow: **packages**, **`import`**, visibility across compilation units, encapsulation trade-offs — book ch.8 (pp.103–117). Today’s polymorphic APIs become cleaner when **public surface** vs **package-private** implementation is deliberate.
