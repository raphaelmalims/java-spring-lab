# Lecture 1 — OOP basics: objects, classes, messages, and a quick tour of Java

**Date:** Mon 5 Oct 2026 · **Book:** *Object-Oriented Programming and Java*, 2nd ed. — Ch.1 Introduction (pp.1–5), Ch.2 Object, Class, Message and Method (pp.7–14), Ch.3 A Quick Tour of Java (pp.17–36; read §3.1–3.7 closely, skim §3.8–3.12 on operators/control flow)

**Time plan (1 h):** 10 min recap (nothing yet — instead, write down what you already believe "OOP" means in 3 bullets) · 20 min read this lecture + book pages · 30 min hands-on (exercises below).

**Goal for today:** be able to explain an object-oriented program in terms of *objects sending messages* and write a small, correct Java class yourself.

---

## 1. The big idea (Ch.1)

- **Procedural** programming organizes code around *processes* ("check in a book", "make a reservation"). **Object-oriented** programming organizes code around the *things* in the problem and how they *interact*.
- The book's running story: Benjamin asks Sean (a salesperson) to take an order. Benjamin does not know *how* Sean fulfils the request, only *what* he can ask. This is **information hiding**: the sender of a message doesn't know how the receiver handles it.
- OOP is a natural fit for **simulation** (it started with Simula in the 1970s) and, more usefully for you today, for large systems where many parts must change independently — e.g. Spring Boot services made of cooperating objects.
- Java (1995): compiled to **bytecode** and run on the **JVM**, so the same compiled program runs anywhere a JVM exists. Automatic memory management (garbage collection) is a major reason it replaced C/C++ in much enterprise code. (We study the JVM in Week 5.)

## 2. Objects, classes, messages, methods (Ch.2)

| Term | Meaning | Java mapping |
|---|---|---|
| **Object** | A thing with **state** (data) and **behaviour** (what it can do) | An instance created with `new` |
| **Class** | A blueprint/template that describes the state and behaviour of a *kind* of object | `class Foo { … }` |
| **Message** | A request sent from one object (sender) to another (receiver), with optional **parameters/arguments** | A method call, `receiver.doThing(arg)` |
| **Method** | The code in the receiver that implements how to respond to a valid message | A method declared in the class |
| **Valid / invalid message** | Valid = receiver has a matching method; invalid = it doesn't | Compile error in Java (the compiler checks) |
| **Client / server** | The object *requesting* a service is the client; the one *providing* it is the server. One object can be both | Caller vs callee |
| **Instance** | One concrete object of a class | `Foo f = new Foo();` |

Key mental model: **calling a method is sending a message**. `account.deposit(50)` = "account, please deposit 50".

**Class vs object analogy:** the class is the cookie cutter; objects are the cookies. Each object has its *own copy* of the instance state but all share the class's method code.

## 3. A quick tour of Java (Ch.3)

### 3.1 Primitive types vs references
- 8 primitives: `byte short int long float double char boolean`. They hold the *value* directly.
- Everything else (`String`, arrays, your own classes) is an **object**, and a variable of that type holds a **reference** (a pointer-like handle) to it, or `null`.
- Consequence: `==` on primitives compares values; `==` on references compares *identity* (same object?). Use `.equals(...)` for content comparison.

### 3.2 Defining an object (class)
A class has:
- **Fields** (instance variables): the state.
- **Methods**: the behaviour; each has a return type (or `void`), a name, and parameters.
- **Constructors**: special methods named like the class, with no return type, run by `new` to initialize state.

Illustrative shape (not an exercise solution — a deliberately different domain):

```java
public class Lamp {
    private boolean on;          // state, hidden from outside
    private final int watts;     // can never change after construction

    public Lamp(int watts) {     // constructor
        this.watts = watts;
    }

    public void switchOn()  { on = true; }   // behaviour (a valid message)
    public boolean isOn()   { return on; }
}
```

```java
Lamp reading = new Lamp(40);   // instantiate
reading.switchOn();            // send a message
```

### 3.5 Representational independence
Outside code uses `isOn()`, never the field `on` directly. That lets you later change *how* the state is stored (say, as a brightness level) without breaking callers. This is the book's term for what is usually called **encapsulation**. Rule of thumb: **fields private, behaviour public**.

### 3.6 Overloading
Several methods may share a name if their **parameter lists differ** (number or types). The compiler picks one by looking at the arguments. Return type alone does **not** distinguish overloads.

### 3.7 Initialization and constructors
- If you write *no* constructor, Java gives you a default no-argument one. As soon as you write any constructor, the default disappears.
- Fields get default values (`0`, `false`, `null`) if you don't set them; **local variables don't** — the compiler forces you to assign them.
- A constructor is the right place to guarantee the object starts in a **valid state** (an *invariant*).

### 3.8–3.12 Control flow, arrays, return values (skim)
- `if/else`, `switch`, `for`, `while`, `do/while`, `break/continue`; blocks `{ }` limit variable scope.
- Arrays are objects with a fixed length: `int[] a = new int[5];` → `a.length`, zero-based indexing.
- A method returns exactly one value (or none with `void`). Parameters are **passed by value** — for object parameters, the *reference* is copied (so a method can mutate the object, but can't re-point the caller's variable).

### Modern Java footnote (not in the book)
Since Java 10 you may write `var x = 42;` or `var name = "hello";` for **local** variables (type inferred at compile time — still statically typed). Since Java 16, `record Point(int x, int y) {}` declares an immutable data class in one line. Hands-on practice for `var`, records, and related features is on **Fri** per `CURRICULUM.md` Week 1.

---

## 4. Key terms (glossary)

object · class · instance · state / field · behaviour / method · message · parameter (argument) · receiver / sender · client / server · information hiding · representational independence (encapsulation) · constructor · overloading · primitive type · reference · `null` · bytecode · JVM

## 5. Common pitfalls

1. **`==` vs `.equals()`** on `String` and other objects. `==` compares references.
2. **`NullPointerException`**: calling a method on a `null` reference. Initialize references; know which can be `null`.
3. **Public fields**: anyone can break your object's rules. Keep fields `private`.
4. **Forgetting `new`** or declaring a variable but never instantiating — you hold `null`, not an object.
5. **Constructor with a return type** (e.g. `public void Lamp()`): it becomes an ordinary method, not a constructor.
6. **Shadowing**: a parameter named like a field hides the field; use `this.field = field`.
7. **Integer division**: `5 / 2` is `2`; use a `double` operand for `2.5`.
8. **Confusing a class with an object**: `Lamp.switchOn()` vs `reading.switchOn()` — instance methods need an instance.
9. **Overloading ≠ overriding**: overloading = same name, different parameters (this chapter); overriding comes with inheritance (Week 2).
10. **Mutating a parameter and expecting the caller's variable to change**: Java is always pass-by-value (of the reference).

## 6. Primer: when to use a class vs a function

Not every piece of code needs a class. Ask these questions in order:

1. **Does it need to remember something between calls (state)?**
   - No → a **static function** (a method taking inputs and returning output) is usually enough. Example: converting units, validating an email format, computing a tax amount.
   - Yes → continue.
2. **Is that state something you must protect rules about (e.g. balance never negative)?**
   - Yes → a **class**: private fields + methods that enforce the rules.
3. **Is it just a bundle of values with no rules beyond "these belong together"?**
   - Yes → a **record** (modern Java) — immutable data carrier.
4. **Will different implementations of the same ability exist, or do you want to swap/mock it?**
   - Yes → an **interface** (+ classes implementing it). (Week 2.)

Heuristics:
- **Nouns with lifecycle and rules → classes** (`Account`, `Order`, `Connection`).
- **Verbs/calculations → functions** (`calculateTax`, `parseDate`).
- **Don't create a class named `XxxManager`/`XxxHelper` that is only a bag of static methods unless the methods are truly stateless utilities.** Conversely, don't force a class on a one-line calculation.
- **Pure functions** (same input → same output, no side effects) are the easiest code to test and to run concurrently. You'll come to value that in Weeks 7–8.
- Java has no free-standing functions: a "function" lives as a `static` method in some class (e.g. `Math.max`). Keep such utility classes final, stateless and small.

---

## 7. Hands-on (Mon — optional prep; weekly code starts Wed)

Week 1 targets ~**30 min of code per day**; the **five proof exercises** live in `CURRICULUM.md` Week 1 (**#1** bank account on **Wed**, not today). Use today's slot for the lecture + book; add the items below only if you have time — they are **not** extra proof obligations.

**Optional notes (no code required):**

1. **Message tracing.** In `notes/`, describe sender, receiver, message, parameters, and method for: *"A customer asks a barista for a flat white with oat milk; the barista replies with the price."* Optionally implement as two classes in `exercises/2026-10-05-ch1-3/` if you want practice before **#1**.
2. **Overload sketch.** On paper or in `notes/`, sketch three overloads of one method name with different parameter lists and write two sentences on how the compiler picks an overload for `int` vs `long` vs `String`.

**Optional code (scratch folder `exercises/2026-10-05-ch1-3/`):** a small domain class (two private fields, constructor, three methods, two instances in `main`) using any domain you like — including a bank account if you are eager, but **Wed #1** is the canonical bank-account exercise for the proof set.

## 8. Self-check (answer in `notes/`, then check against the book)

1. In the Benjamin/Sean example, which is the message, what are its parameters, and what is the method? What does "information hiding" mean there?
2. What is the difference between a class and an object, and what keyword links them?
3. What does a constructor do, and what happens if you declare no constructor? What if you declare one with parameters only?
4. Given `String a = new String("hi"); String b = new String("hi");`, what do `a == b` and `a.equals(b)` give, and why?
5. Java is "pass by value". Explain what that means when you pass an object to a method, and state what a method can and cannot change about the caller's variable.

*Stuck for more than 15 min? Write the exact error/question in `notes/` and ask for a review — hints and feedback, not solutions.*
